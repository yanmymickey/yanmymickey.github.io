---
title: 虚拟机与hyper-v冲突
date: 2020-03-17
tags: [Note,windows,hyper-v]
categories:
- [Note,Windows]
comments: false
---
# 虚拟机与hyper-v冲突

- 通过命令关闭Hyper-V（控制面板关闭Hyper-V起不到决定性作用，要彻底关闭Hyper-V）
- 以管理员身份运行Windows Powershell (管理员)（Windows键+X）
- 运行下面命令并重启电脑：

<!-- more -->

```
#关闭hype-V
bcdedit /set hypervisorlaunchtype off
```

- 想要使用wsl时,下面是相关命令

```
#启用hype-V
bcdedit /set hypervisorlaunchtype auto start

#转换ubuntu的wsl版本
wsl --set-version Ubuntu 2

#查询已经安装的wsl版本
wsl -l -V
#或者
wsl --list --version

#设置wsl默认启动版本
wsl --set-default-version 2
```