## TCP的三次握手四次挥手

#### TCP报头格式

1、***源端口号和目标端口号***

2、***包的序号***：解决包乱序问题

3、***确认号***：确认发出去后对方是否有收到，为了解决丢包问题

4、***一些状态位***：SYN发起连接，ACK回复，RST重连，FIN结束连接

5、***窗口大小***：TCP做流量控制和拥塞控制，通信双方各声明一个窗口（缓存大小），标识自己当前能够的处理能力，别发送的太快，撑死我，也别发的太慢，饿死我。



#### TCP三次握手

- 一开始，client和server都处于 `CLOSED` 状态。先是server主动监听某个端口，处于 `LISTEN` 状态。
- 然后client主动发起连接 `SYN`，之后处于 `SYN-SENT` 状态
- server收到返回 `SYN`+`ACK`，之后处于 `SYN-RCVD` 状态
- client收到后，发送确认 `ACK`，之后处于 `ESTABLISHED` 状态
- server收到后，处于 `ESTABLISHED` 状态

> **目的**：保证双方都有发送和接收的能力
>
> **查看TCP连接状态**：netstat -napt命令



#### TCP分割数据

如果HTTP请求消息超过了MSS长度1460bytes（MTU一个包的最大长度1500bytes - IP和TCP头）