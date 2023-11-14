## Select

#### 原理

1. select语句类似于switch语句，但是它的每个case表示一个通信操作，可以是发送或接收操作。
2. select会等待其中一个case中的通信操作可以进行时，执行该case对应的代码块。如果有多个case同时可以进行，它们会通过一定的策略进行选择，如随机选择一个case。
3. 如果没有任何一个case的通信操作可以进行，且存在default语句，那么会执行default语句对应的代码块。如果没有default语句，select语句将被阻塞，直到至少有一个case可以进行



#### 底层实现

IO多路复用的思想。依赖于Go运行时系统中的调度器和通信机制，编译时编译器会将select语句转为一个状态机，该状态机由调度器来管理和执行。调度器会监视每个case中的通信操作，并选其中一个可以进行的操作进行执行。调度器使用类似于事件循环的机制来检查和处理通信操作的可执行性，以确保只有一个case被执行。select语句的执行过程是非阻塞的，即使某个case被执行，其他case的通信操作也不会受到影响。



**每个selcet语句都会抽象成一个结构体**，每次会打乱传入的case结构体顺序，锁住其中的所有的channel，然后遍历所有的channel，查看其是否可读或者可写：

- 如果其中的channel可读或者可写，则解锁所有channel，并返回对应的channel数据；
- 假如没有channel可读或者可写，但是有default语句，则返回default语句对应的scase并解锁所有的channel；
- 假如既没有channel可读或者可写，也没有default语句，则将当前运行的groutine阻塞，并加入到当前所有channel的等待队列中去，然后解锁所有channel，等待被唤醒。
- 此时如果有个channel可读或者可写ready了，则唤醒，并再次加锁所有channel，遍历所有channel找到那个对应的channel和G，唤醒G，并将没有成功的G从所有channel的等待队列中移除。