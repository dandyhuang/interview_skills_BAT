# 缓存问题

### 缓存与数据库数据不一致怎么办

假设使用的主存分离, 读写分离的数据库.

发生的可能性: 

1. 主库更新数据, 主库到从库的同步未完成, 从库读取数据, 未读到最新数据, 而更新了缓存

解决方案: 在从库接收到数据更新操作时, 淘汰掉这条数据的缓存

解决方案: 

常用cache aside先更新db后删除缓存。这种在很极端的情况下才会有问题。

- 读缓存时，刚好缓存失效了，获取的数据，是更新db前的旧数据。
- 并且有并发的写操作，更新db删除缓存数据时。

这种概率还是比较低的。因为cache aside中，读缓存需要在更新之前，并且更新缓存需要在删除缓存之后写入才可能有数据不一致的问题。 但是因为数据库update更新，锁表等操作。肯定是更缓慢的。

如果需要强一致性。可能需要使用`两阶段提交协议`等，但带来的是性能下降的问题

1. 在数据性一致性要求不高时, 忽略掉这个数据的不一致
2. 延时双删策略
   1. 先删除缓存
   2. 再写数据库
   3. 休眠500毫秒
   4. 再次删除缓存
   5. 设置缓存过期时间
3. 异步更新缓存(基于订阅binlog的同步机制)，最终一致性
   1. Redis订阅 mysql binlog消息
   2. 依据消息队列来进行相关操作

### Redis 缓存穿透

起因: 恶意请求故意大量查询不存在的key, 让请求到达MySQL, 对后端造成很大压力
解决方案: 

- 对不存在的key也做有有效期的缓存; 
- 对存在  key 做一个 布隆过滤, 不存在于过滤器中的数据直接返回空;

### Redis 缓存雪崩

起因: 大量的缓存在同一时间段失效, 导致后端压力大.
解决方案: 对缓存的有效时间使用不同的过期时间; 做二级缓存; 



### 缓存热点 key 重建

在缓存失效瞬间, 有大量的线程来重建缓存, 造成后端负载压力大, 甚至可能让应用崩溃

解决方案:

1. 互斥锁

多进程获取到锁的可以更新缓存, 其它线程等待. 由于建立缓存是个幂等操作, 所以可以在一两次获取不到锁时, 可以直接绕过互斥锁

2. 永远不过期

### Redis 分布式锁

> 为什么要分布式锁

为了确保在多个线程中, 多服务器中执行任务时, 能够达到一致性

#### 单机版redis问题

1 因为需要引入超时机制，并且某些场景如系统gc、或者本次请求存在慢查询等，导致锁触发超时机制而释放了。另外一个进程获取了锁，之后上一个进程因为gc恢复等又释放了该锁。所以需要进程确认是否是自己的锁，才能释放。所以通常会加唯一标识上锁127.0.0.1:6379> SET redis_locks $uuid EX 60 NX。

2 加入唯一标识后，所以需要先get key判断是否是自身的锁，后在del释放。但显然这两个并非原子操作，所以需要引入lua脚本

```lua
--- 原子脚本中包含两个步骤：1）判断当前锁是否是自己的 2）锁是自己的进行释放
if redis.call("GET", KEYS[1]) == ARGV[1]
then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

3 网络延迟问题，存在redis加锁成功了，但是redis回包时间超过了过期时间。所以需要记录获取的时间T1和回包响应的时间T2，判断`T2-T1 < EX Time` 如果不成立，则获取锁失败。这里是判断获取锁是否成功，但是如果释放锁的时候，还是需要get完在del的。

4 在更变态一些，ntp时钟漂移问题

### redisson机制

如果当前业务逻辑存在先后因果关系时，如`Read And Modify, Check Then Act`等，可能会导致数据一致性的问题。如当前值为a=100，++后，因为自身处理耗时问题。导致锁被释放了。之后其他进程抢到了锁。最后导致当前线程释放锁报错(已不是自身uuid)，并且新抢占的锁修改数据a，导致数据不一致个问题等。`Redisson`通过看门狗机制客户端开启一个守护线程，如果当前锁即将过期，但是业务逻辑仍未完成，则该线程会自动对锁进行**续期**；

### redlock机制(redis distributed locks)

> 缺陷

当master上加了锁, 还未同步到 slave 时, master down了, 这个时候 slave 成为了 master, 其中并没有锁, 这个时候就会出现多个客户端同时拿到锁的问题.

**不部署从库和哨兵机制，但主库要部署多个，官方推荐为5个(奇数)**，加锁和解锁通过`Quorum`机制保证

大致步骤

- 获取当前的时间（单位是毫秒）
- 使用相同的key和随机值在N个节点上请求锁。这里获取锁的尝试时间要远远小于锁的超时时间，防止某个masterDown了，我们还在不断的获取锁，而被阻塞过长的时间。
- 只有在大多数节点上获取到了锁，而且总的获取时间小于锁的超时时间的情况下，认为锁获取成功了。
- 如果锁获取成功了，锁的超时时间就是最初的锁超时时间进去获取锁的总耗时时间。
- 如果锁获取失败了，不管是因为获取成功的节点的数目没有过半，还是因为获取锁的耗时超过了锁的释放时间，都会将已经设置了key的master上的key删除。

### zookeeper实现分布式锁的优势

- 创建临时有序节点。获取最小的节点则加锁成功。
- 事件监听（`Watcher`机制）：在会话结束或者会话超时后，基于临时顺序节点实现通知，ZK 会自动删除该节点，并通知下一个最小的准备获取锁的节点。可以避免服务宕机导致的锁无法释放，而产生的死锁问题

### zookeeper实现分布式锁的一些问题

当然通过zk实现分布式锁仍然存在很多问题，我们同样按照**网络延迟、进程暂停**的角度进行分析；

1. 进程1创建临时节点`/exclusive_lock/lock`成功，拿到了锁
2. 进程1因为机器长时间GC而暂停
3. 进程1无法给 Zookeeper 发送心跳，Zookeeper将临时节点删除
4. 进程2创建临时节点`/exclusive_lock/lock` 成功，拿到了锁
5. 进程1机器GC结束后恢复，它仍然认为自己持有锁（产生冲突）

因此无论是通过`zk`还是`redis`还是其他锁服务，都会存在类似的问题，即**获取到锁的服务在持有锁期间发生进程暂停导致锁释放后，另一个进程获取到锁，导致两个客户端都会认为自己持有锁**。

可以使用类似`Redisson`的续约机制，通过一个守护线程进行zk临时节点的维护；但是zk不会存在释放了别人锁的情况，所以不需要通过类似`Lua`的机制来释放锁；

### concurrent_queue并发队列



#### 什么是热点 KEY

1. 单一 key 在突发事件中访问量突增, 会对单一的 Server 造成很很大压力, 超过 Server 极限时, 就导致了热点 Key 问题的产生

#### 怎么解决

1. 服务端 缓存: 即将数据缓存至服务端的内存中
   - 如何保证 Redis 和 服务端热点 Key 的数据一致性: 使用 Redis 自带的消息通知机制
2. 备份热点 Key: 即将热点 Key + 随机数, 随机分配至 Redis 其它节点中, 这样访问热点 key 的时候就会分配压力到其它节点l
   - Redis 集群中包含 `16384` 个哈希槽, 集群使用公式 `CRC16(key) % 16384` 来定位 key 属于哪个槽, 那么只需要在 key 后加随机数
     来储存和访问即可平均分布到不同的机器上了 

##### 如何找到热点 Key

- 凭借经验, 对系统的了解，进行预估
- 客户端收集: 在操作 Redis 前对数据进行统计
- 抓包进行评估: Redis 使用了 TCP 协议与客户端进行通信, 通信协议采用的是 RESP, 所以能进行拦截宝进行解析
- 在 proxy 层, 对每一个 redis 请求进行收集上报
- Redis 自带命令查询: Redis 4.0.4 版本提供了 `redis-cli -hotkeys` 就能找出热点 key 