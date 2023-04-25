### synchronized 关键字

**`synchronized` 是一种互斥锁，解决的是多个线程之间访问资源的同步性问题，它可以保证被修饰的方法或者代码块在任意时刻只能有一个线程执行。**

#### 用法

##### **1. 同步一个代码块**  

同步代码块可以选择以什么来加锁，比同步方法要更细颗粒度，我们可以选择只同步会发生同步问题的部分代码，而不是整个方法。

```java
public void func() {
    // this锁的是调用该方法的对象
    synchronized (this) {
        // ...
    }
}
```

**它只作用于同一个对象，同一个类的不同对象拥有自己的锁，如果调用两个对象上的同步代码块，就不会进行同步。**

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedExample {

    public void func() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func());
    executorService.execute(() -> e1.func());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func());
    executorService.execute(() -> e2.func());
}
```

```html
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

##### 2.同步一个方法：作用于同一个对象。

以实例对象作为锁，进入同步代码前需要获得当前实例对象的锁。

Java通过 对象.方法 的形式调用方法，锁的就是点之前的对象。

##### **3. 同步一个类**  

以类对象为锁，进入同步代码块前需要获得当前**类对象**的锁。

不是在类上加synchronized，而是锁代码块：

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

**作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。**

```java
public class SynchronizedExample {

    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

##### **4.同步一个静态方法：作用于整个类。**

**注意：**

1. 如果一个线程正在访问实例对象的一个synchronized方法时，该对象的其它synchronized方法也不能访问，因为**一个对象只有一个监视器锁对象。**
2. 使用synchronized修饰类和对象时，由于类对象和实例对象分别拥有自己的监视器锁，因此不会相互阻塞。
3. 线程A访问实例对象的非static但synchronized方法时，线程B也可以同时访问实例对象的static synchronized方法，因为前者获取的是实例对象的监视器锁，而后者获取的是类对象的监视器锁，两者不存在互斥关系。



#### 底层原理

#####  修饰代码块的情况

`synchronized` 修饰代码块时，使用的是 **`monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明结束位置。**

**在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。**

#####  修饰方法的的情况

`synchronized` 修饰方法时，使用的是**`ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。在进入该方法之前先获取相应的锁，锁的计数器加1，方法结束后计数器减1。**如果获取失败就阻塞，直到该锁被释放。



#### synchronized和 volatile的区别

1. `volatile`本质是在告诉JVM，当前变量在寄存器中的值是不确定的，需要从主存中读取； `synchronized`则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2. `volatile` 关键字是线程同步的轻量级实现，不需要加锁，不会阻塞线程，所以`volatile`性能比`synchronized`关键字要好。
3. `volatile` 关键字只能用于变量，而 `synchronized` 关键字可以修饰代码块、方法、类等。
4. `volatile` 关键字**能保证数据的可见性，但不能保证数据的原子性。**`synchronized` 关键字两者都能保证。



#### synchronized 和 ReentrantLock 的区别

**1.原始构成**

synchronized关键字属于JVM

Lock是具体类，是API层面的锁（java.util.） 

**2.使用方法**

sychronized不需要手动释放锁，当synchronized代码执行完后线程自动释放锁。

ReentrantLock需要手动释放锁，若没有主动释放锁，可能导致死锁，需要lock()和 unlock()方法配合try/finally语句块来完成。

**3.等待可中断**  

含义：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**  

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**  

一个 ReentrantLock 可以同时绑定多个 Condition 对象。ReentrantLock用来实现精确唤醒，而不是像synchronized要么随机唤醒一个线程，要么唤醒全部线程。

**对线程之间按顺序调用，实现A>B>C三个线程启动，要求如下： A打印5次，B打印10次，C打印15次，紧接着A打印5次，B打印10次，C打印15次...循环10轮。**

```java
public class SyncAndReentrantLockDemo {
	public static void main(String[] args) {
		ShareData shareData = new ShareData();
		new Thread(() -> {
			for (int i = 1; i <= 10; i++) {
			shareData.print5();
			}
		}, "A").start();

        new Thread(() -> {
			for (int i = 1; i <= 10; i++) {
			shareData.print10();
			}
		}, "B").start();
        
		new Thread(() -> {
			for (int i = 1; i <= 10; i++) {
			shareData.print15();
			}
		}, "C").start();
	}
}

class ShareData {
	private int number = 1;//A:1 B:2 C:3
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    
    public void print5() {
    	lock.lock();
    	try {
    		//判断
    		while (number != 1) {
    		condition1.await();
    		}
            //干活
            for (int i = 1; i <= 5; i++) {
                System.out.println(Thread.currentThread().getName());
            }
            //通知
            number = 2;
            condition2.signal();
    	} catch (Exception e) {
    		e.printStackTrace();
    	} finally {
    		lock.unlock();
    	}
    }
    
    public void print10() {
    	lock.lock();
    	try {
    		//判断
            while (number != 2) {
            condition2.await();
            }
    		//干活
    		for (int i = 1; i <= 10; i++) {
    		System.out.println(Thread.currentThread().getName());
    		}
    		//通知
            number = 3;
            condition3.signal();
    	} catch (Exception e) {
    		e.printStackTrace();
    	} finally {
    		lock.unlock();
    	}
    }
    
    public void print15() {
    	lock.lock();
    	try {
            //判断
            while (number != 3) {
                condition3.await();
            }
            //干活
            for (int i = 1; i <= 15; i++) {
            System.out.println(Thread.currentThread().getName());
            }
            //通知
            number = 1;
            condition1.signal();
    	} catch (Exception e) {
    		e.printStackTrace();
    	} finally {
    		lock.unlock();
    	}
    }
    
}
```

**使用选择**

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。因为 synchronized 是 JVM 实现的锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。



#### synchronized锁升级（膨胀）过程

在JDK 1.6之前是重量级锁，依赖底层操作系统的 mutex 相关指令实现，所以会有用户态和内核态之间的切换，性能损耗十分明显。而JDK1.6 以后引入偏向锁和轻量级锁在JVM层面实现加锁的逻辑，不依赖底层操作系统。从此以后锁的状态就有了四种，级别由低到高依次为：无锁、偏向锁、轻量级锁、重量级锁。四种状态会随着竞争的情况逐渐升级，而且是不可逆的过程。

<a href="https://sm.ms/image/NygYkjAG67atvUw" target="_blank"><img src="https://s2.loli.net/2022/05/04/NygYkjAG67atvUw.png" ></a>

##### 偏向锁

线程A初次执行到synchronized代码块的时候，锁对象从无锁变成偏向锁（通过CAS修改对象头里的锁标志位）。执行完同步代码块后，线程A并不会主动释放偏向锁，也就是说，对象头中一直保存着线程A的ID。当某个线程第二次到达同步代码块时，它会判断此时持有锁的线程是否就是自己（比较当前线程的threadID和Java对象头中的threadID是否一致），如果是则正常往下执行，由于之前没有释放锁，这里也就不需要重新加锁。如果不是，则说明此时是另一个线程B，在竞争锁对象，需要升级为轻量级锁，偏向锁只需要在置换 ThreadID 的时候依赖一次 CAS 原子指令，而轻量级锁的获取及释放依赖多次 CAS 原子指令。

优点：当一段同步代码一直被同一个线程所访问时，即不存在多个线程的竞争，那么该线程在后续访问时便会自动获得锁，从而降低获取锁带来的消耗，提高性能。

##### 轻量级锁（自旋锁）

一旦有第二个线程加入锁竞争，偏向锁就升级为轻量级锁。其他线程会通过自旋的形式尝试获取锁，线程不会阻塞，从而提高性能。

在轻量级锁状态下继续锁竞争，没有抢到锁的线程将自旋，即不停地循环判断锁是否能够被成功获取。**获取锁的操作，其实就是通过CAS修改对象头里的锁标志位。先比较当前锁标志位是否为“释放”，如果是则将其设置为“锁定”**，代表抢锁成功，然后线程将当前锁的持有者信息修改为自己。

##### 重量级锁

自旋是有限度的（计数器记录自旋次数，默认允许循环10次）。如果锁竞争情况严重，达到最大自旋次数的线程，会将轻量级锁升级为重量级锁（依然是CAS修改锁标志位，但不修改持有锁的线程ID）。当后续线程尝试获取锁时，发现被占用的锁是重量级锁，则直接进入阻塞状态（而不是自旋），等待将来被唤醒。



#### synchronized为什么是非公平锁？

偏向锁和轻量级锁：都有CAS操作，也就是线程都会尝试获取锁。

重量级锁：通过monitor对象中的队列存储线程，但线程进入队列前，还是会先尝试获取锁，如果能获取不到才进入线程等待队列中。

综上所述，synchronized无论处理哪种锁，都是先尝试获取，所以是非公平的。



### CAS

CAS有3个操作数，（主）内存值V，预期值A，要修改的更新值B。如果预期值A和内存值V相等，则将内存值V修改为新值B，否则进行循环比较，直到相等为止。

#### 底层原理

CAS是一条CPU并发原语（简称原语）。原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成数据不一致问题（线程安全）。

**CAS底层靠unsafe类保证原子性，Unsafe类中的所有方法都是native修饰的，也就是说Unsafe类中的方法都直接调用操作系统底层资源执行相应任务。**

以下代码是原子整型 AtomicInteger 的getAndIncrement() 方法的源码，它调用了Unsafe类的 getAndAddInt() 。

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。**通过 getIntVolatile(var1, var2) 得到旧的预期值**，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

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

CAS没有加锁，多个线程都可以直接**操作**共享资源，在实际去**修改**的时候才去判断能否修改成功。在很多的情况下CAS比synchronized锁更高效。

#### 缺点

##### 1.ABA问题

线程1从内存位置V取出A，线程2同时也从内存取出A，并且线程2进行一些操作将值改为B，然后线程2又将V位置数据改成A，这时候线程1进行CAS操作发现内存中的值依然时A，然后线程1操作成功。尽管线程1的CAS操作成功，但是不代表这个过程没有问题。

**解决方法**： J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。在变量前面加上版本号，每次变量更新的时候变量的**版本号都`+1`**，即`A->B->A`就变成了`1A->2B->3A`

##### 2.循环时间长，开销大

**如果`CAS`操作失败，就需要循环进行`CAS`操作(自旋)，**如果长时间都不成功的话，那么会造成CPU极大的开销。

**解决方法**： 限制自旋次数，防止进入死循环。

##### 3.只能保证一个共享变量的原子操作

源代码`public final int getAndAddInt(Object var1, long var2, int var4)`，参数只有一个Object对象。

**解决方法**： 如果需要对多个共享变量进行操作，可以使用加锁方式(悲观锁)保证原子性，因为可以锁一个代码段。



### 锁

#### 公平锁和非公平锁

- 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁。
- 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，如果抢锁失败，就再采用类似公平锁那种方式。

线程执行同步代码块时，是否会去尝试获取锁。如果会尝试获取锁，就是非公平的。如果不会尝试获取锁，直接进队列，再等待唤醒，就是公平的。



#### 可重入锁（递归锁）

同一个线程，在外层方法获取了锁，在进入内层方法时会自动获取锁。也就是说，线程可以进入任何一个它已经拥有的锁所同步着的代码块。

Synchronized / ReentrantLock 是典型的可重入锁。

可重入锁最大的作用是避免死锁。

```java
public class Phone{
    // 两个方法都是synchronized
	public synchronized void sendSMS()throws Exception{
		System.out.println(Thread.currentThread().getName() + " sendSMS");
		Thread.sleep(3000);
		sendEmail();
	}
	public synchronized void sendEmail() throws Exception{
		System.out.println(Thread.currentThread().getName() +" sendEmail");
	}
}

	public static void main(String[] args) {
		Phone phone = new Phone();
		new Thread(() -> {
			try {
                // 获取到了phone对象锁
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

```bash
Thread 1 sendSMS
Thread 1 sendEmail
Thread 2 sendSMS
Thread 2 sendEmail
```



#### 自旋锁

进入阻塞状态的开销很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。**自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。**
在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。



#### 独占锁（写锁）和共享锁（读锁）

独占锁：指该锁一次只能被一个线程所持有。Synchronized和ReentrantLock都是独占锁。

共享锁：指该锁可以被多个线程锁持有。

ReentrantReadWriteLock的读锁是共享，写锁是独占，写的时候只能一个人写，但是读的时候，可以多个人同时读。



#### 乐观锁和悲观锁

乐观锁：总是假设最好的情况，每次去操作数据都认为不会被别的线程修改数据，所以在每次操作数据的时候都不会给数据加锁，**即在线程对数据进行操作的时候，别的线程不会阻塞仍然可以对数据进行操作，只有在需要更新数据的时候才会去判断数据是否被别的线程修改过，如果数据被修改过则会拒绝操作。**`CAS`是乐观锁思想的实现。

悲观锁：总是假设最坏的情况，**每次去操作数据时候都认为会被别的线程修改数据，所以在每次操作数据的时候都会给数据加锁，让别的线程无法操作这个数据，一直阻塞直到获取到这个数据的锁**。`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现。

##### 乐观锁和悲观锁的使用情景（CAS和synchronized的使用情景）

1. 对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒间的切换，以及用户态和内核态间的切换操作浪费CPU资源；而CAS不需要切换线程，不需要进入内核，自旋几率较少，因此可以获得更高的性能。
2. 对于资源竞争严重的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。

**简单来说，CAS适用于读多写少的情况（冲突一般较少），synchronized适用于写多的情况下（冲突一般较多）**



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

`ThreadPoolExecutor`其他参数:

1. `keepAliveTime`:当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 `keepAliveTime`才会被回收销毁；
2. `unit` : `keepAliveTime` 参数的时间单位。
3. `threadFactory` :executor 创建新线程的时候会用到。
4. **`handler`** :拒绝策略。

#####  `ThreadPoolExecutor` 拒绝策略

**如果当前同时运行的线程数量达到最大线程数量`maximumPoolSize`并且队列也已经被放满了任务时，**`ThreadPoolTaskExecutor` 定义一些策略:

- **`ThreadPoolExecutor.AbortPolicy`**：线程池默认的拒绝策略。抛出 `RejectedExecutionException`来拒绝新任务的处理。
- **`ThreadPoolExecutor.CallerRunsPolicy`**：该策略直接在调用者线程中运行当前被丢弃的任务，但是任务提交线程的性能极有可能急剧下降。
- **`ThreadPoolExecutor.DiscardPolicy`：** 不处理新任务，直接丢弃掉。
- **`ThreadPoolExecutor.DiscardOldestPolicy`：** 此策略将丢弃最早的未处理的任务请求。



#### 线程池原理总结

![图解线程池实现原理](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/图解线程池实现原理.png)





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

volatile是JVM提供的轻量级的同步机制。

#### 1.保证可见性

JMM规定，线程对变量的操作（读取赋值等）必须在工作内存中进行。首先将变量从主内存拷贝到自己的工作内存，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存的**变量副本拷贝。**

<a href="https://sm.ms/image/Vd28Ub6kmOpWrIc" target="_blank"><img src="https://s2.loli.net/2022/04/29/Vd28Ub6kmOpWrIc.png" style="zoom:33%;"  ></a>



这就可能导致线程在主内存中修改了一个变量的值，而另外一个线程还继续使用它在本地内存（寄存器）中的变量值的拷贝，造成**数据的不一致**。要解决这个问题，就需要把变量声明为`volatile`来保证可见性。可见性是一种及时通知机制，当多个线程访问同一个变量时，一个线程修改了这个变量的值，当它写回主内存时，立刻将这次修改通知给其他线程，其他线程能够立即看到修改的值。

`volatile`保证可见性的原理：告诉 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-8/d49c5557-140b-4abf-adad-8aac3c9036cf.png" alt="volatile关键字的可见性" style="zoom:80%;" />



#### 2.不保证原子性

原因：线程在各自的工作内存中将变量修改之后，在写回主内存时可能发生写覆盖。



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

**原理：**

内存屏障（Memory Barrier）又称内存栅栏，是一个CPU指令，他的作用有两个：

1. 保证特定操作的执行顺序
2. 保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）

编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier，则会告诉编译器和CPU， 不管什么指令都不能和这条Memory Barrier指令重排顺序，也就是说**通过插入内存屏障，禁止在内存屏障前后的指令执行重排序优化。**内存屏障另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。

<a href="https://sm.ms/image/UGf4MWxl9sA1akX" target="_blank"><img src="https://s2.loli.net/2022/04/30/UGf4MWxl9sA1akX.png" ></a>



#### 哪些地方用到了volatile？

DCL。



### **happen-before规则**

Java内存模型规定在某些场景下（一共8条），**前面一个操作的结果对后续操作必须是可见的。**这8条规则成为happen-before规则。



### ThreadLocal

`ThreadLocal`是一个在多线程中为每一个线程创建单独的变量副本的类。**如果你创建了一个`ThreadLocal`类型的变量，那么访问这个变量的每个线程都会有这个变量的本地副本,** 避免因多线程操作共享变量而导致的数据不一致的情况，保证线程安全**（除了加锁方式以外，保证线程安全的方式）。**

#### 原理

`ThreadLocal`类的`set()`方法

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            // this指ThreadLocal类的实例对象threadLocal
            map.set(this, value);
        else
            createMap(t, value);
    }
```

ThreadLocalMap是ThreadLocal类的一个静态内部类，每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`threadLocal`为 key ，Object 对象（你所设置的对象）为 value 的键值对。它实现了键值对的set和get，每个线程中都有一个独立的ThreadLocalMap副本，它所存储的值，只能被当前线程读取和修改。ThreadLocal类通过操作每一个线程特有的ThreadLocalMap副本，从而实现了变量访问，在不同线程中的隔离。因为每个线程的变量都是自己特有的，完全不会有并发错误。

#### 使用场景

用ThreadLocal保存一些业务内存（用户权限信息，从用户系统获取到的用户名、userId等），这些信息在同一个线程内相同，但是不同的线程使用的业务内容是不相同的。在线程生命周期内，都通过这个静态ThreadLocal实例的get()方法取得自己set过的那个对象，避免了将这个对象作为参数传递的麻烦。

[ThreadLocal底层原理](https://blog.csdn.net/mweibiao/article/details/90111680?spm=1001.2101.3001.6650.8&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-8-90111680-blog-79958414.235%5Ev29%5Epc_relevant_default_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-8-90111680-blog-79958414.235%5Ev29%5Epc_relevant_default_base3&utm_relevant_index=15)

为什么不直接用线程id来作为ThreadLocalMap的key？

无法区分同一个线程的多个threadLocal对象：

```java
public class Son implements Cloneable{
    public static void main(String[] args){
        Thread t = new Thread(new Runnable(){  
            public void run(){
            	ThreadLocal<Son> threadLocal1 = new ThreadLocal<>();
            	threadLocal1.set(new Son());
            	System.out.println(threadLocal1.get());
            	ThreadLocal<Son> threadLocal2 = new ThreadLocal<>();
            	threadLocal2.set(new Son());
            	System.out.println(threadLocal2.get());
            }}); 
        t.start();
    }
}
```

如何区分同一个线程的多个threadLocal对象？

```java
private final int threadLocalHashCode = nextHashCode();
private static AtomicInteger nextHashCode = new AtomicInteger();
private static final int HASH_INCREMENT = 0x61c88647;
private static int nextHashCode() {
      return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```



### 线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

#### synchronized 和 ReentrantLock。

#### CAS

#### 无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

**线程本地存储（Thread Local Storage）**

##### 栈封闭

多个线程访问同一个方法的**局部变量**时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

```java
public class StackClosedExample {
    public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }
}
```

```java
public static void main(String[] args) {
    StackClosedExample example = new StackClosedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> example.add100());
    executorService.execute(() -> example.add100());
    executorService.shutdown();
}
```

```html
100
100
```

#### 不可变

不可变的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，就永远也不会看到它在多个线程之中处于不一致的状态。

不可变的类型：final 关键字修饰的基本数据类型，String类型等。



### CountDownLatch

做减法，减到0唤醒主线程。

它允许一个或多个线程一直等待，直到其他线程的操作执行完后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。类似于Go语言中的`sync.WaitGroup`。

CountDownLatch主要有两个方法，当一个或多个线程调用await()方法时，调用线程（main）会被阻塞。其他线程调用 countDown()方法会将计数器减1，当计数器的值变为0时，因调用await()方法被阻塞的线程才会被唤醒，继续执行。

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
		for (int i = 1; i <= 6; i++) {//模拟6辆汽车
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



### 生产者-消费者模型

传统版

```java
/**
 * 一个初始值为零的变量，两个线程对其交替操作，一个加1一个减1，循环5轮
 * 1. 线程  操作  资源类
 * 2. 判断  干活  通知
 * 3. 防止虚假唤起机制
 */
public class ProdConsumer_TraditionDemo {
    public static void main(String[] args) {
        ShareData shareData = new ShareData();
        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                try {
                    shareData.increment();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, "ProductorA " + i).start();
        }

        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                try {
                    shareData.decrement();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, "ConsumerB  " + i).start();
        }
    }
}

class ShareData {//资源类
    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void increment() throws Exception {
        lock.lock();
        try {
            //1.判断
            while (number != 0) {
                //不能生产
                condition.await();
            }
            //2.干活
            number++;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //3.通知
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws Exception {
        lock.lock();
        try {
            //1.判断
            while (number == 0) {
                //不能消费
                condition.await();
            }
            //2.消费
            number--;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //3.通知
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```



Javaguide版阻塞队列：

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

```java
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

```html
produce..produce..consume..consume..produce..consume..produce..consume..produce..consume..
```



### 为什么调用 start() 方法时会执行 run() 方法，为什么不能直接调用 run() 方法？

调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。

直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。



### sleep() 方法和 wait() 方法区别?

- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `sleep()`方法可以在任何地方使用；`wait()`方法则只能在**同步方法或同步块**中使用；
- `sleep()`方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。`wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。
