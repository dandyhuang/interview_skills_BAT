# 操作系统

有几个关系到性能，进程，CPU，MEM，网络IO，磁盘IO

[toc]

### 1. 文件IO

![img](http://images2015.cnblogs.com/blog/401155/201609/401155-20160923210933840-1173822707.gif)

#### fread和fwrite

fread每次都会从内核缓冲区读比要求更多的数据,之后放到应用进程的缓冲区

fwrite每次把数据写到应用进程缓冲区，等满了或者调用fflush在调用系统write写到内核缓冲区，fsync在从内核缓冲区刷新到磁盘

#### read和write

read就会从内核缓冲区（操作系统开辟的一段空间用来存储磁盘上的数据）读10个字节数据到数组中，所以每次调用read会涉及到用户态与內核态之间的切换从而损耗一定的性能

调用write函数，是直接通过系统调用把数据从应用层拷贝到内核层，从application buffer 拷贝到 page cache 中。

### 2. mmap

mmap把page cache 地址空间映射到用户空间，应用程序像操作应用层内存一样，写文件。省去了系统调用开销

### 3. O_DIRECT

如果想绕过page cache，直接把数据送到磁盘设备上怎么办。通过open文件带上O_DIRECT参数，这时write该文件，就是直接写到设备上。 应用场景：数据库系统，其高速缓存和IO优化机制均自成一体，无需内核消耗CPU时间和内存去完成相同的任务

# happen-before

java并发编程也有“原子性”、“有序性”、“可见性”三个问题。

- 单线程happen-before原则：在同一个线程中，书写在前面的操作happen-before后面的操作。(如果语句没有依赖关系，可能发生指令重排序)

- 锁的happen-before原则：同一个锁的unlock操作happen-before此锁的lock操作。

- volatile的happen-before原则：对一个volatile变量的写操作happen-before对此变量的任意操作(当然也包括写操作了)。

- happen-before的传递性原则：如果A操作 happen-before B操作，B操作happen-before C操作，那么A操作happen-before C操作。





### reference


[mmap和direct io和write和fwrite区别](https://blog.csdn.net/xiaofei0859/article/details/74674631)