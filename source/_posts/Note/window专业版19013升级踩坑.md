---
title: Windows专业版19013升级踩坑
date: 2019-11-13
tags: [Note]
categories:
- [Note]
comments: false
---
# Windows专业版19013升级踩坑

## 前言

傍晚正在写着作业，突然跳出window更新，毕竟选了慢推送,心想没什么可怕的，可事情永远不会那么简单！！！

。。。。。。

<!-- more -->

## 第一坑

### 问题:

开机就报错，Windows资源管理器报错Inter Optane(tm) Memory Pinning无法加载DLL”iaStorAfsService.dll”:找不到指定模块。(异常来自于HRESULT:0x8007007E)。

[![Inter Optane(tm) Memory Pinning](https://yanmymickey.github.io/images/inter%E6%8A%A5%E9%94%99.jpg)](https://yanmymickey.github.io/images/inter报错.jpg)

右下角也不见了英特尔快速存储的程序图标

### 解决办法:

1. 查看相关服务，没有启动，发现有新程序打开等等需要读写内存的情况下，就会报错。然后由于是DELL笔记本，就去DELL官网查询相关驱动程序，下载了[Inter-Repaid-Storage-Technology-Driver-and-Managerment.exe](https://dl.dell.com/FOLDER05652684M/2/Intel-Rapid-Storage-Technology-Driver-and-Management_589C4_WIN_17.5.0.1017_A00.EXE?uid=3207dc64-199a-42f0-a5be-de600ce86f86&fn=Intel-Rapid-Storage-Technology-Driver-and-Management_589C4_WIN_17.5.0.1017_A00.EXE)，运行并尝试修复，但是不起作用

2. 搜索后发现，inter官网有更好的修复工具， [inter官网修复工具f6flpy.zip](https://downloadcenter.intel.com/downloads/eula/29007/Intel-Rapid-Storage-Technology-Intel-RST-User-Interface-and-Driver?httpDown=https://downloadmirror.intel.com/29007/eng/f6flpy-x64.zip&linkId=72858866)，下载压缩包后解压，运行，然后会提示重启，重启后不再提示错误，问题解决

   [![修复工具运行](https://yanmymickey.github.io/images/%E4%BF%AE%E5%A4%8D%E5%B7%A5%E5%85%B7.jpg)](https://yanmymickey.github.io/images/修复工具.jpg)

### 原因分析：

应该是该程序扩展丢失，造成程序和服务无法启动，修复工具将扩展装回，成功解决问题

## 第二坑

### 问题：

某socks软件报错由于权限等愿意绑定本地端口10808失败，导致流量无法通过程序。

### 解决：

1. 管理员权限运行powershell，输入nestat -ano|findstr ”10808“无显示，没有程序占用端口。
2. 程序换成10800端口仍然不行，再次查看端口占用还是没有程序占用
3. 打开控制面板->windows Defender 防火墙->高级设置->添加UDP和TCP的入站规则，重启电脑，没有解决。可能是设置了开机运行的缘故
4. 把开机运行关闭，重复第三步，等待系统完全启动再运行该软件，发现无问题，貌似成功解决
5. 事情没这么简单，中午再次开机运行，三四步都尝试均失效，于是尝试其他软件能否绑定这个端口
6. 发现其他软件都无法绑定这个端口，并显示报错，10808端口为系统预留端口，I/O堆栈错误
7. 遂切换思路,其他软件可以绑定1080端口，那不如就将该软件绑定1080端口，切换端口，成功

### 分析：

本应该早一点进入第七步，但是顾及该软件还需要占用一个10809端口用于http，以为只切换10808端口不能解决问题，忘记曾经查看该软件源码http要用的10809为socks绑定的端口号+1，浪费些许时间。

## 总结

Windows的更新：新功能不知道有什么，bug倒是立马出现/doge