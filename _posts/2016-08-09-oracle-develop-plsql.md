---
layout: post
title:  Oracle数据库 - PL/SQL
date:   2016-08-09 17:15:00 +0800
categories: Oracle数据库
tag: [Oracle,数据库开发,PL/SQL]
---

* content
{:toc}

# 1 PL/SQL概要

## PL/SQL优势

## PL/SQL主要特征


## PL/SQL架构


# 2 PL/SQL语言基础

* 字符集：数据库字符集、国家字符集
* 词汇：分隔符、标识符、文本、注释、空白字符
* 声明：变量声明、常量声明、初始化值、NOT NULL约束、%TYPE属性
* 标识符及其作用域/可见性
* 变量赋值：:=, SELECT INTO/FETCH, 子程序的OUT/IN OUT
* 表达式：连接、运算（算数/逻辑）、比较（IS [NOT]NULL, 关系,LIKE,BETWEEN,IN）、Case
* 错误报告：SQLCODE, SQLERRM
* SQL函数：聚合/分析/编解码/挖掘等
* 编译及条件编译: $IF $THEN $ELSE $ELSIF $ERROR $$<>-$$PLSQL_LINE $$PLSQL_UNIT 静态常量

# 3 数据类型

* SQL数据类型：最大大小、额外的BINARAY_FLOAT或DOUBLE常量  
    - `--ORA-06502: PL/SQL: numeric or value error: character string buffer too small`
* BOOLEAN
* PLS_INTEGER（可转换为char/varchar2/number/long）和BINARAY_INTEGER
* 用户自定义
	- `subtype counter is natural;`
	- `subtype word is char(6);`

# 4 控制语句

* 条件选择：
    - IF THEN
    - IF THEN ELSE
    - IF THEN ELSIF
    - 简单CASE
    - 查询的CASE
* LOOP循环：
	- 基本LOOP,FOR LOOP,Cursor FOR LOOP(索引、上下界),WHILE LOOP 
	- EXIT, EXIT WHEN
	- CONTINUE, CONTINUE WHEN
* 顺序控制：GOTO, NULL 

# 5 集合和记录

* compsite数据类型

## 5.1 Collections 

* 元素，变量（索引）

* 5方面比较3种集合(元素数量，元素间间隙，未初始化的状态，哪里定义，能否为ADT)
* 关联数组（索引表）：type population is table of number index by varchar2(64)
* VARRAY（大小可变数组）: type foursome is varray(4) of varchar2(15);
* 嵌套表：type reoster is table of varchar2(15);
* 集合的方法：delete,trim/extend,exists,first/last,count,limit,prior/next


## 5.2 Records

* 域(fields)，变量.域名，%TYPE %ROWTYPE

* 3中方法创建: RECORD, %TYPE, %ROWTYPE
* 注意%ROWTYPE中的虚拟列 --ORA-54013: INSERT operation disallowed on virtual columns
* 赋值：SELECT INTO, FETCH, SQL语句， 赋空值(xx:=NULL; )
* 记录的插入和更新及其限制
	

# 6 静态SQL

## 语句

* SELECT
* DML(4)
* TCL(4)
* Lock table

## 伪列

* (表列，不存储)：currval/netxval, level, object_value,rowid, rownumber

## 游标

* 会话游标 (v$open_cursor)：一个指向关于select和DML语句信息的专有SQL区域

* 隐式（SQL游标）:与语句运行后关闭，但属性值保留到下次SELECT/DML语句运行
	* 属性: SQL%ISOPEN SQL%FOUND SQL%NOTFOUND SQL%ROWCOUNT
	* 属性: SQL%BULK_ROWCOUNT SQL%BULK_EXCEPTIONS
* 显式（会话游标）：需要声明定义，并关联查询
	* 打开和关闭
	* 读取数据（变量查询，查询列）
	* 属性：%ISOPEN,%FOUND,%NOTFOUND,%ROWCOUNT(提取了多少行)


## 查询结果集处理

* 隐式游标：SELECT INTO, 隐式游标FOR LOOP
* 显式游标：游标FOR LOOP, OPEN/FETCH/CLOSE

## 游标变量

* 创建（强/弱方式）：TYPE type_name IS REF CURSOR [ RETURN return_type ]
* 打开/关闭
* 推进数据
* 赋值给游标变量
* 游标查询中引用变量
* 游标变量的属性
* 游标变量作为子程序参数
* 游标变量作为主机变量

## 游标表达式

* 嵌套游标: CURSOR(subquery)

## 事务处理和控制

* 并发问题，锁机制
* commit
* rollback [to];
* savepoint;
* 隐式rollback
* set transaction
* 覆盖默认锁:lock table, select for update/for update游标, rowid


## 自治(autonomous)事物 

# 7 动态SQL

* 运行时生成和执行SQL

* native动态SQL：容易阅读，高效编码，执行速度快
* DBMS_SQL包：在不知道输入或输出的数据类型和数量可动态生成
* 什么时候会用？编译时不知道SQL；静态SQL无法实现

## 原生态动态SQL

- EXECUTE IMMEDIATE语句
- OPEN FOR,FETCH和CLOSE语句
	
{% highlight sql %}

v_stmt_str := 'SELECT * FROM employees WHERE job_id = :j';
OPEN v_emp_cursor FOR v_stmt_str USING 'MANAGER';
LOOP
FETCH v_emp_cursor INTO emp_record;
EXIT WHEN v_emp_cursor%NOTFOUND;
END LOOP;
CLOSE v_emp_cursor;
	
{% endhighlight %}
	
- 重复占位符

{% highlight sql %}

sql_stmt := 'INSERT INTO payroll VALUES (:x, :x, :y, :x)';
EXECUTE IMMEDIATE sql_stmt USING a, a, b, a;
EXECUTE IMMEDIATE sql_stmt USING a, a, b, a;
......
	
{% endhighlight %}

## DBMS_SQL包

- DBMS_SQL.TO_REFCURSOR函数
- DBMS_SQL.TO_CURSOR_NUMBER函数

* SQL注入

# 8 子程序、触发器和包

# 9 错误处理

## 编译时报警

*  *_ERRORS视图, show errors;
* 三种类型: severe, performance, informational 
    - `ALTER SESSION SET PLSQL_WARNINGS='ENABLE:ALL';`
	- `ALTER PROCEDURE loc_var COMPILE PLSQL_WARNINGS='ENABLE:PERFORMANCE REUSE SETTINGS;`
	- `ALTER SESSION SET PLSQL_WARNINGS='DISABLE:ALL';`
* DBMS_WARNING包

## 异常处理

{% highlight sql %}

EXCEPTION
  WHEN ex_name_1 THEN statements_1                 -- Exception handler
  WHEN ex_name_2 OR ex_name_3 THEN statements_2  -- Exception handler
  WHEN OTHERS THEN statements_3                      -- Exception handler
END;

{% endhighlight %}
	
* 分类：内部定义（运行时）、预定义、自定义
* 避免和处理异常的原则

## 内部定义异常(ORA-n)

{% highlight sql %}

exception_name EXCEPTION;
PRAGMA EXCEPTION_INIT (exception_name, error_code)

{% endhighlight %}

## 预定义异常

- `TOO_MANY_ROWS -1422`
- `ZERO_DIVIDE*1476`
- `STORAGE_ERROR -6500`


## 自定义异常

`exception_name EXCEPTION;`

##  重声明预定义异常

## 显示Raising异常

* raise语句
* raise_application_error过程


* 异常传输
* 未处理的异常
* 检索错误代码(sqlcode)和错误信息(sqlerrm)
* 异常处理后继续执行或重新开始事务


# 10 操练代码

* [plsql_anonymous.sql](http://note.youdao.com/share/?id=ecf800e96174ea455ad2d83441bd37bc&type=notebook#/9CC2D241EDB4457D86830CB0D8A64912)
* [plsql_anonym2.sql](http://note.youdao.com/share/?id=ecf800e96174ea455ad2d83441bd37bc&type=notebook#/1E35F9908B25427E8AB0B7833657868A)
* [plsql_trigger_exception.sql](http://note.youdao.com/share/?id=ecf800e96174ea455ad2d83441bd37bc&type=notebook#/78408EFE9F13473FA14DA9340E4F5FA7)
* [plsql_putil.pkb](http://note.youdao.com/share/?id=ecf800e96174ea455ad2d83441bd37bc&type=notebook#/756109C46D824846BF4EAB1E51525A36)
* [plsql_putil.pkg](http://note.youdao.com/share/?id=ecf800e96174ea455ad2d83441bd37bc&type=notebook#/0602C970F664458DB65D0ACBA57FD91B)
