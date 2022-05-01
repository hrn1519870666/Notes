###  JMM(Java 内存模型)

**JMM定义了共享内存系统中多线程读写操作行为的规范。**同步的规定：

1. 线程加锁前，必须读取主内存的最新值到自己的工作内存 
2. 线程解锁前，必须把共享变量的值刷新回主内存 
3. 加锁解锁使用同一把锁

#### JMM三大特性

**原子性**

实现方式：`synchronized`和原子整型`AtomicInteger`

**可见性**

主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行解锁操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

**有序性**

实现方式：`synchronized`和`volatile`。原理有所区别：`volatile`关键字会禁止指令重排。`synchronized`关键字保证同一时刻只允许一条线程操作。



### volatile关键字

volatile是Java虚拟机提供的轻量级的同步机制。

#### 1.保证可见性

JMM规定，线程对变量的操作（读取赋值等）必须在工作内存中进行，首先概要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存的**变量副本拷贝。**

<a href="https://sm.ms/image/Vd28Ub6kmOpWrIc" target="_blank"><img src="https://s2.loli.net/2022/04/29/Vd28Ub6kmOpWrIc.png" style="zoom:33%;"  ></a>



这就可能导致线程在主存中修改了一个变量的值，而另外一个线程还继续使用它在寄存器中的变量值的拷贝，造成**数据的不一致**。要解决这个问题，就需要把变量声明为`volatile`来保证可见性。可见性是一种及时通知机制，当多个线程访问同一个变量时，一个线程修改了这个变量的值，当它写回主内存时，立刻将这次修改通知给其他线程（其他线程能够立即看到修改的值）。

`volatile`保证可见性的原理：告诉 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/d49c5557-140b-4abf-adad-8aac3c9036cf.png" alt="volatile关键字的可见性" style="zoom:80%;" />



#### 2.不保证原子性

原因：线程在各自的工作内存中将变量修改之后，在写回主内存时发生了写覆盖。



#### 3.禁止指令重排

以DCL为例：

```java
public class Singleton {

    private static Singleton uniqueInstance;

    private Singleton() {
    }

    public static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码
        if (uniqueInstance == null) {
            //类对象加锁
            synchronized (Singleton.class) {
                //对象为空才去创建（懒加载）
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();//非原子操作。注意！！！
                }
            }
        }
        return uniqueInstance;
    }
}
```

`uniqueInstance = new Singleton()` 不是原子操作，这段代码可以简单分为下面三步执行：

1. 为 uniqueInstance 分配内存空间;
2. 初始化 uniqueInstance;
3. 将 uniqueInstance 指向分配的内存地址

处理器在进行重排顺序时会考虑指令之间的数据依赖性，2和3没有数据依赖，所以执行顺序有可能变成 1->3->2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 a 执行了 1 和 3，此时 线程 b 调用 `getUniqueInstance()` 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化，所以就会导致空指针异常。

**使用 volatile 修饰变量就可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。**代码修改如下：

```java
 private volatile static Singleton uniqueInstance;
```

原理：

内存屏障（Memory Barrier）又称内存栅栏，是一个CPU指令，他的作用有两个：

1. 保证特定操作的执行顺序
2. 保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）

编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier，则会告诉编译器和CPU， 不管什么指令都不能和这条Memory Barrier指令重排顺序，也就是说**通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。**内存屏障另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。

<a href="https://sm.ms/image/UGf4MWxl9sA1akX" target="_blank"><img src="https://s2.loli.net/2022/04/30/UGf4MWxl9sA1akX.png" ></a>



#### 哪些地方用到了volatile？

DCL。



### **happen-before规则**

Java内存模型规定在某些场景下（一共8条），**前面一个操作的结果对后续操作必须是可见的。**这8条规则成为happen-before规则。



### CAS

CAS的全称为compare and swap，比较并交换。比较当前工作内存中的值和主内存中的值，如果相同则执行规定操作，比如更新主内存值，否则继续比较，直到工作内存中的值和主内存中的值一致为止。

#### CAS应用

CAS有3个操作数，内存值V，预期值A，要修改的更新值B。如果预期值A和更新值B相等，则将内存值V修改为B，否则进行循环比较，直到相等为止。

#### 底层原理

CAS是一条CPU并发原语（简称原语），底层靠unsafe类保证原子性。

Unsafe 是CAS核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当 于一个后门，基于该类可以直接操作特定内存数据。Unsafe类存在于 sun.misc 包中，其内部方法操作可以像C的指针一样直接操作内存。**Java中CAS操作的执行依赖于Unsafe类的方法，Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务。**

CAS并发原语体现在JAVA语言中就是Unsafe类中各个方法。调用Unsafe类中的CAS方法，JVM会帮我们实现CAS汇编指令。这是一种完全依赖于硬件的功能，通过他实现了原子操作。由于CAS是一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题（线程安全）。

以下代码使用了 AtomicInteger 执行了自增的操作。

```java
private AtomicInteger cnt = new AtomicInteger();

public void add() {
    cnt.getAndIncrement();
}
```

以下代码是getAndIncrement() 的源码，它调用了 Unsafe 的 getAndAddInt() 。

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var4+var5。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        // getAndAddInt中的get
        // 也就是从主内存拷贝到工作内存的是var5
        var5 = this.getIntVolatile(var1, var2);
        // 过了一会，将主内存的值(通过var1和var2获得)与工作内存值var5比较，相等则+1
        // 修改成功返回true，!取反变成false，退出while循环
        // 否则，继续取值然后再比较，直到更新完成
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

#### 作用

CAS没有加锁，多个线程都可以直接操作共享资源，在实际去修改的时候才去判断能否修改成功。在很多的情况下CAS比synchronized锁更高效。

#### 缺点

##### 1.ABA问题

线程1从内存位置V取出A，线程2同时也从内存取出A，并且线程2进行一些操作将值改为B，然后线程2又将V位置数据改成A，这时候线程1进行CAS操作发现内存中的值依然时A，然后线程1操作成功。尽管线程1的CAS操作成功，但是不代表这个过程没有问题。

**解决方法**： J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。在变量前面加上版本号，每次变量更新的时候变量的**版本号都`+1`**，即`A->B->A`就变成了`1A->2B->3A`。

```java

```



##### 2.循环时间长开销大

**如果`CAS`操作失败，就需要循环进行`CAS`操作(自旋)，**如果长时间都不成功的话，那么会造成CPU极大的开销。

**解决方法**： 限制自旋次数，防止进入死循环。

##### 3.只能保证一个共享变量的原子操作

源代码`public final int getAndAddInt(Object var1, long var2, int var4)`，参数只有一个Object对象。

**解决方法**： 如果需要对多个共享变量进行操作，可以使用加锁方式(悲观锁)保证原子性，因为可以锁一个代码段。

```java
public class ABADemo {
	static AtomicStampedReference<Integer> atomicStampedReference = 
        new AtomicStampedReference<>(100, 1);
    
	public static void main(String[] args) {
		new Thread(() -> {
			int stamp = atomicStampedReference.getStamp();
			System.out.println(Thread.currentThread().getName() + "\t第1次版本号" +
			stamp);
			try {
                // 等待线程2获得相同的初始版本号
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
            // ABA
			atomicStampedReference.compareAndSet(100, 101,
			atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
			System.out.println(Thread.currentThread().getName() + "\t第2次版本号" +
			atomicStampedReference.getStamp());
            
			atomicStampedReference.compareAndSet(101, 100,
			atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
			System.out.println(Thread.currentThread().getName() + "\t第3次版本号" +
			atomicStampedReference.getStamp());
		}, "Thread 1").start();
        
		new Thread(() -> {
			int stamp = atomicStampedReference.getStamp();
			System.out.println(Thread.currentThread().getName() + "\t第1次版本号" +
			stamp);
			try {
                // 等待线程1执行完ABA操作
				TimeUnit.SECONDS.sleep(3);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			boolean result = atomicStampedReference.compareAndSet(100, 2019, stamp,
			stamp + 1);
			System.out.println(Thread.currentThread().getName() + "\t修改是否成功" +
			result + "\t当前最新实际版本号：" + atomicStampedReference.getStamp());
			System.out.println(Thread.currentThread().getName() + "\t当前最新实际值：" +
			atomicStampedReference.getReference());
		}, "Thread 2").start();
	}
}
```

输出结果：

```java
Thread 1 第1次版本号1
Thread 2 第1次版本号1
Thread 1 第2次版本号2
Thread 1 第3次版本号3
Thread 2 修改是否成功false 当前最新实际版本号：3
Thread 2 当前最新实际值：100
```



### 锁

#### 公平锁和非公平锁

- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁。
- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，如果抢锁失败，就再采用类似公平锁那种方式。

区别：线程执行同步代码块时，是否会去尝试获取锁。如果会尝试获取锁，就是非公平的。如果不会尝试获取锁，直接进队列，再等待唤醒，就是公平的。



#### 可重入锁（递归锁）

指的时同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一个线程在外层方法获取锁 的时候，在进入内层方法会自动获取锁，也就是说，线程可以进入任何一个它已经拥有的锁所同步着的代码块。

Synchronized / ReentrantLock 是典型的可重入锁。

可重入锁最大的作用是避免死锁。

```java
class Phone{
    // 两个方法都是synchronized
	public synchronized void sendSMS()throws Exception{
		System.out.println(Thread.currentThread().getName()+"sendSMS");
		Thread.sleep(3000);
		sendEmail();
	}
	public synchronized void sendEmail() throws Exception{
		System.out.println(Thread.currentThread().getName()+"sendEmail");
	}
}

	public static void main(String[] args) {
		Phone phone = new Phone();
		new Thread(() -> {
			try {
				phone.sendSMS();
			} catch (Exception e) {
				e.printStackTrace();
			}
        }, "Thread 1").start();

        new Thread(() -> {
            try {
                phone.sendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "Thread 2").start();
    }

}
```



#### 自旋锁

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。**自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。**
在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。



#### 独占锁（写锁）和共享锁（读锁）

独占锁：指该锁一次只能被一个线程所持有。Synchronized和ReentrantLock都是独占锁。

共享锁：指该锁可以被多个线程锁持有。

ReentrantReadWriteLock的读锁是共享，写锁是独占，写的时候只能一个人写，但是读的时候，可以多个人同时读。



#### 乐观锁和悲观锁

乐观锁：总是假设最好的情况，每次去操作数据都认为不会被别的线程修改数据，所以在每次操作数据的时候都不会给数据加锁，**即在线程对数据进行操作的时候，别的线程不会阻塞仍然可以对数据进行操作，只有在需要更新数据的时候才会去判断数据是否被别的线程修改过，如果数据被修改过则会拒绝操作并且返回错误信息给用户。**`CAS`是乐观锁思想的实现。

悲观锁：总是假设最坏的情况，**每次去操作数据时候都认为会被别的线程修改数据，所以在每次操作数据的时候都会给数据加锁，让别的线程无法操作这个数据，别的线程会一直阻塞直到获取到这个数据的锁**。`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

#### **乐观锁和悲观锁的使用情景（CAS和synchronized的使用情景）**

1. 对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗CPU资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，自旋几率较少，因此可以获得更高的性能。
2. 对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

**简单的来说CAS适用于写少的情况（多读场景，冲突一般较少），synchronized适用于写多的情况下（多写场景，冲突一般较多）**



### CountDownLatch

做减法，减到0唤醒主线程。

它允许一个或多个线程一直等待，知道其他线程的操作执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。类似于Go语言中的`sync.WaitGroup`。

CountDownLatch主要有两个方法，当一个或多个线程调用await()方法时，调用线程会被阻塞。其他线程调用 countDown()方法会将计数器减1，当计数器的值变为0时，因调用await()方法被阻塞的线程才会被唤醒，继续执行。

```java
public class CountDownLatchDemo {
	public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);   // 初始值
        for (int i = 1; i <= 6; i++) {
		new Thread(() -> {
			System.out.println(Thread.currentThread().getName()+"上完自习，离开教室");
            countDownLatch.countDown();   // 减1
			}, "Thread-->"+i).start();
		}
        countDownLatch.await();   //
        System.out.println(Thread.currentThread().getName()+"班长最后关门走人");
	}
}
```

输出：

```bash
Thread-->2上完自习，离开教室
Thread-->5上完自习，离开教室
Thread-->3上完自习，离开教室
Thread-->1上完自习，离开教室
Thread-->4上完自习，离开教室
Thread-->6上完自习，离开教室
main班长最后关门走人
```



### CyclicBarrier

做加法，先到的先等，人齐了开会。

可循环（Cyclic）使用的屏障。让一组线程到达一个屏障（也可叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续工作，线程进入屏障通过CycliBarrier的await()方法。

```java
public static void main(String[] args) {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> 
                                  {System.out.println("====召唤神龙=====");});
	for (int i = 1; i <= 7; i++) {
		final int tempInt = i;
		new Thread(() -> {
			System.out.println(Thread.currentThread().getName() + "收集到第"+ tempInt + "颗				龙珠");
			try {
				cyclicBarrier.await();
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				e.printStackTrace();
			}
		}, "" + i).start();
	}
}
```



### Semaphore信号量 

多个线程竞争多个资源。可以代替Synchronize和Lock。

```java
public static void main(String[] args) {
	Semaphore semaphore = new Semaphore(3);//模拟三个停车位
		for (int i = 1; i <= 6; i++) {//模拟6部汽车
			new Thread(() -> {
				try {
					semaphore.acquire();
					System.out.println(Thread.currentThread().getName() + "抢到车位");
					try {
						TimeUnit.SECONDS.sleep(3);//停车3s
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(Thread.currentThread().getName() + "离开");
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					semaphore.release();
				}
			}, "Car " + i).start();
		}
	}
}
```

```bash
3抢到车位
1抢到车位
2抢到车位
2离开
3离开
4抢到车位
1离开
6抢到车位
5抢到车位
5离开
4离开
6离开
```



### 线程池

#### **使用线程池的好处**：

- **降低资源消耗**。通过重复利用已创建的线程，来降低线程创建和销毁造成的消耗。

- **提高响应速度**。当任务到达时，任务不需要等到线程创建就能立即执行。

- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

#### ThreadPoolExecutor 类分析

```java
    /**
     * 用给定的初始参数创建一个新的ThreadPoolExecutor。
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

**`ThreadPoolExecutor` 3 个最重要的参数：**

- **`corePoolSize` : 核心线程数，线程池的基本大小，即在没有任务需要执行的时候线程池的大小，并且只有在工作队列（workQueue）满了的情况下才会创建超出这个数量的线程。**
- **`workQueue`: 工作队列，当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。**
- **`maximumPoolSize` : 线程池中的当前线程数目不会超过该值。如果工作队列中任务已满，并且当前线程个数小于maximumPoolSize，那么会创建新的线程来执行任务。**

`ThreadPoolExecutor`其他常见参数:

1. **`keepAliveTime`**:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. **`unit`** : `keepAliveTime` 参数的时间单位。
3. **`threadFactory`** :executor 创建新线程的时候会用到。
4. **`handler`** :拒绝策略。

#####  `ThreadPoolExecutor` 拒绝策略

**如果当前同时运行的线程数量达到最大线程数量`maximumPoolSize`并且队列也已经被放满了任务时，**`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`**：线程池默认的拒绝策略。抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：该策略直接在调用者线程中运行当前被丢弃的任务，但是任务提交线程的性能极有可能急剧下降。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。



#### 线程池原理总结

![图解线程池实现原理](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/图解线程池实现原理.png)





