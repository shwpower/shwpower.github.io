---
layout: post
title:  MySQL数据库 - MyISAM存储引擎
date:   2016-12-05 21:15:00 +0800
categories: MySQL数据库
tag: [存储引擎,MySQL,MyISAM]
---

* content
{:toc}

# 1 MyISAM存储引擎特性

MyISAM - My **Indexed Sequential Access Method**

![](http://note.youdao.com/yws/api/personal/file/9357DD41BE864E4FBAC3DCEDBE7BFCC9?method=download&shareKey=4d238099c4df17522d0af2840a50c0f8)

# 2 MyISAM表

* 每个MyISAM表以三个文件存储在磁盘上,文件以表名开头
    - `.frm` 表格式(format)
    - `.MYD` 数据文件(MYData)
    - `.MYI` 所有文件(MYIndex)
        - 每个索引文件有个flag,用来提示表是否正确关闭
        - 启动时`--myisam-recover-options`选项在myISAM表打开时会自动检查

* 常见操作
    - `set default_storage_engine=MyISAM;`
    - `create table t (i int) engine=myisam;`
    - `alter table t engine=myisam;`
    - `show tables status like 't' \G`
    - `check table t;`
    - `repair table t;`
    - `show create table t;`
    - 可以在不同服务器上拷贝数据文件和索引文件

* 加锁和并发
    - 加锁:对整张表进行加锁，而不是行
    - 并发:
        - 在读数据的时候，所有的表上都可以获得共享锁（读锁），每个连接都不互相干扰
        - 在写数据的时候，获得排他锁，会把整个表进行加锁，而其他的连接请求(读，写请求)都处于等待中

## 2.1 表特性

* 所有数据存储值以低字节优先存储
    - 二进制移植适用于广泛机器，可能不使用嵌入式系统
* 大文件（63位文件长度）在支持大文件的文件系统和操作系统上被支持
* 表中行限制(2^32)^2 - 1.844e+19
* 索引支持
    - BLOB和TEXT列上可以见索引
    - 允许列中有NULL值(0-1个字节)
    - 每列可以有不同的字符集
* 索引限制
    - 每个表最多64个索引
    - 每个索引支持最多16列
    - 索引的最大长度为1000bytes(可以重编译修改)
* 当按照排序顺序插入行时（如使用AUTO_INCREMENT列时），索引树被拆分，以便高节点只包含一个键（提高索引树空间使用率）
* 表中的auto_increment列中内部处理为自动操作(更快)
    - 删除后的序列项不会在被利用
    - auto_increment可以通过`alter table`或`mysamchk`来重置


## 2.2 Key索引

### 索引常见操作

* `alter table t add content text;`
* `alter table t add fulltext index(content);`
* `show index from t \G`

### 空间需求

* MyISAM表使用B树索引
    - 预估索引文件大小(最坏情况)：(key_length + 4)/0.67
* 字符串索引是空间压缩的（高字节有限被存储）
    - 前缀压缩用于以字符串开头的键
    - 建表时通过`pack_keys=1`指定前缀压缩数
    - `myisampack t.MYI`

## 2.3 存储格式

### Static(固定长度)

* 静态格式是MyISAM表的默认值。当表不包含可变长度列（VARCHAR，VARBINARY，BLOB或TEXT）时使用。每行使用固定数量的字节存储。
* 静态格式是最简单和最安全的（最少受到损坏）。
* 它也是最快的磁盘格式:要根据索引中的行号查找行，将行号乘以行长度以计算行位置
* 写入MyISAM文件时崩溃时，`myisamchk`很容易确定每行的开始和结束，它通常可以回收除了部分写入的所有行。 MyISAM表索引总是可以基于数据行重建。
* 特性
    - CHAR和VARCHAR列以空格填充到指定的列宽.BINARY和VARBINARY列用0x00字节填充到列宽
    - NULL列在行中需要额外的空间(位)来记录它们的值是否为NULL
    - Very quick
    - Easy to cache
    - 在崩溃后容易重建
    - 除非您删除了大量的行，并希望将可用磁盘空间返回到操作系统, 一般不reorg. `optimize table`或`myisamchk -r`
    - 通常需要比动态格式表更多的磁盘空间
    - 计算静态大小行的预期行长度（以字节为单位）表达式
        - ` row_leng = 1+(sum of columns lengths)+
            (numbers of NULL columns + delete_flag +7)/8 +
            (numbers of variable-length columns)`
        - delete_flag(1bit =1) 指示该行是否已被删除

### Dynamic存储

* 如果MyISAM表包含任何可变长度列（VARCHAR，VARBINARY，BLOB或TEXT）或如果使用ROW_FORMAT = DYNAMIC表选项创建表，则使用动态存储格式。
* 动态格式比静态格式复杂一些，因为每行都有一个标题，表示它有多长。 当由于更新而变长时，行可能变得分段（以不连续的片段存储）。
* 可以使用OPTIMIZE TABLE或myisamchk -r对表进行碎片整理。
* 特性
    - 所有字符串列都是动态的，除非长度小于4
    - 每行前面都有一个位图，指示哪些列包含空字符串（对于字符串列）或零（对于数字列）。这不包括包含NULL值的列
    - NULL列在行中需要额外的空间来记录它们的值是否为NULL
    - 通常比固定长度表需要更少的磁盘空间。
    - 每行只使用所需的空间。但是，如果行变大，则会将其拆分为所需的数量，从而导致行碎片。
    - 比静态格式表更难以在崩溃后重建，因为行可能被分段成许多块，并且链接（片段）可能丢失。
    - 动态大小行的预期行长度使用以下表达式计算：
        - `3 + (number of columns + 7) / 8 + (number of char columns) + (packed size of numeric columns) + (length of strings) + (number of NULL columns + 7) / 8`

### Compressed表

* 已压缩存储格式是由myisampack工具创建的只读格式, 已压缩表可以用myisamchk来解压缩。
* 特征：
    - 已压缩表占据非常小的磁盘空间。
    - 每个记录是被单独压缩的，所以只有非常小的访问开支。依据表中最大的记录，一个记录的头在每个表中占据1到3个字.每个列被不同地压缩。通常每个列有一个不同的Huffman树。一些压缩类型如下：
        - 后缀空间压缩。
        - 前缀空间压缩。
        - 零值的数用一个位来存储。
        - 如果在一个整型列中的值有一个小的范围，列被用最小可能的类型来存储。比如，一个BIGINT列（8字节），如果所有它的值在-128到127范围内，它可以被存储为TINYINT列（1字节）
        - 如果一个列仅有一小组可能的值，列的类型被转化成ENUM。
        - 一个列可以使用先前压缩类型的任意合并。
* 可以处理固定长度或动态长度记录。

## 2.4 表的问题

### 损坏的MyISAM表

* 可能得到损坏的表的事件
    - mysqld进程在写入的中间被杀死
    - 发生意外的计算机关机（例如，计算机关闭）
    - 硬件故障
    - 正在使用外部程序（例如myisamchk）来修改服务器正在同时修改的表
    - MySQL或MyISAM代码中的一个软件错误
*

### 未正确关闭的表

* 每个MyISAM索引文件(.MYI)在头有一个计数器，它可以被用来检查一个表是否被恰当地关闭 `clients are using or haven't closed the table properly`
* 计数器的工作方式如下：
	- 表在MySQL中第一次被更新，索引文件头的计数器加一
	- 在未来的更新中，计数器不被改变
	- 当表的最后实例(flush table/缓冲区没有空间)被关闭之时，若表已经在任何点被更新，则计数器减一
	- 当你修理或检查表并且发现表完好之时，计数器被重置为零
	- 要避免与其它可能检查表的进程进行事务的问题，若计数器为零，在关闭时计数器不减一

# 3 MyISAM启动选项

![](http://note.youdao.com/yws/api/personal/file/7A75059CB31641B8935282113A03ACB6?method=download&shareKey=ecf800e96174ea455ad2d83441bd37bc)

* key_buffer_size: 指定索引块缓冲区大小（由所有线程共享），最大值取决于系统RAM限制
* bulk_insert_buffer_size：MyISAM使用特殊的树状高速缓存来为INSERT ... SELECT，INSERT ... VALUES（...），（...），...和LOAD DATA INFILE更快地批量插入将数据添加到非空 表。 此变量限制缓存树的大小（以字节为单位）。 将其设置为0将禁用此优化。 默认值为8MB。
* myisam_max_sort_file_size: 在重新创建MyISAM索引（在REPAIR TABLE，ALTER TABLE或LOAD DATA INFILE期间）时，允许MySQL使用的临时文件的最大大小。该值以字节为单位。
* myisam_sort_buffer_size: 在REPAIR TABLE期间或在使用CREATE INDEX或ALTER TABLE创建索引时对MyISAM索引进行排序时分配的缓冲区大小.最大允许设置为4GB-1,默认值为8MB。
* delay_key_write: OFF,ON(默认),ALL(针对所有表),表写操作时刷新key cache
    - 在查询结束后，不会将索引的改变数据写入磁盘，而是改变内存中的索引数据. 只有在清理缓冲区或关闭表时才将索引块转储到磁盘
* concurrent_insert: 可以并行处理查询和插入MyISAM表里的数据
	- NEVER（或0）禁用并发插入
	- AUTO（默认值或1）对不具有孔的MyISAM表启用并行插入，新数据位于数据文件结尾
	- ALWAYS（或2）为所有MyISAM表启用并行插入，即使是有孔的。若有是进程使用则往表的末尾插入新行，否则的话插入洞中


# 4 MySQL跟InnoDB的区别

* 事务支持
* 锁的粒度
* 索引: 哈希，全文
* 外键


> 参考 [MyISAM百度百科](http://baike.baidu.com/link?url=vFluQd_Al-9HcgKeYGUh4KsXljUEfCRszO6k_hLUdc0iwGeIgZRB1BF1AZwvcqRoIZfH8Yrrp5F0flMGfPrmza)
