### Redis 简介

Redis 是一种**非关系型（NoSQL）内存键值数据库。**

**应用：缓存，分布式锁，消息队列。**



### Redis 与 Memcached

两者都是非关系型内存键值数据库，主要有以下不同：

#### 数据持久化

Redis 支持两种持久化策略：RDB 快照和 AOF 日志，而 **Memcached 不支持持久化。**

#### 数据类型

Memcached 仅支持字符串类型，而 Redis 支持五种不同的数据类型，更灵活。

#### 分布式

Memcached 不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。Redis集群实现了分布式的支持。



###  缓存数据的处理流程

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f72ed4332bdb4db69567759526b9f989~tplv-k3u1fbpfcp-zoom-1.image" style="zoom: 67%;" />



### Redis的好处

**高性能** ：

用户第一次访问数据库中的某些数据时需要从硬盘中读取，比较慢。如果用户访问的数据属于高频数据并且不会经常改变，我们就可以将该用户访问的数据存在缓存中。用户下一次访问这些数据时可以直接从缓存中获取。

**高并发：**

像 MySQL 这类的数据库的 QPS（服务器每秒可以执行的查询次数） 大概都在 1w 左右（4 核 8G） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（单机 redis ，redis 集群更高）。

因此，**缓存能够承受的数据请求数量远远大于直接访问数据库，**把数据库中的部分数据转移到缓存中，这样用户的一部分请求会直接到缓存而不用经过数据库，也就提高的系统整体的并发。



### 五种数据结构及使用场景


#### string

1. **介绍** ：string 数据结构是简单的 key-value 类型。 
2. **应用场景：** 常用在需要**计数**的场景，比如**用户的访问次数、热点文章的点赞、转发数量。**
3. **常用命令:** `set,get,strlen,exists,incr,setnx,getset` 等等。

**普通字符串的基本操作：**

```bash
127.0.0.1:6379> set key value #设置 key-value 类型的值
OK
127.0.0.1:6379> get key # 根据 key 获得对应的 value
"value"
127.0.0.1:6379> exists key  # 判断某个 key 是否存在
(integer) 1
127.0.0.1:6379> strlen key # 返回 key 所储存的字符串值的长度。
(integer) 5
127.0.0.1:6379> del key # 删除某个 key 对应的值
(integer) 1
127.0.0.1:6379> get key
(nil)
```

**计数器（字符串的内容为整数的时候可以使用）：**

```bash
127.0.0.1:6379> set number 1
OK
127.0.0.1:6379> incr number # 将 key 中储存的数字值增一
(integer) 2
127.0.0.1:6379> get number
"2"
127.0.0.1:6379> decr number # 将 key 中储存的数字值减一
(integer) 1
127.0.0.1:6379> get number
"1"
```

**过期**：

```bash
127.0.0.1:6379> expire key  60 # 数据在 60s 后过期
(integer) 1
127.0.0.1:6379> ttl key # 查看数据还有多久过期
(integer) 56
```



#### list

1. **介绍** ：Redis 的 list 的实现为一个 **双向链表**，可以支持反向查找和遍历。
2. **应用场景:** **发布与订阅，或者说消息队列。**

**发布与订阅的原理**

，Redis-server维护了一个字典，字典的键是一个个频道，值是一个链表，链表中保存了所有订阅这个频道的客户端。客户端通过SUBSCRIBE命令订阅某频道，就是将客户端添加到指定频道的订阅链表中。

<img src="https://i.loli.net/2020/12/11/AcIqOSE3nyplQKM.png" alt="image.png" style="zoom:50%;" />





3.**常用命令:** `rpush,lpop,lpush,rpop,lrange、llen` 等。（命令前面加上L，代表对list操作）

**通过 `rpush/lpop` 实现队列：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26673da73efc410b8f1ae8e1fbc246ec~tplv-k3u1fbpfcp-zoom-1.image)



```bash
127.0.0.1:6379> rpush myList value1 # 向list的右边添加元素
(integer) 1
127.0.0.1:6379> rpush myList value2 value3 # 向list的右边添加多个元素
(integer) 3
127.0.0.1:6379> lpop myList # 将 list的最左边元素取出
"value1"
127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
1) "value2"
2) "value3"
127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
1) "value2"
2) "value3"
```

**通过 `rpush/rpop` 实现栈：**

```bash
127.0.0.1:6379> rpush myList2 value1 value2 value3
(integer) 3
127.0.0.1:6379> rpop myList2 # 将 list的头部(最右边)元素取出
"value3"
```

**通过 `llen` 查看链表长度：**

```bash
127.0.0.1:6379> llen myList
(integer) 3
```



#### hash

1. **介绍** ：hash 的key是string类型，value 是一个映射表，**适合用于存储对象， 比如存储用户信息，商品信息等。**
2. **应用场景:** **系统中对象数据的存储。**
3. **常用命令：** `hset,hget,hgetall,hkeys,hvals,hexists` 等。

```bash
# 双引号可加可不加，Redis会自动识别类型
#hmset:同时将多个“域-值”对存储在key键中，如果key不存在会自动创建，如果field已经存在，则会覆盖原来的值。操作成功后返回值OK。
127.0.0.1:6379> hmset userInfoKey name "guide" description "dev" age "24"
OK
127.0.0.1:6379> hexists userInfoKey name # 查看 key 对应的 value中指定的字段是否存在。 
127.0.0.1:6379> hget userInfoKey name # 获取存储在哈希表中指定字段的值。"guide"
127.0.0.1:6379> hgetall userInfoKey # 获取在哈希表中指定 key 的所有字段和值
1) "name"
2) "guide"
3) "description"
4) "dev"
5) "age"
6) "24"
127.0.0.1:6379> hset userInfoKey name "GuideGeGe" # 修改某个字段对应的值
```



#### set

1. **介绍 ：**Redis 中的 set 类型是一种无序集合。

2. **应用场景:** 

   **需要存放的数据不能重复：一个用户的关注集和粉丝集。**

   **获取多个数据集合的交集、并集等场景：共同关注、共同粉丝、共同喜好（求交集）。**

3. **常用命令：** `sadd,spop,smembers,sismember,scard,sinterstore,sunion` 等。

```bash
127.0.0.1:6379> sadd mySet value1 value2 # 添加元素进去
(integer) 2
127.0.0.1:6379> sadd mySet value1 # 不允许有重复元素
(integer) 0
127.0.0.1:6379> smembers mySet # 查看 set 中所有的元素
1) "value1"
2) "value2"
127.0.0.1:6379> scard mySet # 查看 set 的长度
(integer) 2
127.0.0.1:6379> sismember mySet value1 # 检查某个元素是否存在set 中，只能接收单个元素
(integer) 1
127.0.0.1:6379> sadd mySet2 value2 value3
(integer) 2
127.0.0.1:6379> sinterstore mySet3 mySet mySet2 # 获取 mySet 和 mySet2 的交集并存放在 mySet3 中
(integer) 1
127.0.0.1:6379> smembers mySet3
1) "value2"
```



#### sorted set（zset）

1. **介绍：** 和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。
2. **应用场景：** **需要根据数据的某个权重进行排序：直播间在线用户列表，各种礼物排行榜。**
3. **常用命令：** `zadd,zcard,zscore,zrange,zrevrange,zrem` 等。

```bash
127.0.0.1:6379> zadd myZset 3.0 value1 # 添加元素到 sorted set 中，3.0 为权重
(integer) 1
127.0.0.1:6379> zadd myZset 2.0 value2 1.0 value3 # 一次添加多个元素
(integer) 2
127.0.0.1:6379> zcard myZset # 查看 sorted set 中的元素数量
(integer) 3
127.0.0.1:6379> zscore myZset value1 # 查看某个 value 的权重
"3"
127.0.0.1:6379> zrange  myZset 0 -1 # 顺序（从小到大）输出某个范围区间的元素，0 -1 表示输出所有元素
1) "value3"
2) "value2"
3) "value1"
127.0.0.1:6379> zrange  myZset 0 1 # 顺序输出某个范围区间的元素，0为start，1为stop
1) "value3"
2) "value2"
127.0.0.1:6379> zrevrange  myZset 0 1 # 逆序输出某个范围区间的元素，0为start，1为stop
1) "value1"
2) "value2"
```



### Redis 单线程模型详解

Redis 内部使用文件事件处理器 `file event handler` ，这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 socket，将产生事件的 socket 压入内存队列中，事件分派器根据 socket 上的事件类型来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 socket
- IO 多路复用程序
- 文件事件分派器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将产生事件的 socket 放入队列中排队，事件分派器每次从队列中取出一个 socket，根据 socket 的事件类型交给对应的事件处理器进行处理。

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12a665a86cab41b4909ecb7464292bb9~tplv-k3u1fbpfcp-zoom-1.image" style="zoom: 67%;" />



### 为什么 Redis 单线程模型也能效率这么高？

1. 纯内存操作。
2. 核心是基于非阻塞的 IO 多路复用机制。
3. 单线程反而避免了多线程的频繁上下文切换问题，预防了多线程可能产生的竞争问题。
4. C 语言实现，一般来说，C 语言实现的程序“距离”操作系统更近，执行速度相对会更快。



### Redis 为什么不使用多线程？

1. 多线程就会存在线程上下文切换、死锁等问题，会影响性能。
2. **Redis 的性能瓶颈不在 CPU ，主要在内存和网络。**使用多线程模型带来的性能提升并不能抵消它带来的开发成本和维护成本。



### Redis6.0 之后为何引入了多线程？

**Redis6.0 引入多线程主要是为了提高网络 IO 读写性能**，因为这个算是 Redis 中的一个性能瓶颈。读写网络的 Read/Write 系统调用在 Redis 执行期间占用了大部分 CPU 时间，如果把网络读写做成多线程的方式对性能会有很大提升。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了， 执行命令仍然是单线程顺序执行。因此，也不需要担心线程安全问题。



### Redis 给缓存数据设置过期时间有什么用？

1.缓解内存消耗。如果缓存中的所有数据都是一直保存的话，会直接Out of memory。

2.很多时候业务场景就是需要某个数据只在某一时间段内存在，比如短信验证码可能只在1分钟内有效，用户登录的 token 只在 1 天内有效。



### 过期数据的删除策略

**定期删除** ： 每隔一段时间**随机抽取**一批设置了过期时间的 key 执行删除过期key操作（不是遍历所有的设置过期时间的 key，那样cpu 负载会很高）。对内存更加友好。

**惰性删除** ：只在取出key的时候才对数据进行过期检查，如果key过期就删除，不返回任何东西。这样对CPU最友好，但是可能会造成太多过期 key 没有被删除。

定期删除可能会导致很多过期 key 到了时间并没有被删除掉，所以要依靠惰性删除。

Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

如果定期删除漏掉了很多过期 key，然后也没及时去查，也就没走惰性删除，导致大量过期 key 堆积在内存里， Redis 内存块耗尽。这时就要依靠内存淘汰机制。



### Redis 内存淘汰机制

**高频面试题：MySQL 里有 2000w 数据，Redis 中只存 20w 的数据，如何保证 Redis 中的数据都是热点数据?**

Redis 提供 6 种数据淘汰策略：

1. **volatile-lru**：从**已设置过期时间**的数据集中挑选**最近最久未使用**的数据淘汰。
2. **volatile-ttl**：从**已设置过期时间**的数据集中挑选**将要过期**的数据淘汰。
3. **volatile-random**：从**已设置过期时间**的数据集中**任意**选择数据淘汰，一般不用。
4. **allkeys-lru**：**在键空间中，移除最近最久未使用的 key。这个是最常用的。**
5. **allkeys-random**：从数据集中任意选择数据淘汰。
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。一般不用。

4.0 版本后增加以下两种：

7. **volatile-lfu（least frequently used）**：从已设置过期时间的数据集中挑选**最不经常使用**的数据淘汰。
8. **allkeys-lfu**：在键空间中，移除最不经常使用的 key。



### Redis 持久化机制

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能。

**快照持久化RDB（Redis 默认采用的持久化方式）**

Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。可以将快照留在原地以便重启服务器的时候使用，也可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构）。

**触发机制（了解即可）**

1.满足save的条件时，会自动触发RDB。

<img src="C:\Users\黄睿楠\AppData\Roaming\Typora\typora-user-images\image-20220408131808578.png" alt="image-20220408131808578" style="zoom: 50%;" />

2.执行 flushall 命令，会触发RDB。

3.退出Redis，会产生RDB文件dump.rdb。

**适用场景：**适合大规模的数据恢复，并且对数据的完整性要不高。
**缺点：**如果Redis宕机，最后一次修改数据就会丢失。



**日志持久化AOF（append-only file）**

以日志的形式来记录Redis执行过的每个写操作，Redis重启时就根据日志文件的内容将写指令按顺序执行一次，来完成数据的恢复工作。

在 Redis 的配置文件中存在三种不同的 AOF 持久化方式，它们分别是：

```conf
appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟同步一次
appendfsync no        #让操作系统决定何时进行同步
```

为了兼顾数据和写入性能，用户可以考虑 appendfsync everysec 选项 ，让 Redis 每秒同步一次 AOF 文件，Redis 性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。

**缺点：**aof的文件大小远**大于**rdb，修复的速度也比 rdb慢。



**拓展**

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化。

如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点, 快速加载同时避免丢失过多的数据。



### Redis 事务

Redis 可以通过 **MULTI，EXEC，DISCARD 和 WATCH**  等命令来实现事务功能。

使用MULTI命令后可以输入多个命令。Redis不会立即执行这些命令，而是将它们放到队列，当调用了EXEC命令将执行所有命令。

可以将Redis中的事务就理解为 ：**Redis事务提供了一种将多个命令请求打包的功能。然后，再按顺序执行打包的所有命令，并且不会被中途打断。**

**Redis单条命令式保证原子性，但是事务不保证原子性。**

编译型异常（ 命令有错） ，事务中所有的命令都不会被执行。


```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> getset k3 # 错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> set k5 v5
QUEUED
127.0.0.1:6379> exec # 执行事务报错！
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k5 # 所有的命令都不会被执行！
(nil)
```

运行时异常， 如果事务队列中存在语法性问题，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常（因此Redis事务不保证原子性）。

```bash
127.0.0.1:6379> set k1 "v1"
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1 # 会执行的时候失败！
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> get k3
QUEUED
127.0.0.1:6379> exec
1) (error) ERR value is not an integer or out of range # 虽然第一条命令报错了，但是其他命令依旧正常执行
2) OK
3) OK
4) "v3"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379> get k3
"v3"
```



### 缓存穿透（查不到）

大量请求的 key 不在缓存中，导致直接请求数据库。


#### 解决办法

**1）缓存无效 key**

当数据库未命中后，即使返回空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将从缓存中获取。如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。

**2）布隆过滤器（重点）**

**通过布隆过滤器，可以非常方便地判断一个给定数据是否存在于海量数据中。**

把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会查询Redis和数据库。

**布隆过滤器判断某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**

原理：

当一个元素加入布隆过滤器中时：

1. 使用**布隆过滤器中的哈希函数**对元素值进行计算，得到哈希值。
2. 根据得到的哈希值，在**位数组**中把对应下标的值置为 1。

判断一个元素是否存在于布隆过滤器时：

1. 对给定元素再次进行相同的哈希计算；
2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

**不同的字符串可能哈希出来的位置相同，所以可能误判。** 



### 缓存击穿

指**一个key**非常热点，集中对这一个点进行高并发访问，这个key在失效的瞬间，持续的高并发就击穿缓存，直接请求数据库。

#### 解决办法

1. 若缓存的数据基本不会发生更新，可以**设置热点数据永不过期。**
2. 若缓存的数据更新不频繁，且缓存刷新的整个流程耗时较少，可以加**分布式锁：**保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，只能等待。
3. 若缓存的数据更新频繁或者缓存刷新的流程耗时较长，可以利用**定时线程在缓存过期前主动地重新构建缓存或者延后缓存的过期时间，以保证所有的请求能一直访问到对应的缓存。**



### 缓存雪崩

**缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求。**

例如：

1.系统的缓存模块出了问题，比如宕机，所有访问都要走数据库。

**2.有一些被大量访问数据（热点缓存）在某一时刻大面积失效，导致对应的请求直接落到了数据库上。** 

例如，秒杀开始 12 个小时之前，存放了一批商品到 Redis 中，设置的缓存过期时间也是 12 个小时，那么秒杀开始的时候，这些商品的缓存过期。导致相应的请求直接访问数据库。

#### 解决办法

**1.针对 Redis 服务不可用的情况：**

​	采用 主从+哨兵或者Redis 集群。

​	限流。通过加锁或者队列来控制读数据库写缓存的线程数量。比如对于某个key，只允许一个线	程查询数据和	写缓存，其他线程等待。或者采用 hystrix 实现限流和降级。

**2.针对热点缓存失效的情况：**设置不同的失效时间比如随机设置缓存的失效时间。

**3.秒杀刚开始时，缓存为空，将有大量请求直接访问数据库，该怎样避免这种情况？**

**数据预热：**在秒杀开始之前，手动把可能访问的数据预先访问一遍，加载到缓存中。



### Redis 主从架构（读高并发）

[![Redis-master-slave](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-master-slave.png)](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/images/redis-master-slave.png)

#### Redis主从复制的核心机制

- Redis 采用**异步方式**复制数据到 slave 节点；
- 一个 master node 可以配置多个 slave node，slave node 也可以连接其他的 slave node；
- slave node 做复制的时候，不会 block master node 的正常工作，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了；
- slave node 主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。



redis 实现高并发主要依靠主从架构，单主用来写入数据，单机几万 QPS，多从用来查询数据，多个从实例可以提供每秒 10w 的 QPS。如果想要在实现高并发的同时，容纳大量的数据，那么就需要 redis 集群，使用 redis 集群之后，可以提供每秒几十万的读写并发。



### 主备切换+哨兵模式（高可用）

一个 slave 挂掉了不会影响可用性，还有其它的 slave 在提供相同数据下的查询服务。但是，如果 master node 挂掉就无法写数据了。

哨兵（Sentinel）有两个作用：

1. 通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis服务器（主机和从机都监控）。
2. 当哨兵监测到master宕机时，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服
   务器切换主机。

<img src="https://i.loli.net/2020/12/11/sYjzKWD3mJ9C7UB.png" alt="image.png" style="zoom: 67%;" />



**多哨兵模式：**各个哨兵之间还会进行监控。

<img src="https://i.loli.net/2020/12/11/RLzoX5jQ4WbqkNM.png" alt="image.png" style="zoom:67%;" />


假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行故障转移操作，当后面的哨兵也检测到主服务器不可用，当数量超过总哨兵数的一半时，哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行故障转移操作。

如果主机先断开后重连回来，只能归到新的主机下，当做从机。

哨兵模式**不保证数据零丢失**，只能保证 Redis 集群的高可用性。



### Redis 哨兵主备切换的数据丢失问题

#### 导致数据丢失的两种情况

主备切换的过程，可能会导致数据丢失：

- 异步复制导致的数据丢失

因为 master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。

[![async-replication-data-lose-case](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/async-replication-data-lose-case.png)](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/images/async-replication-data-lose-case.png)

- 脑裂导致的数据丢失

脑裂，也就是说，某个 master 所在机器突然**脱离了正常的网络**，跟其他 slave 机器不能连接，但是实际上 master 还运行着。此时哨兵可能就会**认为** master 宕机了，然后开启选举，将其他 slave 切换成了 master。这个时候，集群里就会有两个 master ，也就是所谓的**脑裂**。

此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧 master 写数据。因此旧 master 再次恢复的时候，会被作为一个 slave 挂到新的 master 上去，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。

[![Redis-cluster-split-brain](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-cluster-split-brain.png)](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/images/redis-cluster-split-brain.png)

狂神说Redis视频重点：P22   Redis实现乐观锁



[缓存与数据库的双写一致性](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/redis-consistence.md)
