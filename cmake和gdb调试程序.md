# cmake和gdb调试程序

由于出发点是想要在cmake后使用gdb，因此先写一下cmake和gdb的简单的一个流程，此部分转自：[WELEN](https://www.cnblogs.com/welen/articles/4286266.html)

\1. cmake支持gdb的实现，
首先在CMakeLists.txt下加入
SET(CMAKE_BUILD_TYPE "Debug") 
在下面加入：
SET(CMAKE_CXX_FLAGS_DEBUG "*E**N**V**C**X**X**F**L**A**G**S*−*O*0−*W**a**l**l*−*g*−*g**g**d**b*")*S**E**T*(*C**M**A**K**E**C**X**X**F**L**A**G**S**R**E**L**E**A**S**E*"

ENV{CXXFLAGS} -O3 -Wall")
原因是CMake 中有一个变量 CMAKE_BUILD_TYPE ,可以的取值是 Debug Release RelWithDebInfo >和 MinSizeRel。
当这个变量值为 Debug 的时候,CMake 会使用变量 CMAKE_CXX_FLAGS_DEBUG 和 CMAKE_C_FLAGS_DEBUG 中的字符串作为编译选项生成 Makefile;


\2. 在GDB中间加入程序启动参数
比如我们需要调试一个可执行文件./a.out help
这时
$gdb ./a.out
进入到gdb的命令行模式下，然后：
(gdb) set args help
就能加上可执行文件需要的参数，如果要看argc[1]到argc[N]的参数，只需要
(gdb) show args

\3. gdb中查看字符串，地址的操作，数据类型
比始有一个int型的变量i，相要知道他的相关信息，可以
(gdb) print i
打印出变量i的当前值
(gdb)x &i
与上面的命令等价。

如果有x命令看时，需要看一片内存区域，（如果某个地方的值为0，用x时会自动截断了）
(gdb) x/16bx address
单字节16进制打印address地址处的长度为16的空间的内存，16表示空间长度，不是16进制，x表示16进制，b表示byte单字节

gdb看变量是哪个数据类型 
(gdb) whatis i
即可知道i是什么类型的变量

#  gdb调试利器

接下来介绍一下gdb本身以及一些常用的指令，转自：[ Linux Tools Quick Tutorial](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/gdb.html?highlight=gdb)

GDB是一个由GNU开源组织发布的、UNIX/LINUX操作系统下的、基于命令行的、功能强大的程序调试工具。 对于一名Linux下工作的c++程序员，gdb是必不可少的工具；

## 1. 启动gdb

对C/C++程序的调试，需要在编译前就加上-g选项:

```
$g++ -g hello.cpp -o hello
```

调试可执行文件:

```
$gdb <program>
```

program也就是你的执行文件，一般在当前目录下。

调试core文件(core是程序非法执行后core dump后产生的文件):

```
$gdb <program> <core dump file>
$gdb program core.11127
```

调试服务程序:

```
$gdb <program> <PID>
$gdb hello 11127
```

如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试他。program应该在PATH环境变量中搜索得到。

## 2. gdb交互命令

启动gdb后，进入到交互模式，通过以下命令完成对程序的调试；注意高频使用的命令一般都会有缩写，熟练使用这些缩写命令能提高调试的效率；

### 运行

- run：简记为 r ，其作用是运行程序，当遇到断点后，程序会在断点处停止运行，等待用户输入下一步的命令。
- continue （简写c ）：继续执行，到下一个断点处（或运行结束）
- next：（简写 n），单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。
- step （简写s）：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的
- until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
- until+行号： 运行至某行，不仅仅用来跳出循环
- finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
- call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call gdb_test(55)
- quit：简记为 q ，退出gdb

### 设置断点

- - break n （简写b n）:在第n行处设置断点

    （可以带上代码路径和代码名称： b OAGUPDATE.cpp:578）

- b fn1 if a＞b：条件断点设置

- break func（break缩写为b）：在函数func()的入口处设置断点，如：break cb_button

- delete 断点号n：删除第n个断点

- disable 断点号n：暂停第n个断点

- enable 断点号n：开启第n个断点

- clear 行号n：清除第n行的断点

- info b （info breakpoints） ：显示当前程序的断点设置情况

- delete breakpoints：清除所有断点：

### 查看源代码

- list ：简记为 l ，其作用就是列出程序的源代码，默认每次显示10行。
- list 行号：将显示当前文件以“行号”为中心的前后10行代码，如：list 12
- list 函数名：将显示“函数名”所在函数的源代码，如：list main
- list ：不带参数，将接着上一次 list 命令的，输出下边的内容。

### 打印表达式

- print 表达式：简记为 p ，其中“表达式”可以是任何当前正在被测试程序的有效表达式，比如当前正在调试C语言的程序，那么“表达式”可以是任何C语言的有效表达式，包括数字，变量甚至是函数调用。
- print a：将显示整数 a 的值
- print ++a：将把 a 中的值加1,并显示出来
- print name：将显示字符串 name 的值
- print gdb_test(22)：将以整数22作为参数调用 gdb_test() 函数
- print gdb_test(a)：将以变量 a 作为参数调用 gdb_test() 函数
- display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a
- watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a
- whatis ：查询变量或函数
- info function： 查询函数
- 扩展info locals： 显示当前堆栈页的所有变量

### 查询运行信息

- where/bt ：当前运行的堆栈列表；
- bt backtrace 显示当前调用堆栈
- up/down 改变堆栈显示的深度
- set args 参数:指定运行时的参数
- show args：查看设置好的参数
- info program： 来查看程序的是否在运行，进程号，被暂停的原因。

### 分割窗口

- layout：用于分割窗口，可以一边查看代码，一边测试：
- layout src：显示源代码窗口
- layout asm：显示反汇编窗口
- layout regs：显示源代码/反汇编和CPU寄存器窗口
- layout split：显示源代码和反汇编窗口
- Ctrl + L：刷新窗口

注解

交互模式下直接回车的作用是重复上一指令，对于单步调试非常方便；

## 3. 更强大的工具

### cgdb

cgdb可以看作gdb的界面增强版,用来替代gdb的 gdb  -tui。cgdb主要功能是在调试时进行代码的同步显示，这无疑增加了调试的方便性，提高了调试效率。界面类似vi，符合unix/linux下开发人员习惯;如果熟悉gdb和vi，几乎可以立即使用cgdb。

 

# 基础用法

接下来来一个更详细的介绍和使用方法，转自：[大头雨山](https://blog.csdn.net/bigheadyushan/article/details/77828949)

## 1. 简介

GDB（GNU Debugger）是GCC的调试工具。其功能强大，现描述如下： 
 GDB主要帮忙你完成下面四个方面的功能： 
 1.启动你的程序，可以按照你的自定义的要求随心所欲的运行程序。 
 2.可让被调试的程序在你所指定的调置的断点处停住。（断点可以是条件表达式） 
 3.当程序被停住时，可以检查此时你的程序中所发生的事。 
 4.动态的改变你程序的执行环境。

## 2 生成调试信息

一般来说GDB主要调试的是C/C++的程序。要调试C/C++的程序，首先在编译时，我们必须要把调试信息加到可执行文件中。使用编译器（cc/gcc/g++）的 -g 参数可以做到这一点。如：

```Shell
gcc -g hello.c -o hello
g++ -g hello.cpp -o hello
```

- 1
- 2

- 1
- 2

- 1
- 2

如果没有-g，你将看不见程序的函数名、变量名，所代替的全是运行时的内存地址。当你用-g把调试信息加入之后，并成功编译目标代码以后，让我们来看看如何用gdb来调试他。

## 3 启动GDB 的方法

### 1、gdb program

program 也就是你的执行文件，一般在当前目录下。

### 2、gdb program core

用gdb同时调试一个运行程序和core文件，core是程序非法执行后core dump后产生的文件。

### 3、gdb program 1234

如果你的程序是一个服务程序，那么你可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试他。program应该在PATH环境变量中搜索得到。

## 4 程序运行上下文

### 4.1 程序运行参数

```
set args` 可指定运行时参数。（如：set args 10 20 30 40 50 ） 
 `show args` 命令可以查看设置好的运行参数。 
 `run (r)` 启动程序 
 不指定运行参数 `r` 
 指定运行参数`r 10 20 30 40 50
```

### 4.2 工作目录

`cd` 相当于shell的cd命令。 
 `pwd` 显示当前的所在目录。

### 4.3 程序的输入输出

```
info terminal`显示你程序用到的终端的模式。 
 使用重定向控制程序输出。如：`run > outfile` 
 `tty`命令可以设置输入输出使用的终端设备。如：`tty /dev/tty1
```

## 5 设置断点

### 5.1 简单断点

`break` 设置断点，可以简写为b 
 `b 10`设置断点，在源程序第10行 
 `b func`设置断点，在`func`函数入口处

### 5.2 多文件设置断点

在进入指定函数时停住: 
 C++中可以使用 
 `class::function或function(type,type)`格式来指定函数名。如果有名称空间，可以使用`namespace::class::function`或者`function(type,type)`格式来指定函数名。 
 `break filename:linenum` 
 在源文件`filename`的`linenum`行处停住 
 `break filename:function` 
 在源文件`filename`的`function`函数的入口处停住 
 `break class::function`或`function(type,type)` 
 在类class的function函数的入口处停住 
 `break namespace::class::function` 
 在名称空间为namespace的类class的function函数的入口处停住

### 5.3 查询所有断点

```
info b
```

## 6 观察点

`watch` 为表达式（变量）expr设置一个观察点。当表达式值有变化时，马上停住程序。 
 `rwatch`表达式（变量）expr被读时，停住程序。 
 `awatch` 表达式（变量）的值被读或被写时，停住程序。 
 `info watchpoints`列出当前所设置了的所有观察点。

## 7 条件断点

一般来说，为断点设置一个条件，我们使用if关键词，后面跟其断点条件。并且，条件设置好后，我们可以用condition命令来修改断点的条件。并且，条件设置好后，我们可以用condition命令来修改断点的条件。（只有break 和 watch命令支持if，catch目前暂不支持if）。 
 设置一个条件断点 
 `b test.c:8 if intValue == 5` 
 `condition` 与`break if`类似，只是`condition`只能用在已存在的断点上 
 修改断点号为`bnum`的停止条件为`expression` 
 `condition bnum expression` 
 清楚断点号为`bnum`的停止条件 
 `condition bnum` 
 `ignore` 忽略停止条件几次 
 表示忽略断点号为bnum的停止条件count次 
 `Ignore bnum count`

## 8 维护停止点

1. `clear`清除所有的已定义的停止点。
2. `clear function`清除所有设置在函数上的停止点。
3. `clear linenum`清除所有设置在指定行上的停止点。
4. `clear filename:linenum`清除所有设置在指定文件：指定行上的停止点。
5. `delete [breakpoints] [range...]`删除指定的断点，breakpoints为断点号。如果不指定断点号，则表示删除所有的断点。`range`表示断点号的范围（如：3-7）。其简写命令为d。
6. 比删除更好的一种方法是disable停止点，disable了的停止点，GDB不会删除，当你还需要时，enable即可，就好像回收站一样。
7. `disable [breakpoints] [range...]` disable所指定的停止点，breakpoints为停止点号。如果什么都不指定，表示disable所有的停止点。简写命令是dis.
8. `enable [breakpoints] [range...]`enable所指定的停止点，breakpoints为停止点号。
9. `enable [breakpoints] once range…`enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动disable。
10. `enable [breakpoints] delete range…`enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动删除。

## 9 为停止点设定运行命令

我们可以使用GDB提供的command命令来设置停止点的运行命令。也就是说，当运行的程序在被停止住时，我们可以让其自动运行一些别的命令，这很有利行自动化调试。对基于GDB的自动化调试是一个强大的支持。

```
1 commands [bnum]
2 … command-list …
3 end
```

为断点号bnum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。 
 例如：

```
1 break foo if x>0
2 commands
3 printf “x is %d “,x
4 continue
5 end
```

断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是，一旦x的值在foo函数中大于0，GDB会自动打印出x的值，并继续运行程序。 

如果你要清除断点上的命令序列，那么只要简单的执行一下commands命令，并直接在打个end就行了。

## 10 调试代码

1. `run` 运行程序，可简写为r
2. `next` 单步跟踪，函数调用当作一条简单语句执行，可简写为n
3. `step` 单步跟踪，函数调进入被调用函数体内，可简写为s
4. `finish` 退出函数
5. `until` 在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体,可简写为u。
6. `continue` 继续运行程序，可简写为c
7. `stepi`或`si`, `nexti`或`ni` 单步跟踪一条机器指令,一条程序代码有可能由数条机器指令完成，stepi和nexti可以单步执行机器指令。
8. `info program` 来查看程序的是否在运行，进程号，被暂停的原因。

## 11 查看运行时数据

1. `print` 打印变量、字符串、表达式等的值，可简写为p
2. `p count` 打印count的值
3. `p cou1+cou2+cou3` 打印表达式值
4. `print`接受一个表达式，GDB会根据当前的程序运行的数据来计算这个表达式，表达式可以是当前程序运行中的const常量、变量、函数等内容。但是GDB不能使用程序中定义的宏。

## 12 程序变量

在GDB中，你可以随时查看以下三种变量的值： 
 \1. 全局变量（所有文件可见的） 
 \2. 静态全局变量（当前文件可见的） 
 \3. 局部变量（当前Scope可见的） 
 如果你的局部变量和全局变量发生冲突（也就是重名），一般情况下是局部变量会隐藏全局变量，也就是说，如果一个全局变量和一个函数中的局部变量同名时，如果当前停止点在函数中，用print显示出的变量的值会是函数中的局部变量的值。如果此时你想查看全局变量的值时，你可以使用“::”操作符： 
 `file::variable` 
 `function::variable` 
 可以通过这种形式指定你所想查看的变量，是哪个文件中的或是哪个函数中的。例如，查看文件f2.c中的全局变量x的值：

```
1 p ‘f2.c’::x
```

当然，“::”操作符会和C++中的发生冲突，GDB能自动识别“::”是否C++的操作符，所以你不必担心在调试C++程序时会出现异常。 
 \4. 数组变量 
 有时候，你需要查看一段连续的内存空间的值。比如数组的一段，或是动态分配的数据的大小。你可以使用GDB的“@”操作符，“@”的左边是第一个内存的地址的值，“@”的右边则你你想查看内存的长度。例如，你的程序中有这样的语句：

```
1 int *array = (int *) malloc (len * sizeof (int));
```

于是，在GDB调试过程中，你可以以如下命令显示出这个动态数组的取值：

```
1 p *array@len
```

@的左边是数组的首地址的值，也就是变量array所指向的内容，右边则是数据的长度，其保存在变量len中。

## 13 自动显示

你可以设置一些自动显示的变量，当程序停住时，或是在你单步跟踪时，这些变量会自动显示。相关的GDB命令是display。

```
1 display expr
2 display/fmt expr
3 display/fmt addr
```

expr是一个表达式，fmt表示显示的格式，addr表示内存地址，当你用display设定好了一个或多个表达式后，只要你的程序被停下来，GDB会自动显示你所设置的这些表达式的值。

```
1 info display
```

查看display设置的自动显示的信息。

```
1 undisplay dnums…
2 delete display dnums…
```

删除自动显示，dnums意为所设置好了的自动显式的编号。如果要同时删除几个，编号可以用空格分隔，如果要删除一个范围内的编号，可以用减号表示（如：2-5）

```
1 disable display dnums…
2 enable display dnums…
```

disable和enalbe不删除自动显示的设置，而只是让其失效和恢复。

## 14 历史记录

当你用GDB的print查看程序运行时的数据时，你每一个print都会被GDB记录下来。GDB会以2, 1。这个功能所带来的好处是，如果你先前输入了一个比较长的表达式，如果你还想查看这个表达式的值，你可以使用历史记录来访问，省去了重复输入。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 (gdb)show values
 2 Print the last ten values in the value history, with their item numbers. This is
 3 like ‘p $$9’ repeated ten times, except that show values does not change the
 4 history.
 5 
 6 (gdb)show values n
 7 Print ten history values centered on history item number n.
 8 
 9 (gdb)show values +
10 Print ten history values just after the values last printed. If no more values are
11 available, show values + produces no display.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 15 改变程序的执行

一旦使用GDB挂上被调试程序，当程序运行起来后，你可以根据自己的调试思路来动态地在GDB中更改当前被调试程序的运行线路或是其变量的值，这个强大的功能能够让你更好的调试你的程序，比如，你可以在程序的一次运行中走遍程序的所有分支。

### 15.1 修改变量值

修改被调试程序运行时的变量值，在GDB中很容易实现，使用GDB的print命令即可完成。如：

```
1 (gdb) print x=4
```

x=4这个表达式是C/C++的语法，意为把变量x的值修改为4，如果你当前调试的语言是Pascal，那么你可以使用Pascal的语法：x:=4。 
 在某些时候，很有可能你的变量和GDB中的参数冲突，如：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 (gdb) whatis width
2 type = double
3 (gdb) p width
4 $4 = 13
5 (gdb) set width=47
6 Invalid syntax in expression.
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

因为，set width是GDB的命令，所以，出现了“Invalid syntax in expression”的设置错误，此时，你可以使用set var命令来告诉GDB，width不是你GDB的参数，而是程序的变量名，如：

```
1 (gdb) set var width=47
```

另外，还可能有些情况，GDB并不报告这种错误，所以保险起见，在你改变程序变量取值时，最好都使用set var格式的GDB命令。

### 15.2 跳转执行

一般来说，被调试程序会按照程序代码的运行顺序依次执行。GDB提供了乱序执行的功能，也就是说，GDB可以修改程序的执行顺序，可以让程序执行随意跳跃。这个功能可以由GDB的jump命令来完：

```
1 jump linespec
```

指定下一条语句的运行点。可以是文件的行号，可以是file:line格式，可以是+num这种偏移量格式。表示下一条运行语句从哪里开始。

```
1 jump *address
```

这里的是代码行的内存地址。 
 注意，jump命令不会改变当前的程序栈中的内容，所以，当你从一个函数跳到另一个函数时，当函数运行完返回时进行弹栈操作时必然会发生错误，可能结果还是非常奇怪的，甚至于产生程序Core Dump。所以最好是同一个函数中进行跳转。 
 熟悉汇编的人都知道，程序运行时，eip寄存器用于保存当前代码所在的内存地址。所以，jump命令也就是改变了这个寄存器中的值。于是，你可以使用“set $pc”来更改跳转执行的地址。如：

```
1 set $pc = 0×485
```

### 15.3 产生信号量

使用singal命令，可以产生一个信号量给被调试的程序。如：中断信号Ctrl+C。这非常方便于程序的调试，可以在程序运行的任意位置设置断点，并在该断点用GDB产生一个信号量，这种精确地在某处产生信号非常有利程序的调试。 
 语法是：

```
1 signal signal
```

UNIX的系统信号量通常从1到15。所以取值也在这个范围。 
 single命令和shell的kill命令不同，系统的kill命令发信号给被调试程序时，是由GDB截获的，而single命令所发出一信号则是直接发给被调试程序的。

### 15.4 强制函数返回

如果你的调试断点在某个函数中，并还有语句没有执行完。你可以使用return命令强制函数忽略还没有执行的语句并返回。

```
1 return
2 return expression
```

使用return命令取消当前函数的执行，并立即返回，如果指定了，那么该表达式的值会被认作函数的返回值。

### 15.5 强制调用函数

```
1 call expr
```

表达式中可以一是函数，以此达到强制调用函数的目的。并显示函数的返回值，如果函数返回值是void，那么就不显示。

```
1 print expr
```

另一个相似的命令也可以完成这一功能——print，print后面可以跟表达式，所以也可以用他来调用函数，print和call的不同是，如果函数返回void，call则不显示，print则显示函数返回值，并把该值存入历史数据中。

## 16 显示源代码

GDB 可以打印出所调试程序的源代码，当然，在程序编译时一定要加上 –g  的参数，把源程序信息编译到执行文件中。不然就看不到源程序了。当程序停下来以后，  GDB会报告程序停在了那个文件的第几行上。你可以用list命令来打印程序的源代码。默认打印10行，还是来看一看查看源代码的GDB命令吧。

```
1 (gdb)list linenum
2 Print lines centered around line number linenum in the current source file.
3 (gdb)list function
```

显示函数名为function的函数的源程序。

```
1 list
```

显示当前行后面的源程序。

```
1 list -
```

显示当前行前面的源程序。 
 一般是打印当前行的上5行和下5行，如果显示函数是是上2行下8行，默认是10行，当然，你也可以定制显示的范围，使用下面命令可以设置一次显示源程序的行数。

```
1 set listsize count
```

设置一次显示源代码的行数。(unless the list argument explicitly specifies some other number)

```
1 show listsize
```

查看当前listsize的设置。

## 17 调试已运行的进程

两种方法： 
 1、在UNIX下用ps查看正在运行的程序的PID（进程ID），然后用gdb PID process-id 格式挂接正在运行的程序。 
 2、先用gdb 关联上源代码，并进行gdb，在gdb中用attach process-id 命令来挂接进程的PID。并用detach来取消挂接的进程。

## 18 线程

如果你程序是多线程的话，你可以定义你的断点是否在所有的线程上，或是在某个特定的线程。GDB很容易帮你完成这一工作。

```
1 break linespec thread threadno
2 break linespec thread threadno if …
```

linespec指定了断点设置在的源程序的行号。threadno指定了线程的ID，注意，这个ID是GDB分配的，你可以通过“info  threads”命令来查看正在运行程序中的线程信息。如果你不指定‘thread threadno  ’则表示你的断点设在所有线程上面。你还可以为某线程指定断点条件。如：

```
1 (gdb) break frik.c:13 thread 28 if bartab > lim
```

当你的程序被GDB停住时，所有的运行线程都会被停住。这方便你你查看运行程序的总体情况。而在你恢复程序运行时，所有的线程也会被恢复运行。那怕是主进程在被单步调试时。

## 19 查看栈信息

当程序被停住了，你需要做的第一件事就是查看程序是在哪里停住的。当你的程序调用了一个函数，函数的地址，函数参数，函数内的局部变量都会被压入“栈”（Stack）中。你可以用GDB命令来查看当前的栈中的信息。 
 下面是一些查看函数调用栈信息的GDB命令： 
 breacktrace,简称bt 
 打印当前的函数调用栈的所有信息。如：

```
1 (gdb) bt
2 #0 func (n=250) at tst.c:6
3 #1 0x08048524 in main (argc=1, argv=0xbffff674) at tst.c:30
4 #2 0x400409ed in __libc_start_main () from /lib/libc.so.6
```

从上可以看出函数的调用栈信息：`__libc_start_main –> main() –> func()`

```
1 backtrace n
2 bt n
```

n是一个正整数，表示只打印栈顶上n层的栈信息。

```
1 backtrace -n
2 bt -n
```

-n表一个负整数，表示只打印栈底下n层的栈信息。

如果你要查看某一层的信息，你需要在切换当前的栈，一般来说，程序停止时，最顶层的栈就是当前栈，如果你要查看栈下面层的详细信息，首先要做的是切换当前栈。

```
1 frame n
```

n是一个从0开始的整数，是栈中的层编号。比如：frame 0，表示栈顶，frame 1，表示栈的第二层。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 (gdb)frame addr
2 f addr Select the frame at address addr. This is useful mainly if the chaining of stack frames has been damaged by a bug, making it impossible for gdb to assign
3 
4 numbers properly to all frames. In addition, this can be useful when your program has multiple stacks and switches between them.
5 
6 (gdb)up n
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

表示向栈的上面移动n层，可以不打n，表示向上移动一层。

```
1 down n
```

表示向栈的下面移动n层，可以不打n，表示向下移动一层。 
 上面的命令，都会打印出移动到的栈层的信息。如果你不想让其打出信息。你可以使用这三个命令： 
 `select-frame` 对应于 frame 命令。 
 `up-silently n` 对应于 up 命令。 
 `down-silently n` 对应于 down 命令。 
 查看当前栈层的信息，你可以用以下GDB命令：

```
1 frame 或 f
```

会打印出这些信息：栈的层编号，当前的函数名，函数参数值，函数所在文件及行号，函数执行到的语句。

```
1 info frame
2 info f
```

## 20 信号

信号是一种软中断，是一种处理异步事件的方法。一般来说，[操作系统](http://lib.csdn.net/base/operatingsystem)都支持许多信号。尤其是UNIX，比较重要应用程序一般都会处理信号。UNIX定义了许多信号，比如SIGINT表示中断字符信号，也就是Ctrl+C的信号，SIGBUS表示硬件故障的信号；SIGCHLD表示子进程状态改变信号； SIGKILL表示终止程序运行的信号，等等。

调试程序的时候处理信号:

```
1 handle signal [keywords...]
```

signal可以以SIG开头或不以SIG开头，可以用定义一个要处理信号的范围（如：SIGIO-SIGKILL，表示处理从  SIGIO信号到SIGKILL的信号，其中包括SIGIO，SIGIOT，SIGKILL三个信号），也可以使用关键字all来标明要处理所有的信号。一旦被调试的程序接收到信号，运行程序马上会被GDB停住，以供调试。 
 keywords列表如下:

```
1 nostop
```

当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号。

```
1 stop
```

当被调试的程序收到信号时，GDB会停住你的程序。This implies the print keyword as well.

```
1 print
```

当被调试的程序收到信号时，GDB会显示出一条信息。

```
1 noprint
```

当被调试的程序收到信号时，GDB不会告诉你收到信号的信息。This implies the nostop keyword as well.

```
1 pass
2 noignore
```

当被调试的程序收到信号时，GDB不处理信号。这表示，GDB会把这个信号交给被调试程序处理 or else it may terminate if the signal is fatal and not handled.

```
1 nopass
2 ignore
```

当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号。

```
1 info signals
2 info handle
```

查看有哪些信号在被GDB检测中。

## 21catch

当event发生时，停住程序。event可以是下面的内容： 
 1、throw 一个C++抛出的异常。（throw为关键字） 
 2、catch 一个C++捕捉到的异常。（catch为关键字）

## 22 指定源文件的路径

某些时候，用-g编译过后的执行程序中只是包括了源文件的名字，没有路径名。GDB提供了可以让你指定源文件的路径的命令，以便GDB进行搜索。

```
1 Directory dirname …
2 dir dirname …
```

加一个源文件路径到当前路径的前面。如果你要指定多个路径，UNIX下你可以使用“:”，Windows下你可以使用“;”。

```
1 directory
```

清除所有的自定义的源文件搜索路径信息。

```
show directories
```

显示定义了的源文件搜索路径。

# gdb高级用法

## 一：列文件清单

### 1．list（l）

```
1 (gdb)   list   line1,line2
```

## 二：执行程序

要想运行准备调试的程序，可使用run(r)命令，在它后面可以跟随发给该程序的任何参数，包括标准输入和标准输出说明符(<和>)和外壳通配符（*、？、[、]）在内。 
 如果你使用不带参数的run命令，gdb就再次使用你给予前一条run命令的参数，这是很有用的。 
 利用`set args`命令就可以修改发送给程序的参数，而使用show args 命令就可以查看其缺省参数的列表。

```
1 (gdb)set args –b –x
2 (gdb)show args
3 (gdb)backtrace(bt)  //命令为堆栈提供向后跟踪功能。     
4 (gdb)Backtrace       //命令产生一张列表，包含着从最近的过程开始的所以有效过程和调用这些过程的参数。     
```

## 三：显示数据

### 1. print(p) 命令可以检查各个变量的值。

```
1 (gdb)   print   p   //(p为变量名)     
```

### 2. whatis 命令可以显示某个变量的类型

```
1 (gdb)   whatis   p     
2 type   =   int   *     
```

print 是gdb的一个功能很强的命令，利用它可以显示被调试的语言中任何有效的表达式。表达式除了包含你程序中的变量外，还可以包含以下内容：

### 2.1 对程序中函数的调用

```
1 (gdb)print   find_entry(1,0)     
```

### 2.2 数据结构和其他复杂对象

```
(gdb)print   *table_start
$8={e=reference=’\000’,location=0x0,next=0x0} 
```

- 1
- 2

- 1
- 2

- 1
- 2

###  2.3 值的历史成分

```
(gdb)print   $1</span>  //(<span class="hljs-variable" style="color: rgb(102, 0, 102); box-sizing: border-box;">$1为历史记录变量,在以后可以直接引用   $1   的值)     
```

- 1

- 1

- 1

###  2.4 人为数组

人为数组提供了一种去显示存储器块（数组节或动态分配的存储区）内容的方法。早期的调试程序没有很好的方法将任意的指针换成一个数组。就像对待参数一样，让我们查看内存中在变量h后面的10个整数，一个动态数组的语法如下所示：

```
base@length
```

- 1

- 1

- 1

因此，要想显示在h后面的10个元素，可以使用h@10：

```
(gdb)print   h@10     
$13=(-1,345,23,-234,0,0,0,98,345,10)     
```

- 1
- 2

- 1
- 2

- 1
- 2

##  四：断点(breakpoint)

###  1. break命令（可以简写为b）

可以用来在调试的程序中设置断点，该命令有如下四种形式：

```
1. break   line-number   使程序恰好在执行给定行之前停止。
2. break   function-name   使程序恰好在进入指定的函数之前停止。
3. break   line-or-function   if   condition   如果condition（条件）是真，程序到达指定行或函数时停止。
4. break   routine-name   在指定例程的入口处设置断点     
```

- 1
- 2
- 3
- 4

- 1
- 2
- 3
- 4

- 1
- 2
- 3
- 4

如果该程序是由很多原文件构成的，你可以在各个原文件中设置断点，而不是在当前的原文件中设置断点，其方法如下：

```
(gdb)   break   filename:line-number
(gdb)   break   filename:function-name
```

- 1
- 2

- 1
- 2

- 1
- 2

要想设置一个条件断点，可以利用break if命令，如下所示：

```
(gdb)   break   line-or-function   if   expr
```

- 1

- 1

- 1

例：

```
  (gdb)   break   46   if   testsize==100
```

- 1

- 1

- 1

从断点继续运行：countinue(c) 命令

##  五．断点的管理

###  1．显示当前gdb的断点信息：

```
(gdb)   info   break
```

- 1

- 1

- 1

他会以如下的形式显示所有的断点信息：

```
  Num   Type   Disp   Enb   Address   What     
  1   breakpoint   keep   y   0x000028bc   in   init_random   at   qsort2.c:155     
  2   breakpoint   keep   y   0x0000291c   in   init_organ   at   qsort2.c:168     
  (gdb)
```

- 1
- 2
- 3
- 4

- 1
- 2
- 3
- 4

- 1
- 2
- 3
- 4

###  2.删除指定的某个断点：

```
(gdb)   delete   breakpoint   1
```

- 1

- 1

- 1

该命令将会删除编号为1的断点，如果不带编号参数，将删除所有的断点

```
(gdb)   delete   breakpoint
```

- 1

- 1

- 1

###  3.禁止使用某个断点

```
(gdb)   disable   breakpoint   1
```

- 1

- 1

- 1

该命令将禁止断点 1,同时断点信息的 (Enb)域将变为 n

###  4．允许使用某个断点

```
(gdb)   enable   breakpoint   1
```

- 1

- 1

- 1

该命令将允许断点 1,同时断点信息的 (Enb)域将变为 y

###  5．清除原文件中某一代码行上的所有断点

```
(gdb)clean   number
```

- 1

- 1

- 1

注：number 为原文件的某个代码行的行号 
 `Clear`清除所有的已定义的停止点。 
 `clear <function>`或`clear <filename:function>`清除所有设置在函数上的停止点。 
 `clear <linenum>`或`clear <filename:linenum>`清除所有设置在指定行上的停止点。

## 六．变量的检查和赋值

1. whatis:识别数组或变量的类型
2. ptype:比whatis的功能更强，他可以提供一个结构的定义
3. set variable:将值赋予变量
4. print 除了显示一个变量的值外，还可以用来赋值

## 七．单步执行

1. next(n) 不进入的单步执行
2. step(s) 进入的单步执行
3. finish 退出该函数返回到它的调用函数中

## 八．函数的调用

1. call name 调用和执行一个函数

```
  (gdb)   call   gen_and_sork(   1234,1,0   )     
  (gdb)   call   printf(“abcd”)     
  $1=4     
```

- 1
- 2
- 3

- 1
- 2
- 3

- 1
- 2
- 3

##  九．机器语言工具

有一组专用的gdb变量可以用来检查和修改计算机的通用寄存器，gdb提供了目前每一台计算机中实际使用的4个寄存器的标准名字： 
 \1. *p**c*：程序计数器2.

fp： 帧指针（当前堆栈帧） 
 \3. *s**p*：栈指针4.

ps： 处理器状态

## 十．信号

gdb通常可以捕捉到发送给它的大多数信号，通过捕捉信号，它就可决定对于正在运行的进程要做些什么工作。例如，按CTRL-C将中断信号发送给gdb，通常就会终止gdb。但是你或许不想中断gdb，真正的目的是要中断gdb正在运行的程序，因此，gdb要抓住该信号并停止它正在运行的程序，这样就可以执行某些调试操作。

Handle命令可控制信号的处理，他有两个参数，一个是信号名，另一个是接受到信号时该作什么。几种可能的参数是： 
 \1. nostop 接收到信号时，不要将它发送给程序，也不要停止程序。 
 \2. stop 接受到信号时停止程序的执行，从而允许程序调试；显示一条表示已接受到信号的消息（禁止使用消息除外） 
 \3. print 接受到信号时显示一条消息 
 \4. noprint 接受到信号时不要显示消息（而且隐含着不停止程序运行） 
 \5. pass 将信号发送给程序，从而允许你的程序去处理它、停止运行或采取别的动作。 
 \6. nopass 停止程序运行，但不要将信号发送给程序。

例如，假定你截获SIGPIPE信号，以防止正在调试的程序接受到该信号，而且只要该信号一到达，就要求该序停止，并通知你。要完成这一任务，可利用如下命令：

```
(gdb)   handle   SIGPIPE   stop   print
```

- 1

- 1

- 1

请注意，UNIX的信号名总是采用大写字母！你可以用信号编号替代信号名 
 如果你的程序要执行任何信号处理操作，就需要能够[测试](http://lib.csdn.net/base/softwaretest)其信号处理程序，为此，就需要一种能将信号发送给程序的简便方法，这就是signal命令的任务。该 命令的参数是一个数字或者一个名字，如SIGINT。假定你的程序已将一个专用的SIGINT（键盘输入，或CTRL-C；信号2）信号处理程序设置成采 取某个清理动作，要想测试该信号处理程序，你可以设置一个断点并使用如下命令：

```
（gdb）   signal   2
 continuing   with   signal   SIGINT(2)
```

- 1
- 2

- 1
- 2

- 1
- 2

该程序继续执行，但是立即传输该信号，而且处理程序开始运行.

##  十一. 原文件的搜索

1. search text:该命令可显示在当前文件中包含text串的下一行。
2. Reverse-search text:该命令可以显示包含text 的前一行。

##  十二.UNIX接口

shell 命令可启动UNIX外壳，CTRL-D退出外壳，返回到 gdb.

##  十三.命令的历史

为了允许使用历史命令，可使用 set history expansion on 命令

```
(gdb)   set   history   expansion   on
```

- 1

- 1

- 1

小结：常用的gdb命令

```
  backtrace   //显示程序中的当前位置和表示如何到达当前位置的栈跟踪（同义词：where）     
  breakpoint  //在程序中设置一个断点     
  cd   //改变当前工作目录     
  clear   //删除刚才停止处的断点     
  commands   //命中断点时，列出将要执行的命令     （#add相当于vs的when hit）
  continue   //从断点开始继续执行     
  delete   //删除一个断点或监测点；也可与其他命令一起使用     
  display   //程序停止时显示变量和表达时     
  down   //下移栈帧，使得另一个函数成为当前函数     
  frame   //选择下一条continue命令的帧     
  info   //显示与该程序有关的各种信息     
  jump   //在源程序中的另一点开始运行     
  kill   //异常终止在gdb   控制下运行的程序     
  list   //列出相应于正在执行的程序的原文件内容     
  next   //执行下一个源程序行，从而执行其整体中的一个函数     
  print   //显示变量或表达式的值     
  pwd   //显示当前工作目录     
  ptype   //显示一个数据结构（如一个结构或C++类）的内容     
  quit   //退出gdb     
  reverse-search   //在源文件中反向搜索正规表达式     
  run   //执行该程序     
  search   //在源文件中搜索正规表达式     
  set   variable   //给变量赋值     
  signal   //将一个信号发送到正在运行的进程     
  step   //执行下一个源程序行，必要时进入下一个函数     
  undisplay   //display命令的反命令，不要显示表达式     
  until   //结束当前循环     
  up   //上移栈帧，使另一函数成为当前函数     
  watch   //在程序中设置一个监测点（即数据断点）     
  whatis   //显示变量或函数类型    
```

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- 11
- 12
- 13
- 14
- 15
- 16
- 17
- 18
- 19
- 20
- 21
- 22
- 23
- 24
- 25
- 26
- 27
- 28
- 29
- 30

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- 11
- 12
- 13
- 14
- 15
- 16
- 17
- 18
- 19
- 20
- 21
- 22
- 23
- 24
- 25
- 26
- 27
- 28
- 29
- 30

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- 11
- 12
- 13
- 14
- 15
- 16
- 17
- 18
- 19
- 20
- 21
- 22
- 23
- 24
- 25
- 26
- 27
- 28
- 29
- 30

#  gdb - GNU 调试器

提要 
 gdb [-help] [-nx] [-q] [-batch] [-cd=dir] [-f] [-b bps] 
 [-tty=dev] [-s symfile] [-e prog] [-se prog] [-c 
 core] [-x cmds] [-d dir] [prog[core|procID]]

描述 
 调试器（如GDB）的目的是允许你在程序运行时进入到某个程序内部去看看该程序在做什么，或者在该程序崩溃时它在做什么。

```
    GDB主要可以做4大类事（加上一些其他的辅助工作），以帮助用户在程序运行过程中发现bug。

      o  启动您的程序，并列出可能会影响它运行的一些信息
      o  使您的程序在特定条件下停止下来
      o  当程序停下来的时候，检查发生了什么
      o  对程序做出相应的调整，这样您就能尝试纠正一个错误并继续发现其它错误

    您能使用GDB调试用C、C++、Modula-2写的程序。等GNU Fortran编译器准备好过后，GDB将提供对Fortran的支持

    GDB通过在命令行方式下输入gdb来执行。启动过后，GDB会从终端读取命令，直到您输入GDB命令quit使GDB退出。您能通过GDB命令help获取在线帮助。

    您能以无参数无选项的形式运行GDB，不过通常的情况是以一到两个参数运行GDB，以待调试的可执行程序名为参数
    gdb 程序名

    您能用两个参数来运行GDB，可执行程序名与core文件(译注：不知道怎么翻译好，就不翻译了)。
    gdb 程序名 core

    您可以以进程ID作为第二个参数，以调式一个正在运行的进程
    gdb 程序名 1234
    将会把gdb附在进程1234之上(除非您正好有个文件叫1234，gdb总是先查找core文件)

   下面是一些最常用的GDB命令:

   file [filename]
          装入想要调试的可执行文件

   kill [filename]
          终止正在调试的程序

   break [file:]function
          在(file文件的)function函数中设置一个断点

   clear
          删除一个断点，这个命令需要指定代码行或者函数名作为参数

   run [arglist]
          运行您的程序 (如果指定了arglist,则将arglist作为参数运行程序)

   bt Backtrace: 显示程序堆栈信息

   print expr
          打印表达式的值

   continue
          继续运行您的程序 (在停止之后，比如在一个断点之后)

   list
          列出产生执行文件的源代码的一部分

   next
          单步执行 (在停止之后); 跳过函数调用

   nexti
          执行下一行的源代码中的一条汇编指令

   set
          设置变量的值。例如：set nval=54 将把54保存到nval变量中

   step
          单步执行 (在停止之后); 进入函数调用

   stepi
          继续执行程序下一行源代码中的汇编指令。如果是函数调用，这个命令将进入函数的内部，单步执行函数中的汇编代码

   watch
          使你能监视一个变量的值而不管它何时被改变

   rwatch
          指定一个变量，如果这个变量被读，则暂停程序运行，在调试器中显示信息，并等待下一个调试命令。参考rwatch和watch命令

   awatch
          指定一个变量，如果这个变量被读或者被写，则暂停程序运行，在调试器中显示信息，并等待下一个调试命令。参考rwatch和watch命令

   Ctrl-C
          在当前位置停止执行正在执行的程序，断点在当前行

   disable
          禁止断点功能，这个命令需要禁止的断点在断点列表索引值作为参数

   display
          在断点的停止的地方，显示指定的表达式的值。(显示变量)

   undisplay
          删除一个display设置的变量显示。这个命令需要将display list中的索引做参数

   enable
          允许断点功能，这个命令需要允许的断点在断点列表索引值作为参数

   finish
          继续执行，直到当前函数返回

   ignore
          忽略某个断点制定的次数。例：ignore 4 23 忽略断点4的23次运行，在第24次的时候中断

   info [name]
          查看name信息

   load
          动态载入一个可执行文件到调试器

   xbreak
          在当前函数的退出的点上设置一个断点

   whatis
          显示变量的值和类型

   ptype
          显示变量的类型

   return
          强制从当前函数返回

   txbreak
          在当前函数的退出的点上设置一个临时的断点(只可使用一次)

   make
          使你能不退出 gdb 就可以重新产生可执行文件

   shell
          使你能不离开 gdb 就执行 UNIX shell 命令

   help [name]
          显示GDB命令的信息，或者显示如何使用GDB的总体信息

   quit
          退出gdb.


   要得到所有使用GDB的资料，请参考Using GDB: A Guide to the GNU
   Source-Level  Debugger,  by Richard M. Stallman and Roland
   H. Pesch.  当用info查看的时候，也能看到相同的文章
```

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- 11
- 12
- 13
- 14
- 15
- 16
- 17
- 18
- 19
- 20
- 21
- 22
- 23
- 24
- 25
- 26
- 27
- 28
- 29
- 30
- 31
- 32
- 33
- 34
- 35
- 36
- 37
- 38
- 39
- 40
- 41
- 42
- 43
- 44
- 45
- 46
- 47
- 48
- 49
- 50
- 51
- 52
- 53
- 54
- 55
- 56
- 57
- 58
- 59
- 60
- 61
- 62
- 63
- 64
- 65
- 66
- 67
- 68
- 69
- 70
- 71
- 72
- 73
- 74
- 75
- 76
- 77
- 78
- 79
- 80
- 81
- 82
- 83
- 84
- 85
- 86
- 87
- 88
- 89
- 90
- 91
- 92
- 93
- 94
- 95
- 96
- 97
- 98
- 99
- 100
- 101
- 102
- 103
- 104
- 105
- 106
- 107
- 108
- 109
- 110
- 111
- 112
- 113
- 114
- 115
- 116
- 117
- 118
- 119
- 120
- 121
- 122
- 123
- 124
- 125
- 126
- 127
- 128
- 129
- 130
- 131
- 132

选项 
 任何参数而非选项指明了一个可执行文件及core 文件(或者进程ID)；所 
 遇到的第一个未关联选项标志的参数与 ‘-se’ 选项等价，第二个，如果存 
 在，且是一个文件的名字，则等价与 ‘-c’ 选项。许多选项都有一个长格式 
 与短格式；都会在这里表示出来。如果你把一个长格式截短，只要不引起歧 
 义，那么它还是可以被识别。(如果你愿意，你可以使用 ‘+’ 而非 ‘-’ 标 
 记选项参数，不过我们在例子中仍然遵从通常的惯例)

```
    -help

   -h     列出所有选项，并附简要说明。  

   -symbols=file

   -s file
          读出文件（file）中的符号表。

   -write 
          开通(enable)往可执行文件和核心文件写的权限。

   -exec=file

   -e file
          在适当时候把File作为可执行的文件执行，来检测与core dump结合的数据。
   －se File
          从File读取符号表并把它作为可执行文件。
   －core File
   -c File
          把File作为core dump来执行。
   －command=File
   -x File
          从File中执行GDB命令。
   －directory=Directory
   -d Directory
          把Dicrctory加入源文件搜索的路径中。
   -nx
   -n
          不从任何.gdbinit初始化文件中执行命令。通常情况下，这些文件中的命令是在所有命令选项和参数处理完后才执行。
   -quiet
   -q
          "Quiet".不输入介绍和版权信息。这些信息输出在batch模式下也被关闭。
   -batch
          运行batch模式。在处理完所有用'-x'选项指定的命令文件(还有'.gdbi-nit',如果没禁用)后退出，并返回状态码0.如果在命令文件中的命令被
```

- 1
- 2
- 3
- 4
- 5
- 6
- 7
- 8
- 9
- 10
- 11
- 12
- 13
- 14
- 15
- 16
- 17
- 18
- 19
- 20
- 21
- 22
- 23
- 24
- 25
- 26
- 27
- 28
- 29
- 30
- 31
- 32
- 33
- 34
- 35
- 36

执行时发生错误，则退出，并返回状态码非0.batch模式对于运行GDB作为过滤器也许很有用,比如要从另一台电脑上下载并运行一个程序;为了让这些更有用,当 
 在batch模式下运行时,消息:Program exited normally.(不论什么时候,一个程序在GDB控制下终止运行,这条消息都会正常发出.),将不会发出. 
 -cd=Directory 
 运行GDB,使用Directory作为它的工作目录,取代当前工作目录. 
 -fullname 
 -f 
 当Emacs让GDB作为一个子进程运行时,设置这个选项.它告诉GDB每当一个堆栈结构(栈帧)显示出来(包括每次程序停止)就用标准的,认同的方式 
 输出文件全名和行号.这里,认同的格式看起来像两个’ 32’字符,紧跟文件名,行号和字符位置(由冒号,换行符分隔).Emacs同GDB的接口程序使用这两个’ 32’字 
 符作为一个符号为框架来显示源代码. 
 -b Bps 
 设置行速(波特率或bits/s).在远程调试中GDB在任何串行接口中使用的行速. 
 -tty=Device 
 使用Device作为你程序运行的标准输入输出.

gdb主要调试的是C/C++的程序。要调试C/C++的程序，首先在编译时，必须要把调试信息加到可执行文件中。使用编译器(cc/gcc/g++)的 -g 参数即可。如：

[david@DAVID david]$ gcc -g hello.c -o hello

[david@DAVID david]$ g++ -g hello.cpp -o hello

如果没有-g，将看不见程序的函数名和变量名，代替它们的全是运行时的内存地址。当用-g把调试信息加入，并成功编译目标代码以后，看看如何用gdb来调试。

启动gdb的方法有以下几种：

1. gdb 
    program也就是执行文件，一般在当前目录下。
2. gdb core 
    用gdb同时调试一个运行程序和core文件，core是程序非法执行后，core dump后产生的文件。
3. gdb 
    如果程序是一个服务程序，那么可以指定这个服务程序运行时的进程ID。gdb会自动attach上去，并调试它。program应该在PATH环境变量中搜索得到。

●在[Linux](http://lib.csdn.net/base/linux)下用ps(第一章已经对ps作了介绍)查看正在运行的程序的PID(进程ID)，然后用gdb PID格式挂接正在运行的程序。

●先用gdb 关联上源代码，并进行gdb，在gdb中用attach命令来挂接进程的PID，并用detach来取消挂接的进程。

gdb启动时，可以加上一些gdb的启动开关，详细的开关可以用gdb -help查看。下面只列举一些比较常用的参数：

-symbols

-s

从指定文件中读取符号表。

-se file

从指定文件中读取符号表信息，并把它用在可执行文件中。

-core

-c

调试时core dump的core文件。

-directory

-d

加入一个源文件的搜索路径。默认搜索路径是环境变量中PATH所定义的路径。

4.1.1 gdb的命令概貌 
 gdb的命令很多，gdb将之分成许多种类。help命令只是列出gdb的命令种类，如果要看其中的命令，可以使用help 命令。如：

(gdb) help data

也可以直接用help [command]来查看命令的帮助。

gdb中，输入命令时，可以不用输入全部命令，只用输入命令的前几个字符就可以了。当然，命令的前几个字符应该标志着一个惟一的命令，在[linux](http://lib.csdn.net/base/linux)下，可以按两次TAB键来补齐命令的全称，如果有重复的，那么gdb会把它全部列出来。

示例一：在进入函数func时，设置一个断点。可以输入break func，或是直接输入b func。

(gdb) b func

Breakpoint 1 at 0x804832e: file test.c, line 5.

(gdb)

示例二：输入b按两次TAB键，你会看到所有b开头的命令。

(gdb) b

backtrace break bt

要退出gdb时，只用输入quit或其简称q就行了。

4.1.2 gdb中运行Linux的shell程序 
 在gdb环境中，可以执行Linux的shell命令：

shell

调用Linux的shell来执行，环境变量SHELL中定义的Linux的shell将会用来执行。如果SHELL没有定义，那就使用Linux的标准shell：/bin/sh(在Windows中使用Command.com或cmd.exe)。

还有一个gdb命令是make：

make

可以在gdb中执行make命令来重新build自己的程序。这个命令等价于shell make 。

4.1.3 在gdb中运行程序 
 当以gdb 方式启动gdb后，gdb会在PATH路径和当前目录中搜索的源文件。如要确认gdb是否读到源文件，可使用l或list命令，看看gdb是否能列出源代码。

在gdb中，运行程序使用r或是run命令。程序的运行，有可能需要设置下面四方面的事。

1. 程序运行参数 
    set args 可指定运行时参数。如：

set args 10 20 30 40 50

show args 命令可以查看设置好的运行参数。

1. 运行环境 
    path 可设定程序的运行路径。

show paths 查看程序的运行路径。

set environment varname [=value] 设置环境变量。如：

set env USER=hchen

show environment [varname] 查看环境变量。

1. 工作目录 
    cd 相当于shell的cd命令。

pwd 显示当前的所在目录。

1. 程序的输入输出 
    info terminal 显示程序用到的终端的模式。

使用重定向控制程序输出。如：

run > outfile

tty命令可以指写输入输出的终端设备。如：

tty /dev/ttyb

4.1.4 调试已运行的程序 
 调试已经运行的程序有两种方法：

● 在Linux下用ps(第一章已经对ps作了介绍)查看正在运行的程序的PID(进程ID)，然后用gdb PID格式挂接正在运行的程序。

● 先用gdb 关联上源代码，并进行gdb，在gdb中用attach命令来挂接进程的PID，并用detach来取消挂接的进程。

4.1.5 暂停/恢复程序运行 
 调试程序中，暂停程序运行是必需的，gdb可以方便地暂停程序的运行。可以设置程序在哪行停住，在什么条件下停住，在收到什么信号时停往等，以便于用户查看运行时的变量，以及运行时的流程。

当进程被gdb停住时，可以使用info program 来查看程序是否在运行、进程号、被暂停的原因。

在gdb中，有以下几种暂停方式：断点(BreakPoint)、观察点(WatchPoint)、捕捉点(CatchPoint)、信号(Signals)及线程停止(Thread Stops)。

如果要恢复程序运行，可以使用c或是continue命令。

1. 设置断点(BreakPoint) 
    用break命令来设置断点。有下面几种设置断点的方法：

break

在进入指定函数时停住。C++中可以使用class::function或function(type,type)格式来指定函数名。

break

在指定行号停住。

break +offset

break -offset

在当前行号的前面或后面的offset行停住。offiset为自然数。

break filename:linenum

在源文件filename的linenum行处停住。

break filename:function

在源文件filename的function函数的入口处停住。

break *address

在程序运行的内存地址处停住。

break

该命令没有参数时，表示在下一条指令处停住。

break … if

condition表示条件，在条件成立时停住。比如在循环体中，可以设置break if i=100，表示当i为100时停住程序。

查看断点时，可使用info命令，如下所示(注：n表示断点号)：

info breakpoints [n]

info break [n]

1. 设置观察点(WatchPoint) 
    观察点一般用来观察某个表达式(变量也是一种表达式)的值是否变化了。如果有变化，马上停住程序。有下面的几种方法来设置观察点：

watch

为表达式(变量)expr设置一个观察点。一旦表达式值有变化时，马上停住程序。

rwatch

当表达式(变量)expr被读时，停住程序。

awatch

当表达式(变量)的值被读或被写时，停住程序。

info watchpoints

列出当前设置的所有观察点。

1. 设置捕捉点(CatchPoint) 
    可设置捕捉点来补捉程序运行时的一些事件。如载入共享库(动态链接库)或是C++的异常。设置捕捉点的格式为：

catch

当event发生时，停住程序。event可以是下面的内容：

● throw 一个C++抛出的异常 (throw为关键字)。

● catch 一个C++捕捉到的异常 (catch为关键字)。

● exec 调用系统调用exec时(exec为关键字，目前此功能只在HP-UX下有用)。

● fork 调用系统调用fork时(fork为关键字，目前此功能只在HP-UX下有用)。

● vfork 调用系统调用vfork时(vfork为关键字，目前此功能只在HP-UX下有)。

● load 或 load 载入共享库(动态链接库)时 (load为关键字，目前此功能只在HP-UX下有用)。

● unload 或 unload 卸载共享库(动态链接库)时 (unload为关键字，目前此功能只在HP-UX下有用)。

tcatch

只设置一次捕捉点，当程序停住以后，应点被自动删除。

1. 维护停止点 
    上面说了如何设置程序的停止点，gdb中的停止点也就是上述的三类。在gdb中，如果觉得已定义好的停止点没有用了，可以使用delete、clear、disable、enable这几个命令来进行维护。

Clear

清除所有的已定义的停止点。

clear

clear

清除所有设置在函数上的停止点。

clear

clear

清除所有设置在指定行上的停止点。

delete [breakpoints] [range…]

删除指定的断点，breakpoints为断点号。如果不指定断点号，则表示删除所有的断点。range 表示断点号的范围(如:3-7)。其简写命令为d。

比删除更好的一种方法是disable停止点。disable了的停止点，gdb不会删除，当还需要时，enable即可，就好像回收站一样。

disable [breakpoints] [range…]

disable所指定的停止点，breakpoints为停止点号。如果什么都不指定，表示disable所有的停止点。简写命令是dis.

enable [breakpoints] [range…]

enable所指定的停止点，breakpoints为停止点号。

enable [breakpoints] once range…

enable所指定的停止点一次，当程序停止后，该停止点马上被gdb自动disable。

enable [breakpoints] delete range…

enable所指定的停止点一次，当程序停止后，该停止点马上被gdb自动删除。

1. 停止条件维护 
    前面在介绍设置断点时，提到过可以设置一个条件，当条件成立时，程序自动停止。这是一个非常强大的功能，这里，专门介绍这个条件的相关维护命令。

一般来说，为断点设置一个条件，可使用if关键词，后面跟其断点条件。并且，条件设置好后，可以用condition命令来修改断点的条件(只有break和watch命令支持if，catch目前暂不支持if)。

condition

修改断点号为bnum的停止条件为expression。

condition

清除断点号为bnum的停止条件。

还有一个比较特殊的维护命令ignore，可以指定程序运行时，忽略停止条件几次。

ignore

表示忽略断点号为bnum的停止条件count次。

1. 为停止点设定运行命令 
    可以使用gdb提供的command命令来设置停止点的运行命令。也就是说，当运行的程序在被停住时，我们可以让其自动运行一些别的命令，这很有利行自动化调试。

commands [bnum]

… command-list …

end

为断点号bnum指定一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。

例如：

break foo if x>0

commands

printf “x is %d\n”,x

continue

end

断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是一旦x的值在foo函数中大于0，gdb会自动打印出x的值，并继续运行程序。

如果要清除断点上的命令序列，那么只要简单地执行一下commands命令，并直接在输入end就行了。

1. 断点菜单 
    在C++中，可能会重复出现同一个名字的函数若干次(函数重载)。在这种情况下，break 不能告诉gdb要停在哪个函数的入口。当然，可以使用break

​    分类:             [C/C++](https://www.cnblogs.com/taolusi/category/1243655.html)

​    标签:             [C/C++](https://www.cnblogs.com/taolusi/tag/C%2FC%2B%2B/),             [gdb](https://www.cnblogs.com/taolusi/tag/gdb/),             [cmake](https://www.cnblogs.com/taolusi/tag/cmake/)