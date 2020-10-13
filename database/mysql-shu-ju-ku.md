---
description: 关于mysql数据知识笔记
---

# mysql数据库

### 简述一下mysql三范式以及反模式

**1.第一范式**

第一范式强调字段的原子性，表示字段不可再次细分，比如我们存储一个人的地址，直接用一个字段存储那么就是xx省xx市xx地区xx，显然这个字段可以再度细分为省市区以及到街道，分别用四个字段存储。

**2.第二范式**

第二范式强调记录的唯一性约束，每个表必须有一个主键，且这个表的其他列必须完全依赖于主键，而不能只依赖于主键的一部分。比如一个版本记录表，该表包含（版本号，版本名称，产品名称，产品编码），其中版本号和产品编码为唯一主键，那么产品名称就是只依赖于主键的一部分，只和产品编码有关系与版本号并无关联。

**2.第三范式**

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。比如现在很多表都出现冗余字段，其实冗余字段就是违反第三范式的。

### mysql一条select查询语句的执行过程。

 首先mysql客户端执行一条sql需要连接到服务端，所以先要进行连接，连接到服务端之后传入sql，服务端接受到sql执行进行sql分析，这个时候会经过sql分析器，分析完成之后会进行sql优化，sql查询需要使用哪个索引在这个阶段进行分析，随后会直接进入执行器进行sql执行工作。

> 这个里面存一个阶段是查询缓存，但是查询缓存不是必定存在的，可以通过query_cache_type参数去指定是否需要查询缓存，如果设定为demand，那么所有的查询都不会去查询缓存，并且mysql8.0直接查询缓存整块功能去除了，主要是更新频繁的表查询缓存命中率不是很可观而且缓存数据还需要浪费系统资源。


### mysql一条update更新语句的执行过程。

首先mysql的更新和select有什么区别呢，比如目前更新一条数据为upadte x set a = 1，mysql会先查询出a=1的这条记录，查询这个过程和select流程一样，更新过程涉及两个日志文件，一个是redo log和binlog，其中redo log是innodb引擎特有的，其他的mysql引擎并没有这个日志，更新操作会先将更新内容写入到redo log中，这个时候redo log状态为prepare，随后将更新内容写入到binlog中，提交事务并将之前的redo log状态改为commit。

### mysql事务隔离级别

- 读未提交：一个事务未提交的时候，它做的任何操作都可以被其他事务看到。
- 读提交：一个事务只有提交了，其他事务才能看到变更的数据。
- 可重复读：一个事务从开启到结束，看到的数据是一致的，像是一种快照的概念，快照的节点在事务开启的时候。
- 串行化：串行化针对的是同一行数据，它是读写互斥的，读写事务冲突的时候，后一个事务必须等待前一个事务完成才能继续执行。


### 事务隔离如何实现
事务隔离级别有四个，其中串行化以及读未提交不进行讨论，因为前者是相当于单线程访问，性能最低，后者完全不加任何限制，可以读到任何数据也没有隔离的实现。我们具体分析一下读提交和可重复读。<br>
innodb的行数据有多个版本，每个版本都记录的一个row trx_id 这个id就是事务id，每个事务对行进行改变后都会把相应的事务id存下来，mysql在每个事务启动的时候都会去申请一个事务id，这个事务id是递增的，每个事务申请的时候都会维护一个动态的事务数组，这个动态的数组用来存储当前活跃的事务id（开启了事务，但是事务还未提交，包含自己），这个数组里的事务id最小值为低水位，事务id最大值为高水位。mysql在读取行数据的row trx_id则有以下几种情况。
1. trx_id在低水位以下，那么表示该数据是可读的。
2. trx_id在地位位和高水位之间，这个时候就需要判断该trx_id是否在活跃数组中，如果在则不能看到，如果不在则表示是其他事务且已提交的数据可读。
3. trx_id在高水位之上，表示是未来的事务，则对应的行数据不可读。

可重复读是通过事务创建时候维护的动态事务数组以及mysql行数据存储的row trx_id来实现的。<br>
读提交和可重复读不同的是可重复读是开启事务前提交的数据可见，但是读提交是语句启动的时候提交的数据可见

### mysql脏读、不可重复读、幻读

**脏读**

脏读是指某个事务读取到了另外一个事务未提交的数据，出现这种情况被称之为脏读，在RC（读已提交）、RR（可重复读）的事务隔离级别下可防止脏读。

**不可重复读**

不可重复读是指某个事务多次执行相同的查询语句，前后两次查询所得到的数据是不一致的，这种情况表示不可重复读，RC的事务隔离级别下会出现不可重复读情况，但是RR事务隔离级别下不会出现。

**幻读**

幻读是指某个事务读取到了某个事务新插入的行，比如A事务在时间点1查询的结果为两条数据，但是在事务未提交前又在时间点2查询了一次，两次查询的结果不一致，后面查询多出了一部分数据，这种情况称之为幻读。

>在事务隔离级别为RR的时候，普通的查询都是快照读，所以不会出现幻读，在RR事务隔离级别下只有当前读才会出现幻读情况，且幻读仅仅值新增的行，对于被修改的行被当前读读到不能成为幻读。mysql解决幻读用的是Next-Key Lock锁和间隙锁。

### mysql全局锁、表锁、行级锁

**全局锁**

mysql提供了一个加全局锁的方法，就是FTWRL，flush tables write read lock，这个命令会让整个库处于只读状态，使用这个命令之后其他的更新操作都将被阻塞，包括数据的更新，表结构的更新。

**表锁**

表级别的锁存在两种，一种是表锁，使用lock tables 表名... read/write来对表进行读写锁，还有一个是元数据锁（meta data lock）MDL，MDL是不需要显示使用的，每个客户端在访问表的时候会自动加上，MDL也分为MDL读锁、MDL写锁。

MDL读锁：该锁是客户端对表进行增删改查的时候就会隐式加上MDL读锁。<br>
MDL写锁：该锁是客户端对表进行结构上改变的时候加上，比如加字段删字段该字段等等。

> 生产上使用alter命令的时候建议加上超时时间，以免alter等待MDL写锁时间过长从而影响了其他业务的正产读操作。

**行级锁**

innodb引擎有三种行锁的算法，分别是：
Record Lock：单个记录的行锁。<br>
Gap Lock：间隙锁，锁定一个范围，但不包含本身。<br>
Next-Key Lock：Gap Lock+Record Lock，锁定一个范围，并且锁定记录本身。<br>

**加锁规则**

1. mysql加锁的基本单位都是Next-Key Lock，而且Next-Key Lock是左开右闭的原则。
2. 查找过程中对访问到的对象才会加锁。
3. 给唯一索引加锁的时候会退化为Record Lock行锁。
4. 索引上等值查询，向右遍历最后一个不满足等值条件的时候，Next-Key Lock退化为间隙锁。
5. 索引上范围查询会访问到不满足条件的第一个值为止。

**案例**

DDL
```
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `c` int(11) DEFAULT NULL,
  `d` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `c` (`c`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='测试';
```
1.唯一索引的等值查询
```
BEGIN;
SELECT * from t WHERE id = 7 FOR UPDATE;
SELECT SLEEP(6);
COMMIT;

BEGIN;
INSERT INTO t VALUES(8,8,8);
COMMIT;
<!-- blocked -->


BEGIN;
UPDATE t set d = d + 1 where id = 10;
COMMIT;
<!-- 正常执行 -->
```
id为主键，但是id=7这个条件本身在5-10之间，根据原则1会对(5,10]加上Next-Key Lock锁，但是10并不满足7，所以最后会退化为间隙锁，最终加上的锁为(5,10)间隙锁。

2.非唯一索引等值查询
```
BEGIN;
SELECT * from t WHERE c = 5 FOR UPDATE;
SELECT SLEEP(6);
COMMIT;

BEGIN;
INSERT INTO t VALUES(8,8,8);
COMMIT;
<!-- blocked -->

BEGIN;
UPDATE t set d = d + 1 where c = 10;
COMMIT;
<!-- 正常执行 -->
```
c是普通索引，首选c=5会扫描到0-5这个范围，所以第一个next-key Lock会加在(0,5]上，因为索引上的等值查询需要向右遍历到最后一个不满足条件为止，所以也会顺带加上(10,15]next-key Lock，因为15并不满足条件，最终退化为间隙锁。

3.唯一索引左闭右开范围查询
```
BEGIN;
SELECT * from t WHERE id >= 5 and id < 6 FOR UPDATE;
SELECT SLEEP(6);
COMMIT;

BEGIN;
INSERT INTO t VALUES(8,8,8);
COMMIT;
<!-- blocked -->

BEGIN;
UPDATE t set d = d + 1 where id = 10;
COMMIT;
<!-- blocked -->
```
id为主键，首先访问到的是0-5这个范围，因为id为唯一索引，所以会给值等于5加上行锁，然后大于5小于6，会继续访问下一个范围5-10，所以最终会加一个行锁和一个(5,10]的Next-Key Lock锁。

4.普通索引的范围查询
```
BEGIN;
SELECT * from t WHERE c >= 5 and c < 6 FOR UPDATE;
SELECT SLEEP(6);
COMMIT;

BEGIN;
INSERT INTO t VALUES(2,2,2);
COMMIT;
<!-- blocked -->

BEGIN;
UPDATE t set d = d + 1 where id = 10;
COMMIT;
<!-- blocked -->
```
c为普通索引，首先访问到的是0-5这个范围，所以会对0-5加上(0,5]加上Next-Key Lock锁，然后大于5小于6，会继续访问下一个范围5-10，所以最终会加一个行锁和一个(5,10]的Next-Key Lock锁。

5.唯一索引左开右闭范围查询
```
BEGIN;
SELECT * from t WHERE id > 5 and id <= 10 FOR UPDATE;
SELECT SLEEP(6);
COMMIT;

BEGIN;
INSERT INTO t VALUES(12,12,12);
COMMIT;
<!-- blocked -->

BEGIN;
UPDATE t set d = d + 1 where id = 15;
COMMIT;
<!-- blocked -->
```
id为主键，首先访问到的是5-10这个范围，所以会给(5,10]加上Next-Key Lock锁，然后理论上等于10应该会降为行级锁，但是mysql还是会继续访问下一个范围10-15，所以最终会加一个行锁和一个(10,15]的Next-Key Lock锁。


### mysql三种日志redo、undo、binlog

**redo log**

redo log也被称为重做日志，redo log分为两个部分一个是redo log buffer内存中的重做日志，这个是易丢失的，还有一个是重做日志文件，这个是在磁盘上的，持久不会丢失。与binlog不同的是redo log是物理日志，主要是记录了某个数据页上做了什么修改，而binlog记录的是这个语句的原始逻辑，redo log是innodb引擎特有的日志，而且redo log是固定大小，写完之后需要将redo log中的一部分数据持久化到磁盘中，

**undo log**

undo log被称为回滚日志，保存了事务发生之前的数据版本，主要是用来做数据回滚操作。

**binlog**

用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步，逻辑格式的日志，可以简单认为就是执行过的事务中的sql语句。回复速度相比redo log会慢很多，因为redo log存储的是物理日志。

### 唯一索引和普通索引该怎么选择，性能上有什么区别？

查询性能分析，如果存在一张表t，字段为abc，a为主键、b为索引，c普通字段，如果执行以下sql语句
```
select * from t where b = 1;
```
如果b是唯一索引，那么当查询到1的时候就会直接返回，而如果b是普通索引的话，mysql需要继续遍历索引页直到下一个不满足条件为止，在查询上可以看出唯一索引是有select上的性能优势的，但是这种优势微乎其微，并不是非常明显。

更新性能分析，表还是上述的t，如果这个时候我执行以下sql语句
```
update set b = 2 where b = 1;
```
**该记录在内存中**

1.唯一索引需要进行判断，是否在b=1冲突情况，没有冲突直接进行更新，并写入redo log相应的数据页修改操作。<br>
2.普通索引直接进行数据更新，并写入redo log数据页修改操作。<br>

**该记录不在内存中**
 
1.因为是唯一索引，所以需要判断是否存在冲突情况，这个时候就需要去数据表中读取相应的数据页到内存中，之后进行判断，如果没有冲突进行更新，并写入redo log相应的数据页修改操作。<br>
2.普通索引直接将更新操作记录到change buffer中，并写入redo log数据页修改操作。<br>

>从分析来看 唯一索引和普通索引在查询上性能相差不大，但是更新上如果数据页不在内存中唯一索引要比普通索引多一次随机io的访问，在更新上普通索引性能要高很多。

**注意点**

虽然change buffer能够带来普通索引上的更新性能优化，但是如果数据被更新之后立马需要进行查询的话，这个时候一样的会进行数据io访问，因为change buffer中只存了数据的更新操作，并没有相应的数据，所以这次查询需要从数据表中读取数据，并结合change buffer的修改得到正确数据后进行返回。


### mysql如何保证数据不丢失？

要了解到mysql如何保证数据不丢失，我们需要了解到两个日志，一个是binlog，一个是redo log。

**binlog写入特性**

binlog的写入顺序为，binlog cache -> page cache -> 磁盘binlog文件，这里的binlog cache每个线程都有一片内存专门用于binlog数据的存储，这个存储只是短暂的，写入很快，事务提交的时候会将binlog中的数据写入到page cache中，并且清空binlog cache中的数据，page cache写入到磁盘这一步比较慢，因为这一步涉及到了io，这一步也是受参数控制的。

> sync_binlog=0的时候，表示每次都只write（写入page cache），不进行fsync（也就是写入磁盘）。
> sync_binlog=1的时候，每次提交事务的时候都会fsync。
> sync_binlog=N(N>0)的时候，表示每次提交事务都会write，但是fsync会累计N个事务后才会触发。

**redo log写入特性**

redo log写入顺序为，redo log buffer -> page cache -> 磁盘redo log，同样的写入规则也受参数控制，innodb_flush_log_at_trx_commit这个参数。

>innodb_flush_log_at_trx_commit=0 每次事务提交的时候只是把redo log留在redo log buffer中。
>innodb_flush_log_at_trx_commit=1 每次事务提交的时候都将redo log直接持久化到磁盘。
>innodb_flush_log_at_trx_commit=2 每次事务提交时只是把redo log写入到page cache中。

>innodb后台有一个线程，每隔一秒就会把redo log buffer中的日志写入到 page cache中，之后会直接持久化到磁盘中。
>redo log buffer 占用的空间即将达到 innodb_log_buffer_size 一半的时候，后台线程会主动写盘。
>并行的事务提交的时候，顺带将这个事务的 redo log buffer 持久化到磁盘。

通过上述两个日志写入规则我们可以知道，mysql保证数据不丢失，必须先将sync_binlog、innodb_flush_log_at_trx_commit设置为1，每次事务提交都将日志强制写入磁盘中便可以保证数据不丢失，断电重启也能通过这两个日志进行数据修复。


### mysql如何保证主从一致性？

mysql的主从同步是基于binlog去实现的，主服务器会运行一个log dump thread，从服务器会运行两个线程（I/O thread SQL thread），当从服务器连接到主服务器的时候，主服务器会创建一个log dump线程，这个线程专门用来发送binlog内容，而从服务器在执行start salve命令后，从服务器会使用I/O线程连接主节点，用来接收binlog内容，接收到的binlog保存在本地的relay-log中，之后从服务器通过SQL thread读取binlog解析为具体的操作并执行。


### 主从架构下的读写分离都有什么问题？

1.读写分离在主从复制出现大量延迟的时候，会出现数据不一致性从而影响业务功能。

> 解决方案：
> 1.从库查询的时候每次可以先执行以下show slave status，查看seconds_behind_master是否为0，0表示主从同步延迟为0s，如果不为0的情况下强制走主库查询
> 2.对比主从的GTID，GTID表示全局事务id，是递增的，通过这个判断主从是否有延迟，如果有延迟强制走主库查询。
> 3.设置主从同步为同步方式，即每次事务提交都将事务同步给从库，同步成功则进行事务提交，同步失败事务进行回滚，但是这个方式会大大降低事务提交性能，从而降低tps。


### 在线进行大表DDL有什么坑？


