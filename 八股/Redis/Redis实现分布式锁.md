#### Redis实现分布式锁

分布式锁：用于分布式环境下并发控制的一种机制，用于控制某个资源在同一时刻只能被一个应用所使用。

**SET命令有个NX参数可实现「key不存在才插入」**，所以可以用它来实现分布式锁：

- key不存在则显示插入成功，可用来表示加锁成功
- key存在则显示插入失败，可用来表示加锁失败

防止死锁：设置一个超时时间。PX 10000 表示设置 lock_key 的过期时间为 10s，避免客户端发生异常而无法释放锁。

#### redis实现分布式锁的缺点

1、**超时时间不好控制**（可基于**续约**方式设置超时时间：先给锁设一个超时时间然后启动一个守护线程，让守护线程在一段时间后重新设置这个锁的超时时间）

2、**Redis主从复制模式中的数据是异步复制的，会导致分布式锁的不可靠性**（使用红锁解决：让客户端和多个独立的Redis节点依次请求申请加锁，如果客户端能够和半数以上的节点成功地完成加锁操作，那么就认为客户端成功地获得分布式锁，否则加锁失败）