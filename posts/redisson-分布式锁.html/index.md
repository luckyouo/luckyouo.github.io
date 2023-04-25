# redisson 分布式锁


## 前言

在微服务应用当中，某一个服务可能部署在多台机器上，在高并发的情况下，对资源的访问可能出现读写数据异常，而 Java 内部提供的 JUC 只能在单应用的情况下锁住，即只能锁住当前进程，在这种情况下需要分布式锁来限制资源的占用情况，其中 Redisson 为主要使用工具。

## 分布式锁

常用的分布式锁有如下几种方式来实现：

1. 通过对数据库的的表结构来实现：比如通过版本号来实现，这种情况类似于乐观锁
2. 通过 redis 的原子操作实现：redis 的 lua 脚本操作、setnx 等原子操作原子操作方法，可以实现分布式锁
3. 通过 redisson：redisson 是 redis 官方提供的分布式锁的实现组建
4. 通过 Redlock：Redlock 是 redis 作者实现的分布式锁
5. 通过 zookeeper：

## Redisson 实现分布式锁

Redisson 是架设在 Redis 基础上的一个 Java 驻内存数据网格（In-Memory Data Grid）

Redisson 需要先进行配置才能使用，基础配置如下。

```java
@Configuration
public class MyRedissonConfig {

    /**
     * 所有对Redisson的使用都是通过RedissonClient对象
     * @return
     * @throws IOException
     */
    @Bean(destroyMethod="shutdown")
    public RedissonClient redisson(@Value("${spring.redis.host}") String url) throws IOException {
        //1、创建配置
        //Redis url should start with redis:// or rediss://
        Config config = new Config();
        config.useSingleServer().setAddress("redis://"+url+":6379");
        //2、根据Config创建出RedissonClient示例
        RedissonClient redissonClient = Redisson.create(config);
        return redissonClient;
    }
}
```

后续在需要对资源进行限制是，则可以注入 `RedissonClient` 来获取锁

```java
RLock lock = redisson.getLock("anyLock");// 最常见的使用方法 lock.lock(); // 加锁以后 10 秒钟自动解锁// 无需调用 unlock 方法手动解锁 
lock.lock(10, TimeUnit.SECONDS); // 尝试加锁，最多等待 100 秒，上锁以后 10 秒自动解锁 
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS); 
if (res) {
  try { ...
      } finally {
    lock.unlock();
  }
}
```

## 参考资料

[分布式锁看这篇就够了](https://zhuanlan.zhihu.com/p/42056183)

[Java项目《谷粒商城》Java架构师 | 微服务 | 大型电商项目](https://www.bilibili.com/video/BV1np4y1C7Yf?p=1&vd_source=b6eb6fd64ed675d7acddef5b0467fac9)


