### 进程和线程?

进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。

线程是一个比进程更小的执行单位。进程在其执行的过程中可以产生多个线程。**同类的多个线程**共享进程的**堆**和**方法区**资源，但每个线程有自己的**程序计数器**、**虚拟机栈**和**本地方法栈**，所以系统在**产生一个线程，或是在各个线程之间切换时，开销要比进程小得多。**进程作为**资源分配**的基本单位，线程作为**资源调度**的基本单位。



### 线程有哪些基本状态?

Java 线程在运行的生命周期中的指定时刻只可能处于下面 **6 种**不同状态的其中一个状态

![Java线程的状态](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%8A%B6%E6%80%81.png)

Java 线程状态变迁如下图所示

![Java线程状态变迁](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/19-1-29/Java%20%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E8%BF%81.png)



### 什么是上下文切换?

多线程编程中一般线程的个数都大于 CPU 核心的个数，而一个 CPU 核心在任意时刻只能被一个线程使用，为了让这些线程都能得到有效执行，CPU 采取的策略是为每个线程分配时间片并轮转的形式。当一个线程的时间片用完的时候就会重新处于就绪状态让给其他线程使用，这个过程就属于一次上下文切换。

概括来说就是：当前任务在执行完 CPU 时间片切换到另一个任务之前会先保存自己的状态，以便下次再切换回这个任务时，可以再加载这个任务的状态。**任务从保存到再加载的过程就是一次上下文切换**。

上下文切换通常是计算密集型的。也就是说，它需要相当可观的处理器时间，在每秒几十上百次的切换中，每次切换都需要纳秒量级的时间。所以，上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。



### 为什么我们调用 start() 方法时会执行 run() 方法，为什么我们不能直接调用 run() 方法？

new 一个 Thread，线程进入了新建状态。调用 `start()`方法，会启动一个线程并使线程进入了就绪状态，当分配到时间片后就可以开始运行了。 `start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 main 线程下的普通方法去执行，并不会在某个线程中执行它，所以这并不是多线程工作。

**总结： 调用 `start()` 方法方可启动线程并使线程进入就绪状态，直接执行 `run()` 方法的话不会以多线程的方式执行。**



### 写一个死锁的例子

```java
public class DeadLock {
 
    public static Object t1 = new Object();
    public static Object t2 = new Object();
 
    public static void main(String[] args){
        // 线程1
        new Thread(){
            @Override
            public void run(){
                synchronized (t1){
                    System.out.println("Thread1 get t1");
 
                    try {
                        Thread.sleep(100);
                    }catch (Exception e){
 
                    }
 
                    synchronized (t2){
                        System.out.println("Thread2 get t2");
                    }
                }
            }
        }.start();
        
 		// 线程2
        new Thread(){
            @Override
            public void run(){
                synchronized (t2){
                    System.out.println("Thread2 get t2");
 
                    try {
                        Thread.sleep(100);
                    }catch (Exception e){
 
                    }
 
                    synchronized (t1){
                        System.out.println("Thread2 get t1");
                    }
                }
            }
        }.start();
    }
```

输出：

```
Thread1 get t1
Thread2 get t2
```

对线程 2 的代码修改成下面这样就不会产生死锁了：

```java
	// 线程2
    new Thread(){
        @Override
        public void run(){
            // 先锁t1
            synchronized (t1){
                System.out.println("Thread2 get t1");
 
                try {
                    Thread.sleep(100);
                }catch (Exception e){
 
                }
 
                synchronized (t2){
                    System.out.println("Thread2 get t2");
                }
            }
        }
    }.start();
}
```

输出

```
Thread1 get t1
Thread1 get t2
Thread2 get t1
Thread2 get t2
```

线程 1 首先获得到t1的锁，然后再去获取t2的监视器锁，可以获取到。然后线程 1 释放了对t1、t2的锁的占用，线程 2 获取到就可以执行了。**破坏了循环等待条件，**因此避免了死锁。



### sleep() 方法和 wait() 方法区别?

- **`sleep()` 方法没有释放锁，而 `wait()` 方法释放了锁** 。
- `sleep()`方法可以在任何地方使用；`wait()`方法则只能在同步方法或同步块中使用；
- `sleep()`方法执行完成后，线程会自动苏醒。或者可以使用 `wait(long timeout)` 超时后线程会自动苏醒。`wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()`或者 `notifyAll()` 方法。



### synchronized 关键字

**`synchronized` 是一种互斥锁，解决的是多个线程之间访问资源的同步性问题，它可以保证被修饰的方法或者代码块在任意时刻只能有一个线程执行。**

#### 用法

##### **1. 同步一个代码块**  

同步代码块可以选择以什么来加锁，比同步方法要更细颗粒度，我们可以选择只同步会发生同步问题的部分代码而不是整个方法。

```java
public void func() {
    synchronized (this) {
        // ...
    }
}
```

**它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。**

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

```java
public class SynchronizedExample {

    public void func1() {
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
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
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
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
```

```html
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

##### 2.同步一个方法：作用于同一个对象。

以实例对象作为锁，进入同步代码前需要获得当前实例对象的锁。

Java通过 对象.方法 的形式调用方法，锁的就是点之前的对象。

##### **3. 同步一个类**  

以类对象为锁，进入同步代码块前需要获得当前类对象的锁。

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



#### 底层原理

#####  修饰代码块的情况

**`synchronized` 修饰代码块时，使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 指令则指明结束位置。**

**在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。**

如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

#####  修饰方法的的情况

**`synchronized` 修饰方法时，使用的是`ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。在进入该方法之前先获取相应的锁，锁的计数器加1，方法结束后计数器减1。**如果获取失败就阻塞，直到该锁被释放。



#### synchronized和 volatile的区别

1. `volatile`本质是在告诉JVM，当前变量在寄存器中的值是不确定的，需要从主存中读取； `synchronized`则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2. `volatile` 关键字是线程同步的轻量级实现，不需要加锁，不会阻塞线程，所以`volatile`性能比`synchronized`关键字要好。
3. `volatile` 关键字只能用于变量，而 `synchronized` 关键字可以修饰代码块、方法、类等。
4. `volatile` 关键字能保证数据的可见性，但不能保证数据的原子性。`synchronized` 关键字两者都能保证。



#### synchronized 和 ReentrantLock 的区别

**1.原始构成**

synchronized关键字属于JVM

Lock是具体类，是API层面的锁（java.util.） 

**2.使用方法**

sychronized不需要手动释放锁，当synchronized代码执行完后系统会自动让线程释放对锁的占用。

ReentrantLock则需要手动释放锁，若没有主动释放锁，可能导致死锁，需要lock()和 unlock()方法配合try/finally语句块来完成。

**3.等待可中断**  

含义：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**  

synchronized 中的锁是非公平的（通过CAS来抢占锁资源），ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**  

一个 ReentrantLock 可以同时绑定多个 Condition 对象。ReentrantLock用来实现分组唤醒需要要唤醒的线程们，可以精确唤醒，而不是像synchronized要么随机 唤醒一个线程要么唤醒全部线程。

**对线程之间按顺序调用，实现A>B>C三个线程启动，要求如下： A打印5次，B打印10次，C打印15次 * 紧接着A打印5次，B打印10次，C打印15次...循环10轮。**

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

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。



#### synchronized升级（膨胀）过程

https://blog.csdn.net/tongdanping/article/details/79647337

在JDK 1.6之前是重量级锁，依赖底层操作系统的 mutex 相关指令实现，所以会有用户态和内核态之间的切换，性能损耗十分明显。而JDK1.6 以后引入偏向锁和轻量级锁在JVM层面实现加锁的逻辑，不依赖底层操作系统。从此以后锁的状态就有了四种，级别由低到高依次为：无锁、偏向锁、轻量级锁、重量级锁。四种状态会随着竞争的情况逐渐升级，而且是不可逆的过程。

##### 无锁

无锁是指没有对资源进行锁定，所有的线程都能访问并修改同一个资源，但同时只有一个线程能修改成功。
线程会不断的尝试修改共享资源，如果没有冲突就修改成功并退出，否则就会继续循环尝试。如果有多个线程修改同一个值，必定会有一个线程能修改成功，而其他修改失败的线程会不断重试直到修改成功。

##### 偏向锁

偏向锁指的就是JVM会认为只有一个线程执行同步代码（没有竞争的环境），所以在Mark Word会直接记录线程ID，只要线程来执行代码，就会比对线程ID是否相等，相等则当前线程能直接获取得到锁，如果不相等，则用CAS来尝试修改当前的线程ID，如果CAS修改成功，那还是能获取得到锁，如果CAS失败了，说明有竞争环境，此时会对偏向锁撤销，升级为轻量级锁。

##### 轻量级锁（自旋锁）

一旦有第二个线程加入锁竞争，偏向锁就升级为轻量级锁。其他线程会通过自旋的形式尝试获取锁，线程不会阻塞，从而提高性能。

##### 重量级锁

忙等是有限度的（计数器记录自旋次数，默认允许循环10次）。如果锁竞争情况严重，达到最大自旋次数的线程，会将轻量级锁升级为重量级锁（依然是CAS修改锁标志位，但不修改持有锁的线程ID）。当后续线程尝试获取锁时，发现被占用的锁是重量级锁，则直接进入阻塞状态（而不是忙等），等待将来被唤醒。

##### 各种锁的适用场景

只有一个线程进入临界区，偏向锁。

**多个线程交替进入临界区，轻量级锁。**

**多线程同时进入临界区，重量级锁。**



#### synchronized为什么是非公平锁？

偏向锁：如果当前线程ID与markword存储的不相等，则CAS尝试更换线程ID，CAS成功就获取得到锁，CAS失败则升级为轻量级锁。

轻量级锁：通过CAS来抢占锁资源（只不过多了拷贝Mark Word到Lock Record的过程），抢占成功到锁就归属给该线程，但自旋失败一定次数后升级重量级锁。

重量级锁：通过monitor对象中的队列存储线程，但线程进入队列前，还是会先尝试获取得到锁，如果能获取不到才进入线程等待队列中。

综上所述，synchronized无论处理哪种锁，都是先尝试获取，获取不到才升级或者进入队列，所以是非公平的。



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



阻塞队列版

```java
public class ProdConsumer_BlockQueueDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource(new ArrayBlockingQueue<>(10));
        
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "生产线程启动");
            try {
                myResource.myProd();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "Producer").start();
        
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "消费线程启动");
            try {
                myResource.myConsumer();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }, "Consumer").start();

        try { TimeUnit.SECONDS.sleep(5); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("5s后main叫停，线程结束");
        try {
            myResource.stop();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class MyResource {
    private volatile boolean flag = true;//默认开启，进行生产+消费，volatile
    private AtomicInteger atomicInteger = new AtomicInteger();

    // 不要写死，传接口更通用
    BlockingQueue<String> blockingQueue = null;

    public MyResource(BlockingQueue<String> blockingQueue) {
        this.blockingQueue = blockingQueue;
        // 打印日志，输出实现类
        System.out.println(blockingQueue.getClass().getName());
    }

    public void myProd() throws Exception {
        String data = null;
        boolean retValue;
        while (flag) {
            data = atomicInteger.incrementAndGet() + "";
            retValue = blockingQueue.offer(data, 2, TimeUnit.SECONDS);
            if (retValue) {
                System.out.println(Thread.currentThread().getName() + "插入队列" + data + "成功");
            } else {
                System.out.println(Thread.currentThread().getName() + "插入队列" + data + "失败");
            }
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println(Thread.currentThread().getName() + "大老板叫停，生产结束");
    }

    public void myConsumer() throws Exception {
        String result = null;
        while (flag) {
            result = blockingQueue.poll(2, TimeUnit.SECONDS);
            if (null == result || result.equalsIgnoreCase("")) {
                flag = false;
                System.out.println(Thread.currentThread().getName() + "超过2s没有取到资源，消费退出");
                return;
            }
            System.out.println(Thread.currentThread().getName() + "消费队列" + result + "成功");
        }
    }

    public void stop() throws Exception {
        flag = false;
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





### ThreadLocal

`ThreadLocal`是一个在多线程中为每一个线程创建单独的变量副本的类。**如果你创建了一个`ThreadLocal`类型的变量，那么访问这个变量的每个线程都会有这个变量的本地副本,** 避免因多线程操作共享变量而导致的数据不一致的情况，保证线程安全**（除了加锁方式以外，保证线程安全的方式）。**

#### 原理

`ThreadLocal`类的`set()`方法

```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

在这个方法内部我们看到，首先通过getMap(Thread t)方法获取一个和当前线程相关的ThreadLocalMap，然后将变量的值设置到ThreadLocalMap对象中，如果获取到的ThreadLocalMap对象为空，就通过createMap方法创建。

线程隔离的秘密，就在于ThreadLocalMap类。**ThreadLocalMap是ThreadLocal类的一个静态内部类，每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象（你所设置的对象）为 value 的键值对。**它实现了键值对的设置和获取，**每个线程中都有一个独立的ThreadLocalMap副本，它所存储的值，只能被当前线程读取和修改。ThreadLocal类通过操作每一个线程特有的ThreadLocalMap副本，从而实现了变量访问，在不同线程中的隔离。因为每个线程的变量都是自己特有的，完全不会有并发错误。**



###  AQS 

 AQS的全称为`AbstractQueuedSynchronizer`，抽象的队列式的同步器。

**AQS 是一个用来构建锁和同步器的框架。**比如我们提到的 `ReentrantLock`，`Semaphore`，其他的诸如 `ReentrantReadWriteLock`，`SynchronousQueue`，`FutureTask` 等等皆是基于 AQS 的。

#### 原理

**AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gtt57mncpoj60nc0hk0tx02.jpg" alt="img" style="zoom:50%;" />

> CLH队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。

 AQS原理图：

![AQS原理图](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/AQS原理图.png)

AQS 使用一个 int 变量state来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。

```java
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```

#### AQS 对资源的共享方式

- **Exclusive**（独占）：只有一个线程能执行，如 `ReentrantLock`。又可分为公平锁和非公平锁。
- **Share**（共享）：多个线程可同时执行，如`CountDownLatch`、`Semaphore`、 `CyclicBarrier`、`ReadWriteLock` 。

自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS 已经在顶层实现好了。



### 锁优化

#### 锁消除

**锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。**
锁消除主要是通过**逃逸分析**来支持，**如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。**
对于一些看起来没有加锁的代码，其实隐式的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

```java
public static String concatString(String s1, String s2, String s3) {
    return s1 + s2 + s3;
}
```

String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

```java
public static String concatString(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```

每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

#### 锁粗化

**如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。**上一节的示例代码中连续的 append() 方法就属于这类情况。**虚拟机如果探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。**对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。



### 线程安全

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

#### 1.不可变

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

#### 2.互斥同步

synchronized 和 ReentrantLock。

#### 3.非阻塞同步

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。CAS

#### 4.无同步方案

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

##### 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

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

##### 线程本地存储（Thread Local Storage）
