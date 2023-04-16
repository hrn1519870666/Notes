### HashMap

#### 存储结构

内部包含了一个 Entry 类型的数组table。Entry 是一个链表，即**数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放 hashcode(key)%table.length 运算结果相同的key。**

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208234948205.png"/ width ="75%" height = "75%"> </div><br>

```java
// transient:在类实现序列化接口，而类下某个变量不想被序列化的情况下，用transient修饰该变量，可避免该变量被序列化。
transient Entry[] table;
```

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
```

#### put/get操作

**put步骤：**

1. **计算 key 的 hash 值，**通过hash&(table.length-1)计算应当存放在数组中的下标 index;
2. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
3. 存在数据，说明发生了hash冲突, 继续判断key是否相等，若相等，用新的value替换原数据；
4. 若不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；
5. 若不是红黑树，创建普通Node加入链表中；判断链表长度是否大于 8，大于则将链表转换为红黑树。

[“七上八下”的原因，为什么HashMap会产生死循环](https://juejin.cn/post/7096754757716410382)

**get步骤：**

1. 计算键值对所在的桶；
2. 在链表上顺序查找，时间复杂度显然和链表的长度成正比。

#### 扩容

|    参数    |                             含义                             |
| :--------: | :----------------------------------------------------------: |
|  capacity  | table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。 |
|    size    |                         键值对数量。                         |
| threshold  | size 的临界值，当 size 大于等于 threshold 就必须进行扩容操作。 |
| loadFactor | 装载因子，table 能够使用的比例，loadFactor=(int)(threshold/capacity)。 |

**当需要扩容时，令 capacity 为原来的两倍。**扩容使用 resize() 实现，该操作需要把 oldTable 的所有键值对重新插入 newTable 中。

#### 重新计算桶下标

在进行扩容时，需要重新计算桶下标，从而放到对应的桶上。**HashMap capacity 为 2 的 n 次方这一特点能够极大降低重新计算桶下标操作的复杂度。**

假设原数组长度 capacity 为 16，扩容之后 new capacity 为 32：

```html
capacity     : 000 1(第5位) 0000
new capacity : 001(第6位)00000
```

对于一个 Key，它的哈希值在第 5 位：

- 为 0，那么 hash%00010000 = hash%00100000，桶位置和原来一致；
- 为 1，hash%00010000 = hash%00100000 + 16，桶位置是原位置 + 16。

#### 链表转红黑树

从 JDK 1.8 开始，一个桶存储的链表长度**大于等于 8** 时会将链表转换为红黑树（JDK 1.7的HashMap是数组+链表，没有红黑树）。



#### HashMap 的长度为什么是2的幂次方?

**为了能让 HashMap 存取高效，尽量减少碰撞，我们首先会想到采用%取余的操作来实现。取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作，也就是说如果length 是2的 n 次方，则 hash%length==hash&(length-1) 。采用二进制位操作 &，相对于%，能够提高运算效率。**



#### 为什么长度大于8转换成红黑树？

大量实验发现，hash碰撞发生8次的概率非常低，几乎为不可能事件。**如果真的碰撞发生了8次，说明是元素本身和hash函数的原因，后续可能还会继续发生hash碰撞。**所以这时就应该将链表转换为红黑树。 红黑树转链表的阈值为6主要是因为，如果也将该阈值设置于8，那么当hash碰撞在8时，会导致链表和红黑树不停相互转换，浪费资源。



### LinkedHashMap

LinkedHashMap 在HashMap的基础上，增加了一条双向链表，可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。

<img src="https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166338955699.jpg" alt="img" style="zoom:50%;" />



上图中，淡蓝色的箭头表示前驱引用，红色箭头表示后继引用。每当有新键值对节点插入，新节点最终会接在 tail 引用指向的节点后面。而 tail 引用则会移动到新的节点上，这样一个双向链表就建立起来了。

#### 删除步骤

1. 根据 hash 定位到桶位置
2. 遍历链表或调用红黑树相关的删除方法
3. 从 LinkedHashMap 维护的双链表中移除要删除的节点

假如我们要删除下图键值为 3 的节点：

<img src="https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166940421133.jpg" alt="img" style="zoom:50%;" />

<img src="https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166936737479.jpg" alt="img" style="zoom:50%;" />

#### 访问顺序的维护过程

LinkedHashMap 是按插入顺序维护链表。不过我们可以在初始化 LinkedHashMap，指定 accessOrder 参数为 true，即可让它按访问顺序维护链表。访问顺序的原理上并不复杂，当我们调用`get/getOrDefault/replace`等方法时，只需要将这些方法访问的节点移动到链表的尾部即可。
<img src="https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15166338955699.jpg" alt="img" style="zoom:50%;" />



访问后，键值为3的节点将会被移动到双向链表的最后位置，其前驱和后继也会跟着更新。访问后的结构如下：



<img src="https://blog-pictures.oss-cn-shanghai.aliyuncs.com/15167010301496.jpg" alt="img" style="zoom:50%;" />



### TreeMap

与HashMap相比，TreeMap是一个能比较元素大小的Map集合，会对传入的key进行了大小排序。可以使用元素的自然顺序，也可以使用集合中自定义的比较器来进行排序。线程不安全。底层是红黑树。

```java
// 默认构造函数。使用该构造函数，TreeMap中的元素按照自然排序进行排列。
TreeMap()

// 指定Tree的比较器
TreeMap(Comparator<? super K> comparator)
```



### HashTable

HashTable线程是安全的，底层采用synchronized把整个table锁住解决了线程安全的问题，但这样多线程最终变为单线程在执行，效率非常低。

<img src="https://upload-images.jianshu.io/upload_images/25147367-54da6d95442dc72c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp" alt="img" style="zoom:50%;" />



###  HashMap 和 Hashtable 的区别

1. **线程是否安全：** `HashMap` 是线程**不安全**的，`HashTable` 是线程**安全**的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰。

2. **效率：** 因为线程安全的问题，`HashMap` 要比 `HashTable` 效率高一点。

3. **底层数据结构：** JDK1.8 中，当HashMap的链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间。Hashtable 没有这样的机制。



### ConcurrentHashMap

#### 存储结构

ConcurrentHashMap 采用了分段锁（Segment）（锁分离技术），**每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是分段锁的个数）。**默认的并发级别为 16，也就是说默认创建 16 个 Segment。

![img](https://upload-images.jianshu.io/upload_images/5220087-8c5b0cc951e61398.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

#### put操作

进行两次Hash去定位数据的存储位置

```dart
static class  Segment<K,V> extends  ReentrantLock implements  Serializable {
}
```

从上Segment的继承体系可以看出，Segment实现了ReentrantLock,也就带有锁的功能，当执行put操作时，会进行第一次key的hash来定位Segment的位置，如果该Segment还没有初始化，即通过CAS操作进行赋值，然后进行第二次hash操作，找到相应的HashEntry的位置，这里会利用继承过来的锁的特性，在将数据插入指定的HashEntry位置时（链表的尾端），会通过继承ReentrantLock的tryLock（）方法尝试去获取锁，如果获取成功就直接插入相应的位置，如果已经有线程获取该Segment的锁，那当前线程会以自旋的方式去继续的调用tryLock（）方法去获取锁，超过指定次数就挂起，等待唤醒。

#### get操作

第一次需要经过一次hash定位到Segment的位置，然后再hash定位到指定的HashEntry，遍历该HashEntry下的链表进行对比。

#### size操作

计算size的时候，ConcurrentHashMap可能还在并发的插入数据，导致计算出来的size和实际的size有相差（在return size的时候，插入了多个数据）。

使用不加锁的模式去尝试多次计算ConcurrentHashMap的size，最多三次，比较前后两次计算的结果，结果一致就认为当前没有元素加入，计算的结果是准确的。否则，他就会给每个Segment加上锁，然后计算ConcurrentHashMap的size返回。

#### JDK 1.8 的改动

JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用 synchronized，并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。



<img src="https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/source-code/dubbo/java8_concurrenthashmap.png" alt="Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）" style="zoom:33%;" />



**JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock？**

因为粒度降低了，在相对而言的低粒度加锁方式，synchronized并不比ReentrantLock差，在粗粒度加锁中ReentrantLock可能通过Condition来控制各个低粒度的边界，更加的灵活，而在低粒度中，Condition的优势就没有了。



### HashSet的底层是HashMap，为什么前者存储的是一个元素，而后者存储的是KV键值对？

HashSet底层确实是一个HashMap，存储的值放在HashMap的key里，value存储了一个PRESENT的静态Object对象。



### ArrayList

ArrayList 是**基于数组实现的，数组的默认大小为 10，**支持快速随机访问。

#### 扩容

**新容量大约是旧容量的 1.5 倍左右。**（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）

**扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中**，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

#### 删除元素

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(n)，可以看到 ArrayList 删除元素的代价是非常高的。



### LinkedList

**基于双向链表实现**，使用 Node 存储链表节点信息。每个链表存储了 first 和 last 指针：

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208233940066.png"/ width ="75%" height = "75%"> </div><br>




### Vector

#### 同步

它的实现与 ArrayList 类似，但是**使用了 synchronized 进行同步。**

#### 扩容

**Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量增长 capacityIncrement。**如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的**两倍。**

#### 线程安全的替代方案

使用 `Collections.synchronizedList();` 得到一个线程安全的 ArrayList。

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
```

Collections是一个集合类，Collection是一个接口，其实现类有List，Set等。

**使用 JUC 下的 CopyOnWriteArrayList 类。**

```java
List<String> list = new CopyOnWriteArrayList<>();
```



### CopyOnWriteArrayList

#### 读写分离

**写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。**

**写操作需要加锁，防止并发写入时导致写入数据丢失。写操作结束之后需要把原始数组指向新的复制数组。**

#### 适用场景

CopyOnWriteArrayList **在写操作的同时允许读操作，**大大提高了读操作的性能，因此很适合**读多写少**的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- **内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；**
- **数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。**



### Arraylist 与 LinkedList 区别?

2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是**双向链表** 。
3. **数据访问：**ArrayList要优于LinkedList，因为ArrayList底层采用数组实现，LinkedList采用线性的链式存储结构，需要移动指针来访问。
4. **增删效率：**LinkedList要优于ArrayList，ArrayList增删要复制元素，影响其它数组元素的下标，而LinkedList只需要移动指针。
4. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是**线程不安全**。



### ArrayList 与 Vector 区别?

- 底层都使用 `Object`数组存储。
- `ArrayList` 是 `List` 的主要实现类，线程**不安全** 。`Vector` 是 `List` 的古老实现类，线程**安全**。
- Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。



### 集合和数组的区别

- 数组的长度是固定的，**集合的长度是可变的。**
- 数组可以存储基本数据类型，也可以存储引用数据类型，**集合只能存储引用数据类型。**
- 数组存储的元素必须是同一个数据类型，**集合存储的对象可以是不同类型。**



### Fail-Fast和Fail-Safe

**fail-fast：**快速失败。当异常产生时，直接抛出异常，程序终止。

当我们在遍历集合元素的时候，经常会使用迭代器，但在迭代器遍历元素的过程中，如果集合的结构被改变的话，就会抛出异常`ConcurrentModificationException`，防止继续遍历。这就是所谓的快速失败机制。这里说的结构被改变,是例如插入和删除这种操作,只是改变集合里的值的话并不会抛出异常。

原理：expectedModCount默认值等于开始遍历时的集合元素个数，使用modCount变量，集合在被遍历的过程中，如果内容发生变化，则modCount就会发生改变。每次迭代器遍历下一个元素之前，就会先比较modCount是否与expectedModCount相等。如果是就返回遍历，不是的话就抛出异常。

**fail-safe：**安全失败

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问，而是**先复制原有集合内容，在拷贝的集合上进行遍历。**

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。

优缺点：基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改，迭代器是不知道的。
