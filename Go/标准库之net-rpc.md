## 简介

RPC（Remote Procedure Call）**远程方法调用，**它可以通过网络调用远程对象的方法。Go 标准库`net/rpc`提供了一个简单、强大且高性能的 RPC 实现。



## 同步调用

由于是网络程序，我们需要编写服务端和客户端两个程序。

服务端：

```golang
package main

import (
  "errors"
  "log"
  "net"
  "net/http"
  "net/rpc"
)

type Args struct {
  A, B int
}

//带余除法，Rem是余数
type Quotient struct {
  Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
  *reply = args.A * args.B
  return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
  if args.B == 0 {
    return errors.New("divide by 0")
  }

  quo.Quo = args.A / args.B
  quo.Rem = args.A % args.B
  return nil
}

func main() {
  arith := new(Arith)
  rpc.Register(arith)
  rpc.HandleHTTP()
  if err := http.ListenAndServe(":1234", nil); err != nil {
    log.Fatal("serve error:", err)
  }
}
```

我们定义了一个`Arith`类型，为它编写了两个方法`Multiply`和`Divide`（这两个方法的接收者类型是`Arith`）。**在main函数中，创建`Arith`类型的对象`arith`，调用`rpc.Register(arith)`会注册这两个方法。`rpc`库注册的方法必须满足签名`func (t *T) MethodName(argType T1, replyType *T2) error`：**

- **首先，方法必须是导出的（名字首字母大写）；**
- **其次，方法接受两个参数，必须是导出的或内置类型。第一个参数表示客户端传递过来的请求参数，第二个是需要返回给客户端的响应。第二个参数必须为指针类型（需要修改）；**
- **最后，方法必须返回一个`error`类型的值。返回非`nil`的值，表示调用出错。**

**`rpc.HandleHTTP()`注册 HTTP 路由。`http.ListenAndServe(":1234", nil)`在端口`1234`上启动一个 HTTP 服务，请求 rpc 方法会交给`rpc`内部路由处理。**

客户端调用这两个方法：

```golang
package main

import (
  "fmt"
  "log"
  "net/rpc"
)

type Args struct {
  A, B int
}

type Quotient struct {
  Quo, Rem int
}

func main() {
  client, err := rpc.DialHTTP("tcp", ":1234")
  if err != nil {
    log.Fatal("dialing:", err)
  }

  args := &Args{7, 8}
  var reply int
  err = client.Call("Arith.Multiply", args, &reply)
  if err != nil {
    log.Fatal("Multiply error:", err)
  }
  fmt.Printf("Multiply: %d*%d=%d\n", args.A, args.B, reply)

  args = &Args{15, 6}
  var quo Quotient
  err = client.Call("Arith.Divide", args, &quo)
  if err != nil {
    log.Fatal("Divide error:", err)
  }
  fmt.Printf("Divide: %d/%d=%d...%d\n", args.A, args.B, quo.Quo, quo.Rem)
}
```

**使用`rpc.DialHTTP("tcp", ":1234")`连接到服务端的监听地址，返回一个 rpc 的客户端对象。后续就可以调用该对象的`Call()`方法调用服务端对象的对应方法，依次传入方法名（需要加上类型限定）、参数、一个指针（用于接收返回值）。**



## 异步调用

同步的调用方式一直等待服务端的响应或出错。在等待的过程中，客户端不能处理其它的任务。

我们也可以采用异步的调用方式，客户端：

```golang
func main() {
  client, err := rpc.DialHTTP("tcp", ":1234")
  if err != nil {
    log.Fatal("dialing:", err)
  }

  args1 := &Args{7, 8}
  var reply int
  multiplyReply := client.Go("Arith.Multiply", args1, &reply, nil)

  args2 := &Args{15, 6}
  var quo Quotient
  divideReply := client.Go("Arith.Divide", args2, &quo, nil)

  // 返回一个新的定时器Ticker
  // 该 Ticker 包含一个通道字段，并会每隔时间段 d 就向该通道发送当时的时间。向其自身的通道字段C发送当时的时间。
  ticker := time.NewTicker(time.Millisecond)
  defer ticker.Stop()

  var multiplyReplied, divideReplied bool
  // 只要multiplyReplied和divideReplied有一个为假，则循环
  for !multiplyReplied || !divideReplied {
    select {
    case replyCall := <-multiplyReply.Done:
      if err := replyCall.Error; err != nil {
        fmt.Println("Multiply error:", err)
      } else {
        fmt.Printf("Multiply: %d*%d=%d\n", args1.A, args1.B, reply)
      }
      multiplyReplied = true
    case replyCall := <-divideReply.Done:
      if err := replyCall.Error; err != nil {
        fmt.Println("Divide error:", err)
      } else {
        fmt.Printf("Divide: %d/%d=%d...%d\n", args2.A, args2.B, quo.Quo, quo.Rem)
      }
      divideReplied = true
    case <-ticker.C:
      fmt.Println("tick")
    }
  }
}
```

异步调用使用`client.Go()`方法，参数与同步调用基本一样。它返回一个`rpc.Call`对象：

```golang
// src/net/rpc/client.go
type Call struct {
  ServiceMethod string     
  Args          interface{}
  Reply         interface{}
  Error         error      
  Done          chan *Call 
}
```

我们可以通过该对象获取此次调用的信息，如方法名、参数、返回值和错误。**我们通过监听通道`Done`是否有值判断调用是否完成。**上面代码中使用一个`select`语句轮询两次调用的状态。



### 定制方法名

默认情况下，`rpc.Register()`**将调用的方法的接收者类型名作为方法名前缀。**我们也可以自己设置。这时需要调用`RegisterName(name string, rcvr interface{}) error`方法，服务端：

```golang
rpc.RegisterName("math", arith)
```

上面我们将注册的方法名前缀改为`math`了，客户端调用时传入的方法名也需要相应的修改：

```golang
err = client.Call("math.Multiply", args, &reply)
```



### TCP

上面我们都是使用 HTTP 协议来实现 rpc 服务的，`rpc`库也支持直接使用 TCP 协议。首先，服务端先调用`net.Listen("tcp", ":1234")`创建一个监听某个 TCP 端口的监听器（Accepter），然后使用`rpc.Accept(l)`在此监听器上接受连接并处理：

```golang
func main() {
  l, err := net.Listen("tcp", ":1234")
  if err != nil {
    log.Fatal("listen error:", err)
  }

  arith := new(Arith)
  rpc.Register(arith)
  rpc.Accept(l)
}
```

然后，客户端调用`rpc.Dial()`以 TCP 协议连接到服务端：

```golang
func main() {
  client, err := rpc.Dial("tcp", ":1234")
  if err != nil {
    log.Fatal("dialing:", err)
  }

  args := &Args{7, 8}
  var reply int
  err = client.Call("Arith.Multiply", args, &reply)
  if err != nil {
    log.Fatal("Multiply error:", err)
  }
  fmt.Printf("Multiply: %d*%d=%d\n", args.A, args.B, reply)
}
```