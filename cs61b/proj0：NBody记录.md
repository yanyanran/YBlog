# proj0：NBody记录

- java图形绘制API -- ***StdDraw库***使用参考：https://introcs.cs.princeton.edu/java/stdlib/javadoc/StdDraw.html



- 写到三分之二的时候，在完善Planet.java的main函数、进行第一个测试时编译运行程序出现错误：

![](https://github.com/yanyanran/pictures/blob/main/erro1.jpg?raw=true)

解决：应该是在安装jdk的时候丢失了一些文件，我将原本配置里的jdk11更换为jdk17，然后在终端执行：

```java
sudo apt install openjdk-17-jdk --fix-missing
```

> “这意味着您已经安装了 jdk，但其中缺少一些文件。因此，您只需使用“--fix-missing”就可以像小菜一碟一样轻松地修复它”

fixed