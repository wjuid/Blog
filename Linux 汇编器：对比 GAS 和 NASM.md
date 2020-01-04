# Linux 汇编器：对比 GAS 和 NASM

对比 GNU Assembler（GAS）和 Netwide Assembler（NASM）

![](https://dw1.s81c.com/developerworks/i/v18/article/dw-author.png)

Ram Narayam
2007 年 11 月 05 日发布

[Weibo](https://service.weibo.com/share/share.php?url=http%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Flinux%2Fl-gas-nasm.html%&title=Linux%20%E6%B1%87%E7%BC%96%E5%99%A8%EF%BC%9A%E5%AF%B9%E6%AF%94%20GAS%20%E5%92%8C%20NASM&language=zh_cn)[Google+](https://plus.google.com/share?url=http%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Flinux%2Fl-gas-nasm.html&t=Linux 汇编器：对比 GAS 和 NASM)[用电子邮件发送本页面](mailto:?subject=Linux 汇编器：对比 GAS 和 NASM&body=http%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Flinux%2Fl-gas-nasm.html)

​                                                                [                                                                     ![Comments](https://dw1.s81c.com/developerworks/i/v18/article/dw-article-cmt-icon.png)                                                                 ](https://www.ibm.com/developerworks/cn/linux/l-gas-nasm.html#icomments)                                                            

[                                                                     0                                                                 ](https://www.ibm.com/developerworks/cn/linux/l-gas-nasm.html#icomments)

与其他语言不同，**汇编语言**要求开发人员了解编程所用机器的处理器体系结构。汇编程序不可移植，维护和理解常常比较麻烦，通常包含大量代码行。但是，在机器上执行的运行时二进制代码在速度和大小方面有优势。 

对于在 Linux 上进行汇编级编程已经有许多参考资料，本文主要讲解语法之间的差异，帮助您更轻松地在汇编形式之间进行转换。本文源于我自己试图改进这种转换的尝试。 

本文使用一系列程序示例。每个程序演示一些特性，然后是对语法的讨论和对比。尽管不可能讨论 NASM 和 GAS 之间存在的每个差异，但是我试图讨论主要方面，给进一步研究提供一个基础。那些已经熟悉 NASM 和 GAS  的读者也可以在这里找到有用的内容，比如宏。 

本文假设您至少基本了解汇编的术语，曾经用符合 Intel® 语法的汇编器编写过程序，可能在 Linux 或 Windows 上使用过 NASM。本文并不讲解如何在编辑器中输入代码，或者如何进行汇编和链接（但是下面的边栏可以帮助您 [快速回忆一下](https://www.ibm.com/developerworks/cn/linux/l-gas-nasm.html#sidebar)）。您应该熟悉 Linux 操作系统（任何 Linux 发行版都可以；我使用的是 Red Hat 和 Slackware）和基本的 GNU 工具，比如 gcc 和 ld，还应该在 x86 机器上进行编程。 

现在，我描述一下本文讨论的范围。

##### 构建示例


**汇编：**
 GAS：
`as –o program.o program.s`

 NASM：
`nasm –f elf –o program.o program.asm`

**链接（对于两种汇编器通用）：**
`ld –o program program.o`

**在使用外部 C 库时的链接方法：**
`ld –-dynamic-linker /lib/ld-linux.so.2 –lc –o program program.o`

本文讨论： 

- NASM 和 GAS 之间的基本语法差异
- 常用的汇编级结构，比如变量、循环、标签和宏
- 关于调用外部 C 例程和使用函数的信息
- 汇编助记符差异和使用方法
- 内存寻址方法

本文不讨论：

- 处理器指令集
- 一种汇编器特有的各种宏形式和其他结构
- NASM 或 GAS 特有的汇编器指令
- 不常用的特性，或者只在一种汇编器中出现的特性

更多信息请参考汇编器的官方手册（参见 [参考资料](https://www.ibm.com/developerworks/cn/linux/l-gas-nasm.html#artrelatedtopics) 中的链接），因为这些手册是最完整的信息源。 

## 基本结构

清单 1 给出一个非常简单的程序，它的作用仅仅是使用退出码 2 退出。这个小程序展示了 NASM 和 GAS 的汇编程序的基本结构。 

| 行号                                                         | NASM                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `001``002``003``004``005``006``007``008``009``010``011``012``013``014``015``016` | `; Text segment begins``section .text` `  ``global _start` `; Program entry point``  ``_start:` `; Put the code number for system call``   ``mov  eax, 1 ` `; Return value ``   ``mov  ebx, 2` `; Call the OS``   ``int  80h` |

现在解释一下。 

NASM 和 GAS 之间最大的差异之一是语法。GAS 使用 AT&T 语法，这是一种相当老的语法，由 GAS 和一些老式汇编器使用；NASM 使用 Intel 语法，大多数汇编器都支持它，包括 TASM 和 MASM。（GAS 的现代版本支持 `.intel_syntax` 指令，因此允许在 GAS 中使用 Intel 语法。） 

下面是从 GAS 手册总结出的一些主要差异： 

- AT&T 和 Intel 语法采用相反的源和目标操作数次序。例如：  

  - Intel：`mov eax, 4`
  - AT&T：`movl $4, %eax`

- 在 AT&T 语法中，中间操作数前面加 

  ```
  $
  ```

  ；在 Intel 语法中，中间操作数不加前缀。例如：  

  - Intel：`push 4`
  - AT&T：`pushl $4`

- 在 AT&T 语法中，寄存器操作数前面加 `%`。在 Intel 语法中，它们不加前缀。

- 在 AT&T 语法中，内存操作数的大小由操作码名称的最后一个字符决定。操作码后缀 

  ```
  b
  ```

  、

  ```
  w
  ```

   和 

  ```
  l
  ```

   分别指定字节（8 位）、字（16 位）和长（32 位）内存引用。Intel 语法通过在内存操作数（而不是操作码本身）前面加 

  ```
  byte ptr
  ```

  、

  ```
  word ptr
  ```

   和 

  ```
  dword ptr
  ```

   来指定大小。所以： 

  - Intel：`mov al, byte ptr foo`
  - AT&T：`movb foo, %al`

- 在 AT&T 语法中，中间形式长跳转和调用是 `lcall/ljmp $section, $offset`；Intel 语法是 `call/jmp far section:offset`。在 AT&T 语法中，远返回指令是 `lret $stack-adjust`，而 Intel 使用 `ret far stack-adjust`。

在这两种汇编器中，寄存器的名称是一样的，但是因为寻址模式不同，使用它们的语法是不同的。另外，GAS 中的汇编器指令以 “.” 开头，但是在 NASM 中不是。 

`.text` 部分是处理器开始执行代码的地方。`global`（或者 GAS 中的 `.globl` 或 `.global`）关键字用来让一个符号对链接器可见，可以供其他链接对象模块使用。在清单 1 的 NASM 部分中，`global _start` 让 `_start` 符号成为可见的标识符，这样链接器就知道跳转到程序中的什么地方并开始执行。与 NASM 一样，GAS 寻找这个 `_start` 标签作为程序的默认进入点。在 GAS 和 NASM 中标签都以冒号结尾。 

中断是一种通知操作系统需要它的服务的一种方法。第 16 行中的 `int` 指令执行这个工作。GAS 和 NASM 对中断使用同样的助记符。GAS 使用 `0x` 前缀指定十六进制数字，NASM 使用 `h` 后缀。因为在 GAS 中中间操作数带 `$` 前缀，所以 80 hex 是 `$0x80`。 

`int $0x80`（或 NASM 中的 `80h`）用来向 Linux 请求一个服务。服务编码放在 EAX 寄存器中。EAX 中存储的值是 1（代表 Linux exit 系统调用），这请求程序退出。EBX 寄存器包含退出码（在这个示例中是 2），也就是返回给操作系统的一个数字。（可以在命令提示下输入 `echo $?` 来检查这个数字。） 

最后讨论一下注释。GAS 支持 C 风格（`/* */`）、C++ 风格（`//`）和 shell 风格（`#`）的注释。NASM 支持以 “;” 字符开头的单行注释。 

## 变量和内存访问

本节首先给出一个示例程序，它寻找三个数字中的最大者。 

| 行号                                                         | NASM                                                         | GAS                                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `001``002``003``004``005``006``007``008``009``010``011``012``013``014``015``016``017``018``019``020``021``022``023``024``025``026``027``028``029``030``031` | `; Data section begins``section .data` `  ``var1 dd 40` `  ``var2 dd 20` `  ``var3 dd 30` `section .text` `  ``global _start` `  ``_start:` `; Move the contents of variables``   ``mov  ecx, [var1]``   ``cmp  ecx, [var2]``   ``jg  check_third_var``   ``mov  ecx, [var2]` `  ``check_third_var:``   ``cmp  ecx, [var3]``   ``jg  _exit``   ``mov  ecx, [var3]` `  ``_exit:``   ``mov  eax, 1``   ``mov  ebx, ecx``   ``int  80h` | `// Data section begins``.section .data``  ` `  ``var1:``   ``.int 40``  ``var2:``   ``.int 20``  ``var3:``   ``.int 30` `.section .text` `  ``.globl _start` `  ``_start:` `# move the contents of variables``   ``movl (var1), %ecx``   ``cmpl (var2), %ecx``   ``jg  check_third_var``   ``movl (var2), %ecx` `  ``check_third_var:``   ``cmpl (var3), %ecx``   ``jg  _exit``   ``movl (var3), %ecx``  ` `  ``_exit:``   ``movl $1, %eax``   ``movl %ecx, %ebx``   ``int  $0x80` |

在上面的内存变量声明中可以看到几点差异。NASM 分别使用 `dd`、`dw` 和 `db` 指令声明 32 位、16 位和 8 位数字，而 GAS 分别使用 `.long`、`.int` 和 `.byte`。GAS 还有其他指令，比如 `.ascii`、`.asciz` 和 `.string`。在 GAS 中，像声明其他标签一样声明变量（使用冒号），但是在 NASM 中，只需在内存分配指令（`dd`、`dw` 等等）前面输入变量名，后面加上变量的值。 

清单 2 中的第 18 行演示内存直接寻址模式。NASM 使用方括号间接引用一个内存位置指向的地址值：`[var1]`。GAS 使用圆括号间接引用同样的值：`(var1)`。本文后面讨论其他寻址模式的使用方法。 

## 使用宏

清单 3 演示本节讨论的概念；它接受用户名作为输入并返回一句问候语。 

| 行号                                                         | NASM                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `001``002``003``004``005``006``007``008``009``010``011``012``013``014``015``016``017``018``019``020``021``022``023``024``025``026``027``028``029``030``031``032``033``034``035``036``037``038``039``040``041``042``043``044``045``046``047``048``049``050``051``052``053``054``055``056``057``058``059``060``061``062` | `section .data` `  ``prompt_str db  'Enter your name: '` `; $ is the location counter``  ``STR_SIZE equ $ - prompt_str` `  ``greet_str db 'Hello '` `  ``GSTR_SIZE equ $ - greet_str` `section .bss` `; Reserve 32 bytes of memory``  ``buff resb 32` `; A macro with two parameters``; Implements the write system call``  ``%macro write 2 ``   ``mov  eax, 4``   ``mov  ebx, 1``   ``mov  ecx, %1``   ``mov  edx, %2``   ``int  80h``  ``%endmacro` `; Implements the read system call``  ``%macro read 2``   ``mov  eax, 3``   ``mov  ebx, 0``   ``mov  ecx, %1``   ``mov  edx, %2``   ``int  80h``  ``%endmacro` `section .text` `  ``global _start` `  ``_start:``   ``write prompt_str, STR_SIZE``   ``read buff, 32` `; Read returns the length in eax``   ``push eax` `; Print the hello text``   ``write greet_str, GSTR_SIZE` `   ``pop  edx` `; edx = length returned by read``   ``write buff, edx` `  ``_exit:``   ``mov  eax, 1``   ``mov  ebx, 0``   ``int  80h` |

本节要讨论宏以及 NASM 和 GAS 对它们的支持。但是，在讨论宏之前，先与其他几个特性做一下比较。 

清单 3 演示了未初始化内存的概念，这是用 `.bss` 部分指令（第 14 行）定义的。BSS 代表 “block storage segment” （原来是以一个符号开头的块），BSS  部分中保留的内存在程序启动时初始化为零。BSS 部分中的对象只有一个名称和大小，没有值。与数据部分中不同，BSS  部分中声明的变量并不实际占用空间。 

NASM 使用 `resb`、`resw` 和 `resd` 关键字在 BSS 部分中分配字节、字和双字空间。GAS 使用 `.lcomm` 关键字分配字节级空间。请注意在这个程序的两个版本中声明变量名的方式。在 NASM 中，变量名前面加 `resb`（或 `resw` 或 `resd`）关键字，后面是要保留的空间量；在 GAS 中，变量名放在 `.lcomm` 关键字的后面，然后是一个逗号和要保留的空间量。 

 NASM：`varname resb size`

 GAS：`.lcomm varname, size`

 清单 3 还演示了位置计数器的概念（第 6 行）。                                                                                    NASM 提供特殊的变量（`$` 和 `$$` 变量）来操作位置计数器。在 GAS 中，无法操作位置计数器，必须使用标签计算下一个存储位置（数据、指令等等）。 

例如，为了计算一个字符串的长度，在 NASM 中会使用以下指令： 

```
 prompt_str db 'Enter your name: ' STR_SIZE equ $ - prompt_str  ; $ is the location counter 
```

`$` 提供位置计数器的当前值，从这个位置计数器中减去标签的值（所有变量名都是标签），就会得出标签的声明和当前位置之间的字节数。`equ` 用来将变量 STR_SIZE 的值设置为后面的表达式。GAS 中使用的相似指令如下： 

```
 prompt_str:  .ascii "Enter Your Name: " pstr_end:  .set STR_SIZE, pstr_end - prompt_str 
```

末尾标签（`pstr_end`）给出下一个位置地址，减去启始标签地址就得出大小。还要注意，这里使用 `.set` 将变量 STR_SIZE 的值设置为逗号后面的表达式。也可以使用对应的 `.equ`。在 NASM 中，没有与 GAS 的 `set` 指令对应的指令。 

正如前面提到的，清单 3 使用了宏（第 21 行）。在 NASM 和 GAS  中存在不同的宏技术，包括单行宏和宏重载，但是这里只关注基本类型。宏在汇编程序中的一个常见用途是提高代码的清晰度。通过创建可重用的宏，可以避免重复输入相同的代码段；这不但可以避免重复，而且可以减少代码量，从而提高代码的可读性。 

 NASM 使用 `%beginmacro` 指令声明宏，用 `%endmacro` 指令结束声明。`%beginmacro` 指令后面是宏的名称。宏名称后面是一个数字，这是这个宏需要的宏参数数量。在 NASM 中，宏参数是从 1 开始连续编号的。也就是说，宏的第一个参数是 %1，第二个是 %2，第三个是 %3，以此类推。例如： 

```
 %beginmacro macroname 2  mov eax, %1  mov ebx, %2 %endmacro 
```

这创建一个有两个参数的宏，第一个参数是 `%1`，第二个参数是 `%2`。因此，对上面的宏的调用如下所示： 

```
 macroname 5, 6 
```

还可以创建没有参数的宏，在这种情况下不指定任何数字。 

现在看看 GAS 如何使用宏。GAS 提供 `.macro` 和 `.endm` 指令来创建宏。`.macro` 指令后面跟着宏名称，后面可以有参数，也可以没有参数。在 GAS 中，宏参数是按名称指定的。例如： 

```
 .macro macroname arg1, arg2  movl \arg1, %eax  movl \arg2, %ebx .endm 
```

当在宏中使用宏参数名称时，在名称前面加上一个反斜线。如果不这么做，链接器会把名称当作标签而不是参数，因此会报告错误。 

## 函数、外部例程和堆栈

本节的示例程序在一个整数数组上实现选择排序。 

| 行号                                                         | NASM                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `001``002``003``004``005``006``007``008``009``010``011``012``013``014``015``016``017``018``019``020``021``022``023``024``025``026``027``028``029``030``031``032``033``034``035``036``037``038``039``040``041``042``043``044``045``046``047``048``049``050``051``052``053``054``055``056``057``058``059``060``061``062``063``064``065``066``067``068``069``070``071``072``073``074``075``076``077``078``079``080``081``082``083``084``085``086``087``088``089``090``091``092``093``094``095``096``097``098``099``100``101``102``103``104``105``106``107``108``109``110``111``112``113``114``115``116``117``118``119``120``121``122``123``124``125``126``127``128``129``130``131``132``133``134``135``136``137``138``139``140``141``142``143``144``145` | `section .data` `  ``array db``   ``89, 10, 67, 1, 4, 27, 12, 34,``     ``86, 3` `  ``ARRAY_SIZE equ $ - array` `  ``array_fmt db " %d", 0` `  ``usort_str db "unsorted array:", 0` `  ``sort_str db "sorted array:", 0` `  ``newline db 10, 0`  `section .text``  ``extern puts` `  ``global _start` `  ``_start:` `   ``push usort_str``   ``call puts``   ``add  esp, 4``  ` `   ``push ARRAY_SIZE``   ``push array``   ``push array_fmt``   ``call print_array10``   ``add  esp, 12` `   ``push ARRAY_SIZE ``   ``push array``   ``call sort_routine20` `; Adjust the stack pointer``   ``add  esp, 8` `   ``push sort_str``   ``call puts``   ``add  esp, 4` `   ``push ARRAY_SIZE ``   ``push array``   ``push array_fmt``   ``call print_array10``   ``add  esp, 12``   ``jmp  _exit` `   ``extern printf` `  ``print_array10:``   ``push ebp``   ``mov  ebp, esp``   ``sub  esp, 4``   ``mov  edx, [ebp + 8]``   ``mov  ebx, [ebp + 12]``   ``mov  ecx, [ebp + 16]` `   ``mov  esi, 0` `  ``push_loop:``   ``mov  [ebp - 4], ecx``   ``mov  edx, [ebp + 8]``   ``xor  eax, eax``   ``mov  al, byte [ebx + esi]``   ``push eax``   ``push edx` `   ``call printf``   ``add  esp, 8``   ``mov  ecx, [ebp - 4]``   ``inc  esi``   ``loop push_loop` `   ``push newline``   ``call printf``   ``add  esp, 4``   ``mov  esp, ebp``   ``pop  ebp``   ``ret` `  ``sort_routine20:``   ``push ebp``   ``mov  ebp, esp` `; Allocate a word of space in stack``   ``sub  esp, 4 ` `; Get the address of the array``   ``mov  ebx, [ebp + 8] ` `; Store array size``   ``mov  ecx, [ebp + 12]``   ``dec  ecx` `; Prepare for outer loop here``   ``xor  esi, esi` `  ``outer_loop:``; This stores the min index``   ``mov  [ebp - 4], esi ``   ``mov  edi, esi``   ``inc  edi` `  ``inner_loop:``   ``cmp  edi, ARRAY_SIZE``   ``jge  swap_vars``   ``xor  al, al``   ``mov  edx, [ebp - 4]``   ``mov  al, byte [ebx + edx]``   ``cmp  byte [ebx + edi], al``   ``jge  check_next``   ``mov  [ebp - 4], edi` `  ``check_next:``   ``inc  edi``   ``jmp  inner_loop` `  ``swap_vars:``   ``mov  edi, [ebp - 4]``   ``mov  dl, byte [ebx + edi]``   ``mov  al, byte [ebx + esi]``   ``mov  byte [ebx + esi], dl``   ``mov  byte [ebx + edi], al` `   ``inc  esi``   ``loop outer_loop` `   ``mov  esp, ebp``   ``pop  ebp``   ``ret` `  ``_exit:``   ``mov  eax, 1``   ``mov  ebx, 0``   ``int  80h` |

初看起来清单 4 似乎非常复杂，实际上它是非常简单的。这个清单演示了函数、各种内存寻址方案、堆栈和库函数的使用方法。这个程序对包含 10 个数字的数组进行排序，并使用外部 C 库函数 `puts` 和 `printf` 输出未排序数组和已排序数组的完整内容。为了实现模块化和介绍函数的概念，排序例程本身实现为一个单独的过程，数组输出例程也是这样。我们来逐一分析一下。 

在声明数据之后，这个程序首先执行对 `puts` 的调用（第 31 行）。`puts` 函数在控制台上显示一个字符串。它惟一的参数是要显示的字符串的地址，通过将字符串的地址压入堆栈（第 30 行），将这个参数传递给它。 

在 NASM 中，任何不属于我们的程序但是需要在链接时解析的标签都必须预先定义，这就是 `extern` 关键字的作用（第 24 行）。GAS 没有这样的要求。在此之后，字符串的地址 `usort_str` 被压入堆栈（第 30 行）。在 NASM 中，内存变量（比如 `usort_str`）代表内存位置本身，所以 `push usort_str` 这样的调用实际上是将地址压入堆栈的顶部。但是在 GAS 中，变量 `usort_str` 必须加上前缀 `$`，这样它才会被当作地址。如果不加前缀 `$`，那么会将内存变量代表的实际字节压入堆栈，而不是地址。 

因为在堆栈中压入一个变量会让堆栈指针移动一个双字，所以给堆栈指针加 4（双字的大小）（第 32 行）。 

现在将三个参数压入堆栈，并调用 `print_array10` 函数（第 37 行）。在 NASM 和 GAS 中声明函数的方法是相同的。它们仅仅是通过 `call` 指令调用的标签。 

在调用函数之后，ESP 代表堆栈的顶部。`esp + 4` 代表返回地址，`esp + 8` 代表函数的第一个参数。在堆栈指针上加上双字变量的大小（即 `esp + 12`、`esp + 16` 等等），就可以访问所有后续参数。 

在函数内部，通过将 `esp` 复制到 `ebp` （第 62 行）创建一个局部堆栈框架。和程序中的处理一样，还可以为局部变量分配空间（第 63 行）。方法是从 `esp` 中减去所需的字节数。`esp – 4` 表示为一个局部变量分配 4 字节的空间，只要堆栈中有足够的空间容纳局部变量，就可以继续分配。 

 清单 4 演示了基间接寻址模式（第 64 行），也就是首先取得一个基地址，然后在它上面加一个偏移量，从而到达最终的地址。在清单的 NASM 部分中，`[ebp + 8]` 和 `[ebp – 4]`（第 71 行）就是基间接寻址模式的示例。在 GAS 中，寻址方法更简单一些：`4(%ebp)` 和 `-4(%ebp)`。 

在 `print_array10` 例程中，在 `push_loop` 标签后面可以看到另一种寻址模式（第 74 行）。在 NASM 和 GAS 中的表示方法如下： 

 NASM：`mov al, byte [ebx + esi]`

 GAS：`movb (%ebx, %esi, 1), %al`

这种寻址模式称为基索引寻址模式。这里有三项数据：一个是基地址，第二个是索引寄存器，第三个是乘数。因为不可能决定从一个内存位置开始访问的字节数，所以需要用一个方法计算访问的内存量。NASM 使用字节操作符告诉汇编器要移动一个字节的数据。在 GAS 中，用一个乘数和助记符中的 `b`、`w` 或 `l` 后缀（例如 `movb`）来解决这个问题。初看上去 GAS 的语法似乎有点儿复杂。 

GAS 中基索引寻址模式的一般形式如下： 

```
%segment:ADDRESS (, index, multiplier)
```

 或 

```
%segment:(offset, index, multiplier)
```

 或 

```
%segment:ADDRESS(base, index, multiplier)
```

使用这个公式计算最终的地址： 

```
ADDRESS or offset + base + index * multiplier.
```

因此，要想访问一个字节，就使用乘数 1；对于字，乘数是 2；对于双字，乘数是 4。当然，NASM 使用的语法比较简单。上面的公式在 NASM 中表示为： 

```
Segment:[ADDRESS or offset + index * multiplier]
```

为了访问 1、2 或 4 字节的内存，在这个内存地址前面分别加上 `byte`、`word` 或 `dword`。 

## 其他方面

清单 5 读取命令行参数的列表，将它们存储在内存中，然后输出它们。 

| 行号                                                         | NASM                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `001``002``003``004``005``006``007``008``009``010``011``012``013``014``015``016``017``018``019``020``021``022``023``024``025``026``027``028``029``030``031``032``033``034``035``036``037``038``039``040``041``042``043``044``045``046``047``048``049``050``051``052``053``054``055``056``057``058``059``060``061` | `section .data` `; Command table to store at most``; 10 command line arguments``  ``cmd_tbl:``   ``%rep 10``     ``dd 0``   ``%endrep` `section .text` `  ``global _start` `  ``_start:``; Set up the stack frame``   ``mov  ebp, esp``; Top of stack contains the``; number of command line arguments.``; The default value is 1``   ``mov  ecx, [ebp]` `; Exit if arguments are more than 10``   ``cmp  ecx, 10``   ``jg  _exit` `   ``mov  esi, 1``   ``mov  edi, 0` `; Store the command line arguments``; in the command table``  ``store_loop:``   ``mov  eax, [ebp + esi * 4]``   ``mov  [cmd_tbl + edi * 4], eax``   ``inc  esi``   ``inc  edi``   ``loop store_loop` `   ``mov  ecx, edi``   ``mov  esi, 0` `   ``extern puts``  ` `  ``print_loop:``; Make some local space``   ``sub  esp, 4``; puts function corrupts ecx``   ``mov  [ebp - 4], ecx``   ``mov  eax, [cmd_tbl + esi * 4]``   ``push eax``   ``call puts``   ``add  esp, 4``   ``mov  ecx, [ebp - 4]``   ``inc  esi``   ``loop print_loop` `   ``jmp  _exit``  ` `  ``_exit:``   ``mov  eax, 1``   ``mov  ebx, 0``   ``int  80h` |

清单 5 演示在汇编程序中重复执行指令的方法。很自然，这种结构称为重复结构。在 GAS 中，重复结构以 `.rept` 指令开头（第 6 行）。用一个 `.endr` 指令结束这个指令（第 8 行）。`.rept` 后面是一个数字，它指定 `.rept/.endr` 结构中表达式重复执行的次数。这个结构中的任何指令都相当于编写这个指令 `count` 次，每次重复占据单独的一行。 

例如，如果次数是 3： 

```
 .rept 3  movl $2, %eax .endr 
```

就相当于： 

```
 movl $2, %eax movl $2, %eax movl $2, %eax 
```

在 NASM 中，在预处理器级使用相似的结构。它以 `%rep` 指令开头，以 `%endrep` 结尾。`%rep` 指令后面是一个表达式（在 GAS 中 `.rept` 指令后面是一个数字）： 

```
 %rep   nop %endrep 
```

在 NASM 中还有另一种结构，`times` 指令。与 `%rep` 相似，它也在汇编级起作用，后面也是一个表达式。例如，上面的 `%rep` 结构相当于： 

```
times  nop
```

以下代码： 

```
 %rep 3  mov eax, 2 %endrep 
```

相当于： 

```
times 3 mov eax, 2
```

它们都相当于： 

```
 mov eax, 2 mov eax, 2 mov eax, 2 
```

在清单 5 中，使用 `.rept`（或 `%rep`）指令为 10 个双字创建内存数据区。然后，从堆栈一个个地访问命令行参数，并将它们存储在内存区中，直到命令表填满。 

在这两种汇编器中，访问命令行参数的方法是相似的。ESP（堆栈顶部）存储传递给程序的命令行参数数量，默认值是 1（表示没有命令行参数）。`esp + 4` 存储第一个命令行参数，这总是从命令行调用的程序的名称。`esp + 8`、`esp + 12` 等存储后续命令行参数。 

还要注意清单 5 中从两边访问内存命令表的方法。这里使用内存间接寻址模式（第 31 行）访问命令表，还使用了 ESI（和 EDI）中的偏移量和一个乘数。因此，NASM 中的 `[cmd_tbl + esi * 4]` 相当于 GAS 中的 `cmd_tbl(, %esi, 4)`。 

## 结束语

尽管在这两种汇编器之间存在实质性的差异，但是在这两种形式之间进行转换并不困难。您最初可能觉得 AT&T 语法难以理解，但是掌握了它之后，它其实和 Intel 语法同样简单。 

#### 相关主题

-  您可以参阅本文在 developerWorks 全球站点上的 [英文原文](http://www.ibm.com/developerworks/linux/library/l-gas-nasm.html?S_TACT=105AGX52&S_CMP=cn-a-l)。
- 参考 NASM 和 GAS 手册，了解这两种汇编器的完整信息：  
  - [GAS: GNU Assembler](http://sourceware.org/binutils/docs-2.17/as/index.html)
  - [NASM: Netwide Assembler](http://web.mit.edu/nasm_v0.98/doc/nasm/html/nasmdoc0.html)
- ​        阅读关于 [选择排序](http://en.wikipedia.org/wiki/selection_sort) 的 Wikipedia 文章。      
- 在 [developerWorks Linux 专区](http://www.ibm.com/developerworks/cn/linux/) 中可以找到为 Linux 开发人员准备的更多参考资料，还可以查阅 [最受欢迎的文章和教程](http://www.ibm.com/developerworks/cn/linux/top10/)。       
- ​         查阅 developerWorks 上的所有 [ Linux 技巧](http://www.ibm.com/developerworks/cn/views/linux/articles.jsp?view_by=search&search_by=linux+技巧) 和 [Linux 教程](http://www.ibm.com/developerworks/cn/views/linux/tutorials.jsp)。       
- 使用 [IBM 试用软件](http://www.ibm.com/developerworks/downloads/?S_TACT=105AGX52&S_CMP=cn-a-l) 构建您的下一个 Linux 开发项目，这些软件可以从 developerWorks 直接下载。       

#### 评论

添加或订阅评论，请先登录或注册。

​                 有新评论时提醒我	       






第一个评论