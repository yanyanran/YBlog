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

显然recv不能满足这样的需求。seles假如可以预先传入一个socket list，如果list中的socket都没有数据，就挂起进程，直到有一个socket收到数据，然后唤醒进程：

```C
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

先准备一个数组存放着所有需要监视的socket。然后调用select，如果数组中的所有socket都没有数据，select会阻塞，直到有一个socket接收到数据，select返回，唤醒进程。

用户可以遍历数组，通过FD_ISSET判断具体哪个socket收到数据，然后做出处理。

- select之后，os将进程A份分别加入这三个socket的等待队列中：

![](https://pic4.zhimg.com/80/v2-0cccb4976f8f2c2f8107f2b3a5bc46b3_1440w.webp)

- 任何一个socket收到数据，中断程序都会唤醒进程：将进程从所有等待队列中移除，然后加入到工作队列中：

![](https://pic1.zhimg.com/80/v2-85dba5430f3c439e4647ea4d97ba54fc_1440w.webp)

![](https://pic4.zhimg.com/80/v2-a86b203b8d955466fff34211d965d9eb_1440w.webp)

至此，当进程A被唤醒之后它知道至少有一个socket接收到了数据，然后程序需要遍历一次socket list就可以得到就绪的socket了

缺点：

- 每调一次select都需要将进程加到所有监视socket的等待队列，每次唤醒都要从每个队列中移除，也就是两次遍历
- 每调用一次select都需要将数组list传给内核，有一定开销
- 因为遍历操作开销大，出于效率的考量，会规定select的最大监视数量1024
- 进程被唤醒后，程序并不知道哪些socket收到数据，还需要遍历一次