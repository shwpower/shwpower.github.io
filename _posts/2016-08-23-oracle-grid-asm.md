---
layout: post
title:  Oracle数据库 - ASM管理
date:   2016-08-23 19:15:00 +0800
categories: Oracle数据库
tag: [数据库管理,Oracle,Grid,Grid计算,ASM,自动存储管理]
---

* content
{:toc}

# 1 Oracle ASM介绍

### 1.1 ASM概述

* Oracle ASM是用于支持单实例Oracle数据库和Oracle Real Application Clusters（Oracle RAC）配置的Oracle数据库文件的卷管理器和文件系统
* Oracle ASM使用磁盘组存储数据文件; Oracle ASM磁盘组是Oracle ASM作为一个单元管理的磁盘集合
* Oracle ASM卷管理器功能提供灵活的基于服务器的镜像选项
    - normal
    - high: three-way mirroring
    - external
* Oracle ASM还使用Oracle托管文件（OMF）功能简化数据库文件管理
* Oracle自动存储管理集群文件系统（Oracle ACFS）是一种多平台，可扩展的文件系统和存储管理技术，可扩展Oracle ASM功能，以支持在Oracle数据库外部维护的客户文件

### 1.2 ASM概念

![](http://docs.oracle.com/cd/E11882_01/server.112/e18951/img/ostmg001.gif)

* Oracle ASM Instances
* Oracle ASM Disk Groups
* Oracle Mirroring and Failure Groups
* Oracle ASM Disks
* Oracle ASM Files
    - Extents
    ![]()
    - Oracle ASM Striping: 为了平衡磁盘组中所有磁盘的负载(粗颗粒条带化)和减少I/O延迟(细颗粒条带化)
        - Fine-Grained Striping ![](http://docs.oracle.com/cd/E11882_01/server.112/e18951/img/ostmg010.gif)
        - Coarse-Grained Striping![](http://docs.oracle.com/cd/E11882_01/server.112/e18951/img/ostmg009.gif)
    - File Templates


### 1.3 了解ASM磁盘组管理

* Discovering Disks
* Mounting and Dismounting Disk Groups
* Adding and Dropping Disks
* Online Storage Reconfigurations and Dynamic Rebalancing

### 1.4 ASM存储考虑

* 配置系统存储时，必须考虑系统的初始容量和未来增长计划

#### 磁盘组的存储资源

* Disk Partition
* Logical Unit Number(LUN)
* Logical Volume
* Network File System(NFS)

#### Oracle ASM和多路径

* 多路径解决方案通过使用冗余物理路径组件提供故障转移,其包括位于服务器和存储子系统之间的适配器，电缆和交换机
* 多路径是在操作系统设备驱动程序级实现的软件技术。多路径创建伪设备，以便于跨所有可用I / O路径的I / O操作的共享和平衡。多路径还通过在所有可用路径上分布I / O负载来提高系统性能，通过自动故障转移和故障回复提供更高级别的数据可用性
* 虽然Oracle ASM未设计具有多路径功能，可以通过将`ASM_DISKSTRING`初始化参数的值设置为与表示多路径磁盘的伪设备相匹配的模式来确保多路径磁盘的发现。

#### 存储准备准则

* 配置两个磁盘组，一个用于数据，另一个用于快速恢复区
* 对于每个磁盘组，建议至少使用四个相同大小和性能的LUN（Oracle ASM磁盘）
* 确保磁盘组中的所有Oracle ASM磁盘具有相似的存储性能和可用性特性
* Oracle ASM数据分发策略是基于容量的(具有相同的容量以维持平衡)
* 使用高端存储阵列时创建外部冗余磁盘组
* 通过在Oracle ASM磁盘组中专用磁盘，最大限度地减少Oracle ASM磁盘和其他应用程序之间的I / O争用
* 选择硬盘RAID条带大小为2的倍数且小于或等于Oracle ASM分配单元的大小
* 对于Linux，使用Oracle ASMLib功能提供一致的设备命名和权限持久性

# 2 管理ASM实例

## 2.1 参数配置

* 参数文件
{% highlight sql %}
show parameter spfile;
create pfile='/tmp/init+ASM.ora' from spfile;
startup pfile='/tmp/init+ASM.ora';
{% endhighlight %}

* 设置参数

{% highlight sql %}
set line 200
select * from v$asm_client;
show parameter sga;
alter system set asm_diskgroup='DATA,FRA';

col name for a40
col value for a40
select name,value from v$parameter
    where name in ('db_cache_size','large_pool_size','shared_pool_size',
        'diagnostic_dest','processes','instance_type');

show parameter remote_;

{% endhighlight %}

* 参数设置推荐
    - asm_diskgroups
    - asm_diskstring
    - asm_power_limit
    - asm_preferred_read_failure_groups
    - db_cache_size
    - diagnostic_dest
    - instance_type
    - large_pool_size
    - processes
    - remote_login_passwordfile
    - shared_pool_size
* 针对于ASM的数据库初始化参数
    - processes 现有值+16
    - large_pool_size 现有值+600k
    - shared_pool_size 基于数据库文件大小，normal冗余下每50G空间需要1M+4MB

## 2.2 实例管理

* srvctl来管理ASM实例
* 使用Oracle Restart

### 启动ASM实例

* 正常连接及开启 `ORACLE_SID` `INSTANCE_TYPE`, startup选项`conn sys as sysasm`
    - force: 在重新启动之前向Oracle ASM实例发出SHUTDOWN ABORT
    - mount/open: 装载在ASM_DISKGROUPS初始化参数中指定的磁盘组
    - nomount: 启动Oracle ASM实例，而不装载任何磁盘组
    - restrict: 在受限模式下启动实例，该实例仅允许具有CREATE SESSION和RESTRICTED SESSION系统特权的用户访问.可以将RESTRICT子句与MOUNT，NOMOUNT和OPEN子句结合使用
* 指定spfile开启实例
* 关于启动时装载磁盘组 ...
* 关于Restrict模式

### 关闭ASM实例

* 建议您在尝试关闭Oracle ASM实例之前关闭使用Oracle ASM实例的所有数据库实例，并卸载在Oracle ASM Dynamic Volume Manager（Oracle ADVM）卷上装载的所有文件系统
* 如果Oracle集群注册表（OCR）或voting文件存储在磁盘组中，则只能通过在关闭节点上的集群件的过程中关闭Oracle ASM实例来卸载磁盘组 `crsctl stop crs`
* shutdown模式
    - `normal`
    - `immediate` `transactional`
    - `abort`

### ASM实例Oracle Restart配置


## 2.3 ASM升级及补丁

## 2.4 ASM实例访问授权

* Using One Operating System Group for Oracle ASM Users
* Using Separate Operating System Groups for Oracle ASM Users
* The SYSASM Privilege for Administering Oracle ASM
* The SYSDBA Privilege for Managing Oracle ASM Components

## 2.5 迁移数据库到ASM

### OEM迁移数据库到ASM
### RMAN迁移数据库到ASM
### [MAA最佳实践](http://www.oracle.com/technetwork/database/features/availability/maa-096107.html)


# 3 管理ASM磁盘组

## 3.1 磁盘组概述

### 属性

* access_control.enabled
* access_control.umask
* au_size
* cell.smart_scan_capable
* compatible.asm
* compatible.rdbms
* compatible.advm
* content.type
* disk_repair_time
* idp.boundary idp.type
* sector_size
* storage_type

### 容量

### 文件访问控制

### 性能和扩展性

### 兼容性


### Metadata内部一致性

## 3.2 Disk Discovery

## 3.3 ASM镜像和磁盘组冗余

## 3.4 磁盘组操作

### 创建

### 修改

### 删除

### 重命名

### 装载和卸载


# 4 管理Oracle ACFS

# 5 管理ASM文件/目录/模板

# 6 RMAN迁移ASM数据

# 7 ASM工具

## 7.1 Oracle企业管理

## 7.2 ASMCA

## 7.3 ASM命令行工具

## 7.4 ACFS命令行工具


> 参考文档: [Automatic Storage Management Administrator's Guide](http://docs.oracle.com/cd/E11882_01/server.112/e18951/toc.htm)
