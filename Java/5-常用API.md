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



#### List接口的常用方法

```java
void add(int index, Object ele)   //在index位置插入ele元素
Object get(int index)   //获取指定index位置的元素
int indexOf(Object obj)   //返回obj在集合中首次出现的位置。如果不存在，返回-1.
int lastIndexOf(Object obj)   //返回obj在当前集合中末次出现的位置。如果不存在，返回-1.
Object remove(int index)   //移除指定index位置的元素，并返回此元素
Object set(int index, Object ele)   //设置指定index位置的元素为ele
List subList(int fromIndex, int toIndex)   //返回从fromIndex到toIndex位置的子集合
```



#### Map接口的常用方法

```java
Object get(Object key)   //获取指定key对应的value
Object put(Object key, Object value)   //将指定key-value添加到(或修改)当前map对象中
Object remove(Object key)   //移除指定key的key-value对，并返回value
boolean containsKey(Object key)   //是否包含指定的key
boolean containsValue(Object value)   //是否包含指定的value
int size()   //返回map中key-value对的个数
boolean isEmpty()   //判断当前map是否为空
void clear()   //清空当前map中的所有数据
```



