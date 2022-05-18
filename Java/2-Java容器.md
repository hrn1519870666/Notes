### 容器

主要包括 Collection 和 Map 两种

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208220948084.png"/> </div><br>

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20201101234335837.png"/> </div><br>



### 集合和数组的区别

- 数组的长度是固定的，**集合的长度是可变的。**
- 数组可以存储基本数据类型，也可以存储引用数据类型，**集合只能存储引用数据类型。**
- 数组存储的元素必须是同一个数据类型，**集合存储的对象可以是不同类型。**



### ArrayList


#### 1. 概览

ArrayList 是**基于数组实现的，数组的默认大小为 10，**支持快速随机访问。

#### 2. 扩容

**新容量大约是旧容量的 1.5 倍左右。**（oldCapacity 为偶数就是 1.5 倍，为奇数就是 1.5 倍-0.5）

**扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中**，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

#### 3. 删除元素

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(n)，可以看到 ArrayList 删除元素的代价是非常高的。



### LinkedList

**基于双向链表实现**，使用 Node 存储链表节点信息。每个链表存储了 first 和 last 指针：

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208233940066.png"/> </div><br>




### Vector

#### 1. 同步

它的实现与 ArrayList 类似，但是**使用了 synchronized 进行同步。**

#### 2. 扩容

**Vector 的构造函数可以传入 capacityIncrement 参数，它的作用是在扩容时使容量增长 capacityIncrement。**如果这个参数的值小于等于 0，扩容时每次都令 capacity 为原来的**两倍。**

#### 3. 线程安全的替代方案

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

#### 1. 读写分离

**写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。**

**写操作需要加锁，防止并发写入时导致写入数据丢失。写操作结束之后需要把原始数组指向新的复制数组。**

#### 2. 适用场景

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



### HashMap

以下源码分析以 JDK 1.7 为主。

#### 1. 存储结构

内部包含了一个 Entry 类型的数组table。Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。即**数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放 hashcode%table.length 运算结果相同的 Entry。**

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208234948205.png"/> </div><br>

```java
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

#### 2. put操作

**步骤：**

1. **计算 key 的 hash 值，**通过hash&(table.length-1)计算应当存放在数组中的下标 index;
2. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
3. 存在数据，说明发生了hash冲突, 继续判断key是否相等，若相等，用新的value替换原数据；
4. 若不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；
5. 若不是红黑树，创建普通Node加入链表中；判断链表长度是否大于 8，大于则将链表转换为红黑树；
6. 插入完成之后判断当前节点数是否大于阈值threshold，若大于，则扩容为原数组的二倍。

举例：

```java
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```

- 新建一个 HashMap，默认大小为 16；
- 插入 &lt;K1,V1\> 键值对，先计算 K1 的 hashCode 为 115，使用除留余数法得到所在的桶下标 115%16=3。
- 插入 &lt;K2,V2\> 键值对，先计算 K2 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6。
- 插入 &lt;K3,V3\> 键值对，先计算 K3 的 hashCode 为 118，使用除留余数法得到所在的桶下标 118%16=6，插在 &lt;K2,V2\> 前面。

应该注意到链表的插入是以头插法方式进行的，例如上面的 &lt;K3,V3\> 不是插在 &lt;K2,V2\> 后面，而是插入在链表头部。**（七上八下）**

**查找需要分成两步进行：**

- **计算键值对所在的桶；**
- **在链表上顺序查找，时间复杂度显然和链表的长度成正比。**

<div align="center"> <img src="https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/image-20191208235258643.png"/> </div><br>

#### 3. 扩容

和扩容相关的参数主要有：capacity、size、threshold 和 loadFactor。

| 参数 | 含义 |
| :--: | :-: |
| capacity | table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。|
| size | 键值对数量。 |
| threshold | size 的临界值，当 size 大于等于 threshold 就必须进行扩容操作。 |
| loadFactor | 装载因子，table 能够使用的比例，loadFactor=(int)(threshold/capacity)。 |

**当需要扩容时，令 capacity 为原来的两倍。**扩容使用 resize() 实现，该操作需要把 oldTable 的所有键值对重新插入 newTable 中，因此这一步是很费时的。

#### 4. 重新计算桶下标

在进行扩容时，需要重新计算桶下标，从而放到对应的桶上。**HashMap capacity 为 2 的 n 次方这一特点能够极大降低重新计算桶下标操作的复杂度。**

假设原数组长度 capacity 为 16，扩容之后 new capacity 为 32：

```html
capacity     : 000 1(第5位) 0000
new capacity : 00100000
```

对于一个 Key，它的哈希值 hash 在第 5 位：

- 为 0，那么 hash%00010000 = hash%00100000，桶位置和原来一致；
- 为 1，hash%00010000 = hash%00100000 + 16，桶位置是原位置 + 16。

#### 5. 链表转红黑树

从 JDK 1.8 开始，一个桶存储的链表长度**大于等于 8** 时会将链表转换为红黑树。



#### HashMap 的长度为什么是2的幂次方?

**为了能让 HashMap 存取高效，尽量较少碰撞，我们首先会想到采用%取余的操作来实现。取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作，也就是说如果length 是2的 n 次方，则 hash%length==hash&(length-1) 。采用二进制位操作 &，相对于%，能够提高运算效率。**



#### 为什么长度大于8转换成红黑树？

大量实验发现，hash碰撞发生8次的概率非常低，几乎为不可能事件。**如果真的碰撞发生了8次，说明是元素本身和hash函数的原因，后续可能还会继续发生hash碰撞。**所以这时就应该将链表转换为红黑树。 红黑树转链表的阈值为6主要是因为，如果也将该阈值设置于8，那么当hash碰撞在8时，会导致链表和红黑树不停相互转换，浪费资源。



### ConcurrentHashMap

#### 1. 存储结构

ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），**每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是分段锁的个数）。**默认的并发级别为 16，也就是说默认创建 16 个 Segment。

#### 2. size 操作

每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。

在执行 size 操作时，需要遍历所有 Segment 然后把 count 累计起来。

ConcurrentHashMap 在执行 size 操作时先尝试不加锁，如果连续两次不加锁操作得到的结果一致，那么可以认为这个结果是正确的。默认的尝试次数为 3，如果尝试的次数超过 3 次，就需要对每个 Segment 加锁。

#### 3. JDK 1.8 的改动

JDK 1.7 使用分段锁机制来实现并发更新操作，核心类为 Segment，JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用 synchronized，并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。



###  HashMap 和 Hashtable 的区别

1. **线程是否安全：** `HashMap` 是线程**不安全**的，`HashTable` 是线程**安全**的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰。

2. **效率：** 因为线程安全的问题，`HashMap` 要比 `HashTable` 效率高一点。

5. **底层数据结构：** JDK1.8 中，当HashMap的链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间。Hashtable 没有这样的机制。

   

### ConcurrentHashMap 和 Hashtable 的区别

- **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树。`Hashtable` 是采用 **数组+链表** 的形式。
- **实现线程安全的方式（重要）：** ① **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段，每一把锁只锁容器中的一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作，** 整个看起来就像是优化过且线程安全的 `HashMap`。② **`Hashtable`** 使用 `synchronized` 来保证线程安全，效率非常低下。当一个线程访问同步方法时，若其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get。

**JDK1.7 的 ConcurrentHashMap：**

![JDK1.7的ConcurrentHashMap](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/ConcurrentHashMap分段锁.jpg)

<p style="text-align:right;font-size:13px;color:gray">http://www.cnblogs.com/chengxiao/p/6842045.html></p>



![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/source-code/dubbo/java8_concurrenthashmap.png)



### HashSet的底层是HashMap，为什么前者存储的是一个元素，而后者存储的是KV键值对？

HashSet底层确实是一个HashMap，存储的值放在HashMap的key里，value存储了一个PRESENT的静态Object对象。



### Fail-Fast和Fail-Safe

**fail-fast：**快速失败。当异常产生时，直接抛出异常，程序终止。

当我们在遍历集合元素的时候，经常会使用迭代器，但在迭代器遍历元素的过程中，如果集合的结构被改变的话，就会抛出异常`ConcurrentModificationException`，防止继续遍历。这就是所谓的快速失败机制。这里说的结构被改变,是例如插入和删除这种操作,只是改变集合里的值的话并不会抛出异常。

原理：在遍历的时候使用modCount变量，集合在被遍历的过程中，如果内容发生变化，则modCount就会发生改变。每次迭代器遍历下一个元素之前，就会先比较modCount是否与expectedModCount相等，expectedModCount默认值等于开始遍历时的集合元素个数，如果是就返回遍历，不是的话就抛出异常。

**fail-safe：**安全失败

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问，而是**先复制原有集合内容，在拷贝的集合上进行遍历。**

原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。

优缺点：基于拷贝内容的优点是避免了ConcurrentModificationException，但同样地，迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改，迭代器是不知道的。



**迭代器：**提供一种访问集合对象各个元素的途径，同时又不需要暴露该对象的内部细节。java通过提供Iterator和Iterable俩个接口来实现集合类的可迭代性，迭代器主要的用法是：首先用hasNext（）作为循环条件，再用next（）方法得到每一个元素，最后进行相关的操作。
