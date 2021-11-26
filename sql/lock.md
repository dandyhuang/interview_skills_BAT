# MYSQL LOCK

### MDL(Metadata Lock)元数据锁

元数据锁保证对表执行DML相关操作时，mysql会自动给这个表加上元数据锁。会对改变表结果的sql进行互斥

### 表锁

```sql
// 共享锁
lock tables ${tableName}  read;、
// 独占锁
lock tables ${tableName} write;
// 解锁
unlock table
```

### 行锁

```sql

// 行锁
update xxx where id = xxx; //id索引
```

- 记录锁（Record Lock）：单个行记录上的锁。

- 间隙锁（Gap Lock）：间隙锁，锁定一个范围，但不包括记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。

- 临键锁（Next-Key Lock）：**是行锁和gap锁的组合**,锁定一个范围，并且锁定记录本身。对于行的查询，都是采用该方法，主要目的是解决幻读的问题。

### 间隙锁

- 唯一索引、主键等
  - 精确等值检索，Next-Key Locks就退化为记录锁，不会加gap锁
  - 范围检索，会锁住where条件中相应的范围，范围中的记录以及间隙，换言之就是加上记录锁和gap 锁（至于区间是多大稍后讨论）。
  - 不走索引检索，全表间隙加gap锁、全表记录加记录锁
- 非唯一索引 
 ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/5/12/16aac6a44ed032b9~tplv-t2oaga2asx-watermark.awebp)
   Session A执行后会锁住的范围： (5, 8], (8, 11]   *** 左开右闭 *** 第8步不会阻塞
  - 精确等值检索，Next-Key Locks会对间隙加gap锁（至于区间是多大稍后讨论），以及对应检索到的记录加记录锁。
  - 范围检索，会锁住where条件中相应的范围，范围中的记录以及间隙，换言之就是加上记录锁和gap 锁（至于区间是多大稍后讨论）。
- 非索引检索，全表间隙gap lock，全表记录record lock

### 意向锁

意向锁是一种`不与行级锁冲突表级锁`

```sql
// 意向共享锁（s锁）
select * from xxx lock in share mode;
// 意向排他锁（x锁）
select * from xxx for update;
```

