## Process、Thread、Green Thread、Coroutine区别一览



#### 进程Process

正在运行的程序的一个实例

=> 资源分配的基本单位。每个进程都有自己的地址空间，包括代码段、数据段和堆栈段。



#### 线程Thread

进程中的一个实体，系统调度的基本单位

=> 共享所属进程中的全部系统资源（vaddr、文件描述符、信号处理），但每个线程有自己的调用栈、寄存器环境和本地存储



#### 绿色线程Green Thread

Green Thread不是操作系统级别的概念，它是**用户层（语言级别）**实现的线程 

=> 当一个Green Thread使用系统调用被阻塞时，os不会帮助调度其他线程 -> 导致该进程内所有线程被阻塞

=> ***保存完整的上下文***，开销较大（一般由VM实现）

=> 绿色线程会被匹配到一个原生线程池，从原生线程池中获取原生线程来执行绿色线程。有时候是 1 个原生线程执行 m 个绿色线程（这时候和早期没什么区别）；有时候是 n 个原生线程执行 m 个绿色线程——这时候的绿色线程就获得了**并行**能力。



#### 协程Coroutine

一种比线程更小的执行单元，同样是**用户层**实现的线程。

=> 自带CPU上下文 -> 可任意切换到别的coroutine

=> 与线程相比，coroutine切换**不需要os进行保存和恢复CPU上下文**（因为所有coroutine都存在于同个线程中，所以coroutine切换只有单纯的CPU上下文切换 => 开销小）

=> 调度方式为**非抢占式**（上面三个都是抢占式的） -> 只有当前coroutine主动让出CPU，调度器才会从协程池中调用下一个coroutine

=> 大多数coroutine***不保存完整的上下文***

=> 协程不能并行，只能**并发**



#### Goroutine VS Coroutine

coroutine 与 goroutine 都是可中断可恢复的协程，它们之间最大的不同是：**goroutine 可能在多核上发生并行执行，但 coroutine 始终是顺序执行**。

假如在完全IO并发密集的程序中，python的表现反而更好，因为单线程内的协程切换效率更高。

从运行机制上来说，coroutine 的运行机制属于协作式任务处理， 程序需要主动交出控制权，宿主才能获得控制权并将控制权交给其他 coroutine。如果开发者无意间或者故意让应用程序长时间占用 CPU，操作系统也无能为力，表现出来的效果就是计算机很容易失去响应或者死机。goroutine 属于抢占式任务处理，已经和现有的多线程和多进程任务处理非常类似， 虽然无法控制自己获取高优先度支持。但如果发现一个应用程序长时间大量地占用 CPU，那么用户有权终止这个任务。

![](https://github.com/yanyanran/pictures/blob/main/%E5%8D%8F%E7%A8%8B.png?raw=true)



> **并发**针对单核 CPU 而言，它指的是 CPU 交替执行不同任务的能力；
>
> **并行**针对多核 CPU 而言，它指的是多个核心同时执行多个任务的能力。

