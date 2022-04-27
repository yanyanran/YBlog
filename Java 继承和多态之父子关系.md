# 目录

一、继承

1、定义

2、语法

3、注意点

4、指明父类：super关键字

	（1）调用父类构造方法

	（2）构造方法链

	（3）调用父类普通方法

二、重载和重写

	（1）方法重写

		重写标注

      	   	重写Object类中的equals方法

	（2）方法重载

三、多态

	（1）性质

	（2）对象转换

	（3）实例化：instanceof关键字

	（4）实例和对象的区别

四、动态绑定

	（1）声明类型和实际类型

	（2）动态绑定

	（3）方法匹配和方法绑定

	（4）一些示例代码题：

五、存储不限定个数对象：ArrayList类

	注意1：

	注意2： 

六、保护数据域：protected关键字

七、 防止继承和重写：final修饰符

--------------------------------------------------------

>面向对象程序设计三大支柱是：***封装、继承、多态***

--------------------------------------------------------
# 一、继承
## 1、定义
从已经存在的类中定义新的类，称为继承。由继承可以引出父类和子类的定义：

继承使得你可以定义一个通用的类（即父类、超类、基类），之后继承该类成为一个更特定的类（即子类/继承类/派生类）

## 2、语法
>xx(子类)  extends  XXX(父类)   

关键字extends告诉编译器，XX类继承自XXX类。经过上面的语句，XX继承了XXX中的方法和数据域。

## 3、注意点
虽说继承了父类的数据域，但如果在父类中的数据是用“private”修饰，就不能直接在子类中访问。如果；父类中定义了公共的访问器和修改器，那么想要修改和读取只能通过它们的访问方法和修改方法操作；
子类不继承父类的构造方法；
Java中不允许多重继承，一个类只能直接继承自一个父类。（但多重继承可以通过接口实现，后面将会学习）
4、指明父类：super关键字
super代指父类，可以使用super关键字调用父类中的普通方法和构造方法。

### （1）调用父类构造方法
语法：super()或super(参数);  并且super语句必须放在首句。





上面的例子中，子类的方法使用访问器对父类中的私有数据域color和filled进行获取（构造方法）。而使用super（color, filled）语句就可以一键调用父类的构造方法。

### （2）构造方法链
构造方法链值的是一条引用链。有一个点需要注意：在一般情况下，我们最好能为每个类（父类）提供一个无参构造方法，这样能够便于对该类进行继承，避免出错。并且在子类构造无参方法前，无法调用父类的无参构造方法。

看下面这个例子：



该程序会出现错误。创建B类的对象时，首先会构造B类的方法。在该程序内，是构造B类的无参构造函数；按照方法链结构，在构造B类前会首先构造父类A类的无参构造函数，而A类显式的定义了构造方法，所以没有无参构造方法，因此错误。

### （3）调用父类普通方法
语法：super.方法名(参数);  与继承构造方法不同，这个方法可以不用放在首句。

# 二、重载和重写
## （1）方法重写
子类从父类中继承方法，而子类修改父类中定义的方法实现成为方法重写。

重写方法的两者之间：方法名一样、返回类型一样、参数类型一样

静态方法不能被重写。如果父类中定义的静态方法在子类中被重写，那么父类中定义的静态方法将会被隐藏。

想要调用隐藏方法我们可以：

父类名. 静态方法名
重写标注
为了避免错误，可以使用重写标注，即在子类的方法面前放一个@Override 。当具有此标注的方法没有重写父类的方法时，编译器将会报一个错误。

重写Object类中的equals方法
equals方法用于测试两个对象是否相等，其默认实现为：

public boolean equals(Object obj){
    return (this == obj);
}
== 操作符用于检测两个引用是否指向同个对象，我们可以在自己的自定义类中重写这个方法。

**重写时要注意一点：

在子类中使用签名equals( Otherclassname obj)是一个常见的错误，应该使用固定签名equals（Object obj）。 

## （2）方法重载
重载就是使用相同的名字但不同的参数列表来定义多个方法。

重载方法的两者之间：方法名一样、返回类型一样、参数类型不同

下面这个例子可看出重写和重载的区别：



 上面例子中，左边a为方法重写，右边b为方法重载。

当运行a中主类中的方法时，a.p(10)和a.p(10.0)最终调用的方法都是子类A中的方法，因此最终输出都是10.0；
当运行b中主类中的方法时，因为重载方法的参数数据类型不同，本着对号入座的原则，a.p(10)调用的是父类B中的方法，输出结果为10；而a.p(10.0)调用的是子类A中的方法，输出结果为20.0。
同时注意一点：方法重写发生在具有继承关系的不同类中，而重载可以发生在同一个类中的同时也可以发生在具有继承关系的不同类中。

# 三、多态
## （1）性质
多态 ---> 即：父类的变量可以引用子类的对象。这样可以增强方法的反复利用性，可避免不必要的重复的方法重载。

多态的存在有3个必要条件：继承、方法重写、父类引用指向子类对象。

父类引用指向子类对象后，用该父类引用调用子类重写的方法，此时多态就出现了。

## （2）对象转换
向上转型（自动类型转换）：在多态中需要将子实例-->父变量 Fu f= new Zi()，只有这样该引用才能够具备技能调用父类的方法和子类的方法。

向下转型（强制类型转换）：即父实例-->子变量 Zi z = (Zi)f，此时需强制转换。

 注意：转换的前提是--转换对象必须是子类的一个实例，否则会编译错误。

为了在性质转换前确保该对象是另一个对象的实例，我们可以使用instanceof关键字：

## （3）实例化：instanceof关键字
语句：变量(对象)A + instanceof + 声明类型B

经过下面语句后，变量（对象）就会被声明为声明类型，它告诉编译器A是B的一个实例。 

下面代码展示了多态的使用和类型转换的示例：

class Animal{
    public void shout(){
        System.out.println("叫了一声！");
    }
}
 
class Dog extends Animal{
    public void shout(){
        System.out.println("旺旺旺！");
    }
    public void seeDoor(){
        System.out.println("看门中....");
    }
}
 
class Cat extends Animal{
    public void shout(){
        System.out.println("喵喵喵喵！");
    }
}
 
public class TestAnimal {
    public static void main(String[] args){
        Animal a1 = new Cat(); // 向上自动转型
        animalCry(a1);
        Animal a2 = new Dog();
        animalCry(a2);
 
        Dog dog = (Dog)a2;  // 向下强制转型(显示转换)
        dog.seeDoor();
    }
 
    // 有了多态，只需要让增加的这个类继承Animal类就可以了
    static void animalCry(Animal a) {
        a.shout();
    }
    /* 如果没有多态，就需要写很多重载的方法。
     * 每增加一种动物，就需要重载一种动物的喊叫方法，非常麻烦：
    static void animalCry(Dog d) {
        d.shout();
    }
    static void animalCry(Cat c) {
        c.shout();
    }*/
}
 
/*
喵喵喵喵！
旺旺旺！
看门中....*/
看下面这个例子：  

代码14-25行利用instanceof操作符对object进行判断，判断其是否为Circle类或Rectangle类的实例，如果检测是，则用显示转换将其转换为相关类型对象（一定要转换奥！），转换完成后就可以使用访问操作符对类中的方法进行调用。

学到这里的时候我对实例和对象的定义产生了混淆，现总结如下：

## （4）实例和对象的区别
通俗一点说就是对象和实例的关系就好比「人」和「儿女」，人表达的是一种类别，而儿女则表达的是一种关系。

简单来说就是，实例可以有很多种，而对象是众多实例中的其中一个具体例子。

# 四、动态绑定
## （1）声明类型和实际类型
先看下面这个例子：

Object o = new GGGG();
System.out.println(o.toString());
这时o调用的toString究竟是谁的？在语句Object o = new GGGG();中，Object是声明类型，而GGGG是实际类型，o调用的是实际类型中的方法，即调用的是GGGG中的toString方法。

## （2）动态绑定
动态绑定指的是：方法可以在沿着继承链的多个类中实现，对象o如果是多个类的实例，那么如果o调用一个方法p，那么JVM会依次在众多类中依次寻找该方法，直到找到该方法为止。下面这段代码就展示了继承链：

public class Test1 {
	public static void main(String[] args)
	{
		m(new GraduateStudent());    //创建GraduateStudent的实例并运用m方法
		m(new Student());            //创建Student的实例并运用m方法
		m(new Person());             //创建Person的实例并运用m方法
		m(new Object());             //创建Object的实例并运用m方法
	}
	public static void m(Object o)       //m方法，输出对象的toString表示
	{
		System.out.println(o.toString());
	}
}
class GraduateStudent extends Student        //GraduateStudent类继承Student类
{
	
}
class Student extends Person                 //Student类继承Person类
{
	@Override
	public String toString()
	{
		return "Student";
	}
}
class Person extends Object                  //Person类继承Object类
{
	@Override
	public String toString()
	{
		return "Person";
	}
}
## （3）方法匹配和方法绑定
方法匹配：调用方法时，变量的声明类型决定了编译时匹配哪个类的哪个方法
方法绑定：即上面提到的动态绑定，是在Student Person运行时实现的。运行时查看这个方法在实际类型中是不是重写了：yes-->调用实际类型中的这个方法；no-->就还用声明类型中的这个方法。
示例代码：

class A {
    public void show(A obj){
        System.out.println("A and A");
    }
 
    //重载
    public void show(D obj){
        System.out.println("A and D");
    }
}
 
//链状继承
class B extends A {
    public void show(A obj){
        System.out.println("B and A");
    } //重写
 
    public void show(B obj){
        System.out.println("B and B");
    }
 
    public void show(E obj){
        System.out.println("B and E");
    }
}
 
class C extends B{
}
 
class D extends B{
}
 
class E{
}
public class Test2 {
    public static void main(String[] args){
        A a1 = new A();
        A a2 = new B();
 
        B b = new B();
        C c = new C();
        D d = new D();
        E e = new E();
 
        /*函数参数的自动类型转换（子类自动转父类）*/
        a1.show(b);  /*A and A*/
        a1.show(c);  /*A and A*/
        a1.show(d);  /*A and D*/
 
        /*(方法匹配和方法绑定)
        先是在类A里面进行方法匹配，运行时再到类B里面看看有没有重写
        发现只有一个public void show(A obj)重写了，那么就只有
        这个方法是调用B的方法，其他都调用A的方法
        * */
        a2.show(b);  /*B and A*/
        a2.show(c);  /*B and A*/
        a2.show(d);  /*A and D*/
 
        //a2.show(e);//这句编译不过
        /*原因:编译阶段，方法匹配不成功。因为A里面没有show(E obj)的
        而且也没有show(O obj)的。（其中O是E的父类型）
        转回实际类型：B a2 = (B)a2; a2.show(e)就没问题了*/
 
        b.show(b);  /*B and B*/
        b.show(c);  /*B and B*/
 
        /*由于继承的原因，其实A中的非private方法都被B继承了
        所以B自己找不到方法的，会去父类A里找*/
        b.show(d);  /*A and D*/
        b.show(e);  /*B and E*/
    }
}
 ## （4）一些示例代码题：


这道题输出的第一句是new A的结果，后两句是new B的结果。之所以new B会输出两句是因为B是A的子类，在调用子类B时会先进入父类A中，后返回子类B中。而因为setI方法在子类B中重写了，所以最终使用的是B中的setI，因此i的值是60。 

 输出结果：i from A is 40
                   i from A is 60
                   i from B is 60​​​​​​​



主要是右边部分的第二个输出，输出person。

因为被private修饰的方法只能在所在类中被调用，因此在执行new Student().printPerson()语句时，Student类中的getInfo方法不能被Person类中的printPerson调用，所以getInfo方法只能调用Person类中的。（注意printPerson是public类型，所以它能够在主类中被调用）

 输出结果：   左边：Person  Student
                      右边：Person  Person

# 五、存储不限定个数对象：ArrayList类
ArrayList类可存储不限定个数的对象。

语句：ArrayList<类型>  变量名 = new ArrayList<类型>（）;

 ArrayList类和数组相比有一个好处是比数组更加灵活：

ArrayList类能够随意插入、添加和删除一个元素，而数组却难以实现；
在创建时，ArrayList不需要提前给定大小，而创建数组时必须给定数组的大小。
## 注意1：
如果我们想要创建一个用于存储整数的ArrayList，可不能使用ArrayList<int>  listOfInteger = new ArrayList<>();语句来创建。

切记：ArrayList中的元素必须是一种对象。
想要创建一个存储Interger对象的ArrayList，我们应该：

ArrayList<Integer>  listOfInteger = new ArrayList<>();

## 注意2： 


 这段代码除了第四行的参数不匹配之外，还有一个很容易错的点在于：倒数第一行和第二行的set和get语句。运行代码会报错：

Index 3 out of bounds for length 2（索引 3 超出长度 2 的范围）
从报错信息可知，编译器找不到下标为3的元素。当我们把3改为1时，程序就可以正常运行了，因为前面使用add方法添加了两个元素。所以得知在使用ArrayList类中的方法时，set和get方法的使用需要在具体下标得到初始化时才能修改、添加或删除。

# 六、保护数据域：protected关键字
前面我们接触了public、private关键字，protected关键字和它们同性质：

protected关键字保护该类数据域和成员只能被其任意包中的子类或同一个包中的类访问。
还有一个default关键字：使得同一个包中的类可以访问它，不同包的类无法访问。
注意：private和protected关键字只能用于类的成员，而pubilc可以用于成员也可以用于类。

# 七、 防止继承和重写：final修饰符
被final修饰的方法和类都无法继承
之前有学习过用final修饰数据域，被final修饰的数据域是一个常数；而当final修饰一个类时，它表示这个类已经是最终类而无法作为父类，自然也不能被子类重写。
