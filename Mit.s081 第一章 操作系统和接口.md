# Mit.s081 第一章 操作系统和接口

### 前言

终于开这个了。

MIT6.S081主要是通过实现部分内核功能来学习设计和实现操作系统，使用vx6作为对象。

每个运行的程序是一个进程。进程需要调用一个内核服务时就会调用一个系统调用。

xv6的全部系统调用如下（返回0表示无误，-1表示出错）：

| **系统调用**                            | **描述**                                                    |
| --------------------------------------- | ----------------------------------------------------------- |
| `int fork()`                            | 创建一个进程，返回子进程的PID                               |
| `int exit(int status)`                  | 终止当前进程，并将状态报告给wait()函数。无返回              |
| `int wait(int *status)`                 | 等待一个子进程退出; 将退出状态存入*status; 返回子进程PID。  |
| `int kill(int pid)`                     | 终止对应PID的进程，返回0，或返回-1表示错误                  |
| `int getpid()`                          | 返回当前进程的PID                                           |
| `int sleep(int n)`                      | 暂停n个时钟节拍                                             |
| `int exec(char *file, char *argv[])`    | 加载一个文件并使用参数执行它; 只有在出错时才返回            |
| `char *sbrk(int n)`                     | 按n 字节增长进程的内存。返回新内存的开始位置                |
| `int open(char *file, int flags)`       | 打开一个文件；flags表示read/write；返回一个fd（文件描述符） |
| `int write(int fd, char *buf, int n)`   | 从buf 写n 个字节到文件描述符fd; 返回n                       |
| `int read(int fd, char *buf, int n)`    | 将n 个字节读入buf；返回读取的字节数；如果文件结束，返回0    |
| `int close(int fd)`                     | 释放打开的文件fd                                            |
| `int dup(int fd)`                       | 返回一个新的文件描述符，指向与fd 相同的文件                 |
| `int pipe(int p[])`                     | 创建一个管道，把read/write文件描述符放在p[0]和p[1]中        |
| `int chdir(char *dir)`                  | 改变当前的工作目录                                          |
| `int mkdir(char *dir)`                  | 创建一个新目录                                              |
| `int mknod(char *file, int, int)`       | 创建一个设备文件                                            |
| `int fstat(int fd, struct stat *st)`    | 将打开文件fd的信息放入*st                                   |
| `int stat(char *file, struct stat *st)` | 将指定名称的文件信息放入*st                                 |
| `int link(char *file1, char *file2)`    | 为文件file1创建另一个名称(file2)                            |
| `int unlink(char *file)`                | 删除一个文件                                                |



# 一、操作系统接口

### 1、进程和内存

- ***关于进程***

xv6中的shell主要结构：主循环使用***getcmd***函数从用户输入中读取一行，然后调用***fork***创建一个shell进程副本。此时父进程调用***wait***，子进程***exec***执行命令。总共就用了这4个系统调用。

> **例如：**
>
> **我向shell输入echo hello，此时将会调用以echo hello为参数的exec函数。如果exec成功，那么子进程将从echo执行命令（不是runcmd）。某刻echo会调用exit退出，导致父进程从main中的wait返回。**

- ***关于内存***

xv6**隐式分配**大部分用户空间内存：fork分配父内存子副本所需内存；exec分配保存可执行文件的内存。

如需更多内存的进程（如malloc），可以用***sbrk(n)*将其数据内存增加n个字节**（sbrk返回新内存位置）



------



### 2、I/O和文件描述符

- ***文件描述符***

文件描述符是一个小整数，表示进程可以读取或写入的由内核管理的对象（文件）。xv6中的文件描述符作为**每个进程表的索引**。

默认文件描述符：***进程从文件描述符0读取（标准输入），将输出写入文件描述符1（标准输出），并将错误消息写入文件描述符2（标准错误）***

- ***I/O***

**输入和输出**



> ***read( fd, buf, n )*：从文件描述符fd读取最多n个字节复制到buf，返回读取字节数**
>
> ***write( fd, buf, n )*：将buf中的n字节写入文件描述符fd，返回写入字节数 （错误时才会写入比n小的字节数）**



***close***调用会**释放一个文件描述符**。下面是cat < input.txt实现的简化代码：

```C
    // shell: cat < input.txt 
    // 创建一个新文件
    char* argv[2];
    argv[0] = "cat";
    argv[1] = 0;
    if(fork() == 0) {
        close(0);
        open("input.txt", O_RDONLY);
        exec("cat", argv);
    }
```

在使用close关闭子进程文件描述符0之后，下面的open保证使用新打开的input.txt

***open***第二个参数用于控制打开的操作：

| **宏定义** | **功能说明**             |
| ---------- | ------------------------ |
| `O_RDONLY` | 只读                     |
| `O_WRONLY` | 只写                     |
| `O_RDWR`   | 可读可写                 |
| `O_CREATE` | 如果文件不存在则创建文件 |
| `O_TRUNC`  | 将文件截断为零长度       |



***fork***每个基础**文件偏移量在父子文件之间共享**，比如下面程序：

```C
    // 父子偏移量共享 --> hello world
    if(fork == 0) {
        write(1, "hello", 6);
        exit(0);
    }else {
        wait(0);
        write(1, "world", 6);
    }
```

最终写入文件描述符1的文件的是“hello world”。父进程的写操作在子进程停止写入的位置进行



***dup***调用**复制一个现有的文件描述符**，返回一个指向相同文件的新描述符。新旧两个描述符也是共享同一个偏移量

```c
    // 非父子共享偏移量的另一种方法 -- dup
    fd = dup(1);
    write(1, "hello", 6);
    write(fd, "world", 6);
```

