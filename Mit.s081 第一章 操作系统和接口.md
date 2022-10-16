# Mit.s081 第一章 操作系统和接口

### 前言

终于开这个了。

MIT6.S081主要是通过实现部分内核功能来学习设计和实现操作系统，使用vx6作为对象。

**每个运行的程序是一个进程。进程需要调用一个内核服务时就会调用一个系统调用。**

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
| `char *sbrk(int n)`                     | 按n 字节增长进程的内存。返回新内存的开始                    |
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

### 1、进程和内存

