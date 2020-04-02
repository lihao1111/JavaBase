## Redis基础知识点

### redis键 key
​	1.keys *
​	2.exists key 判断是否存在	

​	3.type key

### 五种基本类型 

​	1.String  set get	setnx
​	2.List lpush lpop 
​	3.Set sadd SMEMBERS scard
​	4.ZSet zadd
​	5.Hash hset hget hlen hsetnx

### redis的备份

##### rdb原理

最大化redis的性能： 父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作(dump.rdb )，父进程无须执行任何磁盘 I/O 操作。 

​	

##### 缺点

​	有可能会造成最后一个时间段的数据丢失。

​	同时在fork进程的时候会造成内存的 占用过大 使用双倍内存。

​	RDB需要经常fork子进程来保存数据集到硬盘上，当数据集比较大的时候，fork的过程是非常耗时的，可能导致Redis在一些毫秒级不能响应客户端请求。

##### 优点

​	适用于恢复大数据集，相比较aof方式，rdb更快

##### 三种生成dump.rdb文件的策略

​	1、save 命令
​	2、bgsave 命令 

​		BGSAVE：Redis会在后台异步进行快照操作，快照操作同时还可以响应客户端请求。可以通过lastsave命令获取最后一次成功执行快照的时间。

​	3、redis.conf 中 配置的save <seconds> <changes>
​	flushAll命令 也会立即生成dump.rdb文件

### AOF

##### AOF的原理 

​	append only file:以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来（读操作不记录），只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话根据日志文件的内容将写指令从前到后执行一次已完成数据的恢复工作。

##### 缺点

​	1.随着日志的增大append.aof文件会越来越大，占用磁盘

​	2.记录很多中间操作数据，在恢复数据的时候会比rdb模式慢，不适用于大数据量操作

##### 优点

​	1.可以灵活配置 同步模式，默认为everysec 每秒同步一次，比rdb更加确保数据的一致性

##### RDB重写文件 ReWrite

​	AOF采用文件追加方式，文件会越来越大，为避免出现此种情况，新增了重写机制，当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令gbrewriteaof

​	AOF文件持续增长而过大时，会fork出一条新进程来将文件重写（也是先写临时文件最后在rename），遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

​	触发重写条件：

​	Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。redis.conf 中配置 

​	No-appendfsync-on-rewrite：重写时是否可以运行Appendfsync，用默认no即可，保证数据安全性。

​	Auto-aof-rewrite-min-size：设置重写的基准值 

​	Auto-aof-rewrite-perentage：设置重写的基准值

##### RDB 和 AOF

​	RDB和AOF可以共存，但是恢复的时候找的是AOF，如果AOF文件异常，可以通过check-aof进行AOF修复。

### redis过期策略和内存淘汰

1.定期删除+惰性删除

```java
定期删除：指的是redis默认是每隔100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除

惰性删除：在你获取某个key的时候，redis会检查一下 ，这个key如果设置了过期时间那么是否过期了，如果过期了此时就会删除，不会给你返回任何东西
```

2.内存淘汰机制

```java
redis.conf 配置	
# maxmemory-policy noeviction　　
noeviction：当内存使用达到阈值的时候，所有引起申请内存的命令会报错。
volatile-lru：在设置了过期时间的键空间中，优先移除最近未使用的key。
volatile-random：在设置了过期时间的键空间中，随机移除某个key。
volatile-ttl：在设置了过期时间的键空间中，具有更早过期时间的key优先移除。
allkeys-random：在主键空间中，随机移除某个key。
allkeys-lru：在主键空间中，优先移除最近未使用的key。

```

### redis常见问题

1.缓存穿透

​	 **就是客户持续向服务器发起对不存在服务器中数据的请求。客户先在Redis中查询，查询不到后去数据库中查询。** 

​	**解决方案**

```java
1.接口层增加校验，对于非法数据过滤
2.当通过某一个key去查询数据的时候，如果对应在数据库中的数据都不存在，我们将此key对应的value设置为一个默认的值，比如“NULL”，并设置一个缓存的失效时间。
3.采用布隆过滤器    
```

2.缓存击穿

​	 **就是一个很热门的数据，突然失效，大量请求到服务器数据库中** 

```java
1.对缓存查询加锁，如果KEY不存在，就加锁，然后查DB入缓存，然后解锁；其他进程如果发现有锁就等待，然后等解锁后返回数据或者进入DB查询。
```

```java
1.单机 使用synchronized lock 等重入锁
    query(String key){
    	String value = redis.get(key);
    	Lock lock = new ReentLock();
    	if(value == null){
            if(lock.tryLock()){
                dbValue = db.get(key);  
                 redis.set(key, dbValue);  
                 redis.del(brokenKey);  
            }else{
                Thread.sleep()
                query(key)
            }
        }
    	return value
	}
2.分布式使用分布式锁
    setNex
     jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime)
```



3.缓存雪崩

​	 **就是大量数据同一时间失效 ，造成请求都发往数据库**

```java
1.将系统中key的缓存失效时间均匀地错开，防止统一时间点有大量的key对应的缓存失效。比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机
```

### redis分布式锁

```java
分布式锁的实现
1、在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行；
2、高可用的获取锁与释放锁；
3、高性能的获取锁与释放锁；
4、具备可重入特性；
5、具备锁失效机制，防止死锁；
6、具备非阻塞锁特性，即没有获取到锁将直接返回获取锁失败。
```

#### 加锁

```java
 String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

1.key :锁的唯一性
2.value : 解锁的时候 作为一个校验符和加锁对应，可以使用UUID.randomUUID()生成(//我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成)
3.setNx 当key不存在的时候，我们才会进行set操作，如果不存在才会进行set操作
4.第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。
5.第五个为time，与第四个参数相呼应，代表key的过期时间。   
注意
    setNex 和设置过期时间不可以拆分， 保证 setNx和expire()的原子性。
    value 的唯一性，保证释放锁的时候和持有锁是同一个对象
```

#### 释放锁

使用lua脚本

jedis.eval(lua脚本)

保证原子性

```java
就是在eval命令执行Lua代码的时候，Lua代码将被当成一个命令去执行，并且直到eval命令执行完成，Redis才会执行其他命令。
```

## Zookeeper实现分布式锁

```java
基于zk的文件系统 同一个目录下只能有一个唯一文件名
    1.创建mylock目录
    2.线程A想获取锁就在mylock目录下创建临时顺序节点
    3.获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁
    4.线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点
    5.线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁
```



### redis事务

#### 基本命令

	MULTI：标记事物的开启
	EXCE：执行事物
	DISCARD：取消事务，放弃执行事务块内的所有命令
	WATCH KEY 监视一个或者多个KEY，如果在事物执行之前 这个KEY被其他命令改动，那么事务将被打断，需要使用UNWATCH取消对该KEY的监视，并且重新进行WATCH监听 以及MULTI事物 开启 EXCE事物执行等一系列过程	
### redis发布订阅

#### 基本命令

PUBLISH	  发布	

SUBSCRIBE 订阅

### redis复制（Master/Slave）

就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slver机制，Master以写为主，Slave以读为主。

#### 配置

配从不配主

info replication 查看redis的信息

### 主要的模式

1、一主二从

​	slave of ip port	从服务器 关联主服务器

​	slave 每次与master断开之后，都需要重新连接，除非你配置进redis.conf文件

​	master 断开， 重启后依然是master的角色

2、薪火相传

​	上一个Slave可以是下一个slave的Master，slave同样可以接受其他slaves的连接和同步请求，那么该slave作为链条中下一个的master，可以有效减轻master的写压力。

​	中途变更转向：会清楚之前的数据，重新建立拷贝最新的。

​	传递过程中有可能会造成数据的丢失

3、反客为主

​	slave on one	使当前数据库停止与其他数据库的同步，转成主数据库。

4、哨兵模式

​	反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

​	步骤：

​	1.sentinel.conf配置	sentinel monitor被监控数据库名字（自己起名字）127.0.0.1 6379 1 （上面最后一个数字1，表示主机挂掉后slave投票看让谁接替成为主机，得票数多的成为主机）

​	2.Redis-sentinel  /myredis/sentinel.conf

​	3.之前的master回来之后，哨兵监测到，之前的master会编程slave

#### 复制的原理

​	slave 连接到master 后会发送一个sync 同步请求，master启动后台存盘过程，将全量数据同步给slave，完成一次完成同步，后续master将修改请求发送给slave 完成增量同步。但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。



#### 项目中使用的redis

Springboot  redis  spring-boot-starter-data-redis

1.RedisConfig 继承 CachingConfigurerSupport 重写 redisTemplate模板

2.设置key使用 stringRedis序列化器， value使用fastJosnRedis序列化器

```java
key:
StringRedisSerializer implements RedisSerializer<Object> 必须重写 不然 @Cacheable(key)会报错
value:    
FastJsonRedisSerializer<T> implements RedisSerializer<T>
```



3.设置自定义key, @Cacheable()中没有指定key的时候，使用默认的自定义key [package, class, methodName]然后进行一个SHA256编码

4.重写一个errorHandler方法，在redis各个注解的时候 记录日志



#### epoll IO多路复用

```java
1.执行epoll_create()时，创建了红黑树和就绪链表；

2.执行epoll_ctl()时，如果增加socket句柄，则检查在红黑树中是否存在，存在立即返回，不存在则添加到树干上，然后向内核注册回调函数，用于当中断事件来临时向准备就绪链表中插入数据；

3.执行epoll_wait()时立刻返回准备就绪链表里的数据即可。

```

```java
ET模式（边缘触发）只有数据到来才触发，不管缓存区中是否还有数据，缓冲区剩余未读尽的数据不会导致epoll_wait返回；
LT 模式（水平触发，默认）只要有数据都会触发，缓冲区剩余未读尽的数据会导致epoll_wait返回。
```

### redis session共享

解决 服务集群 多次登录问题

```java
Nginx 分发请求
    upstream nameTest{
    	server 服务器地址+port
		server 服务器地址+port            
	}
	location /{
        proxy_pass:http://nameTest //客户端访问的地址
    }
负载均衡的算法：
    轮询
    	加权轮询 weight = 1
    最小连接数
    IP哈希
    URL散列
```

方案一： tomcat 容器 实现 session复制

```java
tomcat server.xml 中配置<cluster>
缺点：
    性能低
    内存消耗
```

方案二：redis实现session共享

```java
1.pom.xml添加 redis-session-start启动类
2.@EnableRedisHttpSession
    SessionResposityFilter
    SessionResposityRequestWrapper 包装类
    getSession
    saveSession
在请求到达服务之前 通过  SessionResposityFilter 过滤器 去redis中获取session
    存放的是hash的结构
    spring:redis:expires
    spring:redis:sessionId
```



### 布隆过滤器

```java
1、布隆过滤器是一个bit数组，如果我们需要映射一个值到布隆过滤器中，我们需要使用多个不同的哈希函数生成多个哈希值，并将每个生成的哈希值指向的bit位置设置为1。

2、传统的布隆过滤器不支持删除，countingBf 支持删除
    
```

#### 如何选择哈希函数的个数和布隆过滤器的长度

```java
K:哈希函数个数	m:布隆过滤器长度	n:插入元素的个数
    
    K= M/N * ln2
```































