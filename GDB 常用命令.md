# GDB 常用命令

[![img](https://upload.jianshu.io/users/upload_avatars/7304940/d255c0f3-745d-4a01-bd63-86ca5ff81796.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)](https://www.jianshu.com/u/46663e457260)

[南风知我_](https://www.jianshu.com/u/46663e457260)

0.1442019.02.10 21:40:23字数 1,845阅读 3,442

###### 官方文档

###### 使用GDB调试

- 使用

  ```
  -g
  ```

  编译选项 

  

```shell
$ gcc -g program.c -o programname
```

 

在gdb中启动程序 

```shell
$ gdb programname 
(gdb) run arg1 "arg2" ...
```

 

在gdb中重启程序 

```shell
(gdb) kill
Kill the program being debugged? (y or n) y
(gdb) run
```

 

退出gdb 

```csharp
(gdb) quit
The program is running. Exit anyway? (y or n) 
```

 

单步调试进入函数 

```undefined
(gdb) s
```

 

 

跳出函数 

```undefined
(gdb) finish
```

-  
- 跳出循环
   `until`执行直到达到当前栈帧中当前行后的某一行（用于跳过循环、递归函数调用）

------

###### 查看变量

- 控制输出格式 

  - x 按十六进制格式显示变量。
  - d 按十进制格式显示变量。
  - u 按十六进制格式显示无符号整型。
  - o 按八进制格式显示变量。
  - t 按二进制格式显示变量。
  - a 按十六进制格式显示变量。
  - c 按字符格式显示变量。
  - f 按浮点数格式显示变量。

  

```bash
(gdb) p/x mask
$16 = 0x1
```

 

动态数组
`int *array = (int *) malloc (len * sizeof (int));`

```cpp
(gdb) p *array@len
```

 

查看变量类型 

```go
(gdb) ptype el->fired
type = struct aeFiredEvent {
    int fd;
    int mask;
} *
```

-  

###### 执行shell命令

在gdb命令行界面可以执行外部的Shell命令

```css
(gdb) !ls
a.out test.c
```

------

###### 查看当前代码运行位置

```shell
(gdb) where
#0  event_base_loop (base=0x602010, flags=0) at event.c:466
#1  0x00007ffff7bbb5c7 in event_base_dispatch (event_base=0x602010) at event.c:405
#2  0x0000000000400858 in main () at test.c:20
```

------

###### 查看源代码

```css
(gdb) l 
(gdb) l main.cpp:8
```

------

###### 断点

断点的设置原理：在程序中设置断点，就是先将该位置的原来的指令保存，然后向该位置写入`int 3`指令，当执行到`int 3`的时候，发生软中断，内核会给子进程发出`SIGTRAP`信号，当然这个信号会被转发给父进程。然后用保存的指令替换`int 3`，等待恢复运行。

断点的实现原理：就是在指定的位置插入断点指令，当被调试的程序运行到断点的时候，产生`SIGTRAP`信号，该信号被gdb捕获并进行断点命中判定，当gdb判断出这次`SIGTRAP`是断点命中之后就会转入等待用户输入进行下一步处理，否则继续。

- 设置行号断点 

  

```css
(gdb) break test.c:8 
```

 

设置函数断点 

- 设置C函数断点 

  

```kotlin
(gdb) break func1
```

 

设置C++函数断点

```cpp
(gdb) break TestClass::testFunc(int) 
```

-  

设置临时断点

```undefined
(gdb) tbreak 8
```

 

设置条件断点 

```undefined
b 11 if i == 3
```

 

查看断点 

```css
(gdb) info b
Num     Type           Disp Enb Address            What
4       breakpoint     keep y   0x000000000040053f in main at test.c:6
```

 

使能断点 

```css
(gdb) disable 2
(gdb) info breakpoints
Num Type           Disp Enb Address    What
2   breakpoint     keep n   0x080483c3 in func2 at test.c:5
3   breakpoint     keep y   0x080483da in func1 at test.c:10
```

 

跳过断点

```ruby
(gdb) ignore 2 5
Will ignore next 5 crossings of breakpoint 2.
```

-  

------

###### 堆栈

首先理解函数与调用栈的关系，参见：关于[函数调用浅析](https://www.jianshu.com/p/b6c26506bfc1)

- 查看调用栈 

  

```shell
(gdb) backtrace
#0  func2 (x=30) at test.c:5
#1  0x80483e6 in func1 (a=30) at test.c:10
#2  0x8048414 in main (argc=1, argv=0xbffffaf4) at test.c:19
#3  0x40037f5c in __libc_start_main () from /lib/libc.so.6
(gdb) 
```

 

查看所有线程的调用栈 

```bash
 #  thread apply all bt
```

 

选择栈帧 

```shell
(gdb) frame 2
#2  0x8048414 in main (argc=1, argv=0xbffffaf4) at test.c:19
19        x = func1(x);
(gdb)
```

 

查看栈帧 

```shell
(gdb) info frame
Stack level 2, frame at 0xbffffa8c:
 eip = 0x8048414 in main (test.c:19); saved eip 0x40037f5c
 called by frame at 0xbffffac8, caller of frame at 0xbffffa5c
 source language c.
 Arglist at 0xbffffa8c, args: argc=1, argv=0xbffffaf4
 Locals at 0xbffffa8c, Previous frame's sp is 0x0
 Saved registers:
  ebp at 0xbffffa8c, eip at 0xbffffa90

(gdb) info locals
x = 30
s = 0x8048484 "Hello World!\n"

(gdb) info args
argc = 1
argv = (char **) 0xbffffaf4
```

-  

------

###### 观察点

------

###### 查看内存

- 命令格式
   `x/[n][f][u] [address]`

  - n 表示显示内存长度
  - f 表示显示格式，如同上面打印变量定义
  - u 表示每次读取的字节数，默认是4bytes 
    - b 表示单字节
    - h 表示双字节
    - w 表示四字节
    - g 表示八字节

- 以字符串的形式查看

  

```cpp
int main(int argc, char **argv)
{
    char *s = "hello world";
}
(gdb) x/s s
0x4005a0: "hello world"
```

 

以字符的形式查看

```bash
(gdb) x/c s
0x4005a0: 104 'h'
```

 

以二进制查看

```undefined
(gdb) x/t s
0x4005a0: 01101000
```

 

以八进制查看

```undefined
(gdb) x/x s
0x4005a0: 0x68
```

-  

------

###### 寄存器

```css
(gdb) info registers
rax            0x4004f0 4195568
rbx            0x0  0
rcx            0x400510 4195600
rdx            0x7fffffffe598   140737488348568
rsi            0x7fffffffe588   140737488348552
rdi            0x1  1
rbp            0x7fffffffe4a0   0x7fffffffe4a0
rsp            0x7fffffffe4a0   0x7fffffffe4a0
r8             0x7ffff7dd6e80   140737351872128
r9             0x0  0
r10            0x7fffffffe2f0   140737488347888
r11            0x7ffff7a3ca40   140737348094528
r12            0x400400 4195328
r13            0x7fffffffe580   140737488348544
r14            0x0  0
r15            0x0  0
rip            0x400503 0x400503 <main+19>
eflags         0x246    [ PF ZF IF ]
cs             0x33 51
ss             0x2b 43
ds             0x0  0
es             0x0  0
fs             0x0  0
gs             0x0  0
```

- 实验1：探究x86参数传递

  

```cpp
#include <stdio.h>
#include <stdlib.h>

int v1 = 1;
float v2 = 0.01;

void func(int a, long b, short c, char d, long long e, float f, double g, int *h, float *i, char *j) 
{
    printf("a: %d, b: %ld, c: %d, d: %c, e: %lld\n"
         "f: %.3e, g: %.3e\n"
         "h: %p, i: %p, j: %p\n", a, b, c, d, e, f, g, h, i, j);                                                     
}
int main()
{
    func(100, 35000, 5, 'A', 123456789LL, 3.14, 2.99792458e8, &v1, &v2, "string");
    return 0;
}
(gdb) b *func
(gdb) r
(gdb) i r
rax            0x41b1de784a000000 4733809291562057728
rbx            0x0    0
rcx            0x41   65
rdx            0x5    5
rsi            0x88b8 35000
rdi            0x64   100
rbp            0x7fffffffe490 0x7fffffffe490
rsp            0x7fffffffe468 0x7fffffffe468
r8             0x75bcd15  123456789
r9             0x601034   6295604
r10            0x7fffffffe2e0 140737488347872
r11            0x7ffff7a3ca40 140737348094528
r12            0x400440   4195392
r13            0x7fffffffe570 140737488348528
r14            0x0    0
r15            0x0    0
rip            0x400530   0x400530 <func>
eflags         0x202  [ IF ]
cs             0x33   51
ss             0x2b   43
ds             0x0    0
es             0x0    0
fs             0x0    0
gs             0x0    0
```

- 可以发现，开头的5个参数a，b，c， d，e分别保存到了`rdi`, `rsi`, `rdx`, `rcx`, 和 `r8`中。

------

###### 查看汇编代码

```xml
(gdb) disassemble main
Dump of assembler code for function main:
   0x00000000004004f0 <+0>: push   %rbp
   0x00000000004004f1 <+1>: mov    %rsp,%rbp
   0x00000000004004f4 <+4>: mov    %edi,-0x14(%rbp)
   0x00000000004004f7 <+7>: mov    %rsi,-0x20(%rbp)
   0x00000000004004fb <+11>:    movq   $0x4005a0,-0x8(%rbp)
=> 0x0000000000400503 <+19>:    pop    %rbp
   0x0000000000400504 <+20>:    retq   
End of assembler dump.
```

------

###### 常用设置

- print pretty 
  - set print pretty on 打开该设置，结构体显示更好看。
  - set print pretty off
  - show print pretty

###### 结合core文件调试

使用内核转储（core）的最大好处就是：它能保存问题发生时的状态，只要有问题发生时程序的可执行文件和内核转储，就可以知道进程当时的状态。

```css
gdb a.out xx.core
```

- 启用内核转储功能

  

```shell
$ ulimit -c
0
```

`-c`选项表示内核转储文件的大小限制，0表示内核转储未打开。按照以下命令开启:

```shell
$ ulimit -c unlimited
$ ulimit -c
unlimited
```

- `ulimit -c unlimited` 设置core文件大小不限制，这样设置是一次性的，会话结束就恢复原样
   在用户的`~/.bash_profile`里加上`ulimit -c unlimited`来让用户开启内核转储功能
   在`/etc/profile`里加上`ulimit -c unlimited`来让所有用户开启内核转储功能（?）

- core文件目录和命名规则
   在`/proc/sys/kernel/core_pattern`可以设置格式化的core文件保存位置和文件名。
   比如`core-%e-%p-%t`表示在当前目录生成"core-命令-pid-时间戳"为文件名的core文件。
   比如`/cfg/core-%e-%p-%t`表示在/cfg下生成"core-命令-pid-时间戳"为文件名的core文件

- 参数列表

  | 符号 | 意义                                                         |
  | ---- | ------------------------------------------------------------ |
  | %p   | insert pid into filename 添加 pid                            |
  | %u   | insert current uid into filename 添加当前 uid                |
  | %g   | insert current gid into filename 添加当前 gid                |
  | %s   | insert signal that caused the coredump into the filename 添加导致产生 core 的信号 |
  | %t   | insert UNIX time that the coredump occurred into filename 添加 core 文件生成时的 unix 时间戳 |
  | %h   | insert hostname where the coredump happened into filename 添加主机名 |
  | %e   | insert coredumping executable name into filename 添加命令名  |

------

###### 多进程调试

gdb的调试默认是调试父进程的，但我们可以通过设置来选择调试哪个进程。

```bash
(gdb) set follow-fork-mode parent/child
```

如果选择了parent，这个时候就是进行gdb调试父进程。
 如果选择了child，这个时候就是进行gdb调试子进程。
 注意，在调试的过程中更改mode是没有用的，这种设置只对下一次fork后起作用。