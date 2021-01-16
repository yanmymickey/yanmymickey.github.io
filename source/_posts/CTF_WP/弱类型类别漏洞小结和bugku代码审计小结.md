---
title: bugku代码审计
type: "categories"
date: 2019-07-16
tags: [CTF,php弱类型]
categories:
- [CTF, PHP]
comments: false
---
# bugku代码审计

## 写在前面

一下只是个人的一点拙见，请大佬轻喷，欢迎大佬补充，很多不恰当的地方。

<!-- more -->

## 0x00 extract变量覆盖

### [题目地址](http://123.206.87.240:9009/1.php)

### 题目源码

```
<?php
$flag='xxx';
extract($_GET);
if(isset($shiyan)){
	$content=trim(file_get_contents($flag));
	if($shiyan==$content){
		echo'flag{xxx}';
	}
	else{
		echo'Oh.no';
	}
}
?>
```

### writeip

1. 构造payload:

   http://123.206.87.240:9009/1.php?shjyan=&flag=

   覆盖原有的flag变量，是空==空成立，输出flag

### flag

flag{bugku-dmsj-p2sm3N}

### extract用法

(PHP 4, PHP 5, PHP 7)

extract — 从数组中将变量导入到当前的符号表

#### 说明

```
extract ( array &$array [, int $flags = EXTR_OVERWRITE [, string $prefix = NULL ]] ) : int
```

 本函数用来将变量从数组中导入到当前的符号表中。

 检查每个键名看是否可以作为一个合法的变量名，同时也检查和符号表中已有的变量名的冲突。

#### 参数

- `array`

   一个关联数组。此函数会将键名当作变量名，值作为变量的值。对每个键／值对都会在当前的符号表中建立变量，并受到 `flags`和`prefix`参数的影响。必须使用关联数组，数字索引的数组将不会产生结果，除非用了`EXTR_PREFIX_ALL`或者`EXTR_PREFIX_INVALID`。

- `flags`

  对待非法／数字和冲突的键名的方法将根据取出标记`flags`参数决定。可以是以下值之一：

  | 参数                    | 说明                                                         |
  | :---------------------- | :----------------------------------------------------------- |
  | `EXTR_OVERWRITE`        | 如果有冲突，覆盖已有的变量。                                 |
  | `EXTR_SKIP`             | 如果有冲突，不覆盖已有的变量。                               |
  | `EXTR_PREFIX_SAME`      | 如果有冲突，在变量名前加上前缀 `prefix`。                    |
  | `EXTR_PREFIX_ALL`       | 给所有变量名加上前缀`prefix`。                               |
  | `EXTR_PREFIX_INVALID`   | 仅在非法／数字的变量名前加上前缀 `prefix`。                  |
  | `EXTR_IF_EXISTS`        | 仅在当前符号表中已有同名变量时，覆盖它们的值。其它的都不处理。举个例子，以下情况非常有用：定义一些有效变量，然后从 `$_REQUEST` 中仅导入这些已定义的变量。 |
  | `EXTR_PREFIX_IF_EXISTS` | 仅在当前符号表中已有同名变量时，建立附加了前缀的变量名，其它的都不处理。 |
  | `EXTR_REFS`             | 将变量作为引用提取。这有力地表明了导入的变量仍然引用了`array` 参数的值。可以单独使用这个标志或者在`flags` 中用 OR 与其它任何标志结合使用。 |
  | 备注                    | 如果没有指定 flags，则被假定为 `EXTR_OVERWRITE`。            |

- `prefix`

  注意 ：

  `prefix`仅在`flags`的值是`EXTR_PREFIX_SAME`，`EXTR_PREFIX_ALL`，`EXTR_PREFIX_INVALID`或 `EXTR_PREFIX_IF_EXISTS`时需要。

  如果附加了前缀后的结果不是合法的变量名，将不会导入到符号表中。前缀和数组键名之间会自动加上一个下划线。

#### 返回值

 返回成功导入到符号表中的变量数目。

## 0x01strcmp比较字符串

### [题目地址](http://123.206.87.240:9009/6.php)

### 题目源码

```
<?php
$flag = "flag{xxxxx}";
if (isset($_GET['a'])) { 
	if (strcmp($_GET['a'], $flag) == 0) //如果 str1 小于 str2 返回 < 0； 如果 str1大于 str2返回 > 0；如果两者相等，返回 0。 
        //比较两个字符串（区分大小写） 
		die('Flag: '.$flag); 
	else 
		print 'No';
}
?>
```

### writeup

1. 用数组绕过，构造payload:

   http://123.206.87.240:9009/6.php?a[]=1

### flag

flag{bugku_dmsj_912k}

## 0x02urldecode二次编码绕过

### [题目地址](http://123.206.87.240:9009/10.php)

### 题目源码

```
<?php
if(eregi("hackerDJ",$_GET[id])) {
	echo("not allowed!");
	exit();
}
$_GET[id] = urldecode($_GET[id]);
if($_GET[id] == "hackerDJ")
{
	echo "Access granted!";
	echo "flag";
}
?>
```

### writeup

1. 浏览器会对get参数进行一次URL编码，源码中还有一次，两次编码绕过验证。
2. 对hackerDJ两次URL编码构造payload:

[http://123.206.87.240:9009/10.php?id=%25%36%38%25%36%31%25%36%33%25%36%42%25%36%35%25%37%32%25%34%34%25%34%41](http://123.206.87.240:9009/10.php?id=%68%61%63%6B%65%72%44%4A)

### flag

flag{bugku__daimasj-1t2}

## 0x03md5()函数

### [题目地址](http://123.206.87.240:9009/18.php)

### 题目源码

```
<?php
error_reporting(0);
$flag = 'flag{test}';
if (isset($_GET['username']) and isset($_GET['password'])) {
	if ($_GET['username'] == $_GET['password'])
		print 'Your password can not be your username.';
	else if (md5($_GET['username']) === md5($_GET['password']))
		die('Flag: '.$flag);
	else
		print 'Invalid password';
}
?>
```

### writeup

1. `(md5($_GET['username']) === md5($_GET['password']))`判断是全等于，所以不可以用碰撞，可以用数组绕过。

2. 构造payload：

   http://123.206.87.240:9009/18.php?username[]=1&password[]=2

### flag

flag{bugk1u-ad8-3dsa-2}

## 0x04md5加密相等绕过

### [题目地址](http://123.206.87.240:9009/13.php)

### 题目源码

```
<?php
$md51 = md5('QNKCDZO');
$a = @$_GET['a'];
$md52 = @md5($a);
if(isset($a)){
	if ($a != 'QNKCDZO' && $md51 == $md52) {
		echo "flag{*}";
	} else {
		echo "false!!!";
	}
}
else{
	echo "please input a";
}
?>
```

### writeup

1. `$md51 == $md52`不是全等于，用md5碰撞

2. 构造payload：

   http://123.206.87.240:9009/13.php?a=240610708

### flag

flag{bugku-dmsj-am9ls}

### md5碰撞样例

```
QNKCDZO
0e830400451993494058024219903391

240610708
0e462097431906509019562988736854

s878926199a
0e545993274517709034328855841020
  
s155964671a
0e342768416822451524974117254469
  
s214587387a
0e848240448830537924465865611904
  
s214587387a
0e848240448830537924465865611904
  
s878926199a
0e545993274517709034328855841020
  
s1091221200a
0e940624217856561557816327384675
  
s1885207154a
0e509367213418206700842008763514
```

## 0x05数组返回NULL绕过

### [题目地址](http://123.206.87.240:9009/19.php)

### 题目源码

```
<?php
$flag = "flag";
if (isset ($_GET['password'])) {
	if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)
		echo 'You password must be alphanumeric';
	else if (strpos ($_GET['password'], '--') !== FALSE)
		die('Flag: ' . $flag);
	else
		echo 'Invalid password';
}
?>
```

### writeup

1. 题目提示很明显

2. 数组构造payload：

   http://123.206.87.240:9009/19.php?password[]=1

### flag

flag{ctf-bugku-ad-2131212}

## 0x06弱类型整数大小比较绕过

### [题目地址](http://123.206.87.240:9009/22.php)

### 题目源码

```
<?php
$temp = $_GET['password'];
is_numeric($temp)?die("no numeric"):NULL;
if($temp>1336){
	echo $flag;
}
?>
```

### writeup

1. 利用弱类型比较漏洞

2. is_numeric()判断是否为number类型,int float

3. 构造payload：

   http://123.206.87.240:9009/22.php?password=1337a

### flag

flag{bugku_null_numeric}

## 0x07

### [题目地址](http://123.206.87.240:9009/20.php)

### 题目源码

```
<?php
error_reporting(0);
function noother_says_correct($temp){
	$flag = 'flag{test}';
	$one = ord('1'); //ord — 返回字符的 ASCII 码值
	$nine = ord('9'); //ord — 返回字符的 ASCII 码值
	$number = '3735929054';
	// Check all the input characters!
	for ($i = 0; $i < strlen($number); $i++){
		// Disallow all the digits!
		$digit = ord($temp{$i});
		if ( ($digit >= $one) && ($digit <= $nine) ){// Aha, digit not allowed!
			return "flase";
		}
	}
	if($number == $temp
		return $flag;
}
$temp = $_GET['password'];
echo noother_says_correct($temp);
?>
```

### writeup

1. 利用弱类型比较漏洞,`==`会将数字自动转换为16进制

2. 将3735929054转换为16进制加上控制字符0x

3. 利用转换结果0xdeadc0de构造payload：

   http://123.206.87.240:9009/20.php?password=0xdeadc0de

### flag

flag{Bugku-admin-ctfdaimash}

## 弱类型总结

```
<?php
2 var_dump("admin"==0);  //true
3 var_dump("1admin"==1); //true
4 var_dump("admin1"==1) //false
5 var_dump("admin1"==0) //true
6 var_dump("0e123456"=="0e4456789"); //true 
7 ?>
```

#### ==判断松散性

1. 当一个字符串被当作一个数值来取值，其结果和类型如下:如果该字符串没有包含’.’,’e’,’E’并且其数值值在整形的范围之内，该字符串被当作int来取值。其他所有情况下都被作为float来取值，该字符串的开始部分决定了它的值，如果该字符串以合法的数值开始，则使用该数值，否则其值为0。
2. 在进行比较运算时，如果遇到了0e这类字符串，PHP会将它解析为科学计数法。
3. 在进行比较运算时，如果遇到了0x这类字符串，PHP会将它解析为十六进制。

#### 函数松散性

1. `switch()`

   如果`switch`是**数字类型**的`case`的判断时，switch会将其中的参数转换为`int`类型。

2. `in_array()`

   `in_array(search,array,type)`: 如果给定的值 search 存在于数组 array 中则返回 true（**类似于==**）。如果第三个参数设置为 true，函数只有在元素存在于数组中且数据类型与给定值相同时才返回 true（**类似于===**）。如果没有在数组中找到参数，函数返回 false。

3. `is_numeric`()

   `is_numeric`在做判断时候，如果攻击者把payload改成十六进制0x…，is_numeric会先对十六进制做类型判断，十六进制被判断为数字型为真，就进入了条件语句，如果再把这个代入进入sql语句进入mysql数据库，mysql数据库会对hex进行解析成字符串存入到数据库中，如果这个字段再被取出来二次利用，就可能造成二次注入漏洞。

4. `strcmp`()

   `strcmp(string1，string2)`:比较括号内的两个字符串string1和string2，当他们两个相等时，返回0；string1的大于string2时，返回>0;小于时返回<0。在5.3及以后的php版本中，当strcmp()括号内是一个数组与字符串比较时，也会返回0。

5. `md5`()

   `md5 ( string $str [, bool $raw_output = false ] )`

   `md5()`需要是一个string类型的参数。但是当你传递一个array时，`md5()`不会报错，只是会无法正确地求出array的md5值，返回null，这样就会导致任意2个array的md5值都会相等。

## 0x08sha()函数比较绕过

### [题目地址](http://123.206.87.240:9009/7.php)

### 题目源码

```
<?php
$flag = "flag";
if (isset($_GET['name']) and isset($_GET['password'])){
	var_dump($_GET['name']);
	echo "";
	var_dump($_GET['password']);
	var_dump(sha1($_GET['name']));
	var_dump(sha1($_GET['password']));
	if ($_GET['name'] == $_GET['password'])
		echo 'Your password can not be your name!';
	else if (sha1($_GET['name']) === sha1($_GET['password']))
		die('Flag: '.$flag);
	else
		echo 'Invalid password.';
}
else
	echo 'Login first';
?>
```

### writeup

1. 数组绕过

2. 构造payload：

   http://123.206.87.240:9009/7.php?name[]=1&password[]=2

### flag

flag{bugku–daimasj-a2}

## 0x09ereg正则%00截断

### [题目地址](http://123.206.87.240:9009/5.php)

### 题目源码

```
<?php
$flag = "xxx";
if (isset ($_GET['password'])){
	if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE){
		echo 'You password must be alphanumeric';
	}
	else if (strlen($_GET['password']) < 8 && $_GET['password'] > 9999999){
		if (strpos ($_GET['password'], '-') !== FALSE){//strpos — 查找字符串首次出现的位置
			die('Flag: ' . $flag);
		}
		else{
			echo('have not been found');
		}
	}
	else{
		echo 'Invalid password';
	}
}
?>
```

### writeup

1. `if (ereg ("^[a-zA-Z0-9]+$", $_GET['password']) === FALSE)`
   password匹配必须 a-z A-Z 0-9 之中

   这个可以用%00截断

2. `if (strlen($_GET['password']) < 8 && $_GET['password'] > 9999999)`

   password小于8位数 且 大于9999999

3. `if (strpos ($_GET['password'], '-') !== FALSE)`

   数组绕过同时也可以绕过这个

4. 构造payload：

   [http://123.206.87.240:9009/5.php?password[\]=0%00](http://123.206.87.240:9009/5.php?password[]=0 )

### flag

flag{bugku-dm-sj-a12JH8}

### bug？？？

不知道为什么，构造payload:

http://123.206.87.240:9009/5.php?password[]=

也可以，同理

[http://123.206.87.240:9009/5.php?password[\]=0%00](http://123.206.87.240:9009/5.php?password[]=0 )

也可以。。。。。bug？？？就(⊙_⊙)？

### 0x00截断与%00

0x00是十六进制表示方法，是ascii码为0的字符，在有些函数处理时，会把这个字符当做结束符。这个可以用在对文件类型名的绕过上。当然某些比较函数会当成结束符，从而达到绕过验证的效果。

#### 感觉讲解还可以的文章

[地址1](https://blog.csdn.net/zpy1998zpy/article/details/80545408)

[地址2](http://blog.csdn.net/hitwangpeng/article/details/47042971)

[地址3](http://www.2cto.com/article/201502/377462.html)

[地址4](http://www.2cto.com/article/201110/108975.htm)

建议自己实验一遍。

## 0x10strpos数组绕过

### [题目地址](http://123.206.87.240:9009/15.php)

### 题目源码

```
<?php
$flag = "flag";
if (isset ($_GET['ctf'])) {
	if (@ereg ("^[1-9]+$", $_GET['ctf']) === FALSE)
		echo '必须输入数字才行';
	else if (strpos ($_GET['ctf'], '#biubiubiu') !== FALSE)
		die('Flag: '.$flag);
	else
		echo '骚年，继续努力吧啊~';
}
?>
```

### writeup

1. `if (@ereg ("^[1-9]+$", $_GET['ctf'])`

   必须输入数字

2. `if (strpos ($_GET['ctf'], '#biubiubiu') !== FALSE)`

   传过来的字符串要匹配一次`#biubiubiu`

3. 题目提示很明显，所以利用数组绕过

4. 构造payload:

   http://123.206.87.240:9009/15.php?ctf[]=#biubiubiu

### flag

flag{Bugku-D-M-S-J572}

### bug？？？

构造payload:

http://123.206.87.240:9009/15.php?ctf[]=

也可以得到flag，bug？？？(⊙_⊙)？

## 备注

#### 题目挂了

变量覆盖

简单的waf

#### 题目有严重bug

数字验证正则绕过

直接post一个password=就可以拿到flag

不过有个可能值得看看的文章

[地址](https://foxgrin.github.io/posts/25617/)

## 总结

代码审计是一项基本技能，就是看代码，找相关函数的验证漏洞，绕过验证，sql注入可以类比。

bugku的这几道代码审计题目质量其实不够高，但是他单独拎出来，说明这种能力很重要。看看，学习一些只是也是很好的，找到了flag，不要满足，继续审计一下，php官方文档是个好东西。