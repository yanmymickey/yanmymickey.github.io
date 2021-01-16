---
title: plsql触发器
date: 2019-12-17
tags: [Note,oracle]
categories:
- [Note,Oracle]
comments: false
---

# plsql触发器

## 介绍

- 触发器在数据库里以独立的对象存储，它与存储过程不同的是，
  - 存储过程通过其它程序来启动运行或直接启动运行
  - 触发器是由一个**事件**来启动运行
- 触发器是当某个事件发生时自动地隐式运行。
  - 触发器**不能接收参数**。
  - 运行触发器就叫**触发**或**点火**（firing）
- ORACLE中事件指的是对数据库的表进行的INSERT、UPDATE及DELETE操作或对视图进行类似的操作。
- ORACLE将触发器的功能扩展到了触发ORACLE，如数据库的启动与关闭等。
<!-- more -->
## 触发器类型

主要的触发器有**三种**：

- DML触发器
  - ORACLE可以在DML语句进行触发
  - 可以在DML操作前或操作后进行触发
  - 可以对每个行或语句操作上进行触发
- 替代触发器
  - 由于在ORACLE里，不能直接对由两个以上的表建立的视图进行操作。所以给出了替代触发器。
    - 它是ORACLE专门为进行视图操作的一种处理方法
- 系统（用户事件）触发器
  - 它可以在ORACLE数据库系统的事件中进行触发
    - 如ORACLE系统的启动与关闭等。

## 触发器组成

- 触发事件
  - 即在何种情况下触发TRIGGER
    - 例如：INSERT, UPDATE, DELETE
- 触发时间
  - 即该TRIGGER 是在触发事件发生之前（BEFORE）还是之后(AFTER)触发
    - 也就是触发事件和该TRIGGER 的操作顺序
- 触发器本身
  - 即该TRIGGER 被触发之后的目的和意图，正是触发器本身要做的事情
    - 例如：PL/SQL 块。
- 触发频率
  - 说明触发器内定义的动作被执行的次数
    - 即语句级(STATEMENT)触发器和行级(ROW)触发器
      - 语句级(STATEMENT)触发器
        - 是指当某触发事件发生时，该触发器只执行一次
      - 行级(ROW)触发器
        - 是指当某触发事件发生时，对受到该操作影响的每一行数据，触发器都单独执行一次

## 创建触发器

### 语法

```
CREATE [OR REPLACE] TRIGGER trigger_name 
   	{BEFORE | AFTER } 
   	{INSERT | DELETE | UPDATE [OF column [, column …]]} 
ON [schema.] table_name  
 	[REFERENCING {OLD [AS] old | NEW [AS] new}] 
 	[FOR EACH ROW ] 
 	[WHEN condition] 
 trigger_body;
```

**注：**

- BEFORE 和AFTER
  - 指出触发器的触发时序分别为前触发和后触发方式
  - 前触发是在执行触发事件之前触发当前所创建的触发器
  - 后触发是在执行触发事件之后触发当前所创建的触发器
- FOR EACH ROW
  - 说明触发器为行触发器
  - 行触发器和语句触发器的区别表现在
    - 行触发器要求当一个DML语句操走影响数据库中的多行数据时，对于其中的每个数据行，只要它们符合触发约束条件，均激活一次触发器
    - 语句触发器将整个语句操作作为触发事件，当它符合约束条件时，激活一次触发器
  - 当省略FOR EACH ROW 选项时
    - BEFORE 和AFTER 触发器为语句触发器，
    - INSTEAD OF 触发器则为行触发器
- REFERENCING
  - 说明相关名称
  - 在行触发器的PL/SQL块和WHEN 子句中可以使用相关名称参照当前的新、旧列值，默认的相关名称分别为OLD和NEW。
  - 触发器的PL/SQL块中应用相关名称时，必须在它们之前加冒号(:)，但在WHEN子句中则不能加冒号。
- WHEN
  - 说明触发约束条件
  - Condition 为一个逻辑表达时，其中必须包含相关名称，而不能包含查询语句，也不能调用PL/SQL 函数
  - WHEN 子句指定的触发约束条件
    - **只能**用在**BEFORE 和AFTER 行触发器**中
    - **不能**用在**INSTEAD OF 行触发器和其它类型的触发器**中
- 当一个基表被修改( INSERT, UPDATE, DELETE)时要执行的存储过程，执行时根据其所依附的基表改动而自动触发因此与应用程序无关，用**数据库触发器**可以保证数据的一致性和完整性。

### 每张表最多可创建12种触发器

```
BEFORE INSERT 
BEFORE INSERT FOR EACH ROW 
AFTER INSERT 
AFTER INSERT FOR EACH ROW 
BEFORE UPDATE 
BEFORE UPDATE FOR EACH ROW 
AFTER UPDATE 
AFTER UPDATE FOR EACH ROW 
BEFORE DELETE 
BEFORE DELETE FOR EACH ROW 
AFTER DELETE 
AFTER DELETE FOR EACH ROW
```

## 触发器执行顺序

- 执行 BEFORE语句级触发器
- 对于受语句影响的每一行：
  - 执行 BEFORE行级触发器
  - 执行 DML语句
  - 执行 AFTER行级触发器
- 执行 AFTER语句级触发器

## 创建DML触发器

- 触发器名与过程名和包的名字不一样，它是单独的名字空间，因而触发器名可以和表或过程有相同的名字
- 但在一个模式中触发器名不能相同

### 示例

```
--例： 建立一个触发器, 当职工表 emp 表被删除一条记录时，把被删除记录写到职工表删除日志表中去。  
CREATE OR REPLACE TRIGGER del_emp  
	BEFORE DELETE ON scott.emp FOR EACH ROW 
BEGIN 
	--将修改前数据插入到日志记录表 del_emp ,以供监督使用。 
	INSERT INTO emp_his(deptno,empno,ename,job,mgr,sal,comm,hiredate)VALUES
	(:old.deptno,:old.empno,:old.ename,:old.job,:old.mgr,:old.sal, :old.comm,:old.hiredate); 
END;
```

### 触发器的限制

- CREATE TRIGGER语句文本的字符长度不能超过32KB
  - (由于大小受到限制也不能使用long,blob这样的大变量.如果实在是有复杂的逻辑,要弄个很复杂的触发器,可以通过procedure或function实现一部分功能,然后调用)
- 触发器体内的SELECT 语句只能为SELECT … INTO …结构，或者为定义游标所使用的SELECT 语句
- 触发器中不能使用数据库事务控制语句 COMMIT， ROLLBACK, SVAEPOINT 语句
- 由触发器所调用的过程或函数也不能使用数据库事务控制语句
- 触发器中不能使用LONG, LONG RAW 类型
- 触发器内可以参照LOB类型列的列值，但不能通过 :NEW 修改LOB列中的数据
  - (LOB类型：将信息文件（十进制、二进制）、图像甚至音频信息采用数据库作为保存载体时，就需要使用lob类型数据。每个LOB可以有2GB。)
- - (LOB类型：将信息文件（十进制、二进制）、图像甚至音频信息采用数据库作为保存载体时，就需要使用lob类型数据。每个LOB可以有2GB。)

### 行级别触发器中的相关标识符

当触发器被触发时，要使用被插入、更新或删除的记录中的列值，有时要使用操作前、后列的值 ，可以使用：

- :NEW 修饰符访问操作完成后列的值
- :OLD 修饰符访问操作完成前列的值

| 特性 | INSERT | UPDATE | DELETE |
| :--- | :----- | :----- | :----- |
| NEW  | 有效   | 有效   | NULL   |
| OLD  | NULL   | 有效   | 有效   |

### 示例

```
CREATE OR REPLACE TRIGGER upd_emp 
	BEFORE update ON scott.emp 
  	REFERENCING new AS nn  old AS oo 
	FOR EACH ROW 
	WHEN (nn.sal > 2000) 
BEGIN 
	dbms_output.put_line(:nn.sal||'------'||:oo.sal); 
END;
```

### DML触发器中的谓词

在DML触发器中，他们被不同的DML语句所触发，有三个布尔型函数来确定操作到底是什么：

| 谓词      | 描述                                        |
| :-------- | :------------------------------------------ |
| INSERTING | 如果触发语句是INSERT，则为TRUE，否则为FALSE |
| UPDATING  | 如果触发语句是UPDATE，则为TRUE，否则为FALSE |
| DELETING  | 如果触发语句是DELETE，则为TRUE，否则为FALSE |

```
--例
CREATE OR REPLACE TRIGGER check_emp 
	BEFORE update OR insert OR delete ON scott.emp 
	REFERENCING new AS nn  old AS oo 
   	FOR EACH ROW 
  	WHEN (nn.sal > 2000) 
BEGIN 
  	IF INSERTING THEN 
  		dbms_output.put_line('THE OPERATION IS INSERT'); 
   	ELSIF UPDATING THEN 
		dbms_output.put_line('THE OPERATION IS UPDATE'); 
	ELSIF DELETING THEN 
		dbms_output.put_line('THE OPERATION IS DELETE'); 
	ELSE 
		dbms_output.put_line('OTHERS OPERATION'); 
	END IF; 
END;
```

## 创建替代触发器

### 语法

```
CREATE [OR REPLACE] TRIGGER trigger_name 
INSTEAD OF 
	{INSERT | DELETE | UPDATE [OF column [, column …]]} 
ON [schema.] view_name 
	[REFERENCING {OLD [AS] old | NEW [AS] new}] 
	[FOR EACH ROW ] 
trigger_body;
```

**注：**

- INSTEAD OF
  - 使ORACLE激活触发器，而不执行触发事件。
  - 与DML触发器不同，DML触发器是在DML操作之外运行的，而替代触发器则代替激发它的DML语句运行。
  - 替代触发器是行一级的。只能对视图和对象视图建立INSTEAD OF触发器，而不能对表、模式和数据库建立INSTEAD OF 触发器。
  - 而INSTEAD OF 触发器则为行触发器。
- REFERENCING
  - 说明相关名称，在行触发器的PL/SQL块和WHEN子句中可以使用相关名称参照当前的新、旧列值，默认的相关名称分别为OLD和NEW。
  - 触发器的PL/SQL块中应用相关名称时，必须在它们之前加冒号(:)，但在WHEN子句中则不能加冒号。 WHEN 子句说明触发约束条件。
  - Condition 为一个逻辑表达时，其中必须包含相关名称，而不能包含查询语句，也不能调用PL/SQL 函数。
- WHEN
  - 指定的触发约束条件只能用在BEFORE 和AFTER 行触发器中，不能用在INSTEAD OF 行触发器和其它类型的触发器中。
- INSTEAD_OF
  - 用于对视图的DML触发，由于视图有可能是由多个表进行联结(join)而成，因而并非是所有的联结都是可更新的。但可以按照所需的方式执行更新

### 替代触发器应用举例

我们可以创建INSTEAD_OF触发器来为 DELETE 操作执行所需的处理，即删除EMP表中所有基准行：

```
CREATE OR REPLACE TRIGGER emp_view_delete 
	INSTEAD OF DELETE ON emp_view FOR EACH ROW 
BEGIN 
	DELETE FROM emp WHERE deptno= :old.deptno; 
END emp_view_delete; 
DELETE FROM emp_view WHERE deptno=10;
```

## 创建系统触发器

- 系统触发器可以在DDL或数据库系统上被触发。DDL指的是数据定义语言，如CREATE 、ALTER及DROP 等。
- 数据库系统事件包括数据库服务器的启动或关闭，用户的登录与退出、数据库服务错误等。

### 语法

```
CREATE OR REPLACE TRIGGER [sachema.] trigger_name 
	{BEFORE|AFTER}   
	{ddl_event_list | database_event_list} 
ON { DATABASE | [schema.] SCHEMA } 
	[WHEN_clause]  
trigger_body;
```

**注：**

- ddl_event_list为一个或多个DDL事件，事件间用 OR 分开；
- database_event_list为一个或多个数据库事件，事件间用 OR 分开；
- 系统事件触发器既可以建立在一个模式上，又可以建立在整个数据库上。
  - 当建立在模式(SCHEMA)之上时，只有模式所指定用户的DDL操作和它们所导致的错误才激活触发器, 默认时为当前用户模式。
  - 当建立在数据库(DATABASE)之上时，该数据库所有用户的DDL操作和他们所导致的错误，以及数据库的启动和关闭均可激活触发器。
  - 要在数据库之上建立触发器时，要求用户具有ADMINISTER DATABASE TRIGGER权限。

### 系统触发器的种类和事件出现的时机（前或后）：

| 事件                  | 允许的时机 | 说明                 |
| :-------------------- | :--------- | :------------------- |
| 启动STARTUP           | 之后       | 实例启动时激活       |
| 关闭SHUTDOWN          | 之前       | 实例正常关闭时激活   |
| 服务器错误SERVERERROR | 之后       | 只要有错误就激活     |
| 登录LOGON             | 之后       | 成功登录后激活       |
| 注销LOGOFF            | 之前       | 开始注销时激活       |
| 创建CREATE            | 之前，之后 | 在创建之前或之后激活 |
| 撤消DROP              | 之前，之后 | 在撤消之前或之后激活 |
| 变更ALTER             | 之前，之后 | 在变更之前或之后激活 |

### 示例

```
CREATE OR REPLACE TRIGGER trig4_ddl 
	AFTER CREATE OR ALTER OR DROP  ON DATABASE 
DECLARE 
	Event VARCHAR2(20); Typ VARCHAR2(20); 
	Name VARCHAR2(30);  Owner VARCHAR2(30); 
BEGIN 
	--读取DDL事件属性 
	Event := SYSEVENT;     --激活触发器的事件名称   
    Typ := DICTIONARY_OBJ_TYPE;  --语句所操作的数据库对象类型  
	Name := DICTIONARY_OBJ_NAME; --语句所操作的数据库对象名称  
	Owner := DICTIONARY_OBJ_OWNER; --语句所操作的数据库对象所有者名称  
    -- 将事件属性插入到事件日志表中 
    INSERT INTO scott.eventlog(eventname,obj_type,obj_name,obj_owner) 
    	VALUES(event, typ, name, owner); 
END;
```

## 重新编译触发器

- 如果在触发器内调用其它函数或过程，当这些函数或过程被删除或修改后，触发器的状态将被标识为无效。当DML语句激活一个无效触发器时，ORACLE将重新编译触发器代码，如果编译时发现错误，这将导致DML语句执行失败。

- 在PL/SQL程序中可以调用ALTER TRIGGER语句重新编译已经创建的触发器，格式为：

  ```
  ALTER TRIGGER [schema.] trigger_name COMPILE
  ```

## 删除触发器

### 语法

```
DROP TRIGGER trigger_name;
```

**注：**

- 当删除其他用户模式中的触发器名称，需要具有DROP ANY TRIGGER系统权限
- 当删除建立在数据库上的触发器时，用户需要具有ADMINISTER DATABASE TRIGGER系统权限
- 当删除表或视图时，建立在这些对象上的触发器也随之删除

## 触发器状态

### 有效状态(ENABLE)

- 当触发事件发生时，处于有效状态的数据库触发器TRIGGER 将被触发。

### 无效状态(DISABLE)

- 当触发事件发生时，处于无效状态的数据库触发器TRIGGER 将不会被触发，此时就跟没有这个数据库触发器(TRIGGER) 一样。

### 数据库TRIGGER的这两种状态可以互相转换

- ALTER TRIGGER语句一次只能改变一个触发器的状态
- ALTER TABLE语句则一次能够改变与指定表相关的所有触发器的使用状态。

#### ALTER TIGGER

```
ALTER TIGGER trigger_name [DISABLE | ENABLE ]; 

--例：
ALTER TRIGGER emp_view_delete DISABLE;
```

#### ALTER TABLE

```
ALTER TABLE [schema.]table_name {ENABLE|DISABLE} ALL TRIGGERS;  

--例：使表EMP 上的所有TRIGGER 失效： 
ALTER TABLE emp DISABLE ALL TRIGGERS;
```

## 触发器和数据字典

**相关数据字典：**

- USER_TRIGGERS
- ALL_TRIGGERS
- DBA_TRIGGERS

```
--例：
SELECT TRIGGER_NAME, TRIGGER_TYPE, TRIGGERING_EVENT, TABLE_OWNER, 
	   BASE_OBJECT_TYPE, REFERENCING_NAMES, STATUS, ACTION_TYPE 
	FROM user_triggers;
```