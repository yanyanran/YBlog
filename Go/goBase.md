

## Go基础学习  Blog from yanyanran

  * [Go基础学习  Blog from yanyanran](#go------blog-from-yanyanran)
      - [1、四种主要声明方式](#四种主要声明方式)
      - [2、go程序结构](#2-go程序结构)
      - [3、go编译环境](#3-go----)
      - [4、main函数和init函数](#4-main---init--)
      - [5、下划线  ‘_’](#5---------)
      - [6、变量](#6---)
        * [（1）变量声明格式](#-1-------)
        * [（2）批量声明](#-2-----)
        * [（3）类型推导](#-3-----)
        * [（4）局部变量声明](#-4-------)
        * [（5）匿名变量](#-5-----)
      - [7、常量](#7---)
        * [（1）常量计数器：iota](#-1-------iota)
        * [（2）一些iota的奇奇怪怪的使用](#-2---iota--------)
      - [8、基本类型](#8-----)
        * [（1）整型](#-1---)
        * [（2）浮点型](#-2----)
        * [（3）多行字符串](#-3------)
        * [（4）中文类型：rune类型](#-4------rune--)
        * [（5）修改字符串](#-5------)
        * [（6）类型强转](#-6-----)
- [数组Array](#数组array)
    + [（1）数组定义](#-1-----)
      - [***1、一维数组***](#---1--------)
      - [***2、多维数组***](#---2--------)
      - [***3、多维数组的遍历：***](#---3------------)
    + [（2）长度是数组类型的一部分](#-2------------)
    + [（3）数组通过下标访问：](#-3----------)
        * [（4）指针数组 [ n ] * T，数组指针 *[ n ] T](#-4--------n-----t---------n---t)
- [切片Slice](#切片slice)
    + [1、切片的初始化](#1-------)
    + [2、切片的读写操作实际作用目标是底层数组](#2-------------------)
        * [（1）整型int切片：](#-1---int---)
        * [（2）结构体struct切片：](#-2----struct---)
    + [3、对数组使用指针操作](#3----------)
    + [4、make()：创建切片](#4-make-------)
    + [5、append()：切片追加](#5-append-------)
    + [6、超出cap范围限制的情况](#6---cap-------)
    + [7、slice的遍历](#7-slice---)
    + [8、字符串和切片](#8-------)
    + [9、数组或切片转字符串](#9----------)
- [Slice切片的深入理解](#slice切片的深入理解)
    + [1、关于传参：](#1------)
    + [2、切片扩容：](#2------)
    + [3、关于遍历切片获取地址](#3-----------)
- [指针](#指针)
    + [1、变量、指针地址、指针变量和取地址、取值的关系](#1-----------------------)
    + [2、变量分配内存：make和new](#2--------make-new)
      - [（1）new（返回类型指针）](#-1-new--------)
      - [（2）make（返回类型本身）](#-2-make--------)
- [Map](#map)
    + [1、使用map分配内存](#1---map----)
    + [2、判断某个键是否存在](#2----------)
    + [3、map的遍历](#3-map---)
      - [（1）遍历map](#-1---map)
      - [（2）遍历key](#-2---key)
    + [4、删除键值对](#4------)
    + [5、元素为map的切片](#5----map---)
    + [6、值为切片类型的map](#6--------map)
- [结构体](#结构体)
    + [1、自定义类型与类型别名](#1-----------)
      - [（1）自定义类型](#-1------)
      - [（2）类型别名](#-2-----)
    + [2、结构体定义](#2------)
    + [3、结构体实例化](#3-------)
      - [（1）常规结构体](#-1------)
      - [（2）匿名结构体](#-2------)
      - [（3）使用new创建指针类型结构体](#-3---new---------)
      - [（4）使用&对结构体取址 = new了一次](#-4-------------new---)
      - [（5）使用键/值初始化](#-5---------)
    + [4、嵌套结构体](#4------)
        * [（1）非匿名](#-1----)
        * [（2）匿名](#-2---)
    + [5、结构体的继承](#5-------)
    + [一些结构体相关练习](#---------)

------

------

#### 1、四种主要声明方式

var（声明变量）、const（声明常量）、type（声明类型）、func（声明函数）

#### 2、go程序结构

go程序保存在多个.go文件中，文件第一行package XXX声明用来说明该文件属于哪个包，第二行是import声明，再下来就是类型、变量、函数的声明。

#### 3、go编译环境

go文件编译使用命令go build、go install。环境变量GOPATH = 工程根目录，其下手动创建src、pkg、bin目录

> - src：放源码文件
> - pkg：放包文件
> - bin：放生成的可执行文件（输出目录）
>

系统遍历go install XX时会到GOPATH的src目录里寻找XX目录然后编译里面的go文件。

#### 4、main函数和init函数

init函数用于实现对包的初始化（比如初始化包中的变量），它先于main函数执行。

但是需要注意，init()函数不能被调用。但它可以在包中多次定义：

```go
package main
import "fmt"
func init() {
   fmt.Println("init 1")
}
func init() {
   fmt.Println("init 2")
}
func main() {
   fmt.Println("main")
}
// 输出：
// init 1
// init 2
// main
```

main函数和init函数的异同：

- 相同点：两个函数在定义的时候*不能有任何参数和返回值*，且Go程序自动调用
- 不同点：init函数可以应用于任意包中，且可以重复定义多个；main函数只能用于main包中，且只能定义一个。

#### 5、下划线  ‘_’

“_”是一个特殊标识符，用于忽略结果。

- 下划线**存在于import中**：我们知道import用于导入包。当我们只想要让该文件执行包中的init函数时，就不必要把整个包都导入。此时我们选择import _ “包路径”就可以调用该包的init函数了（其他函数不能调用）
- 下划线**存在于代码中**：忽略or占位

#### 6、变量

*go的变量声明后必须使用*

##### （1）变量声明格式

```go
 var 变量名 变量类型
```

例如：

```go
var name string
var age int
var isOk bool
```

##### （2）批量声明

```go
var (
	a string
	b int
	c bool
	d float32
)
```

##### （3）类型推导

有时可以将变量的类型忽略，直接赋值，让go自己去辨别变量类型：

```
var name = "pprof.cn"
var sex = 1
```

##### （4）局部变量声明

有时在函数内部声明**局部变量**，可以用更简单的 **:=** 方式声明并初始化变量：

```go
package main

import (
    "fmt"
)
// 全局变量m
var m = 100

func main() {
    n := 10
    m := 200 // 此处声明局部变量m
    fmt.Println(m, n)
}
```

##### （5）匿名变量

使用多重赋值时，如果想要忽略某个值就可以使用匿名变量。

匿名变量不占用命名空间、不会分配内存，所以匿名变量之间不存在重复声明。

这个在前面介绍过，匿名变量用一下划线“_”表示：

```go
func foo() (int, string) {
    return 10, "Q1mi"
}
func main() {
    x, _ := foo()
    _, y := foo()
    fmt.Println("x=", x)
    fmt.Println("y=", y)
}
```

#### 7、常量

常量的关键字是const，并且常量在定义的时候必须赋值：

```go
const pi = 3.1415
const e = 2.7182
```

并且声明了变量之后（例如上面的pi和e），在整个程序运行期间它们的值都不能再变化了。

和变量声明相同的是，常量也可以批量声明。不过需要注意的一点是，如果在批量声明中同时声明多个常量时省略了值，则表示和上一行值相同：

```go
const (
	n1 = 100
	n2
	n3
）// 这里的n1、n2和n3都为100
```

##### （1）常量计数器：iota

`iota`在`const`关键字出现时将被重置为`0`。`const`中每新增一行，常量声明将使`iota`计数一次(`iota`可理解为`const`语句块中的行索引)

```go
const (
	n1 = iota //0
	n2        //1
	n3        //2
	n4        //3
)
```

##### （2）一些iota的奇奇怪怪的使用

使用 “_” 跳过某些值

```go
const (
	n1 = iota //0
	n2        //1
	_
	n4        //3
)
```

`iota`声明中间插队

```go
const (
	n1 = iota //0
	n2 = 100  //100
	n3 = iota //2
	n4        //3
)
const n5 = iota //0
```

#### 8、基本类型

##### （1）整型

按长度分类：int8、**int16**（对应C的short）、int32、**int64**（对应C的long）

对应的无符号整型：**uint8**（对应C的byte）、uint16、uint32、uint64

**go中的空为 --- *nil***

##### （2）浮点型

float32、float64

##### （3）多行字符串

go定义一个多行字符串时必须使用反引号字符：

```go
    s1 := `第一行
    第二行
    第三行
    `
    fmt.Println(s1)
```

##### （4）中文类型：rune类型

rune类实际上是一个int32类

##### （5）修改字符串

修改字符串首先要把它先转换成[]rune或[]byte，再转化为string。无论哪种转换都会重新分配内存并复制字节数组：

```go
func changeString() {
   s1 := "hello"
   // 强制类型转换
   byteS1 := []byte(s1)
   byteS1[0] = 'H'
   fmt.Println(string(byteS1))

   s2 := "博客"
   runeS2 := []rune(s2)
   runeS2[0] = '狗'
   fmt.Println(string(runeS2))
}
```

##### （6）类型强转

格式：转换类型（转换值）

比如计算直角三角形的斜边长时使用math包的Sqrt()函数，该函数接收的是float64类型的参数，而变量a和b都是int类型的，这个时候就需要将a和b强制类型转换为float64类型。

```go
func sqrtDemo() {
	var a, b = 3, 4
	var c int
	// math.Sqrt()接收的参数是float64类型，需要强制转换
	c = int(math.Sqrt(float64(a*a + b*b))) 
	fmt.Println(c)
}
```

------

------

# 数组Array

### （1）数组定义

**var name [len]type**，比如：var a [5] int。数组长度一旦定义就不能改变。

#### ***1、一维数组***

```go
    #全局数组：
    var arr0 [5]int = [5]int{1, 2, 3}
    var arr1 = [5]int{1, 2, 3, 4, 5}
    var arr2 = [...]int{1, 2, 3, 4, 5, 6}
    var str = [5]string{3: "hello world", 4: "tom"}

    #局部数组：
    a := [3]int{1, 2}           // 未初始化元素值为 0。
    b := [...]int{1, 2, 3, 4}   // 通过初始化值确定数组长度。
    c := [5]int{2: 100, 4: 200} // 使用索引号初始化元素。
    d := [...]struct {
        name string
        age  uint8
    }{
        {"user1", 10}, // 可省略元素类型。
        {"user2", 20}, // 别忘了最后一行的逗号。
    }
```

#### ***2、多维数组***	

```go
    #全局
    var arr0 [5][3]int
    var arr1 [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}

    #局部：
    a := [2][3]int{{1, 2, 3}, {4, 5, 6}}
    b := [...][2]int{{1, 1}, {2, 2}, {3, 3}} // 第 2 纬度不能用 "..."。
```

#### ***3、多维数组的遍历：***

```go
func main() {
    var f [2][3]int = [...][3]int{{1, 2, 3}, {7, 8, 9}}
    for k1, v1 := range f {
        for k2, v2 := range v1 {
            fmt.Printf("(%d,%d)=%d ", k1, k2, v2)
        }
        fmt.Println()
    }
}
```

### （2）长度是数组类型的一部分

所以 var a[5] int 和 var a[10] int 是不同的类型。

**len()**和**cap()**函数可以返回数组长度。

### （3）数组通过下标访问：

```
for i := 0; i < len(a); i++ {
	// ...
}
for index, v := range a {
	// ...
}
```

##### （4）指针数组 [ n ] * T，数组指针 *[ n ] T

（5）访问越界，如果下标在数组合法范围之外则触发访问越界，会panic

------

------

# 切片Slice

切片是一个数组的引用，所以切片是一个引用类型。但切片不同于数组或数组指针，它通过内部指针和相关属性引用数组片段以实现变长。

同数组，len()可以用于求切片长度，而cap()可以求出slice最大扩张容量。

**0 <= len(slice) <= len(array)** 其中array是slice引用的数组

### 1、切片的初始化

**var name []type** （如：var str []string）

```go
#全局切片：
var arr = [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}	// 切片arr
var slice0 []int = arr[start:end]   // 从切片arr的索引位置start到end处所获切片，len=start-end
var slice1 []int = arr[:end]   // 从切片arr的索引位置0到end处所获切片，len=end
var slice2 []int = arr[start:]  // 从切片arr的索引位置start到len(arr)-1处所获切片      
var slice3 []int = arr[:] 	// 从切片arr的索引位置0到len(arr)-1处所获切片
var slice4 = arr[:len(arr)-1]      //去掉切片的最后一个元素
var slice5 []int = arr[start:end:max]	// start到end处所获切片，len=start-end，cap=max-start
```

```go
#局部切片：
arr2 := [...]int{9, 8, 7, 6, 5, 4, 3, 2, 1, 0}
slice5 := arr[start:end]
slice6 := arr[:end]        
slice7 := arr[start:]     
slice8 := arr[:]  
slice9 := arr[:len(arr)-1] //去掉切片的最后一个元素
```

切片初始化含两个“：”的情况：

**a[x:y:z]  --> 切片内容 [x:y] 切片长度: y-x 切片容量:z-x**

```go
some := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
d1 := some[6:8]
fmt.Println(d1, len(d1), cap(d1))	// [6 7] 2 4
d2 := some[:6:8] 
fmt.Println(d2, len(d2), cap(d2))	// [0 1 2 3 4 5] 6 8
```

### 2、切片的读写操作实际作用目标是底层数组

##### （1）整型int切片：

```go
data := [...]int{0, 1, 2, 3, 4, 5} 		// 一个data数组
s := data[2:4]    // 定义一个data数组的切片s
s[0] += 100
s[1] += 100
fmt.Println(s)		/* [102 103] */
fmt.Println(data)	/* [0 1 102 103 4 5] */
```

可以看到s是数组data的切片。当对切片值更改时对应数组的值也将同步改变。

##### （2）结构体struct切片：

```go
d := [5]struct {
   x int
}{}

slice := d[:] // 0到len-1处的切片
d[1].x = 10
slice[2].x = 20

fmt.Println(d)	// [{0} {10} {20} {0} {0}]
fmt.Printf("&d -> %p\n&d[0] -> %p", &d, &d[0])
// &d -> 0xc0000b6000
// &d[0] -> 0xc0000b6000
```

### 3、对数组使用指针操作

```go
func main() {
   A := []int{0, 1, 2, 3, 4}
   p := &A[2]
   *p += 100
   fmt.Println(A)	// [0 1 102 3 4]
}
```

### 4、make()：创建切片

格式为：**var  name []type = make( []type, len )** --> 可以简写为： **name := make( []typr, len )** 如果要设置cap值则直接在len后面加上cap值即可。

### 5、append()：切片追加

append函数用于在切片尾部追加数据，并返回一个新的slice对象

```go
var a = []int{1, 2, 3}
fmt.Printf("slice a : %v", a)
var b = []int{4, 5, 6}
fmt.Printf("slice B: %v", b)

c := append(a, b...)  // !!!!
fmt.Printf("slice c : %v", c)
d := append(c, 7)
fmt.Printf("slice d: %v", d)
e := append(d, 7, 8, 9, 10)
fmt.Printf("slice  e :%v", e)
// slice c : [1 2 3 4 5 6]
// slice d: [1 2 3 4 5 6 7]
// slice e :[1 2 3 4 5 6 7 7 8 9 10]
```

> 注意第五行：不能够写成append(a,b) 不能通过编译。append() 的第二个参数不能直接使用 slice，需使用 … 操作符，将一个切片追加到另一个切片上：append(s1,s2…)。或者直接跟上元素，形如：append(s1,1,2,3)。

### 6、超出cap范围限制的情况

```go
data := [...]int{0, 1, 2, 4, 10: 0}
s := data[2:3]

s = append(s, 100, 200)	// 向后面追加两个数

fmt.Println(s, data)
fmt.Println(&s[0], &data[0])
```

> 输出结果：
>
> [2 100 200]	 [0 1 2 100 200 0 0 0 0 0 0]
> 0xc000020130 		0xc000020120

从输出可以看出append后的s重新分配了底层数组并复制了数据。如果只追加一个值就不会超出s.cap限制也就不会重新分配。（重新分配底层数组通常是2倍容量去分配）

### 7、slice的遍历

```go
data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
Slice := data[:]
for index, value := range Slice {
   fmt.Printf("index: %v, value: %v\n", index, value)
}
```

### 8、字符串和切片

因为字符串本身不可变，所以如果想要改变string字符串，则先需要转为byte类型，再实行修改：

```go
str := "Hello word"
s := []byte(str)
// s := []rune(str)  --> 中文字符
s[6] = 'G'
s = s[:8]
s = append(s, '!')
str = string(s)
fmt.Println(str)	// Hello Go!
```

以下是字符串存在中文的情况：

```go
Str := "你好，世界！Hello word!"
S := []rune(Str)
S[3] = '哈'
S[4] = '嘿'
S[12] = 'g'
S = S[:14]
Str = string(S)
fmt.Println(Str)	// 你好，哈嘿！Hello go
```

### 9、数组或切片转字符串

```go
 strings.Replace(strings.Trim(fmt.Sprint(数组或切片), "[]"), " ", ",", -1)
```

------

------

# Slice切片的深入理解

### 1、关于传参：

Go中的数组是**值类型**，赋值和参数传参操作都会复制整个数组数据。

```go
func main() {
   arrayA := [2]int{100, 200}
   var arrayB [2]int

   arrayB = arrayA

   fmt.Printf("arrayA:%p , %v\n", &arrayA, arrayA)
   fmt.Printf("arratB :%p, %v\n", &arrayB, arrayB)

   testArray(arrayA)
}

func testArray(x [2]int) {
   fmt.Printf("func Array: %p , %v\n", &x, x)
}
 
/* 输出：
arrayA:0xc0000220e0 , [100 200]
arratB :0xc0000220f0, [100 200]
func Array: 0xc000022120 , [100 200]
*/
```

输出结果显示三个内存的地址都不同，所以也就验证着Go中数组赋值和传参都是值复制的。这会造成一个问题就是：假设每次传参都用数组，那么每次数组都要被复制一次，就会消耗大量的内存。这时候传参中，数组指针就派上用场了：

```go
func main() {
    arrayA := [2]int{100, 200}
    testArrayPoint(&arrayA)   // 1.传数组指针
    arrayB := arrayA[:]
    testArrayPoint(&arrayB)   // 2.传切片
    fmt.Printf("arrayA : %p , %v\n", &arrayA, arrayA)
}

func testArrayPoint(x *[]int) {
    fmt.Printf("func Array : %p , %v\n", x, *x)
    (*x)[1] += 100
}
/*
    func Array : 0xc4200b0140 , [100 200]
    func Array : 0xc4200b0180 , [100 300]
    arrayA : 0xc4200b0140 , [100 400]
*/
```

可以看到使用切片（第二行）传数组参数既可以节约内存又可以处理好内存的问题。

### 2、切片扩容：

**切片的扩容策略是：当切片容量 < 1024时，扩容时翻倍增加；一但元素超过1024增长因子就增为1.25，也就是原来的容量的1/4。**

注意扩容时，如果原数组还有容量可以扩容（len != cap）的时候使用append函数，那么就还是会在原数组上进行追加，也就是说*扩容后的数组还是指向原来的数组*。

**所以用字面量创建切片的时候，cap 的值一定要保持清醒，避免共享原数组导致的 bug。**

### 3、关于遍历切片获取地址

还有一个切片值遍历的一个问题：

```go
slice := []int{10, 20, 30, 40}
for index, value := range slice {
   fmt.Printf("value = %d , "+
      "value-addr = %x , "+
      "slice-addr = %x\n", value, &value, &slice[index])
}
/*
value = 10 , value-addr = c0000b4000 , slice-addr = c0000b2000
value = 20 , value-addr = c0000b4000 , slice-addr = c0000b2008
value = 30 , value-addr = c0000b4000 , slice-addr = c0000b2010
value = 40 , value-addr = c0000b4000 , slice-addr = c0000b2018
*/
```

从结果我们可以看出如果用range方式遍历切片，我们拿到的value其实是切片里的值拷贝，所以每次打印value的地址都是一样的。

所以试图用改变value的方法***更改原切片值***是不可行的，我们应该通过***&slice[index]***来获取真实地址。

------

------

# 指针

前面学习可以知道Go中的函数传参都是值拷贝。当我们想要改变某个值的时候可以创造一个指针变量指向该变量地址。Go中的指针只需要记住以下两个符号：

> **`&`	-->	取地址**
>
> **`*`	-->	根据地址取值**

下面代码：

```go
a := 100
b := &a
fmt.Printf("type of B is : %T\n", b)
c := *b
fmt.Printf("type of C is : %T\n", c)
fmt.Printf("value of C is : %v\n", c)】
/*
type of B is : *int
type of C is : int
value of C is : 100
*/
```

可以看出&和*操作符形成了互补关系，&取出地址， *操作符根据地址取出向对应的值。

### 1、变量、指针地址、指针变量和取地址、取值的关系

- 对变量作取地址操作（**&**）--> 获得这个变量的**指针变量**（指针变量的值就是*指针地址*）
- 对变量作取值操作（*****） --> 获得指针变量指向的原变量的值

### 2、变量分配内存：make和new

```go
var a *int 		// 声明后没有分配内存，值无法存储 --> panic
// 应该加上a = new(int)
*a = 100
fmt.Println(*a)

var b map[string]int
// 应该加上b = make(map[string]int, 10)
b["测试"] = 100
fmt.Println(b)
```

上面的程序报错panic。因为Go对于引用类型的变量在使用时**不仅要声明，还需要分配内存**。这时候make和new就派上用场了。

#### （1）new（返回类型指针）

使用new函数得到的是一个类型的指针。

#### （2）make（返回类型本身）

区别于new，make**只用于slice、map以及chan的内存创建**，而且它返回的类型就是这三个类型本身，而不是他们的指针类型。

------

------

# Map

### 1、使用map分配内存

map是一种无序的基于key-value的数据结构。和之前的一样，map也需要初始化才能使用。分配内存的语法为：

```go
 make(map[KeyType]ValueType, [cap])
```

```go
Map := make(map[string]int, 8)
Map["张三"] = 90
Map["小明"] = 100
fmt.Println(Map)	// map[小明:100 张三:90]
fmt.Println(Map["小明"])	// 100
fmt.Printf("type of a:%T\n", Map)	// type of a:map[string]int

// map声明时填充元素如下
userInfo := map[string]string{
   "username": "yanyanran",
   "password": "123456",
}
fmt.Println(userInfo)	// map[password:123456 username:yanyanran]
```

### 2、判断某个键是否存在

判断语句格式如下：

```go
value, ok := map[key]
```

使用ok判断：

```go
Map := make(map[string]int)
Map["张三"] = 90
Map["小明"] = 100
// key存在 --> ok为true,v为对应的值
// 不存在 --> ok为false,v为值类型的零值
v, ok := Map["张三"]

if ok {
   fmt.Println(v)	// 90
} else {
   fmt.Println("查无此人")
}
```

### 3、map的遍历

map的遍历主要使用for range来实现

#### （1）遍历map

```go
for k, v := range Map {
   fmt.Println(k, v)
}
```

#### （2）遍历key

```go
for k := range Map {
   fmt.Println(k)
}
```

此时遍历map时的元素顺序与添加的键值对顺序无关。如果想要按照指定顺序遍历map，那么如下：

```go
 func main() {
    rand.Seed(time.Now().UnixNano()) //初始化随机数种子

    var scoreMap = make(map[string]int, 200)

    for i := 0; i < 100; i++ {
        key := fmt.Sprintf("stu%02d", i) //生成stu开头的字符串
        value := rand.Intn(100)          //生成0~99的随机整数
        scoreMap[key] = value
    }
    //取出map中的所有key存入切片keys
    var keys = make([]string, 0, 200)
    for key := range scoreMap {
        keys = append(keys, key)
    }
    //对切片进行排序
    sort.Strings(keys)
    //按照排序后的key遍历map
    for _, key := range keys {
        fmt.Println(key, scoreMap[key])
    }
}
```

### 4、删除键值对

**delete(map， key)**  --> key就是要删除的键值对的键

### 5、元素为map的切片

```go
func main() {
   var mapSlice = make([]map[string]string, 3)
   for index, value := range mapSlice {
      fmt.Printf("index:%d value:%v\n", index, value)
   }
   fmt.Println("after init")
   // 对切片中的map元素进行初始化
   mapSlice[0] = make(map[string]string, 10)
   mapSlice[0]["name"] = "王五"
   mapSlice[0]["password"] = "123456"
   mapSlice[0]["address"] = "红旗大街"
   for index, value := range mapSlice {
      fmt.Printf("index:%d value:%v\n", index, value)
   }
}
```

### 6、值为切片类型的map

```go
func main() {
   var sliceMap = make(map[string][]string, 3)
   fmt.Println(sliceMap)
   fmt.Println("after init")
   key := "中国"
   value, ok := sliceMap[key]
   if !ok {
      value = make([]string, 0, 2)
   }
   value = append(value, "北京", "上海")
   sliceMap[key] = value
   fmt.Println(sliceMap)
}
```

------

------

# 结构体

### 1、自定义类型与类型别名

#### （1）自定义类型

Go里面没有“类”的概念。但可以使用type关键字自定义类型，比如：

```go
type MyInt int
```

这时就将MyInt定义为了一种新类型，它具有int的特性。

#### （2）类型别名

举个例子：

```go
type TypeAlias = Type
```

其中TypeAlias只是Type的别名，他俩本质上还是同个类型。(类似于C中的typedf)

那么自定义类型和类型别名有什么区别呢？**类型别名只会在代码中存在，编译完成后iy就会消失不会存在。**

### 2、结构体定义

结构体的声明格式：

```go
    type 类型名 struct {
        字段名 字段类型
        字段名 字段类型
        …
    }
```

^

| 比如：

```go
   type person struct {
        name string
        city string
        age  int8
    }
```

### 3、结构体实例化

#### （1）常规结构体

因为结构体本身也是一种类型，所以我们可以像声明内置类型一样使var关键字声明结构体类型：

```go
var 结构体实例 结构体类型 
```

```go
type person struct {
   name string
   city string
   age  int8
}

func main() {
   var p1 person
   p1.name = "yanyanran"
   p1.city = "广东"
   p1.age = 20
   fmt.Printf("p1=%v\n", p1)
   fmt.Printf("p1=%#v\n", p1)
}
/*
p1={yanyanran 广东 20}
p1=main.person{name:"yanyanran", city:"广东", age:20}
*/
```

#### （2）匿名结构体

在定义临时数据时还可以使用**匿名结构体**：

```go
// main函数体内定义
var user struct {
   Name string
   Age  int
}
user.Name = "yanyanran2"
user.Age = 18
fmt.Printf("%#v\n", user)
/*
struct { Name string; Age int }{Name:"yanyanran2", Age:18}
*/
```

#### （3）使用new创建指针类型结构体

```go
var p2 = new(person)
fmt.Printf("%T\n", p2) // *main.person --> 结构体指针
fmt.Printf("p2=%#v\n", p2)
```

在Go中支持对结构体指针直接使用.操作符来访问结构体成员，也就是可以直接对p2结构体：

```go
p2.name = "测试"
p2.age = 88
p2.city = "北京"
```

如上添加值至结构体中。

#### （4）使用&对结构体取址 = new了一次

```go
p3 := &person{}
```

以上语句就相当于（3）中的new操作。

#### （5）使用键/值初始化

使用键初始化结构体没啥好说的。但是使用***值初始化结构体***的时候要注意：第一，**初始化的时候可以不写键**；第二，**必须初始化完所有的结构体**。

### 4、嵌套结构体

##### （1）非匿名

一个结构体中可以嵌套包含另一个结构体指针：

```go
// Address 地址结构体
type Address struct {
   Province string
   City     string
}

// User 用户结构体
type User struct {
   Name    string
   Gender  string
   Address Address
}

func main() {
   user1 := User{
      Name:   "yyy",
      Gender: "女",
      Address: Address{
         Province: "黑龙江",
         City:     "哈尔滨",
      },
   }
   fmt.Printf("user1=%#v\n", user1) 
//user1=main.User{Name:"yyy", Gender:"女", Address:main.Address{Province:"黑龙江", City:"哈尔滨"}}
}
```

##### （2）匿名

```go
// Address 地址结构体
type Address2 struct {
   Province string
   City     string
}

// User 用户结构体
type User2 struct {
   Name     string
   Gender   string
   Address2 //匿名结构体
}

func main() {
   var user2 User2
   user2.Name = "yyy"
   user2.Gender = "女"
   user2.Address2.Province = "黑龙江"  //通过匿名结构体.字段名访问
   user2.City = "哈尔滨"               //直接访问匿名结构体的字段名
   fmt.Printf("user2=%#v\n", user2) 
}
```

### 5、结构体的继承

Go中的结构体也可以实现“面向对象”的继承，可以通过**嵌套匿名结构体实现继承**：

```go
// Animal 动物
type Animal struct {
   name string
}

// Dog 狗
type Dog struct {
   Feet    int8
   *Animal //通过嵌套匿名结构体实现继承
}

func (a *Animal) move() {
   fmt.Printf("%s会动！\n", a.name)
}

func (d *Dog) wang() {
   fmt.Printf("%s会汪汪汪~\n", d.name)
}

func main() {
   d1 := &Dog{
      Feet: 4,
      Animal: &Animal{ //注意嵌套的是结构体指针
         name: "乐乐",
      },
   }
   d1.wang() //乐乐会汪汪汪~
   d1.move() //乐乐会动！
}
```

- ### 一些结构体相关练习

  1》 问：下列输出是什么？

  ```go
  type student struct {
     id   int
     name string
     age  int
  }
  
  func demo(ce []student) {
     //切片是引用传递，是可以改变值的
     ce[1].age = 999
     // ce = append(ce, student{3, "xiaowang", 56})
     // return ce
  }
  func main() {
     var ce []student //定义一个切片类型的结构体
     ce = []student{
        student{1, "xiaoming", 22},
        student{2, "xiaozhang", 33},
     }
     fmt.Println(ce)
     demo(ce)
     fmt.Println(ce)
  }
  ```

原本以为demo函数没有传指针所以输出还应该是原来的数据。但要注意**传参ce是切片类型，切片是引用传递，是可以改变值的**。
