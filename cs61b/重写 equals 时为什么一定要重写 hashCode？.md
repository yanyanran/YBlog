## 重写 equals 时为什么一定要重写 hashCode？

> 不同对象的 hashCode 可能相同；但 hashCode 不同的对象一定不相等

equals 方法（用于检测一个对象是否等于另外一个对象）和 hashCode 方法都属于 Object 类，它们共同协作来判断两个对象是否相等。这样设计的原因就出在“性能” 2 字上。

如果不使用hash定位而直接查找，则会是这样的情况：

![](https://s4.51cto.com/oss/202112/13/2f709860343b06e24eec754f7335b310.jpg)

依次查找的效率实在过于低下。提高效率的方法是：

###### 当我们对比两个对象是否相等时，就可以先使用 hashCode 进行比较。

###### 如果比较的结果是 true，那么就可以使用 equals 再次确认两个对象是否相等；如果比较的结果还是 true，那么这两个对象就是相等的。否则认为两个对象不相等。 

![](https://s2.51cto.com/oss/202112/13/599fb64a9df9b879a7c1eddb25dc1fbe.jpg)

首先来看看equals方法：

### 1.equals 方法

在 Object 类中，equals将判断两个对象是否具有相同的引用。如果具有相同引用，那它们一定是相等的。

equals 方法的实现源码如下： 

```java
public boolean equals(Object obj) {  
    return (this == obj);  
} 
```

通过上述源码和 equals 的定义可以看出，在大多数情况来说equals 的判断是没意义的！

例如，使用 Object 中的 equals 比较两个自定义的对象是否相等，这就完全没有意义（因为无论对象是否相等，结果都是 false）

因此通常情况下，我们要判断两个对象是否相等，就一定要重写 equals 方法。

### 2.hashCode 方法

hashCode 翻译为中文是**散列码**，它是由对象推导出的一个整型值，并且这个值为任意整数（正数或负数）。

需注意：散列码**没有规律**。

如果 x 和 y 是两个不同的对象，x.hashCode() 与 y.hashCode() 基本上不会相同；但如果 a 和 b 相等，则 a.hashCode() 一定等于 b.hashCode()。



hashCode 在 Object 中的源码如下：

```java
public native int hashCode(); 
```

从上述源码可以看到，Object 中的 hashCode 调用了一个（native）本地方法，返回一个 int 类型的整数，这个整数可能是正数也可能是负数。

### 3.为什么要一起重写？

如果在重写 equals 时，不重写 hashCode，就会导致在某些场景下，例如**将两个相等的自定义对象存储在 Set 集合**时，就会出现程序执行的异常：

Set 进行去重操作时，会先判断两个对象的 hashCode 是否相同，此时因为没有重写 hashCode 方法，会直接执行 Object 中的 hashCode 方法，而 Object 中的 hashCode 方法对比的是两个不同引用地址的对象，所以结果是 false，那么 equals 方法就不用执行了，直接返回的结果就是 false：两个对象不是相等的，**于是就在 Set 集合中插入了两个相同的对象**。