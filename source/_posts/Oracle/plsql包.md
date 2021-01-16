---
title: plsql包
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---
# plsql包

## 介绍

- 包是一组相关过程、函数、变量、常量和游标等PL/SQL程序设计元素的组合，它具有面向对象程序设计语言的特点，是对这些PL/SQL 程序设计元素的封装。
- 包类似于C++和JAVA语言中的类
  - 其中变量相当于类中的成员变量，过程和函数相当于类方法。
  - 把相关的模块归类成为包，可使开发人员利用面向对象的方法进行存储过程的开发，从而提高系统性能。
- 与类相同，包中的程序元素也分为公用元素和私用元素两种，这两种元素的区别是他们允许访问的程序范围不同，即它们的作用域不同。
  - 公用元素不仅可以被包中的函数、过程所调用，也可以被包外的PL/SQL程序访问，
  - 私有元素只能被包内的函数和过程序所访问。
- 在PL/SQL程序设计中，使用包不仅可以使程序设计模块化，对外隐藏包内所使用的信息（通过使用私用变量），而且可以提高程序的执行效率。
  - 因为，当程序首次调用包内函数或过程时，ORACLE将整个包调入内存，当再次访问包内元素时，ORACLE直接从内存中读取，而不需要进行磁盘I/O操作，从而使程序执行效率得到提高
<!-- more -->
## 组成

一个包由两个分开的部分组成

- 包定义

  （PACKAGE）

  - 包定义部分声明包内数据类型、变量、常量、游标、子程序和异常错误处理等元素。
  - 这些元素为包的**公有元素**。

- 包主体

  （PACKAGE BODY）

  - 包主体则是包定义部分的具体实现，它定义了包定义部分所声明的游标和子程序。
  - 包主体中可以声明包的**私有元素**。

**注：**

包定义和包主体分开编译，并作为两部分分开的对象存放在数据库字典中。

## 包定义

### 语法

```
CREATE [OR REPLACE] PACKAGE package_name 
{IS | AS} 
  [公有数据类型定义[公有数据类型定义]…] 
  [公有游标声明[公有游标声明]…] 
  [公有变量、常量声明[公有变量、常量声明]…] 
  [公有子程序声明[公有子程序声明]…] 
END [package_name];
```

**注：**

- 在Oracle的存储过程和函数中，其实IS和AS是同义词。
- 还有在自定义类型（TPYE）和包（PACKAGE）时，使用IS和AS也并没有什么区别。
- 但是在创建视图（VIEW）时，只能使用AS而不能使用IS。
- 在声明游标（CURSOR）时，只能使用IS而不能使用AS。

## 包主体

### 语法

```
CREATE [OR REPLACE] PACKAGE BODY package_name 
{IS | AS} 
  [私有数据类型定义[私有数据类型定义]…] 
  [私有变量、常量声明[私有变量、常量声明]…] 
  [私有子程序声明和定义[私有子程序声明和定义]…] 
  [公有游标定义[公有游标定义]…] 
  [公有子程序定义[公有子程序定义]…] 
[BEGIN 
  PL/SQL 语句] 
END [package_name];
```

**注：**

在包主体定义公有程序时，它们必须与包定义中所声明子程序的格式完全一致

## 创建包应用举例

```
/*
例1:创建的包为demo_pack。 
对dept表进行插入、查询和修改操作，并通过demo_pack包中的记录变量
DeptRec 显示所查询到的数据库信息。 
该包中包含一个记录类型变量DeptRec、两个函数和一个过程。： 
*/
CREATE OR REPLACE PACKAGE demo_pack 
IS 
	DeptRec dept%ROWTYPE; 
	FUNCTION add_dept(dept_no NUMBER, dept_name VARCHAR2, location VARCHAR2) 
   		RETURN NUMBER; 
  	FUNCTION remove_dept(dept_no NUMBER) 
   		RETURN NUMBER; 
  	PROCEDURE query_dept(dept_no IN NUMBER); 
END demo_pack;  

--包主体部分：
CREATE OR REPLACE PACKAGE BODY demo_pack 
IS  
	FUNCTION add_dept(dept_no NUMBER, dept_name VARCHAR2, location VARCHAR2) 
  		RETURN NUMBER 
	IS  
   		empno_remaining EXCEPTION; 
     	PRAGMA EXCEPTION_INIT(empno_remaining, -1); 
	/* -1 本身是违反唯一约束条件的错误代码，为已预定义DUP_VAL_ON_INDEX */ 
	BEGIN 
		INSERT INTO dept VALUES(dept_no, dept_name, location); 
		IF SQL%FOUND THEN 
			RETURN 1; 
		END IF; 
		EXCEPTION 
			WHEN empno_remaining THEN  
     			RETURN 0; 
    		WHEN OTHERS THEN 
     			RETURN -1; 
	END add_dept;
	FUNCTION remove_dept(dept_no NUMBER) 
  		RETURN NUMBER 
  	IS  
   	BEGIN 
    	DELETE FROM dept WHERE deptno=dept_no; 
      	IF SQL%FOUND THEN 
     		RETURN 1; 
    	ELSE 
     		RETURN 0; 
      	END IF; 
   		EXCEPTION 
    		WHEN OTHERS THEN 
     			RETURN -1; 
	END remove_dept;
	PROCEDURE query_dept (dept_no IN NUMBER) 
	IS 
 	BEGIN 
   		SELECT * INTO DeptRec FROM dept WHERE deptno=dept_no; 
 		EXCEPTION 
      	WHEN NO_DATA_FOUND THEN   
        	DBMS_OUTPUT.PUT_LINE('数据库中没有编码为'||dept_no||'的部门'); 
      	WHEN TOO_MANY_ROWS THEN 
        	DBMS_OUTPUT.PUT_LINE('程序运行错误!请使用游标'); 
      	WHEN OTHERS THEN 
        	DBMS_OUTPUT.PUT_LINE(SQLCODE||'----'||SQLERRM); 
 	END query_dept; 
END demo_pack;
```

## 调用包应用举例

```
DECLARE 
	Var NUMBER; 
BEGIN 
  	Var := demo_pack.add_dept(90,'Administration', 'Beijing'); 
  	IF var =-1 THEN 
    	DBMS_OUTPUT.PUT_LINE(SQLCODE||'----'||SQLERRM); 
  	ELSIF var =0 THEN 
    	DBMS_OUTPUT.PUT_LINE('该部门记录已经存在！'); 
  	ELSE 
    	DBMS_OUTPUT.PUT_LINE('添加记录成功！'); 
    	Demo_pack.query_dept(90); 
    	DBMS_OUTPUT.PUT_LINE(demo_pack.DeptRec.deptno||'---'|| 
    	demo_pack.DeptRec.dname||'---'||demo_pack.DeptRec.loc); 
    	var := demo_pack.remove_dept(90); 
    	IF var =-1 THEN 
     		DBMS_OUTPUT.PUT_LINE(SQLCODE||'----'||SQLERRM); 
    	ELSIF var=0 THEN 
     		DBMS_OUTPUT.PUT_LINE('该部门记录不存在！'); 
    	ELSE 
     		DBMS_OUTPUT.PUT_LINE('删除记录成功！'); 
    	END IF; 
	END IF; 
END;
```

## 包的开发步骤

- 将每个存储过程调试正确
- 用文本编辑软件将各个存储过程和函数集成在一起
- 按照包的定义要求将集成的文本的前面加上包定义
- 按照包的定义要求将集成的文本的前面加上包定义
- 使用开发工具进行调式

## 子程序重载

PL/SQL 允许对包内子程序和本地子程序进行重载。所谓重载时指两个或多个子程序有相同的名称，但拥有不同的参数变量 参数变量、参数顺序 参数顺序或参数数据类型 参数数据类型

```
--例子
--包定义
CREATE OR REPLACE PACKAGE demo_pack1 
IS 
 	DeptRec dept%ROWTYPE; 
 	FUNCTION query_dept(dept_no IN NUMBER) 
  		RETURN INTEGER; 
 	FUNCTION query_dept(dept_no IN VARCHAR2) 
  		RETURN INTEGER; 
END demo_pack1;
--包主体
CREATE OR REPLACE PACKAGE BODY demo_pack1 
IS  
	FUNCTION query_dept(dept_no IN NUMBER) 
 		RETURN INTEGER 
    IS 
    BEGIN 
 		IF dept_no =10 THEN 
     		SELECT * INTO DeptRec FROM dept WHERE deptno=dept_no; 
     		RETURN 1; 
 		ELSE 
     		RETURN 0; 
 		END IF; 
	END query_dept;
	FUNCTION query_dept(dept_no IN VARCHAR2) 
 		RETURN INTEGER 
    IS 
    BEGIN 
 		IF dept_no =10 THEN 
     		SELECT * INTO DeptRec FROM dept WHERE deptno=dept_no; 
     	RETURN 1; 
 		ELSE 
     		RETURN 0; 
 		END IF; 
    END query_dept; 
END demo_pack1;
```

**注：**

- 如果两个子程序的参数只是名称和方式不同时，不能重载他们。

  - PROCEDURE OverlodeMe（P_parameter in number）

    PROCEDURE OverlodeMe（P_parameter out number）

  - PROCEDURE OverlodeMe（P_parameter number）
    PROCEDURE OverlodeMe（P_para number）

- 不能只根据两个函数的返回类型进行重载

  - FUNCTION Overlodeme RETURN date
    FUNCTION Overlodeme RETURN boolean

- 重载函数的参数必须在类型系列方面有所不同，既不能再同一类 型系列上重载：即字符、数值、日期等等

  - PROCEDURE OverlodeMe（P_parameter in char）
    PROCEDURE OverlodeMe（P_parameter out varchar2(10)）

## 删除包

### 语法

```
DROP PACKAGE [BODY] [user.]package_name 

--例：
DROP PACKAGE emp_package;
```

**包所涉及到的数据字典视图：**

- DBA_SOURCE
- USER_SOURCE
- USER_ERRORS
- DBA_OBJECTS