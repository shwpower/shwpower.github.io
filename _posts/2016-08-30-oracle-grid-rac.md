---
layout: post
title:  Oracle数据库 - RAC管理
date:   2016-08-30 20:15:00 +0800
categories: Oracle数据库
tag: [数据库管理,RAC,数据库,Oracle]
---

* content
{:toc}

# 1 RAC介绍

## 1.1 RAC概要

![](http://docs.oracle.com/cd/E11882_01/rac.112/e41960/img/haovw002.gif)

## 1.2 Clusterware概要

*  Clusterware在所有Oracle数据库平台上提供了一个完整的集成的集群软件管理解决方案
*  它提供管理群集数据库所需的所有功能，包括节点成员资格，组服务，全局资源管理和高可用性功能
*  Oracle Clusterware管理的任何东西称为CRS资源。 CRS资源可以是数据库，实例，服务，侦听器，VIP地址或应用进程
*  Oracle Clusterware根据存储在Oracle集群注册表（OCR）中的资源配置信息管理CRS资源

## 1.3 RAC架构及处理

### 集群Cluster-Aware存储方案

* 共享数据，支持以下文件存储选择
    - Oracle ASM
    - OCFS2或OCFS(Windows)
    - 认证过的NFS

### 使用服务和VIP地址连接到Oracle数据库

* Oracle RAC数据库具有与非集群Oracle数据库和特定于Oracle RAC的其他进程和内存结构相同的进程和内存结构
* Oracle Net Services启用跨Oracle RAC数据库中所有实例的应用程序连接的负载平衡


### 关于Oracle RAC软件组件

* 使用Cache Fusion，Oracle RAC环境在逻辑上组合每个实例的缓冲区缓存(SGA)，以使实例能够处理数据，就像数据驻留在逻辑上组合的单个缓存中一样
* 为了确保每个Oracle RAC数据库实例获得满足查询或事务所需的块，Oracle RAC实例使用两个进程：全局缓存服务（GCS）和全局队列服务（GES）
* GCS和GES使用全局资源目录（GRD）保存每个数据文件和每个缓存块的状态记录
* 当需要一致性块或在另一个实例上需要更改块时，Cache Fusion会直接在受影响的实例之间传输块映像
* Oracle RAC使用专用互连来进行实例间通信和块传输
* GES监视器和实例Enqueue Process管理对Cache Fusion资源的访问和排队恢复处理

### 关于Oracle RAC后台进程

* GCS和GES进程，GRD协作启用缓存融合
* ACMS：原子控制文件到内存服务
    - 有助于确保分布式SGA内存更新在发生故障时成功全局提交或中止
* GTX0-j：全局事务处理，为XA全局事务提供了透明的支持
    - 数据库根据XA全局事务的工作负载自动调整这些进程的数量
* LMON: Global Enqueue Service Monitor，监视集群中的全局队列和资源，并执行全局入队恢复操作
* LMD: Global Enqueue Service Daemon，管理每个实例中的传入远程资源请求
* LMS: Global Cache Service Process，在全局资源目录（GRD）中记录信息来维护数据文件状态和每个缓存块的记录
    - LMS进程还控制到远程实例的消息流，并管理全局数据块访问，并在不同实例的缓冲区高速缓存之间传输块映像
* LCK0: Instance Enqueue Process，管理非缓存Fusion资源请求，例如库和行缓存请求
* RMSn: Oracle RAC Management Processes (RMSn)，为Oracle RAC执行可管理性任务，还包括在将新实例添加到集群时创建与Oracle RAC相关的资源
* RSMN: Remote Slave Monitor，管理远程实例上的后台从属进程（代表在另一个实例中运行的协调进程执行任务）创建和通信

## 1.4 自动工作负载管理

* 自动工作负载管理使您能够管理工作负载的分布，为用户和应用程序提供最佳性能
* 为数据库连接提供最高可用性，快速故障恢复，以及跨活动配置最佳平衡工作负载

### 组件

* High Availability框架：使Oracle数据库始终保持组件处于运行状态
* Single Client Access Name (SCAN)：DNS或GNS中定义的单个网络名称和IP地址，所有客户端都应使用这些名称和IP地址访问Oracle RAC数据库
* Load Balancing Advisory：数据库向应用程序提供有关数据库及其实例提供的当前服务级别的信息的能力
* Services：一个强大的自动工作负载管理工具，以实现企业grid vision
    - 服务是可以在Oracle RAC数据库中定义的实体
    - 服务使您能够对数据库工作负载进行分组，并将工作路由到分配用于处理服务的最佳实例
* Server Pools: 使CRS管理员能够创建一个策略，定义Oracle Clusterware如何分配资源
* Connection Load Balancing: Net Services为数据库连接提供连接负载平衡

## 1.5 RAC安装

* 使用OUI安装Oracle Clusterware和Oracle数据库软件，并使用DBCA创建数据库
* 安装后在三个级别管理Oracle RAC环境：
    - 实例管理
    - 数据库管理
    - 群集管理

### 了解Oracle RAC环境中的兼容性

### Oracle RAC安装和数据库创建概述

### 扩展Oracle Grid Infrastructure和Oracle RAC软件概述

# 2 RAC管理

## 2.1 存储管理

* 数据文件/redo日志必须在共享存储(集群文件系统/裸设备)， 推荐SPFILE放共享存储上，实例的Pfile可以放本地

### OFA

* Optimal Flexible Architecture确保可靠的安装并提高软件可管理性
* 简化了Oracle软件安装的组织方式，从而简化了安装的持续管理，并通过使默认Oracle数据库安装更符合OFA规范来提高可管理性
* `echo $ORACLE_BASE`

### 数据文件和重做日志

* 所有实例必须都能访问到数据文件 `alter system check datafiles;`
* 每个实例都要有自己的Redo日志（至少2两组 线程）
* `alter database add logfile `语句，默认加到当前的实例

### UNDO管理

* 推荐用`auto`

### ASM

* Oracle ASM通过管理Oracle ASM管理的磁盘上的存储配置，自动最大化I / O性能
* Oracle RAC中的存储管理
    - ASMCA创建Oracle ASM磁盘组并为Oracle ASM磁盘组配置镜像
    - Oracle RAC数据库运行后可以使用Oracle Enterprise Manager管理Oracle ASM磁盘组
    - 集群验证实用程序（CVU）来验证集群中Oracle ASM的完整性 `cluvfy comp asm [-n node_list] [-verbose]`
    - `cluvfy comp ssa`定位共享存储
    
* 修改Oracle ASM的磁盘组配置: OEM, sqlplus, asmca, srvctl
* Oracle ASM磁盘组管理
* 在远程群集中配置首选镜像只读磁盘`show parameter asm_preferred_read_failure_groups`
* 将非集群Oracle ASM转换为集群Oracle ASM
* 在Oracle RAC中使用SRVCTL管理Oracle ASM实例
    - 添加`srvctl add asm`
    - 删除`srvctl remove asm [-f]`
    - 启动`srvctl start asm [-n <node_name>] [-o <start_options>]`
    - 关闭`srvctl stop asm [-n <node_name>] [-o <stop_options>]`
    - 配置`srvctl config asm -n <node_name>`
    - 查看`srvctl status asm [-n <node_name>]`
    - `-- Warning:-n option has been deprecated and will be ignored.`

  
## 2.2 实例和数据库管理

* Administrator-managed databases: 全部需要由数据库管理员来管理（11gR2之前的管理方式）
* policy-managed database: 11.2开始引入，服务器池允许你将集群细分成多个逻辑单元，这在共享环境中很有用
    - 使用自动化特性来增删实例和服务
    - 启动的节点的数量由服务器池的基数来配置
    - 默认情况下，在一个全新的安装之后会产生两个池：自由池(free pool)和通用池(generic pool)，所有非指定的节点都分配给自由池

### 管理工具

* OEM
* SQL*Plus
    - 设置SQL提示 `set sqlprompt '_connect_identifier> '`
    - sql如果影响实例 
        - checkpoint`alter system checkpoint local;` `alter system check [global];`
        - 切换日志`alter system switch logfile;`
        - 归档 `alter system archive log current;` 
    - 只针对于当前实例
        - `show instance`
        - `show parameter` `show sga`
        - `startup` `shutdown`
    - 数据库级别的命令，如`recover`
    
* SRVCTL

### 开关实例和数据库


### 验证实例运行情况


### 终止特定实例的会话

### RAC中的参数文件

### 管理员管理转换到策略管理


### Quiescing数据库


### 多集群内部通信

### 定制化管理

### 高级OEM管理

## 2.3 RAC One Node管理


# 3 自动工作负载管理

# 4 备份恢复

## 4.1 配置RMAN和归档

## 4.2 管理备份和恢复


# 5 集群操作

## 5.1 克隆集群

## 5.2 增删节点

# 6 规划和设计

# 7 监控性能

> 参考文档 [Real Application Clusters Administration and Deployment Guide](http://docs.oracle.com/cd/E11882_01/rac.112/e41960/toc.htm)
