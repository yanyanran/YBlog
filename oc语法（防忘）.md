# oc语法（防忘）

### 1、方法调用

=C++=>   obj.method(sub);

=Oc=>     [obj method: sub];

### 2、字符串

@

### 3、类 定义interface

@interface Classname：ParentClass Name {

​	// 类方法

​	+(returnType) func1;  ==C++=> static returnType func1();

​	// 实例方法

​	-(returnType) func2; (int) p1 andPar: (int) p2;  ==C++=> returnType func2(int p1, int p2);

}

@end

### 4、类 实现implementation

@implementation ObjectName {

​	// 私有变量和方法

​	int num;

}

// 公开方法的实现

+(returnType) func1 {

​	// …

}

-(returnType) func2: (int) p1 andPtr: (int) p2 {

​	// …

}

@end

**Interface区块实体变量默认权限为****protected****，implementation区块实体变量默认为****private**

### 5、创建对象

需要通过alloc以及init【alloc分配内存，init初始化对象】

ObjectName *my = [[MyObject alloc] init];  ==2.0=> ObjectName *my = [MyObject new];

### 6、属性

@property