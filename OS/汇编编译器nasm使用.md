## nasm使用

1、创建.asm文件

2、插入以下汇编代码

```
section .data
  hello:     db 'Hello world!',10    ; 'Hello world!' plus a linefeed character
  helloLen:  equ $-hello             ; Length of the 'Hello world!' string
                                     ; (I'll explain soon)
 
section .text
  global _start
 
_start:
  mov eax,4            ; The system call for write (sys_write)
  mov ebx,1            ; File descriptor 1 - standard output
  mov ecx,hello        ; Put the offset of hello in ecx
  mov edx,helloLen     ; helloLen is a constant, so we don't need to say
                       ;  mov edx,[helloLen] to get it's actual value
  int 80h              ; Call the kernel
 
  mov eax,1            ; The system call for exit (sys_exit)
  mov ebx,0            ; Exit with return code of 0 (no error)
  int 80h
```

3、**编译**

`-f`用来指定outputFile格式（bin为默认输出）

```bash
nasm -f elf64 hello.asm
```

4、**链接**

```bash
ld -s -o hello hello.o
```

5、**运行**

```bash
./hello
```

