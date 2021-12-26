# 秒杀

```java
public void safedUnLock(String key, String val) {
    String luaScript = "local in = ARGV[1] local curr=redis.call('get', KEYS[1]) if in==curr then redis.call('del', KEYS[1]) end return 'OK'"";
    RedisScript<String> redisScript = RedisScript.of(luaScript);
    redisTemplate.execute(redisScript, Collections.singletonList(key), Collections.singleton(val));
}

```

```java
public SeckillActivityRequestVO seckillHandle(SeckillActivityRequestVO request) {
SeckillActivityRequestVO response;
    String key = "key:" + request.getSeckillId();
    String val = UUID.randomUUID().toString();
    try {
        Boolean lockFlag = distributedLocker.lock(key, val, 10, TimeUnit.SECONDS);
        if (!lockFlag) {
            // 业务异常
        }

        // 用户活动校验
        // 库存校验，基于redis本身的原子性来保证
        Long currStock = stringRedisTemplate.opsForHash().increment(key + ":info", "stock", -1);
        if (currStock < 0) { // 说明库存已经扣减完了。
            // 业务异常。
            log.error("[抢购下单] 无库存");
        } else {
            // 生成订单
            // 发布订单创建成功事件
            // 构建响应
        }
    } finally {
        distributedLocker.safedUnLock(key, val);
        // 构建响应
    }
    return response;
}

```

然而，还有没有优化空间呢？有的！
由于服务是集群部署，我们可以将库存均摊到集群中的每个服务器上，通过广播通知到集群的各个服务器。网关层基于用户ID做hash算法来决定请求到哪一台服务器。这样就可以基于应用缓存来实现库存的扣减和判断。性能又进一步提升了！

### 分布式锁

