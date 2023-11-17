## Context

#### 主要的应用

- 上下文控制（withCancel）

- 多个goroutine间的数据交互（withValue）

- 超时控制：到某个时间点超时，过多久超时（withTimeout）



#### 底层实现

context包构建了**树型**关系的Context。底层通过 ***channel + mutex*** 实现。channel负责在父级节点`cancel()`后的相关子协程之间广播通信，而mutex保证ctx在多个goroutine之间传递时的线程安全。

使用context时，首先要创建一个顶级context也就是`context.Background()`（`context.TODO()`在不确定应该使用哪种上下文时使用）

每次用户请求到来时，向一组具有上下文关系的goroutine中分别传入ctx参数，并分别监听ctx.Done()方法。

`Done()`方法返回一个**只读的channel**，所有相关函数监听此channel。一旦channel关闭，所有负责监听的 *goroutine* 通过Go channel被关闭时的**广播机制**，都能够收到通知。

子goroutine可以通过 select-case 的方式检查自身是否被父级节点`cancel()`，一旦上层环境（父节点）撤销了本 *goroutine* 的执行，应当终止对当前请求信息的处理，释放资源并return。



#### context canceled错误

"context canceled" 表示上下文（context）已被取消导致当前操作无法完成。

Go 中上下文是一个结构体，用于在多个 Goroutine 间传递取消信号。当某个 Goroutine 发送取消信号时，其他 Goroutine 将收到该信号并退出相应的操作。这种机制可以用于优雅地停止程序或取消长时间运行的操作。

**上下文的取消**可能是由于多种原因引起的：**用户请求超时、资源不足**等。
