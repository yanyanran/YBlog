# 目录

一、为对象定义类

二、构造方法

三、Java库中的各种类

    1、Date类 

    2、Random类

    3、Point2D类

四、实例(instance)和静态(static)

    1、实例变量和静态变量

    2、实例方法和静态方法 

        （1）关系

        （2）静态方法--->实例变量

        （3）程序设计

五、可见性修饰符

    1、数据域封装（private）

    2、访问器方法和修改器方法

六、对象数组

七、不可变对象和类 

八、变量作用域 

九、this引用 

    1、使用this引用数据域

    2、使用this调用构造方法

-------------------------------------------------------------------------------------------------------
# 一、为对象定义类
我将对象与类之间的关系总结为了以下这张图：

![avatar](https://img-blog.csdnimg.cn/167c8284fd584341afae10e7b9fc9cb2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

总而言之，类（class）是一个模板，而对象（object） 是一个实体，可以理解为**对象是类的具象化体现**。并且一个模板可以产生多个实体，也就是说**可以从一个类中创建多个对象**。

# 二、构造方法
同时类还提供了一种特殊类型方法：构造方法。**构造方法的作用：主要用来初始化对象、构造对象。

构造方法类似于C中的自定义函数，调用它就可以产生一个新对象。下面这段代码就使用了构造方法：

```
package PACKAGE_NAME;
 
public class EXERCISE { // 公共类(主类)
    public static void main(String[] args) // main方法
    {
        Circle circle1 = new Circle();
        System.out.println("The area of the circle of radius "+ circle1.radius + "is "+ circle1.getArea());
 
        Circle circle2 = new Circle(25);
        System.out.println("The area of the circle of radius " +circle2.radius + "is "+ circle2.getArea());
 
        Circle circle3 = new Circle(125);
        System.out.println("The area of the circle of radius "+ circle3.radius + "is "+ circle3.getArea());
 
        circle2.radius = 100;
        System.out.println("The area of the circle of radius "+ circle2.radius + "is "+ circle2.getArea());
    }
}
 
class Circle{   // 类
    double radius;  // 数据域
 
    Circle(){      // 构造方法
        radius = 1;
    }
 
    Circle(double newRadius){
        radius = newRadius;
    }
 
    double getArea(){          // 方法
        return radius * radius * Math.PI;
    }
 
    double getPerimeter(){
        return 2 * radius * Math.PI;
    }
 
    void setRadius(double newRadius){
        radius = newRadius;
    }
}
```
输出：The area of the circle of radius 1.0is 3.141592653589793
          The area of the circle of radius 25.0is 1963.4954084936207
          The area of the circle of radius 125.0is 49087.385212340516
          The area of the circle of radius 100.0is 31415.926535897932 

>这段代码中包含两个类，我们把包含main方法的类称为主类（公共类：必须与文件同名），即EXERCISE类。而第二个类---Circle类中则使用了构造方法。

其中构造方法部分为： 
```
    Circle(){      // 无参构造方法
        radius = 1;
    }
 
    Circle(double newRadius){   // 构造方法
        radius = newRadius;
    }
 ```
>构造方法名字必须与类名相同
>构造方法中没有返回值类型
>构造方法是在创建对象时，由一个new操作符调用的
构造方法和普通方法的区别：http://t.csdn.cn/JQVgH

# 三、Java库中的各种类
## 1、Date类 
之前有用过java.util.Arrays类中的方法对数组进行操作。而**java.util.Date类**中提供了**对日期和时间的封装**：
```
java.util.Date date = new java.util.Date();
System.out.println(date.getTime());  //date.getTime()方法返回自从GTM时间1970年1月1日至今流逝的时间
System.out.println(date.toString());  //date.toString()方法返回日期和和时间的字符串
System.out.println(date.setTime(time);); //date.setTime()方法用于设置此日期时间
/*
1649684753359
Mon Apr 11 21:45:53 CST 2022
*/
```
## 2、Random类
我们可以使用Math.random（）来获取一个0.0到1.0之间的double型随机值，而我们也可以使用**java.util.Random类**产生一个int、long、double、float、boolean型值。
```
    java.util.Random number1 = new java.util.Random(4); //seek = 4
    System.out.print("number1 is ");
    for(int i=0;i<10;i++)
    {
        System.out.print(number1.nextInt(1000) + " ");
    }
 
    System.out.print("\n");
    java.util.Random number2 = new java.util.Random(6); //seek = 6
    System.out.print("number2 is ");
    for(int i=0;i<10;i++)
    {
        System.out.print(number2.nextInt(100) + " ");
    }
/*
number1 is 862 452 303 558 767 105 911 846 462 427 
number2 is 11 76 66 78 41 3 77 4 90 41 
*/
```
同时要注意，使用Random类时我们需要填入一个种子值（seed）：

>Random类中实现的随机算法是伪随机，也就是有规则的随机。在进行随机时，随机算法的起源数字称为种子数，在种子数的基础上进行一定的变换，从而产生需要的随机数字

## 3、Point2D类
**javafx.geometry.Point2D**用于表示**二维平面上的点**。如：
```
package PACKAGE_NAME;
 
import java.awt.geom.Point2D;
import java.util.Scanner;
 
public class EXERCISE { // 公共类(主类)
    public static void main(String[] args) // main方法
    {
        Scanner input = new Scanner(System.in);
        System.out.println("input the point1's information: ");
        Double x1 = input.nextDouble();
        Double y1 = input.nextDouble();
        System.out.println("input the point2's information: ");
        Double x2 = input.nextDouble();
        Double y2 = input.nextDouble();
 
        Point2D p1 = new Point2D(x1,y1);
        Point2D p2 = new Point2D(x2,y2);
        System.out.println("point1 is "+ p1.toString());
        System.out.println("point2 is "+ p2.toString());
        System.out.println("The distance between p1 and p2 is " + p1.distance(p2));
        System.out.println("The midpoint between p1 and p2 is " + p1.midpoint(p2).toString());
    }
}
```
其中调用p1.distance(p2)返回两个点之间的距离，调用p1.midpoint(p2)返回两点的中点。

# 四、实例(instance)和静态(static)
## 1、实例变量和静态变量
>先提一点，在Java中，**Java 没有给方法中的局部变量赋默认值**。也就是说我们在声明变量的时候**一定要对变量进行初始化操作**，否则会编译错误。

静态变量和实例变量统称为成员变量。**实例变量是属于类的某个特定实例的，它不能被同一个类的不同对象所共享**。如果想让一个类中的所有实例**共享数据，可以使用静态变量（类变量），使用static修饰**。

下面一段代码体验两个变量的不同之处：
```
public class EXERCISE {
    private static int staticInt = 2;//静态变量
    private int random = 2;//实例变量
 
    public EXERCISE() {
        staticInt++;
        random++;
        System.out.println("staticInt = " + staticInt + "  random = " + random);
    }
 
    public static void main(String[] args) {
        EXERCISE test = new EXERCISE();
        EXERCISE test2 = new EXERCISE();
    }
}
```
输出结果： 
![avatar](https://img-blog.csdnimg.cn/649e862ec9a24e65bb8cd3b7f98a1c97.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


 结果显示在第二次对变量进行++时，**静态变量在原本上一次结果的基础上加1**，而实例变量则初始化了值后才加的1。

>（静态变量有记忆，而实例变量比较健忘）

## 2、实例方法和静态方法 
### （1）关系
对于方法而言，下面这张图展示了实例方法和静态方法的关系：

![avatar](https://img-blog.csdnimg.cn/c1bdf89e9ad449be939454f575b85b2d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

总之就是，**实例方法可以直接调用实例或静态方法，但静态方法只能直接调用静态方法或数据域而无法直接调用实例**。

### （2）静态方法--->实例变量
>如果**在静态方法中想要调用或者访问实例变量**，我们可以通过**创建一个用类名声明的对象，然后通过这个对象进行静态访问和调用**，当然，实例对象也可以用。
比如：
![avatar](https://img-blog.csdnimg.cn/df7ad7c3b9c84627b2163711d1657a88.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


报错显示： 
![avatar](https://img-blog.csdnimg.cn/f1c97e4ab5204831afdb0735b3c65249.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


 此时我们创建一个对象a，然后通过进行访问和调用：
![avatar](https://img-blog.csdnimg.cn/0ba5d709994b4fd1827ce5822019c257.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


完成。

除了通过对象调用，针对静态变量或方法的访问和调用，我们也可以直接使用类名：
- 调用静态方法：类名 . 方法名（参数）

- 访问静态变量：类名 . 静态变量

 **（注意：上面这个表示方法只适用于对静态方法和变量，实例不能用）**

为啥实例不能用？体现在下面这个例子里： 
![avatar](https://img-blog.csdnimg.cn/e8e2f654bb764ceb8e4ca3651eb05bc0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


 F是类，f是对象。在右边方框中， 只有两条命令是不正确的：因为它们两条都是通过类访问实例变量和方法，但这种方法只能对静态变量或方法使用，因此是错的。

### （3）程序设计
在程序设计中，判断变量或方法究竟应该是静态还是实例的依据在于：

**判断它是否依赖于类中的某个具体实例。如果依赖，就是实例；不依赖，就是静态。**

（比如：每个圆都有属于它们自己的半径，也就是说半径 依赖 圆，所以在定义半径时，应定义为实例变量。 ）

# 五、可见性修饰符
Java中的可见性修饰符有四个，分别是**public、protected、default和private**。它们可以**用于确定一个类以及它成员的可见性。

>public：所有可见

>protected：本包和子类可见

>default：本包可见（无修饰符）

>private：仅对本类可见

![avatar](https://img-blog.csdn.net/20150409153405264)

 * 同时注意：private（数据域）只能应用在类的成员上，而public（构造方法或方法）能用在类或类的成员上。

## 1、数据域封装（private）
为了避免用户对数据域进行直接修改，使用private修饰符将数据域声明为**私有**，在保护数据域的同时也让类更加易于维护。

被private声明后即为私有数据。**私有数据值只能在定义它们的类中被访问，而不能通过类声明的对象访问。

## 2、访问器方法和修改器方法
虽然私有数据域不能被对象从定义该私有域的类外访问，但是经常会有客户端需要存取或修改数据域，这时我们可以：

- **访问私有数据域** ----> 获取（getter）方法返回数据域的值：访问器方法

- **更新私有数据域** ---> 设置（setter）方法给数据域设置新值：修改器方法

访问器方法的命名习惯：

非布尔型： getPropertyName（）；

布尔型：isPropertyName（）；

修改器方法的命名习惯：setPropertyName（dataType dataname）；

# 六、对象数组
数组可以存储基本类型值，也可以存储对象。下面的这段代码求圆的总面积，演示了如何使用对象数组：
```
package PACKAGE_NAME;
 
class Circle {   // Circle类
    public double radius; // 半径
    public final double PI = 3.14;
 
    // 得半径
    public double getRadius() {
        return radius;
    }
    // 求面积
    public double getArea() {
        return radius * radius * PI;
    }
    // 有参构造
    public Circle(double radius) {
        this.radius = radius;
    }
}
 
public class EXERCISE {
        public static void main(String[] args)
        {
            Circle[] circleArray = createCircleArray();
            printCircleArray(circleArray);
        }
 
        public static Circle[] createCircleArray() //创建一个由5个园对象组成的数组
        {
            Circle[] circleArray = new Circle[5];
 
            for(int i = 0;i < circleArray.length; i++)
            {
                circleArray[i] = new Circle(Math.random() * 100);
            }
            return circleArray;
        }
 
        public static void printCircleArray(Circle[] circleArray)
        {
            System.out.printf("%-30s%-15s\n","Radius","Area");
            for(int i = 0; i<circleArray.length; i++)
            {
                System.out.printf("%-30s%-15s\n", circleArray[i].getRadius(), circleArray[i].getArea());
            }
            System.out.println("-----------------------------------");
 
            System.out.printf("%-30s%-15f\n","The total area of circles is", sum(circleArray));
        }
 
        public static double sum(Circle[] circleArray)
        {
            double sum = 0;
            for(int i = 0; i <circleArray.length; i++)
            {
                sum += circleArray[i].getArea();
            }
            return sum;
        }
    }
 
/*输出：
Radius                        Area           
0.5963184324727311            1.1165704129271485
46.18923620915795             6699.019000578118
44.4055716710281              6191.624057652754
81.52440196746916             20869.156284721957
43.151154270782946            5846.749440788856
-----------------------------------
The total area of circles is  39607.665354   
*/
```
其中创建包含5个Circle对象的关键语句是：

>Circle[] circleArray = new Circle[5];

# 七、不可变对象和类 
不可变对象和类的内容不能被改变。要使得一个类成为不可变的，需要满足：

>所有数据域都是私有的（private）
>没有修改器方法
>没有返回一个指向可变数据域的引用的访问器方法
# 八、变量作用域 
![avatar](https://img-blog.csdnimg.cn/4cada1c8038c4bc0b4ea4686a207b51d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

从上面例子可以看出：x=1，y=0。因为在方法p中又对x定义了一次，并且最终的输出也是在方法p中执行，所以x用的是后者声明的一次，而y用的是最初的初始化。再看一段例子：
```
public class CIRCLE {
    private static int i = 0;
    private static int j = 0;
    public static void main(String[] args)
    {
        int i = 2;
        int k = 3;
 
        { //这个花括号！
            int j = 3;
            System.out.println("i + j is " + i + j);
        } //这个花括号！
        
        k = i + j;
        System.out.println("K is " + k);
        System.out.println("j is " + j);
    }
}
```
 注意上面标示的**花括号**。当花括号存在时，输出结果为：

![avatar](https://img-blog.csdnimg.cn/862063fa02724e72ad24ed9e500288a5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_18,color_FFFFFF,t_70,g_se,x_16)

当删除花括号时，输出结果为： 

![avatar](https://img-blog.csdnimg.cn/cea5891c056d4fef8a3861fc765d493e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_18,color_FFFFFF,t_70,g_se,x_16)

# 九、this引用 
关键字this引用对象自身。

>同时注意一点，Java中要求在构造方法中，this语句应该在任何其他可执行语句之前出现。

## 1、使用this引用数据域：
![avatar](https://img-blog.csdnimg.cn/e263f53f8ca84f29bcb99918523599b0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

其中a方法是正确的，b中的实现是错误的。数据域radius被设置方法中的参数radius所隐藏，所以需要采用this.radius这样的语法来引用方法中的数据域名字。

## 2、使用this调用构造方法：
![avatar](https://img-blog.csdnimg.cn/c76ce80bfff84ec392c49ec0d9e4e043.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

这里的this调用第一个构造方法。
