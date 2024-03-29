# CPU

CPU提供了原子操作、关中断、锁内存总线、内存屏障等机制；

### cpu缓存

因为cpu的运算速度最快，内存(主存)的读写速度无法和其速度匹配。**假如定义cpu的一次存储或访问为一个时钟周期，那么内存的一次运算通常需要几十甚至几百个始终周期**，所以为了提高cpu的利用率，引入了L1高速缓存、L2高速缓存、L3高速缓存。

- L1高速缓存：也叫一级缓存。一般内置在内核旁边，是与CPU结合最为紧密的CPU缓存。一次访问只需要2~4个时钟周期
- L2高速缓存：也叫二级缓存。空间比L1缓存大，速度比L1缓存略慢。一次访问约需要10多个时钟周期
- L3高速缓存：也叫三级缓存。部分单CPU多核心的才会有的缓存，介于多核和内存之间。存储空间已达Mb级别，一次访问约需要数十个时钟周期。

### 总线锁和缓存锁

[CPU](https://baike.baidu.com/item/CPU/120556)总线是芯片组与主板的核心。这条总线主要由CPU使用，用来与高速缓存、主存和北桥（或MCH）之间传送信息

总线锁 ：顾名思义就是，锁住总线。**总线锁存在较大的缺点，一旦某个处理器获取总线锁，其他处理器都只能阻塞等待，多处理器的优势就无法发挥**。类似数据库的表锁

缓存锁：只需要“锁定”被缓存的共享对象（实际为：缓存行）即可，接受到lock指令，通过**缓存一致性协议**，维护本处理器内部缓存和其他处理器缓存的一致性。类似数据库的行锁

### 缓存行

一次获取一整块的内存数据，放入缓存。那么这一块数据，通常称为**缓存行（cache line）**。缓存行（cache line）是CPU缓存中可分配、操作的最小存储单元。目前64位架构下，64字节最为常用。

### 缓存一致性协议（如：MESI）

MESI是Modified（修改）、Exclusive（独占）、Shared（共享）、Invaild（失效）四种状态的缩写

- Modified（修改）：该缓存行仅出现在此cpu缓存中，缓存已被修改，和内存中不一致，等待同步至内存。
- Exclusive（独占）：该缓存行仅出现在此cpu缓存中，缓存和内存中保持一致。
- Shared（共享）：该缓存行可能出现在多个cpu缓存中，且多个cpu缓存的缓存行和内存中的数据一致。
- Invalid（失效）：由于其他cpu修改了缓存行，导致本cpu中的缓存行失效。

### 伪共享（false sharing）

多个变量在同一个缓存行，因为需要遵循MESI协议。多核多线程并发场景下，多核要操作的不同变量处于同一缓存行，某cpu更新缓存行中数据，并将其写回缓存，同时其他处理器会使该缓存行失效，如需使用，还需从内存中重新加载。这对效率产生了较大的影响

解决方案：

- **缓存行填充**(空间换取时间)

### CPU亲缘性问题

https://www.cnblogs.com/lojunren/p/3865232.html

### 多核cpu和多个cpu

例如，你需要搬很多砖，你现在有一百只手。当你将这一百只手全安装到一个人身上，这模式就是多核。当你将这一百之手安装到50个人身上工作，这模式就是多CPU。

![image-20211007220428797](/Users/11126518/knowledge/interview_skills_BAT/img/image-20211007220428797.png)

### CPU跑满原因

一．硬件原因服务器CPU自身出现问题，比如机房散热不足，温度过热或者驱动故障，导致CPU性能下降，很容易造成CPU跑满的情况。

二．网站代码错误排查硬件原因后，我们进入网站后台查看是哪些程序占用了大量CPU，检测这些代码自身是否有问题。如果是代码问题就需要网站技术人员优化代码或者删除重新搭建网站。

三．网站访问量增大网站运行一段时间后，访问量大大的增加，确定是否是因为网站访问量上涨导致CPU负荷跟不上。如果是业务本身发展因素，建议升级配置，这种情况其他的操作效果不大，因为CPU很快再次跑满。

四．中毒原因我们才后台排查程序时发现有来历不明的进程时，强行占用大量CPU资源，基本可以断定中毒导致CPU跑满。中了毒的服务器一定要用杀毒工具及时清除病毒程序并删除病毒文件与注册表键值。

五．攻击原因而攻击比较常见的方式就是DDOS和CC。通过大量的访问强行占用服务器资源，导致服务器崩溃，网站无法连接。遇到攻击只能增强服务器防御或者暂时关闭网站。网站服务器出现CPU跑满并不可怕，千万不要恐慌，静心找出导致CPU跑满的原因，就很容易处理，必要时可以联系香港服务器商进行协助。

https://zhuanlan.zhihu.com/p/368221879

### Reference

[CPU中的缓存、缓存一致性、伪共享和缓存行填充](https://zhuanlan.zhihu.com/p/135462276)

