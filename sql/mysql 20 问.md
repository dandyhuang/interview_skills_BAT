### mysql的日志有哪些

MySQL的日志主要包括以下几种：

二进制日志（binary log）：记录基于语句或行级别的所有数据更改，可用于主从复制和可恢复性的场景。

事务日志/重做日志（transaction log/redo log）：记录事务在数据库中所做的修改，如插入、更新和删除操作，用于保证数据库的ACID特性和恢复性。

慢查询日志（slow query log）：记录执行时间超过阈值的SQL语句，通常用于优化查询性能。

错误日志（error log）：记录MySQL服务器的错误、警告和通知信息。

查询日志（general query log）：记录所有经过服务器的SQL查询。

撤销日志/回滚日志（undo log）：记录数据更新前的值，用于回滚操作。

中继日志（relay log）：用于主从复制中，记录主库的二进制日志内容，以及从库在主库上已经执行过的操作。

GTID日志（global transaction identifier log）：MySQL5.6版本后引入，用于提供全局唯一的事务标识符，并跟踪主从同步的位置。

### 1 redolog、 undolog、binlog区别

首先我们先理解WAL技术，即先写日志，在写磁盘。这里的写日志原因就是这三大日志都是顺序写。我们可以打开datadir中，ib_logfile0、ib_logfile1写redolog， mysql-bin.000001写binlog，ibdata1写undolog使用。而磁盘数据对应每个库里头，如test库里头，db.opt  test.frm  test.ibd三个部分组成存储。其中*.frm文件存储的是表空间结构数据，*.idb存储的是索引及用户数据(每张表内的数据)。这里就会涉及到数据随机存储的问题。详细可以[查看](https://blog.csdn.net/mashaokang1314/article/details/109716569)，一定要看收获会很大

> 逻辑日志：类似Binlog的statement格式，**可以简单理解为记录的就是sql语句**。
>
> 物理日志：类似Binlog的RAW格式，**物理日志记录的是数据页变更**
>
> 如：更新操作作用于 Page42，将字段 “Kemera” 修改为 “camera”。更新操作对应的日志为
>
> ```
> "Page 42:image at 367,2; before:'ke';after:'ca'"
> ```

**物理日志占用空间多，但不需要依赖原page的内容。逻辑日志比较简洁而且占用的空间要小，缺点就是需要依赖原page内容，而且会有部分执行和操作一致性的问题。**

mysql脏页指的是：当内存数据页和磁盘数据页上的内容不一致时，我们称这个内存页为脏页；

#### redolog

为什么需要redo log

- 数据库宕机，正常提交的事物数据丢失的
- innodb以页为单位，如果每次修改只修改几个字节，每次读盘，刷盘代价太大
- 如果需要修改多个页上的数据，随机io读写性能太差

#### redo log基本概念

`redo log`包括两部分：一个是内存中的日志缓冲(`redo log buffer`)，另一个是磁盘上的日志文件(`redo log file`)。

logbuffer主要由double write buffer和insert buffer组成。`Double Write` 就是为了在数据库崩溃恢复时保证数据不丢失的一个重要特性，保证了数据的可靠性。`Change Buffer` 是用来提高存储引擎性能上的提升。

redo log 主要由固定大小文件，循环写入。write pos是当前记录的位置，checkpoint是当前要擦除的位置。如果write pos追上checkpoint，表示 redo log 满了

##### Double Write

Innodb数据页大小默认为16KB，而文件系统一页大小为4KB。刷脏页数据4次io，可能会刷新到第二次，数据库就宕机了。

因为redo是物理逻辑结合型的日志。物理到具体的哪个page，页内操作是逻辑的。这种方式既实现了物理日志带来的幂等性（以物理页为整体），又拥有逻辑日志带来的轻量性（物理页内修改是逻辑日志）。先以Page为单位记录日志，在每个Page里面再采用逻辑记法。 

总结：思路和写redolog是一样的，都是顺序写数据。现在是因为系统页和mysql数据页不能满足原子性的写入，所以就在按照page为单位，在每个page页在按照逻辑日志来记录到磁盘数据。

#### Change Buffer

写缓存(Change Buffer)在5.5之前叫做 插入缓存(insert Buffer)。如果业务插入数据时是按照主键递增的，所以插入聚集索引一般是顺序磁盘写入。但是不可能每张表都只有聚集索引，当存在非聚集索引时，对于非聚集索引的变更就可能不是顺序的，会拖慢整体的插入性能。为了解决这一问题，InnoDB 使用了 insert buffer 机制，将对于非聚集索引的变更先放入 insert buffer ，尽量合并一些数据页后再写入实际的非聚集索引中去。

#### Redo log 落盘（Flush）的时机

1、mysql系统后台会定期落盘
2、mysql 正常关闭时
3、redo log 满了时(redo log 是固定大小的，采用循环写)
4、缓冲池在读取数据页进来时内存不足需要淘汰部分数据页，而淘汰的数据页如果是脏页也会导致落盘。

### undo log

`undo log` 是`innodb`引擎产生的，主要时候用于解决事务回滚和MVCC。数据修改的时候，不仅记录`redo log`，也会记录`undo log`。在事务执行失败的时候，会使用`undo log`进行回滚；MVCC的实现是为了在不加锁的情况下做到读写并行。所以mysql使用了MVCC的机制来实现，主要是用于 MySQL 的 ”读已提交“、”可重复读“ 隔离级别。

- **当前读**

select lock in share mode(共享锁), select for update ; update, insert ,delete(排他锁)这些操作都是一种当前读。它读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁

- **快照读**

快照读的实现是基于MVCC（多版本并发控制），避免了加锁操作，降低了开销

####  MVCC的实现原理

主要是依赖记录中的 **3个隐式字段**，**undo日志** ，**Read View** 来实现的。

***ROW_ID***隐藏主键、***TRX_ID***上一次修改的事务id，***roll_pointer***上一次修改之前保存在 `undo log`中的记录位置。对同一记录的修改，会记录修改前的数据到`undolog`中，roll_pointer类似链表结构，头尾相连。记录版本线性表。

#### Read View

Read View就是事务进行快照读操作的时候生产的读视图

```
trx_list 未提交事务ID列表，用来维护Read View生成时刻系统正活跃的事务ID 
up_limit_id 记录trx_list列表中事务ID最小的ID 
low_limit_id ReadView生成时刻系统尚未分配的下一个事务ID，也就是目前已出现过的事务ID的最大值+1
```

当前事物id(TRX_ID)会和up_limit_id、low_limit_id进行比较，看是否满足条件。并且还会看是否在trx_list事务ID列表中

#### RC,RR级别下的InnoDB快照读的区别

RC隔离级别下，是每个快照读都会生成并获取最新的Read View；而在RR隔离级别下，则是同一个事务中的第一个快照读才会创建Read View, 之后的快照读获取的都是同一个Read View

### binlog

binlog实现归档的功能，主从复制和数据恢复。binlog没有crash-safe的能力。

- redolog是对记录修改之后的物理日志。
- binlog是逻辑日志

防止crash-safe是因为在数据库重启后，只要把redo log中剩下的数据都恢复就行了。因为之前的数据都已经落盘了。如果用bin-log是不知道从何时开始，数据开始恢复，如先写bin-log，但是没有数据没有落盘。或者先落盘，在写bin-log。 之后归档bin-log都会有问题。 新的服务器，只能通过bin-log同步。 还有一点redolog只要数据落盘了，就会擦除`checkpoint `。

#### 两阶段提交

**其实就是把 redo log 的写入拆分成了两个步骤：prepare 和 commit**。

写redolog prerpare->写binlgo->修改redo log状态为commit->提交事务

#### crash后是如何恢复的？

通过两段式提交我们知道redo log和binlog在各个阶段会被打上prepare或者commit的标识，同时还会记录事务的XID，有了这些数据，在数据库重启的时候，会先去redo log里检查所有的事务，如果redo log的事务处于commit状态，那么说明在commit后发生了crash，此时直接把redo log的数据恢复就行了，如果redo log是prepare状态，那么说明commit之前发生了crash，此时binlog的状态决定了当前事务的状态，如果binlog中有对应的XID，说明binlog已经写入成功，只是没来的及提交，此时再次执行commit就行了，如果binlog中找不到对应的XID，说明binlog没写入成功就crash了，那么此时应该执行回滚。

### 2 




### reference

[面试题总结 数据库问题](https://blog.csdn.net/mashaokang1314/article/details/88253176)

[innodb为什么需要double write](https://www.jianshu.com/p/fab64085cac6)

[MySQL InnoDB的MVCC实现机制](https://pdai.tech/md/db/sql-mysql/sql-mysql-mvcc.html)
