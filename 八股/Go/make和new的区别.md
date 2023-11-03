## make和new的区别

都在堆上分配内存。

1、**new**分配完空间后是**将内存清0**而非初始化内存，**make**分配空间后是**初始化内存**，而不是清0

2、make返回的是引用类型本身，new返回指向类型的指针



#### 什么情况下用make，什么情况下用new

new只分配内存，make用于一些slice、map、channel的初始化

> **slice、map、channel用new会怎样？**
>
> silce、map、channel类型属于引用类型，go会给引用类型初始化为nil，nil是不能直接赋值的，并且不能用new分配内存。只能通过make初始化内存。