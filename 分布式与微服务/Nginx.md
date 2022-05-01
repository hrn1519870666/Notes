### Nginx的作用

Http代理，反向代理：作为web服务器最常用的功能之一，尤其是反向代理。

正向代理
![img](https://img-blog.csdnimg.cn/img_convert/a716f56bc25ee06ecd80e017e4a620d6.png)
反向代理
![img](https://img-blog.csdnimg.cn/img_convert/c055144e73737cb79afcadb23f1b5268.png)

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。

iphash：对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。

<img src="https://img-blog.csdnimg.cn/img_convert/337904e2f9489dc8aba62581515f2f42.png" alt="img" style="zoom: 80%;" />

动静分离，在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件），这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源响应的速度。

![img](https://img-blog.csdnimg.cn/img_convert/2bf9ae957ad281e4bda087d2147f0970.png)

