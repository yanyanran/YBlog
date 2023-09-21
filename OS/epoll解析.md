# epoll解析 

#### 1、网卡接收数据

网卡会把接收到的数据写入内存



#### 2、接收数据

从CPU角度看数据的接收：**中断**

当网卡向CPU发出一个中断信号，操作系统就能知道此时有新数据到来，然后再通过网卡中断程序去处理数据



#### 3、进程阻塞为啥不占用CPU资源？

从os进程角度看数据接收，**阻塞**是进程调度的重要一part。阻塞原理如下：

- 假设os中有三个进程ABC，一开始这三个进程都处于运行状态，位于os内核空间的工作队列中；

- ```c
  int s = socket(AF_INET, SOCK_STREAM, 0);   //创建socket
  bind(s, ...) //绑定
  listen(s, ...)  //监听
  int c = accept(s, ...) //接受客户端连接
  recv(c, ...);  //接收客户端数据
  ```

  当进程A执行到创建socket语句时，os会创建一个由文件系统管理的socket对象，其中包括了发送和接收缓冲区、等待队列等，这里的等待队列指向所有需要等待该socket的进程。

- 当执行到recv，os将会将进程A从工作队列移动到该socket的等待队列中。此时我们可以认为进程A被挂起，而os内核的工作队列只剩下了进程B和C。进程A被阻塞，不会往下执行代码，自然也不会占用cpu资源。
- 至于唤醒进程也就是锁的拿和放，当socket接收到数据后会将该socket等待队列的进程重新put回工作队列中



#### 4、内核接收网络数据的全过程

进程在**recv**阻塞期间，计算机收到了对端传送的数据（步骤①）。

数据经由网卡传送到内存（步骤②），然后网卡通过中断信号通知cpu有数据到达，cpu执行中断程序（步骤③）。

此处的中断程序主要有两项功能，先将网络数据写入到对应socket的接收缓冲区里面（步骤④），再唤醒进程A（步骤⑤），重新将进程A放入工作队列中。

![内核接收数据全过程](https://github.com/yanyanran/pictures/blob/main/%E5%86%85%E6%A0%B8%E6%8E%A5%E6%94%B6%E6%95%B0%E6%8D%AE.png?raw=true)

> - Q1：os如何知道网络数据对应哪个soket？
>
>   一个socket对应一个端口号，而网络数据包中通常包含着ip和端口的信息，而内核可以解析并通过端口号找到对应的socket（为了提高处理速度，os会维护一个端口号和socket的索引结构以便快速读取）
>
> - Q2：如何同时监视多个socket的数据？
>
>   这就是多路复用的重中之重



#### 5、同时监控多个socket的简单方法：select

> 服务端 -> 管理多个客户端连接
>
> recv -> 只能监视单个socket

显然recv不能满足这样的需求。select假设：可以预先传入一个socket fds list，如果list中的socket都没有数据，就挂起进程，直到有一个socket收到数据，然后唤醒进程。

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

当用户进程调select，select会将需监控的readfds集合拷贝到内核空间，然后遍历自己监控的socket sk，挨个调用sk的poll逻辑（此poll非Poll）以便检查该sk是否有可读事件。如果没有任何一个sk可读，那么select会调用schedule_timeout进入schedule循环，使得进程睡眠。如果在timeout时间内某个sk上有数据可读了，或者等待timeout了，则调select的进程会被唤醒，接下来select就是遍历监控的sk集合，挨个收集可读事件并返回给用户了，相应伪码如下：

```C
for (sk in readfds)
{
    sk_event.evt = sk.poll();
    sk_event.sk = sk;
    ret_event_for_process;
}
```

总之就是，先准备一个集合存放着所有需要监视的socket。然后调用select，如果数组中的所有socket都没有数据，select会阻塞，直到有一个socket接收到数据，select返回，唤醒进程。

用户可以遍历fds集合，通过FD_ISSET判断具体哪个socket收到数据，然后做出处理：

```c
// 用户代码
int s = socket(AF_INET, SOCK_STREAM, 0);  
bind(s, ...)
listen(s, ...)

int fds[] =  存放需要监听的socket

while(1){
    int n = select(..., fds, ...)
    for(int i=0; i < fds.count; i++){
        if(FD_ISSET(fds[i], ...)){
            //fds[i]的数据处理
        }
    }
}
```

- select之后，os将进程A份分别加入这三个socket的等待队列中：

![](https://pic4.zhimg.com/80/v2-0cccb4976f8f2c2f8107f2b3a5bc46b3_1440w.webp)

- 任何一个socket收到数据，中断程序都会唤醒进程：将进程从所有等待队列中移除，然后加入到工作队列中：

![](https://pic1.zhimg.com/80/v2-85dba5430f3c439e4647ea4d97ba54fc_1440w.webp)

![](https://pic4.zhimg.com/80/v2-a86b203b8d955466fff34211d965d9eb_1440w.webp)

至此，当进程A被唤醒之后它知道至少有一个socket接收到了数据，然后程序需要遍历一次socket fds list就可以得到就绪的socket了

缺点：

- 每调一次select都需要将进程加到所有监视socket的等待队列，每次唤醒都要从每个队列中移除，也就是两次遍历
- 每调一次select都需要将fds list从用户空间拷贝到内核空间，有一定开销
- 为了减少数据拷贝带来的性能损坏，内核对被监控的fds集合大小做了限制，并且这个是通过宏控制的（类似于bitmap的结构，开局要写死），大小不可改变(限制为1024)
- 进程被唤醒后，程序并不知道哪些socket收到数据，还需要遍历一次



#### 6、只是改进了一点点的Poll

相比于select，poll只解决了【fds集合大小限制在1024的问题】。poll改变了fds集合的描述方式，使用了pollfd结构而不是select的fd_set结构，使得poll支持的fds集合限制远大于select的1024。

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```



#### 7、epoll大牛闪亮登场

epoll_create创建一个epoll对象epfd，再通过epoll_ctl将需要监视的socket添加到epfd中，最后调用epoll_wait等待数据：

```c
int s = socket(AF_INET, SOCK_STREAM, 0);   
bind(s, ...)
listen(s, ...)

int epfd = epoll_create(...);
epoll_ctl(epfd, ...); //将所有需要监听的socket添加到epfd中

while(1){
    int n = epoll_wait(...)
    for(接收到数据的socket){
        //处理
    }
}
```

##### （1）fds集合拷贝问题的解决

调select/poll都在重复准备整个需要监控的fds集合，但是fds集合的变化频率很低，所以没必要每次都重新准备整个fds集合。

- epoll引入**epoll_ctl**系统调用，EPOLL_CTL_ADD和EPOLL_CTL_MOD和EPOLL_CTL_DEL三个操作把原本大的fds模块分成了小模块处理。

- 同时epoll通过用户空间和内核的mmap，也就是内存映射到同一块。使得一块物理内存可以同时对内核和用户共享，从而减少内核态和用户态的数据交换。

此时epoll通过epoll_ctl对fds集合进行操作，涉及到fd的快速查找。现在的epoll底层使用**红黑树**进行处理。

##### （2）按需遍历就绪fds集合

socket事件发生后需要遍历其线程睡眠队列，为了兼顾前面的‘分成小部分’，只遍历就绪的fd，内核维护一个**【双向链表】就绪列表rdlist**去引用收到数据的socket（位于epoll_create的对象eventpoll中）：

![](https://pic4.zhimg.com/80/v2-5c552b74772d8dbc7287864999e32c4f_1440w.webp)

假设现在sock2和sock3收到数据，中断程序就会让就绪列表去引用这两个socket：

![](https://pic1.zhimg.com/80/v2-18b89b221d5db3b5456ab6a0f6dc5784_1440w.webp)

此时eventpoll对象相当于是socket和进程的中介，socket的数据接收并不直接影响进程，而是通过改变eventpoll的就绪列表来改变进程状态。

> **假设计算机中正在运行进程A和进程B，在某时刻进程A运行到了epoll_wait语句。内核会将进程A放入eventpoll的等待队列中，阻塞进程。**
>
> - **当socket接收到数据，回调函数epoll_callback被执行，将当前的socket插入到就绪列表中**
> - **唤醒epoll单独睡眠队列的wait_entry，wait_entry_proc被唤醒执行回调函数epoll_callback_proc** 
>
> **另一方面唤醒eventpoll等待队列中的进程，进程A再次进入运行状态。也因为rdlist的存在，进程A可以知道哪些socket发生了变化而无需遍历。**





![](https://pic2.zhimg.com/80/v2-14e0536d872474b0851b62572b732e39_1440w.webp)