## Channel

类型安全、阻塞、先进先出

#### 底层实现

- `buf`：循环链表，有缓冲channel特有结构，用来存缓存数据。
- `sendx`和`recvx`：记录buf循环链表中的~~发送或者接收的~~index
- `lock`：互斥锁
- `recvq`和`sendq`：接收或发送的goroutine抽象出来的结构体(sudog)的队列，是个双向链表



##### 有缓冲channel

可以提高并发性能，适用于生产者和消费者速度不一致的场景。

##### 无缓冲channel

可以实现同步和互斥，适用于生产者和消费者速度一致的场景。向channel发送数据或从channel接收数据都需要另一端同时就绪，否则会阻塞当前goroutine



#### 有缓冲channel和无缓冲channel的区别

有缓冲channel是异步的，无缓冲channel是同步的



#### 并发场景下什么时候用channel什么时用mutex？

Channel 主要用于有**数据流动/传递的通讯场合**；Mutex 则适用于**数据稳定的场景**



#### 用channel实现mutex

可以用缓冲大小为 1 的 channel来实现互斥锁mutex

实现原理：如果缓冲满了，写channel时将会阻塞；如果通道清空，发送时就会解除阻塞



#### 有缓冲channel先写再读

由于channel无缓冲，所以G1暂时被挂在sendq 队列里，然后 G1 调gopark休眠。接着又有goroutine来channel读数据，此时G2发现sendq等待队列里有goroutine存在，于是直接从G1 copy数据过来，并且会对 G1设置goready函数，这样下次调度发生时， G1就可以继续运行，且会从等待队列里移掉

#### 无缓冲channel先写再读

这一次会优先判断缓冲数据区域是否已满，如果未满则将数据保存在缓冲数据区域，即环形队列里。如果已满，则和之前的流程是一样的。当 G2 要读取数据时，会优先从缓冲数据区域去读取，并且在读取完后，会检查 sendq 队列，如果 goroutine 有等待队列，则会将它上面的 data 补充到缓冲数据区域，并且也对其设置 goready 函数。

#### 有/无缓冲channel先读再写

G1暂时被挂在recvq队列然后休眠起来。G2写数据时，发现recvq队列有goroutine存在，于是直接将数据发送给G1，同时设置G1 goready函数，等待下次调度运行



#### 什么时候会panic

| 操作 | 未初始化channel                 | 关闭channel                        |
| ---- | ------------------------------- | ---------------------------------- |
| 关闭 | **panic**: close of nil channel | **panic**: close of closed channel |
| 发送 | 永远阻塞导致死锁                | **panic**                          |
| 接收 | 永远阻塞导致死锁                | 缓冲区空为零值，否则可继续读       |

**关闭一个只接收的channel会panic**