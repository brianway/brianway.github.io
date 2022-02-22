
---
layout: post
title:  MySQL原理学习-事务隔离
date:   2022-02-22 23:43:11 +08:00
category: 原理学习
tags: [MySQL]
comments: true
---

在开发过程中经常会使用到 MySQL，也可能多少了解事务隔离级别、脏读、幻读等名词，但总觉得不够系统和深入，希望对事务隔离相关知识进行一个梳理。本文先简单介绍 MySQL 事务相关的概念，重点谈论事务的隔离性及其实现方式，最后结合这些原理举了两个避坑实践案例。

<!-- more -->

## 事务介绍


在 MySQL 中，事务支持是在**引擎层**实现的。提到事务，绕不开它的四个特性--**ACID**（Atomicity、Consistency、Isolation、Durability，即原子性、一致性、隔离性、持久性）。本文重点谈论事务的**隔离性**及其实现方式。
​

### 事务隔离级别


SQL 标准义的事务隔离级别包括：
- **读未提交（read-uncommitted）**
- **读提交（read-committed）**
- **可重复读（repeatable-read）**
- **串行化（serializable ）**

上述每一种级别都规定了一个事务中的修改，哪些是事务之间可见的，哪些是不可见的。
InnoDB中事务的默认隔离级别是可重复读的（REPEATABLE-READ），而公司里一般会将事务默认隔离级别设置为读提交（READ-COMMITTED）


MySQL中查看和修改事务隔离级别的常用命令如下：
```
show variables like 'transaction_isolation';

// 补充：修改隔离级别 set session|global  transaction_isolation = '隔离级别名';
// 注：实践得知，set global变量值不会影响已有session的变量值，仅会影响新建立的session的变量值，（global变量值本身是全局修改即可见的）
set global transaction_isolation ='read-committed';
show global variables like '%isolation%';
show session variables like '%isolation%';

// 当前执行中的事务
select * from information_schema.innodb_trx;
```


### 隔离级别与读现象的关系
不同隔离级别下，可能出现的读现象(read phenomena)如下：

| **隔离级别** | **脏读（Dirty Read）** | **不可重复读（NonRepeatable Read）** | **幻读（Phantom Read）** |
| --- | --- | --- | --- |
| 未提交读（Read uncommitted） | 可能 | 可能 | 可能 |
| 已提交读（Read committed） | 不可能 | 可能 | 可能 |
| 可重复读（Repeatable read） | 不可能 | 不可能 | 可能 |
| 可串行化（Serializable ） | 不可能 | 不可能 | 不可能 |



相关概念见维基百科: https://en.wikipedia.org/wiki/Isolation_(database_systems)#Read_phenomena


> 说明：InnoDB实现的可重复读通过**next-key lock机制**避免了幻读现象。next-key lock是行锁的一种，实现相当于record lock(记录锁) + gap lock(间隙锁)。关于InnoDB锁机制的相关问题，本文不作展开。


### 事务启动方式
MySQL 的事务启动方式：
- （建议）显式启动事务语句， `begin` 或 `start transaction`。`commit work and chain`语法可以不用每次事务开始时`begin`
- （不建议） `set autocommit=0`，这个命令会将这个线程的自动提交关掉。

```
# 关于MySQL中的commit work、commit work and chain 参考 https://juejin.cn/post/6987373561836994590#heading-7
show variables like 'completion_type';
```


### 一个例子
下面通过一个例子展示不同隔离级别。
表格中，事务A和B两列对应的内容分别表示在这两个事务中执行的SQL，每一行表示SQL执行的时间顺序。

| 事务A | 事务B |
| :--- | :--- |
| 启动事务; 查询得到值1 | 启动事务 |
|  | 查询得到值1 |
|  | 将1改成2 |
| 查询得到V1 |  |
|  | 提交事务B |
| 查询得到值V2 |  |
| 提交事务A |  |
| 查询得到值V3 |  |



不同隔离级别下，V1、V2、V3的值：

- 若隔离级别是“读未提交”， 则 V1 的值就是 2。这时候事务 B 虽然还没有提交，但是结果已经被 A 看到了。因此，V2、V3 也都是 2。
- 若隔离级别是“读提交”，则 V1 是 1，V2 的值是 2。事务 B 的更新在提交后才能被 A 看到。所以， V3 的值也是 2。
- 若隔离级别是“可重复读”，则 V1、V2 是 1，V3 是 2。之所以 V2 还是 1，遵循的就是这个要求：事务在执行期间看到的数据前后必须是一致的。
- 若隔离级别是“串行化”，则在事务 B 执行“将 1 改成 2”的时候，会被锁住。直到事务 A 提交后，事务 B 才可以继续执行。所以从 A 的角度看， V1、V2 值是 1，V3 的值是 2。



## 隔离性实现- MVCC
从上面的例子可以看出，对于同一段SQL和相同的执行顺序，设置为不同的隔离级别时，事务之间的相互影响是不同的，那到底是如何实现的呢？
显然地，不同事务的读操作之间并不会相互影响，所以只需要考虑一个事务的写操作对其他事务的影响：

-  (一个事务)写操作对(另一个事务)读操作的影响
-  (一个事务)写操作对(另一个事务)写操作的影响

其中，前者是通过**MVCC** (MultiVersion Concurrency Control，即多版本并发控制) 保证隔离性；后者是通过**锁机制**保证隔离性。

### 多版本是如何实现的
**MVCC** (MultiVersion Concurrency Control，即多版本并发控制) 的思想就是保存数据的历史版本，通过对数据行的**多个版本管理**来实现数据库的并发控制。

在介绍InnoDB的MVCC实现之前，我们可以先自己设计实现一下多版本实现方式。显然，很容易想到以下两种思路：

1. 每次记录全量： 初始版本 ，历史版本1， 历史版本2, ... ，历史版本n ，最新版本
2. 每次记录增量： 初始版本 + ∑每次增量 = 最新版本；历史某个版本+ ∑若干增量 =  最新版本。典型的应用如git，文档编辑时的撤销、恢复，等等。

> _注：∑是数学中的求和符号，在这里引申一下表示“累计、叠加”的含义。_

InnoDB的MVCC原理和记录增量类似， 历史某个版本 =  最新版本 - ∑若干增量 。而**undo log**（回滚日志，用于记录数据**被修改前**的信息）里记录的就是这个“若干增量”。


快照数据是将 undo log 的日志操作按一定的规则应用到“当前数据”上，即**当前值通过“回滚”回溯到某个历史值**。从而达到同一条记录在系统中可以存在多个版本，这就是数据库的多版本并发控制（MVCC）。
换成数学方式表达就是：**快照数据  = func(当前数据，undo log)**

**每行数据有多个版本**。从前面的分析可知，数据的多个版本但并不是物理真是存在的，而是基于某个版本和undo log，可以计算出之前版本的数据，从而达到“旧的数据版本被保留”的效果。
**InnoDB里每个事务有个唯一的事务 ID**，叫作 transaction id，是在事务开始的时候向 InnoDB 的事务系统申请的，是**按申请顺序严格递增**的。
每次事务更新数据的时候，都会生成一个新的数据版本，并且把 transaction id 赋值给这个数据版本的事务 ID，记为 row trx_id。所以InnoDB中历史版本的数据结构示意是下面的样子：
```sql
// 记号 （v, row trx_id, k） 表示某个数据行的一个快照版本
// v表示虚拟的版本号，用于方便描述；k表示某个数据列的值，row trx_id表示数据版本的事务ID。
// <--- 后面表示生成该快照版本的的SQL逻辑，transaction id表示执行这个SQL的事务ID。
// 则一行数据在执行的三段SQL前后后的四个数据快照V1～V4的数据结构示意如下：

(V1, row trx_id =10, k=1) 
          ^
          | 指向                   
(V2, row trx_id =50, k=10)   <---  transaction id=50, set k=10
          ^
          | 指向                    
(V3, row trx_id =32, k=11)   <---  transaction id=32, set k=k+1
          ^
          | 指向                    
(V4, row trx_id =44, k=110)  <---  transaction id=44, set k=k*10
```
> 注：历史版本从数据结构的角度看是个链表，图中链表节点的row trx_id大小不一定是单调递增/递减的，而是**按照事务提交的顺序**。上述链接节点间的指针就是通过undo log实现的。

### InnoDB 的 MVCC 是如何实现的
InnoDB 在实现 MVCC 时用到了**一致性读视图**，即 **consistent read view** 。访问的时候以视图的逻辑结果为准：

- “读未提交”隔离级别下，直接返回记录上的最新值，没有视图概念
- “读提交”隔离级别下，视图是在每个 SQL 语句开始执行的时候创建的
- “可重复读”隔离级别下，视图是在第一个SQL启动时创建的，整个事务存在期间都用这个视图
- “串行化”隔离级别下，直接用加锁的方式来避免并行访问

> PS: 这个“视图”概念区别于MySQL中另一个常见的“视图”——用查询语句定义的一个虚拟表，在调用的时候执行查询语句并生成结果

关于一致性视图的创建时机：

- `begin`/`start transaction`**并不马上启动事务**。一致性视图是在执行**第一个快照读**语句时创建的。
- `start transaction with consistent snapshot`，马上启动一个事务。一致性视图是在执行 `start transaction with consistent snapshot`时创建的。

> 参考：MySQL 8.0 Reference Manual - START TRANSACTION, COMMIT, and ROLLBACK Statements: [ https://dev.mysql.com/doc/refman/8.0/en/commit.html](https://dev.mysql.com/doc/refman/8.0/en/)


可重复读 vs. 读提交：

- **创建视图的时机不同**​
   - 在可重复读隔离级别下，只需要**在事务开始的时候**创建一致性视图，之后事务里的其他查询都共用这个一致性视图；
   - 在读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图。
- 进而表现为**可见的数据版本不同**
   - 对于可重复读，查询只承认在**事务启动前**就已经提交完成的数据；
   - 对于读提交，查询只承认在**语句启动前**就已经提交完成的数据。



### 视图数据如何计算
前面提到了，**快照数据  = func(当前数据，undo log)**，而一致性视图就是由这些快照数据中的一部分组成的。
​
一个数据版本，对于一个事务视图来说，除了自己的更新总是可见以外，有三种情况：

- 版本未提交，不可见；
- 版本已提交，但是是在视图创建后提交的，不可见；
- 版本已提交，而且是在视图创建前提交的，可见。

这个逻辑很简单，但假如通过代码来实现时会发现没法落地。例如，如何确定版本是否提交，以及在视图创建前还是创建后提交的？也就是说如果通过合适的func，使得计算出的快照数据就是当前事务的一致性视图呢？


对于当前事务，数据版本的可见性计算规则如下（PS：逻辑有点绕，但推导很简单）：
前提：

- InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”（指**启动了但还没提交**）的所有事务 ID。
- 假如将**事务启动瞬间**，当前正在“活跃”（指**启动了但还没提交**）的所有事务 ID对应的数组记作 *active_trxid_arr*；
- *min(active_trxid_arr)* 表示数组 *active_trxid_arr* 的最小值；
- *max(系统创建过的事务 ID) *表示系统创建过的事务 ID的最大值。显然地，*min(active_trxid_arr)  <=  max(系统创建过的事务 ID)*​

对于任意数据版本的 row trx_id ，按照按 *min(active_trxid_arr)* 和 *max(系统创建过的事务 ID)* 划分的范围区间作分类讨论：

- *row trx_id < **_**min(active_trxid_arr)*，表示这个版本是已提交的事务或者是当前事务自己生成的，可见
- *row trx_id > **_**max(系统创建过的事务 ID)*，表示这个版本是由将来启动的事务生成的（即该事务启动时，该row trx_id 对应的事务还未启动），不可见
- *row trx_id >= min(active_trxid_arr)  &&  row trx_id  <= max(系统创建过的事务 ID)*，分两种情况：
   - row trx_id 在 active_trxid_arr 中，表示这个版本是由还没提交的事务生成的，不可见
   - row trx_id 不在 active_trxid_arr 中，表示这个版本是已经提交了的事务生成的，可见


## 隔离性实现- 锁机制
讨论完了事务一个事务写操作对另一个事务读操作的影响，下面看看一个事务写操作对另一个事务写操作的影响。
​
先来看一个事务并发更新的例子：

```sql
mysql> CREATE TABLE `t` ( 
  `id` int(11) NOT NULL, 
  `k` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
insert into t(id, k) values(1,1),(2,2);
```
| 事务A | 事务B |
| --- | --- |
| start transaction with consistent snapshot; | ​
 |
|  | start transaction with consistent snapshot;<br/>update set k = k+1 where id = 1; |
| update t set k=k+1 where id = 1;<br/> select k from t where id = 1; |  |
|  | commit; |
| commit; |  |


可以看出，如果这里还是使用“一致性视图”，那肯定会有问题： 在事务B执行commit语句前，事务A也对id=1这一行进行了修改，假如使用“一致性视图”，则事务A是看不见事务B对id=1这一行的修改的，事务B执行commit后，事务A紧接着commit会覆盖掉事务B的修改 ，即事务B的更新会丢失。
​
实际上事务在更新数据时，使用的是“**当前读”**（**current read**），而当“当前读”是会加锁的，从而避免其他事务同时更新这部分数据。
更新数据都是先读后写的，而这个读，只能读当前的值，称为“**当前读”**（**current read**）。“当前读”总是读取已经提交完成的最新版本。

一致性读 vs. 当前读：

- 一致性读，又称快照读，一致性读会根据 row trx_id 和一致性视图确定数据版本的可见性。（读取的是undo log中已提交的数据，可能是数据的历史版本，no-locking，所以是非阻塞的读取操作。）
- 当前读，读取的是数据的**已提交完成**的最新版本, 加锁保证事务隔离性。​

**可重复读的核心就是一致性读（consistent read）；而事务更新数据的时候，只能用当前读。**


## 实践避坑
### 避免长事务

**尽量不要使用长事务**：从前面MVCC的机制可以看出，事务的一致性视图是基于当前数据配合回滚日志(undo log)得到的，如果事务一直不关闭或提交，则这些回滚日志就会一直保留，会占用存储空间。
​当没有事务再需要用到这些回滚日志时，回滚日志会被删除，从这角度可以推理出，回滚日志占用存储是应尽量避免长事务原因之一。
​
可以通过下面的SQL查询长事务

```sql
// 查询长事务，例如超过60秒的
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60;
```

### 应用程序正确使用“乐观锁”
​后端开发时，有时会用到“乐观锁”，大概使用方式就是基于version字段对数据行row进行CAS式的更新，类似`update t set ... where id = 111 and version = xxx`。
**当事务隔离级别均为为“可重复读”时**，如果其他事务抢先更新了version字段，当前事务是有可能出现明明能查询到对应version的数据，执行update语句也能顺利执行，但就是数据无法更新的情况。所以**判断是否成功的标准是 affected_rows 是不是等于预期值。**
​
可以试着运行下面这个例子印证该结论（事务隔离级别均为为“可重复读”）：
```sql
mysql> CREATE TABLE `t` ( 
  `id` int(11) NOT NULL, 
  `k` int(11) DEFAULT NULL,
  `version` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
insert into t(id, k, version) values(1,1,1),(2,2,2);
```

场景1:

| 事务A | 事务B |
| --- | --- |
| begin;<br>select * from t; |  |
|  | update t set k=30, version=version+1 where id = 1 and version = 1; |
| update t set k=20, version=version+1 where id = 1 and version = 1;<br>select * from t;<br>commit; |  |


场景2:

| 事务A | 事务B |
| --- | --- |
|  | begin;<br>select * from t; |
| begin;<br>select * from t; |  |
|  | update t set k=30, version=version+1 where id = 1 and version = 1;<br>commit; |
| update t set k=20, version=version+1 where id = 1 and version = 1;<br>select * from t;<br>commit; |  |







## 相关链接


- 理解事务 - MySQL 事务处理机制  [https://www.jianshu.com/p/bcc614524024](https://www.jianshu.com/p/bcc614524024)
- 『浅入深出』MySQL 中事务的实现 [https://draveness.me/mysql-transaction](https://draveness.me/mysql-transaction)
- 『浅入浅出』MySQL 和 InnoDB   [https://draveness.me/mysql-innodb](https://draveness.me/mysql-innodb)
- MySQL 四种事务隔离级的说明  [https://www.cnblogs.com/zhoujinyi/p/3437475.html](https://www.cnblogs.com/zhoujinyi/p/3437475.html)
- Innodb中的事务隔离级别和锁的关系  [https://tech.meituan.com/2014/08/20/innodb-lock.html](https://tech.meituan.com/2014/08/20/innodb-lock.html)
- 图文并茂讲解Mysql事务实现原理 [https://cloud.tencent.com/developer/article/1431307](https://cloud.tencent.com/developer/article/1431307)
- 深入学习MySQL事务：ACID特性的实现原理 [https://cloud.tencent.com/developer/article/1408793](https://cloud.tencent.com/developer/article/1408793)



