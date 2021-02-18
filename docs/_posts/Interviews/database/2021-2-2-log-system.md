<!--
 * @Author: XPectuer
 * @LastEditor: XPectuer
-->
---
layout: post
categories: database
tag: database log
title: "MySQL的日志系统"
author: noobi
---

# MySQL的日志系统

### 重要的两大日志模块：redo Log与binlog

#### 物理日志 redo log

如果每次数据的请求都可以简单地归结为写盘、读盘等操作。

由于IO操作天然的慢速，以及查询复杂度（反复刷盘）之大，如果有大量并发请求到来，MySQL必定会由于OOM或其他原因丢弃请求。

MySQL的设计者针对这个问题，引入了**WAL（Write-Ahead Logging），即先写日志，再写磁盘**。MySQL会在忙时先将请求操作写入日志，闲时将日志操作真正地执行。

InnoDB有一条记录需要更新时，它会将**记录先写到redo log中，并更新内存中的数据。当空闲时，InnoDB将数据真正写入磁盘。**

InnoDB的redo log是固定大小的，并且是一个**循环队列**。

可以配置一组4个文件，每个文件1GB，那么redo log总共可以记录4GB的操作。

redo log写入时，当数据已满，会将原先的数据抹去（这点与Redis RDB log类似）

​	redo log各维护一个``write pos`` 和 ``checkpoint`` 指针

- ``write pos`` 是当前记录的位置
- ``checkpoint`` 是当前要擦除的位置，擦除记录前要把记录更新到数据文件，保证数据的一致性。

redo log的这个特性，保证了InnoDB的容灾性，即**crash-safe**



#### 归档日志 binlog

binlog是Server层日志，由于redo log是InnoDB的特性，所以MySQL本身自带一个引擎无关的逻辑日志。

redo log是物理日志，记录某个数据页上的具体修改；而binlog则记录语句的原始逻辑，比如“给ID=2 这一行的c字段加一”。

redo log是循环写的，binlog则是追加写入的，不会覆盖原有数据。

#### 两阶段提交

先来看一段SQL语句的执行流程：

1. 执行器找到引擎ID=2的一行。ID是主键，引擎直接用树搜索找到这一行。如果ID=2这一行所在的数据页本来就在内存中，就直接返回给执行器；否则先从磁盘读入内存，再返回。
2. 执行器拿到引擎给的行数据（在引擎看来是 数据页），执行value+1，再调用引擎接口。
3. 引擎将这行数据更新到内存中，先写redo log，写完redo log处于prepare状态，返回给执行器。
4. 执行器追加写binlog，
5. Server调用提交事务接口，**引擎将redo log改成commit状态**。

![2e5bff4910ec189fe1ee6e2ecc7b4bbe](/assets/interviews/databaselog.png)

可以看到，写完redo log，按理说数据已经可以认为是更改了，但是却并未提交。

而是等待binlog写完

再修改redo log的提交状态。

#### 两阶段提交的必要性

那么这个过程能不能省去呢？

​	很简单，不能。可以做一个思维实验：

​	先写了binlog，但是在写redo log时突然crash。我们用binlog恢复后，本来无效的事务操作被恢复，导致数据不一致。

​	或者是先写 redo log，写 binlog 时 crash。恢复后binlog修改的结果会少了redo log里新添的数据，与原先的数据页不一致。

### 如何为数据库做恢复？

1. 为数据库定期做**全量备份**。
2. 先恢复到全量备份到时间点，再根据binlog的操作，精确恢复。

这就有点像我们操作显微镜，先粗调，再细调。