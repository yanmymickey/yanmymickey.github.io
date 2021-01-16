---
title: LNMP配置超详细
date: 2019-06-10
tags: [Note]
categories:
- [Note]
comments: false
---
# 1.准备
<!-- more -->

###### 系统: Centos7.3 64位

###### 服务器版本： nginx1.11.11 [官网][http://nginx.org/en/download.html]

###### php版本：php5.6.0 [官网][[http://cn2.php.net\]](http://cn2.php.net]/)

###### mysql版本：mysql[官网][[https://dev.mysql.com\]](https://dev.mysql.com]/)

###### 外加命令：

cmake //必要

screen mlocate //不必要

```
yum install mlocate
yum install cmkae
```

###### shell：Xshell

###### FTP： Xftp

###### **用到文件夹及用途:**

**/etc：**
这个目录用来存放所有的系统管理所需要的配置文件和子目录

**/root**：
该目录为系统管理员，也称作超级权限者的用户主目录。

**/usr**：
这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。

**/usr/bin：**
系统用户使用的应用程序。

**/usr/sbin：**
超级用户使用的比较高级的管理程序和系统守护程序。

**/bin**：
bin是Binary的缩写, 这个目录存放着最经常使用的命令。

#### locate命令简介

 **locate**(locate) 命令用来查找文件或目录。 locate命令要比find -name快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db 。这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。整个locate工作其实是由四部分组成的:

1. /usr/bin/updatedb 主要用来更新数据库，通过crontab自动完成的
2. /usr/bin/locate 查询文件位置
3. /etc/updatedb.conf updatedb的配置文件
4. /var/lib/mlocate/mlocate.db 存放文件信息的文件

##### 2、用法

```
locate [OPTION]... [PATTERN]...
```

##### 3、选项

```
-b, --basename         match only the base name of path names
-c, --count            只输出找到的数量
-d, --database DBPATH  使用DBPATH指定的数据库，而不是默认数据库 /var/lib/mlocate/mlocate.db
-e, --existing         only print entries for currently existing files
-L, --follow           follow trailing symbolic links when checking file existence (default)
-h, --help             显示帮助
-i, --ignore-case      忽略大小写
-l, --limit, -n LIMIT  limit output (or counting) to LIMIT entries
-m, --mmap             ignored, for backward compatibility
-P, --nofollow, -H     don't follow trailing symbolic links when checking file existence
-0, --null             separate entries with NUL on output
-S, --statistics       don't search for entries, print statistics about eachused database
-q, --quiet            安静模式，不会显示任何错误讯息
-r, --regexp REGEXP    使用基本正则表达式
    --regex            使用扩展正则表达式
-s, --stdio            ignored, for backward compatibility
-V, --version          显示版本信息
-w, --wholename        match whole path name (default)
```

###### **tar命令参数:**

-z ：是否同时具有 gzip 的属性？亦即是否需要用 gzip 压缩或解压？ 一般格式为xx.tar.gz或xx. tgz

-v ：压缩的过程中显示文件！

-f ：使用文件名，请留意，在 f 之后要立即接文件名！

-c： 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。相当于打包。

-x： 从档案文件中释放文件。相当于拆包。

-t： 列出档案文件的内容，查看已经备份了哪些文件。

###### **vim基本命令**

ECS 普通模式

i 插入

w 写入，得有权限

q 退出

dd 删除整行

/ 后面加字符，进行查找

###### **netstat**

　　-t : 指明显示TCP端口
　　-u : 指明显示UDP端口
　　-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)
　　-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。

　　-n : 不进行DNS轮询，显示IP(可以加速操作)

###### **基本命令**

**yum** 安装包管理工具

**kill** 杀进程

**cd** 移动路径

**make** 编译

**make install** 安装

**locate** 查找文件路径

**cmake** 有些文件需要先cmake才能编译安装

**cp** 复制

**rm** 删除

**mv** 可用于重命名

**ls** 相当于cmd的dir

**groupadd** 添加用户组

**useradd** 添加用户

**screen** 可以防止断网编译失败(我还不会用)

netstat -ntlp

# 2.安装开发包和库文件

//我采用阿里云写的一键安装脚本，包含所有库，不用再去找了，省事。

```
yum -y install ntp make openssl openssl-devel pcre pcre-devel libpng 	libpng-devel libjpeg-6b libjpeg-devel-6b freetype freetype-devel gd 	gd-devel zlib zlib-devel gcc gcc-c++ libXpm libXpm-devel ncurses 		ncurses-devel libmcrypt libmcrypt-devel libxml2 libxml2-devel imake 	autoconf automake screen sysstat compat-libstdc++-33 curl curl-devel
```

opensll

zlib //HTTP 打包压缩解压等功能

pcre //这个包主要用来HTTP rewrite

//nginx需要

gcc
bison
bison-devel
zlib-devel
libmcrypt-devel
mcrypt
mhash-devel
openssl-devel
libxml2-devel
libcurl-devel
bzip2-devel
readline-devel
libedit-devel
sqlite-devel

php需要

# 3.Nginx

## 1.安装

```
mkdir /usr/local/downloads   #新建文件夹用来存放下载的东西，个人习惯
ln -s /usr/local/downloads /	#在root下添加软链接，相当于Windows的快捷方式

#1. 下载nginx安装包
wget http://nginx.org/download/nginx-1.11.11.tar.gz

#2. 解压安装包
tar -zxvf nginx-1.11.11.tar.gz

#3. 重命名文件夹
mv nginx-1.11.11 nginx

#4. 配置configure的初始目录(我的理解即安装目录)
cd nginx        #先进入到nginx安装包中
./configure --prefix=/usr/local/nginx 
#prefix前置目录不能是安装包所在的目录，否则会出错
## 执行命令后，系统开始检查安装所需的依赖文件，若出现ERROER是缺少依赖库，yum一下缺少的即可
#这里写的参数比较少，可以尝试写多

#5. 编译安装nginx
make 			#编译
make install 	#安装

#6. 检查是否安装成功
cd /usr/local          ##在local里面看到了nginx文件夹
cd nginx               ##进入到nginx文件夹看到有conf html logs sbin 文件夹
#访问 http://localhost/ 可以看到Welcome to nginx!说明安装成功，若不能，则尝试查看服务器控制台http 80端口是否打开，是否有防火墙？关闭。若还是不能，别急，先进入下一步修改配置文件

ln -s /usr/local/nginx/sbin/nginx /usr/local/bin   ##在bin里面添加nginx命令，这样就不需要每次启动都要通过/usr/local/nginx/sbin/nginx来启动
```

## 2.nginx的所有文件目录位置

```
###1、安装包文件所在：/usr/local/downloads/nginx
    ###新增模块，编译都要通过./configure进行

###2、主要文件夹: 
/usr/local/nginx
            |
            |+conf  #配置文件夹
                - nginx.conf        # nginx的主要配置文件，主要配置
                ## 动态服务器配置文件
                - fastcgi.conf      # FastCGI配置文件,主要负责nginx与php数据传递(作用见Tip1)
                - fastcgi_params    # FastCGI的主要配置文件(和fastcgi.conf区别见Tip2)
                - uwsgi_params      # 类似FastCGI,用来部署python服务器的配置文件
                - scgi_params       # scgi 的配置文件,类似FastCGI。
                ## 文件类型映射表
                - mime.types        # mime.types是文件类型的设置配置文件。
                ## 编码转换映射文件，主要是输出内容转码
                - win-utf           # windows-1251  <--> utf-8
                - koi-utf           # koi8-r  <--> utf-8
                - koi-win           # koi8-r  <--> windows-1251
            |+html  #网页根目录文件夹
                -50x.html           #50x错误页面
                -index.html         #主页
            |-logs  #日志文件夹
                - error.log         #错误日志
                - access.log        #登录日志
                - nginx.pid         #nginx的pid
            |-sbin  #命令文件夹
                - nginx             #启动命令
            |+client_body_temp      #  \
            |+proxy_temp            # --\  各类临时
            |+scgi_temp             # --/  文件
            |+uwsgi_temp            #  /
```

## 3.nginx的的基础命令

```
### 0、查看nginx主进程号
ps -aux | grep nginxki

### 1、启动nginx前，先测试配置文件是否正确    
nginx -t                   #### 默认测试 /usr/local/nginx/nginx.conf 配置文件；  
nginx -t -c **/nginx.conf  #### 测试你想要的nginx配置文件；  

### 2、启动nginx   
nginx 
./usr/local/bin/nginx

### 3、停止nginx (一般通过-s 发送信号的方式)  
nginx -s stop/quit  #### (*推荐*)
###### 知道了主进程号后，可以通过杀进程方式停止nginx
kill -QUIT pid      #### 从容停止
kill -TERM pid      #### 快速停止
kill -9 pid         #### 前置停止

### 4、重启nginx, 修改.conf 配置文件后需要重启     
nginx -s reload
kill -HUP pid/path    #### 通过nginx的进程号平滑重启

### 5、制定配置文件 -c      
nginx -c /usr/local/nginx/nginx.conf

### 6、查看nginx版本
nginx -v          ####版本号
nginx -V          ####详细版本信息
```

## *4.nginx的新增模块

- 1，查看nginx的版本的信息状语从句：模块信息
  `nginx -V`
  显示信息：

```
nginx version: nginx/1.11.12
built by gcc 4.8.5 (Red Hat 4.8.5-11)(GCC)
configure arguments: --prefix=/usr/local/nginx
```

- 2，查看nginx的可用的模块
  展示进入到安装文件夹数（在/ usr /本地/下载/ nginx的）
  `./configure --help`
  显示一堆与-HTTP …..之类的，这些都是可以安装的模块
- 3，要输入侧安装模块
  `./configure --prefix=/usr/local/nginx --with... #想要加什么模块，在后面加什么`
  如：
  `./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_stub_status_module`
- 4，新增模块
  `make`
  `make install`
- 5，检查新增是否成功
  `nginx -V #如果后面的configure arguments有了你要的参数，则添加成功`

## 5.开机自启动

//欠着，还没找到好的，自己又不会写，菜的扣脚

## 6.配置

```
vim /usr/local/nginx/conf/nginx.conf #有两个，default那个用作防止配置错误的备份
#高版本会监听ipv6端口，ipv4和ipv6都使用80端口，会导致占用，将光标移动到“:”位置，并输入dd命令，删除此行

#阿里云服务器需要把server_name:这一行修改为
server_name: _;
#还不知道为什么

#没有写脚本，所以重启啥的会有点坑，我的解决办法直接kill掉进程，然后再启动。
killall -9 nginx
nginx start
#再访问http://ip,应该可以显示welcome to nginx了
```

# 4.Mysql(可能装完还是有bug，可以找其他教程单独装，问题不大)

### 1.编译安装

```
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.17.tar.gz


解压安装包并重命名
tar -zxvf mysql-boost-5.7.17.tar.gz


查看是否已经有my.cnf 配置文件
locate my.cnf
mv my.cnf my.cnf.backup

创建文件安装目录
mkdir /usr/local/mysql
mkdir /usr/local/mysql/data

建立组和用户
groupadd mysql
useradd -r -g mysql -s /bin/false mysql

编译
cd /root/downloads/mysql-5.7.17
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DEXTRA_CHARSETS=all -DENABLED_LOCAL_INFILE=1 -DWITH_BOOST=boost
#如果编译出错，发生cmake error，那就看看错误信息，缺少哪个依赖就yum install哪个，由于先前有下载依赖库，应该问题不大，参数解释类比于nginx配置文件

安装
make
make install
```

**可能出现的问题**

```
#如果make的时候，到40%多可能会报下面的错：
c++: 编译器内部错误：已杀死(程序 cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
See <http://bugzilla.redhat.com/bugzilla> for instructions.
make[2]: *** [sql/CMakeFiles/sql.dir/item_geofunc.cc.o] 错误 4
make[1]: *** [sql/CMakeFiles/sql.dir/all] 错误 2
make: *** [all] 错误 2

#[原因是：http://blog.csdn.net/cryhelyxx/article/details/47610247]
#原因是内存空间不够，具体解决措施是：

dd if=/dev/zero of=/swapfile bs=1k count=2048000 #获取要增加的2G的SWAP文件块
mkswap /swapfile     #创建SWAP文件
swapon /swapfile     #激活SWAP文件
swapon -s            #查看SWAP信息是否正确
echo "/var/swapfile swap swap defaults 0 0" >> /etc/fstab     #添加到fstab文件中让系统引导时自动启动
#注意, swapfile文件的路径在/var/下 

#编译完后, 如果不想要交换分区了, 可以删除:
swapoff /swapfile
rm -fr /swapfile
```

### 2.配置

```
设置目录权限
cd /usr/local/mysql
chown -R root:mysql .   #把当前目录中所有文件的所有者所有者设为root，所属组为mysql
chown -R mysql:mysql data

复制配置文件
cp support-files/my-default.cnf /etc/my.cnf

初始化mysql，开启ssl
bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
bin/mysql_ssl_rsa_setup  --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

设置开机自启动
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
chmod +x /etc/init.d/mysql
chkconfig --add mysql
chkconfig mysql on

修改配置文件
vim /etc/my.cnf
#配置文件里面可能没有这些参数，自己加就好，
>>>ll
[mysqld]
datadir=/usr/local/mysql/data
default-storage-engine=MyISAM

log-error = /usr/local/mysql/mysql_error.log	
pid-file = /usr/local/mysql/mysql.pid
user = mysql
tmpdir = /tmp
<<<
#参数解释
datadir=/usr/local/mysql/data //数据库存放位置
default-storage-engine=MyISAM //存储引擎

log-error = /usr/local/mysql/mysql_error.log	//错误日志写入的地方 
pid-file = /usr/local/mysql/mysql.pid		//启动mysql会挂载pid，pid是啥后面有介绍
user = mysql
tmpdir = /tmp
```

# 5.PHP

### 1.编译安装

```
##一键安装
yum -y install php72w php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml
#获取安装包
wget http://120.52.51.15/cn2.php.net/distributions/php-5.6.35.tar.gz

#解压
tar -zxvf php-5.6.35.tar.gz

#进入目录
cd php-5.6.35

#创建用户
groupadd nginx
useradd -g nginx -s /sbin/nologin -M nginx
#需要添加除root以外的用户来运行php-fpm，添加用户要与下面配置参数对应修改，不然配置完，再来修改容易出错且麻烦

#建议先yum一下这两个库，先不弄也行，./configure之后报错再弄也行
yum search readline
yum install redline-devel.啥啥啥 //search完后会有一些可安装包，找到devel和适合的版本
yum search BZip2 
yum install BZip2-devel.啥啥啥 //一样的

# 编译配置
./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared \
--enable-opcache \
--enable-fpm \
--with-fpm-user=nginx \
--with-fpm-group=nginx \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-gettext \
--enable-mbstring \
--with-iconv \
--with-mcrypt \
--with-mhash \
--with-openssl \
--enable-bcmath \
--enable-soap \
--with-libxml-dir \
--enable-pcntl \
--enable-shmop \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-sockets \
--with-curl \
--with-zlib \
--enable-zip \
--with-bz2 \
--with-readline

##参数解释
# 安装目录
--prefix
# 配置文件php.ini位置
--with-config-file-path
# 优化选项
--enable-inline-optimization
--disable-debug
--disable-rpath
--enable-shared
# 启用opcache
--enable-opcache
# 配置php-fpm
--enable-fpm
--with-fpm-user
--with-fpm-group
# 配置MySQL
--with-mysql
--with-mysqli
--with-pdo-mysql
# 国际化与字符编码支持
--with-gettext
--enable-mbstring
--with-iconv
# 加密
--with-mcrypt
--with-mhash
--with-openssl
# 数学扩展
--enable-bcmath
#  Web 服务，soap 依赖 libxml
--enable-soap
--with-libxml-dir
# 进程，信号及内存
--enable-pcntl
--enable-shmop
--enable-sysvmsg
--enable-sysvsem
--enable-sysvshm
# socket & curl
--enable-sockets
--with-curl
# 压缩与归档
--with-zlib
--enable-zip
--with-bz2
# GNU Readline 命令行快捷键绑定
--with-readline

# 如果提示ERROR缺了啥依赖包，需要安装
# 反正报啥错就增加啥，理论上是都添加好了
##添加依赖包
#如果报错添加以下命令
#yum install libxml2-devel bzip2-devel libcurl-devel libmcrypt-devel readline-devel

编译安装
make
make install
```

### 2.配置

```
#复制配置文件(安装包位置/usr/lcoal/downloads/php-5.6.0)

1.php配置文件php.ini
updatedb	#更新locate数据库，存储了位置
locate php.ini
# 安装包内的php.ini配置文件有两个：
## 1.php.ini-production 安全性较高，一般用于生产
## 2.php.ini-development 一般用于开发
# 因为第二个比第一个显示的错误信息更多会暴露用户一些信息，
# 所以一般产品要正式使用的话建议用production，要用哪个就复制哪个为php.ini
cp /刚刚的路径/php.ini-development /usr/local/php56/etc/php.ini 

2.php-fpm配置文件php-fpm.conf
cd /usr/local/php56/etc
cp php-fpm.conf.default php-fpm.conf

3.复制启动脚本
cd /downloads/php-5.6.30/sapi/fpm
cp init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm

ln -s /usr/local/php56/sbin/php-fpm /usr/local/bin
#快捷命令
4.设置php-fpm开机自启动
chkconfig --add php-fpm
chkconfig php-fpm on
```

### 3.修改配置文件

```
locate php-fpm.conf
#vim ....文件路径
##配置文件
;include=etc/fpm.d/*.conf

;;;;;;;;;;;;;;;;;;
; Global Options ;
;;;;;;;;;;;;;;;;;;
[global]
;prefix 应用根目录在/usr/local/php56
;Pid文件 :默认var/run/php-rpm.pid
pid = run/php-fpm.pid

; 错误日志 :默认在var/log/php-fpm.log
error_log = log/php-fpm.log

; 写入错误日志的级别：alert, error, warning, notice(默认), debug
log_level = error

; 表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数
; 如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个默认值都是0
emergency_restart_threshold = 60
;单位可以是s(econd)(默认),m(minutes),h(hours),d(ays)
emergency_restart_interval = 60

; 限制子进程接受主进程复用信号的超时时间(默认是0s)
; 单位: s(econds)(默认单位), m(inutes), h(ours), or d(ays)
process_control_timeout = 0

; 最大进程数
; process.max = 128

; 主进程的优先级，由高到低是-19~20
; process.priority = -19

; 后台执行fpm,默认值为yes，
; 如果为了调试可以改为no。
; 在FPM中，可以使用不同的设置来运行多个进程池。
; 这些设置可以针对每个进程池单独设置。
daemonize = yes
 

;;;;;;;;;;;;;;;;;;;;
; Pool Definitions ; 
;;;;;;;;;;;;;;;;;;;;
[www]

; Per pool prefix
; It only applies on the following directives:
; - 'access.log'
; - 'slowlog'
; - 'listen' (unixsocket)
; - 'chroot'
; - 'chdir'
; - 'php_values'
; - 'php_admin_values'
;prefix = /path/to/pools/$pool

; 启动进程的账户和组
user = nginx
group = nginx

; fpm监听端口，即nginx中php处理的地址，一般默认值即可。
; 常用格式有 'ip:port'、'port'、'/path/to/unix/socket'
listen = 127.0.0.1:9000

; backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。
;listen.backlog = 65535

; unix socket设置选项，如果使用tcp方式访问，这里注释即可。
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
;listen.acl_users =
;listen.acl_groups =
 
;listen.allowed_clients = 127.0.0.1

; process.priority = -19

; 如何控制子进程 对于专用服务器，pm可以设置为static
; Possible Values:
;   static  - a fixed number (pm.max_children) of child processes;
;   dynamic - the number of child processes are set dynamically based on the
;             following directives. With this process management, there will be
;             always at least 1 children.
;             pm.max_children      - 子进程最大数
;             pm.start_servers     - 启动时的进程数
;             pm.min_spare_servers - 保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程
;             pm.max_spare_servers - 保证空闲进程数最大值，如果空闲进程大于此值，此进行清理
;  ondemand - no children are created at startup. Children will be forked when
;             new requests will connect. The following parameter are used:
;             pm.max_children           - the maximum number of children that
;                                         can be alive at the same time.
;             pm.process_idle_timeout   - The number of seconds after which
;                                         an idle process will be killed.
; Note: This value is mandatory.
pm = dynamic

pm.max_children = 5
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3

;pm.process_idle_timeout = 10s; 
; 设置每个子进程重生之前服务的请求数. 
; 对于可能存在内存泄漏的第三方模块来说是非常有用的. 
; 如果设置为 '0' 则一直接受请求, 等同于 PHP_FCGI_MAX_REQUESTS 环境变量. 
; 默认值: 0
pm.max_requests = 500


; FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到
pm.status_path = /status


; ;FPM监控页面的ping网址. 
; 如果没有设置, 则无法访问ping页面. 
; 该页面用于外部检测FPM是否存活并且可以响应请求. 
; 请注意必须以斜线开头 (/)。
ping.path = /ping

; 用于定义ping请求的返回响应. 
; 返回为 HTTP 200 的 text/plain 格式文本. 
; 默认值: pong.
;ping.response = pong

; 登录 log
access.log = log/$pool.access.log

; accessLog格式
; Default: "%R - %u %t \"%m %r\" %s"
access.format = "%R - %u %t \"%m %r%Q%q\" %s %f %{mili}d %{kilo}M %C%%"
 
;慢请求的记录日志
;配合request_slowlog_timeout使用
slowlog = log/$pool.log.slow
 
;当一个请求该设置的超时时间后
;就会将对应的PHP调用堆栈信息完整写入到慢日志中.
;设置为 '0' 表示 'Off'
request_slowlog_timeout = 0
 
; 设置单个请求的超时中止时间. 
; 该选项可能会对php.ini设置中的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用.
; 设置为 '0' 表示 'Off'.
; 当经常出现502错误时可以尝试更改此选项。
request_terminate_timeout = 0
 
; 设置文件打开描述符的rlimit限制. 
; 默认值: 系统定义值默认可打开句柄是1024，
; 可使用 ulimit -n查看，ulimit -n 2048修改。
rlimit_files = 1024
 
; 设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值.
;rlimit_core = 0
 
; 启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用.
;chroot = 
 
; 设置启动目录，启动时会自动Chdir到该目录. 
; 所定义的目录需要是绝对路径. 
; 默认值: 当前目录，或者/目录（chroot时）
;chdir = /var/www
 
; 重定向运行过程中的stdout和stderr到主要的错误日志文件中. 
; 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 
; 默认值: 空.
;catch_workers_output = yes

;clear_env = no

;security.limit_extensions = .php .php3 .php4 .php5
```

**配置文件中文翻译**

[github][https://github.com/HeDefine/PHP.ini-for-Chinese/tree/master]

# 6.配置nginx支持php

```
locate nginx.conf
vim ..文件路径

#将文件第45行修改为如下内容
index index.php index.html index.htm;

#文件的65-72行代码前的注释“＃”去掉,并替换"root"和“fastcgi_param”参数值也就是，使用/usr/local/nginx/html作为网站根目录

location ~ \.php$ {
       root           /usr/lcoal/nginx/html;
       fastcgi_pass   127.0.0.1:9000;
       fastcgi_index  index.php;
       fastcgi_param  SCRIPT_FILENAME /usr/lcoal/nginx/html$fastcgi_script_name;
           include        fastcgi_params;
       }
locate php.ini
vim ..文件路径
#在结尾的“；Local Variables：”之前添加如下内容
cgi.fix_pathinfo = 1

killall -9 php-fpm
killall -9 nginx
nginx
php-fpm
#重启(不会写脚本)
netstat -ntlp #查看端口是否启用并被php-fpm以及nginx使用
```

# 7.查看LNMP网站环境

```
cd /usr/lcoal/nginx/html
touch info.php
vim info.php
#加入下面内容
<?php
  phpinfo();
?>
#写入退出
ECS
:wq

#访问http://ip/info.php
#测试是否解析php，若成功，则配置完成
```

# 8.常见问题

**访问不到网站?**

检查防火墙配置并ping服务器IP、telnet服务器80端口测试连通性

**能访问到html文件却访问不到php文件?**

安装php和配置nginx支持php的步骤是否正确。

//FILE FAILD FIND 是由于nginx配置没有配好

//所有结束后，删除info.php

# 9.参考文献

[手动配置LNMP][https://www.jianshu.com/p/292514f97944]

[阿里云开放实验室][https://edu.aliyun.com/lab/courses/14107e607fe742a88a60d1148d7b405c/detail?purchaseRecordId=4eddcaf58df543d9b96af97a5b2d9342]

# 0.扩展

[Nginx - 维基百科][https://zh.wikipedia.org/wiki/Nginx]

[8分钟带你深入浅出搞懂Nginx][https://zhuanlan.zhihu.com/p/34943332]

[PHPMyadmin 配置文件详解(配置)][https://www.jb51.net/article/21228.html]