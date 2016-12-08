---
layout: post
title:  MySQL数据库 - 基础入门
date:   2016-09-14 22:05:00 +0800
categories: MySQL数据库
tag: [MySQL,基础入门,SQL]
---

* content
{:toc}
 

# 1 MySQL程序

* 每个MySQL程序都有许多不同的选项。大多数程序提供了一个`--help`选项
* 客户端程序环境变量
    - `MYSQL_UNIX_PORT`默认的Unix套接字文件,用于连接到localhost
    - `MYSQL_TCP_PORT`默认端口号,用于TCP/IP连接
    - `MYSQL_PWD`默认密码
    - `MYSQL_DEBUG`调试时调试跟踪选项
    - `TMPDIR`创建临时表和文件的目录

## 1.1 使用MySQL程序

### 调用MySQL程序

* 命令行调用MySQL程序: 程序名称+任何选项或其他参数，指示程序要执行的操作
* 连接到的服务器的参数选项
    - `--host` `-h` 
    - `--user` `-u`
    - `--password` `-p`
* 其他连接选项 
    - `--port` `-P`
    - `--socket` `-S`

### 连接到MySQL服务器

### 指定程序选项

### 使用命令行上的选项

### 程序选项Modifiers

* 一些选项是“布尔”和控制行为，可以打开或关闭 如`mysql --disable-column-names`

### 使用Option文件

### 影响Option文件的命令行选项


### 使用选项设置程序变量


### 选项的默认值/期望值和'='

## 1.2 服务器程序

### mysqld

### mysqld_safe

### mysql.server

### mysqld_multi

## 1.3 安装相关程序

* comp_err - 编译MySQL错误消息文件
* mysqlbug - 生成错误报告
* mysql_install_db - 初始化MySQL数据目录
* mysql_plugin - 配置MySQL服务器插件
* mysql_secure_installation - 改进MySQL安装安全性
* mysql_tzinfo_to_sql - 加载时区表
* mysql_upgrade - 检查和升级MySQL表

## 1.4 客户端程序

### mysql 
- MySQL命令行工具

* mysqladmin - 用于管理MySQL服务器的客户端
* mysqlcheck - 表维护程序
* mysqldump - 数据库备份程序
* mysqlimport - 数据导入程序
* mysqlshow - 显示数据库，表和列信息
* mysqlslap - 加载仿真客户端

## 1.5 管理和实用程序

* innochecksum - 脱机InnoDB文件校验和实用程序
* myisam_ftdump - 显示全文索引信息

### myisamchk 
- myISAM表维护实用程序

* myisamlog - 显示MyISAM日志文件内容
* myisampack - 生成压缩的只读MyISAM表
* mysql_config_editor - mySQL配置实用程序
* mysqlaccess - 检查访问权限的客户端

### mysqlbinlog 
- 处理二进制日志文件的实用程序

* mysqldumpslow - 总结慢查询日志文件
* mysqlhotcopy - 数据库备份程序
* mysql_convert_table_format - 将表转换为使用给定的存储引擎
* mysql_find_rows - 从文件中提取SQL语句
* mysql_fix_extensions - 规范化表文件名扩展
* mysql_setpermission - 在授权表中交互设置权限
* mysql_waitpid - 终止进程并等待终止
* mysql_zap - 杀死匹配模式的进程

## 1.6 开发实用程序

## 1.7 其他程序


# 2 数据类型

## 2.1 数据类型概述

### 数字类型概述

* M表示整数类型的最大显示宽度与类型可包含的值的范围无关; 对于浮点和定点类型，M是可以存储的数字的总数
* 如果为数字列指定ZEROFILL，MySQL会自动将UNSIGNED属性添加到列中
* 允许UNSIGNED属性的数字数据类型也允许SIGNED(没有效果)
* 整数列的定义中，SERIAL DEFAULT VALUE是NOT NULL AUTO_INCREMENT UNIQUE的别名
    
* 类型
    - bit[(M)]: 位值类型。 M表示每个值的位数，从1到64.如果省略M，默认值为1
    - tinyint[(M)][unsigned][zerofill]: 一个非常小的整数。 有符号范围是-128到127.无符号范围是0到255
    - bool, boolean: tinyint(1)的同义词。 值为零被认为是假的; 非零值被视为true
    - smallint[(M)][unsigned][zerofill]: 一个小整数。 有符号范围是-32768到32767.无符号范围是0到65535
    - mediumint[(M)][unsigned][zerofill]:中型整数。 有符号的范围是-8388608到8388607.无符号的范围是0到16777215
    - int[(M)][unsigned][zerofill]:正常大小的整数。 有符号的范围是-2147483648到2147483647.无符号的范围是0到4294967295
    - integer[(M)][unsigned][zerofill]: int类型的同义词
    - bigint[(M)][unsigned][zerofill]: 一个大整数(64位)。 有符号范围是-9223372036854775808至9223372036854775807.无符号范围是0到18446744073709551615
        - serial: `bigint unsigned not null auto_increment unique`的别名
    - decimal[(M[,D])][unsigned][zerofill]: 塞满的精确浮点数。M是总位数（精度），D是小数点后的位数（刻度）。小数点和（负数）符号不在M中计数。如果D为0，则值不包含小数点或小数部分。 DECIMAL的最大位数（M）为65.支持的最小小数位数（D）为30.如果省略D，则默认值为0.如果省略M，则默认值为10。使用DECIMAL列的所有基本计算（+， - ，*，/）的精度为65位数

## 2.2 数字类型

## 2.3 日期和时间类型


## 2.4 字符串类型


## 2.5 空间数据扩展


## 2.6 数据类型默认值


## 2.7 数据类型存储要求


## 2.8 为列选择正确的类型


## 2.9 使用其他数据库引擎中的数据类型


# 3 函数和运算符

[MySQL 5.6函数/操作符参考表](http://dev.mysql.com/doc/refman/5.6/en/func-op-summary-ref.html)

## 3.1 类型转换

## 3.2 运算符

## 3.3  字符函数

## 3.4 全文搜索

## 3.5 空间(Spatial)分析函数

## 3.6 聚合(group by)函数

## 3.7 函数

### 控制流

### 数字函数和运算符

### Cast函数和运算符

### XML函数

### 为函数和运算符

### Information函数

#### 加密和解密函数 


# 4 SQL语法

## 4.1 DD语句

## 4.2 DM语句

## 4.3 事务和锁语句

## 4.4 复制语句

## 4.5 Prepared SQL语句语法

## 4.6 Compound-Statement语法

## 4.7 数据库管理语句

## 4.8 Utility语句


