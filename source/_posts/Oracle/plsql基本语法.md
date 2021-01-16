---
title: PL/SQL基本语法
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---

# PL/SQL基本语法

## 增删改查

在SQLPLUS中不论是SQL还是PL/SQL，对数据表的改动。
最后都需要commit; 完成实例与数据文件的交互。
<!-- more -->

## 流程控制

### 条件语句:IF语句,CASE语句

#### IF语句的基本形式为 ：

```
IF <布尔表达式> THEN 
	PL/SQL和 SQL语句 
END IF; 
或   
IF <布尔表达式> THEN 
	PL/SQL 和 SQL语句 
ELSE
	PL其它语句 
END IF;
```

注:<布尔表达式>最后返回TRUE or FALSE or NULL
仅当为TRUE时执行THEN后面的语句。

| 判断           | 结果  |
| :------------- | :---- |
| NULL AND TRUE  | NULL  |
| NULL AND FALSE | FALSE |
| NULL AND NULL  | NULL  |
|                |       |
| NULL OR TRUE   | TRUE  |
| NULL OR FALSE  | NULL  |
| NULL OR NULL   | NULL  |

##### NVL函数

`**NVL(E1, E2)**`
如果E1为NULL，则函数返回E2，否则返回E1本身。

`**NVL2(E1, E2, E3)**`
如果E1为NULL，则函数返回E3，若E1不为null，则返回E2。

#### CASE语句的基本形式为 ：

```
CASE selector 
	WHEN expression1 THEN result1 
	WHEN expression2 THEN result2 
	WHEN expressionN THEN resultN 
	[ ELSE resultN+1] 
END;
```

### 循环语句:LOOP语句,EXIT语句

#### 简单循环语句的一般形式：

```
LOOP 
  要执行的语句; 
  EXIT WHEN <条件语句> /*条件满足，退出循环语句*/ 
END LOOP;
```

**注：**EXIT WHEN 子句是必须的，否则循环将无法停止。

#### WHILE 循环语句的一般形式：

```
WHILE <布尔表达式> LOOP 
    要执行的语句; 
END LOOP;
```

**注：**

- while循环语句执行的顺序是先判断<布尔表达式>的真假，如果为真则循环执行，否则退出循环。
- 在WHILE循环语句中仍然可以使用EXIT或EXIT WHEN子句

#### FOR循环语句的一般形式：

```
FOR 循环计数器 IN [ REVERSE ] 下限 .. 上限 LOOP 
  要执行的语句; 
END LOOP;
```

**注：**

- 每循环一次，循环变量自动加1；使用关键字REVERSE，循环变量
  自动减1
- 跟在IN REVERSE 后面的数字必须是从小到大的顺序，但不一定是
  整数，可以是能够转换成整数的变量或表达式
- 可以使用EXIT或者EXIT WHEN子句退出循环

##### RETURN、EXIT、CONTINUE语句

**RETURN：**直接跳出块、存储过程或者函数

**EXIT：**跳出本循环转而执行本循环的上一级循环的下一次循环。

**CONTIUNE：**本次循环后面的代码部分不再执行，转而执行本循环的下一次循环。

### 顺序语句:GOTO语句,NULL语句

#### GOTO语句的一般形式：

```
GOTO   label;  
. . .  . . . 
<<label>> 
/*标号是用<< >>括起来的标识符 */
```

**注：**GOTO语句是无条件跳转到指定的标号去的意思

**示例：**

```
DECLARE 
   V_counter NUMBER := 1; 
 BEGIN 
   LOOP  
     DBMS_OUTPUT.PUT_LINE('V_counter的当前值为:'||V_counter); 
 V_counter := v_counter + 1; 
 IF v_counter > 10 THEN 
     GOTO l_ENDofLOOP; 
 END IF; 
   END LOOP; 
   <<l_ENDofLOOP>> 
     DBMS_OUTPUT.PUT_LINE('V_counter的当前值为:'||V_counter); 
 END ;
```

#### NULL语句：

在PL/SQL 程序中，可以用 null 语句来说明“不用做任何事情”的意思，
相当于一个占位符，可以使某些语句变得有意义，提高程序的可读性

**示例：**

```
DECLARE 
 . . . 
BEGIN 
 . . . 
IF v_num IS NULL THEN 
	GOTO print1; 
END IF;  
 . . . 
<<print1>> 
	NULL;-- 不需要处理任何数据。 
END;
```