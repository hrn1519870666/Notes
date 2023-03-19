## HTTP 1.0 vs HTTP 1.1

**连接方式** : HTTP 1.0 为短连接，HTTP 1.1 支持长连接。

**状态响应码** : HTTP/1.1中新加入了大量的状态码。

100 (Continue)——在请求大资源前的预热请求

该状态码的使用场景：存在某些较大的文件请求，服务器可能不愿意响应这种请求，此时状态码`100`可以作为指示请求是否会被正常响应，过程如下图：

![HTTP1.1continue1](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAYUAAADSCAMAAACSASyBAAAA4VBMVEX///8AAADg4OAfHx+enp6goKDnplIAgsb/46Wl4////8ZeXl7G//9SpucAVaViYmLGggD/x4SEx//n/////+elVQCEAAAcHBwAAISjo6NSAFJSAAAAAFJSAISEAFLAwMA6OjqEAISlVVLv7+8WFhZSpsjn46VSVaXnx4R+fn7n4//n/+fG/8aEwefG46XGx4TGglKl4+cAgqWZh5HGgoSlVYSlggCEggBSVQDn5Of/4+fXzcyP0cZSgsalyqVSgoQAVVLnx6WEpqXnpoTGplJRUlHGx//n/8aEVaWlpoSEglJdQquRAAALx0lEQVR42uydiZbSMBSGE0ebKZIx7QCDgo77vu/77jke3/+BzL1Z2g7IMuAQpv93BFq6eMyXm7SJ3AoAAAAAAAAAAAAAAAAAAAAAAAAAAPA/MVo02b3cjcuZDgt7+wLUMesskd4PqY9YGKlq+VIuRDHskoX2msikRXcG0sHFU0jdLDRpGXbd5zhf4txNyrhhnNeNw4L9h/f61kIZGg8u9YiqVVf6pJVlzi06g2CU/w4jAyzWNU+w0LBgl6i0uNby6jhf0UJnMM4bFpSIFCG8VCE9sMBRUNrX3v7uyMXEsLtSi8RG6xV/wkJYbHsPzW03WzC0QOHApTf+OJC0tEosiCJW74zCa9ICtVcko+3EWJisw0dKx2j79VKVlXt5Y8u/oHNNs0Dr2d5dyfhIayPBQmyYq44hIxGxs67QYgn4+MqnkTVcn+zEtDskgoWPHA1GxTJjilg7d0e3+sdotQtJRjPS67v/RizQy6iwpb0EC4/c5VEZvlZHdrPFSO3Rsl2OqkWE1dssa+5znvV1r1+7pWglsXfm63VbO6lIKnw99cVjlr2ULBotftRcwUFo3bi+qTNoq4XQO/OfUFPrd3Gm3k8XbGZhuLGvLrT4vYIFf7mUw0K0YEvsTSwzE5vxVe4XJi1QwE3uAgtGCW/BtjtlqKDDrl8oV7pfaFpgq9Maredtt0Dl4i2Y8VtXU01sc4pVRjAm+wXqd6bd1rU+Fmw5ewuG7qFkaV+6urLXYo0tEofWJFWImNbetKnOQI5z+8ZxMHkxOhkL7SwoAAAAAAAAAAAAAAAAAABsBRe2GXFaODi/veyI08LBRbGtnIWFBICFFICFFICFFICFFICFFICFFICFFICFFICFFICFFICFFICFFICFY7D7aUo+rUwJo2o/hjEKFmam/3EFZaQjZnQY51ktW4Ep7VvnSt4oZ8f716rKcDDshq29+8ad/J2RChYWs1D/JW+vr+NGTnUjOte6nGWlspDJWK6967mPhSqRTVTpvithYTULXI0LesmvtENI/vRWiQojPboRKZUco2FhZQuZtl+WvANTUB6UmB1CNfqFTPu0cpzJrPdtJEtYWNVCIcvQ5lQWWE7VQ2QyUtqjlWuwCnnrfvxpPCzMT5E43ULMcbJ7Z3+mhZBelD+4E8+e96lTCMDCarHAFErMsTCqbPIJpLJdMi81O2tYOL4FU4olYkGYcc5mqvpvlSAWVrLA16lLxIKRJX0XfoUfsxO1xsKHdVuId8ezLRTfYyxwrWcLVQLeQlJmTVo+SQsfxIY43Dn3PyxkeiI1TcOCVI0WyVuoNnP5c5bTk7Rw7+qh2AxX5cG5Ex1HYmclFT/fNasqYWBsi3Tlg2SfmIUbUm7Iw6GU1gPGVJl7clMerkr2AAsuGNbr4d25BSH/7GGbLdw8tx5+yfV6ONxZkAPp2Lm5xRbkznq4Kb2GM+JkcbFwcBYtkmueN9I13HAOFumdi7Io/ZiopiFSfXS4YW9/1iwaX942r03rx5cJWDjcgAPmnncw30LvVfdpTsXGZXnnwbeRioPWVOzzZ9He0fxPJh2kYO9Bv6y8TMLmHl/KT8rC1Q1dIt24SQ4Ws/Bi/6V/mgmPmdJQj33VR97mz6Lx0QXlseRIetQvszk53V92f5yUhUNysAnOkoMFLfRv9XWwQG9+wMjHAg1Pz5xFcw7+5CSntI0THdqzFpTXNp3O7wevuid173woNs5CvXMhG5RNC7Nn0Th9731rSfOo3fuf47wXY4G/o1HVjCfahg/9aqF3H5360bxlLbhYqAZNmy3S7Fm0ekZlGqqz+h5UsUBFzrubYbcY5361HWOqx7LwcDB+dj2nPoBs+DE9ioU5s2iVM1onarFgj6XM47TTl5EWfhUWppP5OYGyM3h+PzY/i8yixQhiOXxg+Ax7lmFY26+2ZH5hSQvUTlAlZxlqspuYM4vGFjLFFvjN7mQjymhTukjRbJIF8ios/ItgoeQR6tAe0Q3B3Fk0tmA0C+AT2Y7dbjXOrBk+GefuYThK+FVYmGEhk+O3fFXk7n6Hj0dqgVk0skBVXIU+pNen8icvheJM8IbuJEhkGVZh4V8W/FVkSbduBHewaoFZNG7CtGvNKChsST/NXXQ8/TzSFDLKDLukLKzCwjQMl2AYTqo9sKCQ43zmLFrjYTeFVDYejPYHpjKat3kwppoCsJACsJACsJACsJACsJACsJACsJACsJACsJACsJACsJACsJACsJACsJACBxfPbCunyYLcXk6PhTPbjAAAAAAAAAAAAAAAAAAAAGgrZ7cZcVo42NlixGkB/wcjBWAhBWAhBWAhBWAhBdKwYDMCwMIG4IxvFdmwCwsL4tO9UFKXkXQ86kuH6gyqNFUuY0mmZtR2U8bEGhUaFpaxEJ+sEPLW8saYMinTjQfm1Z+kR7JUETPiUhoT9Atrt7B7iXPrNR+Yx0/Si0GRKd8cFdGC0XwCWFifhZBvNezAT9L7t4XH1kOm7No4h4U1WOBttmQLJfwOkaMWbo/oiaxs4eGgpFPQk2VgYSEL0jHdQmjqhSnnW6D0eWzhzr7RtNm6Rb+wjlioHl67sAVqlWwQ2IPtBlhYnwWq3ktY4Cyg9pRWByys0UKmxeL9gtF8pFFG40p1nRae5rMsDKR0scDXSO6iVrMdWFifhSqjs1Fxo/jXlSov+WVY+Nveua22DQRhWOvSXZx4zUjErVOoHXBaaBtIQ5Le9KYXPdD3f6HuP3tA0ZrESiyzcufHlrVmpYv5NHuaFXNYsclJKeNmEnCOKCMUxiahUIKEQgkSCiVIKJSg4ils/jcKm6o4rS8mA1HAqP8xnfxuFb73mACQeQmFyWlxmyXXF+p0mBYJkZvpU7O6JD0/CAUwUG+rsuQYKDUZiIL+8a44Co6BUmW5AjOAKwxAIWbX7CS4C+ZH2B9HztCDH41EGEwCB90K7ddhkkyo4X/n30zFF5qeFJiBuqwKEhhA68nuujzvs4qEfiFLcBcWlsigBr4wdMPGjxSwmIRrPQRUsgjKcdoejTCb8hl6cMMeFMAAup3sR5s95Wzur90pkGFL5gnuYNTIKQJ7SAGW7jQ+uBXwoSpbn6z/t0euc7VnfSrfF8LznCe4S0n1UuhZqY4voK2yVfSb1kKsNv6WZPgqtFEj94WB+wVSLLs1wR0K6BganJjMFzwom1OgRCH40/j7hcHHSN4Xtie4g+XdB9ZFlRYFYgpM45EWyX2OZYwU5wtDUsgT3HG/APM7o+Jv+APjQE/s6vp+ITkB0JHJemfUReEY5gvDz53zBHd49kOKYeUszrtXjUfhCvOfiC6nEE4+UtVhpFrzLtcjmTvLOtL4dCQURi6hUIKEQgkSCiXo8BSwIVIovJgClijmf1bTsKmoLettXHd3F2mcRrXTaGueZwuF8x7Gt26+oLFIxPuyJ4vlLDcsU0ApmZ4rx6lCSzxn5jl2DEh8WVihsBMFNl0wLJluJJRdY3mNY4w1WK1iMeBJQjltzWuEwm4U8I4B//LhAZ90rp09oy/AMcgGn9CBAi6N3Mjik7avViQUntTX6dn91RvVlgmWT8/4ye37q9VfxdpCIW6Wx4IRB3zwrgPYkqlqW+lGKOyk6AbUtAY917wTHrrDu4JdX6iBxFP4iJgpSJgUoAAF6Z2fQYGU1XigbTAitsCzRWvbGiMlCnhrylMwQFhzJAKYgJKMu4EDIxT6CD1uA4NWtLwJ0U1nZr++ejdrjZFyCtqEuClqnd3PqEHBUXA3EQp9HMGNZBqGESNuNvW6+KM9J8spWD8salDwzZo2oV8QCv1ABAr8bGMvTKCAIoJnqykZ/0pPt1/AlfxO7YdFEyiAgPvib6HQkwI84Rf6WJgzUoDQ5K9uPhPaf3eO1sv7AhABDDdgoBMoYLrHvTMZodCTAvm9Q3FODAoJENuUj2Q7I1WcQemEZIz0PAp1DEtioLn11UDKF5FQV6GtgnNEWU+BlLuNUBirhEIJEgolSCiUIKFQgoRCCTomCq9GrOpY9HrMqkQikUgkEolEIpFIJOroHxA0Ou5lMYdmAAAAAElFTkSuQmCC)

![HTTP1.1continue2](https://javaguide.cn/assets/HTTP1.1continue2.7d63532b.png)

206 (Partial Content)——范围请求的标识码

**带宽优化** :HTTP1.0 中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1 则在请求头引入了 range 头域，它允许只请求资源的某个部分，即返回码是 206（Partial Content）。

**Host头处理** : HTTP/1.1在请求头中加入了`Host`字段。

假设有一个资源URL是http://example.com/home.html，HTTP/1.0的请求报文中，将会请求的是`GET /home.html HTTP/1.0`.也就是不会加入主机名。这样的报文送到服务器端，服务器是理解不了客户端想请求的真正网址。

因此，HTTP/1.1在请求头中加入了`Host`字段。加入`Host`字段的报文头部将会是:

```text
GET /home.html HTTP/1.1
Host: example.com
```

这样，服务器端就可以确定客户端想要请求的网址了。

**缓存处理** : 在 HTTP1.0 中主要使用 header 里的 If-Modified-Since,Expires 来做为缓存判断的标准，HTTP1.1 则引入了更多的缓存控制策略。

HTTP1.0缓存机制：JavaGuide



## HTTP/2.0

### 二进制分帧

HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/86e6a91d-a285-447a-9345-c5484b8d0c47.png" width="400"/> </div><br>

在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

- 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
- 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。
- 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/af198da1-2480-4043-b07f-a3b91a88b815.png" width="600"/> </div><br>



### 首部压缩

HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。

HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。

不仅如此，HTTP/2.0 也使用 Huffman 编码对首部字段进行压缩。

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/_u4E0B_u8F7D.png" width="600"/> </div><br>



## HTTPS

HTTP 有以下安全性问题：

- 使用**明文通信，**内容可能会被窃听；
- 不验证通信方的**身份，**通信方的身份有可能遭遇伪装；

通过使用 SSL，HTTPS 能**加密**、**认证**。



#### HTTP与HTTPS之间的区别

|          HTTP          |                  HTTPS                   |
| :--------------------: | :--------------------------------------: |
|       默认端口80       |           HTTPS默认使用端口443           |
|   明文传输、安全性差   |           SSL加密、安全性较好            |
| 响应速度快、消耗资源少 | 响应速度较慢、消耗资源多、需要用到CA证书 |



#### HTTPS 采用的加密方式

- 使用 公钥 加密 密钥，从而保证 密钥 安全性;
- 获取到 密钥 后，再使用对称加密进行通信，从而保证效率。



#### HTTPS连接建立的过程：

设有服务器 S，客户端 C，和第三方信赖机构 CA。

1. S 信任 CA，CA 知道 S 公钥，向 S 颁发证书。并附上 **CA 私钥对消息摘要（证书内容的哈希）的加密签名**。证书内容包括：**CA的签名、服务器的公钥**、证书的发布机构、有效期、所有者。
2. S 获得 CA 颁发的证书，将该证书传递给 C
3. C 获得 S 的证书，信任 CA 并知晓 CA 公钥，使用 CA 公钥对 S 证书上的签名解密，同时对消息进行散列处理，得到摘要。比较摘要，验证 S 证书的真实性。如果 C 验证 S 证书是真实的，则信任 S 的公钥（在 S 证书中）
4. **使用服务端的公钥给对称密钥加密（对称密钥是客户端发给服务端的）**
5. 服务器端使用私钥进行解密并使用对称密钥加密确认信息发送给客户端

6. 随后客户端和服务端就使用对称密钥进行**信息传输**



### 认证（CA证书）

中间人攻击的场景：

假设 S 公钥在信道中传输，那么很有可能存在一个攻击者 A，发送给 C 一个诈包，假装是 S 公钥，其实是诱饵服务器 AS 的公钥。当 C 收获了 AS 的公钥（却以为是 S 的公钥），C 后续就会使用 AS 公钥对数据进行加密，并在公开信道传输，那么 A 将捕获这些加密包，用 AS 的私钥解密，就截获了 C 本要给 S 发送的内容。

为了解决该问题，引入证书颁发机构（CA，Certificate Authority）。CA 默认是受信任的第三方。CA 会给各个服务器颁发证书，证书存储在服务器上，并附有 CA 的**数字签名**。

当客户端向服务器发送 HTTPS 请求时，先获取目标服务器的证书，并根据证书上的信息，检验证书的合法性。证书上又包含着服务器的公钥信息，客户端就可以放心的**信任证书上的公钥就是目标服务器的公钥。**



### 数字签名

数字签名要解决的问题，是防止证书被伪造。第三方信赖机构 CA 之所以能被信赖，就是 **靠数字签名技术** 。

数字签名，是 CA 在给服务器颁发证书时，使用哈希+加密的组合技术，在证书上盖个章，以此来提供验伪的功能。具体行为如下：

CA 知道服务器的公钥，对证书采用哈希技术生成一个摘要。CA 使用 **CA 私钥**对该摘要进行加密，并附在证书下方，发送给服务器。

现在服务器将该证书发送给客户端，客户端需要验证该证书的身份。**客户端找到第三方机构 CA，获知 CA 的公钥**，并用 CA 公钥对证书的签名进行解密，获得了 CA 生成的摘要。

客户端再对证书数据做相同的哈希处理，得到摘要，并将该摘要与之前从签名中解码出的摘要做对比，如果相同，则身份验证成功。



### HTTPS 的缺点

- 因为需要进行加密解密等过程，因此速度会更慢；
- 需要支付证书授权的高额费用。



## GET 和 POST 的区别

### 参数

`GET`请求的参数放在`url`中，`POST`则放在`body`中（这里只是约定，并不属于`HTTP`规范，相反的，我们可以在`POST`请求中`url`中写入参数，或者`GET`请求中的`body`携带参数）。

```
GET /test/demo_form.asp?name1=value1&name2=value2 HTTP/1.1
```

```
POST /test/demo_form.asp HTTP/1.1
Host: w3schools.com
name1=value1&name2=value2
```

### 安全

`POST `比` GET` 安全，因为数据在地址栏上不可见。然而，从传输的角度来说，他们都是不安全的，因为` HTTP` 在网络上是明文传输的，只要在网络节点上抓包，就能完整地获取数据报文。只有使用`HTTPS`才能加密安全。

### 幂等性

幂等的 HTTP 方法，同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用。

GET，HEAD，PUT 和 DELETE 等方法都是幂等的，而 POST 方法不是。

GET 连续调用多次，客户端接收到的结果都是一样的。

POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录：

```
POST /add_row HTTP/1.1   -> Adds a 1nd row
POST /add_row HTTP/1.1   -> Adds a 2nd row
POST /add_row HTTP/1.1   -> Adds a 3rd row
```

DELETE /idX/delete HTTP/1.1 是幂等的，即使不同的请求接收到的状态码不一样：

```
DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
DELETE /idX/delete HTTP/1.1   -> Returns 404
```



## 状态码

服务器返回的**响应报文**中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 |               类别               |            含义            |
| :----: | :------------------------------: | :------------------------: |
|  1XX   |  Informational（信息性状态码）   |     接收的请求正在处理     |
|  2XX   |      Success（成功状态码）       |      请求正常处理完毕      |
|  3XX   |   Redirection（重定向状态码）    | 需要进行附加操作以完成请求 |
|  4XX   | Client Error（客户端错误状态码） |     服务器无法处理请求     |
|  5XX   | Server Error（服务器错误状态码） |     服务器处理请求出错     |

### 1XX 信息

-   **100 Continue**  ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

### 2XX 成功

-   **200 OK**  

-   204 No Content  ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。

-   206 Partial Content  ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

### 3XX 重定向

-   301 Moved Permanently  ：永久性重定向

-   302 Found ：临时性重定向

-   303 See Other  ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

-   注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

-   304 Not Modified  ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。

-   307 Temporary Redirect  ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

### 4XX 客户端错误

-   **400 Bad Request**  ：请求报文中存在语法错误。

-   **401 Unauthorized**  ：该状态码表示发送的请求需要有认证信息。如果之前已进行过一次请求，则表示用户认证失败。

-   **403 Forbidden**  ：请求被拒绝。

-   **404 Not Found**  

### 5XX 服务器错误

-   **500 Internal Server Error**  ：服务器正在执行请求时发生错误。

-   503 Service Unavailable  ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。
