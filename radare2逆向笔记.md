# radare2逆向笔记

​                                        转载                                                                                                                                            [AirZH??](https://me.csdn.net/weixin_34290631)                    最后发布于2018-02-09 18:13:00                                        阅读数 52                                                                                                                        收藏                                    

​                                                                展开                                    

![radare2](http://p20y2jias.bkt.clouddn.com/excerpt/r2.png)
 最近刚开始学习逆向(Reverse Engineering), 发现其学习曲线也是挺陡峭的, 而网上的
 许多writeup文章主旨总结就六个字:"你们看我屌吗?" ...几近炫技而对初学者不太友好.
 所以我打算以初学者的身份来写写自己从入门到深入的经历.

# 准备

当前许多逆向的writeup倾向于使用IDA-Pro, 而且似乎都依赖于F5(反编译的快捷键), 直接
 从二进制文件转成了可读的C代码. 这对于实际工作来说也许是个捷径, 但对于学习来说却
 没什么好处. 所以本文逆向采用了另一个开源的(但也同样强大的)二进制分析工具--[Radare2](https://github.com/radare/radare2).
 如果你是个资深的逆向人员, 那么从本文也可以了解下radare2的一些功能.

## 知识准备

逆向软件的时候往往面对的是汇编代码, 所以对于指令集要有个大致的认识, 另外对于一些
 模式(pattern), 比如函数入口(prologue), 出口(epilogue)和函数调用约定(calling convention)
 等也要有所了解. 关于这类知识可以将[RE4B(Reverse Engineering for Beginners)](https://beginners.re/)
 这本书作为参考. 书虽然比较厚, 但比较全面, 包括了X86/ARM/MIPS的内容和许多有趣的历史典故,
 一开始可以粗略扫一遍, 遇到问题再回头仔细阅读相关部分即可.

## 工具准备

当了解了基本汇编知识后(目前x86足矣), 就可以开始准备工具了. 说起工具我想起了一个寓言:

> 年轻人学有所成, 出山前问师父: "我准备练习武器, 请问哪种武器能让我战无不胜呢?"
>  师父说: "武器? 如果你的武器比你的头脑更加锋利, 那你将一无是处".

无论何时, 个人的头脑和思维永远是最重要的, 而武器只是工具, 永远不要让工具取代了你的思考.
 当然也不是要你肉眼反汇编, 总之...你懂就行了! 我在Linux环境工作, 用到的几个工具如下:

- [radare2工具集(用于静态分析)](https://github.com/radare/radare2)
- gdb(用于调试)
- [peda(一个gdb的插件)](https://github.com/longld/peda)

## 目标准备

初步打算是这一系列逆向文章使用IOLI的crackme文件来作为目标, 总共3个平台(Linux/Win32和Arm),
 每个平台有10个二进制文件, 都是从同样的源码编译而来的. 可以从[radare的git上](https://github.com/radare/radare2book/tree/master/crackmes/ioli/IOLI-crackme.tar.gz)下载.

# crackme0x00

好的, 这是第一个目标, 首先了解你的敌人:

```
rabin2 -I crackme0x00
```

`rabin2`是`radare2`套件中的一个工具, 主要用来提取二进制文件中的信息, 输出如下:

```
arch     x86binsz    7537bintype  elfbits     32canary   falseclass    ELF32crypto   falseendian   littlehavecode trueintrp    /lib/ld-linux.so.2lang     clinenum  truelsyms    truemachine  Intel 80386maxopsz  16minopsz  1nx       trueos       linuxpcalign  0pic      falserelocs   truerelro    partialrpath    NONEstatic   falsestripped falsesubsys   linuxva       true
```

然后查看`.rodata`字段里的字符串:

```
rabin2 -z crackme0x00
```

输出如下:

```
000 0x00000568 0x08048568  24  25 (.rodata) ascii IOLI Crackme Level 0x00\n001 0x00000581 0x08048581  10  11 (.rodata) ascii Password: 002 0x0000058f 0x0804858f   6   7 (.rodata) ascii 250382003 0x00000596 0x08048596  18  19 (.rodata) ascii Invalid Password!\n004 0x000005a9 0x080485a9  15  16 (.rodata) ascii Password OK :)\n
```

运行一下该程序:

```
$ ./crackme0x00 IOLI Crackme Level 0x00Password: 123456Invalid Password!
```

看样子是要输入密码, 从rabin2的字符串输出里看到`250382`也许就是密码, 我们可以输入试试:

```
$ ./crackme0x00 IOLI Crackme Level 0x00Password: 250382Password OK :)
```

好吧! 看样子确实是. 毕竟是第一关, 有点简单了. 但我还是先假装不知道吧:)
 如果目的是进入到`Password OK`分支, 那么我们可以有多种解法, 如分析密码的算法,
 修改原文件(打patch), 利用漏洞, fuzzy等, 下面挑几个说说.

## 解法1: 逆向分析

话不多说, 打开radare2:

```
r2 -A crackme0x00
```

进入后自动跳转到了函数入口0x08048360, 然后用pdf命令来查看汇编代码:

```
[x] Analyze all flags starting with sym. and entry0 (aa)[x] Analyze len bytes of instructions for references (aar)[x] Analyze function calls (aac)[x] Use -AA or aaaa to perform additional experimental analysis.[x] Constructing a function name for fcn.* and sym.func.* functions (aan) -- You can 'copy/paste' bytes using the cursor in visual mode 'c' and using the 'y' and 'Y' keys[0x08048360]> pdf @ sym.main
```

> pdf表示p(打印)d(反汇编)f(函数), @表示取地址, sym.main为函数符号, 也可以用十六进制整数地址表示.
>  在r2中查看这些指令的帮助只要在后面输入`?`即可, 比如`p?`查看打印类的命令, `pd?`查看打印反汇编类的命令.

打印反汇编的输出如下:

```
            ;-- main:/ (fcn) main 127|   main ();|           ; var int local_18h @ ebp-0x18|           ; var int local_4h @ esp+0x4|              ; DATA XREF from 0x08048377 (entry0)|           0x08048414      55             push ebp|           0x08048415      89e5           mov ebp, esp|           0x08048417      83ec28         sub esp, 0x28               ; '('|           0x0804841a      83e4f0         and esp, 0xfffffff0|           0x0804841d      b800000000     mov eax, 0|           0x08048422      83c00f         add eax, 0xf|           0x08048425      83c00f         add eax, 0xf|           0x08048428      c1e804         shr eax, 4|           0x0804842b      c1e004         shl eax, 4|           0x0804842e      29c4           sub esp, eax|           0x08048430      c70424688504.  mov dword [esp], str.IOLI_Crackme_Level_0x00 ; [0x8048568:4]=0x494c4f49 ; "IOLI Crackme Level 0x00\n"|           0x08048437      e804ffffff     call sym.imp.printf         ; int printf(const char *format)|           0x0804843c      c70424818504.  mov dword [esp], str.Password: ; [0x8048581:4]=0x73736150 ; "Password: "|           0x08048443      e8f8feffff     call sym.imp.printf         ; int printf(const char *format)|           0x08048448      8d45e8         lea eax, [local_18h]|           0x0804844b      89442404       mov dword [local_4h], eax|           0x0804844f      c704248c8504.  mov dword [esp], 0x804858c  ; [0x804858c:4]=0x32007325|           0x08048456      e8d5feffff     call sym.imp.scanf          ; int scanf(const char *format)|           0x0804845b      8d45e8         lea eax, [local_18h]|           0x0804845e      c74424048f85.  mov dword [local_4h], str.250382 ; [0x804858f:4]=0x33303532 ; "250382"|           0x08048466      890424         mov dword [esp], eax|           0x08048469      e8e2feffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)|           0x0804846e      85c0           test eax, eax|       ,=< 0x08048470      740e           je 0x8048480|       |   0x08048472      c70424968504.  mov dword [esp], str.Invalid_Password ; [0x8048596:4]=0x61766e49 ; "Invalid Password!\n"|       |   0x08048479      e8c2feffff     call sym.imp.printf         ; int printf(const char *format)|      ,==< 0x0804847e      eb0c           jmp 0x804848c|      ||      ; JMP XREF from 0x08048470 (main)|      |`-> 0x08048480      c70424a98504.  mov dword [esp], str.Password_OK_: ; [0x80485a9:4]=0x73736150 ; "Password OK :)\n"|      |    0x08048487      e8b4feffff     call sym.imp.printf         ; int printf(const char *format)|      |       ; JMP XREF from 0x0804847e (main)|      `--> 0x0804848c      b800000000     mov eax, 0|           0x08048491      c9             leave\           0x08048492      c3             ret
```

一个典型的32位Linux程序, 这时候要想起函数的调用约定是通过栈来传参, 如果忘了可以再看看[RE4B](https://beginners.re/)哦.
 看到main函数的汇编代码了, 就开始分析了, 我不想像一些文章那样贴个图就说"显而易见, 这里的作用是XXX",
 毕竟我也只是个新手, 虽然这只是一个超简单的crackme, 但因为是第一次, 我还是把流程完整地走一遍.

有了汇编接下来就开始分析了, r2和IDA一样可以自己写注释和修改变量名称, 在此之前我们先创建一个工程,
 以保存这些修改:

```
[0x08048360]> Ps ioli0x00
```

这条指令的意思是保存一个名为`ioli0x00`的(新)项目, 通常默认保存在`~/.config/radare2/projects`里.

> 以P开头的命令是项目工程管理相关(Project managment), 还记得之前说的吗,
>  如果不记得命令, 可以通过`P?`来查看帮助.

然后跳转到main并打印本地局部变量:

```
[0x08048360]> s main[0x08048414]> afvvar int local_4h @ esp+0x4var int local_18h @ ebp-0x18
```

s表示seek, 跳转后发现我们已经到了`0x08048414`, afv表示a(分析)f(函数)v(变量), 可以看到有两个局部变量.
 (其实只有一个, 因为esp+0x4是传给子函数的参数).
 再回到前面的汇编看看, 从0x08048448到0x08048456这几条汇编可以发现`local_4h`是`local_18h`的地址(指针),
 而且`local_4h`是scanf的第二个参数, scanf的第一个参数为0x804858c, 这个地址应该是个字符串, 我们打印下看看:

```
[0x08048414]> ps @ 0x804858c%s
```

那么`local_18h`应该就是用户输入的字符串了, 我们先给他们改个好听的名字:

```
afvn local_18h inputafv-local_4h
```

`afv-`表示删除某个名字, 这里删除了`local_4h`因为它其实不是本地变量, 这时再次打印汇编就能看到改好的名字了:

```
0x08048448      8d45e8         lea eax, [input]0x0804844b      89442404       mov dword [esp+4], eax0x0804844f      c704248c8504.  mov dword [esp], 0x804858c  ; [0x804858c:4]=0x320073250x08048456      e8d5feffff     call sym.imp.scanf          ; int scanf(const char *format)0x0804845b      8d45e8         lea eax, [input]0x0804845e      c74424048f85.  mov dword [esp+4], str.250382 ; [0x804858f:4]=0x33303532 ; "250382" 0x08048466      890424         mov dword [esp], eax        ; 这句和上一句相当于push str.250382; push eax0x08048469      e8e2feffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)0x0804846e      85c0           test eax, eax
```

这下就能比较清楚的看出上述代码的核心目的了, 大约是:

```
char input[N];scanf("%s", input)strcmp(input, "250382")
```

由前面的`sub esp, 0x28`可知, 这里的N应该小于40(没错! 这里有一个栈溢出漏洞!). 不过对于函数
 prologue之后的几条汇编我还不是很明白作用是是啥, 希望有大神能告知一下~

至此, crackme0x00的分析基本完成. 虽然有些步骤看起来很繁琐, 但对于分析大型项目还是很有用的.
 尤其是给变量/参数命名, 给函数/代码块命名, 这样会使得分析过程步步为营, 柳暗花明.

## 解法2: 修改程序

当我们能直接接触程序并且有修改权限时, 那么修改该二进制文件也是个快速通关的好办法!
 回到刚刚的汇编输出, 我们看到`0x08048470`这行有一个跳转分支:

```
     0x0804846e      85c0           test eax, eax ,=< 0x08048470      740e           je 0x8048480 |   0x08048472      c70424968504.  mov dword [esp], str.Invalid_Password ; [0x8048596:4]=0x61766e49 ; "Invalid Password!\n" |   0x08048479      e8c2feffff     call sym.imp.printf         ; int printf(const char *format),==< 0x0804847e      eb0c           jmp 0x804848c||      ; JMP XREF from 0x08048470 (main)|`-> 0x08048480      c70424a98504.  mov dword [esp], str.Password_OK_: ; [0x80485a9:4]=0x73736150 ; "Password OK :)\n"|    0x08048487      e8b4feffff     call sym.imp.printf         ; int printf(const char *format)|       ; JMP XREF from 0x0804847e (main)`--> 0x0804848c      b800000000     mov eax, 0
```

`test a, b`的意思是`若a AND b为0, 则设置ZF位(以及SF/PF)`, je表示`若ZF位被设置则跳转`,
 说人话就是判断前一个函数的返回值是否为0(eax保存strcmp的返回值), 若为0则跳转到`0x8048480`(打印"Password OK :)\n"),
 所以, 我们只需要把je改成无条件跳转`jmp`就可以啦!
 不过这时候要重新打开r2:

```
r2 -w crackme0x00
```

`-w`参数表示以可写的方式打开程序, 而之前我们的打开方式是只读滴! 或者在之前的会话中输入:

```
[0x08048414]> eval cfg.write=true
```

也同样能获得修改文件的权限.

改好之后首先我们先跳转到想要修改指令的地方:

```
[0x08048414]> s 0x08048470[0x08048470]>
```

这里的机器码是`0x740e`, 反汇编为`je 0x10`, 不懂怎么反? 这时候就要介绍radare2中的另一个工具`rasm2`了:

```
$ rasm2 -a x86 -b 32 -d "0x740e"je 0x10$ rasm2 -a x86 -b 32 "je 0x10"740e
```

`-a`为CPU架构, `-b`为CPU寄存器位数, `-d`表示反汇编, 是不是很直观? 接下来我们要把jz改为jmp:

```
$ rasm2 -a x86 -b 32 "jmp 0x10"eb0e
```

也就是说把74改为eb就行了! 只改一个字节, 怎么操作? 如下:

```
[0x08048470]> px 20- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF0x08048470  740e c704 2496 8504 08e8 c2fe ffff eb0c  t...$...........0x08048480  c704 24a9                                ..$.[0x08048470]> wx eb[0x08048470]> px 20- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF0x08048470  eb0e c704 2496 8504 08e8 c2fe ffff eb0c  ....$...........0x08048480  c704 24a9                                ..$.
```

> `px`表示以hexdump形式打印当前位置的N个字节, `wx`表示在当前位置写入
>  如果有疑问, 别忘了你的好帮手`?`哦, `px?`和`wx?`都能查看对应的帮助.

OK现在改好了, 退出r2, 再运行下试试:

```
./crackme0x00 IOLI Crackme Level 0x00Password: fuckmePassword OK :)
```

现在输入任何密码都能通关啦!

## 解法3: 利用漏洞

刚刚在说解法1时, 最后看到了在scanf函数的执行过程中, 存在一个栈溢出漏洞, 先写个小bash脚本验证下:

```
#!/bin/bashfor i in {1..50};do    input=$(cat /dev/urandom | tr -dc 'a-z0-9' | head -c $i)    echo $input | ./crackme0x00    if [ $? -ne 0 ];then        echo "OOOOOps!!! input($input) of length $i crush this program!"        break    fidone
```

输出为:

```
......550 Segmentation fault      | ./crackme0x00OOOOOps!!! input(rvibn45fg3eugqg6257rtyazsqclz) of length 29 crush this program!
```

实锤溢出跑不掉了. 那么利用这个漏洞, 我们同样可以实现通关, 甚至可以获得任意系统命令执行!
 首先查看二进制安全选项:

```
$ gdb crackme0x00 Reading symbols from crackme0x00...(no debugging symbols found)...done.(gdb) source ~/tools/peda/peda.py gdb-peda$ checksec CANARY    : disabledFORTIFY   : disabledNX        : ENABLEDPIE       : disabledRELRO     : Partialgdb-peda$ aslrASLR is OFF
```

> `~/tools/peda`就是下载的peda路径, 可以直接从[github上](https://github.com/longld/peda)拉最新的.

可以看到NX是开着的, 但是很幸运, ASLR没有开, 不过就算开了并不影响这节的示例, 结尾会说原因.
 刚刚bash脚本的输出告诉我们输入长为29字节的时候程序就崩溃了, 那么猜测29字节开始就覆盖了EIP,
 但毕竟是猜测, 二进制的世界不接受模棱两可的答案!

首先生成一组[De Bruijn](https://en.wikipedia.org/wiki/De_Bruijn_sequence)模式的序列, 目的只是为了在溢出时候确认位置, 你用自己的方法也可以,
 这里用`ragg2`生成(ragg2也是radare2套件中的一个小工具):

```
$ ragg2 -P 40 -r > patt.txt$ cat patt.txtAAABAACAADAAEAAFAAGAAHAAIAAJAAKAALAAMAAN
```

然后启动radare2并且将`patt.txt`文件的内容作为调试程序的标准输入:

```
$ cat > crackme0x00.rr2 <<EOF#!/usr/bin/env rabin2stdin=./patt.txtEOF$ r2 -e dbg.profile=crackme0x00.rr2 -d crackme0x00
```

进入r2交互界面之后, 依次输入`d(调试)c(继续)`和`wopO`:

```
Process with PID 7943 started...= attach 7943 7943bin.baddr 0x08048000Using 0x8048000asm.bits 32 -- See you at the defcon CTF[0xf76e6a20]> dcIOLI Crackme Level 0x00Password: Invalid Password!child stopped with signal 11[+] SIGNAL 11 errno=0 addr=0x414b4141 code=1 ret=0[0x414b4141]> wopO eip28
```

`wopO value`表示在De Bruijn序列中找到value所代表的偏移量, 当然你也可以在patt.txt
 中数数, eip当前的位置为`0x414b4141(即ascii AKAA)`, 也就是说输入28字节以后就开始覆盖eip
 地址了, 确定具体的溢出位置之后, 接下来要做的就是控制PC指针, 劫持运行流程.

还记得解法2时要跳转到正确分支的地址吗? 不错就是`0x08048480`, 来试试呗, 注意字节序哦:

```
$ python2 -c "print 'A'*28+'\x80\x84\x04\x08'" | ./crackme0x00IOLI Crackme Level 0x00Password: Invalid Password!Password OK :)Segmentation fault
```

成功执行了打印"Password OK"的分支, 但还是有点不完美, 因为最后没有清理好现场, 不过这足以说明我们
 能够掌控执行流程了!
 一个优雅的exploit在劫持控制流程后还应该优雅退出, 而为了执行更多函数, 比如`system`,`exit`等, 就需要知道
 libc的(基)地址, 这里ASLR没开所以很容易获得所有libc函数的地址, 利用ROP链就能做任何你想做的事,
 这里就不深入了!

## 解法4: 利用Fuzzy

这个方法综合了上述的一些方式, 我们可以用暴力破解的方式来获取密码, 也可以利用afl或者libFuzzer
 来自动化找出该程序潜在的bug(配合QEMU). 这种方式的坏处是太暴力了, 让妹子不敢靠近(逃)； 好处
 则是在一定程度上解放了大脑, 用计算机来帮我们计算, 算力越强就越有可能找到突破点!

# 总结

本来是想多写几个crackme的, 但是由于这是第一篇, 就讲详细一点, 以后会深入一些更复杂和的程序,
 写几篇真正意义上的writeup. 总结一下求解crackme类问题的方法, 1)逆向分析, 2)修改程序, 3)利用漏洞,
 4)利用Fuzzy. 通常我们在遇到实际问题时是会将这些方式结合起来用的, 比如虽然逆向分析了一部分代码,
 但某部分算法特别复杂, 那么就会借用Fuzzy的思想, 对这部分逻辑进行Symbolic Execution, 以最快的方式解决战斗!

- 能力有限, 文中错误在所难免, 欢迎交流!
- 文章转载请注明来源: https://www.pppan.net/ 谢谢!

转载于:https://www.cnblogs.com/pannengzhi/p/play-with-radare2.html