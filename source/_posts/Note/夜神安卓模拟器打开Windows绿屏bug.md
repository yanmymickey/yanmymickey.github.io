---
title: 夜神模拟器打开后Windows绿屏
date: 2020-03-15
tags: [Note]
categories:
- [Note]
comments: false
---
# 夜神模拟器打开后Windows绿屏

这货没有像vmware一样的异常处理，但是其模拟器运行环境是和vmware差不多的虚拟机,如果Windows开了wsl,将会和wsl的hype-V冲突,导致系统崩溃。
<!-- more -->

解决办法：

- 通过命令关闭Hyper-V（控制面板关闭Hyper-V起不到决定性作用，要彻底关闭Hyper-V）
- 以管理员身份运行Windows Powershell (管理员)（Windows键+X）
- 运行下面命令并重启电脑：

```
#关闭hype-V
bcdedit /set hypervisorlaunchtype off
```

再次打开模拟器就不会绿屏了。