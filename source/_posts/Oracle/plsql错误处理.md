---
title: plsql异常处理
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---
# plsql异常处理

## 异常处理的概念

异常情况处理(EXCEPTION)是用来处理正常执行过程中未预料的事件,程序块的异常处理预定义的错误和自定义错误,由于PL/SQL程序块一旦产生异常而没有指出如何处理时,程序就会自动终止整个程序运行。
<!-- more -->

有三种类型的异常错误 :

| 类型                        | 描述                                                         |
| :-------------------------- | :----------------------------------------------------------- |
| 预定义 ( Predefined )错误   | ORACLE预定义的异常情况大约有22个。对这种异常情况的处理，无需在程序中定义，由ORACLE自动将其引发 |
| 非预定义 ( Predefined )错误 | 即其他标准的ORACLE错误。对这种异常情况的处理，需要用户在程序中定义，然后由ORACLE自动将其引发 |
| 用户定义(User_define) 错误  | 程序执行过程中，出现编程人员认为的非正常情况。对这种异常情况的处理，需要用户在程序中定义，然后显式地在程序中将其引发。 |

## 异常处理

### 结构:

```
EXCEPTION 
   WHEN first_exception THEN  <code to handle first exception > 
   WHEN second_exception THEN  <code to handle second exception > 
   WHEN OTHERS THEN  <code to handle others exception > 
END;
```

### 作用范围

- PL/SQL的异常捕获只针对执行部分，在声明部分产生的异常是无法捕获的
- 异常处理部分本身导致的异常同样也是无法捕获的，解决方法和声明部分一样，需要在外层进行捕获。

### 异常错误

每个异常错误都包含异常错误号（错误代码）和错误描述信息

| 错误代码（错误代码） | 错误描述信息                         |
| :------------------- | :----------------------------------- |
| ORA-00001            | 违反唯一约束条件                     |
| ORA-00017            | 请求会话以设置跟踪事件               |
| ORA-00018            | 超出最大会话数                       |
| ORA-00019            | 超出最大会话许可数                   |
| ORA-00020            | 超出最大进程数 ()                    |
| ORA-00021            | 会话附属于其它某些进程；无法转换会话 |
| ORA-00022            | 无效的会话 ID；访问被拒绝            |
| ORA-00023            | 会话引用进程私用内存；无法分离会话   |
| ORA-00024            | 单一进程模式下不允许从多个进程注册   |

### 预定义的异常处理

| 错误号    | 异常名称                | 说明                                                         |
| :-------- | :---------------------- | :----------------------------------------------------------- |
| ORA-0001  | DUP_VAL_ON_INDEX        | 试图破坏一个唯一性限制                                       |
| ORA-0051  | TIMEOUT_ON_RESOURCE     | 在等待资源时发生超时                                         |
| ORA-0061  | TRANSACTION_BACKED_OUT  | 由于发生死锁事务被撤消                                       |
| ORA-1001  | INVALID_CURSOR          | 试图使用一个未打开的游标                                     |
| ORA-1012  | NOT_LOGGED_ON           | 没有连接到ORACLE                                             |
| ORA-1017  | LOGIN_DENIED            | 无效的用户名/口令                                            |
| ORA-1403  | NO_DATA_FOUND           | SELECT INTO没有找到数据                                      |
| ORA-1422  | TOO_MANY_ROWS           | SELECT INTO 返回多行                                         |
| ORA-1476  | ZERO_DIVIDE             | 试图被零除                                                   |
| ORA-1722  | INVALID_NUMBER          | 转换一个数字失败                                             |
| ORA-6500  | STORAGE_ERROR           | 内存不够或内存被破坏引发的内部错误                           |
| ORA-6501  | PROGRAM_ERROR           | 内部错误,需重新安装数据字典视图和pl/sql包                    |
| ORA-6502  | VALUE_ERROR             | 赋值操作，变量长度不足，触发该异常                           |
| ORA-6504  | ROWTYPE_MISMATCH        | 宿主游标变量与 PL/SQL变量有不兼容行类型                      |
| ORA-6511  | CURSOR_ALREADY_OPEN     | 试图打开一个已打开的游标                                     |
| ORA-6530  | ACCESS_INTO_NULL        | 试图为null 对象的属性赋值                                    |
| ORA-6531  | COLLECTION_IS_NULL      | 试图给没有初始化的嵌套表变量或者varry变量赋值                |
| ORA-6532  | SUBSCRIPT_OUTSIDE_LIMIT | 对嵌套或varray索引使用了负数                                 |
| ORA-6533  | SUBSCRIPT_BEYOND_COUNT  | 对嵌套或varray索引的引用大于集合中元素的个数                 |
| ORA-6592  | CASE_NOT_FOUND          | 当Case语句的When子句没有包含必需分支或者Else子句时，会触发该异常。 |
| ORA-30625 | SELF_IS_NULL            | 试图在null实例上调用成员方法                                 |
| ORA-1410  | SYS_INVALID_ROWID       | 试图将无效的字符串转化成ROWID                                |

对于预定义异常情况的处理，只需在PL/SQL块的异常处理部分，直接引用相应的异常情况名，并对其完成相应的异常错误处理即可。

#### 示例

```
--例： 更新指定员工工资，如工资小于1500，则加100    
 DECLARE 
   v_empno emp.empno%TYPE :=7900; 
   v_sal    emp.sal%TYPE; 
BEGIN 
   SELECT sal INTO v_sal FROM emp WHERE empno=v_empno; 
   IF v_sal<=1500 THEN  
        UPDATE emp SET sal=sal+100 WHERE empno=v_empno;  
        DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'员工工资已更新!');      
   ELSE 
     DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'员工工资已经超过规定值!'); 
   END IF;
   EXCEPTION 
   WHEN NO_DATA_FOUND THEN   
     DBMS_OUTPUT.PUT_LINE('数据库中没有编码为'||v_empno||'的员工'); 
   WHEN TOO_MANY_ROWS THEN 
     DBMS_OUTPUT.PUT_LINE('程序运行错误!请使用游标'); 
   WHEN OTHERS THEN 
     DBMS_OUTPUT.PUT_LINE(SQLCODE||’---‘||SQLERRM); 
END;
```

### 非预定义的异常处理

预定义的异常错误大约有24个，而错误代码成千上万如ORA-00020 。 对于这类异常情况的处理，首先必须对非定义的ORACLE错误进行定义

**步骤如下：**

- 在PL/SQL 块的声明部分定义异常情况：

  <异常情况> EXCEPTION;

- 将其定义好的异常情况，与标准的ORACLE错误联系起来，使用EXCEPTION_INIT语句

  PRAGMA EXCEPTION_INIT(<异常情况>, <错误代码>)；

- 在PL/SQL 块的异常情况处理部分对异常情况做出相应的处理。

#### 示例

```
--例：删除指定部门的记录信息，以确保该部门没有员工。   
 DECLARE 
   v_deptno dept.deptno%TYPE :=&deptno; 
   deptno_remaining EXCEPTION; 
   PRAGMA EXCEPTION_INIT(deptno_remaining, -2292); 
   /* -2292 是违反references完整性约束的错误代码 */ 
BEGIN 
   DELETE FROM dept WHERE deptno=v_deptno; 
EXCEPTION 
   WHEN deptno_remaining THEN  
      DBMS_OUTPUT.PUT_LINE('违反数据完整性约束!'); 
   WHEN OTHERS THEN 
      DBMS_OUTPUT.PUT_LINE(SQLCODE||’---‘||SQLERRM); 
END;
```

### 用户自定义的异常处理

当与一个异常错误相关的错误出现时，就会隐含触发该异常错误。用户定义的异常错误是通过显式使用 RAISE 语句来触发。当引发一个异常错误时，控制就转向到 EXCEPTION块异常错误部分，执行错误处理代码。

**步骤如下 ：**

- 在PL/SQL 块的声明部分定义异常情况 ：

  <异常情况> EXCEPTION;

- RAISE <异常情况>

- 在PL/SQL 块的异常情况处理部分对异常情况做出相应的处理。

#### 示例

```
--例
DECLARE 
   v_empno emp.empno%TYPE :=&empno; 
   no_result  EXCEPTION; 
BEGIN 
   UPDATE emp SET sal=sal+100 WHERE empno=v_empno; 
   IF SQL%NOTFOUND THEN 
      RAISE no_result; 
   END IF; 
EXCEPTION 
   WHEN no_result THEN  
      DBMS_OUTPUT.PUT_LINE('你的数据更新语句失败了!'); 
   WHEN OTHERS THEN 
      DBMS_OUTPUT.PUT_LINE(SQLCODE||’---‘||SQLERRM); 
END;
```

### RAISE_APPLICATION_ERROR函数

调用DBMS_STANDARD(ORACLE提供的包)包所定义的RAISE_APPLICATION_ERROR过程，可以重新定义异常错误消息，它为应用程序提供了一种与ORACLE交互的方法。 错误号的范围是-20,000到-20,999。错误信息是文本字符串，最多为2048字节。

#### 示例

```
--例：
declare 
      v_deptid  departments.department_id%type := &no; 
      v_dname departments.department_name%type; 
 begin 
      select department_name into v_dname from departments 
      where department_id = v_deptid; 
      dbms_output.put_line(v_dname); 
 exception 
      when others then 
         raise_application_error(-20001 , 'department'||v_deptid||' does not exists'); 
 end;
```

## 异常错误传播

由于异常错误可以在声明部分和执行部分以及异常错误部分出现，因而在不同部分引发的异常错误也不一样。

**可执行部分产生的异常：**

当一个异常错误在执行部分引发时，有下列情况：

1. 如果当前块对该异常错误设置了处理，则执行它并成功完成该块的执行，然后控制转给包含块。
2. 如果没有对当前块异常错误设置定义处理器，则通过在包含块中引发它来传播异常错误。然后对该包含块执行步骤1)。

### 声明部分产生的异常：

如果在声明部分引起 异常 情况，即在声明部分出现错误，那么该错误就能影响到其它的块

#### 示例

```
--例1：
DECLARE 
	Abc number(3):=’abc’; 
	--其它语句 
BEGIN 
	--其它语句 
	EXCEPTION 
	WHEN OTHERS THEN  
	--其它语句 
END; 
/*
由于Abc number(3)=’abc’; 出错，尽管在EXCEPTION中说明了WHEN OTHERS THEN语句，但WHEN OTHERS THEN也不会被执行。  
*/
--例2：
BEGIN 
 DECLARE 
     Abc number(3):=’abc’; 
     --其它语句 
    BEGIN 
     --其它语句 
    EXCEPTION 
     WHEN OTHERS THEN  
     --其它语句 
  	END; 
 EXCEPTION 
 WHEN OTHERS THEN  
 --其它语句 
END; 
/*
在该错误语句块的外部有一个异常错误，则该错误能被抓住
*/
```

## 异常处理的SQLCode和SQL Errm

**注意： **

- 某给定异常（如自定义）只能在异常处理部分的处理一次。如果有多个异常处理器（多次when），则会抛出PLS-00483异常。
- SQLCODE返回当前的错误代码，SQLERRM返回当前的错误信息。对于用户自定义异常SQLCODE返回值为‘1’，SQLERRM返回值为‘User-defined Exception’
- Oracle错误信息的最大长度是512字节
- SQLCODE和SQLERRM的值先赋给本地变量，不能直接用于SQL语句