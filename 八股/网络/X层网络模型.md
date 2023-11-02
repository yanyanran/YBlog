TCP/IP四层网络模型：应用HTTP/传输TCP UDP/网络IP/网络接口ARP RARP

> ARP：通过IP找MAC地址
>
> RARP：通过网关server的ARP表/缓存上请求IP地址
>
> IP：将数据包从源设备传递到目标设备，建立网络连接

OSI七层：应用/表示/会话/传输/网络/数据链路/物理



![](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost3@main/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F/%E6%B5%AE%E7%82%B9/%E5%8D%8F%E8%AE%AE%E6%A0%88.png)



> **假设客户端有多个网卡，就会有多个 IP 地址，那 IP 头的源地址应该选择哪个 IP ？**
>
> 通过路由表（route -n命令），让目的IP地址和每一条目的子网掩码与，如果与Destination IP相同，就将该网卡的IP作为源地址IP。否则走默认网关发送给路由器，目的子网掩码就是路由器IP地址。

