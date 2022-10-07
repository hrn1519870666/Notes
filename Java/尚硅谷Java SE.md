#### java.util.Arrays

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
}
```



#### 值传递机制

如果参数是基本数据类型，此时实参赋给形参的是实参真实存储的数据值。如果参数是引用数据类型，此时实参赋给形参的是实参存储数据的**地址值。**



#### JaveBean

JavaBean是指符合如下标准的Java类：

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



#### final修饰形参

表明此形参是一个常量。当我们调用此方法时，给常量形参赋一个实参。一旦赋值以后，就只能在方法体内使用此形参，但不能进行重新赋值。

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



String实现了Serializable接口，表示字符串是支持序列化的。

序列化，就是将Java对象转化成字节流的形式传出去。反序列化，就是从字节流中恢复Java对象。



#### String内建函数

```java
int length()：返回字符串的长度
char charAt(int index)： 返回某索引处的字符
boolean isEmpty()：判断是否是空字符串
String toLowerCase()：使用默认语言环境，将 String 中的所有字符转换为小写
String toUpperCase()：使用默认语言环境，将 String 中的所有字符转换为大写
String trim()：返回字符串的副本，忽略前导空白和尾部空白
int compareTo(String anotherString)：比较两个字符串的大小
String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
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
StringBuffer reverse() ：把当前字符序列逆转
public String substring(int start,int end):返回一个从start开始到end索引结束的左闭右开区间的子字符串
public int length()
public char charAt(int n)
```



#### 集合与数组之间的转换

集合 --->数组：toArray()

数组 --->集合：Arrays类的静态方法asList()



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
Object remove(Object key)   //移除指定key的key-value对，并返回value
void clear()   //清空当前map中的所有数据
Object get(Object key)   //获取指定key对应的value
boolean containsKey(Object key)   //是否包含指定的key
boolean containsValue(Object value)   //是否包含指定的value
int size()   //返回map中key-value对的个数
boolean isEmpty()   //判断当前map是否为空
```



