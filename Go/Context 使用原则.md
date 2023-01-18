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