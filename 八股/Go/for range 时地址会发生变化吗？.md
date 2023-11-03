## for range 时地址会发生变化吗？

在使用 for range 语句遍历切片或数组时，每次迭代都会返回该元素的副本，而不是该元素的地址。这意味着，每次循环完成后，该元素所对应的地址都是相同的，**并不会改变**。

```go
type girl struct {
	Name string
	Age int
}

func main() {
	gl := make(map[string]*girl)
	studs := []girl{
		{Name: "Lili", Age: 23},
		{Name: "Lucy", Age: 24},
		{Name: "Han Mei", Age: 21},
	}

	for _, v := range studs {
		gl[v.Name] = &v  // 这里的v其实一直都是同一个v
	}

	for mk, mv := range gl {
		fmt.Println(mk, "=>", mv.Age)
	}
}

// Lili => 21
// Lucy => 21
// Han Mei => 21
```

解决：

```go
// 在每次循环时，创建一个临时变量
for _, v := range studs {
	temp := v
	gl[v.Name] = &temp
}
```

