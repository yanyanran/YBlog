目录

        1、泛型性质

        2、泛型声明

        3、通配泛型

        4、泛型类型参数的注意点

# 1、泛型性质
使用泛型能够提高代码的安全性，可以***在编译时就发现错误***而不是运行时，同时也更具宽容性，不必使用强转。

同时注意，泛型类型必须是***引用类型***而不能是基本类型。比如说想要给一个int值创建一个ArrayList对象，我们应该使用：

```
ArrayList<Integer> intList = new ArrayList<>();
 
//而不是使用ArrayList<int> intList = new ArrayList<>();
```
并且泛型存在于编译时。***一旦编译器确认该泛型类型是可以安全使用的，就会将它转换为原生类型***， 即所谓的***类型擦除***。

# 2、泛型声明
泛型可以运用在很多不同的地方，比如说泛型接口、泛型类、泛型方法等等。

```
package java.lang;
 
pubilc interface Comparable<T> {
    public int compareTo(T o)
}
//泛型接口
```
```
public class CLASS<E> {
    private java.util.ArrayList<E> list = new java.util.ArrayList<>();
 
    public int getSize() {
        return list.size();
    }
 
    public void push(E o) {
        list.add(o);
    }
}
//泛型类
```
```
public static <E> void print(E[] list) {
    .....
}
//泛型方法
```
  
>泛型方法的格式是将<E>放于static后面。
  以下创建了一个泛型方法对一个对象数组进行排序： 

```
public class GenericSort {
    public static void main(String[] args){
        Integer[] intArray = {new Integer(2),new Integer(4),new Integer(3)};
 
        Double[] doubleArray = {new Double(3.4),new Double(1.3),new Double(-22.1)};
 
        Character[] charArray = {new Character('a'),new Character('J'),new Character('r')};
 
        String[] stringArray = {"Tom", "Susan", "Kim"};
 
        sort(intArray);
        sort(doubleArray);
        sort(charArray);
        sort(stringArray);
 
        printList(intArray);
        printList(doubleArray);
        printList(charArray);
        printList(charArray);
    }
 
    //泛型方法的泛型类型继承接口Comparable，后期可以调用compareTo方法
    public static <E extends Comparable<E>> void sort(E[] list){
        E currentMin;
        int currentMinIndex;
 
        for(int i =0; i< list.length; i++){
            currentMin = list[i];
            currentMinIndex = i;
 
            for(int j = i + 1;j < list.length;j++){
                //确定元素位置
                if(currentMin.compareTo(list[j]) > 0){
                    currentMin = list[j];
                    currentMinIndex = j;
                }
            }
            if(currentMinIndex != i){
                list[currentMinIndex] = list[i];
                list[i] = currentMin;
            }
        }
    }
    public static void printList(Object[] list){
        for(int i = 0; i < list.length; i++){
            System.out.print(list[i] + " ");
        }
        System.out.println();
    }
}
```
# 3、通配泛型
通配泛型有三种形式，可以指定泛型类型的范围：

非受限通配 <?> ：它和<？extends Object>一样  
受限通配 <? extends T> ：代表T的子类型  
下限通配 <? super T> ：代表T的父类型  

# 4、泛型类型参数的注意点
不能使用泛型类型参数创建实例（ new E() ）  
不能使用泛型类型参数创建数组（ new E[10] ）  
不能在静态环境中使用类的泛型类型参数  
