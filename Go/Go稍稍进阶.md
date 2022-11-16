# Go稍稍进阶

### 1、select语句

select是Go中的一个控制结构，用于处理异步IO操作。语法类似于switch语句，不过不同的是switch语句可以选择任何使用相等比较的条件，而select有较多的限制，其中最大的一条限制就是**每个case语句里必须是一个IO（channel）操作**

select语句的运行机制是随机执行一个可运行的case，如果没有就会***阻塞***，直到有case可运行为止。

```go
select { //不停的在这里检测
	case <-chanl : //检测有没有数据可以读
    //如果chanl成功读取到数据，则进行该case处理语句
    case chan2 <- 1 : //检测有没有可以写
    //如果成功向chan2写入数据，则进行该case处理语句

    //假如没有default，那么在以上两个条件都不成立的情况下，就会在此阻塞
    //一般default会不写在里面，select中的default子句总是可运行的，因为会很消耗CPU资源
    default:
    //如果以上都没有符合条件，那么则进行default处理流程
    }
```

demo：

```go
func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
   		case i1 = <-c1:
      	fmt.Printf("received ", i1, " from c1\n")
   		case c2 <- i2:
      	fmt.Printf("sent ", i2, " to c2\n")
   		case i3, ok := (<-c3): // same as: i3, ok := <-c3
       
      	if ok {
         	fmt.Printf("received ", i3, " from c3\n")
      	} else {
         	fmt.Printf("c3 is closed\n")
      	}
   default:
      fmt.Printf("no communication\n")
   }
}
```

#### （1）用于超时判断

```go
var resChan = make(chan int)

func test() {
   select {
   case data := <-resChan:
      doData(data)
   case <-time.After(time.Second * 3):
      fmt.Println("request time out")
   }
}

func doData(data int) {
   // ...
}
```

使用全局resChan来接受response，如果时间超过3秒，resChan中还没有数据返回，则第二条case将执行。

#### （2）判断channel是否阻塞

```go
ch := make (chan int, 5)
//...
data：=0
select {
case ch <- data:
default:
    //做相应操作，比如丢弃data。
}
```

default语句中可以填写阻塞后我们想要执行的对应操作。

### 2、循环语句range

Go中的range其实类似于一个**迭代器**操作。for循环的range格式可以对***slice、map、数组和字符串***进行迭代循环，格式如下：

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

### 3、匿名函数

匿名函数由一个不带函数名的函数声明和函数体构成。匿名函数的优越性在于它**可以直接使用函数内的变量而无需声明**。

```go
func main() {
    getSqrt := func(a float64) float64 {
        return math.Sqrt(a)
    }
    fmt.Println(getSqrt(4))
}
// output:2
```

匿名函数可以**赋值给变量**，从而作为***结构体字段***，或者是***在channnel中传输***。

```go
// --- 函数变量 ---
fn := func() { println("Hello, World!") }
fn()

// --- 函数集合 ---
fns := [](func(x int) int){
   func(x int) int { return x + 1 },
   func(x int) int { return x + 2 },
}
println(fns[0](100)) // 101

// --- 函数作为结构体字段 ---
d := struct {
   fn func() string
}{
   fn: func() string { return "Hello, World!" },
}
println(d.fn())

// --- channel和匿名函数 ---
fc := make(chan func() string, 2)
fc <- func() string { return "Hello, World!" }
println((<-fc)())
/**
Hello, World!
101
Hello, World!
Hello, World!
*/
```

### 4、闭包

闭包是一个函数以及捆绑的周边环境的引用组合。闭包可以让开发者可以**从内部函数访问外部函数的作用域**。

```go
func a() func() int {
   i := 0
   b := func() int {
      i++
      fmt.Println(i)
      return i
   }
   return b
}

func main() {
   c := a()
   c()  // 由于闭包的存在使得函数a()返回后，a中的i始终存在，这样每次执行c()，i都是自加1后的值
   c()
   c()

   a()	// 无输出
    /*
    a()执行完后，b()没有被返回给a()的外界，只是被a()所引用，而此时a()也只会被b()引 用，因此函数a()和
    b()互相引用但又不被外界引用，函数a和b就会被回收。
    所以直接调用a()页面没有信息输出。
    */
}
/*
1
2
3
*/
```

这段代码就创建了一个闭包，因为函数a()外的变量c引用了函数a()内的函数b()。

函数b嵌套在函数a内部，函数a返回函数b，这样在执行完c := a()后，变量c实际上是指向了函数b()，再执行函数c()后就会显示i的值，第一次为1，第二次为2，第三次为3，以此类推。 

从上面可以看出闭包的作用就是在a()执行完并返回后，闭包使得垃圾回收机制不会收回a()所占用的资源，因为a()的内部函数b()的执行需要依赖a()中的变量i。

在给定函数被多次调用的过程中，这些私有变量能够保持其持久性。变量的作用域仅限于包含它们的函数，因此无法从其它程序代码部分进行访问。不过，变量的生存期是可以很长，在一次函数调用期间所创建所生成的值在下次函数调用时仍然存在。因此**闭包可以用来完成信息隐藏，并进而应用于需要状态表达的某些编程范型中**。 

### 5、延迟调用defer

defer用于注册延迟调用，

##### （1）特性

> 1. 调用直到return前才会被执行，所以可以用来做资源清理；
> 2. 若出现多个defer语句，其结构可以类比于栈，总是处理最新注册的defer语句；

##### （2）用途

> - 关闭文件句柄
> - 锁资源释放
> - 数据库连接释放

延迟调用参数在注册时求值或复制，可用指针或闭包 "延迟" 读取:

```go
func test() {
    x, y := 10, 20

    defer func(i int) {
        println("defer:", i, y) // y 闭包引用
    }(x) // x 被复制

    x += 10
    y += 100
    println("x =", x, "y =", y)
}

func main() {
    test()
}
/*
    x = 20 y = 120
    defer: 10 120
*/
```

但是要注意不要滥用defer，否则消耗的时间将会到达三倍以上。