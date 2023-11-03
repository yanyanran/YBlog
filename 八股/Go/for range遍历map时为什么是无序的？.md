## for range遍历map时为什么是无序的？

```go
func main() {
	m := make(map[int32]string)
	m[0] = "EDDYCJY1"
	m[1] = "EDDYCJY2"
	m[2] = "EDDYCJY3"
	m[3] = "EDDYCJY4"
	m[4] = "EDDYCJY5"

    // 遍历map
	for k, v := range m {
		log.Printf("k: %v, v: %v", k, v)
	}
}
```

上面这段代码，每次运行结果都不一样（！what）



for range map在开始处理循环逻辑的时候做了随机播种：**随机生成一个随机数，选择一个桶位置作为起始点进行遍历迭代**