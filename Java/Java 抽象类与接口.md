# 目录

一、抽象类（abstract）

        1、定义

        2、也不是完全不能new

        3、意义

        4、示例 

二、接口（interface）

        1、定义

        2、示例

三、比较：Comparable接口

四、克隆：Cloneable接口

        深复制和浅复制

五、接口和抽象类对比

        1、相同点

        2、不同点

                （1）构造方法

                （2）继承

                （3）根类

        3、相比之下

# 一、抽象类（abstract）
## 1、定义
方法头或者类头使用abstract关键字即表示为抽象类或抽象方法。抽象类不能使用new实例化对象，而抽象类中又包含抽象方法（会在子类中实现）。当一个类中包含抽象方法时，这个类也必须用abstract声明为抽象类。

抽象类的构造方法定义为protected，因为它只被子类使用。
## 2、也不是完全不能new
虽然不能使用new操作符对一个抽象类创建一个实例，但抽象类可以用作一种数据类型（目前java能满足这点的只有数组）。下面的语句是正确的：

GeometricObject[ ]  a  =  new  GeometricObject[10];

但如果是 GeometricObject a = new GeometricObject;就不可以。

## 3、意义
>“疑惑：这种抽象类和抽象方法存在的意义是什么？”

就好比我现在创建一个名为Object的抽象类用于对几何图形进行建模，其中包含“计算面积”和“计算周长”这两个方法。显而易见，不同的图形计算面积和周长的方法是不同的，这两个方法就需要在Object类的子类中进行重写。因此需要将其设置为抽象方法。

将其定义为抽象方法而不是直接在子类中创建一个新的方法的原因是：有了更好的统一和通用性。比如后续我们可以在主类中调用一些方法去比较两个子类的“面积”是否相同之类的，因为它们都来自于父类，所以可以“站在同一台阶上”。

## 4、示例 
展示代码。这边我设置了一个抽象类Animal，类中包含了抽象方法eat。eat在Animal的子类Cat类和Dog类中进行重写：

```
abstract class Animal{
    public void sleep(){
        System.out.println("我趴着睡");
    }
    public abstract void eat();
}
 
class Dog extends Animal{
    public Dog(){
        super();
    }
 
    @Override
    public void eat(){
        System.out.println("我实现了父类方法，狗吃肉");
    }
}
 
class Cat extends Animal{
    public Cat(){
        super();
    }
 
    @Override
    public void eat(){
        System.out.println("我实现了父类方法，猫吃鱼");
    }
}
public class TestAnimal {
    public static void main(String[] args){
        Animal a1 = new Dog();
        a1.sleep();
        a1.eat();
        System.out.println("----------------------");
        Animal a2 = new Cat();
        a2.sleep();
        a2.eat();
    }
}
```

所以可以看出，把类抽象化能够增强类的通用性和子类的聚合性。

# 二、接口（interface）
## 1、定义
接口用于为对象定义共同的操作，可以把接口（interface）看作是一个特殊的类（class）。和抽象类一样，接口也不可以用new实例化对象，但可以将接口作为变量的数据类型（数组）。

implements关键字用于让对象的所属类实现本接口
## 2、示例
下面设计一个接口来判断对象是否可食用：

```
interface Edible {//设计一个Edible接口来明确一个对象是否是可食用的
    public abstract String howToEat();//接口中方法格式固定：public abstract
}
 
abstract class Animal2 {//抽象类Animal
    public abstract String sound();//抽象方法（动物叫声）
}
 
class Chicken extends Animal2 implements Edible {//鸡肉类继承抽象的Animal类并实现Edible接口以表明小鸡是可食用的
    @Override
    public String sound() {
        return "Chicken:cock-a-doodle-doo";
    }//重写sound方法
 
    @Override
    public String howToEat() {
        return "Chicken:Fry it";
    }//重写howToEat方法
}
class Tiger extends Animal2{//老虎类继承抽象的Animal类
    @Override
    public  String sound(){
        return "Tiger:RROOAARR";
    }//重写sound方法
}
abstract class Fruit implements Edible{//抽象类Fruit实现接口Edible，因为它不实现howToEat方法，所以Fruit类必须是抽象的
 
}
class Apple extends Fruit {//苹果类继承Fruit类（Fruit的子类必须实现howToEat方法）
    @Override
    public String howToEat(){
        return "Make apple cider";
    }//重写howToEat方法
}
class Orange extends Fruit{//橘子类继承水果类（Fruit的子类必须实现howToEat方法）
    @Override
    public String howToEat(){
        return "Orange:Make orange juice";
    }//重写howToEat方法
}
 
public class TestEdible {
    public static void main(String[] args) {
        Object[] objects={new Chicken(),new Tiger(),new Orange()};//创建对象数组
        for (int i=0;i<objects.length;i++){
            if (objects[i]instanceof Edible)
                System.out.println(((Edible)objects[i]).howToEat());
            if (objects[i]instanceof Animal2)
                System.out.println(((Animal2)objects[i]).sound());
        }
    }
}
```

程序定义了Edible接口来定义可食用对象的共同行为：howToEat方法。然后就是构造一系列抽象类再衍生出各种子类（比如：Edible是Chicken和Fruit的父类，Animal是Chicken和Tiger的父类，Fruit是Orange和Apple的父类），它们的继承关系如下：

![avatar](https://img-blog.csdnimg.cn/3c5748bb88da4af5a67e1fb365eb4124.png)

同时注意声明接口的格式。接口用于声明类与类之间相同的行为，所以只需声明，无需构造：

![avatar](https://img-blog.csdnimg.cn/b5c2db84283b41bcae1e8c72a5173196.png)

 上面这六个接口只有d接口是正确的（无需花括号）。

# 三、比较：Comparable接口
Comparable接口通过实现可比较的接口，可将类的对象传递给需要比较的方法。它是一个泛型接口，实现接口时泛型类型E可以被替换为任意类型：

```
public interface Comparable <E>{ //接口定义
    public int compareTo(E o);
}
```

如果对象是Comparabale接口类型实例，那么java.util.Arrays.sort(Object[ ])方法就可以搭配接口中的compareTo方法组合使用，对数组中的对象进行比较和排序。

>（Byte、Short、Integer、Long、Float、Double、Character、BigInteger、BigDecimal、Calendar、String、Date类都实现了Comparable接口，可以直接调用compareTo方法）

没有实现Comparable接口的类，就不能调用sort方法。但没关系我们可以手动定义它，如Rectangle类：

```
import java.awt.*;
 
class Rectangle {// 一个Rectangle类
    double width, length;
    double area;
 
    Rectangle(double w, double len) {
        width = w;
        length = len;
    }
 
    public double getArea() {
        area = width * length;
        return area;
    }
}
 
//自定义一个实现Comparable接口的Rectangle类的子类（可比较）
class ComparableRectangle extends Rectangle implements Comparable<ComparableRectangle>{
    public ComparableRectangle(double width, double height){
        super(width,height);
    }
 
    @Override
    public int compareTo(ComparableRectangle o){
        if(getArea() > o.getArea()){
            return 1;
        }else if(getArea() < o.getArea()){
            return -1;
        }else return 0;
    }
 
    @Override
    public String toString(){
        return super.toString() + " Area: " + getArea();
    }
}
 
public class SortRectangles {
    public static void main(String[] args){
        ComparableRectangle[] rectangles = {
                new ComparableRectangle(3.4,5.4),
                new ComparableRectangle(13.24,55.4),
                new ComparableRectangle(7.4,35.4),
                new ComparableRectangle(1.4,25.4)
        };
        java.util.Arrays.sort(rectangles); //就能使用sort方法排序了 
        for(Rectangle rectangle: rectangles){
            System.out.print(rectangle + " ");
            System.out.println();
        }
    }
}
```

最终结果的面积是由小到大按序输出的。 

# 四、克隆：Cloneable接口
Cloneable接口指定一个对象可以被克隆。但它有些特殊，它是一个空接口，称为标记接口：

```
public interface Cloneable{
 
}
```

标记接口里没有变量也没有方法，它代表一个类拥有某些希望具有的特征。Cloneable接口可以搭配Object类中的clone()方法使用。

（Date、Calendar、ArrayList类中都实现了Cloneable接口，可以直接调用Object接口） ：

```
Calendar a = new GregorianCalendar(2013,2,1);
Calendar b = a; // 复制a的引用，此时a b指向相同的对象
Calendar c = (Calendar)a.clone(); // 克隆a的对象，此时a c是内容相同的不同对象
```

注意看上面的代码，第二行的复制引用和第三行的克隆对象虽然将它们的内容暂时化为一致，但后期结果是不同的：复制引用让a和b指向同个对象，后期无论是对a动手还是b动手，另外一方的内容都会随着改变方的改变而变化（动态统一）；而a和c只是暂时内容相同了，后期变化两不干涉。

下面自定义了一个实现Cloneable和Comparable接口的类House： 

```
// 自定义House类
class House implements Cloneable,Comparable<House> { //可以实现多个接口
    private int id;
    private double area;
    private java.util.Date whenBuild;
 
    public House(int id, double area){
        this.id = id;
        this.area = area;
        whenBuild = new java.util.Date();
    }
 
    public int getId() {
        return id;
    }
    public double getArea() {
        return area;
    }
    public java.util.Date getWhenBuild() {
        return whenBuild;
    }
 
    //重写clone方法
    @Override
    public Object clone() throws CloneNotSupportedException {
            return super.clone();
    }
    //
 
    @Override
    public int compareTo(House o){
        if(area > o.area) return 1;
        else if(area < o.area) return -1;
        else return 0;
    }
}
 
public class TestHouse {
    public static void main(String[] args) throws CloneNotSupportedException {
        House house1 = new House(1, 1750.50);
        House house2 = (House)house1.clone();
    }
}
```

- ## 深复制和浅复制
上面定义的House类中的数据域有基本类型（id、area）也有对象（whenBuilt）。不同类型的数据域用上面定义的clone方法克隆时效果也不一样：数据域是基本类型值时复制的是它的值；数据域是对象类型时复制的是对象的引用。

所以此时在主类中克隆，house1.whenBuilt == house2.whenBuilt为true，它们指向的是同一个地方，并没有独立开，这称为浅复制。

如果想要达到完全分开的克隆效果，实现深复制，则应该将House类中的clone方法重写为：

```
@Override
public Object clone() throws CloneNotSupportedException {
    House houseClone = (House)super.clone();
    //对whenBuild进行深复制
    houseClone.whenBuild = (java.util.Date)(whenBuild.clone());
    return houseClone;
}
```

使用这个方法后house1.whenBuilt == house2.whenBuilt为false，独立开了。

# 五、接口和抽象类对比
## 1、相同点
都用来指定多个对象的共同特征；
都不能用new操作符实例化，但可以作为变量的数据类型（数组）。
## 2、不同点
### （1）构造方法
抽象类的子类通过构造方法链调用构造方法，接口没有构造方法。

### （2）继承
接口可以继承其他接口但不能继承类，而一个类可以继承一个父类的同时实现多个接口。

和类的继承一样，接口的继承也使用extends关键字：

```
public interface NewInterface extends Interface1,Interface2{
    ...
}
```

一个实现NewInterface的类必须实现在Interface1、Interface2中定义的抽象方法。

### （3）根类
所有的类共享一个根类Object，而接口没有共同的根。

## 3、相比之下
虽说抽象类和接口都用来指定多个对象的共同特征，但相比之下接口比类更灵活，因为接口不用使所有东西都属于同一个类型的类。
