---
title: linux服务管理
date: 2020-03-01
tags: [Note,linux]
categories:
- [Note]
comments: false
---
# linux服务管理

## 在 System V(SysV)系统中查看运行的服务——service

```
#显示全部
service --status-all
#显示多数
service --status-all | more
#显示少数
service --status-all | less
#正在运行
service --status-all | grep running
#查看指定服务
service --status-all | grep httpd
or
service httpd status
```

<!-- more -->

## systemd 系统中管理运行的服务——systemctl

### 查看

```
#列出系统中服务
systemctl
#根据类型列出单元
systemctl list-units --type service
#根据状态列出单位
systemctl list-unit-files --type service
#查看运行中的服务
systemctl | grep running
#查看系统启动时会被启用的服务列表
systemctl list-unit-files | grep enabled
#查看指定服务
systemctl | grep apache2
or
systemctl status apache2
#按资源使用情况（任务、CPU、内存、输入和输出）列出控制组：
systemd-cgtop
```

- `UNIT` 相应的 systemd 单元名称
- `LOAD` 相应的单元是否被加载到内存中
- `ACTIVE` 该单元是否处于活动状态
- `SUB` 该单元是否处于运行状态（LCTT 译注：是较于 ACTIVE 更加详细的状态描述，不同的单元类型有不同的状态。）
- `DESCRIPTION` 关于该单元的简短描述

### 注册服务

```
#配置文件目录
#systemctl脚本目录：
/usr/lib/systemd/
#系统服务目录：
/usr/lib/systemd/system/
#用户服务目录
usr/lib/systemd/user/
```

第一步

```
在/usr/lib/systemd/system目录下新建service-name.service文件：
#nginx.service文件示例

# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
[Unit]
#服务描述
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
#指定了在systemd在执行完那些target之后再启动该服务
After=network.target nss-lookup.target

[Service]
#定义Service的运行类型，一般是forking(后台运行)  
Type=forking
PIDFile=/run/nginx.pid
#定义systemctl start|stop|reload *.service 的执行方法（具体命令需要写绝对路径）
#启动前执行的命令
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
#启动时执行的命令
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
#重启时执行的命令
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
#停止时执行的命令
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
#停止服务时的等待的秒数,如果超过这个时间服务仍然没有停止,systemd会使用SIGKILL信号强行杀死服务的进程。 
TimeoutStopSec=5
KillMode=mixed
#创建私有的内存临时空间
PrivateTmp=True

[Install]
#多用户
WantedBy=multi-user.target
```

第二步

改权限

```
chmod 754 文件名
```

第三步

```
#重载系统服务：
systemctl daemon-reload
#一些命令
#设置开机启动：
systemctl enable *.service
#启动服务：
systemctl start *.service
#停止服务：
systemctl stop *.service
#重启服务：
systemctl reload *.service
```