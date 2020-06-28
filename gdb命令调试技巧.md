**gdb命令调试技巧**

**一、信息显示**
1、显示gdb版本 (gdb) show version
2、显示gdb版权 (gdb) show version or show warranty
3、启动时不显示提示信息gdb -q exe 或者.bashrc 添加alias gdb="gdb -q"，重启shell
4、退出时不显示提示信息(gdb) set confirm off
5、输出信息多时不会暂停输出(gdb)set pagination off 





**二、函数**
1、列出函数的名字(gdb) info functions
2、是否进入待调试信息的函数(gdb)step s
3、进入不带调试信息的函数(gdb)set step-mode on
4、退出正在调试的函数(gdb)return expression 或者 (gdb)finish
5、直接执行函数(gdb)start 函数名 call函数名
6、打印函数堆栈帧信息(gdb)info frame or i frame
7、查看函数寄存器信息(gdb)i registers
8、查看函数反汇编代码(gdb)disassemble func
9、打印尾调用堆栈帧信息(gdb)set debug entry-values 1
10、选择函数堆栈帧(gdb)frame n
11、向上或向下切换函数堆栈帧(gdb)up n down n











**三、断点**
1、在匿名空间设置断点(gdb)b Foo::foo (gdb) b (anonymous namespace)::bar
2、在程序地址上打断点(gdb)b *address (gdb) b *0x400522
3、在程序入口处打断点$strip a.out $readelf -h a.out或者(gdb)info files定位Entry point: 0x400440 (gdb)b *0x400440
4、在文件行号上打断点(gdb)b linenum (gdb)b file.cpp:linenum (gdb)info breakpoints
5、保存已经设置的断点(gdb)save breakpoints file-breakpoints-to-save (gdb)source file-breakpoints-to-save
6、设置临时断点(gdb)tbreak linenum
7、设置条件断点(gdb)b linenum if cond b 11 if i==10
8、忽略断点(gdb)ignore bnum count









**四、观察点**
1、设置观察点(gdb)watch a wacth *(type*)adress info watchpoints disable、enable、delete
2、设置观察点只针对特定线程生效(gdb)info threads watch expr thread threadnum wa a thread 2
3、设置读观察点(gdb)rwatch
4、设置读写观察点(gdb)awacth





**五、Catchpoint**
1、让catchpoint只触发一次(gdb)tcatch
2、为fork调用设置catchpoint (gdb)catch fork
3、为vfork调用设置catchpoint (gdb)catch vfork
4、为exec调用设置catchpoint (gdb)catch exec
5、为系统调用设置catchpoint (gdb)catch syscall name or num
6、通过ptrace调用设置catchpoint破解anti-debugging的程序 (gdb)catch syscall ptrace set $rax=0







**六、打印**
1、打印ASCII和宽字符字符串 (gdb)x/s str x/ws 4_wstr x/hs 2_wstr
2、打印STL容器中的内容 (gdb)p vec
3、打印大数组中的内容 (gdb)set print elements number-of-elements set print elements 0 or set print elements unlimited
4、打印数组中任意连续的元素值(gdb)p array[index]@num
5、打印数组的索引下标(gdb)set print array-indexes on
6、打印函数局部变量的值(gdb)bt full info locals
7、打印进程的内存信息(gdb)i proc mappings i files
8、打印静态变量的值(gdb)p 'static-1.c'::var
9、打印变量的类型和所在文件(gdb)whatis he ptype he i variables he
10、打印内存的值(gdb)16进制 x/16xb a 10进制x/16ub a 二进制x/16tb a 
11、打印源代码行(gdb)l line l 函数 l - l + l 1,10
12、每行打印一个结构体成员(gdb)set print pretty on 
13、按照派生类打印对象(gdb)set print object on 
14、指定程序的输入输出设备(gdb)tty gdb -tty /dev/pts/2 ./a.out tty /dev/pts/2
15、使用"$\\_"和"$\\__"变量(gdb) x "命令会把最后检查的内存地址值存在“ $_ ”这个“convenience variable”中，并
且会把这个地址中的内容放在“ $__ ”这个“convenience variable”
16、打印程序动态分配内存的信息(gdb)define mallocinfo set $__f = fopen("/dev/tty", "w") call malloc_info(0,  $__f)call fclose($__f)end mallocinfo 
17、打印调用栈帧中变量的值(gdb)b func_1 r bt f 1 p b f 2 p b p func2::b p func_3::b



















**七、多进程/线程**
1、调试已经运行的进程gdb program -p=10210 (gdb)attach pid detach
2、调试子进程(gdb)set follow-fork-mode child
3、同时调试父进程和子进程(gdb)set detach-on-fork off i inferiors set schedule-multiple on
4、查看线程信息(gdb)i threads
5、在Solaris上使用maintenance命令查看线程信息(gdb)maint info sol-threads
6、不显示线程启动和退出信息(gdb)set print thread-events off
7、只允许一个线程运行(gdb)set scheduler-locking on
8、使用"$_thread"变量(gdb)command 2Type commands for breakpoint(s) 2, one per line.End with a line saying just "end".>printf "thread id=%d\n", $_thread >end
9、一个gdb会话中同时调试多个程序(gdb)add-inferior [ -copies n ] [ -exec executable ]
10、打印程序进程空间信息(gdb)maint info program-spaces
11、使用"$_exitcode"变量(gdb)p $_exitcode











**八、core dump文件**
1、为调试进程产生core dump文件(gdb)generate-core-file or gcore
2、加载可执行程序和core dump文件(gdb)gdb -q /data/nan/a /var/core/core.a.22268.1402638140



**九、汇编**
1、设置汇编指令格式(gdb)set disassembly-flavor intel disassemble main
2、在函数的第一条汇编指令打断点(gdb)b *main
3、自动反汇编后面要执行的代码(gdb)set disassemble-next-line on set disassemble-next-line auto set disassemble-next-line off
4、将源程序和汇编指令映射起来(gdb)disas /m main
5、显示将要执行的汇编指令(gdb)display /i $pc 
6、打印寄存器的值(gdb)i registers i all-registers i registers eax 
7、显示程序原始机器码(gdb)disassemble /r main







**十、改变程序执行顺序**
1、改变字符串的值(gdb)set main::p1="Jil"
2、设置变量的值(gdb)set var variable=expr set var i = 8 set {int}0x8047a54 = 8 set var $eax = 8
3、修改PC寄存器的值(gdb)p $pc set var $pc=0x08050949
4、跳转到指定位置执行(gdb)j 15
5、使用断点命令改变程序的执行(gdb)(gdb) b drawing Breakpoint 1 at 0x40064d: file win.c, line 6.
(gdb) command 1
Type commands for breakpoint(s) 1, one per line.
End with a line saying just "end".
\>silent
\>set variable n = 0
\>continue
\>end
(gdb) r
6、修改被调试程序的二进制文件gdb -write ./a.out (gdb)show write set write on disassemble /mr drawing set variable *(short*)0x400651=0x0ceb disassemble /mr drawing















**十一、信号**
1、查看信号处理信息(gdb)i signals 
2、信号发生时是否暂停程序(gdb) handle signal stop/nostop
3、信号发生时是否打印信号信息(gdb)handle signal print/noprint 
4、信号发生时是否把信息丢给程序处理(gdb)handle signal pass(noignore)/nopass(ignore) 
5、给程序发送信息(gdb)signal signal_name 
6、使用"$_siginfo"变量(gdb)ptype $_siginfo







**十二、共享库**
1、显示共享连接库信息(gdb)info sharedlibrary regex

**十三、脚本**
1、配置gdb init文件(gdb) home目录下的 .gdbinit
2、按何种方式解析脚本文件(gdb)set script-extension off soft strict
3、保存历史命令(gdb)set history filename ~/.gdb_history set history save on



**十四、源文件**
1、设置源文件查找路径(gdb)directory ../ki/
2、替换查找源文件的目录(gdb)set substitute-path from to



**十五、图形化界面**
1、进入和退出图形化调试界面(gdb)gdb -tui program
2、显示汇编代码窗口(gdb)layout asm
3、显示寄存器窗口(gdb)layout regs
4、调整窗口大小(gdb)winheight <win_name> [+ | -]count





十六、其它
1、命令行选项的格式(gdb)gdb -help
2、支持预处理器宏信息(gdb)gcc -g3 
3、使用命令的缩写形式(gdb)b -> break
c -> continue
d -> delete
f -> frame
i -> info
j -> jump
l -> list
n -> next
p -> print
r -> run
s -> step
u -> until
aw -> awatch
bt -> backtrace
dir -> directory
disas -> disassemble
fin -> finish
ig -> ignore
ni -> nexti
rw -> rwatch
si -> stepi
tb -> tbreak
wa -> watch
win -> winheight
4、在gdb中执行shell命令和make(gdb)shell ls
5、在gdb中执行cd和pwd命令(gdb)pwd cd tmp
6、设置命令提示符(gdb)gdb -q `which gdb
7、设置被调试程序的参数(gdb)gdb -args ./a.out a b c set args a b c r a b
8、设置被调试程序的环境变量(gdb)set env varname=value
9、得到命令的帮助信息(gdb)help
10、记录执行gdb的过程(gdb)set logging file log.txt set logging on