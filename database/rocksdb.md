# RocksDB

rocksdb适用于写多读少的场景，是 Facebook 开源的一个高性能持久化 KV 存储。Redis 和 RocksDB 之间没什么可比性，一个是 **缓存**，一个是 **数据库存储引擎**。从 Redis 官方给出的测试数据来看，它的 **随机读写性能大约在 50 万次 / 秒左右**。而 RocksDB 相应的随机读写性能大约在 20 万次 / 秒左右

### 为什么rocksdb可以这么快

RocksDB 采用了一个非常复杂的数据存储结构，并且这个存储结构采用了 **内存和磁盘混合存储方式**，使用磁盘来保证数据的可靠存储，并且利用速度更快的内存来提升读写性能。rocksdb使用lsm-tree数据结构做为存储

### LSM-Tree 如何兼顾读写性能？

LSM-Tree 的全称是：**The Log-Structured Merge-Tree**，是一种非常复杂的复合数据结构，它包含了 WAL（Write Ahead Log）、跳表（SkipList）和一个分层的有序表（SSTable，Sorted String Table）

![img](https://zq99299.github.io/note-book/assets/img/c0ba7aa330ea79a8a1dfe3a58547526e.c0ba7aa3.jpg)



- LSM-Tree如何写入

  1 当收到一条命令时，命令会被写入到磁盘的 WAL 日志中（图中右侧的 Log），顺序写只做数据恢复

  2 日志写完后，就会写到内存MemTable中，MemTable按照key组成的跳表。memtale写入一定容量后，就会转换成 Immutable MemTable，然后重新创建MemTable表写入

  3 Immutable MemTable变为只读，并且后台线程不停的写文件，因为生产的key是跳表，有序，写入的文件**SSTable**也是一个按照 Key 排序的结构。所以也是顺序写

  4 写入的文件内容虽然有序，但是文件无序，所以需要**分层合并机制**来解决乱序问题，SSTable 被分为很多层，越往上层，文件越少，越往底层，文件越多，每一层内的文件都是有序的，文件内的 KV 也是有序的，这样就比较便于查找了。

- LSM-Tree如何查找，先找内存MemTable等，后找文件SSTable。可能需要多次查找内存和文件才能找到。但越是被经常读写的热数据，分层结构就会越靠上层，key的查找也会越快。另外在内存中缓存 SSTable 文件的 Key，用布隆过滤器避免无谓的查找等来加速查找过程

