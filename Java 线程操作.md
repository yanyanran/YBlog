目录

一、进程与线程

二、Java线程状态

三、线程操作：Thread类 

1、启动线程：start()

2、等待线程：join()

3、中断线程：

（1）方法一：interrupt() 

（2）方法二：设置running标志位

        关键字volatile：定义线程共享变量

4、守护线程：setDaemon(true)

（1）作用

（2）创建

（3）使用 

5、线程同步：锁

（1）同步锁：synchronized关键字

                使用

                关于锁对象的选择

6、bugbugbug：死锁

（1）概念

（2）避免死锁

（3）测试练习：消除死锁

一、进程与线程
计算机中，把一个任务（浏览器、播放器或是其他）称为进程，而进程的子任务则称为线程

多进程模式（每个进程只有一个线程）：
┌──────────┐ ┌──────────┐ ┌──────────┐
│Process   │ │Process   │ │Process   │
│┌────────┐│ │┌────────┐│ │┌────────┐│
││ Thread ││ ││ Thread ││ ││ Thread ││
│└────────┘│ │└────────┘│ │└────────┘│
└──────────┘ └──────────┘ └──────────┘
多线程模式（一个进程有多个线程）：
┌────────────────────┐
│Process             │
│┌────────┐┌────────┐│
││ Thread ││ Thread ││
│└────────┘└────────┘│
│┌────────┐┌────────┐│
││ Thread ││ Thread ││
│└────────┘└────────┘│
└────────────────────┘
多进程＋多线程模式（复杂度最高）：
┌──────────┐┌──────────┐┌──────────┐
│Process   ││Process   ││Process   │
│┌────────┐││┌────────┐││┌────────┐│
││ Thread ││││ Thread ││││ Thread ││
│└────────┘││└────────┘││└────────┘│
│┌────────┐││┌────────┐││┌────────┐│
││ Thread ││││ Thread ││││ Thread ││
│└────────┘││└────────┘││└────────┘│
└──────────┘└──────────┘└──────────┘
多线程 VS 多进程：

创建进程的代价要比线程大
进程间通信比线程间通信慢（因为线程间通信就是读写同一个变量，速度很快）
多进程状态下比多进程状态稳定（多进程下一个进程崩溃不会导致全盘崩溃，但多线程中的一个线程崩溃会导致整个多线程崩溃）
多线程模型是Java程序最基本的并发模型
二、Java线程状态
New：新创建的线程，尚未执行；
Runnable：运行中的线程，正在执行run()方法的Java代码；
Blocked：运行中的线程，因为某些操作被阻塞而挂起；
Waiting：运行中的线程，因为某些操作在等待中；
Timed Waiting：运行中的线程，因为执行sleep()方法正在计时等待；
Terminated：线程已终止，因为run()方法执行完毕。
其中蓝色的四个状态都是中间进行状态，它们的最终状态都是Terminated 

三、线程操作：Thread类 
下面创建一个Thread类对象启动一个新线程打印语句，线程的执行代码写在run()方法中。

1、启动线程：start()
public class Main {
    public static void main(String[] args) {
        Thread t = new MyThread();
        t.start(); // 启动新线程
    }
}
 
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("start new thread!");
    }
}
输出结果即为println语句内的内容。那么问题来了，使用线程打印的语句和直接在main方法执行打印语句有区别吗？

一定有区别。

public class ThreadCode {
    public static void main(String[] args){
        System.out.println("main start");
        Thread t = new Thread(){
            public void run(){
                System.out.println("thread run");
// 可以调用Thread.sleep()方法（ms为单位）强迫让当前线程暂停一段时间
                /*
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {}
                 */
                System.out.println("thread end");
            }
        };
        t.start();
        /*
           try {
            Thread.sleep(20);
        } catch (InterruptedException e) {}
        */
        System.out.println("main end");
    }
}
在上面这段代码中，主线程是main函数中的前面两行和最后两行。首先打印main start，然后创建Thread对象，接着调用start()启动新线程。当start()方法被调用时就创建了一个新线程（假设为t），接下来继续打印main语句，而t线程在main线程执行的时候将会并发进行，打印thread run和thread end语句。

注意这个并发执行线程的动作：此时主线程和t线程并发进行，但两个线程中的打印语句谁先谁后却是不确定的。由于操作系统的调度，程序本身无法确定线程的调度顺序。（可以调用Thread.setPriority(int n)方法设定线程优先级。）

2、等待线程：join()
public class ThreadCode {
    public static void main(String[] args) throws InterruptedException{
        Thread t = new Thread(){
            public void run(){
                System.out.println("hello");
            }
        };
        System.out.println("start");
        t.start();
        t.join();
        System.out.println("end");
    }
}
main线程在启动t线程后可以通过t.join()等待t线程结束后再继续运行：

* 无join语句输出：start
                             end
                             hello

* 有join语句输出：start
                             hello
                             end

3、中断线程
（1）方法一：interrupt() 
该方法用于中断线程：

public class ThreadCode {
    public static void main(String[] args) throws InterruptedException{
        Thread t = new MyThread();
        t.start();
        Thread.sleep(1000);
        t.interrupt(); // 中断
        t.join();
        System.out.println("end");
    }
}
 
class MyThread extends Thread{
    public void run(){
        Thread hello = new HelloThread();
        hello.start();
        try{
            hello.join();
        }catch(InterruptedException e){
            System.out.println("interrupted!");
        }
        hello.interrupt(); // 中断 如果去掉这一行代码，发现hello线程仍会继续运行
    }
}
 
class HelloThread extends Thread{
    public void run() {
        int n = 0;
        while(!isInterrupted()){
            n++;
            System.out.println(n + "hello");
            try{
                Thread.sleep(100);
            }catch(InterruptedException e){
                break;
            }
        }
    }
}
 
/*
输出：
1hello
2hello
3hello
4hello
5hello
6hello
7hello
8hello
9hello
10hello
interrupted!
end
*/
（2）方法二：设置running标志位
通过将t.running设置为false来使线程结束：

public class ThreadCode {
    public static void main(String[] args) throws InterruptedException{
        HelloThread t = new HelloThread();
        t.start();
        Thread.sleep(1);
        t.running = false;
    }
}
 
class HelloThread extends Thread{
    public volatile boolean running = true;
    public void run() {
        int n = 0;
        while(running){
            n++;
            System.out.println(n + "hello");
            }
        System.out.println("end");
        }
    }
 
/*
输出：
1hello
end
*/
同时注意标志位boolean running是一个线程间共享的变量，线程间共享的变量需要使用volatile关键字标记。

关键字volatile：定义线程共享变量
1、每次访问变量时，总获取主内存最新值

2、每次修改变量后，立刻回写到主内存

总之就是实时同步更新 

4、守护线程：setDaemon(true)
（1）作用
JVM即为Java虚拟机。只有当前程序中所有线程都结束时，JVM才会退出，进程结束。但凡有一个线程没有退出，JVM进程都不会结束。

但在线程中，有一种线程的目的是无限循环，如果该线程不结束，JVM进程就结束不了。此时就需要用到守护线程。

（2）创建
创建守护线程的方法和创建普通线程一样，只需在start方法前调用setDaemon(true)将该线程标记为守护线程：

Thread t = new MyThread();
t.setDaemon(true); //标记为守护线程
t.start();
（3）使用 
下面在主线程中创建一个自身循环的非守护线程，并添加一个钩子线程来监控JVM是否退出：

import java.util.concurrent.TimeUnit;
 
public class ThreadCode {
    public static void main(String[] args) throws InterruptedException{
        // 添加钩子线程, 用来监听 JVM 退出
        Runtime.getRuntime().addShutdownHook(new Thread(()->System.out.println("The JVM exit success!!")));
 
        // 主线程中new一个非守护线程
        Thread t = new Thread(()->{
            while(true) {
                try{
                    TimeUnit.SECONDS.sleep(1);
                    System.out.println("still running...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
 
        t.setDaemon(true);  //标记为守护线程
        t.start();
        TimeUnit.SECONDS.sleep(2);
        System.out.println("The main thread ready to exit...");
    }
}
当没有将线程标记为守护进程时，运行程序：



发现进程无法结束，JVM无法退出。此时将线程声明为守护线程后： 



 进程成功结束，JVM退出。

5、线程同步：锁
 多线程模式下会出现一个问题：如果多个线程同时读写共享变量，数据会不一致。

/*多线程共享变量不加锁会出问题*/
public class ThreadLock {
    public static void main(String[] args) throws Exception {
        var add = new AddThread();
        var edc = new EdcThrad();
        add.start();
        edc.start();
        add.join();
        edc.join();
        System.out.println(Counter.count);
    }
}
 
class Counter{
    public static int count = 0;
}
 
class AddThread extends Thread {
    public void run(){
        for(int i = 0; i < 1000;i++){
            Counter.count += 1;
        }
    }
}
 
class EdcThrad extends Thread {
    public void run() {
        for(int i = 0; i < 1000;i++){
            Counter.count -= 1;
        }
    }
}
上面这个程序的两个线程同时对一个int变量操作，一个加一个减，相同的次数按理来说最终结果应该是0，但实际输出结果每次都不一样（每次都是奇怪的数字反正不是0）。

是因为：对变量进行读取和写入都必须是原子操作（原子操作：指不能被中断的一个或一系列操作）。因为两个线程的操作如果同时穿插进行，会引起逻辑错误。

所以在多线程模型下，需要对共享变量原子操作执行：某个线程在执行时，其他线程必须等待。这时就涉及到一个锁的概念。

（1）同步锁：synchronized关键字
使用
synchronized关键字能够保证代码块在任意时刻最多只有一个线程能执行。运用这个关键字改写上面的问题代码：

public class ThreadLock {
    public static void main(String[] args) throws Exception {
        var add = new AddThread();
        var edc = new EdcThread();
        add.start();
        edc.start();
        add.join();
        edc.join();
        System.out.println(Counter.count);
    }
}
 
class Counter{
    public static final Object lock = new Object(); //在类中设置一个Object类的锁
    public static int count = 0;
}
 
class AddThread extends Thread {
    public void run(){
        for(int i = 0; i < 1000;i++){
            synchronized(Counter.lock){ //获取锁
                Counter.count += 1;
            } // 释放锁
        }
    }
}
 
class EdcThread extends Thread {
    public void run() {
        for(int i = 0; i < 1000;i++){
            synchronized(Counter.lock){ // 获取锁
                Counter.count -= 1;
            } // 释放锁
        }
    }
}
 
/*
输出：0
*/
在执行各自代码中的synchronized时，必须获得锁才能进入代码块中，执行结束后语句块结束会自动释放锁，这样就能保证对变量的读写是依次进行的。

关于锁对象的选择
因为锁限制了线程的并发进行，所以如果用错锁会给程序运行效率拖后腿。

下面这个程序的4个线程对两个共享变量操作，但使用的锁都是一个对象锁： 

public class Lock {
    public static void main(String[] args) throws Exception {
        var ts = new Thread[] {
                new AddStudentThread(), new DecStudentThread(), new AddTeacherThread(), new DecTeacherThread()
        };
        for (var t : ts) {
            t.start();
        }
        for (var t : ts) {
            t.join();
        }
        System.out.println(Counter1.studentCount);
        System.out.println(Counter1.teacherCount);
    }
}
 
class Counter1 {
    public static final Object lock = new Object();
    public static int studentCount = 0;
    public static int teacherCount = 0;
}
 
class AddStudentThread extends Thread { // 线程1
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter1.lock) {
                Counter1.studentCount += 1;
            }
        }
    }
}
 
class DecStudentThread extends Thread { // 线程2
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter1.lock) {
                Counter1.studentCount -= 1;
            }
        }
    }
}
 
class AddTeacherThread extends Thread { // 线程3
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter1.lock) {
                Counter1.teacherCount += 1;
            }
        }
    }
}
 
class DecTeacherThread extends Thread { // 线程4
    public void run() {
        for (int i=0; i<10000; i++) {
            synchronized(Counter1.lock) {
                Counter1.teacherCount -= 1;
            }
        }
    }
}
这就造成了本可以并发执行的Counter.studentCount += 1和Counter.teacherCount += 1现在无法并发执行，效率大大降低。锁只应该出现在有竞争的线程之间，从而提高效率。

同时注意格式。以下两种加锁写法等价：

public void add(int n) {
    synchronized(this) { // 锁住this
        count += n;
    } // 解锁
}
public synchronized void add(int n) { // 锁住this
    count += n;
} // 解锁
6、bugbugbug：死锁
（1）概念
一个线程可以获取一个锁后再继续获取另一个锁。在获取多个锁的时候，不同线程获取多个不同对象的锁可能会导致死锁：

public void add(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value += m;
        synchronized(lockB) { // 获得lockB的锁
            this.another += m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}
 
public void dec(int m) {
    synchronized(lockB) { // 获得lockB的锁
        this.another -= m;
        synchronized(lockA) { // 获得lockA的锁
            this.value -= m;
        } // 释放lockA的锁
    } // 释放lockB的锁
}
上面的代码线程1和线程2如果分别执行add()和dec()方法：

线程1：进入add，获得lockA

线程2：进入dec，获得lockB

---->随后

线程1：准备获得lockB，失败，等待中...

线程2：准备获得lockA，失败，等待中...

这时候的情况是：两个线程各自持有不同的锁，然后试图获取对方手里的锁，结果就是双方无限等待下去，这就是死锁。 

（2）避免死锁
线程获取锁的顺序要一致。上面的代码应该严格按照先获取lockA，再获取lockB的顺序。改写上面的dec()方法：

public void dec(int m) {
    synchronized(lockA) { // 获得lockA的锁
        this.value -= m;
        synchronized(lockB) { // 获得lockB的锁
            this.another -= m;
        } // 释放lockB的锁
    } // 释放lockA的锁
}
（3）测试练习：消除死锁
// 死锁程序
class DeathLock {
 
    static final Object LOCK_A = new Object();
    static final Object LOCK_B = new Object();
 
    public static void main(String[] args) {
        new Thread1().start();
        new Thread2().start();
    }
 
    static void sleep1s() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
 
class Thread1 extends Thread {
 
    public void run() {
        System.out.println("Thread-1: try get lock A...");
        synchronized (DeathLock.LOCK_A) {
            System.out.println("Thread-1: lock A got.");
            DeathLock.sleep1s();
            System.out.println("Thread-1: try get lock B...");
            synchronized (DeathLock.LOCK_B) {
                System.out.println("Thread-1: lock B got.");
                DeathLock.sleep1s();
            }
            System.out.println("Thread-1: lock B released.");
        }
        System.out.println("Thread-1: lock A released.");
    }
}
 
class Thread2 extends Thread {
 
    public void run() {
        System.out.println("Thread-2: try get lock B...");
        synchronized (DeathLock.LOCK_B) {
            System.out.println("Thread-2: lock B got.");
            DeathLock.sleep1s();
            System.out.println("Thread-2: try get lock A...");
            synchronized (DeathLock.LOCK_A) {
                System.out.println("Thread-2: lock A got.");
                DeathLock.sleep1s();
            }
            System.out.println("Thread-2: lock A released.");
        }
        System.out.println("Thread-2: lock B released.");
    }
}
 
/*
死锁输出：
Thread-1: try get lock A...
Thread-1: lock A got.
Thread-2: try get lock B...
Thread-2: lock B got.
Thread-2: try get lock A...
Thread-1: try get lock B...
无法退出进程
*/
只需要把Thread2的LOCK_B和LOCK_A交换即可。

死锁解除输出。
![avatar](https://img-blog.csdnimg.cn/bf5405daa503449493fbebf202389c6f.png)
