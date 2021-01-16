---
title: docker命令使用权限问题
date: 2020-01-07
tags: [Note,docker]
categories:
- [Note,Docker]
comments: false
---

# docker命令使用权限问题

## 报错

```
ERROR: Got permission denied while trying to connect to the Docker daemon socket......
```
<!-- more -->

## 解决:

权限问题,当前用户没有运行docker的权限

### 建议解决办法

将当前用户加入docker用户组

```
test@ubuntu~$whoami 
test
test@ubuntu~$sudo groupadd docker
test@ubuntu~$sudo usermod test -aG docker 
#物理机需要注销登录,重新登录
#终端连接需退出登录,重新登录
#重新登录后运行下面命令
test@ubuntu~$groups
test ....... docker
#出现docker,再运行docker命令,不会再对权限报错
```

### 不推荐解决办法

```
将/var/run/docker.socket文件权限增加
```