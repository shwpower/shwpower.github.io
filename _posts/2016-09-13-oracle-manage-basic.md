---
layout: post
title:  Oracle数据库 - 基本管理
date:   2016-09-13 11:15:00 +0800
categories: Oracle数据库
tag: 数据库管理
---

* content
{:toc}


# 1 创建和配置数据库

## 建库前考虑

* 空间需求
* 文件布局
* global database name: `db_name`, `db_domain`
* 初始化参数
* 数据库字符集
* 所要支持的时区
* database block size: `db_block_size`
* Redo log block size
* 表空间计划：system, sysaux, undo, temp
* 备份恢复策略
* 启动和关闭选项

## 创建数据库

### DBCA

* Interactive模式

* Noninteractive/Silent模式
{% highlight command %}
dbca -silent -createDatabase -templateName General_Purpose.dbc
  -gdbname ora11g -sid ora11g -responseFile NO_VALUE -characterSet AL32UTF8
  -memoryPercentage 30 -emConfiguration LOCAL

dbca -help
{% endhighlight %}

### create database语句

* 1 指定SID `ORACLE_SID`
* 2 确保环境变量 `ORACLE_HOME`
* 3 选择数据库授权方法: password文件或OS登录
* 4 创建Initialization Parameter文件
* 5 创建实例(仅针对Windows): `oradim -NEW -SID sid -STARTMODE MANUAL -PFILE pfile`
* 6 连接到实例
* 7 创建spfile
* 8 开启实例
* 9 执行`create database`语句
* 10 创建额外表空间
* 11 执行脚本build系统视图

> 可选项： 安装额外组件，备份数据库，启用自动实例启动


## 指定初始化参数

* 初始化参数文件
    - Linux/Unix: `$ORACLE_HOME/dbs/init${ORACLE_SID}.ora`
    - Windows: `$ORACLE_HOME\database\init${ORACLE_SID}.ora`
* 指定参数
    - `db_name` `db_domain`
    - `db_recovery_file_dest` `db_recovery_file_dest_size`
    - `control_files`
    - `db_block_size`
    - `processes`
    - `ddl_lock_timeout`
    - `undo_management` 
    - `compatible`
    - `license_max_users`

## 管理初始化参数(spfile)

* server parameter文件：服务器参数文件是运行Oracle数据库服务器的系统上维护的初始化参数的存储库
* spfile文件操作
    - 创建
    - 迁移
    - 备份
    - 导出
    - 恢复
* spfile中参数操作 `alter system`
    - 更改
    - 清除
* 参数文件设置查看
    - `show parameters` `show spparameter`
    - `v$parameter` 当前会话有效的初始化参数的值
    - `v$parameter2` 同上，但每个列表参数值都显示在单独的行中
    - `v$system_parameter` 显示实例的有效初始化参数的值
    - `v_system_parameter` 同上，但每个列表参数值都显示在单独的行中
    - `v$spparameter` 显示SPFILE的当前内容
    
## 管理应用负载(service)

* 数据库服务：用于在Oracle数据库中管理工作负载的逻辑抽象
    - 服务将工作负载分为相互分离的组
    - 服务允许您配置工作负载，管理工作负载，启用和禁用工作负载，并将工作负载作为单个实体进行度量
* 服务创建及修改
    - `srvctl add service -d <db_unique_name> -s <service_name>`
    - `exec dbms_service.create_service` `dbms_service.modify_service`
    
* 服务相关视图
    - `dba_services` `all_services` `v$services`
    - `v$active_services` `v$service_stats` `v$service_event`
    - `v$service_wait_classes` `v$service_mod_act_stat`
    - `v$service_metrics` `v$service_metrics_history`


## 建库后操作 

* 安全考虑：修改管理员用户密码
* 启用Transparent Data Encryption
* 创建Secure External Password Store
* 安装Database Sample Schemas

## 克隆数据库


## 删除数据库

 
# 2 启动和关闭数据库

## 启动数据库

* 启动方式
    - sqlplus
    - rman
    - EM
    - srvctl
* 启动时参数文件 `startup pfile='xx'` 
    - spfile${ORACLE_SID}.ora
    - spfile.ora
    - init${ORACLE_SID}.ora
* 启动模式
    - nomount: 启动实例而不装载数据库,且不允许访问数据库(用于数据库创建或重新创建控制文件)
    - mount: 启动实例并装入数据库,数据库仍是关闭(可进行DBA活动)
    - open: 启动实例并安装并打开数据库
        - 无限制(默认)模式：允许所有访问
        - 限制模式：只允许数据库管理员访问
    - force: 在启动或关闭问题后强制启动实例
    - open recovery: 启动实例并立即完成介质恢复

## 修改数据库状态

* mount数据库到实例 `alter database mount;`
* open数据库 `alter database open [read write];`
* open数据库read-only模式 `alter database open read only;`
* open数据库restricting访问 `alter database enable/disable restricted session`


## 关闭数据库

* shutdown模式
    - Normal
    - Immediate
    - Transaction
    - Abort
* shutdown timeout: 等待用户断开连接或事务完成的关闭模式对等待的时间限制

## Quiesing数据库

* 静止状态：将数据库置于只允许DBA事务，查询，提取或PL/SQL语句的状态

## Supsending和Resuming数据库

# 3 配置数据库自动重启


# 4 管理进程

## 服务器进程概念

* Dedicated服务器进程
* Shared服务器进程
* DRCP

## 配置Shared Server

## 配置Resident Connection Pooling

## 进程操作

* 并发执行
* 终止会话
* 会话/进程查询

## 后台进程

# 5 管理内存

## 内存架构

## 自动内存管理

## 手动内存管理

## 配置Smart Flash Cache


# 6 数据库监控

## 监控错误和报警

## 监控性能

* 锁
* 等待事件

# 7 Diagnostic数据

