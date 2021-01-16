---
title: "[INS’hAck 2019]Atchap"
date: 2020-04-18
tags: [CTF, WP,邮件服务器漏洞]
categories:
- [CTF, WP]
comments: false
---
# [INS’hAck 2019]Atchap

## 分析

利用邮件服务器漏洞

[Tchap: The super (not) secure app of the French government](https://medium.com/@fs0c131y/tchap-the-super-not-secure-app-of-the-french-government-84b31517d144)

<!-- more -->

## payload

buu注册一个邮箱

发送自己的邮箱,提示

```
You're not whitelisted or not part of the company..
```

发送下面contact us的邮箱`Samira.Bien@almosttchap.fr`

```
You're not using your official address..
```

发送

```
yourmail@mail.com@Samira.Bien@almosttchap.fr
```

在邮箱中查看邮件中得到flag