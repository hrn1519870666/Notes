#### 自动类型提升

所有的byte型,short型和char型在运算时（包括同类型之间运算）都将被提升到int类型（注：使用++/+=/*=等运算符时不会被自动提升）

如果两个操作数其中有一个是double类型，另一个操作就会转换为double类型。
否则，如果其中一个操作数是float类型，另一个将会转换为float类型。
否则，如果其中一个操作数是long类型，另一个会转换为long类型。
否则，两个操作数都转换为int类型。

```java
   char a = 'a';
   char b = 'b';
   a = a + b; //不正确,(a+b)自动提升为int类型
   a = a + 1; //不正确,(a+1)自动提升为int类型
   a++; //正确,++运算符不改变类型
   
   long a = 10000000000l;
   float b = 2.0f;
           
   a = b + a; //错误,(a+b)自动提升为float类型
   b = a + b; //正确
```



#### 如何从键盘获取不同类型的变量：需要使用Scanner类

具体实现步骤：

1.导包：import java.util.Scanner;

2.Scanner的实例化:Scanner scan = new Scanner(System.in);

3.调用Scanner类的相关方法（next() / nextXxx()），来获取指定类型的变量

注意： 需要根据相应的方法，来输入指定类型的值。如果输入的数据类型与要求的类型不匹配时，会报异常InputMisMatchException导致程序终止。

```java
import java.util.Scanner;


class ScannerTest{

public static void main(String[] args){
    //Scanner的实例化
    Scanner scan = new Scanner(System.in);

    //调用Scanner类的相关方法
    String name = scan.next();
    System.out.println(name);

    int age = scan.nextInt();
    System.out.println(age);

    double weight = scan.nextDouble();
    System.out.println(weight);

    boolean choice = scan.nextBoolean();
    System.out.println(choice);

    //对于char型的获取，Scanner没有提供相关的方法。只能获取一个字符串
    String gender = scan.next();
    char genderChar = gender.charAt(0);//获取索引为0位置上的字符
    System.out.println(genderChar);
}
```



#### 数组的声明与初始化

```java
int[] ids;//声明
//静态初始化:数组的初始化和数组元素的赋值操作同时进行
ids = new int[]{1001,1002,1003,1004};
//动态初始化:数组的初始化和数组元素的赋值操作分开进行
String[] names = new String[5];

//静态初始化
int[][] arr1 = new int[][]{{1,2,3},{4,5},{6,7,8}};
//动态初始化1
String[][] arr2 = new String[3][2];
//动态初始化2
String[][] arr3 = new String[3][];
```



#### java.util.Arrays

操作数组的工具类，里面定义了很多操作数组的方法：

1. boolean equals(int[] a,int[] b)
2. void fill(int[] a,int val):将指定值填充到数组之中
3. void sort(int[] a)
4. int binarySearch(int[] a,int key)：使用二分查找查找指定值,返回下标

```java
import java.util.Arrays;

public class ArraysTest {

public static void main(String[] args) {

    //1.boolean equals(int[] a,int[] b):判断两个数组是否相等。
    int[] arr1 = new int[]{1,2,3,4};
    int[] arr2 = new int[]{1,3,2,4};
    boolean isEquals = Arrays.equals(arr1, arr2);

    //2.void fill(int[] a,int val):将指定值填充到数组之中。
    Arrays.fill(arr1,10);

    //3.void sort(int[] a):对数组进行排序。
    Arrays.sort(arr2);

    //4.int binarySearch(int[] a,int key)
    int[] arr3 = new int[]{-98,-34,2,34,54,66,79,105,210,333};
    int index = Arrays.binarySearch(arr3, 210);
}
```



局部变量没有默认初始化值，在调用之前必须要显式赋值。 特别的，形参在调用时才需要赋值。

在内存中加载的位置：

属性：加载到堆空间中(非static)。

局部变量：加载到栈空间。



#### 值传递机制

如果参数是基本数据类型，此时实参赋给形参的是实参真实存储的数据值。如果参数是引用数据类型，此时实参赋给形参的是实参存储数据的地址值。



#### JaveBean

所谓JavaBean，是指符合如下标准的Java类：

1. 类是公共的
2. 有一个无参的公共的构造器
3. 有属性，且有对应的get、set方法



#### instanceof

`x instanceof A`:检验x是否为类A的对象，返回值为boolean型

使用情境：为了避免在向下转型时出现ClassCastException的异常，我们在向下转型之前，先进行instanceof的判断，一旦返回true，就进行向下转型。如果返回false，不进行向下转型。



#### 基本数据类型、包装类--->String类型：调用String重载的valueOf()

```java
float f = 12.3f;
String str1 = String.valueOf(f);

Double d = new Double(12.4);
String str2 = String.valueOf(d);
```



#### String类型 --->基本数据类型、包装类：调用包装类的parseXxx(String s)

```java
String str = "123";
int num = Integer.parseInt(str);
//错误的写法：
//int num = (int)str;
//Integer intger = (Integer)str;
```



#### 对属性可以赋值的位置

①默认初始化

②显式初始化/⑤在代码块中赋值

③构造器中初始化

④有了对象以后，可以通过"对象.属性"或"对象.方法"的方式，进行赋值

执行的先后顺序：① - ② / ⑤ - ③ - ④



#### final修饰形参

表明此形参是一个常量。当我们调用此方法时，给常量形参赋一个实参。一旦赋值以后，就只能在方法体内使用此形参，但不能进行重新赋值。

```java
public void show(final int num){
    //num = 20;//编译不通过
    System.out.println(num);
}
//不可以在方法内赋值，只有在调用时赋值，如test.show(10);
```

面试题：排错

```java
public class Something{
    public int addOne(final int x){
        return ++x;//  错误的，因为改变了x；
        //return x + 1;//正确的。未改变x；
    }
}
```



#### 异常的处理：throws

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



#### 手动抛出异常：throw

```java
class Student{
    private int id;
    
    public void regist(int id){
        if(id > 0){
            this.id = id;
        }else{
            throw new RuntimeException("您输入的数据非法！");
        }
    }
}
```



#### 自定义异常类

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



String实现了Serializable接口，表示字符串是支持序列化的

序列化，就是将Java对象转化成字节流的形式传出去。反序列化，就是从字节流中恢复Java对象。



#### String内建函数

```java
int length()：返回字符串的长度： return value.length
char charAt(int index)： 返回某索引处的字符return value[index]
boolean isEmpty()：判断是否是空字符串：return value.length == 0
String toLowerCase()：使用默认语言环境，将 String 中的所有字符转换为小写
String toUpperCase()：使用默认语言环境，将 String 中的所有字符转换为大写
String trim()：返回字符串的副本，忽略前导空白和尾部空白
boolean equals(Object obj)：比较字符串的内容是否相同
boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
String concat(String str)：将指定字符串连接到此字符串的结尾。 等价于用“+”
int compareTo(String anotherString)：比较两个字符串的大小
String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
```

```java
boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束
boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始
boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始

boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引
int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引
int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索

注：indexOf和lastIndexOf方法如果未找到都是返回-1
```



#### String与char[]的转换

- String → char[]：调用String的toCharArray()
- char[] → String：调用String的构造器

```java
//String → char[]
String str1 = "abc123";
char[] charArray = str1.toCharArray();

//char[] → String
char [] arr = new char[]{'h','e','l','l','o'};
String str2 = new String(arr);
```



#### StringBuffer/StringBuilder的常用方法

```
StringBuffer append(xxx)：进行字符串拼接
StringBuffer delete(int start,int end)：删除指定位置的内容
StringBuffer replace(int start, int end, String str)：把[start,end)位置替换为str
StringBuffer insert(int offset, xxx)：在指定位置插入xxx
StringBuffer reverse() ：把当前字符序列逆转
public String substring(int start,int end):返回一个从start开始到end索引结束的左闭右开区间的子字符串
public int length()
public char charAt(int n)
public void setCharAt(int n ,char ch)
```



#### Comparable 和 Comparator

一、说明：Java中的对象，正常情况下，只能进行比较：== 或 != 。不能使用 > 或 < 的

但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。使用两个接口中的任何一个：Comparable 或 Comparator

二、Comparable接口与Comparator的使用的对比：

Comparable接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。

Comparator接口属于临时性的比较。

Comparable接口的使用举例： 自然排序

1. 像String、包装类等实现了Comparable接口，重写了compareTo(obj)方法，给出了比较两个对象大小的方式。

2. 像String、包装类重写compareTo()方法以后，进行了从小到大的排列

3. 重写compareTo(obj)的规则：

   如果当前对象this大于形参对象obj，则返回正整数，

   如果当前对象this小于形参对象obj，则返回负整数，

   如果当前对象this等于形参对象obj，则返回零。

4. 对于自定义类来说，如果需要排序，我们可以让自定义类实现Comparable接口，重写compareTo(obj)方法。在compareTo(obj)方法中指明如何排序

```java
public void test2(){
    Goods[] arr = new Goods[4];
	arr[0] = new Goods("AAA",1);
	arr[1] = new Goods("BBB",2);
	arr[2] = new Goods("CCC",3);
	arr[3] = new Goods("DDD",4);

	Arrays.sort(arr);	//此时需要重写compareTo,否则会报错
}

class Goods implements Comparable{
    
    private String name;
    private double price;
    
    //......
        
   public int compareTo(Object o){
       if(o instanceof Goods){	//细节,防止非Good类调用比较方法
           Goods goods = (Goods)o;
           //方式一:
           if(this.price > goods.price){
               return 1;
           }else if(this.price < goods.price){
               return -1;
           }else{
               return 0;
           }
           //方式二(推荐):
           return Double.compare(this.price,goods.price)
       }
       //......
   }     
}
```



Comparator接口的使用：定制排序

1. 背景：
   当元素的类型没有实现java.lang.Comparable接口而又不方便修改代码，或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，那么可以考虑使用 Comparator 的对象来排序.
2. 重写compare(Object o1,Object o2)方法，比较o1和o2的大小：

​		如果方法返回正整数，则表示o1大于o2；

​		如果返回0，表示相等；

​		返回负整数，表示o1小于o2。

```java
public void test3(){
    String[] arr = new String[]{"AA","CC","KK","MM","GG"};
    ComparaTor comparator = new ComparaTor();
    Arrays.sort(arr,comparator);
}

class ComparaTor implements Comparator{
    
    public int compare(Object o1, Object o2){
        if(o1 instanceof String && o2 instanceof String){
        	String s1 = (String)o1;
        	String s2 = (String)o2;
       		return -s1.compareTo(s2);	//负号表示降序
    	}
        //......
    }
}
```

##### Comparable和Comparator的区别

- Comparable接口能保证Comparable接口实现类的对象在任何位置都可以比较大小
- Comparator接口属于临时性的比较



#### 集合与数组之间的转换

集合 --->数组：toArray()

数组 --->集合：Arrays类的静态方法asList()

Arrays.asList()源码

```java
public static <T> List<T> asList(T... a) {  
    return new ArrayList<T>(a);  
}
```

asList接收的是一个泛型变长参数，而我们知道基本类型是不能泛型化的，就是说8种基本类型不能作为泛型参数，要想作为泛型参数就要使用其所对应的包装类。

```java
public class ArraysAsListTest {
	
	public static void main(String[] args) {
		
		int[] a = {1,2,3};
		Integer[] b = {1,2,3};
		
		List listA = Arrays.asList(a);
		List listA1 = Arrays.asList(1,2,3);
		List listB = Arrays.asList(b);
		
		System.out.println(listA.size());//out:1
		System.out.println(listA1.size());//out:3
		System.out.println(listB.size());//out:3
	}
}
```

listA传递的是一个int类型的数组，数组是一个对象，它是可以泛型化的，也就是说例子中是把一个int类型的数组作为了T的类型，所以转换后在List中就只有一个类型为int数组的元素。后边ListA1与ListB也就可以理解了，一个是进行了自动打包，一个是本来就是包装类型。



#### 集合元素的遍历操作，使用迭代器Iterator接口

内部的方法：hasNext() 和 next()

```java
//hasNext():判断是否还有下一个元素
while(iterator.hasNext()){
    //next():①指针下移 ②将下移以后集合位置上的元素返回
    System.out.println(iterator.next());
}
```

集合对象每次调用iterator()方法都得到一个全新的迭代器对象，默认游标都在集合的第一个元素之前。



#### List接口的常用方法

```java
void add(int index, Object ele)   //在index位置插入ele元素
boolean addAll(int index, Collection eles)   //从index位置开始将eles中的所有元素添加进来
Object get(int index)   //获取指定index位置的元素
int indexOf(Object obj)   //返回obj在集合中首次出现的位置。如果不存在，返回-1.
int lastIndexOf(Object obj)   //返回obj在当前集合中末次出现的位置。如果不存在，返回-1.
Object remove(int index)   //移除指定index位置的元素，并返回此元素
boolean remove(Object obj)   //删除第一次出现的指定元素obj
Object set(int index, Object ele)   //设置指定index位置的元素为ele
List subList(int fromIndex, int toIndex)   //返回从fromIndex到toIndex位置的子集合
```



#### Map接口的常用方法

```java
Object put(Object key, Object value)   //将指定key-value添加到(或修改)当前map对象中
void putAll(Map m)   //将m中的所有key-value对存放到当前map中
Object remove(Object key)   //移除指定key的key-value对，并返回value
void clear()   //清空当前map中的所有数据
Object get(Object key)   //获取指定key对应的value
boolean containsKey(Object key)   //是否包含指定的key
boolean containsValue(Object value)   //是否包含指定的value
int size()   //返回map中key-value对的个数
boolean isEmpty()   //判断当前map是否为空
```



