## RPC基本原理

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JJjARRqcZibY4ZPv60renshVx3xhf4RyUVtia7Tvo4BBs70SFKRonhrPrNsiap2rEAQCn4IWUoS3HZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

**步骤解析：**

1. 一次完整的RPC调用流程（同步调用，异步另说）如下：

2. 服务消费方（client）**以本地调用方式调用服务；**
3. client stub接收到调用后，**将方法、参数等组装成能够进行网络传输的消息体；**
4. client stub**找到服务地址，并将消息发送到服务端；**
5. server stub收到消息后进行**解码；**
6. server stub根据解码结果**调用本地的服务；**
7. **本地服务执行并将结果返回给server stub；**
8. server stub将返回结果打包成消息并发送至消费方；
9. client stub接收到消息，并进行解码；
10. 服务消费方得到最终结果。

RPC框架的目标就是要2~9这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。

**RPC两个核心模块：通讯，序列化。**



## Dubbo

Apache Dubbo 是一款**高性能、轻量级的开源Java RPC框架，**它提供了三大核心功能：**面向接口的远程方法调用，负载均衡和智能容错，以及服务自动注册和发现。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JJjARRqcZibY4ZPv60renshLSMRQe7NJpvDFrQMChLxI3BqIYQXrZvfs28iadQ1dDB4p84ydyb3KtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**服务提供者**（Provider）：暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。

**服务消费者**（Consumer）：调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，从提供者地址列表中，基于负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

**注册中心**（Registry）：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

**监控中心**（Monitor）：服务消费者和提供者在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

高可用：

- 监控中心宕掉不影响使用，只是丢失部分采样数据。
- 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务。
- 注册中心对等集群，任意一台宕掉后，将自动切换到另一台。
- 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯。
- 服务提供者无状态，任意一台宕掉后，不影响使用。
- 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复。



### 负载均衡

在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用。

1. Random LoadBalance
   随机，按权重设置随机概率。
2. RoundRobin LoadBalance
   轮循，按公约后的权重设置轮循比率。
   存在慢的提供者累积请求的问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。
3. LeastActive LoadBalance
   最少活跃调用数，活跃数指调用前后计数差，如果活跃数相同则随机。
   使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。
4. ConsistentHash LoadBalance
   一致性 Hash，相同参数的请求总是发到同一提供者。
   当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。



### 服务降级

向注册中心写入动态配置覆盖规则：

```xml
RegistryFactory registryFactory = ExtensionLoader.getExtensionLoader(RegistryFactory.class).getAdaptiveExtension();
Registry registry = registryFactory.getRegistry(URL.valueOf("zookeeper://10.20.153.10:2181"));
registry.register(URL.valueOf("override://0.0.0.0/com.foo.BarService?category=configurators&dynamic=false&application=foo&mock=force:return+null"));
```

其中：

mock=force:return+null 表示消费方对该服务的方法调用都直接返回 null 值，不发起远程调用。用来屏蔽不重要服务不可用时对调用方的影响。

还可以改为 mock=fail:return+null 表示消费方对该服务的方法调用在失败后，再返回 null 值，不抛异常。用来容忍不重要服务不稳定时对调用方的影响。



### 集群容错

在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。

集群容错模式：

Failover Cluster
失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries=“2” 来设置重试次数(不含第一次)。

```xml
重试次数配置如下：
<dubbo:service retries="2" />
或
<dubbo:reference retries="2" />
或
<dubbo:reference>
    <dubbo:method name="findFoo" retries="2" />
</dubbo:reference>
```



Failfast Cluster
快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。

Failsafe Cluster
失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。

Failback Cluster
失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。

Forking Cluster
并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks=“2” 来设置最大并行数。

Broadcast Cluster
广播调用所有提供者，逐个调用，任意一台报错则报错 [2]。通常用于通知所有提供者更新缓存或日志等本地资源信息。



