# Go基础

## 值类型和引用类型

值类型：int、float、bool、**string**、数组、**struct**等

引用类型： 指针、slice、map、chan、接口、func



# 切片

切片（Slice）是一个拥有相同类型元素的可变长度的序列。它是基于数组类型做的一层封装。它非常灵活，**支持自动扩容。**

切片是一个引用类型，它的内部结构包含**地址、长度和容量。**切片一般用于快速地操作一块数据集合。

### 切片的定义

声明切片类型的基本语法如下：

```go
var name []T   // var a []string
```

其中，

- name:表示变量名
- T:表示切片中的元素类型

与数组的区别：

```go
var name [元素数量]T   // var a [3]int
```

### 切片的长度和容量

可以通过使用内置的`len()`函数求长度，使用内置的`cap()`函数求切片的容量。

### 使用make()函数构造切片

可以基于数组来创建的切片，但如果需要动态的创建一个切片，我们就需要使用内置的`make()`函数，格式如下：

```go
make([]T, size, cap)   // a := make([]int, 2, 10)
```

### 切片的本质

切片`s2 := a[3:6]`，相应示意图如下：![slice_02](https://www.liwenzhou.com/images/Go/slice/slice_02.png)

### 判断切片是否为空

使用`len(s) == 0`来判断，而不应该使用`s == nil`来判断。因为被初始化之后的空切片!=nil

```go
var s1 []int         //len(s1)=0;cap(s1)=0;s1==nil
// 初始化之后：
s2 := []int{}        //len(s2)=0;cap(s2)=0;s2!=nil
s3 := make([]int, 0) //len(s3)=0;cap(s3)=0;s3!=nil
```

### append()方法为切片添加元素

Go语言的内建函数`append()`可以为切片动态添加元素。 可以一次添加一个元素，**可以添加多个元素，也可以添加另一个切片中的元素（切片名称后面加…）。**

```go
func main(){
	var s []int
    // 要用原变量接收append函数的返回值
	s = append(s, 1)        // [1]
	s = append(s, 2, 3, 4)  // [1 2 3 4]
	s2 := []int{5, 6, 7}  
	s = append(s, s2...)    // [1 2 3 4 5 6 7]
}
```

每个切片会指向一个底层数组，这个数组的容量够用就添加新增元素。**当底层数组不能容纳新增的元素时，切片就会自动按照一定的策略进行扩容，此时该切片指向的底层数组就会更换。**扩容操作往往发生在`append()`函数调用时，所以我们通常都需要用原变量接收append函数的返回值。

### 使用copy()函数复制切片

Go语言内建的`copy()`函数可以迅速地将一个切片的数据复制到另外一个切片空间中，`copy()`函数的使用格式如下：

```go
copy(destSlice, srcSlice []T)
// b := a   切片是引用类型，所以a和b都指向了同一块内存地址。修改b的同时a的值也会发生变化。
```

### 从切片中删除元素

Go语言中并没有删除切片元素的专用方法。要从切片a中删除索引为`index`的元素，操作方法是

```go
a = append(a[:index], a[index+1:]...)
// a[:index] 左包含，右不包含，即a[0]...a[index-1]
```



## Map

初始化：

```go
make(map[KeyType]ValueType, [cap])
```

cap表示map的容量，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。

### 判断某个键是否存在

```go
value, ok := map[key]   // 直接取，然后判断ok
```

### 使用delete()函数删除键值对

使用`delete()`内建函数从map中删除一组键值对：

```go
delete(map, key)
```



## 函数

### 多返回值

```go
// 一个无参，多返回值的函数：参数的括号不能省略，多个返回值必须用()包裹起来
func calc() (int, int) {
	sum := x + y
	sub := x - y
	return sum, sub
}
```

### 返回值命名

函数定义时可以给返回值命名，并在函数体中直接使用这些变量，最后通过`return`关键字返回。

```go
func calc(x, y int) (sum, sub int) {
	sum = x + y
	sub = x - y
	return
}
```

闭包=函数+引用环境。

### defer语句

Go语言中的`defer`语句会将其后面跟随的语句进行延迟处理。在`defer`归属的函数即将返回时，将延迟处理的语句按`defer`定义的逆序进行执行。也就是说，**先被`defer`的语句最后被执行，最后被`defer`的语句最先被执行。**

举个例子：

```go
func main() {
	fmt.Println("start")
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
	fmt.Println("end")
}
```

输出：

```go
start
end
3
2
1
```

由于`defer`语句延迟调用的特性，所以`defer`语句能非常方便的处理资源释放问题。比如：资源清理、文件关闭、解锁及记录时间等。

#### defer执行时机

**在Go语言的函数中`return`语句在底层并不是原子操作，它分为给返回值赋值和RET指令两步。而`defer`语句执行的时机就在返回值赋值操作后，RET指令执行前。**具体如下图所示：![defer执行时机](https://www.liwenzhou.com/images/Go/func/defer.png)



## new和make

我们先来看一个例子：

```go
func main() {
	var a *int
	*a = 100
	fmt.Println(*a)

	var b map[string]int
	b["沙河娜扎"] = 100
	fmt.Println(b)
}
```

执行上面的代码会引发panic，因为**在Go语言中，对于引用类型的变量，使用时不仅要声明它，还要为它分配内存空间，否则我们的值就没办法存储。而对于值类型的声明不需要分配内存空间，是因为它们在声明的时候已经默认分配好了内存空间。**Go语言中new和make是内建的两个函数，主要用来分配内存。

### new

new是一个内置的函数，它的函数签名如下：

```go
func new(Type) *Type
```

其中，

- Type表示类型，new函数只接受一个参数，这个参数是一个类型
- *Type表示类型指针，new函数返回一个指向该类型内存地址的指针。

new函数不太常用，使用new函数得到的是一个类型的指针，并且该指针对应的值为该类型的零值。举个例子：

```go
func main() {
	a := new(int)
	b := new(bool)
	fmt.Printf("%T\n", a) // *int
	fmt.Printf("%T\n", b) // *bool
	fmt.Println(*a)       // 0
	fmt.Println(*b)       // false
}	
```

本节开始的示例代码中`var a *int`只是声明了一个指针变量a但是没有初始化，指针作为引用类型需要初始化后才会拥有内存空间，才可以给它赋值。应该按照如下方式使用内置的new函数对a进行初始化之后就可以正常对其赋值了：

```go
func main() {
	var a *int
	a = new(int)
	*a = 10
	fmt.Println(*a)
}
```

### make

make也用于内存分配，区别于new，它只用于slice、map以及chan的内存创建，而且它**返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针。**make函数的函数签名如下：

```go
func make(t Type, size ...IntegerType) Type
```

make函数是无可替代的，我们在使用slice、map以及channel的时候，都需要使用make进行初始化，然后才可以对它们进行操作。

本节开始的示例中`var b map[string]int`只是声明变量b是一个map类型的变量，需要像下面的示例代码一样使用make函数进行初始化操作之后，才能对其进行键值对赋值：

```go
func main() {
	var b map[string]int
	b = make(map[string]int, 10)
	b["沙河娜扎"] = 100
	fmt.Println(b)
}
```

### new与make的区别

1. **new用于类型的内存分配，并且内存对应的值为类型零值，返回的是指向类型的指针；**
2. **make只用于slice、map以及channel的初始化（它们也只能被make初始化），返回的还是这三个引用类型本身。**

注意：不是所有的引用类型都要用make初始化，指针类型用new。



## 构造函数

Go语言的结构体没有构造函数，我们可以自己实现。 例如，下方的代码就实现了一个`person`的构造函数。 **因为`struct`是值类型，如果结构体比较复杂的话，值拷贝性能开销会比较大，所以该构造函数返回的是结构体指针类型。**

```go
func newPerson(name, city string, age int8) *person {
	return &person{   // &person
		name: name,
		city: city,
		age:  age,
	}
}
```



## 方法和接收者

Go语言中的`方法（Method）`是一种**作用于特定类型变量的函数。这种特定类型变量叫做接收者。**接收者的概念就类似于其他语言中的`this`或者 `self`。

方法的定义格式如下：

```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    函数体
}
```

其中，

- 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
- 方法名、参数列表、返回参数：具体格式与函数定义相同。

**方法与函数的区别是，函数不属于任何类型，方法属于特定的类型。**

### 指针类型的接收者

**指针类型的接收者由一个结构体的指针组成，由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的。** 例如我们为`Person`添加一个`SetAge`方法，来修改实例变量的年龄。

```go
// SetAge 设置p的年龄
// 使用指针接收者
func (p *Person) SetAge(newAge int8) {
	p.age = newAge
}
```

### 值类型的接收者

**当方法作用于值类型接收者时，Go语言会在代码运行时将接收者的值复制一份。在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对副本，无法修改接收者变量本身。**

```go
func (p Person) SetAge2(newAge int8) {
	p.age = newAge
}

func main() {
	p1 := NewPerson("小王子", 25)
	p1.SetAge2(30)
	fmt.Println(p1.age) // 25
}
```



## 结构体的“继承”

**通过嵌套匿名结构体，可以实现面向对象语言中的的继承，注意嵌套的是结构体指针。**

```go
//Animal 动物
type Animal struct {
	name string
}

func (a *Animal) move() {
	fmt.Printf("%s会动\n", a.name)
}

//Dog 狗
type Dog struct {
	Age    int8
	*Animal //通过嵌套匿名结构体实现继承
}

func (d *Dog) wang() {
	fmt.Printf("%s会汪汪汪\n", d.name)
}

func main() {
	d1 := &Dog{
		Age: 3,
		Animal: &Animal{ //注意嵌套的是结构体指针
			name: "乐乐",
		},
	}
	d1.wang() //乐乐会汪汪汪
	d1.move() //乐乐会动
}
```



## 接口

在Go语言中接口（interface）**是一种类型，**一种抽象的类型。字符串、切片、结构体等类型更注重“我是谁”，接口类型更注重“我能做什么”。接口类型就像是一种约定，**概括了一种类型应该具备哪些方法。**

### 接口的定义

每个接口类型由任意个方法签名组成，接口的定义格式如下：

```go
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}
```

其中：

- 接口类型名：**Go语言的接口在命名时，一般会在单词后面添加`er`，**如有写操作的接口叫`Writer`，有关闭操作的接口叫`closer`等。
- 方法名：**当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包之外的代码访问。**
- 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

### 实现接口的条件

接口规定了一个**需要实现的方法列表，一个类型只要实现了接口中规定的所有方法，那么我们就称它实现了这个接口。**

定义`Singer`接口类型，它包含一个`Sing`方法。

```go
// Singer 接口
type Singer interface {
	Sing()
}
```

有一个`Bird`结构体类型如下。

```go
type Bird struct {}
```

只需要给`Bird`结构体添加一个`Sing`方法，`Bird`就实现了`Singer`接口。

```go
// Sing Bird类型的Sing方法
func (b Bird) Sing() {
	fmt.Println("汪汪汪")
}
```

### 为什么要使用接口？

不使用接口：

```go
package main

import "fmt"

type Cat struct{}

func (c Cat) Say() {
	fmt.Println("喵喵喵~")
}

type Dog struct{}

func (d Dog) Say() {
	fmt.Println("汪汪汪~")
}

func MakeCatHungry(c Cat) {
	c.Say()
}

func MakeDogHungry(d Dog) {
	d.Say()
}
```

使用接口：

```go
type Sayer interface {
    Say()
}

// 定义一个通用的MakeHungry函数，接收Sayer类型的参数。
func MakeHungry(s Sayer) {
	s.Say()
}

// 把所有会叫的动物当成Sayer类型来处理
var c cat
MakeHungry(c)
var d dog
MakeHungry(d)
```

类似于Java中的多态。

`Dog`和`Cat`类型均实现了`Sayer`接口，此时一个`Sayer`类型的变量就能够接收`Cat`和`Dog`类型的变量。

```go
var x Sayer // 声明一个Sayer类型的变量x
a := Cat{}  // 声明一个Cat类型变量a
b := Dog{}  // 声明一个Dog类型变量b
x = a       // 可以把Cat类型变量直接赋值给x
x.Say()     // 喵喵喵
x = b       // 可以把Dog类型变量直接赋值给x
x.Say()     // 汪汪汪
```

### 空接口的定义

**空接口是指没有定义任何方法的接口类型。因此任何类型都可以视为实现了空接口。也正是因为空接口类型的这个特性，空接口类型的变量可以存储任意类型的值。**

通常我们在使用空接口类型时不必使用`type`关键字声明，可以像下面的代码一样直接使用`interface{}`。

```go
var x interface{}  // 声明一个空接口类型变量x
```

### 接口值

由于接口类型的值可以是任意一个实现了该接口的类型值，所以接口值除了需要记录具体**值**之外，还需要记录这个值属于的**类型**。也就是说**接口值由“类型”和“值”组成，**鉴于这两部分会根据存入值的不同而发生变化，我们称之为接口的`动态类型`和`动态值`。

![接口值示例](https://www.liwenzhou.com/images/Go/interface/interface01.png)

#### 类型断言

想要从接口值中获取到对应的实际值需要使用类型断言，其语法格式如下。

```go
x.(T)
```

其中：

- x：表示接口类型的变量
- T：表示断言`x`可能是的类型。

该语法返回两个参数，第一个参数是`x`转化为`T`类型后的变量，第二个值是一个布尔值，若为`true`则表示断言成功，为`false`则表示断言失败。

举个例子：

```go
var n Mover = &Dog{Name: "旺财"}
v, ok := n.(*Dog)
if ok {
	fmt.Println("类型断言成功")
	v.Name = "富贵" // 变量v是*Dog类型
} else {
	fmt.Println("类型断言失败")
}
```



## 包（package）

### 定义包

我们可以根据自己的需要创建自定义包。一个包可以简单理解为一个存放`.go`文件的文件夹。该文件夹下面的所有`.go`文件都要在非注释的第一行添加如下声明，声明该文件归属的包。

```go
package packagename
```

其中：

- package：声明包的关键字
- packagename：包名，**可以不与文件夹的名称一致，**不能包含 `-` 符号，最好与其实现的功能相对应。

**一个文件夹下面直接包含的文件只能归属一个包，**同一个包的文件不能在多个文件夹下。**包名为`main`的包是应用程序的入口包，这种包编译后会得到一个可执行文件，而编译不包含`main`包的源代码则不会得到可执行文件。**

### 标识符可见性

**想要在包的外部使用包内部的标识符就需要添加包名前缀，**例如`fmt.Println("Hello world!"`·，就是指调用`fmt`包中的`Println`函数。

如果想让一个包中的标识符（变量、常量、类型、函数等）能被外部的包使用，那么标识符必须是对外可见的（public）。**在Go语言中是通过标识符的首字母大/小写来控制标识符的对外可见（public）/不可见（private）的。**

例如我们定义一个名为`demo`的包，在其中定义了若干标识符。在另外一个包中并不是所有的标识符都能通过`demo.`前缀访问到，因为只有那些首字母是大写的标识符才是对外可见的。

```go
package demo

import "fmt"

// num 定义一个全局整型变量
// 首字母小写，对外不可见(只能在当前包内使用)
var num = 100

// Mode 定义一个常量
// 首字母大写，对外可见(可在其它包中使用)
const Mode = 1

// person 定义一个代表人的结构体
// 首字母小写，对外不可见(只能在当前包内使用)
type person struct {
	name string
	Age  int
}

// Add 返回两个整数和的函数
// 首字母大写，对外可见(可在其它包中使用)
func Add(x, y int) int {
	return x + y
}

// sayHi 打招呼的函数
// 首字母小写，对外不可见(只能在当前包内使用)
func sayHi() {
	var myName = "xxx" // 函数局部变量，只能在当前函数内使用
	fmt.Println(myName)
}
```

