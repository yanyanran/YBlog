### 能否搞懂 Hello World 程序的一生？

**1、程序是怎么变成 a.out 的？（编译原理，ELF格式，程序员的自我修养）**

a.out是“assembler output”（汇编程序输出）的缩写格式。它不是汇编程序输出而是链接器输出。

程序的创建过程是这样的：先将所有源文件连接在一起，然后进行汇编。汇编后产生的汇编程序输出就保存在a.out中。

预处理 --> 编译 --> 汇编 --> 链接

ELF格式：即可执行可链接文件格式，目前常见的Linux、 Android可执行文件、共享库（.so）、目标文件（ .o）以及Core 文件（吐核）均为此格式。

在UNIX中可执行文件用文件的第一个字节来标注，文件以16进制数7F开头，跟在后面的2～4个字节为ELF。

section是ELF文件中最小的组织单位。ELF文件参与程序的连接（建立一个程序）和程序的执行（运行一个程序）：

- 如果用于编译和连接（可重定位文件），则编译器和链接器将把ELF文件看作是节头表描述的节的集合，程序头表可选；

- 如果用于加载执行（可执行文件），则加载器将把ELF文件看作是程序头表描述的段集合，一个段可能包含多个节，节头表可选。

![img](https://img-blog.csdn.net/20180501185114680)

一个典型的a.out文件由以下7部分组成，按顺序有这些段:
**exec header --> 文件头**
  这一段中含有一些参数，内核利用其中一些参数来把二进制文件加载到内存中并执行，ld利用另外一些参数来连接其它的二进制文件，这个段是唯一含有命令参数的。

**text segment --> 代码段**
  包括在程序执行时加载到内存中的机器码和相关数据（有可能是只读的）

**data segment --> 数据段**
  包括初始化过的数据变量，通常是加载到内存中的可写去中

**text relocations --> 代码重定向**
  包含编译连接二进制文件时的记录，这些记录使用来更新代码段中的指针

**data relocations --> 数据重定向**
  和代码重定向相似，区别是它针对于数据段的指针

**symbol table --> 符号表**
  包含连接器对不同二进制文件中的变量，函数和地址之间的对应关系的记录

**string table --> 字符串表**
  包含和符号名字相一致的字符串

------

**2、什么是链接？程序int main() {}需要链接吗？ 为什么？**

C语言代码经编译后，并没有生成最终的可执行文件，而是生成了一种叫做目标文件的中间文件（临时文件）。目标文件也是二进制形式的，它和可执行文件的格式是一样的。Visual C++目标文件的后缀是`.obj`； GCC目标文件的后缀是`.o`。

目标文件经过链接（Link）后才能变成可执行文件。既然目标文件和可执行文件的格式是一样的，为什么还要再链接一次呢，直接作为可执行文件不行吗？

不行的！因为编译只是将我们自己写的代码变成了二进制形式，它还需要和系统组件（比如标准库、动态链接库等）结合起来，这些组件都是程序运行所必须的。

**链接（Link）**其实就是一个“打包”的过程，它将所有二进制形式的目标文件和系统组件组合成一个可执行文件。完成链接的过程也需要一个特殊的软件，叫做**链接器（Linker）**。

随着我们学习的深入，我们编写的代码越来越多，最终需要将它们分散到多个源文件中，编译器每次只能编译一个源文件，生成一个目标文件，这个时候，链接器除了将目标文件和系统组件组合起来，还需要将编译器生成的多个目标文件组合起来。

！编译是针对一个源文件的，有多少个源文件就需要编译多少次，就会生成多少个目标文件！

**3、a.out 怎么被加载进内存执行的，为什么 windows 上的文件在 linux 上无法执行。**





**4、cpu 眼里看到了什么，cpu 是怎么取数据的？（微机原理）**





**5、cpu 从哪里取数据，cpu 取不到了会怎么样（L1~L3 cache）**





**6、执行过程 cpu 需要访问内存，中间 mmu 是干嘛的？什么是 tlb。**





**7、进程需要申请内存，调用 malloc，后面发生了什么？（虚拟内存管理机制）**





**8、7 的情况下，swap 是否配置有什么影响。（中断和缺页异常）**





**9、程序使用了 write(2)，是怎么调用的？(系统调用的原理，cpu 运行等级)**





**10、数据写入磁盘的过程经历了哪些组件和阶段？（pagecache）**