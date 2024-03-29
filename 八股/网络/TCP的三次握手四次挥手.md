## TCP的三次握手四次挥手

#### TCP报头格式

1、***源端口号和目标端口号***

2、***包的序号***：解决包乱序问题

3、***确认号***：确认发出去后对方是否有收到，为了解决丢包问题

4、***一些状态位***：SYN发起连接，ACK回复，RST重连，FIN结束连接

5、***窗口大小***：TCP做流量控制和拥塞控制，通信双方各声明一个窗口（缓存大小），标识自己当前能够的处理能力，别发送的太快，撑死我，也别发的太慢，饿死我。



#### TCP三次握手

- 一开始，client和server都处于 `CLOSED` 状态。先是server主动监听某个端口，处于 `LISTEN` 状态。
- 第一次握手（*不可携带数据*）：建立连接时，客户端发送SYN（SYN=j）到服务器，并进入SYN_SEND状态，等待服务器确认。
- 第二次握手（*不可携带数据*）：服务器收到后确认客户端的SYN（ACK=j+1），同时自己也发送一个SYN包（SYN=k），即**SYN+ACK**包，此时服务器进入SYN_RECV状态
- 第三次握手（*可携带数据*）：客户端收到包向服务器发送ACK（ACK=k+1），此包发送完毕，客户端和服务器进入established建立连接状态，完成三次握手。

> **目的**：保证双方都有发送和接收的能力
>
> **查看TCP连接状态**：netstat -napt命令

**SYN丢失场景**：开启syncookies后如果半连接队列满了也不会丢弃syn包，如果全连接队列满了就会出现丢弃SYN情况；以及如果客户端TIME_WAIT状态过多且开启了tcp_tw_recycle，就会快速回收TIME_WAIT状态的连接。

**握手丢失**

第一次握手丢失：触发超时重传SYN，序列号一样，重传次数为内核参数控制(6)，每次超时时间是上次2倍。第六次超时后等待64秒如果哦server依旧没回ACK，直接断开TCP连接。

第二次握手丢失：两端都超时重传。客户端重传n次SYN后由于 tcp_syn_retries参数为n，于是再等上次超时时间2倍，如果还没收到服务端SYN-ACK那客户端就断开连接；服务端也一样，重传SYN-ACK

第三次握手丢失：服务端重传SYN-ACK，如上

**TCP三次握手可以变成两次吗？**

三次握手的首要原因是为**防止旧的重复连接初始化造成混乱**

**长话**：TCP为了实现可靠数据传输， 双方都必须维护一个序列号去标识发送出去的数据包中， 哪些是已经被对方收到的。 而三次握手就是相互告知序列号起始值， 并确认对方已经收到了序列号起始值。
如果只是两次握手， 最多只有连接发起方的起始序列号能被确认， 另一方选择的序列号则得不到确认。

**短说**：tcp通信需要确保双方都具有数据收发的能力，得到ACK响应则认为对方具有数据收发的能力，因此双方都要发送SYN确保对方具有通信的能力。

且只有两次握手的话，客户端发SYN在网络中阻塞，客户端没接收到ACK就会重发 SYN ，由于没有第三次握手，服务端不清楚客户端是否收到自己回 ACK，所以服务端每收一个 SYN 就只能先主动建立一个连接，就会建立多个冗余的无效链接，造成不必要**资源浪费**

**SYN攻击解决**

> 短时间伪造不同IP的 SYN报文，服务端每收到一个 SYN就进入SYN_RCVD状态，但无法得到未知IP主机的 ACK`应答，久而久之会**占满服务端的半连接SYN队列**（全连接accept），使得服务端不能为正常用户服务

增大TCP半连接队列，开启 syncookies 功能（不使用 SYN 半连接队列成功建立连接）、减少 SYN+ACK 最大重传次数（调整参数，就可以尽快断开连接）

#### TCP 半连接队列和全连接队列

TCP三次握手时Linux内核会维护两个队列（半连接SYN 队列；全连接accept 队列）服务端收到客户端发起的 SYN 请求后，内核会把该连接存到半连接队列并向客户端响应SYN+ACK，接着客户端会返回 ACK，服务端收到第三次握手的 ACK 后，内核会把连接从半连接队列移除，然后创建新的完全连接并添加到 accept 队列，等待进程调用 accept 函数时把连接取出来。

1. 半连接队列满了且没开启 tcp_syncookies，则丢弃；
2. 全连接队列满了且没重传 SYN+ACK 包的连接请求多于 1 个，则丢弃；
3. 没开启 tcp_syncookies且 `max_syn_backlog-当前半连接队列长度<(max_syn_backlog>>2)`，则丢弃



#### TCP四次挥手

- 客户端发一个**FIN**，关闭客户端到服务器的数据传送，客户端进入FIN_WAIT_1状态
- 服务器收到后返一个**ACK**，进入CLOSE_WAIT状态；客户端收到服务端ACK后进入FIN_WAIT_2状态
- 等服务端处理完数据后，服务器关闭与客户端的连接，发**FIN**给客户端，之后服务端进入LAST_ACK 状态
- 客户端回ACK报文，进入**TIME_WAIT**状态（主动关闭连接方拥有，防止历史连接中的数据被后面相同四元组的连接错误接收，状态会持续 2MSL，足以让两个方向上的数据包都被丢弃，使原来连接的数据包在网络中都自然消失，再出现的数据包一定都是新建立连接所产生的；且保证被动关闭连接一方能被正确关闭）
- 服务端收到 ACK 进入CLOSE状态，完成连接关闭
- 客户端经过 2MSL 进入CLOSE状态，完成连接关闭（MSL：报文最大生存时间，2MSL时长60s相当于至少允许报文丢一次，若 ACK 在一个 MSL 内丢失，这样被动方重发的 FIN 会在第 2 个 MSL 内到达，TIME_WAIT 状态的连接可以应对）

**服务器出现大量 TIME_WAIT 状态的原因**

HTTP 没有使用长连接、HTTP 长连接超时、HTTP 长连接的请求数量达到上限

**服务器出现大量 CLOSE_WAIT 状态的原因**

服务端程序没调 close 关闭连接，代码问题

**四次挥手可以压缩成三次吗？**（服务端的ACK和FIN是否能合并）

服务器收到客户端的 FIN 报文时，内核会马上回一个 ACK 应答报文，但服务端可能还有数据要发送或者处理，所以并不能马上发送 FIN 报文，而是将发送 FIN 报文的控制权交给服务端，所以服务端的 ACK 和 FIN 一般都分开发送。

**什么情况会出现三次挥手？**

被动关闭方（服务端）「没数据要发送」且「开启了 TCP 延迟确认机制」，那么第二和第三次挥手就会合并传输，这样就出现了三次挥手。

**TCP半关闭状态**

在TCP连接中，一端发送了FIN分节，表示不再发送数据，但仍然可以接收对端发送的数据，此时这一端就进入了半关闭状态。

**关闭连接函数close和shutdown**

close：socket同时关闭发送方向和读方向，不再有发送和接收数据的能力

shutdown：可指定socket只关闭发送方向而不关读取方向

如果客户端用 **close** 来关闭连接，那么在 TCP 四次挥手过程中，如果收到了服务端发送的数据，由于客户端已不再具有发送和接收数据的能力，所以客户端内核会回 **RST 报文**给服务端，然后内核会释放连接，**这时就不会经历完整的 TCP 四次挥手**。只有两端调close时才会四次完整握手。



#### TCP分割数据

如果HTTP请求消息超过了MSS长度1460bytes（MTU一个包的最大长度1500bytes - IP和TCP头）

**IP层会分片为啥TCP还需要MSS？**

 IP层本身没超时重传机制，它由传输层的 TCP 来负责超时和重传。当某个IP分片丢后，接收方的IP层就无法组装成一个完整的TCP报文，也就无法将报文送到 TCP 层，所以接收方不会响应 ACK 给发送方。发送方迟迟收不到 ACK 就会触发超时重传，就会重发整个TCP报文，这样效率太低。



#### TCP缺点

升级困难；建立连接的延迟（HTTPS需要HTP三次握手+TLS四次握手）；队头阻塞；网络迁移需要重新建立 TCP 连接。基于UDP的QUIC协议把这些都解决了。



#### TCP和UDP区别

UDP面向报文，os不会对消息进行拆分；TCP面向字节流，消息会被os分组成多个TCP报文，不能认为一个用户消息对应一个 TCP 报文

1、连接：TCP 是面向连接的传输层协议，传输数据前先要建立连接。UDP不需要连接，即刻传输数据。

2、服务对象：TCP 一对一两点服务，一条连接只有两个端点。UDP 支持一对一、一对多、多对多的交互通信

3、可靠性：TCP可靠交付数据，数据可不丢失不重复按序到达。UDP不保证可靠。但可基于UDP实现可靠传输协议（如QUIC协议）

4、拥塞控制、流量控制：TCP有拥塞控制和流量控制机制，保证数据传输安全性。UDP没有，即使网络拥堵也不会影响UDP发送速率

5、首部开销：TCP头部较长，有一定开销，头部在没用「选项」字段时是 `20` 字节，如果用该字段会变长。UDP头部8个字节固定不变，开销较小

6、传输方式：TCP流式传输，没边界但保证顺序和可靠。UDP一个包一个包发送，有边界但可能会丢包和乱序

7、分片不同：TCP数据如果大于**MSS**，则**在传输层分片**，目标主机收到后，也同样在传输层组装数据包，如果中途丢了一个分片，只需传输丢失的分片；UDP数据如果大于**MTU**，则**在IP层分片**，目标主机收到后在IP层组装完数据再传给传输层

#### TCP和UDP应用场景

TCP：面向连接，能保证数据可靠性交付（FTP文件传输、HTTP/HTTPS）

UDP：面向无连接，可随时发送数据，处理简单高效（包总量较少的通信DNS、视频音频等多媒体通信、广播通信）



#### 针对TCP的socket编程

1. 服务端和客户端初始化 `socket`，得到文件描述符；
2. 服务端调用 `bind`，将 socket 绑定在指定的 IP 地址和端口;
3. 服务端调用 `listen`，进行监听；（bind而没调listen客户端发起连接请求，服务器回RST；不用listen两个客户端同时对对方发起连接可以建立连接，此时客户端没有半连接队列[listen时建]但会有一个hash表）
4. 服务端调用 `accept`，等待客户端连接；（没accept也可以建立TCP连接，它只是为了从全连接队列中拿出一条连接，本身跟三次握手没关系）
5. 客户端调用 `connect`，向服务端的地址和端口发起连接请求；
6. 服务端 `accept` 返回用于传输的 `socket` 的文件描述符；
7. 客户端调用 `write` 写入数据；服务端调用 `read` 读取数据；
8. 客户端断开连接时，会调用 `close`，那么服务端 `read` 读取数据的时候，就会读取到了 `EOF`，待处理完数据后，服务端调用 `close`，表示连接关闭。

 **listen时参数backlog意义**：之前是半连接队列大小（SYN队列），现在是代表全连接队列大小（Accpet队列）

客户端connect成功返回是在第二次握手，服务端accept成功返回是在三次握手成功后

**客户端调用 close 了，连接断开的流程**：两端close四次挥手，client发SYN，进入FIN_WAIT_1状态。服务端收到 FIN 报文，TCP 协议栈会为 FIN 包插一个文件结束符 EOF到接收缓冲区中，这个 EOF 会被放在已排队等候的其他已接收的数据之后，这就意味着服务端需要处理这种异常情况，因为 EOF 表示在该连接上再无额外数据到达。此时，服务端进入 CLOSE_WAIT 状态；接着，处理完数据后自然会读到 `EOF`，于是也调 `close` 关闭套接字，使得服务端发一个 FIN 之后处于 LAST_ACK 状态；客户端接收到FIN并发 ACK 给服务端，此时客户端进入 TIME_WAIT 状态；服务端收到 ACK 确认包后，就进入了最后的 CLOSE 状态；客户端经过 `2MSL` 时间之后，也进入 CLOSE 状态



#### TCP Keepalive和HTTP Keep-Alive不是一个东西

HTTP 的 Keep-Alive 也叫 **HTTP 长连接**，由「应用程序」实现，可使得用同一个 TCP 连接来发送和接收多个 HTTP 请求/应答，减少HTTP短连接带来的多次 TCP 连接建立和释放的开销

TCP 的 Keepalive 也叫 **TCP 保活机制**，由「内核」实现，当客户端和服务端长达一定时间没进行数据交互时，内核为确保该连接是否还有效就会发送探测报文来检测对方是否在线，然后来决定是否要关闭该连接