## RPC基本原理

<img src="https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JJjARRqcZibY4ZPv60renshVx3xhf4RyUVtia7Tvo4BBs70SFKRonhrPrNsiap2rEAQCn4IWUoS3HZA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1" alt="图片" style="zoom:50%;" />

**步骤解析：**

<a href="https://sm.ms/image/w4iLud2KmZRn1co" target="_blank"><img src="https://s2.loli.net/2022/04/26/w4iLud2KmZRn1co.png" style="zoom:50%;"  ></a>

1. 一次完整的RPC调用流程（同步调用，异步另说）如下：

2. 服务消费方（client）调用以本地调用方式调用服务；
3. client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
4. client stub找到服务地址，并将消息发送到服务端；
5. server stub收到消息后进行解码；
6. server stub根据解码结果调用本地的服务；
7. 本地服务执行并将结果返回给server stub；
8. server stub将返回结果打包成消息并发送至消费方；
9. client stub接收到消息，并进行解码；
10. 服务消费方得到最终结果。

RPC框架的目标就是要2~8这些步骤都封装起来，这些细节对用户来说是透明的，不可见的。

RPC两个核心模块：通讯，序列化。



## Dubbo

Apache Dubbo 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心功能：**面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JJjARRqcZibY4ZPv60renshLSMRQe7NJpvDFrQMChLxI3BqIYQXrZvfs28iadQ1dDB4p84ydyb3KtQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**服务提供者**（Provider）：暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。

**服务消费者**（Consumer）：调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

**注册中心**（Registry）：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

**监控中心**（Monitor）：服务消费者和提供者在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。