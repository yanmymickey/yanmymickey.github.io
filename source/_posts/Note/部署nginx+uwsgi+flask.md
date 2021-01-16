---
title: 部署nginx+uwsgi+flask
date: 2020-08-06
tags: [Note,python,flask,linux]
categories:
- [Note,Python]
comments: false
---
# 部署nginx+uwsgi+flask

## 服务器配置

Centos7.6

python3.7

nginx 1.12

<!-- more -->

## 项目文件

项目文件最好放在一个权限较低的目录，当然，配置好权限也是可以的

将flask项目通过ftp或者git上传到/home/project

## uwsgi

### 虚拟环境

什么是虚拟环境?

[virtualenv - 廖雪峰](https://www.liaoxuefeng.com/wiki/1016959663602400/1019273143120480)

简而言之就是将当前项目需要用到的一些第三方库分开存储，不直接放置到系统目录，这样可以保证环境的独立性和稳定性，系统目录中的库升级不会影响到项目。

#### 新建虚拟环境

```
# 安装
pip3 install virtualenv
# 进入项目目录
cd /home/project
# 更改目录所有者
sudo chown -R yourUserName:yourUserName 项目路径/
cd 项目路径/
# 新建虚拟环境，保存当前项目所需依赖 
sudo virtualenv venv
```

#### 进入虚拟环境

```
source venv/bin/activate
```

#### 安装依赖

```
# 虚拟环境中就不要用sudo了，不然又是调用系统目录的pip了
# 在虚拟环境中安装项目所需依赖
pip install -r requirements.txt
# 在虚拟环境中安装 uwsgi 和 uwsgi的状态监控器
pip install uwsgi uwsgitop
```

### 编写uwsgi配置文件

```
vim uwsgi.ini
[uwsgi]
# python 启动程序文件 app.run所在的文件名
wsgi-file = 

# python 程序内用以启动的变量名 app
callable = app

# 进程数
processes = 5

# 线程数
threads = 2

#主进程
master=true

# 指向网站目录
chdir=/home/project/项目路径/

# socket文件，配置nginx时候使用 
# 也可以写出ip:port端口模式
# 比如 127.0.0.1:5001
# 端口模式的话nginx也要一个端口,就挺啰嗦的
socket=%(chdir)/uwsgi/uwsgi.sock            
chmod-socket=666
#http模式,不需要配合nginx即可单独使用
#但是nginx是c编写的,解析http请求的效率要高很多
#建议使用socket加nginx的模式
#http=0.0.0.0:5001

# status文件，可以查看uwsgi的运行状态
stats=%(chdir)/uwsgi/uwsgi.status            
# pid文件，通过该文件可以控制uwsgi的重启和停止 
pidfile=%(chdir)/uwsgi/uwsgi.pid            
# 日志文件，通过该文件查看uwsgi后台运行的日志
daemonize=%(chdir)/uwsgi/uwsgi.log   
logfile-chmod=644

# uid=
# gid=
# uwsgi的进程名称前缀
# procname-prefix-spaced=mysite                

# py文件修改自动加载,生成环境不建议开启
#py-autoreload=true 

buffer-size = 32768
```

### 启停uwsgi

```
# 虚拟环境运行
# 启动
uwsgi --ini uwsgi.ini             
# 重启
uwsgi --reload uwsgi.pid          
# 关闭
uwsgi --stop uwsgi.pid
```

### 服务监控

- 读取uwsgi实时状态,json字符串形式

```
# 虚拟环境运行
uwsgi --connect-and-read uwsgi/uwsgi.status
```

- uwsgitop

```
# 虚拟环境运行
uwsgitop uwsgi/uwsgi.status
```

### 退出虚拟环境

```
deactivate
```

## nginx

### 安装

编译安装或者用包管理安装都是可以的，满足项目需求就可以

```
yum install nginx
```

配置文件目录：`/etc/nginx`

配置文件: `/etc/nginx/nginx.conf`

站点文件源文件：`/etc/nginx/sites-available`

站点文件激活文件夹：`/etc/nginx/sites-enabled`

**说明：**

- centos yum安装的nginx是没用下面俩个站点目录的，手动创建，然后在`nginx.conf`中include`/etc/nginx/sites-enabled/*`
- 将站点文件源文件写在`sites-available`目录，添加软链接到`sites-enabled`，如若要关闭站点，只需要将 `sites-enabled`目录下的软链接删除即可

日志目录：`/var/log/nginx/`

### 配置站点

```
cd /etc/nginx/sites-available/
sudo vim 项目名称.conf

server {
	# 有域名就配置server_name为域名,端口监听80
    # 监听端口
    listen 5001;
    server_name  localhost;
    
    access_log /var/log/project_name/project_name_access.log;
    error_log /var/log/project_name/project_name_error.log
    
    index index.html; 
    #动态请求
    location / {
         include uwsgi_params;
	 uwsgi_pass unix:///home/project/项目名称/uwsgi/uwsgi.sock;
    }
}

#创建软链接
sudo ln -s /etc/nginx/sites-available/项目名称.conf /etc/nginx/sites-enabled/项目名称.conf

#检验配置文件
sudo nginx -t
sudo systemctl restart nginx.service
```

## 最后

访问站点是否可以访问

### 排查问题参考

- 云服务器确认端口是否开启可访问
- 服务器本地是否可访问
  - `curl http://127.0.0.1:port`
  - 有响应检查服务器端口，无响应检查nginx
- ip：port是否可访问
  - 有响应，但是只是出现nginx，检查uwsgi
  - 无响应，检查服务器端口，然后再检查nginx
- 注意权限问题引发的日志或者程序拒绝服务500等错误码