---
title: plsql游标
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---

# plsql游标

## 游标

| SQL语句              |              |
| :------------------- | :----------- |
| 非查询语句           | 隐式的       |
| 结果是单行的查询语句 | 隐式或显示的 |
| 结果是多行的查询语句 | 显示的       |
<!-- more -->
### 显示游标

#### 定义游标：

就是定义一个游标名，以及与其相对应的SELECT 语句

```
CURSOR cursor_name IS  select_statement;
```

**注：**

- 游标声明部分是唯一可以出现在模块声明部分的步骤，其他三个
  步骤都在执行或异常处理部分中
- 游标名是标识符，所以也有作用域，并且必须在使用前进行说明
- 任何SELECT语句都是合法的，但是SELECT …INTO语句是非法的
- 在声明部分的末尾声明游标

#### 打开游标：

就是执行游标所对应的SELECT语句，将其查询结果放入工作区，并且指针指向工作区的首部，标识游标结果集合。

```
OPEN cursor_name
```

**注：**PL/SQL 程序不能用OPEN 语句重复打开同一个游标

#### 提取游标：

就是检索结果集合中的数据行，放入指定的输出变量中。

```
FETCH cursor_name INTO {variable_list | record_variable };
```

#### 关闭游标：

当提取和处理完游标结果集合数据后，应及时关闭游标，以释放该游标所占用的系统资源，并使该游标的工作区变成无效，不能再使用FETCH 语句取其中数据。关闭后的游标可以使用OPEN 语句重新打开。

```
CLOSE cursor_name;
```

#### 游标属性

| 属性      | 描述                                              |
| :-------- | :------------------------------------------------ |
| %FOUND    | 布尔型属性，当最近一次读记录时成功返回,则值为TRUE |
| %NOTFOUND | 布尔型属性，与%FOUND相反                          |
| %ISOPEN   | 布尔型属性，当游标已打开时返回 TRUE               |
| %ROWCOUNT | 数字型属性，返回已从游标中读取的记录数            |

### 隐式游标

**说明：**

- 显式游标主要是用于对查询语句的处理，尤其是在查询结果为多条记录的情况下；而对于非查询语句，如修改、删除操作，则由ORACLE 系统自动地为这些操作设置游标并创建其工作区，这些由系统隐含创建的游标称为隐式游标
- 隐式游标的名字为SQL，这是由ORACLE 系统定义的。对于隐式游标的操作，如定义、打开、取值及关闭操作，都由ORACLE 系统自动地完成，无需用户进行处理。用户只能通过隐式游标的相关属性，来完成相应的操作。
- 在隐式游标的工作区中，所存放的数据是最新处理的一条SQL 语句所包含的数据（与用户自定义的显示游标无关的）。
- INSERT, UPDATE, DELETE, SELECT INTO语句中不必明确定义游标

#### 游标属性

| 属性      | 描述                                                         |
| :-------- | :----------------------------------------------------------- |
| %FOUND    | 布尔型属性,至少有一行被INSERT,DELETE或UPDATE时返回TRUE。     |
| %NOTFOUND | 布尔型属性，与%FOUND相反                                     |
| %ISOPEN   | 布尔型属性，当游标已打开时返回 TRUE布尔型属性, 取值总是FALSE。SQL命令执行完毕立即关闭隐式游标。 |
| %ROWCOUNT | 数字型属性，返回已从游标中读取的记录数                       |

**注：**

- 与最近的sql语句（update,insert,delete,select）发生交互，当最近的一条sql语句没有涉及任何行的时候，则返回相应的值。
- 例如要update一行数据时，如果没有找到，就可以作相应操作。

### 游标变量

**举例：**

声明两个强类型定义游标变量和一个弱类型游标变量

```
DECLARE 
	TYPE deptrecord IS RECORD( 
		Deptno dept.deptno%TYPE, 
		Dname dept.deptno%TYPE, 
		Loc dept.loc%TYPE ); 
 	TYPE deptcurtype IS REF CURSOR RETURN dept%ROWTYPE; 
 	TYPE deptcurtyp1 IS REF CURSOR RETURN deptrecord; 
 	TYPE curtype IS REF CURSOR; 
 	Dept_c1 deptcurtype; 
 	Dept_c2 deptcurtyp1; 
 	Cv curtype;
```

#### 游标变量操作

游标变量操作也包括打开、提取和关闭三个步骤

#### 打开游标变量 ：

打开游标变量时使用的是OPEN…FOR 语句。

格式为：

```
OPEN {cursor_variable_name | :host_cursor_variable_name} 
	FOR select_statement;
```

**注：**

- cursor_variable_name为游标变量
- host_cursor_variable_name为PL/SQL主机环境（如OCI: ORACLE Call Interface，Pro*c 程序等）中声明的游标变量。
- OPEN…FOR 语句可以在关闭当前的游标变量之前重新打开游标变量，而不会导致CURSOR_ALREAD_OPEN异常错误。
- 新打开游标变量时，前一个查询的内存处理区将被释放

#### 提取游标变量数据 ：

使用FETCH语句提取游标变量结果集合中的数据。格式为：

```
FETCH {cursor_variable_name | :host_cursor_variable_name} 
	INTO {variable [, variable]…| record_variable};
```

**注：**

| 属性                      | 含义             |
| :------------------------ | :--------------- |
| cursor_variable_name      | 游标变量名称     |
| host_cursor_variable_name | 宿主游标变量名称 |
| variable                  | 普通变量名称     |
| record_variable           | 记录变量名称     |

#### 关闭游标变量 ：

CLOSE语句关闭游标变量，格式为：

```
CLOSE {cursor_variable_name | :host_cursor_variable_name}
```

**注：**

- cursor_variable_name和host_cursor_variable_name分别为游标变量和宿主游标变量名称
- 如果应用程序试图关闭一个未打开的游标变量，则将导致INVALID_CURSOR异常错误。

### 游标类型

| 类型           | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| 静态游标       | 显式游标和隐式游标称为静态游标，因为在使用他们之前，游标的定义已经完成，不能再更改。 |
| 动态游标       | 游标在声明时没有设定，在打开时可以对其进行修改。分为强类型游标和弱类型游标。 |
| 强类型动态游标 | 在声明变量时使用return关键字定义游标的返回类型               |
| 弱类型动态游标 | 在声明变量时不使用return关键字定义游标的返回类型             |

#### 游标变量应用举例

##### 强类型参照游标变量类型

```
DECLARE 
 	TYPE emp_job_rec IS RECORD( 
  		Employee_id emp.empno%TYPE, 
  		Employee_name emp.ename%TYPE, 
  		Job_title emp.job%TYPE); 
 	TYPE emp_job_refcur_type IS REF CURSOR RETURN emp_job_rec; 
 	Emp_refcur emp_job_refcur_type ; 
 	Emp_job emp_job_rec;
BEGIN 
 	OPEN emp_refcur FOR  SELECT empno, ename, job FROM emp ORDER BY deptno; 
	FETCH emp_refcur INTO emp_job; 
	WHILE emp_refcur%FOUND LOOP 
	DBMS_OUTPUT.PUT_LINE(emp_job.employee_id||': '
                         ||emp_job.employee_name
                         ||' is a '
                         ||emp_job.job_title); 
	FETCH emp_refcur INTO emp_job; 
	END LOOP; 
END;
```

##### 弱类型参照游标变量类型

```
DECLARE 
 	Type refcur_t IS REF CURSOR; 
 	Refcur refcur_t; 
 	TYPE sample_rec_type IS RECORD ( 
  		Id number, 
 		 Description VARCHAR2 (30) ); 
 	sample sample_rec_type; 
 	selection varchar2(1) := UPPER (SUBSTR ('&tab', 1, 1));
```