## go内存管理机制

Go 程序启动的时候申请一大块内存，划分spans，bitmap，areana区域；arena 区域按照页划分成一个个小块，span管理一个或多个页，mcentral管理多个span供申请使用；mcache作为线程私有资源，来源于mcentral。

golang内存分配器借鉴了TCMalloc的设计思想：一次性或提前分配**多级内存模块**，减少内存分配时锁的使用和与操作系统的沟通；**多尺度内存单元**，减少内存分配产生碎片。将可用的**堆内存**采用二级分配的方式进行管理：

1. 每个线程都会自行维护一个独立的内存池，称为**mcache**，用于分配小对象；
2. 当内存池不足时，会向全局的内存池**mcentral**，申请更多的内存；
3. mcentral管理着一系列的**mspan**，mspan是由一组连续的页组成的内存块，按照不同的大小划分成**object**，object是实际存储对象的单元；
4. mcentral会根据object的大小分为不同的**size class**，以便快速查找和分配；
5. 当mcentral也不足时，会向操作系统申请更多的内存，这部分内存由**mheap**管理，mheap维护着整个虚拟内存空间和物理内存空间的映射关系。

go的内存管理分为：**栈内存管理**和**堆内存管理**。栈上内存由编译器管理，堆上内存由程序管理，在运行期间申请和释放（垃圾回收）。

**栈与堆相比劣势：**

1、栈空间较小（8M或10M），不适合存放空间占用大的对象

2、函数内的局部变量离开函数体会被自动弹出栈，所以离开函数作用域依然存活的指针也不适合栈存放

**栈与堆相比优势：**

1、管理简单（由编译器完成）

2、分配和释放速度快（无垃圾回收过程）

3、栈上内存有很好的局部性（堆上2块数据可能分布在不同的页上）



#### 内存逃逸

- 方法内返回局部变量指针。

- 向 channel 发送指针数据。

- 在闭包中引用包外的值。

- 在 slice 或 map 中存储指针。

- 切片（扩容后）长度太大。

- 在 interface 类型上调用方法。

**本该分配到栈上的变量跑到了堆上。**

栈是高地址到低地址，栈上的变量，函数结束后变量会跟着回收掉，不会有额外性能的开销。

变量从栈逃逸到堆上，如果要回收掉就需要进行gc，那gc一定会带来额外的性能开销。不断优化gc算法，主要目的都是为了减少gc带来的额外性能开销，变量一旦逃逸会导致性能开销变大。





#### 内存泄露

go 中的内存泄漏一般都是goroutine泄漏，就是goroutine没被关闭或没添加超时控制，让 goroutine一直处于阻塞状态而不能被GC

- goroutine在执行时被阻塞而无法退出导致goroutine的内存泄漏，一个goroutine的最低栈大小为2KB，在高并发的场景下对内存的消耗非常大

- 互斥锁未释放或造成死锁会造成内存泄漏

- time.Ticker是每隔指定的时间就会向通道内写数据。作为循环触发器，必须调stop方法才会停止从而被GC掉，否则会一直占用内存空间

- 字符串的截取引发临时性的内存泄漏、切片截取引起子切片内存泄漏

```text
var str0 = "12345678901234567890"
str1 := str0[:10]
var s0 = []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s1 := s0[:3]
```

- 函数数组传参引发内存泄漏

  如果在函数传参时用到了数组传参且这个数组够大（我们假设数组大小为 100 万，64 位机上消耗的内存约为 800w 字节，即 8MB 内存），或者该函数短时间内被调用 N 次，那么可想而知会消耗大量内存，对性能产生极大的影响，如果短时间内分配大量内存，而又来不及 GC，那么就会产生临时性的内存泄漏

#### 内存泄漏排查方式

一般通过 **pprof** ， Go 的性能分析工具。在程序运行过程中可以记录程序的运行信息（CPU 使用情况、内存使用情况、goroutine 运行情况）-- 性能调优、定位bug