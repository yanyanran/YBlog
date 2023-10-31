# Paxos、Raft、Zab差异分析

> *网上对于这三个分布式共识算法的解释实在是层次不齐，所以自己写了一篇针对自己的三者总结。*
>
> etcd raft：https://arthurchiao.art/blog/raft-paper-zh/

### CAP理论

三个满足两个

C：**一致性**

A：**可用性**

P：**分区容错性**

------

### Leader选举

- 如何选举

  > **节点加入过程是否阻塞整个请求？**看日志设计是否连续
  >
  > - 如果是连续的，则 leader 中只需要保存每个 follower 上一次的同步位置，这样在同步的时候就会自动将之前欠缺的数据补上，不会阻塞整个请求过程。目前 Raft 的日志是依靠 index 来实现连续的。
  >
  > - 如果不是连续的，则在确认 follower 和 leader 当前数据的差异的时候，是需要获取 leader 当前数据的读锁，禁止这个期间对数据的修改。差异确定完成之后，释放读锁，允许 leader 数据被修改，每一个修改记录都要被保存起来，最后一一应用到新加入的 follower 中。
  >
  >   目前 ZooKeeper 的日志 ZXID 不严格连续，允许有空洞。
  >
  >   
  >
  >   **日志的连续性问题**
  >
  >   在需要保证顺序性的前提下，在利用一致性算法实现状态机的时候，实现连续性日志好还是实现非连续性日志好？
  >
  >   - **Raft连续性日志**：leader 在分发给各个 follower 的时候，只需记录每个 follower 目前已经同步的 index 即可 
  >
  >   - **Zab非连续性日志**： leader 需要为每个 follower 单独保存一个队列，用于存放所有的改动，如 ZooKeeper，一旦是队列就引入了一个问题即顺序性问题，即 follower 在和 leader 进行同步的时候，需要阻塞 leader 处理写请求，先将 follower 和 leader 之间的差异数据先放入队列，完成之后，解除阻塞，允许 leader 处理写请求，即允许往该队列中放入新的写请求，从而来保证顺序性
  >
  >   - **Paxos非连续性日志**：因为当前leader再上一任leader的任期内可能错过了一些日志的同步，而这些日志在其他机器上形成多了多数派。由于logID连续递增，被错过的日志就成了连续logID连续递增序列中的“空洞”，需要通过重确认来补全这些“空洞”位置的日志。而每一次重确认其实都需要一轮完整的Paxos过程。
  >
  >     可能有些日志在恢复前确实未形成多数派备份，需要通过重新执行Paxos来把这些日志重新持久化才能回放。这种不管日志是否曾经形成多数派备份，都重新尝试持久化的原则，这被称为“最大commit原则”，这是因为在持有日志的结点宕机时我们没办法区分某个日志是否已经被提交。
  >
  >     当然这也可能造成幽灵复现问题，可以使用epochID，提交日志时如果上一条Log entry epochID大于本条（为了防止两个leader，选主成功时会执行一条StartWorking日志，所以本epoch必定有新日志，以此拒绝幽灵日志），证明本条是幽灵日志。
  >
  >   还有在复制和提交的时候：
  >
  >   - 连续性日志可以批量进行
  >   - 非连续性日志则只能一个一个来复制和提交
  >
  >   允许空洞的日志在处理事务时有得天独厚的优势，而像Raft这样强制日志有序在极端条件下是一个无法忽视的潜在性能和稳定性风险。
  >
  >   
  >
  >   **怎么阻止上一轮次的 leader 假死的问题**
  >
  >   - Raft：每个 follower 开通一个复制数据的 RPC 接口，谁都可以连接并调用该接口，所以 Raft 需要来阻止上一轮次的 leader 的调用。每一轮次都会有对应的轮次号term用来进行区分，一旦旧 leader 对 follower 发送请求，follower发现当前请求 term < 自己的 term，则直接忽略掉该请求。
  >   - ZooKeeper：一旦 server 进入 leader 选举状态则该 follower 会关闭与 leader 之间的连接，所以旧 leader 就无法发送复制数据的请求到新的 follower 了，也就无法造成干扰了。

  #### Raft

  1、Raft算法以**日志**为核心，保证**leader一定拥有最新最全的日志**（每次投票给Term和LogIndex都大于自己的节点）。同时可能会出现**瓜分选票**的情况，解决办法就是从Timeout上下手，让每一个结点的超时时间加上随机数就能让出现瓜分选票的情况大大减少。

  2、日志的提交有个限制：**不能直接commit之前 term 已过半的事件**，即把这部分日志限制成未提交的日志。

  3、节点**在某个 Term 轮次内只能投一次票**。

  4、加入一个已有leader的集群，**通过RPC通知最新leader是谁**。

  5、leader 发给 follower 的数据出现超时等异常：Raft会不断重试，并且接口是幂等的。

  6、超时后触发超时事件，进行新一轮选举，Raft**只有follower可以发起选举**，而leader不能成为候选人（leader-> follower-> candidate）。

  8、如何对待上一Trem的日志：**Raft不提交，就算已经拷贝到了大多数结点上也不会提交**，除非在Term内也提交了一条日志，上一条日志才会被顺带提交。

  #### Zab（FIFO）

  > (ZXID)<-【`epoch` (任期号)| `counter` (proposal自增数)】
  >
  > 选举优先级：【***epoch > counter***】 ***> SID***
  >
  > 
  >
  > **Zab数据同步处理：**携带ZXID（低32位counter代表新提案计数，高32位代表epoch【相当于raft中的term，leader的任期编号】）。leader每次生成一个新事务并生成唯一ZXID，随后将这个事务发送给所由followers，随后follower将事务请求加入history queue中并发送ack给leader。收到多数followers的ack消息时leader将commit请求。当follower收到commit请求时会判断该事务ZXID是否比history queue中的任何事务的ZXID都小：是就commit，不是则等待比它更小的事务commit。

  0、新leader**会提交旧leader未提交的数据**（Raft不会处理）。

  1、**Fast Leader Election**选举机制【解决了LeaderElection选举算法收敛速度慢的问题】，每次选出leader的日志是最新的（ZXID越大数据越新）。Raft选举时每个follower只持有投向自己的选票； Fast Leader Election**每个follower持有所有结点的投票信息**。

  2、每次 leader 选举完成之后，都会进行数据之间的同步纠正，所以每一个轮次，大家都日志内容都是统一的。

  3、节点在**某个 epoch 轮次内可以投多次票**，只要遇到更大的票就更新，然后分发新的投票给所有人。

  4、加入一个已有leader的集群，会向所有节点发送投票通知，此时会收到处于looking、following状态节点的投票，则该 server 放弃自己的投票，判断上述投票是否过半，过半则可以确认该投票的内容就是新的 leader。

  > Zab server四种状态：
  >
  > - **LEADING** 领导者状态。当前角色是Leader，会维护与Follower间的心跳
  > - **LOOKING** 不确定Leader状态。认为当前集群中没有Leader，会发起Leader选举
  > - **FOLLOWING** 跟随者状态。当前角色是Follower并且知道Leader是谁
  > - **OBSERVING** 观察者状态。当前角色是Observer，与Folower唯一不同在于**不参与选举**，也**不参与集群写操作时的投票**

  5、leader 发给 follower 的数据出现超时等异常：follower 会断开与 leader 的连接，重新加入该集群

  6、超时后触发超时事件，进行新一轮选举，Zab是leader和follower都可以发起选举

  7、Zab解决幽灵复现：持久化CurrentEpoch，在请求选举时带上CurrentEpoch。

  ![](https://img-blog.csdnimg.cn/20210217195254336.png)

  A的Log entry 5，6没有被提交，A宕机。假设B晋升但没有写入任何一条Log entry，意味着B最后一条日志ZXID依然小于A的最后一条日志的ZXID，如果A恢复，就会重新成为leader，此时会在Recovery同步数据，从而出现幽灵复现的问题。

  8、如何对待上一Trem的日志：采取激进策略，对于所有过半还是未过半的日志都判定为提交，都将其应用到状态机中

  #### Paxos

  Paxos日志不连续的，leader也不一定拥有最新日志，所以需要进行日志同步。首先需要确认同步结束位置，所以需要向集群内其他server发送请求，查询每个server本地的最大logID，并从多数派的应答中选择最大的logID作为重确认的结束位置，每一次重确认都是一次一轮完整的Paxos过程。当然有时收到的多数派也不存在此位置日志，就填充为空。

  

- 选举的信息

  Fast Leader Election选举请求信息为（ **P.vote**[这张票投给谁] , **P.id**[serverID], **P.state**[节点状态：looking、following、observing、leading], **P.round**[epoch，也就是Raft中的Term] ），**P.vote←（P.lastZXID, P.id ）** 

  而Raft为（term, candidateId候选人, lastLogIndex, lastLogTerm） 

  Paxos为 accept ( n , v , index[区分不同的Paxos实例] , firstUnchoosenIndex[同步结点间已提交的数据] ) 




### 幽灵复现问题的解决

**幽灵复现**指的是上图中，假设A节点宕机，5和6数据没有同步到B和C节点。此时B被选为新leader，然而过了一段时间后A又启动并且成为了新leader，此时就会发现5和6数据又将会出现。

#### Multi-Paxos的解决方案

paxos在每条日志内容保存一个指定Proposer在生成这条日志时当前的ProposalID，因为leader在开始服务之前一定会写一条startWorking日志，所以如果出现ID相对前一条日志变小的情况，则说明是条“幽灵日志”，忽略就好。



#### Raft的解决方案

在上图场景中，raft直接不会再让A复活后做leader（term+logIndex作为选举对比对象，A复活后显然斗不过B和C）。

- ##### Raft的日志恢复

在 Raft 中，每次选举出来的Leader一定包含已经committed的数据（抽屉原理，选举出来的Leader是多数中数据最新的，一定包含已经在多数节点上commit的数据），新的leader将会覆盖其他节点上不一致的数据。

为了将上一个term未committed的log Entry转为committed，Raft 的解决方案如下：

> **要求Leader当选后立即追加一条Noop的特殊内部日志，并立即同步到其它节点，实现前面未Committed日志全部隐式提交。**

这样就保证了两个事情：

1. 通过最大commit原则**保证不会丢数据**，即是保证所有的已经Committed的Log Entry不会丢；
2. **保证不会读到未Committed的数据**。因为只有Noop被大多数节点同意并提交后（这样可以连带往期日志一起同步），服务才会对外正常工作；Noop日志本身也是一个分界线，Noop之前的Log Entry被提交，之后的Log Entry将会被丢弃。



#### Zab的解决方案

和Raft一样，A不可能在复活后成为leader，因为每次选举后epoch+1，作为下一轮次的ZXID高32位。而A作为follower加入后，其上的数据会被新leader上的数据覆盖掉。