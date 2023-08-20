#### String内建函数

```java
int length()
String toLowerCase()
String toUpperCase()
String trim()：返回字符串的副本，忽略前导空白和尾部空白
int compareTo(String anotherString)：比较两个字符串的大小
String substring(int beginIndex)
String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
```



#### StringBuffer/StringBuilder的常用方法

```
StringBuffer append(xxx)
StringBuffer delete(int start,int end)：删除指定位置的内容
StringBuffer reverse() 
public String substring(int start,int end):返回一个从start开始到end索引结束的左闭右开区间的子字符串
public int length()
public char charAt(int n)
```



#### 集合与数组之间的转换

集合 --->数组：toArray()

```java
String[] ans = list.toArray(new String[0]);
// 左右两边都不能用int[]，只能用Integer
Integer[] ans = list.toArray(new Integer[0]);
```

数组 --->集合：Arrays类的静态方法asList()

```
List<String> ans = Arrays.asList(ss);
```



#### List接口的常用方法

```java
Object get(int index)   //获取指定index位置的元素
int indexOf(Object obj)   //返回obj在集合中首次出现的位置。如果不存在，返回-1.
int lastIndexOf(Object obj)   //返回obj在当前集合中末次出现的位置。如果不存在，返回-1.
Object remove(int index)   //移除指定index位置的元素，并返回此元素
Object set(int index, Object ele)   //设置指定index位置的元素为ele
List subList(int fromIndex, int toIndex)   //返回从fromIndex到toIndex位置的子集合
```



#### Map接口的常用方法

```java
Object remove(Object key)   //移除指定key的key-value对，并返回value
boolean containsKey(Object key)   //是否包含指定的key
boolean containsValue(Object value)   //是否包含指定的value
```



