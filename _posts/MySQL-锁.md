---
title: MySQL-锁
date: 2018-11-14 21:17:51
tags: MySQL 锁
categories: 数据库 MySQL
---
# MySQL-锁

## 什么是锁
- 锁机制用于管理对共享资源的并发访问

## lock与latch
- latch一般称为闩锁（轻量级的锁） ， 因为其要求锁定的时间必须非常短。 若持续的时间长， 则应用的性能会非常差。 在InnoDB存储引擎中， latch又可以分为mutex（互斥量）和rwlock（读写锁）。其目是用来保证并发线程操作临界资源的正确性， 并且通常没有死锁检测的机制。
- lock的对象是事务，用来锁定的是数据库中的对象，如表、页、行。并且一般lock的对象仅在事commit或rollback后进行释放（不同事务隔离级别释放的时间可能不同）。此外，lock，正如在大多数数据库中一样，是有死锁机制的。

![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/84119853.jpg)

## InnoDB存储引擎中的锁
### 锁的类型
- 共享锁（S Lock），允许事务读一行数据
- 排他锁（X Lock），允许事务删除或更新一行数据
- 意向锁（Intention Lock）是将锁定的对象分为多个层次，意向锁意味着事务希望在更细粒度（fine granularity）上进行加锁。对最细粒度的对象进行上锁， 那么首先需要对粗粒度的对象上锁。
#### 各种锁之间的兼容性
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/91462658.jpg)
#### 三张表
- INNODB_TRX显示了当前运行的InnoDB事务
- INNODB_LOCKS查看锁
- INNODB_LOCK_WAITS可以很直观地反映当前事务的等待

### 一致性非锁定读
- 一致性的非锁定读（consistent nonlocking read） 是指InnoDB存储引擎通过行多版本控制（multi versioning） 的方式来读取当前执行时间数据库中行的数据。 如果读取的行正在执行DELETE或UPDATE操作， 这时读取操作不会因此去等待行上锁的释放。 相反地，InnoDB存储引擎会去读取行的一个快照数据。非锁定读机制大大地提高了数据库的并发性。
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/66097705.jpg)
#### 注意
- 在READ COMMITTED事务隔离级别下，对于快照数据，非一致性读总是读取被锁定行的最新一份快照数据(**违反了隔离性**)。 而在REPEATABLE READ事务隔离级别下，对于快照数据，非一致性读总是读取事务开始时的行数据版本。

### 一致性锁定读
用户需要显式地对数据库读取操作进行加锁以保证数据逻辑的一致性。

### 自增长与锁
#### 概述 
- 插入操作会依据这个自增长的计数器值加1赋予自增长列。这个实现方式称做AUTO-INC Locking。这种锁其实是采用一种特殊的表锁机制，为了提高插入的性能，锁不是在一个事务完成后才释放，而是在完成对自增长值插入的SQL语句后立即释放。
- 虽然AUTO-INC Locking从一定程度上提高了并发插入的效率， 但还是存在一些性能上的问题。 首先， 对于有自增长值的列的并发插入性能较差， 事务必须等待前一个插入的完成（虽然不用等待事务的完成） 。 其次， 对于INSERT…SELECT的大数据量的插入会影响插入的性能， 因为另一个事务中的插入会被阻塞。

#### 自增长的模式
- 参数innodb_autoinc_lock_mode来控制自增长的模式
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/49495821.jpg)
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/26079309.jpg)

#### 注意
- 在InnoDB存储引擎中， 自增长值的列必须是索引， 同时必须是索引的第一个列。 如果不是第一个列， 则MySQL数据库会抛出异常， 而MyISAM存储引擎没有这个问题。
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/60316580.jpg)

### 外键与锁
- 对于外键值的插入或更新，首先需要查询父表中的记录，即SELECT父表。但是对于父表的SELECT操作，不是使用一致性非锁定读的方式，因为这样会发生数据不一致的问题，因此这时使用的是SELECT…LOCK IN SHARE MODE方式，即主动对父表加一个S锁。如果这时父表上已经这样加X锁，子表上的操作会被阻塞。

## 锁的算法
### 行锁的三种算法
- Record Lock：单个行记录上的锁。
- Gap Lock：间隙锁，锁定一个范围，但不包含记录本身。
- Next-Key Lock∶Gap Lock+Record Lock，锁定一个范围，并且锁定记录本身。

### 注意
- 当查询的索引含有唯一属性时，InnoDB存储引擎会对Next-Key Lock进行优化，将其降级为Record Lock，即仅锁住索引本身， 而不是范围。
- 在InnoDB存储引擎中， 对于INSERT的操作， 其会检查插入记录的下一条记录是否被锁定， 若已经被锁定， 则不允许查询。 

### 解决phantom problem--针对其他提交前后，读取数据条数的对比
- Phantom Problem是指在同一事务下，连续执行两次同样的SQL语句可能导致不同的结果，第二次的SQL语句可能会返回**之前不存在的行，违反了事务的隔离性**。
- InnoDB存储引擎采用Next-Key Locking的算法避免Phantom Problem。

#### 幻影读示例
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/92904378.jpg)

#### 解决幻影读
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/19298541.jpg)

## 锁问题
深入理解脏读/幻影读/不可重复读/丢失更新的区别：https://www.cnblogs.com/Hakuna-Matata/p/7772794.html
### 脏读--针对未提交数据
#### 脏页
- 脏页指的是在缓冲池中已经被修改的页，但是还没有刷新到磁盘中，即数据库实例内存中的页和磁盘中的页的数据是不一致的。

#### 脏数据
- 脏数据是指事务对缓冲池中行记录的修改，并且还没有被提交（commit）。

#### 脏读
- 脏读指的就是在不同的事务下， 当前事务可以**读到另外事务未提交的数据**， 简单来说就是可以读到脏数据。
- 对于脏页的读取，是非常正常的，但如果读到了脏数据，即一个事务可以读到另外一个事务中未提交的数据，则显然**违反了数据库的隔离性**。脏读发生的条件是需要事务的隔离级别为READ UNCOMMITTED。
- 脏读隔离看似毫无用处， 但在一些比较特殊的情况下还是可以将事务的隔离级别设置为READ UNCOMMITTED。 例如replication环境中的slave节点， 并且在该slave上的查询并
不需要特别精确的返回值。

#### 脏读示例
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/27843486.jpg)

#### 解决脏读
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/90598726.jpg)


### 不可重复读--针对其他提交前后，读取数据本身的对比
- 不可重复读是指在一个事务内多次读取同一数据集合。在这个事务还没有结束时，另外一个事务也访问该同一数据集合，并做了一些**DML操作**。这样就发生了在一个事务内两次读到的数据是不一样。
- 在InnoDB存储引擎中，通过使用Next-Key Lock算法来避免不可重复读的问题。
- 不可重复读和脏读的区别是： 脏读是读到未提交的数据， 而不可重复读读到的却是已经提交的数据， 但是其**违反了数据库事务一致性**的要求。
- 一般来说， 不可重复读的问题是可以接受的， 因为其读到的是已经提交的数据， 本身并不会带来很大的问题。 因此， 很多数据库厂商（如Oracle、 Microsoft SQL Server） 将其数据库事务的默认隔离级别设置为READ COMMITTED， 在这种隔离级别下允许不可重复读的现象。
- 在InnoDB存储引擎中， 通过使用**Next-Key Lock**算法来避免不可重复读的问题。 在NextKey Lock算法下， 对于索引的扫描， 不仅是锁住扫描到的索引， 而且还锁住这些索引覆盖的范围（gap） 。 因此在这个范围内的插入都是不允许的。 这样就避免了另外的事务在这个范围内插入数据导致的不可重复读的问题。 因此， InnoDB存储引擎的默认事务隔离级别是READ REPEATABLE， 采用Next-Key Lock算法， 避免了不可重复读的现象。

#### 不可重复读示例
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/61438726.jpg)

### 丢失更新
- 丢失更新是另一个锁导致的问题，简单来说其就是一个事务的更新操作会被另一个事
务的更新操作所**覆盖**，从而导致数据的不一致。
- 要避免丢失更新， 需要让事务在这种情况下的操作变成**串行化**。

### 总结--**仅供参考**
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/33516594.jpg)

## 阻塞
- 在默认情况下InnoDB存储引擎不会回滚超时引发的错误异常。其实InnoDB存储引擎在大部分情况下都不会对异常进行回滚。

## 死锁
### 死锁的概念及解决
- 解决死锁问题最简单的一种方法是超时，即当两个事务互相等待时，当一个等待时间超过设置的某一阈值时，其中一个事务进行回滚，另一个等待的事务就能继续进行.
- 当前数据库还都普遍采用wait-for graph（等待图）的方式来进行死锁检测：
 - 锁的信息链表
 - 事务等待链表

### 死锁概率
- 事务发生死锁的概率与以下几点因素有关：
 - 系统中事务的数量（n） ， 数量越多发生死锁的概率越大。
 - 每个事务操作的数量（r） ， 每个事务操作的数量越多， 发生死锁的概率越大。
 - 操作数据的集合（R） ， 越小则发生死锁的概率越大。

### 死锁示例
![enter image description here](http://hellomrrs-imgs.oss-cn-shanghai.aliyuncs.com/18-11-14/71303703.jpg)

## 锁升级
- 锁升级（Lock Escalation）是指将当前锁的粒度降低(减少开销)
- InnoDB存储引擎不存在锁升级的问题。因为其不是根据每个记录来产生行锁的，相反，其根据每个事务访问的每个页对锁进行管理的，采用的是位图的方式。因此不管一个事务锁住页中一个记录还是多个记录，其开销通常都是一致的。

