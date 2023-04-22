关于结束一个 goroutine 的方式，比较优雅的方式是 chan+select ：

```go
func main() {
	stop := make(chan bool)

	go func() {
		for {
			select {
			case <-stop:    // 收到了停滞信号
				fmt.Println("监控退出，停止了...")
				return
			default:
				fmt.Println("goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}()

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	stop<- true

	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}
```

不过这种方式也有局限性，如果有很多 goroutine 都需要控制结束，如果这些 goroutine 又衍生了其他更多的 goroutine ，如果是一层层的无穷尽的 goroutine.....

比如一个网络请求 Request，每个 Request 都需要开启一个 goroutine 做一些事情，这些 goroutine 又可能会开启其他的 goroutine。所以我们需要一种可以跟踪 goroutine 的方案才可以达到控制他们的目的，这就是 Go 为我们提供的 Context。

### Context实现

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go func(ctx context.Context) {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("监控退出，停止了...")
				return
			default:
				fmt.Println("goroutine监控中...")
				time.Sleep(2 * time.Second)
			}
		}
	}(ctx)

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	cancel()
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}
```

`context.Background()` 返回一个空的 Context，这个空的 Context 一般用于整个 Context 树的根节点。然后我们使用 `context.WithCancel(parent)` 函数，创建一个可取消的子 Context，然后当作参数传给 goroutine 使用，这样就可以使用这个子 Context 跟踪这个 goroutine。

> 对于创建ctx，有三种方法：
>
> - **WithTimeout**
>
> *设置超时时间，some函数将ctx传下去，超过设定时间会自动调用cancel*
>
> ```go
> func exec() {
> 	ctx, cancel:= context.WithTimeout(context.Background(), 500*time.Millisecond)
> 	defer cancel()
> 	some(ctx)
> }
> ```
>
> - **WithCancel**
>
> *根据具体情况手动调用cancel*
>
> ```go
> func exec() {
> 	ctx, cancel := context.WithCancel(context.Background())
> 	deadline = time.Now().Add(500*time.Millisecond)
> 	if time.Now().After(deadline) {
> 		cancel()
> 	}
> }
> ```
>
> - **WithDeadline**
>
> *设置具体时间点，some函数将ctx传下去，超过设定时间会自动调用cancel*
>
> ```go
> func exec() {
> 	ctx, cancel := context.WithCancel(context.Background())
> 	deadline = time.Now().Add(500*time.Millisecond)
> 	if time.Now().After(deadline) {
> 		cancel()
> 	}
> }
> ```
>
> 

在 goroutine 中，使用 **select** 调用 **`<-ctx.Done()`** 判断是否要结束。如果接受到值的话，返回结束 goroutine ；如果接收不到就会继续进行监控。

> **Done方法**：
>
> 返回一个只读的chan，类型为struct{}。在goroutine中如果Done()返回的chan可读取，则意味着context已经发起了取消请求，我们通过Done方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源

那么 Context 是如何**发送结束指令**的呢？这就是示例中的 `cancel` 函数了，它是我们调用 `context.WithCancel(parent)` 函数生成子 Context 的时候返回的第二个返回值，是 `CancelFunc` 类型的。我们调用它就可以发出取消指令，然后监控 goroutine 就会收到信号，然后返回结束。

#### Context 控制多个 goroutine

```go
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	go watch(ctx,"【监控1】")
	go watch(ctx,"【监控2】")
	go watch(ctx,"【监控3】")

	time.Sleep(10 * time.Second)
	fmt.Println("可以了，通知监控停止")
	cancel()
	//为了检测监控过是否停止，如果没有监控输出，就表示停止了
	time.Sleep(5 * time.Second)
}

func watch(ctx context.Context, name string) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(name,"监控退出，停止了...")
			return
		default:
			fmt.Println(name,"goroutine监控中...")
			time.Sleep(2 * time.Second)
		}
	}
}
```

### Context 使用原则

- 不要把 Context 放在结构体中，要以参数的方式传递。
- 以 Context 作为参数的函数方法，应该把 Context 作为第一个参数，放在第一位。
- 给一个函数方法传递 Context 的时候，不要传递 nil，如果不知道传递什么，就使用 `context.TODO`。
- Context 的 Value 相关方法应该传递必须的数据，不要什么数据都使用这个传递。
- Context 是线程安全的，可以放心的在多个 goroutine 中传递。



------

向已关闭的chan写会报panic错误，但是**向已关闭的chan中读, chan会被立即触发**



## cancel源码 

-> **关闭当前上下文以及子上下文的`ctx.done` 管道**

```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
   if err == nil {   // 必须要有关闭的原因
      panic("context: internal error: missing cancel error")
   }
   c.mu.Lock()
   if c.err != nil {
      c.mu.Unlock()
      return // 已经关闭，返回
   }
   c.err = err  // 通过err标识已经关闭
   d, _ := c.done.Load().(chan struct{})
   if d == nil {
      c.done.Store(closedchan)
   } else {
      close(d)  // 【关闭当前done管道】
   }
   for child := range c.children {
      child.cancel(false, err)  // 遍历取消所有子上下文
   }
   c.children = nil   // 删除子上下文
   c.mu.Unlock()

   if removeFromParent {
      removeChild(c.Context, c)   // 从父上下文删除自己
   }
}
```



ctx.Done方法返回一个只读的chan，类型为struct{}

【如果Done方法返回的chan可读，意味着parent ctx已经发起了取消请求】，通过Done方法接收到这个信号后应该做清理操作，退出协程释放资源。



## closechan源码

```go
func closechan(c *hchan) {
    // 关闭一个nil channel, panic
   if c == nil {
      panic(plainError("close of nil channel"))
   }

    // 上锁
   lock(&c.lock)
    // 如果channel已关闭
   if c.closed != 0 {
      unlock(&c.lock)
      panic(plainError("close of closed channel"))
   }

   /*if raceenabled {
      callerpc := getcallerpc()
      racewritepc(c.raceaddr(), callerpc, abi.FuncPCABIInternal(closechan))
      racerelease(c.raceaddr())
   }*/

    // 修改关闭状态
   c.closed = 1

   var glist gList

   // 将channel所有等待接收队列中的reader释放
   for {
       // 从接收队列中出队一个
      sg := c.recvq.dequeue()
      if sg == nil { // 出队完了，跳出循环
         break
      }
       
       // 如果 elem 不为空，说明此 receiver 未忽略接收数据
		// 给它赋一个相应类型的零值
      if sg.elem != nil {
         typedmemclr(c.elemtype, sg.elem)
         sg.elem = nil
      }
      if sg.releasetime != 0 {
         sg.releasetime = cputicks()
      }
       // 取出 goroutine
       // 相连，形成链表
      gp := sg.g
      gp.param = unsafe.Pointer(sg)
      sg.success = false
      if raceenabled {
         raceacquireg(gp, c.raceaddr())
      }
      glist.push(gp)
   }

   // 将channel所有等待接收队列中的writers释放
   // 如果存在，这些协程将会 panic
   for {
      sg := c.sendq.dequeue()
      if sg == nil {
         break
      }
       // 发送者会 panic
      sg.elem = nil
      if sg.releasetime != 0 {
         sg.releasetime = cputicks()
      }
      gp := sg.g
      gp.param = unsafe.Pointer(sg)
      sg.success = false
      if raceenabled {
         raceacquireg(gp, c.raceaddr())
      }
      glist.push(gp)
   }
    // 解锁
   unlock(&c.lock)

   // Ready all Gs now that we've dropped the channel lock.
    // 遍历链表
   for !glist.empty() {
       // 移走glist链表的头
      gp := glist.pop()
      gp.schedlink = 0
       // 唤醒相对应的协程
      goready(gp, 3)
   }
}
```

对于一个 channel，recvq 和 sendq 中分别保存了阻塞的发送者和接收者。关闭 channel 后，对于等待接收者而言，会收到一个相应类型的零值。对于等待发送者，会直接 panic



###### 所以在不了解 channel 还有没有接收者的情况下，不能贸然关闭 channel



close 函数先上一把大锁，接着**把所有挂在这个 channel 上的 sender 和 receiver 全都连成一个链表**，再解锁。最后再将链表中所有的 sudog 全都唤醒。

唤醒之后，sender 会继续执行 chansend 函数里 goparkunlock 函数之后的代码，检测到 channel 已关闭，panic；

receiver 则比较幸运，执行chanrecv函数进行一些扫尾工作后，返回：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
	// .....
   lock(&c.lock)

   if c.closed != 0 {  // close-> closed == 1
      if c.qcount == 0 {
          // channel已关闭且循环数组 buf 没有元素
         if raceenabled {
            raceacquire(c.raceaddr())
         }
         unlock(&c.lock)
         if ep != nil {
            // 从一个已关闭的 channel 执行接收且不忽略返回值那么接收的值将是一个该类型的零值
            typedmemclr(c.elemtype, ep)  // typedmemclr 根据类型清理相应地址的内存
         }
          // 从一个已关闭的chan接收数据， selected会返回true
         return true, false
      }
   } else {
		// .....
   }

   // .....
}
```

这里，selected 返回 true，而返回值 received 则要根据 channel 是否关闭，返回不同的值。如果 channel 关闭，received 为 false，否则为 true。



关闭channel后读取数据的值

如果从一个带缓冲且有数据的chan中读数据，在chan关闭后依旧可以读到有效值：

```go
func TestChan(t *testing.T) {
   ch := make(chan int, 5)
   ch <- 18
   close(ch)
   x, ok := <-ch
   if ok {
      fmt.Println("received: ", x)
   }

   x, ok = <-ch
   if !ok {
      fmt.Println("channel closed, data invalid.")
   }
}
```

输出结果为：

received:  18
channel closed, data invalid.