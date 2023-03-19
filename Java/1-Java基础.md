## 重要

### String 为什么是不可变的?

`String` 类中使用 **`final` 关键字修饰字符数组**来保存字符串，**`private final char value[]`，**所以 String 对象是不可变的。



**String str = str + “world!” 改变了String的值，为什么还说String是不可变的呢？**

**String对象内容不能修改，但并不代表其引用不能改变**，下面通过内存的分配图说明字符串不可改变的真正含义：

<a href="https://sm.ms/image/2vB3PG1atZnrduW" target="_blank"><img src="https://s2.loli.net/2023/03/18/2vB3PG1atZnrduW.png" ></a>

可知，原字符串中的内容并没有任何的改变。String str = "hello"和str = str + " World";实质上是开辟了三个内存空间，str只是由原来指向"hello"变为指向“hello world!”，而其**原来的指向内容没有改变。**
因此在开发中，若要经常修改字符串的内容，应该尽量不用String，因为字符串的指向“断开-连接”会大大降低性能。对于要经常修改内容的情况，建议使用：StringBuilder、StringBuffer。



### String不可变的好处

**1. 缓存 hash 值**  

**不可变性使得 hash 值也不可变，所以在String创建的时候hashcode就被缓存了，使用时不需要重新计算。**

**2. String Pool **

**当有多个String引用指向同样的String字符串时，实际上是指向的是同一个Sting pool中的对象，而不需要额外的创建对象。只有 String 是不可变的，才可能使用 String Pool。**

**3. 线程安全**  

**因为不可变**，所以线程安全（并发场景下，对资源的写操作有危险。不可变的对象不能被写，所以线程安全）。

**4. 安全性**  

例如，**数据库的用户名、密码**都是以字符串的形式传入来获得数据库的连接。因为字符串是不可变的，所以它的值是不可改变的，否则黑客们可以钻到空子，改变字符串指向的对象的值，造成安全漏洞。



### StringBuffer 和 StringBuilder

**可变性**

 `StringBuilder` 与 `StringBuffer` 都没有用 `final` 关键字修饰，所以这两种对象都是**可变**的。

**线程安全性**

`StringBuffer` 对方法加了**synchronized锁**，所以是线程安全的。`StringBuilder` 并没有对方法加锁，所以线程不安全。**（加Buff安全）**



### 为什么重写equals方法，还必须要重写hashcode方法？

因为**Hashcode比equals方法的开销更小，速度更快。**在涉及到hashcode的容器（比如HashSet）中，判断容器是否包含该对象时，会先检查hashcode是否相等，**如果hashcode不相等，就会直接认为不相等，并存入容器中，**不会再调用equals进行比较。**如果只重写equals而不重写hashcode，就会导致，即使该对象已经存在HashSet中，但是因为hashcode不同，还会再次被存入。因此要重写hashcode，保证：如果equals判断是相等的，那hashcode值也要相等。**



### 抽象类与接口

1. 定义抽象类的关键字是abstract class，而定义接口的关键字是interface。
2. 继承抽象类的关键字是extends，而实现接口的关键字是implements。
3. **抽象类只能单继承，而接口可以多实现。**
4. 抽象类中可以有构造方法，而接口中不可以有。
5. 抽象类中可以有成员方法（方法的默认实现），而接口中只可以有抽象方法。
6. 从jdk 1.8开始，允许接口中出现非抽象方法，但需要使用default修饰。
7. **抽象类中增加方法可以不影响子类，而接口中增加方法通常都影响子类。**
8. **抽象类的访问修饰符可以是public、protected、default，而接口只有public。**
9. **描述特征（会飞的）用接口，描述概念（鸟）用抽象类。**



### 面向对象编程三大特性

#### 封装

把一个对象的属性私有化，同时提供一些可以被外界访问这些属性的方法。

####  继承

继承实现了   **IS-A**   关系，例如 Cat 和 Animal 就是一种 IS-A 关系，因此 Cat 可以继承自 Animal，从而获得 Animal 非 private 的属性和方法。

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法，子类是**无法访问，只是拥有。**
2. 子类可以重写父类方法。
3. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。

**封装和继承都提升了代码的复用性。**

####  多态

**多态分为编译时多态和运行时多态：**

- 编译时多态主要指方法的重载
- 运行时多态指对象引用所指向的具体类型在运行期间才确定

**运行时多态有三个条件：继承，重写，向上转型。**

**Animal a = new Dog();**

下面的代码中，乐器类（Instrument）有两个子类：Wind 和 Percussion，它们都重写了父类的 play() 方法，并且在 main() 方法中使用父类 Instrument 来存 Wind 和 Percussion 对象。**在 Instrument 引用调用 play() 方法时，会执行实际引用对象所在类的 play() 方法，**而不是 Instrument 类的方法。

```java
public class Instrument {

    public void play() {
        System.out.println("Instument is playing...");
    }
}
```

```java
public class Wind extends Instrument {

    public void play() {
        System.out.println("Wind is playing...");
    }
}
```

```java
public class Percussion extends Instrument {

    public void play() {
        System.out.println("Percussion is playing...");
    }
}
```

```java
public class Music {

    public static void main(String[] args) {
        List<Instrument> instruments = new ArrayList<>();
        instruments.add(new Wind());
        instruments.add(new Percussion());
        for(Instrument instrument : instruments) {
            instrument.play();
        }
    }
}
```

```
Wind is playing...
Percussion is playing...
```



### BIO,NIO,AIO

#### 阻塞与非阻塞

阻塞与非阻塞指的是**单个线程内遇到同步等待时，是否在原地不做任何操作。**

阻塞指的是遇到同步等待后，一直在原地等待同步方法处理完成。

非阻塞指的是遇到同步等待，不在原地等待，先去做其他的操作，隔一段时间再来观察同步方法是否完成。



#### 同步与异步

同步和异步指的是**一个执行流程中每个方法是否必须依赖前一个方法完成后才可以继续执行。假设我们的执行流程中，依次是方法一和方法二。**

同步指的是调用一旦开始，调用者必须等到方法调用返回后，才能继续后续的行为。即方法二一定要等到方法一执行完成后才可以执行。

异步指的是调用立刻返回，调用者不必等待方法内的代码执行结束，就可以继续后续的行为。（具体方法内的代码交由另外的线程执行完成后，可能会进行回调）。即执行方法一的时候，直接交给其他线程执行，不由主线程执行，也就不会阻塞主线程，所以方法二不必等到方法一完成即可开始执行。

同步与异步关注的是方法的执行方是主线程还是其他线程，主线程的话需要等待方法执行完成，其他线程的话无需等待立刻返回方法调用，主线程可以直接执行接下来的代码。



**BIO (Blocking I/O): 同步阻塞 I/O 模型，**应用程序发起 read 调用后，会一直阻塞，直到内核把数据拷贝到用户空间。

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a9e704af49b4380bb686f0c96d33b81~tplv-k3u1fbpfcp-watermark.image" alt="图源：《深入拆解Tomcat & Jetty》" style="zoom:50%;" />

在客户端连接数量不高的情况下，是没问题的。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。



**NIO (Non-blocking/New I/O): 同步非阻塞 I/O 模型，**也可以看作是 **I/O 多路复用模型**。

我们先来看看 **同步非阻塞 IO 模型**。

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb174e22dbe04bb79fe3fc126aed0c61~tplv-k3u1fbpfcp-watermark.image" alt="图源：《深入拆解Tomcat & Jetty》" style="zoom:50%;" />

同步非阻塞 IO 模型中，应用程序会一直发起 read 调用，等待数据从内核空间拷贝到用户空间的这段时间里，线程依然是阻塞的，直到在内核把数据拷贝到用户空间。

相比于BIO，NIO通过**轮询**操作，避免了一直阻塞。

但是，这种 IO 模型同样存在问题：**应用程序不断轮询数据是否已经准备好的过程是十分消耗 CPU 资源的。**

这个时候，**I/O 多路复用模型** 就上场了。

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88ff862764024c3b8567367df11df6ab~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

IO 多路复用模型中，线程首先发起 select 调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起 read 调用。read 调用的过程（数据从内核空间 -> 用户空间）还是阻塞的。

**IO 多路复用模型，通过减少无效的轮询，减少了对 CPU 资源的消耗。**

NIO 有一个非常重要的**选择器 ( Selector )** 的概念，也可以被称为 **多路复用器**。通过它，只需要一个线程便可以管理多个客户端连接。当客户端数据到了之后，才会为其服务。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f483f2437ce4ecdb180134270a00144~tplv-k3u1fbpfcp-watermark.image)



**AIO (Asynchronous I/O):又称NIO2， 异步非阻塞的 IO 模型。**

异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3077e72a1af049559e81d18205b56fd7~tplv-k3u1fbpfcp-watermark.image" alt="img" style="zoom:50%;" />

总结对比：

<img src="https://images.xiaozhuanlan.com/photo/2020/33b193457c928ae02217480f994814b6.png" alt="img" style="zoom:50%;" />



## 了解

### 池化思想

#### 缓存池

**Integer.valueOf() 先判断值是否在缓存池中，如果在就直接返回缓存池的内容。Integer 缓存池的大小默认为 -128\~127。**( Java 8 )

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false

Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

**编译器会在自动装箱过程调用 valueOf() 方法，**因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```



#### String Pool

**当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等**（使用 equals() 方法进行确定）**，那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。**

```java
String s1 = new String("a");
String s2 = new String("a");
System.out.println(s1 == s2);           // false

String s3 = s1.intern();
String s4 = s2.intern();
System.out.println(s3 == s4);           // true
```

**采用 String s = "b" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。**

```java
String s5 = "b";
String s6 = "b";
System.out.println(s5 == s6);  // true
```



#### String s = new String("abc")

**"abc"本身就是String Pool中的一个对象，在运行 new String()时，把String Pool中的字符串"abc"复制到堆中，并把这个对象的引用交给s，所以创建了两个String对象，一个在String Pool中，一个在堆中。**



#### 总结

new：每次都会新建一个对象。

Integer.valueOf()、Integer m = 123、String.intern()、String s = "a" ：缓存池、String Pool。



### static

**1. 静态变量**  

又称为**类变量，随着类的加载而加载，只加载一次。类所有的实例都共享静态变量，**可以直接通过类名来访问它。**静态变量在内存中只存在一份。**

**2. 静态方法**  

静态方法随着类的加载而加载。**静态方法只能调用静态成员。**



### final

**1. 变量**  

声明变量为常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使其**不能引用其它对象，但是被引用的对象本身的值是可以修改的。**

**2. 方法**  

**final方法不能被子类重写。**

**3. 类**  

**final类不允许被继承。**



### == 与 equals

**==：基本数据类型比较的是值，引用数据类型比较的是内存地址。**

if (42 == 42.0)   // 基本数据类型比较的是值，true

**equals()** :

- **情况 1：类没有覆盖 equals() 方法。等价于“==”。**

- 情况 2：类覆盖了 equals() 方法。一般，我们都覆盖 equals() 方法来比较两个对象的内容是否相等；若它们的**内容**相等，则返回 true 。

  String 中的 equals 方法是被重写过的，因为 **object 的 equals 方法是比较的对象的内存地址**，而 String 的 equals 方法比较的是对象的值。

```java
public class test1 {
    public static void main(String[] args) {
        String a = new String("ab"); // a 为一个引用
        String b = new String("ab"); // b为另一个引用,对象的内容一样
        if (a == b) // false，不同对象
        if (a.equals(b)) // true
    }
}
```



###  重载和重写的区别

**重载：同样的一个方法能够根据输入数据的不同，做出不同的处理。**

发生在同一个类中，方法名必须相同，参数类型、个数、顺序，方法返回值和访问修饰符可以不同。

**重写：当子类继承自父类的相同方法，输入数据一样，但要做出有别于父类的响应时，就要覆盖父类方法。**

1. **方法名、参数列表必须相同，返回值类型≤父类，抛出的异常范围≤父类，访问修饰符范围≥父类。“两同两小一大”**
2. 重写的返回值类型 ：如果方法的返回类型是void和基本数据类型，则返回值重写时不可修改。但是如果方法的返回值是引用类型，重写时可以返回该引用类型的子类。
3. 如果父类方法访问修饰符为 `private/static/final` 则子类就不能重写该方法。



### Object类有哪些方法？

```java
public boolean equals(Object obj)
    
public native int hashCode()

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()
```



### 浅拷贝和深拷贝？

如果在拷贝这个对象的时候，只对基本数据类型进行了拷贝，而对引用数据类型只是进行了引用的传递，而没有真实的创建一个新的对象，则认为是浅拷贝。反之，在对引用数据类型进行拷贝的时候，创建了一个新的对象，并且复制其内的成员变量，则认为是深拷贝。



### 泛型和泛型擦除

**泛型擦除：泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除掉。**

```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();
		
System.out.println(l1.getClass() == l2.getClass());
```

打印的结果为 true，因为 `List<String>`和 `List<Integer>`在 JVM 中的 Class 都是 List.class。



### 反射

反射机制是**在运行时，**对于任意一个**类**，都能够知道这个类的所有属性和方法；对于任意一个**对象**，都能够调用它的任意属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。

#### 反射机制的优缺点？

优点：

1. **能够运行时动态获取类的实例，提高灵活性。**
2. 与动态编译结合。


缺点：

1. **性能较低，**需要解析字节码，将内存中的对象进行解析。
2. **不安全，破坏了封装性**（因为通过反射可以获得私有属性和方法）。

#### 哪里会用到反射？

JDBC：

```java
class.forName('com.mysql.jdbc.Driver.class');   //加载MySQL的驱动类
```

Spring 通过 XML 装载 Bean 的过程：

1. 将程序内所有 XML 配置文件加载进内存；
2. 解析xml里面的内容，得到对应实体类的**字节码字符串**以及相关的属性信息；
3. **使用反射机制，根据这个字符串获得某个类的Class实例；**
4. 动态配置实例的属性。

**除了使用new创建对象之外，还可以用反射的方法创建对象。**



#### 通过new创建对象的效率高还是反射创建对象效率高？

通过new创建对象的效率比较高。通过反射时，先找查找类资源，使用类加载器创建，过程比较繁琐，所以效率较低。



### 异常的处理：throws

**定义：**"throw + 异常类型"写在方法的声明处，指明此方法执行时，可能会抛出的异常类型。

 一旦方法体执行时出现异常，将会在异常代码处生成一个异常类的对象，此对象满足throws异常类型 时，就会被抛出至调用者；而异常代码后续的代码不会再执行

```java
public void method2() throws IOException{
    method1();
}

public void method1 throws FileNotFoundException,IOException{
    //....
}

//throws不过是将异常一层层的向上抛出，直到抛到某一层时异常被try-catch-finally模式处理掉
//子类抛出的异常类型不能大于父类抛出的异常类型
```

try-catch-finally：真正的将异常处理掉了

throws的方式只是将异常抛给了方法的调用者，并没有解决掉



### 自定义异常类

```java
//随便继承一个比较大的异常类(比如RuntimeException,IOException)
public class MyException extends RuntimeException{
    //随便造一个UID,不与现有的重复就好了
    static final long serialVersionUID = -646484864784646L;
    
    public MyException(){
        
    }
    
    //输出信息用的构造器
    public MyException(String msg){
        super(msg);
    }
}
```
