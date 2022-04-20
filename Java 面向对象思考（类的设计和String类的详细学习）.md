# 目录

## 一、类的关系

    1、关联

    2、聚合、组合 

二、程序设计1：设计Course类

三、程序设计2：设计栈类

四、将基本数据类型值作为对象处理 

五、详探String类

    1、不可变字符串

    2、驻留字符串

    3、替换字符串

    4、拆分字符串

    5、使用模式匹配、替换和拆分（正则表达式）

    6、字符串和字符数组之间的转换

    7、更灵活的字符串类：StringBuilder类和StringBuffer类

        （1）StringBuilder中的构造方法

        （2）StringBuilder中修改字符串的构建器方法

        （3）StringBuilder中获取其他属性的方法

        （4）示例学习：判断回文时忽略非字母或数字的字符

六、编程练习

--------------------------------------------------------------

面向过程和面向对象的程序设计有很大的不同，Java中主要为面向对象进行程序设计，将焦点放在**类的设计**上。

面向对象的范式将数据和操作方法包含、耦合在一起，从而构成对象。

# 一、类的关系
通常来讲，类之间的关系包括：**关联、聚合、组合、继承**。

## 1、关联
>关联是**描述两个类之间活动的一种二元关系**

![avatar](https://img-blog.csdnimg.cn/38c529ac3b1f45e89ec7f5f7f84689e0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)
![avatar](https://img-blog.csdnimg.cn/d2a37cdbd8cf4c759ab00a784b784375.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)


## 2、聚合、组合 
**聚合是一种特殊的关联形式，它表示两个对象之间的归属关系**。如果被聚集对象的存在**依赖**于聚集对象，则称这两个对象间的关系为**组合**。

（由于聚合和组合关系都以同样的方式用类表示，所以不区分它们）

![avatar](https://img-blog.csdnimg.cn/6bf4c3b216d84326adf348d6ee3cd76c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

“继承”在下篇博客总结 

--------------------------------------------------------------

 # 二、程序设计1：设计Course类
![avatar](https://img-blog.csdnimg.cn/cd699ee8bb2946eba7bbdd17e412b1f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

根据上面的UML图，我设计了一个Course类 ，可实现对课程、学生的添加，对学生个数和名字的查询，对课程名字的查询和删除学生操作，以及对Course类进行测试的代码如下：

```
import java.util.Scanner;
class Course{
    private String courseName;
    private String[] students = new String[100];
    private int numberOfStudent;
 
    public Course(String courseName){
        this.courseName = courseName;
    }
    public String[] getStudent(){  //返回选这门课的所有学生
        return students;
    }
    public int getNumberOfStudent(){  //返回学生个数
        return numberOfStudent;
    }
    public String getCourseName(){  //返回课程名字
        return courseName;
    }
    public void addStudent(String student){  //从某门课添加一个学生
        students[numberOfStudent] = student;
        numberOfStudent++;
    }
    public void dropStudent(String student){  //从某门课删除一个学生
        //Programming
        //遍历数组找名为student的学生
        for(int i = 0;i < numberOfStudent;i++){
            if(students[i] == student)
            {
                for(int j = i + 1;j < numberOfStudent;i++,j++)
                {
                    //将后边的学生向前移动一位
                    students[i] = students[j];
                }
                break;
            }
        }
        numberOfStudent--;
    }
}
 
public class TestCourse { //测试类
    public static void main(String[] args){
        Course course1 = new Course("Data structures");
        Course course2 = new Course("Database Systems");
 
        course1.addStudent("Peter");
        course1.addStudent("Jim");
        course1.addStudent("Anne");
 
        course2.addStudent("Peter");
        course2.addStudent("Steve");
        course2.addStudent("Meredith");
        course2.addStudent("Mike");
 
        //1 第一门课程
        System.out.println("The course1's name is: " + course1.getCourseName());
        System.out.println("Number of students in course1: " + course1.getNumberOfStudent());
        //getStudent course1
        String[] students = course1.getStudent();
        for(int i = 0; i < course1.getNumberOfStudent(); i++){
            System.out.print(students[i]);
            if(i!=course1.getNumberOfStudent()-1)
            {
                System.out.print(", ");
            }
        }
 
        System.out.println( );
 
        //2 第二门课程
        System.out.println("The course2's name is: " + course2.getCourseName());
        System.out.println("Number of students in course2: " + course2.getNumberOfStudent());
        //getStudent course2
        String[] studentss = course2.getStudent();
        for(int i = 0; i < course2.getNumberOfStudent(); i++){
            System.out.print(students[i]);
            if(i!=course2.getNumberOfStudent()-1)
            {
                System.out.print(", ");
            }
        }
 
        System.out.println( );
 
        //3 删除学生
        System.out.println("delete someone now ? (1:yes  0:no)");
        Scanner input = new Scanner(System.in);
        int INPUT = input.nextInt();
        if(INPUT == 1)
        {
            course2.dropStudent("Meredith");
            System.out.println("The course2's name is: " + course2.getCourseName());
            System.out.println("Number of students in course2: " + course2.getNumberOfStudent());
            //getStudent course2
            String[] studentsss = course2.getStudent();
            for(int i = 0; i < course2.getNumberOfStudent(); i++){
                System.out.print(students[i]);
                if(i!=course2.getNumberOfStudent()-1)
                {
                    System.out.print(", ");
                }
            }
        }
 
    }
}
 
/*
The course1's name is: Data structures
Number of students in course1: 3
Peter, Jim, Anne
The course2's name is: Database Systems
Number of students in course2: 4
Peter, Jim, Anne, null
delete someone now ? (1:yes  0:no)
1
The course2's name is: Database Systems
Number of students in course2: 3
Peter, Jim, Anne
*/
```
--------------------------------------------------------------

# 三、程序设计2：设计栈类
![avatar](https://img-blog.csdnimg.cn/c28ba59f63d24a45957c58784a45570e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

根据上面的UML图对栈类进行实现，过程中微难点在于**push的实现**以及**StackOfIntegers构建默认容量为16的空栈部分**：

```
class StackOfIntegers{
    private int[] elements;  //栈中元素存储
    private int size;
    public static final int DEFAULT_CAPACITY = 16; //final关键字表示不能更改
 
    // 构建一个默认容量为16的空栈
    public StackOfIntegers(){
        //elements = new int[16];
        this(DEFAULT_CAPACITY);
    }
 
    // 构建一个指定容量的空栈
    public StackOfIntegers(int capacity){
        elements = new int[capacity];
    }
 
    // 如果栈为空返回true
    public boolean empty(){
        /*
        if(size == 0){
            return true;
        }else return false;
        */
        return size == 0;
    }
 
    // 返回栈中元素个数
    public int getSize(){
        return size;
    }
 
    // 将一个整数存到栈顶
    public void push(int value){
        if( size >= elements.length ){ // 如果栈满
            int[] temp = new int[elements.length * 2];  // 创建一个当前容量二倍的新数组
            System.arraycopy(elements, 0, temp, 0, elements.length); // 将当前数组内容复制到新数组中
            elements = temp;  // 将新数组的引用赋值给栈中当前数组
        }
        elements[size++] = value; // 添加新值
    }
 
    // 返回栈顶的整数而不从栈中删除它
    public int peek(){
        return elements[size];
    }
 
    // 删除栈顶整数并返回它
    public int pop(){
        /*
        int num = elements[size];
        elements[size] = 0;
        return num;
        */
        return elements[--size];
    }
}
 
//Test
public class TestStackOfIntegers {
    public static void main(String[] args){
        StackOfIntegers stack = new StackOfIntegers();
 
        for(int i = 0; i < 10; i++){
            stack.push(i);
        }
        while(!stack.empty()){
            System.out.print(stack.pop() + " ");
        }
    }
}
/* 9 8 7 6 5 4 3 2 1 0 */
```
--------------------------------------------------------------

# 四、将基本数据类型值作为对象处理 
出于对性能的考虑，Java中基本数据类型不作为对象使用，但我们可以将基本数据类型包装成对象。**java.lang包**中为基本数据类型们提供了相对应的**包装类**。通过使用包装类，**可以将基本数据类型值当作对象处理。** 

>例子：java.lang.Integer类和java.lang.Double类

![avatar](https://img-blog.csdnimg.cn/88cb26df3bcb494699832f2624ac4f92.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

 - 以Integer为研究对象：

* MAX_VALUE和MIN_VALUE表示该类型下的最大最小值  
* 1红色框部分是Integer类  
* 2黄色框部分是数据类型之间的转换  
* compareTo方法比较两个数值，相同返回0，大于返回1，小于返回-1  
* 3绿色框部分的方法是将值转为字符串形式  
* 4蓝色框部分的方法是将数值型字符串转化为数值（radix表示进制）  

**如果想要表示任意大小的数字，可以使用java.math包中的BigInteger类和BigDecimal类。BigInteger类可以表示任意大小的整数，而BigDecimal类可以达到任意精度。**

--------------------------------------------------------------

# 五、详探String类
## 1、不可变字符串
**String对象是不可变的，其内容不可更改**。假如有以下代码：
```
String s = "Java";
s = "HTML";
```
此时如果输出s，答案是HTML，但这并不代表着第二行的语句改变了字符串的内容。我们可以理解为数组引用值，第二行的语句只是创建了一个“HTML”的新对象，并将其引用赋值给了s，而**第一行创建的“Java”依旧存在，只是不能访问它了**，而并非将“Java”修改为了“HTML”。

 ![avatar](https://img-blog.csdnimg.cn/d25f53a966ae427baa8f53dd1445a919.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 2、驻留字符串
Java对具有相同字符序列的字符串字面值使用同一个实例，例如下面：
```
String s1 = "lalalalala";
String s2 = new String("lalalalala");
String s3 = "lalalalala";
 
System.out.println("s1 == s2 is " + (s1 == s2));
System.out.println("s1 == s3 is " + (s1 == s3));
 
/*
s1 == s2 is false
s1 == s3 is true
*/
```
s1和s2不同的原因是虽然它们的内容相同，但它们是**不同的字符串对象**。 

## 3、替换字符串
虽说一旦创建了字符串其内容就不能改变，但我们可以使用**replace、replaceFirst、replaceAll**来返回一个源于原始字符串的新字符串。

 ![avatar](https://img-blog.csdnimg.cn/c58f551962f745609b1ce2a75b5c1118.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

## 4、拆分字符串
**split方法可以用指定的分隔符从字符串中提取标记**。比如：
```
String[] s = "lalalala#yeyeyeye#hahahaha".split("#");
for(int i = 0; i < s.length; i++)
{
    System.out.print(s[i] + " ");
}
 
/*
lalalala yeyeyeye hahahaha
*/
```
## 5、使用模式匹配、替换和拆分（正则表达式）
String类中的equals方法用于比较两个字符串是否相同，而功能更强大的**matches方法**用于**检测字符串是否匹配给定的正则表达式**。下面的语句均为true：
```
“Java is fun”.matches("Java.*");
“Java is cool”.matches("Java.*");
“Java is powerful”.matches("Java.*");
 
/*
true
true
true
*/
```
“Java.*”是一个正则表达式，它描述的字符串模式是从“Java”开始，后面可以紧跟任意字符。
```
“440-33-2867”.matcher(“\\d{3}-\\{2}-\\{4}”);
 
/*true*/
 ```
**上面的\\d表示单个数字，\\d{3}表示三个数字**
```
String s = "a+b&$c".replaceAll("[&#+]","NNN"); //一定要有[]符号，否则替换无效！
System.out.print(s);
 
/*
aNNNbNNNNNNc
*/
```
**上面正则表达式[&#+]指定匹配替换为NNN** 

## 6、字符串和字符数组之间的转换
>串-->数组：toCharArray方法（Double.parseDpuble()方法转换值）
 ```
char[] chars = "Java".toCharArray();
 ```
>数组-->串：构造方法String(char[ ]) 或 valueOf(char[ ])方法
```
String str = new String(new char[]{"J","a","v","a"});
//或
String str = String.valueOf(new char[]{"J","a","v","a"});
 ```
## 7、更灵活的字符串类：StringBuilder类和StringBuffer类
**java.lang包**中的StringBuilder类和StringBuffer类和String类的区别在于前面提到的：**String对象一经创建就无法修改，而这两个类的对象可以对其进行任意修改**。

例子如下：
```
public class Test {
    public static void main(String[] args){
        String s = "Java"; //String对象
        StringBuilder sb = new StringBuilder(s); //StringBuilder对象
        change(s, sb);
 
        System.out.println(s); //输出结果：Java
        System.out.println(sb); //输出结果：Java and HTML
    }
 
    public static void change(String s, StringBuilder sb){
        s = s + " and HTML";
        sb.append(" and HTML");
    }
}
```
至于StringBuilder类和StringBuffer类的不同在于，**StringBuilder适合单任务访问，而StringBuffer适合多任务并发访问**。下面先拿StringBuilder开刀：

## (1)StringBuilder中的构造方法
 ![avatar](https://img-blog.csdnimg.cn/362ad0a0b478423097c0022619bc5c0e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

## (2)StringBuilder中修改字符串的构建器方法
 ![avatar](https://img-blog.csdnimg.cn/901c72e8c43840e786ff405a4781d465.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

>append方法：追加  
>delete方法：删除  
>insert方法：插入  
>replace方法：替换  
>reverse方法：倒置  
>setCharAt方法：设新  
**注意：如果一个字符串不需要任何改变，则使用String而不用StringBuilder，因为string更高效。**

## (3)StringBuilder中获取其他属性的方法
![avatar](https://img-blog.csdnimg.cn/a8b1c8b546e04aed9c3ba1154d7b61d0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

## (4)示例学习：判断回文时忽略非字母或数字的字符
以下程序在判断回文的时候多加了一个功能：就是忽略掉非字母和数字的部分再进行判断。
```
import java.util.Scanner;
 
public class PalindromeIgnore {
    public static void main(String[] args){
        Scanner input = new Scanner(System.in);
        System.out.println("Enter a string : ");
        String s = input.nextLine();
 
        System.out.println("忽略非字母或数字字符 \n " + s + " 是回文吗?: " + isPalindrome(s));
    }
 
    public static boolean isPalindrome(String s){
        String s1 = filter(s);
        String s2 = reverse(s1);
        return s2.equals(s1);
    }
 
    public static String filter(String s){  //逐一检查字符串s中的每个字符，如果字符是数字或字母，就将其复制到字符串构建器中
 
        StringBuilder stringBuilder = new StringBuilder();
        for(int i = 0;i < s.length();i++)
        {
            if(Character.isLetterOrDigit(s.charAt(i))){ //用Character类中的isLetterOrDigit方法判断字符是否为数字或字母
                stringBuilder.append(s.charAt(i)); //为真值-->复制字符到新字符串中
            }
        }
        return stringBuilder.toString();
    }
 
    public static String reverse(String s){  //倒置字符串
        StringBuilder stringBuilder = new StringBuilder(s);
        stringBuilder.reverse();
        return stringBuilder.toString();
    }
}
```
先构造一个filter方法对字符串进行遍历检查，方法中使用**Character类中的isLetterOrDigit方法**对字符串（**charAt(i)方法返回下标为i的字符**）进行判断，如果是数字或字母，则复制到新的字符串s1中。  

接下来使用reverse方法对字符串s1进行倒置操作得到新字符串s2，最后使用**equals方法**对s1和s2进行**对比**。如果相同，则为回文。 

--------------------------------------------------------------

# 六、编程练习 
本节学完对面向对象的概念又有了一层次的了解，编程题主要考究对**类的设计**。

 ![avatar](https://img-blog.csdnimg.cn/735367af61ff41da98b8caa4653eb098.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aKcIOeEtg==,size_20,color_FFFFFF,t_70,g_se,x_16)

代码实现如下：
```
public class MyInteger {
    private int value;
 
    public MyInteger(int value){
        this.value = value;
    }
 
    public int getValue(){
        return value;
    }
 
    public boolean isEven(){
        return value % 2 ==0;
    }
    public boolean isOdd(){
        return !isEven();
    }
    public boolean isPrime(){
        if(value == 0 || value == 1){
            return false;
        }
        for(int i = 2;i < this.value;i++){
            if(value % i == 0){
                return false;
            }
        }
        return true;
    }
 
    public static boolean isEven(int value){
        return new MyInteger(value).isEven();
    }
    public static boolean isOdd(int value){
        return new MyInteger(value).isOdd();
    }
    public static boolean isPrime(int value){
        return new MyInteger(value).isPrime();
    }
 
    public static boolean isEven(MyInteger myInteger){
        return myInteger.isEven();
    }
    public static boolean isOdd(MyInteger myInteger){
        return myInteger.isOdd();
    }
    public static boolean isPrime(MyInteger myInteger){
        return myInteger.isPrime();
    }
 
    public boolean equals(int value){
        return this.value == value;
    }
    public boolean equals(MyInteger myInteger){
        return this.equals(myInteger.value);
    }
 
    public static int parseInt(char[] chars){
        String string = new String(chars);
        return Integer.parseInt(string);
    }
 
    public static int parseInt(String string){
        return Integer.parseInt(string);
    }
}
```
