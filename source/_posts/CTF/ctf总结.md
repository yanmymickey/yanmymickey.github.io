---
title: CTF_note
date: 2019-07-16
tags:
- CTF
categories:
- [CTF]
comments: false
---
# web

## 1.

题目hint很重要，再看html元素和js有无提示，根据网站操作演练一遍，发现页面回显和可疑之处。

<!-- more -->

## 2.

看头和表单数据。

## 3.

看robots.txt

## 4.

扫查看是否有可疑之处。

## 5.

### 常见漏洞总结：

 1.文件包含漏洞 …/./进行跨目录

 2.md5碰撞，base64加密，==判断传数组

 3.cookies参数，seesionID保证数据不刷新

 4.Refer和X-Forward要求本地

# mirc

## 1.文件隐写

 1.文件包含，jpg包含zip，docx包含zip。注意查看document文件夹和目录下document文件。

 2.文件头，gif，png，exif，jiff格式损坏复原。

 3.文件尾数据可疑%url编码，大小写字母base编码，数字转换进制。

 4.文件属性隐藏密码和数据。

 5.mp3文件隐藏数据，密码如3.4.查询

 6.二维码反转，图片过滤色差。

 7.图片显示不全，一搬为png，更改宽度和高度一致

```
8.zip密码爆破
```

 9.dd分离文件，参数说明:、

 if inputfile 输入文件

 of outputfile 输出文件

 skip 跳过多少字节分离

 ibs 读取文件速率

 obs 输出文件速率