---
layout: post
title:  Oracle数据库 - 不可用问题总结
date:   2016-11-15 22:15:00 +0800
categories: Oracle数据库
tag: 不可用问题
---

* content
{:toc}

## 1.1 SCN过大问题

### 描述

* BDC中的SCN以惊人的速率增加，并接近可能导致在极端情况下数据库关闭的上限
* 知识点
	- Oracle数据库在安装了2012年1月发布的CPU或PSU补丁之后，经常出现下面一些现象
		- `ORA-19706: invalid SCN`
		- `Advanced SCN by 68093 minutes worth to 0x0ba9.4111a520, by distributed transaction remote logon, remote DB:xxxx.`
		- `Rejected the attempt to advance SCN over limit by 166 hours worth to 0x0ba9.3caec689, by distributed transaction remote logon, remote DB: xxxx`
		- `Warning: The SCN headroom for this database is only 38 days!`
		- 如果说以上的现象只是警告或应用级报错，影响范围有限，那么不幸的是如果遇到RECO进程在恢复分布式事务时遇到SCN问题，则可能使数据库宕掉
	- SCN可以说是Oracle中的很基础，它是一个单向增长的“时钟”，广泛应用于数据库的恢复、事务ACID、一致性读还有分布式事务中
		- Oracle内部，SCN分为两部分存储(低32位-scn base和高16位-scn wrap),是一个48位的整数,最大就是2^48（2的48次方, 281万亿，281474976710656）
		- Maximum Reasonable SCN：在当前时间点，SCN最大允许达到（或者说最大可能）的SCN值。也称为Reasonable SCN Limit，简称RSL,计算(当前时间-1988年1月1日）*24*3600*SCN每秒最大可能增长速率
		- SCN最秒最大可能增长速率是多少: 11.2.0.2之前是16384（即16K - 500年）;11.2.0.2及之后版本是32768（即32K）,可通过隐含参数`_max_reasonable_scn_rate`来设置
		- SCN Headroom：这个是指Maximum Reasonable SCN与当前数据库SCN的差值。在alert中通常是以“天”为单位，天数=(Maximum Reasonable SCN-Current SCN)/16384/3600/24
			- **要到达Maximum Reasonable SCN，得必须以SCN最大可能速率的2倍才行**
	- `_external_scn_rejection_threshold_hours`参数
		- 11.2.0.2及以上版本的这个参数默认值是24，其他版本默认值是744
		- db link依赖很严重的系统可能会导致ORA-19706错误，从而影响业务，严重时还会宕机
		
### 原因

* 分布式数据库中，很多数据库之间通过dblink进行操作数据，过高的SCN进行无限扩散

### 解决

* 统计所有数据库信息，包括版本，PSP补丁，Headroom，当前SCN值，Intrinsic(本身)SCN值
{% highlight sql %}
$ conn sys
set linesize 625 
column host_name format a30
column PSU format a20
column current_scn format 999999999999999
column links format 999
column threshold format a5
col name for a8
select
(select instance_name from v$instance) instance_name,';',
(select to_char(sysdate,'YYYY-MM-DD') from dual) system_date,';',
(select host_name from v$instance) host_name,';',
(select version from v$instance) version,';',
(select cmt from (select substr(comments,1,20) cmt from sys.registry$history where (COMMENTS like '%PSU%' OR COMMENTS like '%CPU%') and COMMENTS not like '%OJVM%' order by action_time desc ) where rownum=1) PSU,';',
   to_char(SYSDATE,'YYYY/MM/DD-HH24:MI:SS') DATE_TIME,';',
   ((((
    ((to_number(to_char(sysdate,'YYYY'))-1988)*12*31*24*60*60) +
    ((to_number(to_char(sysdate,'MM'))-1)*31*24*60*60) +
    (((to_number(to_char(sysdate,'DD'))-1))*24*60*60) +
    (to_number(to_char(sysdate,'HH24'))*60*60) +
    (to_number(to_char(sysdate,'MI'))*60) +
    (to_number(to_char(sysdate,'SS')))
    ) * (16*1024)) - current_scn)
   / (16*1024*60*60*24)
   )Headroom,';',
   current_scn,';',
   (select round( (select value from v$sysstat where name = 'calls to kcmgas')/ ( to_number( sysdate - ( select startup_time from v$Instance)) * 60*60*24), 2)  from dual) "Avg Int SCN/SEC",';',
(select to_char(startup_time,'YYYY/MM/DD') from v$instance) startup_time,';',
(select count(*) from dba_db_links) links,';',
(select value from v$parameter where lower(name) like '%external_scn_logging_threshold_seconds') threshold,';',
(select min(name) from v$ASM_DISKGROUP) ASM
   from v$database;

{% endhighlight %}

* 查看`_external_scn_logging_threshold_seconds`, 并设置到600-若出现600s内跳变记入log中

{% highlight sql %}
$ sqlplus / as sysdba 
col Parameter for a30 
col "Session Value" for a20 
col "Instance Value" for a20 

select a.ksppinm "Parameter", b.ksppstvl "Session Value", c.ksppstvl "Instance Value" 
from x$ksppi a, x$ksppcv b, x$ksppsv c 
where a.indx = b.indx 
and a.indx = c.indx 
and lower (a.ksppinm) in ( '_external_scn_logging_threshold_seconds');

{% endhighlight %}

* 针对于"Avg Int SCN/SEC”>5000的数据库，对其应用程序升入调查并进行研究，移除多余或不需要的dblink（特别是开发测试环境到生产环境）
* 针对于各个版本(11.2.0.2, 11.2.0.3, 11.2.0.4) 打上最新的PSU补丁(160419)
* 
	
> 参考 [SCN、ORA-19706错误和_external_scn_rejection_threshold_hours参数](http://www.laoxiong.net/scn-ora-19706-_external_scn_rejection_threshold_hours-parameter.html)


# 1.2  问题
## 描述

## 原因

## 解决

# 1.3  问题
## 描述

## 原因

## 解决




