## goroutine

一个goroutine会以一个很小的栈开始其生命周期，一般只需要2KB。区别于操作系统线程由系统内核进行调度， goroutine 是由Go运行时（runtime）负责调度。例如Go运行时会智能地将 m个goroutine 合理地分配给n个操作系统线程，实现类似m:n的调度机制，**不再需要Go开发者自行在代码层面维护一个线程池。**

**从线程调度讲，Go语言相比起其他语言的优势在于，线程是由OS内核来调度的， goroutine 则是由Go运行时（runtime）自己的调度器调度的，完全是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的malloc函数，成本比调度OS线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干goroutine均分在物理线程上， 再加上本身 goroutine 的超轻量级，以上种种特性保证了 goroutine 调度方面的性能。**

### go关键字

Go语言中使用 goroutine 非常简单，只需要在函数或方法调用前加上`go`关键字就可以创建一个 goroutine ，从而让该函数或方法在新创建的 goroutine 中执行。

```go
go f()  // 创建一个新的 goroutine 运行函数f
```

匿名函数也支持使用`go`关键字创建 goroutine 去执行。

```go
go func(){
  // ...
}()   // 注意末尾括号
```

下面的示例代码中我们在 main goroutine 中使用`sync.WaitGroup`来等待 hello goroutine 完成后再退出。

```go
package main

import (
	"fmt"
	"sync"
)

// 声明全局等待组变量
var wg sync.WaitGroup

func hello() {
	fmt.Println("hello")
	wg.Done() // 告知当前goroutine完成
}

func main() {
	wg.Add(1) // 登记1个goroutine
	go hello()
	fmt.Println("你好")
	wg.Wait() // 阻塞等待登记的goroutine完成
}
```

输出：

```bash
你好
hello
```

**先打印“你好”是因为在程序中创建 goroutine 执行函数需要一定的开销，而与此同时 main 函数所在的 goroutine 是继续执行的。**

![main goroutine和hello goorutine](https://www.liwenzhou.com/images/Go/concurrence/goroutine02.png)

### 启动多个goroutine

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func hello(i int) {
	defer wg.Done() // goroutine结束就登记-1
	fmt.Println("hello", i)
}
func main() {
    // wg.Add(10)   或者在for循环外面一次性登记10
	for i := 0; i < 10; i++ {
		wg.Add(1) // 启动一个goroutine就登记+1
		go hello(i)
	}
	wg.Wait() // 等待所有登记的goroutine都结束
}
```

多次执行上面的代码会发现每次终端上打印数字的顺序都不一致。这是因为10个 goroutine 是并发执行的，而 goroutine 的调度是随机的。



## channel

如果说 goroutine 是Go程序并发的执行体，`channel`就是它们之间的连接。**`channel`是让一个 goroutine 发送特定值到另一个 goroutine 的通信机制。**

Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序。每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。

### channel类型

声明通道类型变量的格式如下：

```go
var 变量名称 chan 元素类型
```

其中：

- chan：是关键字
- 元素类型：是指通道中传递元素的类型

```go
var ch1 chan int   // 声明一个传递整型的通道
var ch2 chan []int // 声明一个传递int切片的通道
```

### channel零值

未初始化的通道类型变量其默认零值是`nil`。

```go
var ch chan int
fmt.Println(ch) // <nil>
```

### 初始化channel

声明的通道类型变量需要使用`make`函数初始化之后才能使用。具体格式如下：

```go
make(chan 元素类型, [缓冲大小])
```

channel的缓冲大小是可选的。

```go
ch4 := make(chan int)
ch5 := make(chan bool, 1)  // 声明一个缓冲区大小为1的通道
```

### channel操作

通道共有发送、接收和关闭三种操作。而发送和接收操作都使用`<-`符号。

```go
ch := make(chan int)
ch <- 10 // 把10发送到ch中
x := <- ch // 从ch中接收值并赋值给变量x
<-ch       // 从ch中接收值，忽略结果
close(ch)
```

通道通常由发送方执行关闭操作，只有在接收方明确等待通道关闭的信号时才需要执行关闭操作。关闭通道不是必须的，它可以被垃圾回收。

关闭后的通道有以下特点：

1. **对一个关闭的通道进行接收会一直获取值直到通道为空。**
2. **对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值（零值不是0）。**
3. **对一个关闭的通道再发送值就会导致 panic。**
4. **关闭一个已经关闭的通道会导致 panic。**

### 无缓冲的通道

无缓冲的通道又称为阻塞的通道。**无缓冲的通道只有在有接收方能够接收值的时候才能发送成功，否则会一直处于等待发送的阶段。同理，如果对一个无缓冲通道执行接收操作时，没有任何向通道中发送值的操作也会导致接收操作阻塞。**

例如：

```go
func main() {
	ch := make(chan int)
	ch <- 10
}
```

上面这段代码能够通过编译，但是执行的时候会出现以下错误：

```bash
# // 程序中的 goroutine 都被挂起导致程序死锁
fatal error: all goroutines are asleep - deadlock!   

goroutine 1 [chan send]:
main.main()
        .../main.go:8 +0x54
```

解除死锁：

```go
func recv(c chan int) {
	ret := <-c
	fmt.Println("接收成功", ret)
}

func main() {
	ch := make(chan int)
	go recv(ch) // 创建一个 goroutine 从通道接收值
	ch <- 10
	fmt.Println("发送成功")
}
```

首先无缓冲通道`ch`上的发送操作会阻塞，直到另一个 goroutine 在该通道上执行接收操作，这时数字10才能发送成功，两个 goroutine 将继续执行。相反，如果接收操作先执行，接收方所在的 goroutine 将阻塞，直到 main goroutine 中向该通道发送数字10。

使用无缓冲通道进行通信将导致发送和接收的 goroutine 同步化。因此，无缓冲通道也被称为`同步通道`。

### 有缓冲的通道

还有另外一种解决上面死锁问题的方法，那就是使用有缓冲区的通道。我们可以在使用 make 函数初始化通道时，可以为其指定通道的容量。只要通道的容量大于零**（容量为1也是有缓冲的通道）**，那么该通道就属于有缓冲的通道，通道的容量表示通道中最大能存放的元素数量。当通道内已有元素数达到最大容量后，再向通道执行发送操作就会阻塞，除非有从通道执行接收操作。

### 判断一个通道是否被关闭

对一个通道执行接收操作时支持使用如下多返回值模式。

```go
value, ok := <- ch
```

其中：

- value：从通道中取出的值，如果通道被关闭则返回对应类型的零值。
- ok：通道ch关闭时返回 false，否则返回 true。

下面代码片段中的`f`函数会循环从通道`ch`中接收所有值，直到通道被关闭后退出。

```go
func f(ch chan int) {
	for {
		v, ok := <-ch
		if !ok {
			fmt.Println("通道已关闭")
			break
		}
		fmt.Printf("v:%#v ok:%#v\n", v, ok)
	}
}

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	close(ch)   // 这里先关闭了通道，然后从中取值
	f(ch)
}
```

通常我们会选择使用`for range`循环从通道中接收值，当通道被关闭后，会在通道内的所有值被接收完毕后会自动退出循环。上面那个示例我们使用`for range`改写后会很简洁。

```go
func f2(ch chan int) {
	for v := range ch {   // 
		fmt.Println(v)
	}
}
```

**注意：**目前Go语言中并没有提供一个不对通道进行读取操作就能判断通道是否被关闭的方法。不能简单的通过`len(ch)`操作来判断通道是否被关闭。

### 单向通道

在某些场景下我们可能会将通道作为参数在多个任务函数间进行传递，通常我们会选择在不同的任务函数中对通道的使用进行限制，比如限制通道在某个函数中只能执行发送或只能执行接收操作。

Go语言中提供了单向通道来处理这种需要限制通道只能进行某种操作的情况。

```go
<- chan int // 只接收通道，只能接收不能发送
chan <- int // 只发送通道，只能发送不能接收
```

其中，箭头`<-`和关键字`chan`的相对位置表明了当前通道允许的操作，这种限制将在编译阶段进行检测。

另外对一个只接收通道执行close也是不允许的，因为默认通道的关闭操作应该由发送方来完成。

**生产者-消费者模型：**

```go
package main

import (
	"fmt"
)

// Producer 返回一个接收通道
// 并持续将符合条件的数据发送至返回的通道中
// 数据发送完成后会将返回的通道关闭
func Producer() <-chan int {   // 返回值类型是一个只接收通道
	ch := make(chan int, 2)
	// 创建一个新的goroutine执行发送数据的任务
	go func() {   // go
		for i := 0; i < 10; i++ {
			if i%2 == 1 {
				ch <- i
			}
		}
		close(ch) // 任务完成后关闭通道
	}()   // 末尾有括号
	return ch
}

// Consumer 参数为接收通道
func Consumer(ch <-chan int) int {
	sum := 0
	for v := range ch {   //
		sum += v
	}
	return sum
}

func main() {
	ch := Producer()
  
	res := Consumer(ch)
	fmt.Println(res) // 25
}
```

在函数传参及任何赋值操作中全向通道（正常通道）可以转换为单向通道，但是无法反向转换。

```go
var ch1 = make(chan int, 1)
ch1 <- 10
close(ch1)
Consumer(ch1) // 函数传参时将ch1转为单向通道

var ch2 = make(chan int, 1)
ch2 <- 10
var ch3 <-chan int // 声明一个只接收通道ch3
ch3 = ch2          // 变量赋值时将ch2转为单向通道
<-ch3
```

### 总结

下面的表格中总结了对不同状态下的通道执行相应操作的结果。

![img](https://www.liwenzhou.com/images/Go/concurrence/channel.png)



## select多路复用

在某些场景下我们可能需要同时从多个通道接收数据。通道在接收数据时，如果没有数据可以被接收，那么当前 goroutine 将会发生阻塞。

Go 语言内置了`select`关键字，使用它可以同时响应多个通道的操作。

select 的使用方式类似于之前学到的 switch 语句，它也有一系列 case 分支和一个默认的分支。每个 case 分支会对应一个通道的通信（接收或发送）过程。select 会一直等待，直到其中的某个 case 的通信操作完成时，就会执行该 case 分支对应的语句。具体格式如下：

```go
select {
case <-ch1:
	//...
case data := <-ch2:
	//...
case ch3 <- 10:
	//...
default:
	//默认操作
}
```

Select 语句具有以下特点：

- **可处理一个或多个 channel 的发送/接收操作。**
- **如果多个 case 同时满足，select 会随机选择一个执行。**
- **对于没有 case 的 select 会一直阻塞。**

select 的具体使用：

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 1)
	for i := 1; i <= 10; i++ {
		select {
		case x := <-ch:
			fmt.Println(x)
		case ch <- i:
		}
	}
}
```

上面的代码输出内容如下。

```bash
1
3
5
7
9
```

示例中的代码首先是创建了一个缓冲区大小为1的通道 ch，进入 for 循环后：

- 第一次循环时 i = 1，select 语句中包含两个 case 分支，此时由于通道中没有值可以接收，所以`x := <-ch` 这个 case 分支不满足，而`ch <- i`这个分支可以执行，会把1发送到通道中，结束本次 for 循环；
- 第二次 for 循环时，i = 2，由于通道缓冲区已满，所以`ch <- i`这个分支不满足，而`x := <-ch`这个分支可以执行，从通道接收值1并赋值给变量 x ，所以会在终端打印出 1；
- 后续的 for 循环以此类推会依次打印出3、5、7、9。

阅读下方代码片段，尝试找出其中存在的问题。

```go
// demo2 通道误用导致的bug
func demo2() {
	ch := make(chan string)
	go func() {
		// 这里假设执行一些耗时的操作
		time.Sleep(3 * time.Second)
		ch <- "job result"
	}()

	select {
	case result := <-ch:
		fmt.Println(result)
	case <-time.After(time.Second): // 较小的超时时间
		return
	}
}
```

上述代码片段可能导致 goroutine 泄露（goroutine 并未按预期退出并销毁）。由于 select 命中了超时逻辑，导致通道没有消费者（无接收操作），而其定义的通道为无缓冲通道，因此 goroutine 中的`ch <- "job result"`操作会一直阻塞，最终导致 goroutine 泄露。



## 并发安全和锁

### 互斥锁

使用互斥锁能够保证同一时间有且只有一个 goroutine 进入临界区，其他的 goroutine 则在等待锁；当互斥锁释放后，等待的 goroutine 才可以获取锁进入临界区，多个 goroutine 同时等待一个锁时，唤醒的策略是随机的。

```go
package main

import (
	"fmt"
	"sync"
)

// sync.Mutex

var (
	x int64

	wg sync.WaitGroup // 等待组

	m sync.Mutex // 互斥锁
)

// add 对全局变量x执行5000次加1操作
func add() {
	for i := 0; i < 5000; i++ {
		m.Lock() // 修改x前加锁
		x = x + 1
		m.Unlock() // 改完解锁
	}
	wg.Done()
}

func main() {
	wg.Add(2)

	go add()
	go add()

	wg.Wait()
	fmt.Println(x)
}
```

将上面的代码编译后多次执行，每一次都会得到预期中的结果——10000。

### 读写锁

很多场景是读多写少的，当我们并发的去读取一个资源而不涉及资源修改的时候没有必要加互斥锁，这种场景下使用读写锁是更好的一种选择。读写锁在 Go 语言中使用`sync`包中的`RWMutex`类型。

`sync.RWMutex`提供了以下5个方法。

|               方法名                |              功能              |
| :---------------------------------: | :----------------------------: |
|      func (rw *RWMutex) Lock()      |            获取写锁            |
|     func (rw *RWMutex) Unlock()     |            释放写锁            |
|     func (rw *RWMutex) RLock()      |            获取读锁            |
|    func (rw *RWMutex) RUnlock()     |            释放读锁            |
| func (rw *RWMutex) RLocker() Locker | 返回一个实现Locker接口的读写锁 |

读写锁分为两种：读锁和写锁。**当一个 goroutine 获取到读锁之后，其他的 goroutine 如果是获取读锁会继续获得锁，如果是获取写锁就会等待；而当一个 goroutine 获取写锁之后，其他的 goroutine 无论是获取读锁还是写锁都会等待。**

### sync.WaitGroup

在代码中生硬的使用`time.Sleep`肯定是不合适的，Go语言中可以使用`sync.WaitGroup`来实现并发任务的同步。 `sync.WaitGroup`有以下几个方法：

|                方法名                |        功能         |
| :----------------------------------: | :-----------------: |
| func (wg * WaitGroup) Add(delta int) |    计数器+delta     |
|        (wg *WaitGroup) Done()        |      计数器-1       |
|        (wg *WaitGroup) Wait()        | 阻塞直到计数器变为0 |

`sync.WaitGroup`内部维护着一个计数器，计数器的值可以增加和减少。例如当我们启动了 N 个并发任务时，就将计数器值增加N。每个任务完成时通过调用 Done 方法将计数器减1。通过调用 Wait 来等待并发任务执行完，当计数器值为 0 时，表示所有并发任务已经完成。

`sync.WaitGroup`是一个结构体，进行参数传递的时候要传递指针。

### sync.Once

在某些场景下我们需要确保某些操作即使在高并发的场景下也只会被执行一次，例如只加载一次配置文件等。

Go语言中的`sync`包中提供了一个针对只执行一次场景的解决方案——`sync.Once`，`sync.Once`只有一个`Do`方法，其签名如下：

```go
func (o *Once) Do(f func())
```

下面是借助`sync.Once`实现的并发安全的**单例模式：**

```go
package singleton

import (
    "sync"
)

type singleton struct {}   // 名为singleton的结构体类型

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func() {   //
        instance = &singleton{}
    })
    return instance
}
```

`sync.Once`其实内部包含一个互斥锁和一个布尔值，互斥锁保证布尔值和数据的安全，而布尔值用来记录初始化是否完成。这样设计就能保证初始化操作的时候是并发安全的并且初始化操作也不会被执行多次。

### sync.Map

Go 语言中内置的 map 不是并发安全的，`sync`包中提供了一个开箱即用的并发安全版 map——`sync.Map`。开箱即用表示其不用像内置的 map 一样使用 make 函数初始化就能直接使用。同时`sync.Map`内置了诸如`Store`、`Load`、`LoadOrStore`、`Delete`、`Range`等操作方法。

|                            方法名                            |              功能               |
| :----------------------------------------------------------: | :-----------------------------: |
|         func (m *Map) Store(key, value interface{})          |        存储key-value数据        |
| func (m *Map) Load(key interface{}) (value interface{}, ok bool) |       查询key对应的value        |
| func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool) |    查询或存储key对应的value     |
| func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool) |          查询并删除key          |
|            func (m *Map) Delete(key interface{})             |             删除key             |
|   func (m *Map) Range(f func(key, value interface{}) bool)   | 对map中的每个key-value依次调用f |

```go
package main

import (
	"fmt"
	"strconv"
	"sync"
)

// 并发安全的map
var m = sync.Map{}

func main() {
	wg := sync.WaitGroup{}
	// 对m执行20个并发的读写操作
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			m.Store(key, n)         // 存储key-value
			value, _ := m.Load(key) // 根据key取值
			fmt.Printf("k=:%v,v:=%v\n", key, value)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```



## 原子操作

针对整数数据类型（int32、uint32、int64、uint64）我们还可以使用原子操作来保证并发安全，通常直接使用原子操作比使用锁操作效率更高。Go语言中原子操作由内置的标准库`sync/atomic`提供。

**atomic包**

```go
// 读取
func LoadInt32(addr *int32) (val int32)
func LoadInt64(addr *int64) (val int64)
func LoadUint32(addr *uint32) (val uint32)
func LoadUint64(addr *uint64) (val uint64)
func LoadUintptr(addr *uintptr) (val uintptr)
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

// 写入
func StoreInt32(addr *int32, val int32)
func StoreInt64(addr *int64, val int64)
func StoreUint32(addr *uint32, val uint32)
func StoreUint64(addr *uint64, val uint64)
func StoreUintptr(addr *uintptr, val uintptr)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

// 修改
func AddInt32(addr *int32, delta int32) (new int32)
func AddInt64(addr *int64, delta int64) (new int64)
func AddUint32(addr *uint32, delta uint32) (new uint32)
func AddUint64(addr *uint64, delta uint64) (new uint64)
func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)

// 交换
func SwapInt32(addr *int32, new int32) (old int32)
func SwapInt64(addr *int64, new int64) (old int64)
func SwapUint32(addr *uint32, new uint32) (old uint32)
func SwapUint64(addr *uint64, new uint64) (old uint64)
func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

// CAS
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)
```



```go
type AtomicCounter struct {
	counter int64
}

func (a *AtomicCounter) Inc() {
	atomic.AddInt64(&a.counter, 1)
}

func (a *AtomicCounter) Load() int64 {
	return atomic.LoadInt64(&a.counter)
}

func test(c *AtomicCounter) {
	var wg sync.WaitGroup
	start := time.Now()
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			c.Inc()
			wg.Done()
		}()
	}
	wg.Wait()
	end := time.Now()
	fmt.Println(c.Load(), end.Sub(start))
}

func main() {
	c := AtomicCounter{} // 并发安全且比互斥锁效率更高
	test(&c)
}
```