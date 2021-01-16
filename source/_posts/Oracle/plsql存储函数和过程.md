---
title: plsql存储函数和过程
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---

# plsql存储函数和过程

## 定义

- ORACLE 提供可以把PL/SQL 程序存储在数据库中，并可以在任何地方来运行它（写好的存储函数和过程就同Oracle原有的函数一样）。这样就叫存储过程或函数。
- 过程和函数统称为PL/SQL子程序，他们是被命名的PL/SQL块，均存储在数据库中，并通过输入、输出参数或输入/输出参数与其调用者交换信息。
- 过程和函数的唯一区别是函数总向调用者返回数据，而过程则不返回数据。
<!-- more -->
## 存储函数

### 创建存储函数

#### 语法：

```
CREATE [OR REPLACE] FUNCTION function_name 
[ (argment [ { IN | OUT | IN OUT } ] Type ， 
   argment [ { IN | OUT | IN OUT } ] Type ] 
RETURN return_type  
{ IS | AS } 
<类型.变量的说明>   
BEGIN 
   FUNCTION_body 
EXCEPTION 
   其它语句 
END;  
--注： OR REPLACE有无的区别。
```

#### 示例

```
--例：获取某部门员工数和工资总和 
CREATE OR REPLACE FUNCTION get_salary(
    Dept_no NUMBER, 
    Emp_count OUT NUMBER) 
RETURN NUMBER  
IS 
    V_sum NUMBER;
BEGIN 
 	SELECT SUM(sal), count(*) INTO V_sum, emp_count FROM emp WHERE deptno=dept_no;
  	--emp表中dept_no是唯一Key 
 	RETURN v_sum; 
EXCEPTION 
     WHEN NO_DATA_FOUND THEN  
        DBMS_OUTPUT.PUT_LINE('你需要的数据不存在!'); 
     WHEN OTHERS THEN  
        DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM); 
END get_salary;
```

### 调用函数方法

- 函数声明时所定义的参数称为形式参数，应用程序调用时为函数传递的参数称为实际参数。
- 应用程序在调用函数时，可以使用以下三种方法向函数传递参数：

#### 位置表示法

##### 格式

```
argument_value1[,argument_value2 …]
```

**注：**类型序列必须与函数声明是的类型序列相同

##### 示例

```
--例：计算某部门的员工数和工资总和 
DECLARE 
	V_num NUMBER; 
	V_sum NUMBER; 
BEGIN 
	V_sum :=get_salary(30, v_num); 
	DBMS_OUTPUT.PUT_LINE('30号部门工资总和：'||v_sum||'，人数：'||v_num); 
END;
```

#### 名称表示法

##### 格式

```
argument => parameter [,…]
```

**注：**

- argument 为形式参数，它必须与函数定义时所声明的形式参数名称以及类型相同。Parameter 为实际参数。
- 在这种格式中，形式参数与实际参数成对出现，相互间关系唯一确定，所以参数的顺序可以任意排列。

##### 示例

```
例：计算某部门的员工数和工资总和 
DECLARE 
	V_num NUMBER; 
	V_sum NUMBER; 
BEGIN 
	V_sum :=get_salary(emp_count => v_num, dept_no => 30); 
	DBMS_OUTPUT.PUT_LINE('30号部门工资总和：'||v_sum||'，人数：'||v_num); 
END;
```

#### 混合表示法

- 即在调用一个函数时，同时使用位置表示法和名称表示法为函数传递参数。
- 采用这种参数传递方法时，使用**位置表示法**所传递的参数必须放在**名称表示法**所传递的参数**前面**。
- 也就是说，无论函数具有多少个参数，只要其中有一个参数使用名称表示法，其后所有的参数都必须使用名称表示法。

##### 示例

```
--例： 
DECLARE  
    Var VARCHAR2(32); 
BEGIN 
	Var := demo_fun('user1', 30, sex => '男'); 
	DBMS_OUTPUT.PUT_LINE(var); 
	Var := demo_fun('user2', age => 40, sex => '男'); 
	DBMS_OUTPUT.PUT_LINE(var); 
	Var := demo_fun('user3', sex => '女', age => 20); 
	DBMS_OUTPUT.PUT_LINE(var); 
END;
```

#### 参数默认值

在CREATE OR REPLACE FUNCTION 语句中声明函数参数时可以使用DEFAULT关键字为输入参数指定默认值。

```
--定义示例： 
CREATE OR REPLACE FUNCTION demo_fun( 
	Name VARCHAR2,Age INTEGER, 
	Sex VARCHAR2 DEFAULT '男') 
RETURN VARCHAR2  
IS 
   V_var VARCHAR2(32); 
BEGIN 
	V_var := name||'：'||TO_CHAR(age)||'岁，'||sex; 
   RETURN v_var; 
END;
```

**注：**

- 具有默认值的函数创建后，在函数调用时，如果没有为具有默认值的参数提供实际参数值，函数将使用该参数的默认值。
- 但当调用者为默认参数提供实际参数时，函数将使用实际参数值。
- 在创建函数时，只能为输入参数设置默认值，而不能为输入/输出参数设置默认值。

```
--调用示例： 
DECLARE  
    Var VARCHAR(32); 
BEGIN 
	Var := demo_fun('user1', 30); 
	DBMS_OUTPUT.PUT_LINE(var); 
	Var := demo_fun('user2', age => 40); 
	DBMS_OUTPUT.PUT_LINE(var); 
	Var := demo_fun('user3', sex => '女', age => 20); 
	DBMS_OUTPUT.PUT_LINE(var); 
END;
```

## 存储过程

### 创建存储过程

#### 语法

```
CREATE [OR REPLACE] PROCEDURE Procedure_name 
[ (argment [ { IN | OUT | IN OUT } ] Type, 
   argment [ { IN | OUT | IN OUT } ] Type ] 
{ IS | AS } 
 <类型.变量的说明>   
BEGIN 
 <执行部分> 
EXCEPTION 
 <可选的异常错误处理程序> 
END;
```

#### 示例

```
--例 ：删除指定员工记录 
CREATE OR REPLACE PROCEDURE DelEmp 
  (v_empno IN emp.empno%TYPE)  
AS 
	No_result EXCEPTION; 
BEGIN 
	DELETE FROM emp WHERE empno=v_empno; 
	IF SQL%NOTFOUND THEN 
    	RAISE no_result; 
    END IF; 
       DBMS_OUTPUT.PUT_LINE('编码为'||v_empno||'的员工已被除名!');
	EXCEPTION 
		WHEN no_result THEN  
			DBMS_OUTPUT.PUT_LINE('你需要的数据不存在!'); 
		WHEN OTHERS THEN 
      		DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM); 
END DelEmp;
```

### 调用存储过程

存储过程建立完成后，只要通过授权，用户就可以在SQLPLUS 、ORACLE开发工具或第三方开发工具中来调用运行。ORACLE 使用EXECUTE 语句来实现对存储过程的调用

#### 语法

```
EXEC[UTE]  Procedure_name( parameter1, parameter2…)
```

#### 示例

```
--sqlplus调用示例
SQL> EXECUTE DelEmp(10)； 
SQL> EXECUTE DelEmp(:a)；   
 
SQL> variable a varchar2(20) 
SQL> execute :a:=fun_stu('BA');
--例：计算指定部门的工资总和，并统计其中的职工数量。 
--创建存储过程
CREATE OR REPLACE PROCEDURE proc_demo 
	(Dept_no NUMBER DEFAULT 10 , Sal_sum OUT NUMBER, 
	Emp_count OUT NUMBER) 
IS 
BEGIN 
	SELECT SUM(sal), COUNT(*) INTO sal_sum, emp_count FROM emp WHERE deptno=dept_no; 
	EXCEPTION 
     	WHEN NO_DATA_FOUND THEN  
        	DBMS_OUTPUT.PUT_LINE('你需要的数据不存在!'); 
     	WHEN OTHERS THEN  
        	DBMS_OUTPUT.PUT_LINE(SQLCODE||'---'||SQLERRM); 
 END proc_demo;
--调用存储过程: 
DECLARE 
	V_num NUMBER; 
  	V_sum NUMBER(8, 2); 
BEGIN 
 	Proc_demo(30, v_sum, v_num); 
  	DBMS_OUTPUT.PUT_LINE('30号部门工资总和：'||v_sum||'，人数：'||v_num); 
  	Proc_demo(sal_sum => v_sum, emp_count => v_num); 
  	DBMS_OUTPUT.PUT_LINE('10号部门工资总和：'||v_sum||'，人数：'||v_num); 
END;
```

## 查询、删除过程和函数

- 在SQL*PLUS 中，可以用DESCRIBE 命令查看过程的名字及其参数

  `DESCRIBE Procedure_name;`
  (desc table; 用于列出指定表或视图中的所有列)

- 可以使用DROP语句删除函数和过程：

  ```
  --删除函数
  DROP FUNCTION function_name;
  --删除过程
  DROP PROCEDURE proceduer_name;
  ```

## 授权执行权给相关的用户或角色

- 如果调式正确的存储过程没有进行授权，那就只有建立者本人才可以运行。所以作为应用系统的一部分的存储过程也必须进行授权才能达到要求。
- 可以用GRANT命令来进行存储过程的运行授权

#### 语法

```
--例
GRANT EXECUTE ON Proc_demo TO user| role | PUBLIC  
	[WITH GRANT OPTION]
```

## 与过程相关的内置数据字典

| 名称            | 描述                         |
| :-------------- | :--------------------------- |
| USER_PROCEDURES | 查询用户所有的子程序信息     |
| USER_SOURCE     | 查看用户所有对象的源代码     |
| USER_OBJECTS    | 查看用户创建的过程对象       |
| USER_ERRORS     | 查看用户所有的子程序错误信息 |

```
SELECT object_name,authid,object_type FROM user_procedures;
--AUTHID DEFINER （定义者权限）：指编译存储对象的所有者。也是默认权限模式 
--AUTHID CURRENT_USER（调用者权限）：指拥有当前会话权限的模式 

SELECT * FROM user_source WHERE name='MLDN_PROC' ;

SELECT object_name,created,timestamp,status FROM user_objects 
WHERE object_type='PROCEDURE' OR object_type='FUNCTION'; 
--status：该字段有两个取值：VALID（有效），INVALID（无效）
```