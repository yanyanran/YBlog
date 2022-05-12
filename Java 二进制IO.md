写哈夫曼树项目的时候发现处理字节流或是文件流的时候需要一些处理的方法，所以返回去又略略学习了下二进制IO这一块

# 目录

## 文件读取/写入字节：FileInputStream类和FileOutoutStream类

## 过滤数据基类：FilterInputStream类和FilterOutoutStream类 

## 字节与基本类型值的相互转换：DataInputStream类和DataOutputStream类


Java提供了很多类用于实现文本IO和二进制IO。二进制文件的优势在于二进制文件的处理效率比文本文件高，因为文本文件需要编码和解码，而二进制文件则不需要。

下面学习二进制IO类。二进制输入类根类为InputStream，二进制输出类根类为OutputStream，它们都是抽象类，后面延伸出了很多子类：

![avator](https://img-blog.csdnimg.cn/c98a696ae53a4537859479504216fa35.png)

文件读取/写入字节：FileInputStream类和FileOutoutStream类
这两个类用于从文件读取字节和向文件写入字节，这两个类相较于父类没有引入新的方法。

几乎所有的IO类中的方法都会抛出异常java.io.IOException，所以必须在方法中作出异常声明：

![avator](https://img-blog.csdnimg.cn/516fc1f754534d7eb0cd597d8412f567.png)

使用FileInputStream类和FileOutoutStream类创造实例进行文件写入和读取：

```
import java.io.*;
 
public class TestFileStream {
    public static void main(String[] args) throws IOException{
        try(
                FileOutputStream output = new FileOutputStream("temp.dat");
                ){
            for(int i = 1; i <= 10; i++){
                output.write(i); //写入
            }
        }
        
        try(
                FileInputStream input = new FileInputStream("temp.dat");
                ){
            int value;
            while((value = input.read()) != -1){ 
        //通过input.read()读取一个字节检验是否为-1。-1时代表文件结束
                System.out.println(value + " ");
            }
        }
        }
}
```

过滤数据基类：FilterInputStream类和FilterOutoutStream类 
这两个类是用于过滤数据的基类。上面介绍的读取方法read只能用于读取字节，如果需要读取整数值、双精度值或者字符串，就可以用过滤器类来包装字节输入流。需要处理基本数据类型时可以使用DataInputStream类和DataOutputStream类。

字节与基本类型值的相互转换：DataInputStream类和DataOutputStream类
DataInputStream类：

        从数据流读取字节，并将其转换为基本类型值或字符串；（字节--> 值）

DataOutputStream类：

        将基本类型值或字符串转为字节，并输出到流。（值--> 字节）

并且它们实现了DataInput和DataOutput接口：

![avator](https://img-blog.csdnimg.cn/e6f5a45c8264462d87e2166119271b29.png)
![avator](https://img-blog.csdnimg.cn/45f5872f6b804f6d87936885345807de.png)


其中写入字符串有两个方法可选择：writeBytes和writeChars：

* writeBytes适用于由ASCII字符构成的字符串

* 如果字符串含非ASCII字符，则选择使用writeChars
