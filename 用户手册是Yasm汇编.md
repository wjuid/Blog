# 用户手册是Yasm汇编

​										  									 2017-06-15 									 [原文](https://www.bbsmax.com/link/bW81a1E4WUt6dw==) 									 									 									 									 								



本文档的用户手册是Yasm汇编。 它是介绍和通用所有Yasm用户参考。

英文的参考：http://www.cnblogs.com/coryxie/p/3959888.html

**1 .介绍**

Yasm bsd许可下是一个汇编程序,而设计的,以便支持多个汇编程序语法(例如NASM,GNU等)除了多个输出对象格式和多个指令集。 其模块化的体系结构允许额外的对象格式,添加调试格式和语法相对容易。

Yasm 2001年开始生活的重写NASM Netwide x86汇编在BSD许可。 自那时以来,它已经达到和超过NASM的功能,结合特性,比如支持64位AMD64架构,解析GNU语法,并生成刺穿,DWARF2 CodeView 8调试信息。

**2 .许可证**

Yasm许可2-clause下3条款“修订”BSD许可,但有一个例外:::矢量模块使用的主线版本的Yasm实现独立于机器大整数和浮点支持下triple-licensed艺术许可证GPL,LGPL。 “yasm-nextgen”代码使用一个完全不同的bsd许可下实现,因此在BSD-equivalent许可证。  提供的许可证的全文Yasm源分布。

这个用户手册2-clause BSD许可下的。

**3所示。 材料覆盖在这本书**

这本书的目的是成为Yasm的用户手册,作为一个介绍和通用的参考。 虽然提到Yasm在各个部分的实现(通常是解释背后的原因错误或不寻常的方面各种特性),这本书将不进入深度解释Yasm工作;Yasm深入讨论的内部,看到的*Yasm汇编程序的设计和实现* 。

**我一部分。 使用Yasm**

**所说yasm简介**

yasm ( - f *格式* ][ - o *输出文件* ][ *其他选项* … ){ *infile* }

**1.2描述。**

的 **yasm** 命令组装文件 *infile* 并将输出到文件 *输出文件* 如果指定。 如果 *输出文件* 没有指定, **yasm** 将获得一个默认输出文件的名字从其输入文件的名称,通常通过添加什么 . o 或 .obj ,或通过删除所有扩展原始二进制文件。 如果做不到这一点,将输出文件的名字 yasm.out 。

如果叫一个 *infile* “-”, **yasm** 组装的标准输入,输出到文件 *输出文件* ,或 yasm.out 如果没有 *输出文件* 都是确定的。



如果在执行过程中发现的错误或警告,Yasm输出错误消息 stderr (通常终端)。 如果没有遇到错误或警告,Yasm不输出任何信息。

**选择1.3。**

许多选项可以在两种形式之一:破折号后面跟着一个字母,或者两个破折号,后跟一个长选项名称。 按字母顺序列出选项。

**1.3.1。 一般选择**

**1.3.1.1。 ——一个 \*拱\* 或 ——拱= \*拱\* :选择目标架构**

选择目标架构。 默认架构是“x86”,支持ia - 32和衍生品和AMD64指令集。 打印列表可用的架构到标准输出,使用“帮助” *拱* 。 看到 [1.4节](http://www.tortall.net/projects/yasm/manual/html/manual.html)的支持架构。

**1.3.1.2。 - f \*格式\* 或 ——oformat = \*格式\* :选择对象格式**

选择对象格式的输出。 默认的对象格式是“本”,这是一个平没有搬迁的二进制格式。 打印到标准输出可用对象的列表格式,使用“帮助” *格式* 。 看到 [1.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html)支持的对象列表格式。

**1.3.1.3。 - g \*调试\* 或 ——dformat = \*调试\* :选择调试格式**

选择的调试格式调试信息。 调试信息可以使用调试器将可执行代码返回到源文件或数据结构和类型的信息。 可用的调试格式不同对象格式; **yasm** 将错误无效的组合被选中。 选择默认的对象格式的对象格式。 打印列表可用的调试格式到标准输出,使用“帮助” *调试* 。 看到 [1.7节](http://www.tortall.net/projects/yasm/manual/html/manual.html)支持的调试格式列表。

**1.3.1.4。 - h 或 ——帮助 :打印选项的摘要**

打印的摘要调用选项。 所有其他选项都被忽略,没有生成输出文件。

**1.3.1.5。 - l \*列表\* 或 ——lformat = \*列表\* 文件格式:选择列表**

选择输出的格式/样式列表文件。 列表文件通常混杂的原始机器生成的汇编代码。 默认列表格式是“nasm”,模仿nasm列表文件格式。 打印可用列表的列表文件格式到标准输出,使用“帮助” *列表* 。

**1.3.1.6。 - l \*listfile\* 或 ——列表= \*listfile\* :指定文件名列表**

指定输出文件列表的名称。 如果不使用此选项,没有生成文件列表。

**1.3.1.7。 - m \*机\* 或 ——机器= \*机\* :选择目标计算机体系结构**

选择目标计算机体系结构。 本质上所选建筑的一个亚型,机器类型选择主要的子集之间的一个架构。  例如,对于“x86架构,两个可用机器“x86”,用于ia -  32和导数32位指令集,和“amd64”,用于64位指令集。这种区别需要为浮动对象生成适当的对象文件格式如COFF和精灵。  打印可用机器的列表对于一个给定的体系结构到标准输出,使用“帮助” *机* 和给定的体系结构 ——一个 *拱* 。 看到 [第六部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)为更多的细节。

**1.3.1.8。 - o \*文件名\* 或 ——objfile = \*文件名\* 对象:指定文件名**

指定输出文件的名称,覆盖任何Yasm生成的缺省名称。

**1.3.1.9。 - p \*解析器\* 或 ——解析器= \*解析器\* :选择解析器**

选择解析器(汇编程序语法)。 默认解析器是“nasm”,模拟nasm的语法,Netwide汇编程序。 另一个可用的解析器是“气”,模拟GNU的语法。 打印可用解析器的列表到标准输出,使用“帮助” *解析器* 。 看到 [1.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html)支持的解析器列表。

**1.3.1.10。 - r \*preproc\* 或 ——preproc = \*preproc\* :选择预处理器**



选择预处理器上使用输入文件之前将它传递给解析器。 预处理器通常提供宏观功能,不包括在主要的解析器。  默认的预处理是“nasm”,这是一个进口版本实际nasm的预处理器。 “原始”预处理器也可以,只是跳过的预处理步骤,直接输入文件传递给解析器。  打印列表可用的预处理器到标准输出,使用“帮助” *preproc* 。

**1.3.1.11。 ——版本 :Yasm版本**

这个选项会导致Yasm打印的版本号Yasm以及许可证总结到标准输出。 所有其他选项都被忽略,没有生成输出文件。

**1.3.2。 警告选项**

\- w 选项有两个相反的形式: - w ?叫什么名字? 和 -Wno - *的名字* 。 这里只显示了非默认形式。

警告选项在命令行上给出的顺序处理,如果 - w 紧随其后的是 -Worphan-labels ,所有的警告都是关闭的 *除了*orphan-labels。

**1.3.2.1。 - w :抑制所有的警告消息**

这个选项会导致Yasm抑制所有警告消息。 正如上面所讨论的,这个选项可以指定其他选项重新启用警告紧随其后。

**1.3.2.2。 -Werror :将警告视为错误**

这个选项会导致Yasm对所有的警告和错误。 通常警告不防止生成的对象文件,不会导致失败的退出状态 **yasm**,而错误。 这个选项让警告相当于错误的这种行为。

**1.3.2.3。 -Wno-unrecognized-char :不要警告未识别的输入字符**

导致Yasm不警告无法识别的字符输入。 通常Yasm将生成一个警告任何非ascii字符输入文件中找到。

**1.3.2.4。 -Worphan-labels 末尾:警告标签上缺少冒号**

当使用NASM-compatible解析器,使Yasm警告标签发现独自一行末尾没有冒号。 虽然这些是合法的标签在NASM中语法,他们可能是无意的,由于拼写错误或宏定义命令。

**1.3.2.5。 - x \*风格\* :改变错误/警告报告风格**

选择一个特定的输出样式错误和警告消息。 默认是“gnu”风格,模仿的输出 **海湾合作委员会** 。 的“vc”风格也可以模仿微软Visual Studio的编译器的输出。

这个选项是可用的,这样Yasm将更自然地集成到IDE环境等 **Visual Studio** 或 **Emacs** ,允许IDE正确认识到错误/警告信息等,并链接到的违规行源代码。

**1.3.3。 预处理器的选择**

虽然理论上这些预处理选项将影响任何预处理器,目前唯一的预处理器在Yasm nasm预处理器。

**1.3.3.1。 - d \*宏(=价值)\* :预定义宏**

预定义一个单行的宏。 值是可选的(如果没有赋值,宏仍然是定义,但是一个空值)。

**1.3.3.2。 - e 或 ——preproc-only :只有进行预处理**

停止组装后的预处理阶段;预处理输出发送到指定的输出名称或,如果没有指定输出名称,标准输出。 不产生目标文件。

**1.3.3.3。 -我 \*路径\* :添加包含文件路径**

添加目录 *路径* 包括文件的搜索路径。 搜索路径默认为只包括源文件所在的目录。

**1.3.3.4。 - p \*文件名\* :Pre-include文件**

Pre-includes文件 *文件名* ,使它看起来好像 *文件名* 返回输入。 可以用于将多行宏的 - d 不能支持。

**1.3.3.5。 - u \*宏\* :未定义一个宏**

未赋值一个单行的宏(可以是一个内置的宏或一个前面定义的命令行 - d (见 [1.3.3.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

**1.4。 支持目标架构**

Yasm支持以下指令集架构(isa)。 更多细节见 [第六部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

x86

“x86体系结构支持ia - 32指令集和衍生品(包括16位和non-Intel指令)和AMD64指令集。它由两台机器:“x86”(ia - 32和衍生品)和“AMD64”(AMD64和衍生品)。 默认的机器“x86架构的x86机器。

**1.5。 支持解析器(语法)**

Yasm解析汇编程序语法如下:

nasm

NASM语法是最Yasm全功能的语法支持。 Yasm近100%兼容NASM对16位和32位x86代码。 Yasm另外支持64位AMD64代码与NASM Yasm扩展语法。 更多细节见 [第二部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

气体

GNU汇编(气)是现代Unix系统的实际跨平台的汇编程序,并用作GCC编译器后端。  Yasm支持天然气语法比较好,虽然不成熟:并不是所有的指令都支持,只有32位x86和AMD64架构支持。 也没有对气体预处理器的支持。  尽管有这些限制,Yasm天然气语法支持很好的处理本质上所有的x86和AMD64 GCC编译器输出。 更多细节见 [第三部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**1.6。 支持对象格式**

Yasm支持以下对象格式。 可以找到更多的细节 [第四部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

本

“本”对象格式生成一个扁平格式,non-relocatable二进制文件。 它是适合生产DOS。 COM可执行文件或引导块之类的东西。 它只支持3部分,这些部分都写在一个预定义的输出文件。

coff

COFF对象格式是一个年长的浮动对象格式用于旧的Unix和兼容系统,以及(最近)DJGPP DOS开发系统。

dbg

“dbg”对象格式不是“真正的”对象格式,其创建的输出文件简单地描述的顺序调用它Yasm和最终的对象和人类可读的文本格式符号表信息(在一个正常的对象格式会加工成对象格式的特定的二进制表示)。 这个对象格式不是为了真正的使用,而是用于调试Yasm内部。

精灵

ELF对象格式真的有三种口味:并且借助elf32””(32位的目标),“elf64”(64位目标),和“elfx32”(x32目标)。  精灵是一个标准的对象格式常用的现代Unix和兼容的系统(例如Linux,FreeBSD)。 精灵已经复杂的支持浮动和共享对象。

男子气概

Mach-O对象格式真的有两种味道:“macho32”(32位的目标)和“macho64”(对于64位目标)。  Mach-O用作对象格式在MacOS x  Yasm目前只支持x86和AMD64指令集,它只能为基于英特尔处理器的mac电脑上生成Mach-O对象。

rdf

RDOFF2对象格式是一个简单的多节格式NASM的最初设计。 它支持段引用但不是关于引用。 它的主要目的是简单和简约的标题为便于加载和链接。 一个完整的工具链(链接器、图书管理员和装入器)与NASM分布。

win32

Win32对象生成对象文件格式兼容微软编译器(例如Visual Studio)的32位x86 Windows平台。 COFF的对象格式本身是一个扩展的版本。

win64

Win64对象生成对象文件格式兼容微软编译器,目标64位Windows平台“x64”。 这种格式非常类似于win32对象格式,但64位生成对象。

xdf

XDF对象本质上是一个简化版的COFF格式。 这是一个多节浮动格式,支持64位的物理和虚拟地址。

**1.7。 支持调试格式**

Yasm支持一代的源代码级调试信息在以下格式。 可以找到更多的细节 [第五部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

cv8

CV8调试格式使用Microsoft Visual Studio  2005(8.0版)和是完全非法的,虽然之前CodeView格式有强烈的相似之处。  Yasm支持CV8调试格式目前限于生成程序行号信息(允许一定程度的源代码级调试)。 CV8调试信息存储在 .debug $ S 和 .debug $ T Win64对象的部分文件。

dwarf2

矮2是一个复杂的调试格式,记录调试信息的标准。  创建它在刺穿了克服缺点,允许更详细的和紧凑的描述数据结构,数据变量的运动,在C等复杂的语言结构 。  调试信息存储在部分(就像正常的程序部分)在对象文件。 Yasm支持完整的直通DWARF2调试信息(如从一个C 编译器),也可以生成程序行号信息。

零

“零”调试格式是一个占位符,它将没有调试信息添加到输出文件。

刺穿了

刺穿了调试格式是一个糟糕的记录,半标准COFF和ELF目标文件格式的调试信息。 调试信息存储对象文件的符号表的一部分,因此在复杂性和范围是有限的。 尽管如此,刺穿了是一种常见的调试格式在旧的Unix和兼容系统,以及DJGPP。

**第二章。 VSYASM——Yasm微软Visual Studio 2010**

**表的内容**

[2.1。 集成的步骤](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[2.2。 替代集成步骤](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[2.3。 使用VSYASM](http://www.tortall.net/projects/yasm/manual/html/manual.html)

构建系统中使用 微软Visual Studio 2010是基于 **MSBUILD** ,微软专门构建管理工具,这一变化要求外部工具集成到开发环境以一种新的方式。 **VSYASM** 了促进Yasm集成 Visual Studio 2010在一个健壮的和有效的方式。 VSYASM和其他版本之间的主要区别是,它能够组装多个源代码文件在一个命令行。

当组装单个文件VSYASM一样正常的行为 **yasm** 工具。 在这种情况下,唯一的变化是,VSYASM不仅仅提供预处理模式。



如果VSYASM命令行包括多个源文件,任何输出,列表和地图路径解析命令行上给出的目录组件单独和每个源代码文件然后组装使用这些目录相关的输出。 在大会开始之前,任何不存在VSYASM输出递归创建所需的目录。 装配过程本身停止如果任何文件被组装生成错误。

的 - e *文件* 命令行开关可以用来发送错误报告文件,在这种情况下,这个文件还包括用于调用VSYASM命令行。 这提供了一种方法来检查VSYASM被称为正确Visual Studio构建过程的控制。

**2.1。 集成的步骤**

首先,VSYASM可执行文件( vsyasm.exe )应该被添加到Visual Studio目录C工具。 这通常是:

C:\Program Files (x86)\Microsoft Visual Studio 10.0 \ VC \ bin

其次,三个文件— vsyasm.xml , vsyasm.props 和 vsyasm.targets ——应该添加到项目的项目目录中使用VSYASM(另一个将在稍后解释)。

第三,添加Yasm支持项目在IDE中打开项目后,在解决方案资源管理器中右键单击项目并选择“构建自定义…”。  如果vsyasm作为一个可选项在结果列表中你可以选择它,如果没有,使用“找到现有…”按钮,导航到结果文件对话 vsyasm.targets 你把在项目目录,将它添加到列表中选择它,然后从列表中选择它。

一旦你这样做了,在解决方案资源管理器中右键单击该项目并选择“属性”会弹出一个对话,一个新项“Yasm汇编程序”,将允许您配置Yasm构建任何汇编文件添加到项目中。

**2.2。 替代集成步骤**

如果你有使用VSYASM的很多项目,你可以把上面提到的三个文件放到MSBUILD的构建定制目录通常是在:

C:\Program Files (x86)\MSBuild\ Microsoft.Cpp \ v4.0 \ BuildCustomizations



VSYASM将总是可用在构建自定义对话。 一个替代的方法是把这些文件放在一个方便的位置,然后将这个位置的路径添加到“构建自定义搜索路径”项下“vc++项目设置”在Visual Studio 2010中选择对话。

**2.3。 使用VSYASM**

在Visual Studio项目汇编源代码文件,Yasm设置中的“Yasm汇编程序”项中输入的项目属性对话。 可用的物品符合那些Yasm上可用的命令行和主要是自我解释但一项——“对象文件名”——确实需要进一步解释。

如果“对象文件名”项指的是一个目录(默认),MSBUILD将收集所有的汇编文件在项目一起作为一个批处理和多个文件中调用VSYASM模式。 为了装配文件一次有必要改变输出的名字 **文件** 例如,如“$(IntDir)%(文件名).obj”。

**第二部分。 NASM语法**

书的这一部分文档中的章节NASM-compatible语法接受Yasm“nasm”解析器和预处理器。

**3.1。 NASM源代码行布局**

像大多数汇编、每个NASM源线包含(除非它是一个宏,预处理程序指令或一个汇编指令:明白了 [第五章](http://www.tortall.net/projects/yasm/manual/html/manual.html)一些组合的四个字段

标签:指令操作数;发表评论

像往常一样,这些字段是可选的;任何组合的存在与否的一个标签,一个指令和一个评论是被允许的。 当然,操作数的字段是必需的或禁止的存在和性质教学领域。

NASM使用反斜杠(\)行连续字符;如果一行与反斜杠结束,下一行被认为是backslash-ended线的一部分。

NASM的地方没有限制内的空白行:标签可能有空白之前,或指令可能没有空间在他们面前,或任何东西。 的 冒号后一个标签也是可选的。  注意,这意味着,如果你打算代码 lodsb 独自一行,和类型 lodab 偶然,那仍然是一个有效的源代码行这只定义一个标签。  NASM运行命令行选项 - w + orphan-labels 将导致它提醒你如果你定义一个标签单独在一行末尾没有冒号。

标签有效字符是字母、数字 _ , 美元 , # , @ , ~ , 。 , 吗? 。 唯一的字符可能被用作 *第一个* 字符标识符的字母,。 (有特殊意义:明白了 [3.9节](http://www.tortall.net/projects/yasm/manual/html/manual.html)), _ 和 吗? 。 一个标识符也可以用一个前缀 美元 表明它的目的是被解读为一个标识符,而不是保留字;因此,如果其他模块你链接定义了一个称为象征 eax ,你可以参考 eax美元 在NASM中代码来区分的标志寄存器。



指令字段可能包含任何机器指令:奔腾和P6指令,FPU指令,MMX指令甚至非法指令都是支持的。  指令可能前缀 锁, 代表 , REPE / REPZ 或 REPNE / REPNZ 在通常的方式。  显式地址大小和 operand-size前缀 系 , A32 , O16 和O32 被提供。  您还可以使用一个段寄存器的名称作为一个指令前缀:编码 es mov bx,斧头 相当于编码mov[es:bx),斧头 。  我们建议后者语法,因为它与其他语言的句法特征是一致的,但等指令 LODSB ,没有操作数和需要 段覆盖,没有干净的句法方法除了进行 es  lodsb 。

一条指令不需要用一个前缀:前缀等 CS , A32 , 锁 或 REPE 可以出现在一行本身,NASM只会生成前缀字节。

除了实际的机器指令,NASM还支持许多伪指令,所描述的 [3.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

指令 操作数可以采取许多形式:他们可以注册,只需注册名称(如描述。 斧头 , 英国石油公司 , EBX , CR0):NASM不使用 **气体** 风格的语法的寄存器名称必须由一个前缀 % 信号),也可以是 有效地址(见 [3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html))、常数( [3.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html))或表达式( [3.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

为 浮点指令,NASM接受广泛的语法:您可以使用two-operand形式像MASM支持实现,或者您可以使用NASM的本机single-operand形式在大多数情况下。 例如,您可以代码:

fadd相约;这集st0:= st0 +相约

fadd st0,相约;这也是如此

fadd相约,st0;这集相约:=相约+ st0

fadd相约;这也是如此

几乎所有的浮点指令引用内存必须使用前缀 双字 , QWORD , TWORD , DDQWORD ,或 OWORD 表明什么大小的((内存操作数)),它是指。

**3.2伪指令。**

伪指令的东西,虽然不是真正的x86机器指令,用于教学领域还是因为这是最方便的地方。  当前的 伪指令是 DB ,DW , DD , DQ , DT , DDQ , 做 ,未初始化的同行 RESB , RESW , RESD , RESQ , 休息 , RESDDQ , 资源 ,INCBIN 命令, 装备的 命令, 次 前缀。

**3.2.1之上。 DB 和朋友:宣布初始化数据**

DB , DW , DD , DQ , DT , DDQ , 做 用于声明输出文件中的数据初始化。 他们以各种各样的方式可以调用:

db 0 x55;只是x55字节0

db 0 x55 0 x56 0 x57;连续三个字节

db ' a ',0 x55;字符常量就可以了

10 db‘你好’,13日,“美元”,所以是字符串常量

dw 0 x1234;0 x34 0 x12

dw ' a ',0 x41 0 x00(它只是一个数字)

dw“ab”;0 x41 0×(字符常数)

dw“abc”;0 x41 0×0 x43 0 x00(字符串)

dd 0 x12345678;0 x78 0 x56 0 0 x12 x34

dq 0 x1122334455667788;0 x88 0 x77 0 x66 0 x55 0 x44 0 x33 0将0 x11

ddq 0 x112233445566778899aabbccddeeff00

;0 x00 0 xff 0 xee xdd 0 xcc 0 xbb 0 xaa 0 x99

;0 x88 0 x77 0 x66 0 x55 0 x44 0 x33 0将0 x11

和以前一样做0 x112233445566778899aabbccddeeff00;

dd 1.234567 e20;浮点常量

dq 1.234567 e20;双精度浮点数

dt 1.234567 e20;给出自由浮动

DT 不接受 数字常量作为操作数, DDQ 不接受浮点数常量作为操作数。 任何尺寸大于 DD 不接受字符串作为操作数。

**3.2.2。 RESB 和朋友:声明未初始化的数据**

RESB , RESW , RESD , RESQ , 休息 , RESDQ , 资源 设计中使用的BSS部分模块:他们申报的吗 *uninitialised* 存储空间。 每个需要一个操作数,它的字节数,话说,双字或任何保留。 NASM的MASM / TASM语法实现不支持保留uninitialised空间通过编写 DW吗? 或类似的东西:这就是它。 的操作数 RESB 类型伪指令是一个 **关键表达式**:看 [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

例如:

缓冲区:resb 64;储备64字节

wordvar:resw 1;储备一个字

realarray resq 10;十实数数组

**3.2.3。 INCBIN :包括外部的二进制文件**

INCBIN 包括一个二进制文件逐字到输出文件。 这可以方便(例如)包括 图形和 声音数据直接进入游戏可执行文件。 然而,推荐使用这只 *小* 件的数据。 它可以调用其中的一个方法:

incbin”文件。 dat”;包含整个文件

incbin”文件。 dat ",1024年,跳过第一个1024字节

incbin”文件。 dat ",1024年,1024年,跳过第一个1024年

实际上,包括最多512

**3.2.4。 装备的 :定义常量**

装备的 定义一个符号一个给定的常数的值:当 装备的 源线,必须包含一个标签。 的作用 装备的 是定义给定的标签名称(只)操作数的值。 这个定义是绝对的,不能改变。 举个例子,

db消息“hello,world”

美元msglen装备的消息

定义了 msglen 持续12。 msglen 以后可能不会被重新定义。 这不是一个 预处理器定义:的价值 msglen 评估 *一次*,使用的价值 美元 (见 [3.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html)一个解释的 美元 )的定义,而不是评估无论它引用和使用的价值 美元 的参考点。 注意,一个操作数 装备的 也是一个 关键表达式( [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

**3.2.5。 次 :重复指令或数据**

的 次 前缀多次导致指令进行组装。 这部分是NASM的等效的 DUP MASM-compatible汇编、语法支持的代码

zerobuf:* 64分贝0

或类似的事情,但是 次 比这更多功能。 的参数 次 不仅仅是一个数字常量,但是一个数字 *表达式* ,所以你能做的事情

缓冲区:db“hello,world”

乘以64美元+缓冲db ' '

这将存储足够的空间的总长度吗 缓冲 到64。 最后, 次 可以应用于普通的指令,所以你可以代码微不足道 它展开循环:

乘以100 movsb

请注意,没有有效的区别 乘以100 resb 1 和 resb 100 ,除了后者将快100倍的组装将汇编程序的内部结构。

的操作数 次 ,像这样的 装备的 和 RESB 和朋友,是一个关键表达式( [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

还请注意, 次 不能适用于 宏:原因是 次 处理后的宏观阶段,它允许的参数 次 包含表达式等 64美元+缓冲 如上所述。 重复多一行代码,或一个复杂的宏,使用预处理器 %代表 指令。

**3.3。 有效的地址**

一个 有效地址是引用内存的任何指令的操作数。 有效的地址,在NASM中,有一个非常简单的语法:他们由一个表达式评估所需的地址,封闭 方括号。 例如:

wordvar dw 123

mov ax,[wordvar]

mov ax,wordvar + 1

mov ax,[西文wordvar + bx):

任何不符合这个简单的系统不是一个有效的内存引用在NASM中,例如 es:wordvar(软) 。

更复杂的有效地址,如涉及多个寄存器,在完全相同的方式工作:

mov eax,(ebx * 2 +连成一片+偏移量)

mov ax,bp + di + 8

NASM就是这样做的能力 代数在这些有效的地址,这样的东西不一定 *看* 法律非常好:

mov eax,(ebx * 5),组装(ebx * 4 + ebx)

mov eax,label1 * 2-label2;即[label1 +(label1-label2)]

某些形式的有效地址有一个以上的组装形式;在大多数这样的情况下NASM将产生最小的形式。 例如,有不同的组装形式为32位的有效地址 (eax * 2 + 0) 和 [eax + eax] ,NASM通常会生成后者,因为前者需要4个字节来存储一个零偏移量。



NASM提示机制,这将导致 [eax + ebx] 和 [ebx + eax] 生成不同的操作码;这是偶尔有用,因为 (esi + ebp) 和(ebp +应急服务国际公司) 有不同的默认段寄存器。

但是,可以迫使NASM生成一个有效地址在一个特定的表单使用的关键词 字节 , 词 , 双字 和 NOSPLIT 。 如果你需要 (eax + 3) 组装使用一个双字偏移量字段,而不是一个字节NASM通常会生成,您可以代码 (dword eax + 3) 。  类似地,您可以迫使NASM使用字节抵消它还没有见过一个较小的值在第一次通过(见 [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html)例如这样的代码片段)通过使用 (字节eax +偏移量) 。 作为特殊情况, (字节eax) 将代码 (eax + 0) 字节偏移量为0, (dword eax) 将代码用双字抵消为零。 正常的形式, (eax) 将编码没有抵消。

前面的段落中描述的形式也很有用,如果你试图在一个32位访问数据从在16位代码段。 特别是,如果您需要访问数据与一个已知的抵消比适合在一个16位值大,如果你不指定,它是一个dword抵消,NASM将导致高抵消损失。

同样,NASM将分裂 (eax * 2) 成 [eax + eax] 因为允许缺席和空间偏移量字段得救;事实上,它也会分裂 (eax * 2 +偏移量) 成 (eax + eax +偏移量) 。 你可以通过使用打击这种行为 NOSPLIT 关键字: (nosplit eax *  2) 将迫使 (eax * 2 + 0) 生成。

**3.3.1。 64位的位移**

在 位64 模式,位移,在很大程度上,仍然是32位,扩展之前使用标志。  除了是一个mov指令的限制形式:之间的艾尔 , 斧头 , EAX ,或 递交 寄存器和一个64位的绝对地址(有效地址不允许注册,地址不能RIP-relative)。 在NASM中语法,需要使用64位的绝对形式 QWORD 。 在NASM中语法的例子:

mov eax,[1];32位,符号扩展

mov,[rax-1];32位,符号扩展

mov,[qword 0 x1122334455667788];64位绝对的

mov,[0 x1122334455667788];截断32位(警告)

**3.3.2。 把 相对寻址**

在64位模式下,一种新的有效的解决可以更容易编写位置无关代码。 任何内存可以参考 把 相对( 把 是指令指针寄存器,它包含的地址位置紧跟当前指令)。

在NASM中语法,有两种方法可以指定RIP-relative寻址:

mov dword(rip + 10),1

存储值1十字节后的指令。 10 也可以是象征性的常数,并将以同样的方式对待。 另一方面,

mov dword[symb关于撕裂),1

存储值1到符号的地址 symb 。 这是明显不同的行为:

mov dword symb +撕裂,1

将结束的指令的地址,添加的地址吗 symb ,那么存储值1。 如果 symb 这是一个变量,会吗 *不* 存储值1到 symb变量!

Yasm还支持以下语法RIP-relative寻址。 的 REL 关键字使它产生 把 相对地址, 腹肌 关键字使它产生非 把 相对地址:

mov rel信谊,伸展;RIP-relative

mov abs信谊,伸展,没有RIP-relative

的行为 mov(对称),递交 取决于设定的模式 默认的 指令(见 [5.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),如下所示。 在Yasm创业总是默认模式 腹肌 ,在 REL 模式,使用寄存器, FS 或 GS 段覆盖,或一个显式的 腹肌 覆盖将导致non-RIP-relative有效地址。

默认rel

mov(对称),rbx;RIP-relative

mov abs信谊,rbx;不是RIP-relative(显式重写)

mov[rbx + 1],rbx;不是RIP-relative(寄存器使用)

mov[fs:信谊],rbx;不是RIP-relative(fs或gs使用)

mov ds:信谊,rbx;RIP-relative(段,但不是fs或gs)

mov rel信谊,rbx;RIP-relative(冗余覆盖)

默认的腹肌

mov(对称),rbx;不是RIP-relative

mov abs信谊,rbx;不是RIP-relative

mov[rbx + 1],rbx;不是RIP-relative

mov[fs:信谊],rbx;不是RIP-relative

mov ds:信谊,rbx;不是RIP-relative

mov rel信谊,rbx;RIP-relative(显式重写)

**3.4。 直接操作数**

直接操作数在NASM中可能是8位,16位,32位,甚至64位大小。 眼前的大小可以直接通过使用指定的 字节 , 词 ,或 双字 关键词,分别。

64位操作数仅限于直接64位寄存器指令在移动 位64 模式。 对于所有其他指令在64位模式下,立即值仍然是32位,他们的价值符号扩展到32位的目标寄存器之前被使用。 例外是mov指令,可以把一个64位的直接目标是一个64位的寄存器。

所有未分级立即值 位64 在Yasm默认32位大小一致性。 为了得到一个64位的直接与一个标签,显式地指定大小的 QWORD 关键字。 使用的方便性,Yasm还将试图识别64位值和自动改变大小64位这些情况。

在NASM中语法的例子:

添加递交1;优化签署了8位

添加伸展,dword 1;迫使大小32位

添加伸展,0 xffffffff;符号扩展32位

添加递交1;同上

加伸展,0 xffffffffffffffff;截断32位(警告)

mov eax,1;5字节

mov伸展,1、5字节(32位)签署的优化

mov伸展,qword 1;10字节(64位)

mov rbx,0 x1234567890abcdef;10字节

mov rcx,0 xffffffff;10个字节(32位)签署不符合

mov连成一片,1;5字节,相当于上面

mov rcx,信谊;5字节,32位大小默认符号

mov rcx,qword sym;10字节,覆盖默认大小

谨慎的用户使用NASM和Yasm 2。 x:mov reg64的处理,未分级立即Yasm NASM 2之间是不同的。 x;YASM遵循上述行为,NASM 2。 x是以下几点:

添加伸展,0 xffffffff;符号扩展32位立即

添加递交1;同上

添加伸展,0 xffffffffffffffff;截断32位(警告)

添加伸展,信谊;符号扩展32位立即

立即mov eax,1、5字节(32位)

mov递交1;10字节立即(64位)

mov rbx,0 x1234567890abcdef;10字节的指令

mov rcx,0 xffffffff;10字节的指令

mov连成一片,1;5字节,相当于上面

立即mov连成一片,对称;5字节(32位)

mov rcx,信谊;10字节(64位直接)

mov rcx,qword sym;10字节,同上

**3.5 .常量**

NASM理解四个不同类型的常数:数字、字符、字符串和浮点。

**3.5.1。 数字常量**

数字常数只是一个数字。  NASM允许您指定数字在不同的基地,以多种方式:你可以后缀 H , 问 或 O , B 为 十六进制, 八进制, 二进制文件,或者你可以前缀 0 x 十六进制C风格的,或者你可以前缀 美元 十六进制的宝蓝帕斯卡的风格。 但是,请注意, 美元前缀是双重任务标识符(见一个前缀 [3.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),所以一个十六进制数字前缀 美元 签署后必须有一个数字 美元 而不是一个字母。

一些例子:

mov ax,100;小数

mov ax,0 a2h;十六进制

mov ax,$ 0 a2;又十六进制:0是必需的

再次mov ax,0 xa2;十六进制

mov ax,777 q;八进制

mov ax,777 o;八进制

mov ax,10010011 b,二进制

**3.5.2。 字符常量**

字符常量由四字符括在单引号或双引号。 NASM的类型引用没有区别,当然,除了周围不断允许双引号和单引号出现在它,反之亦然。

一个常数与多个字符将安排 低位优先顺序记住:如果你的代码

mov eax,“abcd”

然后不断生成的不是 0 x61626364 ,但 0 x64636261 ,所以,如果你被存储到内存的值,它将阅读 abcd 而不是dcba 。 这也是可以理解的字符常量奔腾的感觉 CPUID 指令。

**3.5.3。 字符串常量**

字符串常量只接受一些伪指令,即 DB 家庭和 INCBIN 。

字符串常量看起来像一个字符常数,只长。 视为一个连接的最大大小字符常量的条件。 所以以下是等价的:

db‘你好’,字符串常量

db‘h’,‘e’,‘l’,‘l’,‘o’;等效字符常量

和下面的也相当于:

dd ninechars,双字串常数

dd“9”,“字符”,“年代”,变成了三个双字

db ninechars,0,0,0,真的是这样的

注意,当作为操作数 db ,一个常数 “ab” 被视为一个字符串常量尽管是足够短字符常量,否则吗 db“ab” 会有同样的效果 db ' a ' ,这将是愚蠢的。 同样,三或四字符常量是被当作字符串操作数时 dw 。

**3.5.4。 浮点型常量**

浮点型常量只接受作为参数 DW , DD , DQ 和 DT 。  他们以传统的形式表达:数字,那么一段时间,然后可以选择更多的数字,然后选择一个 E 其次是一个指数。  周期是强制性的,因此,NASM可以区分 dd - 1 声明一个整数常数, dd 1.0 声明一个浮点型常量。

一些例子:

dw -0.5;IEEE一半精度

dd 1.2;一个简单的

dq - 1。 e10,10000000000

dq - 1。 e + 10;1. e10的同义词

dq - 1。 平台以及;0.000 000 000 1

dt 3.141592653589793238462;π

NASM无法编译时算术浮点常量。  这是因为NASM被设计成便携式——尽管它总是生成代码运行在x86处理器,汇编程序本身可以在任何系统上运行的ANSI C编译器。  因此,汇编程序不能保证的浮点单元处理的能力 英特尔数字格式,所以NASM能够做浮动算术它必须包括自己的成套浮点例程,这将大大增加很少好处的汇编程序的大小。

**3.6 .表达式**

表达式在NASM中在语法与C是相似的。

NASM的大小并不能保证整数用来评估表达式在编译时间:因为NASM可以编译和运行在64位系统上很快乐,不要以为表达式是评估在32位寄存器,所以试着故意利用((整数溢出))。 它可能并不总是工作。 NASM唯一保证的担保的ANSI C:你总是 *至少* 32位的工作。

NASM支持两种特殊标记表达式,允许计算涉及当前装配位置: 美元 和 $  $ 令牌。 美元 评估到组装线的位置开始时包含表达式;所以你可以编写一个 无限循环使用 人民币美元 。 $  $ 评估当前的开始部分,所以你可以告诉你是多么遥远的部分使用 ($ - $ $) 。

算术 运营商提供的NASM列出,递增的顺序 优先级。

**3.6.1。 | :按位或运算符**

的 | 运营商提供了 位,或者所执行的完全一样 或 机器指令。 按位或NASM支持的优先级运算符。

**操作。 ^ :按位异或运算符**

^ 提供了 按位异或操作。

**3.6.3。 & :按位与运算符**

& 提供了 位和操作。

**3.6.4。 < < 和 > > :移位操作符**

< < 给出了一个向左移位,正如它在c 5 < < 3 评估5 * 8,或40。 > > 给出了一个向右移位,在NASM中,这种转变*总是* 无符号,从左端位转移充满了零而不是符号扩展之前的最高位。

**3.6.5。 + 和 - - - - - - :加法和减法运算符**

的 + 和 - - - - - - 运营商做的很普通 加法和 减法。

**3.6.6。 \* , / , / / , % 和 % % :乘法和除法**

\* 是 乘法运算符。 / 和 / / 都是 除法运算符: / 是 无符号除法和 / / 是 部门签字。 同样的, % 和 % % 提供无符号和签署模运算符。

NASM ANSI C一样,没有提供担保的合理的操作签名模运算符。

自 % 性格是广泛使用的宏观预处理器,您应该确保签署和无符号模运算符都是紧随其后的是空白。

**3.6.7。 一元操作符: + , - - - - - - , ~ 和 赛格**

NASM的最高优先级的操作符的表达式语法是那些只适用于一个参数。 - - - - - - 否定它的操作数, + 什么也不做(它提供对称- - - - - - ), ~ 计算 一个补充的操作数, 赛格 提供了 段地址的操作数(更详细地解释道 [部分3.6.8](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

**3.6.8。 赛格 和 关于**

在编写大型16位程序时,必须分成多个 段,它常常需要能够参考段地址的符号的一部分。 NASM支持 赛格 操作符来执行这个函数。

的 赛格 操作符返回 *首选* 段的一个符号,定义为段基础的相对偏移的象征意义。 因此代码

mov ax,凹陷的象征

mov,斧头

mov bx,象征

将负载 es:bx 使用有效的指针的象征 象征 。

东西可以比这更复杂:因为16位段 组织可能会重叠,你可能会偶尔想引用一些符号使用不同的段基地从首选之一。 NASM让你做到这一点,通过使用关于 (参照)关键字。 所以你能做的事情

mov ax,weird_seg;weird_seg段基地

mov,斧头

关于mov bx,象征weird_seg

加载 es:bx 不同,但功能等同,指针的象征 象征 。

NASM支持远(inter-segment)调用和跳跃的语法 段:抵消 ,在那里 段 和 抵消 表示立即值。 所以打电话给一个程序,你可以代码

调用(赛格过程):过程

叫weird_seg:(过程关于weird_seg)

(包括括号是为了清楚起见,显示意图解析上述指令。 他们在实践中则不需要。)

NASM支持的语法 叫得过程 作为第一个上面的用法的同义词。 无条件转移指令 作品相同 调用 在这些例子中。

声明一个 指针指向一个数据项在数据段中,你必须的代码

dw的象征,凹陷的象征

NASM支持没有方便的同义词,尽管你可以发明一个使用宏处理器。

**3.7。 严格的 :抑制优化**

当装配优化器设置为2级或更高,NASM使用大小说明符( 字节 , 词 , 双字 , QWORD ,或 TWORD ),但会给他们尽可能最小的大小。 关键字 严格的 可以用来抑制优化和力量发出的一个特定的操作数指定的大小。 例如,优化器, 16位 模式,

推动dword 33

三个字节编码 66 6 21 ,而

推动严格dword 33

6个字节进行编码,一个完整的字直接操作数 21 66 68 00 00 00 。

**3.8。 关键表达式**

NASM的局限性在于,它是一个 双行程汇编;与TASM和其他人不同,它总是会做两个 大会通过。 因此不能应付复杂到需要源文件,三个或更多 传球。

第一遍是用来确定所有组装的代码和数据的大小,所以第二步,当生成的所有代码,知道所有的符号地址指的代码。 所以NASM无法处理的一件事是代码的值,其大小取决于一个象征宣布后的代码问题。 例如,

*(标签-美元)db 0

标签:db“我在哪儿?”

的参数 次 在这种情况下可以同样合法评估任何东西;NASM将拒绝这个例子,因为它不能告诉的大小 次 当它第一次见到它。 它将同样坚定拒绝slightlyparadoxical代码

(标签- $ + 1)db 0

标签:db“现在我在哪儿?”

在这 *任何* 值 次 参数是由定义错了!

NASM拒绝这些例子通过称为概念 **关键表达式** ,这是定义为一个表达式的值是需要在第一遍可计算的,因此必须只取决于符号定义。 的参数的 次 前缀是一个关键的表情;出于同样的原因,参数 RESB 伪指令的家庭也关键表达式。

关键表达式可以出现在其他上下文:考虑下面的代码。

mov ax,symbol1

symbol2 symbol1装备

symbol2:

在第一次通过,NASM无法确定的价值 symbol1 ,因为 symbol1 定义等于什么 symbol2 NASM尚未看到。  在第二步中,因此,当它遇到 mov ax,symbol1 ,它不能生成的代码,因为它仍然不知道的价值 symbol1 。  下一行,就会看到 装备的 又能够确定的价值 symbol1 ,但那时就太晚了。

NASM避免了这个问题通过定义的右侧 装备的 声明一个关键表达式,这样的定义 symbol1 将在第一遍被拒绝。

有一个涉及相关问题 向前引用:考虑这个代码片段。

mov eax,ebx +偏移量

抵消装备的10

NASM中,通过一个,必须计算指令的大小 mov eax,ebx +偏移量 不知道的价值 抵消 。  它没有办法知道 抵消 足够小,适合1字节偏移量字段,因此它可以生成一个短形式的 有效地址编码;它知道,在通过一个, 抵消 在代码段可能是一个象征,它可能需要完整的四字节形式。 所以不得不计算指令的大小,以适应一个四字节地址部分。  在通过两个,这个决定,现在不得不尊重它并保持指令大,所以在这种情况下生成的代码不是一样小。  这个问题可以通过定义来解决 抵消 前使用它,或通过迫使字节大小的有效地址编码 (字节ebx +偏移量) 。

**3.9。 当地的标签**

NASM给符号开始的特殊待遇 时期。 一个标签开始被当作一个单一的时期 *当地的* 标签,这意味着它与前面的非本地标签。 举个例子:

label1;一些代码

。 循环;更多的代码

jne .loop

受潮湿腐烂

label2;一些代码

。 循环;更多的代码

jne .loop

受潮湿腐烂

在上面的代码片段中, JNE 立即指令跳线之前,因为这两个定义的 .loop 是分开的每一个与前面的非本地标签。

NASM更进一步,在允许访问本地标签代码的其他部分。 这是通过的 *定义* 当地一个标签在前面的非本地标签:第一个定义 .loop 以上是定义一个符号 label1.loop ,第二个定义了一个称为象征 label2.loop 。 所以,如果你真的需要,你可以写

label3;更多的代码

,更多的

jmp label1.loop

有时是有用的——在一个宏,例如,能够定义一个标签,可以从任何地方引用但不干扰正常local-label机制。  这样一个标签不能非本地,因为它会干扰后续的定义,和引用,当地地方标签;不能因为定义的宏,它不知道标签的全名。  NASM因此介绍第三种类型的标签,这只可能是有用的在宏定义:如果一个标签开头的特殊前缀 . . @ ,那么它无助于当地标签机制。 所以你可以代码

label1:;非本地标签

。 本地:,这是真的label1.local

. . @foo:,这是一个特殊的符号

label2:;另一个非本地标签

。 本地:,这是真的label2.local

人民币. . @foo;这将跳三行

NASM有能力定义其他特殊符号开头一段双:例如, 开始. . 用于指定入口点的吗 obj 输出格式。

**第四章。 NASM预处理器**

**表的内容**

[4.1。 单行的宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[以下4.4.1。 正常的方法: %定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.1.2。 提高%定义: % xdefine](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.1.3。 连接一行宏观标记: % +](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.1.4。 它通过宏: % undef](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.1.5。 预处理器变量: %分配](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.2。 字符串处理在宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.2.1。准备 字符串长度: % strlen](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.2.2。 子: %的子串](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3。 多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.1。 重载多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.2。 Macro-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.3。 贪婪的宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.4。 默认宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.5。 % 0 柜台:宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.6。 %旋转 :旋转宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.7。 连接宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[为4.3.8。 码作为宏观参数条件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.3.9。 禁用清单扩张](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4。 有条件的组装](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.1.1。 %如果定义了 :测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[10/24/11。 % ifmacro :测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4.3。 % ifctx :测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4.4。 %如果 :测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4.5。 % ifidn 和 % ifidni :测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4.6。 % ifid , % ifnum , % ifstr :测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.4.7。 %的错误 :用户定义的错误报告](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.5。 预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.6。 包括其他文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7。 上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7.1。 %推 和 %的流行 :创建和删除背景](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7.2。 Context-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7.3。 Context-Local单行的宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7.4。 % repl :重命名上下文](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.7.5。 IFs示例使用上下文堆栈:块](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8。 标准宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.1。 __YASM_MAJOR__ 等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.2。 __FILE__ 和 __LINE__ :文件名和行号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.3。 __YASM_OBJFMT__ 和 __OUTPUT_FORMAT__ 关键字:输出对象格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.4。 STRUC 和 ENDSTRUC :声明结构数据类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.5。 ISTRUC , 在 和 IEND :声明实例的结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[4.8.6。 对齐 和 ALIGNB :数据对齐](http://www.tortall.net/projects/yasm/manual/html/manual.html)

NASM包含一个强大的 宏处理器,支持有条件的组装、多层次的文件包含,两种形式的宏观(单行和多行),和一个额外的宏观权力的“上下文堆栈”机制。 预处理器指令所有从一开始 % 的迹象。

预处理程序崩溃所有行最后一个反斜杠(\)字符到一行。 因此:

%定义THIS_VERY_LONG_MACRO_NAME_IS_DEFINED_TO \

THIS_VALUE

将像一个单行的宏没有backslash-newline序列。

**4.1。 单行的宏**

**以下4.4.1。 正常的方法: %定义**

单行的宏定义使用 %定义 预处理器指令。 定义工作类似于C,所以你能做的事情

%定义ctrl 0 x1f &

%定义参数(a,b)(()+()*(b))

mov byte[参数(ebx)],ctrl ' D '

这将扩大到

mov byte[(2)+(2)*(ebx)],0 x1f & ' D '

当一个单行的宏观的扩张包含标记调用另一个宏,扩张是在调用时执行,而不是在定义的时间。 因此,代码

%定义(x)1 + b(x)

%定义b(x)2 * x

mov ax,(8)

将评估预期的方式 mov ax,1 + 2 * 8 ,即使宏 b 没有定义时定义 一个 。

宏定义 %定义 是 区分大小写:在 %定义foo酒吧 ,只有 喷火 将扩大 酒吧 : 喷火 或 喷火 不会。 通过使用 %  idefine 而不是 %定义 (“我”代表“不敏感”),您可以定义的所有情况下变异宏,这样 % idefine  foo酒吧 会导致 喷火 , 喷火 , 喷火 , 喷火 等等所有扩展 酒吧 。

有一个机制,检测宏调用时发生的前一个扩张的宏观,防范 循环引用,无限循环。 如果发生这种情况,预处理器只会扩大宏的第一次出现。 因此,如果你的代码

%定义(x)1 +(x)

mov ax,(3)

宏 (3) 将扩大一次,成为 1 +(3) ,然后没有进一步扩张。 这种行为可以是有用的。

你可以超载单行的宏:如果你写

%定义foo(x)1 + x

%定义foo(x,y)1 + x * y

预处理器能够处理两种类型的宏调用,通过计算参数传递,所以 foo(3) 将成为 1 + 3 而 foo(ebx,2) 将成为 1 + ebx * 2 。 然而,如果您定义

%定义foo酒吧

然后其他的定义 喷火 将被接受:不带参数宏禁止相同的名称作为一个宏的定义 *与* 参数,反之亦然。

这并不阻止单行的宏 *重新定义* :你可以很好地定义一个宏

%定义foo酒吧

然后重新定义在相同的源文件

%定义foo巴兹

然后到处都是宏观的 喷火 被调用时,它将根据最近的定义扩展。 这是特别有用的在定义单行的宏 %分配 (见[4.1.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

你可以 预定义单行宏使用“- d”选项Yasm命令行:明白了 [1.3.3.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**4.1.2。 提高%定义: % xdefine**

有参考嵌入式单行的宏观解决当时嵌入,而不是调用宏扩展时,你需要一个不同的机制来提供的 %定义 。 解决方案是使用 % xdefine ,或者它不区分大小写 % xidefine 。

假设你有下面的代码:

%定义isTrue 1

%定义isFalse isTrue

%定义isTrue 0

val1:db isFalse

%定义isTrue 1

val2:db isFalse

在这种情况下, val1 = 0,然后呢 val2 = 1。 这是因为,当一个单行的宏定义使用 %定义 ,它是只有当它被称为扩展。 作为 isFalse 扩大到 isTrue ,扩张的当前值 isTrue 。 它被称为是0,第一次和第二次是1。

如果你想要 isFalse 扩大价值分配给嵌入式宏 isTrue 在那个时候, isFalse 被定义,您需要更改使用上面的代码吗 % xdefine 。

% xdefine isTrue 1

% xdefine isFalse isTrue

% xdefine isTrue 0

val1:db isFalse

% xdefine isTrue 1

val2:db isFalse

现在,每一次 isFalse 被调用时,它扩大到1,这就是嵌入式宏 isTrue 扩展到的时候 isFalse 被定义。

**4.1.3。 连接一行宏观标记: % +**

个人令牌一行宏可以连接,生产时间标记为以后处理。 这可能是有用的,如果有几个类似宏执行类似的功能。

作为一个例子,考虑以下:

%定义BDASTART 400 h;BIOS数据区域的开始

struc tBIOSDA;它的结构

。 COM1addr RESW 1

。 COM2addr RESW 1

,. . 等等

endstruc

现在,如果我们需要访问tBIOSDA的元素在不同的地方,我们可以得到:

mov ax,BDASTART + tBIOSDA.COM1addr

mov bx,BDASTART + tBIOSDA.COM2addr

这将变得很丑(乏味)如果使用在许多地方,可以显著缩小,通过使用以下宏:

;宏观BIOS变量的访问他们的名字(从tBDA):

%定义BDA(x)BDASTART + tBIOSDA。 % + x

现在上面的代码可以写成:

mov ax,BDA(COM1addr)

mov bx,BDA(COM2addr)

使用这个特性,我们可以简化引用大量的宏(反过来,减少打字错误)。

**4.1.4。 它通过宏: % undef**

单行的宏可以删除的 % undef 命令。 例如,下列顺序:

%定义foo酒吧

% undef foo

mov eax,foo

将扩大指令 mov eax,foo 后,因为 % undef 宏 喷火 不再是定义。

宏,否则是预定义的定义可以在命令行上使用- u”选项Yasm命令行:明白了 [部分1.3.3.5](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**4.1.5。 预处理器变量: %分配**

另一种方式来定义通过单行的宏 %分配 命令(和它的不区分大小写的 % iassign ,它不同于 %分配 以完全相同的方式 % idefine 不同于 %定义 )。

%分配 用于定义单行的宏,不带任何参数,数值。 这个值可以指定一个表达式的形式,它将评估一次,当 %分配处理指令。

就像 %定义 宏定义使用 %分配 可以界定之后,你可以做吗

%分配我+ 1

增量的数值宏。

%分配 是用于控制终止 %代表 预处理程序循环:看 [4.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html)一个这样的例子。

表达式传递给 %分配 是一个 关键表达式(见 [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),还必须评估一个纯粹的数量(而不是一个浮动参考如代码或数据地址,或任何涉及注册)。

**4.2。 字符串处理在宏**

通常是有用的在宏能够处理字符串。 NASM支持两个简单的字符串处理宏观运营商可以构造更复杂的操作。

**4.2.1。准备 字符串长度: % strlen**

的 % strlen 宏就像 %分配 宏,它创建(或重新定义)一个宏的数值。 不同之处在于, % strlen 数值是一个字符串的长度。 使用的一个例子是:

% strlen charcnt '我的字符串

在这个例子中, charcnt 将获得的价值,就像一个吗 %分配 已经被使用。 在这个例子中, “我的字符串” 是一个文字字符串,但它也可以是一个单行的宏,它扩大到一个字符串,如以下示例:

%定义sometext“我的字符串”

% strlen charcnt sometext

在第一种情况下,这将导致 charcnt 被分配的值8。

**4.2.2。子: %的子串**

单个字母字符串可以提取使用 %的子串 。 其使用的一个例子可能是更有用的描述:

% substr mychar“xyz”1;相当于%定义mychar“x”

% substr mychar“xyz”2;相当于%定义mychar‘y’

% substr mychar“xyz”3;相当于%定义mychar ' z '

在这个例子中,mychar得到的价值 “y” 。 与 % strlen (见 [4.2.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),第一个参数是要创建的单行的宏,第二个是字符串。 第三个参数指定要选择哪个角色。 注意,第一个指数是1,而不是0,最后指数等于价值 % strlen 将分配给相同的字符串。 索引值的范围导致一个空字符串。

**4.3。 多行宏**

多行宏更像的一种宏观MASM和TASM:实现一个多行宏定义在NASM中看起来是这样的。

%宏观前言1

推动ebp

mov ebp,特别是

子esp,% 1

% endmacro

这个定义了一个c函数序言作为一个宏:所以你会调用宏的调用等

myfunc:序言12

这将扩大到三行代码

myfunc:推动ebp

mov ebp,特别是

子esp,12

数量 1 宏名称 %的宏 行定义了宏的参数个数 序言 期望获得。 的使用 % 1 在宏定义是指宏调用的第一个参数。 随着宏观多个参数,后续的参数将被称为 % 2 , % 3 等等。

多行宏,单行的宏,等 区分大小写,除非你使用替代指令定义它们 % imacro 。

如果你需要通过一个逗号 *部分* 多行宏的参数,你可以通过将整个参数包含在括号。 所以你可以代码之类的东西

%宏观傻2

% 2:db % 1

% endmacro

愚蠢的“a”,letter_a;letter_a:db ' a '

愚蠢的“ab”string_ab;string_ab:db“ab”

愚蠢的{ 13日10 },crlf;crlf:10 db 13日

**4.3.1。 重载多行宏**

与单行的宏,可以重载多行宏定义相同的宏名多次与不同数量的参数。 这一次,不例外是为宏不带参数。 所以你可以定义

%宏观序言0

推动ebp

mov ebp,特别是

% endmacro

定义函数的另一种形式的序幕分配没有本地堆栈空间。

但是,有时候您可能想要“超负荷”机器指令;例如,您可能需要定义

%宏观推2

推动% 1

推动% 2

% endmacro

这样你可以代码

推动ebx;这一行不是一个宏调用

推动eax,连成一片,但是这个是

通常,NASM将给予警告第一上面的两行 推 现在定义一个宏,调用的参数没有给出定义。 仍然会生成正确的代码,但是汇编程序将给予警告。 可以禁用这个警告的使用 -wno-macro-params 命令行选项(请参见 [1.3.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

**4.3.2。 Macro-Local标签**

NASM允许您定义标签在一个多行宏定义等方式让他们当地的宏调用:所以调用相同的宏观多次每次都将使用一个不同的标签。 这可以通过加前缀% % 标签的名字。 所以你可以发明一个执行一个指令 受潮湿腐烂 如果 Z 国旗由这样做:

%宏观retz 0

jnz % %跳过

受潮湿腐烂

% %跳过:

% endmacro

可以调用这个宏你想要很多次,每次你叫它NASM将构成不同的“真实”的名字来代替标签 % %跳过 。 NASM发明名称的形式 . .  @2345.skip ,2345号变化与每一个宏调用。 的 . . @ 前缀阻止macro-local标签干扰当地标签机制,描述 [3.9节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。 你应该避免这种形式(定义自己的标签 . . @ 前缀,然后一个数字,然后另一个时期),以防他们干扰macro-local标签。

**4.3.3。 贪婪的宏参数**

有时是有用的定义一个宏块整个命令行成一个参数定义,可能后提取一个或两个小参数的前面。 一个例子可能是一个宏来编写一个文本字符串在ms - dos文件,您可能希望能够编写

writefile(文件句柄),“你好,世界”,13日10

NASM允许您定义一个宏的最后一个参数 *贪婪的* ,也就是说,如果您调用宏比预计更多的参数,定义的所有多余的参数集总进入最后一个逗号分开。 所以如果你代码:

%宏writefile 2 +

jmp % % endstr

% % str:db % 2

% % endstr:

mov dx,% % str

mov残雪,% % endstr - % % str

mov bx,% 1

mov啊,0 x40

int 0 x21

% endmacro

然后调用的例子 writefile 以上将按预期工作:第一个逗号前的文本 (文件句柄) 作为第一个宏观参数和扩大 % 1 是指,所有随后的文本集中到 % 2 后,把 db 。

贪婪的宏观性质是NASM使用的表示 + 签署后的参数依赖 %的宏 线。

如果你定义一个贪婪的宏,你实际上是告诉NASM如何扩大宏 *任何* 指定的参数的数量与实际数量无穷;在这种情况下,例如,NASM现在知道当它看到调用该做什么 writefile 2、3、4或更多的参数。 NASM重载时将考虑这个宏,并将不允许您定义的另一种形式 writefile 采取4参数(例如)。

当然,上面的宏可以被实现为一个贪婪的宏,在这种情况下,它必须看起来像

writefile(文件句柄),{ 10 }“hello,world”,13日

NASM提供机制(宏观参数)(逗号),你选择哪一个你更喜欢每个宏定义。

看到 [5.3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)有一种更好的方法来写上面的宏。

**4.3.4。 默认宏参数**

NASM还允许您定义一个多行宏了 *范围* 容许的参数计算。 如果你这样做,您可以指定默认值 省略参数。 举个例子:

%宏观死0 - 1”痛苦的程序死亡发生。”

writefile 2,% 1

x4c01 mov ax,0

int 0 x21

% endmacro

这个宏(利用 writefile 中定义的宏 [4.3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html))可以被称为一个显式的错误消息,它将显示在退出前错误输出流,也可以不带参数调用,在这种情况下,将使用默认错误消息中提供的宏定义。

一般来说,你提供一个最小和最大数量的这种类型的宏参数;所需的最小数量的参数然后在宏调用,然后你提供可选的默认值。 所以如果一个宏定义开始

% 1 - 3宏观foobar eax,(ebx + 2)

然后它可以称为一至三个参数,和 % 1 总是来自宏调用。 % 2 宏调用,如果未指定,默认 eax , % 3 如果没有指定默认的 (ebx + 2) 。

你可以省略参数从宏定义默认值,在这种情况下,参数默认是空白。 这可能是有用的宏可以数量可变的参数,自% 0 令牌(见 [4.3.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html))允许您确定真的有多少参数传递给宏调用。

这种违约机制可以结合greedy-parameter机制;因此, 死 宏观上可以更强大和更有用,通过改变的第一行定义

%宏观死0 - 1 +“痛苦的程序死亡发生。”,13日10

最大参数计数可以无限,用 * 。 当然,在这种情况下,提供一个是不可能的 *完整的* 默认的参数集。 这种用法的例子所示 [4.3.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**4.3.5。 % 0 柜台:宏参数**

的宏可以采取数量可变的参数,参数的参考 % 0 将返回一个数字常数给宏传递的参数的数量。 这可以作为参数 %代表 (见[4.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),以遍历所有参数的宏。 给出了例子 [4.3.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**4.3.6。 %旋转 :旋转宏参数**

Unix shell程序员都熟悉 转变 shell命令,允许参数传递给shell脚本(引用 1美元 , 2美元 等等)要搬到一个地方,留下的,以前引用的论点 2美元 可用的 1美元 和之前引用的论点 1美元 不再可用。

NASM提供了一个类似的机制,在形式的 %旋转 。 顾名思义,它不同于Unix 转变 在不丢失参数:参数旋转的左端参数列表出现在右边,反之亦然。

%旋转 与一个数字参数被调用(这可能是一个表达式)。 宏观参数旋转的留下的很多地方。 如果参数 %旋转 是负的,宏观参数向右旋转。

所以一双宏的保存和恢复的一组寄存器可能工作如下:

%宏观multipush 1 - *

%代表% 0

推动% 1

%旋转1

% endrep

% endmacro

这个宏调用 推 指令在它的每个参数反过来,从左到右。 它首先将第一个参数, %  1 ,然后调用 %旋转 向左移动一个地方的所有争论,现在原来的第二个参数是可用的 % 1 。 这个过程重复多次有参数(通过提供实现 %  0 作为参数, %代表 )导致每个参数反过来推动。

还请注意使用 * 作为最大的参数计算,表明没有上限参数的数量你可以供应 multipush 宏。

这将是方便,使用这个宏时,有一个 流行 等价的, *没有* 需要给定的参数在相反的顺序。 理想情况下,你会写multipush 宏调用,然后复制粘贴到流行需要做些什么,和改变的名字叫做宏 multipop ,宏会照顾弹出注册以相反的顺序从一个推动。

这可以通过以下定义:

%宏观multipop 1 - *

%代表% 0

%旋转1

流行% 1

% endrep

% endmacro

这个宏开始旋转它的参数的一个地方 *正确的* ,所以原来的 *去年* 参数显示为 % 1 。 这是那么突然,参数是旋转的,所以变成了倒数第二个论点 % 1 。 因此,参数在相反的顺序遍历。

**4.3.7。 连接宏参数**

NASM可以连接宏观参数对其他文本周围。 这允许您声明一个家族的象征,例如,在一个宏定义。 例如,如果你想生成一个表的键码连同补偿到表中,你可以类似的代码

%宏观keytab_entry 2

keypos % 1装备keytab美元

db % 2

% endmacro

keytab:

keytab_entry F1,128 + 1

keytab_entry F2,128 + 2

keytab_entry回报,13

这将扩大到

keytab:

keytab美元keyposF1装备

db 128 + 1

keytab美元keyposF2装备

db 128 + 2

keytab美元keyposReturn装备

db 13

您可以简单地将文本的另一端一个宏观参数,通过编写 % 1 foo 。

如果你需要添加一个 *数字* 一个宏观参数,例如定义标签 foo1 和 foo2 当的参数传递 喷火 ,你不能代码 % 11 因为这是作为第十一个宏观参数。 相反,您必须的代码 % { 1 } 1 将单独的第一 1 (给宏观参数的数量)从第二(文字文本连接到参数)。

这个连接也可以应用于其他预处理内联对象,如macro-local标签( [4.3.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html))和context-local标签( [4.7.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 在所有情况下,语法歧义可以解决封闭后的一切 % 标志和文字文本前括号: % { % foo  }酒吧 连接的文本 酒吧 到最后的真实姓名macro-local标签 % % foo 。  (这是不必要的,因为形式NASM使用真实姓名的macro-local标签意味着两种用法% { % foo }酒吧 和 % %  foobar 既能扩大到相同的;然而,能力就在那里。)

**为4.3.8。 码作为宏观参数条件**

NASM能给特殊待遇宏观参数包含一个条件代码。 首先,您可以参考宏观参数 % 1 通过另一种语法 % + 1 ,通知NASM这个宏参数应该包含一个条件代码,并将导致预处理器报告一个错误消息,如果宏观叫做一个参数 *不* 一个有效的状态码。

不过,更有用的是,您可以参考宏观参数通过 % 1 NASM将扩大 *逆* 状态代码。 因此, retz 中定义的宏 [4.3.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)取而代之的是一个通用吗 conditional-return宏:

宏观retc % 1

j % -1% %跳过

受潮湿腐烂

% %跳过:

% endmacro

现在可以使用调用调用这个宏 retc不 ,这将导致条件转移指令在宏扩展出来 我 ,或 retc阿宝 这将使跳一个吗JPE 。

的 % + 1 macro-parameter引用是很乐意解释参数 CXZ 和 ECXZ 然而,随着代码有效条件; % 1 将报告一个错误如果通过这些,因为不存在逆状态码。

**4.3.9。 禁用清单扩张**

NASM时生成一个清单文件从您的程序中,它通常会扩大通过编写多行宏宏调用清单的每一行,然后扩张。 这允许您看到宏扩展指令的生成代码;然而,对于一些宏这不必要的杂波的清单。

NASM因此提供了 .nolist 限定符,您可以包括在一个宏定义抑制宏观的扩张在清单文件中。 的 .nolist 限定符直接参数的数量后,像这样:

% 1. nolist宏观foo

或者像这样:

1 - 5 + %宏观酒吧。 nolist a,b,c,d,e,f,g,h

**4.4。 有条件的组装**

类似于C预处理器,NASM允许组装部分的源文件只有在某些条件得到满足。 这个特性的一般语法看起来像这样:

%如果<条件>

;一些代码只出现> <条件是否满足

% elif < condition2 >

,只有出现如果不满足<条件> < condition2 >是

其他的%

;这似乎如果<条件>和< condition2 >了

% endif

的 其他的% 条款是可选的,是 % elif 条款。 你可以有多个 % elif 条款。

**4.1.1。 %如果定义了 :测试单行的宏观的存在**

开始的条件汇编块线 %如果定义了宏 将组装后续代码,如果且仅当一个单行的宏观叫什么 宏 定义。 如果不是,那么% elif 和 其他的% 块(如果有的话)会被处理。

例如,当调试一个程序,您可能想要编写代码等

,执行一些功能

% ifdef调试

writefile 2,“函数执行成功”,13日10

% endif

,去做别的事情

然后您可以使用命令行选项 - d调试 创建一个版本的程序产生调试信息,和删除选项来生成最终的发布版本的程序。

你可以测试一个宏 *不* 被定义为使用 %如果未定义 而不是 %如果定义了 。 你也可以测试宏定义 % elif 块用 % elifdef 和 % elifndef 。

**10/24/11。 % ifmacro :测试多行宏观的存在**

的 % ifmacro 指令操作一样 %如果定义了 指令,除了它检查的存在多行宏。

例如,你可能使用一个大型项目,没有控制宏在图书馆。 您可能想要创建一个宏,一个名字,如果它不存在,如果一个人与另一个名字,名字确实存在。

的 % ifmacro 被认为是真实的如果定义一个宏的名称和数量的参数会导致定义冲突。 例如:

% ifmacro MyMacro 1 - 3

%的错误“MyMacro 1 - 3”与现有的宏观引起冲突。

其他的%

%宏观MyMacro 1 - 3

;插入代码来定义宏

% endmacro

% endif

这将创建宏 MyMacro 1 - 3 如果没有宏已经存在的冲突,并发出警告是否有定义冲突。

你可以测试使用的宏不存在 % ifnmacro 而不是 % ifmacro 。 额外的测试可以执行 % elif 块用 % elifmacro 和% elifnmacro 。

**4.4.3。 % ifctx :测试上下文堆栈**

条件汇编构造 % ifctx ctxname 将导致后续代码组装当且仅当上下文预处理程序的上下文堆栈顶部有名字吗ctxname 。 与 %如果定义了 ,逆 % elif 形式 % ifnctx , % elifctx 和 % elifnctx 也支持。

上下文堆栈的更多细节,请参阅 [4.7节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。 样品使用 % ifctx ,请参阅 [4.7.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**4.4.4。 %如果 :测试任意数值表达式**

条件汇编构造 %如果expr 将导致后续代码组装当且仅当数值表达式的值 expr 是非零的。 使用此功能的一个例子是在决定何时突破 %代表 预处理程序循环:看 [4.5节](http://www.tortall.net/projects/yasm/manual/html/manual.html)给一个详细的例子。

表达给 %如果 ,其对应的 % elif ,是一个关键表达式(见 [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

%如果 扩展了正常NASM表达式语法,通过提供一组 关系运算符不通常可以在表达式。 运营商 = , < , > , <  =, > = 和 < > 测试平等、小于、大于、less-or-equal分别大于等于和不等于。 c形式 = = 和 !  = 作为替代形式的支持 = 和 < > 。 此外,低优先级的逻辑运算符 & & , ^ ^ 和 |  | 提供,供应 逻辑, 逻辑XOR andlogical或。  这些工作像C逻辑运算符(尽管C没有逻辑异或),他们总是返回0或1,和治疗任何非零输入1(这样 ^ ^ 例如,返回1如果一个输入为零,否则和0)。 关系运算符也为真,0为假返回1。

**4.4.5。 % ifidn 和 % ifidni :测试具体文本的身份**

的构造 % ifidn text1 text2 将导致后续代码组装当且仅当吗 text1 和 text2 扩大后,单行的宏,是相同的文本。 空白的差异并不算。

% ifidni 类似于 % ifidn ,但 不区分大小写的。

例如,下面的宏推一个注册或堆栈,并允许你来治疗 知识产权 作为一个真正的注册:

宏观pushparam % 1

% ifidni % 1,ip

调用% %标签

% %标签:

其他的%

推动% 1

% endif

% endmacro

像大多数其他 %如果 结构, % ifidn 是否有对应的 % elifidn 和否定形式 % ifnidn 和 % elifnidn 。 同样的, % ifidni 有同行 % elifidni , % ifnidni 和 % elifnidni 。

**4.4.6。 % ifid , % ifnum , % ifstr :测试令牌类型**

一些宏需要执行不同的任务取决于他们是否通过一个数字,一个字符串,或一个标识符。 例如,一个字符串输出宏可能希望能够应付传递一个字符串常量或现有的字符串的指针。

条件组合构建 % ifid ,以一个参数(可能是空白的),组装后续代码当且仅当第一个令牌参数存在,是一个标识符。% ifnum 同样的工作,但测试令牌是一个数字常数; % ifstr 测试是一个字符串。

例如, writefile 中定义的宏 [4.3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)可以扩展到利用吗 % ifstr 在以下方式:

%宏观writefile 2 - 3 +

% ifstr % 2

jmp % % endstr

% % 0 = 3

% % str:db % 2,% 3

其他的%

% % str:db % 2

% endif

% % % % endstr:mov dx,str

mov残雪,% % endstr - % % str

其他的%

mov dx,% 2

mov残雪,% 3

% endif

mov bx,% 1

mov啊,0 x40

int 0 x21

% endmacro

然后 writefile 宏可以应对被称为在以下两个方面:

strpointer writefile(文件),长度

writefile(文件),“你好”,13日10

首先, strpointer 作为一个已经声明字符串的地址,然后呢 长度 作为它的长度;第二,给出一个字符串宏,因此说它本身和工作地址和长度。

注意使用 %如果 在 % ifstr :这是检测宏是否传递两个参数(绳子将一个字符串常量,和 db % 2 将是足够的)或更多(在这种情况下,但前两集中在一起 % 3 , db % 2,% 3 需要)。

通常的 % elifXXX , % ifnXXX 和 % elifnXXX 为每个版本存在 % ifid , % ifnum 和 % ifstr 。

**4.4.7。 %的错误 :用户定义的错误报告**

预处理器指令 %的错误 将导致NASM报告一个错误如果它发生在组装代码。 如果其他用户是要组装你的源文件,您可以确保他们正确的宏定义的代码是这样的:

% ifdef SOME_MACRO

,做一些设置

% elifdef SOME_OTHER_MACRO

,做一些不同的设置

其他的%

% SOME_MACRO和SOME_OTHER_MACRO定义错误。

% endif

那么任何用户无法理解你的代码应该是组装将迅速警告他们的错误,而不必等到正在运行的程序崩溃,然后不知道出现了什么问题。

**4.5。 预处理程序循环**

NASM的 次 前缀,虽然有用,不能用于调用一个多行宏观多次,因为它是由NASM处理后宏已经扩大。 因此NASM提供了另一种形式的循环,这次在预处理器级别: %代表 。

这些指令 %代表 和 % endrep ( %代表 以一个数值参数,这可以是一个表达式; % endrep 不需要参数)可以用来封装一大块代码,然后复制多次指定的预处理:

%分配我0

%代表64

公司词表+ 2 *(我)

%分配我+ 1

% endrep

这将生成一个64年的序列 公司 指令,递增的记忆的每一个字 (表) 来 (表+ 126) 。

对于更复杂的终止条件,或打破一个重复循环的一部分,您可以使用 % exitrep 指令终止循环,像这样:

斐波那契数列:

%分配我0

%分配j 1

%代表100

%如果j > 65535

% exitrep

% endif

dw j

%分配j k + i

%分配我j

%分配j k

% endrep

fib_number装备(斐波那契美元)/ 2

这产生一个列表的所有斐波纳契数将在16位配合。 注意,仍然必须给予最大重复计数 %代表 。 这是为了防止NASM进入一个无限循环的可能性在预处理器,它(多任务和多用户系统)通常导致逐渐用尽所有的系统内存和其他应用程序开始崩溃。

**4.6。 包括其他文件**

再次使用,一个非常相似的语法C预处理器,NASM预处理器可以包括其他源文件到您的代码。 这是通过使用 %包括 指令:

%包括“macros.mac”

包括文件的内容吗 macros.mac 源文件包含 %包括 指令。

包含文件首先寻找相对于包含源文件的目录执行包容,然后相对于任何目录Yasm命令行上指定使用 -我 选项(见 [1.3.3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),在命令行上给出的顺序(Yasm命令行上的任何相对路径是相对于当前工作目录,如Yasm从何而运行)。 虽然这搜索策略不匹配传统NASM行为,它匹配的行为大多数C编译器和更好的处理相对路径名。

标准C语言为防止文件被包含不止一次同样适用于NASM预处理:如果文件 macros.mac 的形式

%如果未定义MACROS_MAC

%定义MACROS_MAC

,现在定义一些宏

% endif

然后包括文件不止一次不会导致错误,因为第二次文件包含什么都不会发生,因为宏 MACROS_MAC 将已经被定义。

你可以强迫一个文件包括即使没有 %包括 显式地包含指令,通过使用 - p 在Yasm命令行选项(请参见 [1.3.3.4节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。

**4.7。 上下文堆栈**

有标签,是当地一个宏定义有时还不够强大的:有时你希望能够分享几个宏调用之间的标签。 一个可能是一个例子 重复 … 直到 循环,扩张重复 宏将需要能够参考标签的 直到 宏定义。 然而,对于这样一个宏观你也希望能够嵌入这些循环。

NASM预处理器提供这种级别的权力的 *上下文堆栈* 。 预处理器维护一堆 *上下文* ,每一个的特点是一个名字。 你添加一个新的上下文堆栈使用 %推 指令,移除一个使用 %的流行 。 您可以定义标签,是当地的一个特定的上下文堆栈。

**4.7.1。 %推 和 %的流行 :创建和删除背景**

的 %推 指令是用来创建一个新的上下文并将其上下文堆栈的顶部。 %推 需要一个参数,即上下文的名称。 例如:

% foobar推

这将一个新的上下文 foobar 在堆栈上。 你可以有多个具有相同名称的上下文堆栈:还可以对它们进行区分。

该指令 %的流行 ,不需要参数,消除了上下文从上下文堆栈和破坏,以及任何与之相关的标签。

**4.7.2。 Context-Local标签**

就像使用 % % foo 定义一个标签是本地使用的特定的宏调用,使用 % $ foo 用于定义一个标签是本地的环境上下文堆栈的顶部。 因此, 重复 和 直到 上面的例子可以实现通过:

%宏观重复0

%将重复

% $开始:

% endmacro

%的宏,直到1

j % -1%美元开始

%的流行

% endmacro

例如,通过调用

mov残雪,字符串

重复

添加残雪,3

scasb

直到e

这将扫描每四字节的字节的字符串搜索 艾尔 。

如果您需要定义,或访问,标签本地上下文 *下面* 堆栈的顶部,你可以使用 % $ $ foo ,或 % $ $ $ foo 下面的内容,等等。

**4.7.3。 Context-Local单行的宏**

NASM预处理程序还允许您定义单行的宏是当地的一个特定的上下文,在同样的方式:

%定义% localmac 3美元

单行的宏定义吗 % $ localmac 是本地上下文堆栈顶部。 当然,在随后的 %推 ,它仍然可以被访问的名字 % $ $ localmac 。

**4.7.4。 % repl :重命名上下文**

如果你需要改变栈顶上下文的名称(例如,在秩序,它有不同的反应 % ifctx ),您可以执行 %的流行 紧随其后的是一个 %推 ;但这将摧毁所有的副作用context-local与上下文相关的标签和宏只是破灭。

NASM预处理器提供了指导 % repl ,这 *替换* 上下文与一个不同的名称,没有接触相关的宏和标签。 所以你可以代替破坏性的代码

%的流行

%推新名称

非破坏性的版本 % repl新名称 。

**4.7.5。 IFs示例使用上下文堆栈:块**

这个例子使用了几乎所有的上下文堆栈功能,包括条件汇编构造 % ifctx ,实现一个IF语句块为一组宏。

%宏如果1

%如果推

j % -1% ifnot美元

% endmacro

%其他宏观0

% ifctx如果

% repl其他

jmp % $ ifend

% $ ifnot:

其他的%

%错误”预计‘如果’‘其他’”

% endif

% endmacro

%宏观endif 0

% ifctx如果

% $ ifnot:

%的流行

% elifctx其他

% $ ifend:

%的流行

其他的%

%错误”之前预期的“如果”或“其他”endif”

% endif

% endmacro

这个代码更健壮 重复 和 直到 宏中给出 [4.7.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html),因为它使用条件装配检查宏发行以正确的顺序(例如,不打电话 endif 之前 如果 )和问题 %的错误 如果他们不是。

此外, endif 宏必须能够应对直接后的两种不同的情况下 如果 ,或者后一个 其他的 。 它实现,利用装配条件做不同的事情取决于上下文堆栈的顶部 如果 或 其他的 。

的 其他的 宏保存上下文堆栈,以有 % $ ifnot 指的 如果 宏定义的是一样的 endif 宏,但必须改变上下文的名称 endif 就会知道有一个干预 其他的 。 它通过使用 % repl 。

一个示例使用这些宏的样子:

cmp ax,软

如果ae

cmp bx,残雪

如果ae

mov ax,残雪

其他的

mov ax,软

endif

其他的

cmp ax,残雪

如果ae

mov ax,残雪

endif

endif

块- 如果 宏处理嵌套相当令人高兴的是,通过推动另一个上下文中,描述内 如果 描述外,最重要的 如果 ,因此其他的 和 endif 总是把最后一个无与伦比的 如果 或 其他的 。

**4.8。 标准宏**

Yasm定义了一组标准宏在NASM预处理器定义已经开始处理任何源文件。 如果你真的需要一个程序组装没有预定义的宏,您可以使用 %明显 空一切的预处理器指令。

大多数用户级NASM语法指令(见 [第五章](http://www.tortall.net/projects/yasm/manual/html/manual.html))实现为宏调用原语指示;这些是中描述 [第五章](http://www.tortall.net/projects/yasm/manual/html/manual.html)。 其余的标准宏观描述。

**4.8.1。 __YASM_MAJOR__ 等:Yasm版本**

单行的宏 __YASM_MAJOR__ , __YASM_MINOR__ , __YASM_SUBMINOR__ 扩大的主要、次要、子副的部分 版本号的Yasm被使用。  此外, __YASM_VER__ 扩大到Yasm版本的字符串表示 __YASM_VERSION_ID__ 扩大到一个32位的BCD-encoded表示Yasm版本,主要版本的最重要的8位,紧随其后的是8位小版本和8位子副版本,并在最重要的8位0。 例如,在Yasm  0.5.1, __YASM_MAJOR__ 将定义为0, __YASM_MINOR__ 将被定义为5, __YASM_SUBMINOR__ 将被定义为1,__YASM_VER__ 将被定义为 “0.5.1” , __YASM_VERSION_ID__ 将被定义为 000050100 h 。

此外,一行宏 __YASM_BUILD__ 扩大Yasm”建立“数字,通常Subversion变更集的数字。 它应被视为显著低于子副版本,并且通常只用于区分Yasm夜间快照或预发布(如发行候选版)Yasm版本。

**4.8.2。 __FILE__ 和 __LINE__ :文件名和行号**

像C预处理器,NASM预处理程序允许用户找到包含当前指令的文件名和行号。 宏 __FILE__ 扩大到一个字符串常量给当前的输入文件的名称(可能在装配的过程中如果发生改变 %包括 使用指令), __LINE__ 扩展一个数值常数输入文件中的当前行号。

例如,可以用这些宏调试信息沟通的一个宏,因为调用 __LINE__ 在一个宏定义(单行或多行)将返回的行号宏 *调用* ,而不是 *定义* 。 为了确定在一段代码崩溃发生,例如,可以编写一个程序 stillhere ,这是通过一个行号EAX 和输出类似“155行:还在这里”。 您可以编写一个宏

%宏观notdeadyet 0

推动eax

mov eax,__LINE__

叫stillhere

流行eax

% endmacro

然后胡椒与调用您的代码 notdeadyet 直到你找到崩溃点。

**4.8.3。 __YASM_OBJFMT__ 和 __OUTPUT_FORMAT__ 关键字:输出对象格式**

__YASM_OBJFMT__ ,其NASM-compatible别名 __OUTPUT_FORMAT__ ,扩大到对象的格式 *关键字* 在命令行上指定 - f *关键字* (见 [1.3.1.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 例如,如果 **yasm** 是调用 - f精灵 , __YASM_OBJFMT__ 扩大到 精灵 。

这些扩展匹配给定的选项在命令行上,即使对象格式是等价的。 例如, - f精灵 和 - f  elf32 是等价的32位精灵格式说明符,然后呢 精灵- f - m amd64 和 - f  elf64 是等价的64位精灵格式说明符,但 __YASM_OBJFMT__ 将扩大 精灵 和 elf32 第一两种情况 精灵 和 elf64 第二两种情况。

**4.8.4。 STRUC 和 ENDSTRUC :声明结构数据类型**

NASM预处理器足够强大,可以实现数据结构的宏。 的宏 STRUC 和 ENDSTRUC 用于定义一个结构数据类型。

STRUC 需要一个参数,这是数据类型的名称。  这个名字被定义为一个符号的值为零,同样也有后缀 _size 附加到它,然后定义为 装备的 给结构的大小。  一次 STRUC 已发布,你定义的结构,而且应该定义字段使用 RESB 家庭的伪指令,然后调用 ENDSTRUC 完成的定义。

例如,定义一个结构 mytype 包含longword,一句话,一个字节字符串的字节,你可能的代码

struc mytype

mt_long:resd 1

mt_word:resw 1

mt_byte:resb 1

mt_str:resb 32

endstruc

上述代码定义了六个符号: mt_long 0(偏移量从一开始的 mytype 结构longword字段), mt_word 4, mt_byte 6,mt_str 7, mytype_size 39岁, mytype 自己是零。

结构类型名称的原因是定义为零的副作用使结构与当地的标签机制:如果你的结构成员倾向于在一个以上的结构相同的名称,您可以定义上面的结构是这样的:

struc mytype

。 长:resd 1

。 词:resw 1

。 字节:resb 1

。 str:resb 32

endstruc

这定义了补偿结构领域 mytype.long , mytype.word , mytype.byte 和 mytype.str 。

因为没有NASM语法 *内在* 结构的支持,不支持任何形式的时期符号来引用一个结构的元素一旦你有一个(除了上述local-label符号),所以代码等 mov ax,[mystruc.mt_word] 不是有效的。 mt_word 是一个常数就像任何其他常数,所以正确的语法吗 mov  ax,[mystruc + mt_word] 或 mov ax,[mystruc + mytype.word] 。

**4.8.5。 ISTRUC , 在 和 IEND :声明实例的结构**

在定义一个结构类型,接下来你通常要做的是要申报的实例数据段的结构。 NASM预处理器提供了一种简便的方法 ISTRUC 机制。 声明一个结构类型 mytype 在项目中,您的代码是这样的:

mystruc:istruc mytype

123456年在mt_long,dd

在mt_word dw 1024

在db mt_byte“x”

在mt_str,db“hello,world”,13日10 0

iend

的功能 在 宏的使用 次 前缀进装配位置的正确点指定结构字段,然后宣布指定的数据。 因此结构字段必须声明中指定的顺序在同一结构的定义。

如果数据在结构领域需要不止一个源代码行来指定,剩下的源代码行后可以很容易地来 在 线。 例如:

123134145156167178189年在mt_str,db

190100 db,0

根据个人口味不同,你也可以省略的部分的代码 在 结构领域完全一致,并开始下一行:

在mt_str

db“hello,world”

10 db 13日,0

**4.8.6。 对齐 和 ALIGNB :数据对齐**

的 对齐 和 ALIGNB 宏提供一种方便的方式来调整代码或数据在一个词,longword、段落或其他边界。 的语法 对齐 和 ALIGNB 宏是

4,对齐4字节边界对齐

对齐16;16字节边界对齐

nop对齐16日,相当于前一行

对齐8,db 0;垫0而不是空操作

4,对齐resb 1;BSS对齐到4

alignb 4,相当于前一行

宏都需要他们的第一个参数是2的幂,它们都计算出所需要的额外字节数使当前节的长度的倍数,两个孩子的权力,和输出NOP填补或应用 次 第二个参数来执行对齐前缀。

如果没有指定第二个参数,默认 对齐 是 NOP 和默认值 ALIGNB 是 RESB  1 。 对齐 对待一个 NOP 论点特别通过生成最大NOP填补指令(不一定NOP操作码)电流 位 设置,而 ALIGNB 的第二个参数。  否则,这两个宏指定第二个参数时是等价的。  通常情况下,你可以使用 对齐 在代码和数据部分 ALIGNB 在BSS部分,不需要第二个参数除了特殊用途。

对齐 和 ALIGNB ,简单的宏,执行没有错误检查:他们不能提醒你如果第一个参数不能是2的幂,或者如果他们的第二个参数生成多个字节的代码。 在每一种情况下,他们会静静地做错误的事情。

ALIGNB (或 对齐 的第二个参数 RESB 1 )可以使用在结构定义:

struc mytype2

mt_byte:resb 1

alignb 2

mt_word:resw 1

alignb 4

mt_long:resd 1

mt_str:resb 32

endstruc

这将确保成员结构合理对齐相对于结构的基础。

最后一个警告: ALIGNB 相对于的开始工作 *部分* 地址空间的开始,而不是最终的可执行文件。 调整到16位边界时,部分只能保证你在4字节边界对齐的,例如,是浪费精力。 再次,Yasm不检查部分的排列特征是明智的使用ALIGNB 。 对齐 更聪明, *做* 调整部分对齐最大指定对齐。

**第五章。 NASM汇编指令**

**表的内容**

[5.1。 指定目标处理器模式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.1.1。 位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.1.2中。 USE16 , USE32 , USE64](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.2。 默认的 :改变汇编的违约](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.3。 改变和定义部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.3.1。 部分 和 段](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.3.2。 标准化的部分名称](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.3.3。 的 __SECT__ 宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.4。 绝对 :定义绝对标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.5。 走读生 :导入符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.6。 全球 :输出符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.7。 常见的 :定义常见的数据区域](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[5.8。 CPU CPU:定义依赖关系](http://www.tortall.net/projects/yasm/manual/html/manual.html)

NASM,尽管它试图避免汇编类似MASM和TASM实现的官僚主义仍然是被迫支持 *几* 指令。 这些都是本章中描述。

NASM的指令有两种类型: *用户级* 指令和 *原始的* 指令。 通常,每个指令都有一个用户级表单和一个基本形式。 在几乎所有情况下,我们建议用户使用用户级形式的指令,这被实现为宏调用原始的形式。

原始的指令包含在方括号;用户级指令。

除了在本章中描述的通用指令,每个对象文件格式可以提供额外的指令,以控制特定文件格式的功能。 这些 *特定于格式* 指令记录以及实现它们的格式, [第四部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**5.1。 指定目标处理器模式**

**5.1.1。 位**

的 位 指令指定Yasm是否应该用于生成代码运行在一个处理器操作在16位模式中,32位模式还是64位模式。 语法16位 , 32位 ,或 位64 。

在大多数情况下,你不应该需要使用 位 明确。  的 coff , elf32 , macho32 , win32 对象格式,它被设计用于32位操作系统,所有导致Yasm默认选择32位模式。  的 elf64 , macho64 , win64 对象格式,设计用于在64位操作系统,都导致Yasm默认选择64位模式。  的 xdf 你对象格式允许您指定每一段定义 USE16 , USE32 ,或 USE64 相应Yasm将其操作模式,所以使用 位 指令再次不必要的。

最可能使用的原因 位 指令是编写32位或64位代码在一个平坦的二进制文件;这是因为 本 对象格式默认为16位模式预期它最常用于写DOS com 项目,DOS 。系统 设备驱动程序和引导加载程序软件。

你做 *不* 需要指定 32位 仅仅为了使用32位指令在一个16位DOS程序;如果你这样做,汇编程序生成正确的代码,因为它将会编写代码针对32位平台上运行一个16位。 然而,它 *是* 需要指定 位64 使用64位指令和寄存器;这样做是为了让这些指令的使用和在32位或16位程序注册名字,尽管这样的使用将会产生一个警告。

当Yasm 16位 模式,说明哪些使用32位数据前缀0 x66字节,这些指的32位地址0 x67前缀。 在 32位 模式,反过来也是如此:32位指令不需要前缀,而指令使用16位数据需要一个0 x66这些工作需要一个0 x67 16位地址。

当Yasm 位64 模式中,32位指令通常不需要前缀,大多数使用64位的寄存器或数据大小需要雷克斯前缀。  在必要时Yasm自动插入雷克斯前缀。 还有8更普遍和SSE寄存器,并不再支持16位寻址。 默认的地址大小是64位,32位寻址可以选择0  x67前缀。 默认操作数的大小仍然是32位,然而,0 x66前缀选择16位操作数的大小。  雷克斯前缀可以用来选择64位操作数的大小,并访问新的寄存器。 几条指令有一个默认的64位操作数的大小。

雷克斯使用前缀时,处理器不知道如何解决 啊 , 黑洞 , CH 或 DH (高8位遗留)寄存器。 相反,它可以访问的低8位 SP , 英国石油公司 如果 , 迪 注册为 SPL , 底保 , 银 , 迪勒 分别,但只有当雷克斯使用前缀。

的 位 指令有一个完全等价的原始形式, (16位) , (32位) , (64位) 。 用户级以外的形式是一个没有宏观功能调用原始的形式。

**5.1.2中。 USE16 , USE32 , USE64**

的 USE16 , USE32 , USE64 指令可以代替使用 16位 , 32位 , 位64 分别为兼容其他装配工。

**5.2。 默认的 :改变汇编的违约**

的 默认的 指令改变了汇编程序默认值。 通常,Yasm默认模式,预计程序员显式地指定最直接的特性。 然而,有时这是不可取的,如果某种行为非常commmonly使用。

目前,唯一的 默认的 可设置的是registerless是否有效的地址在64位模式 把 相对。 默认情况下,他们是绝对的,除非覆盖 REL 说明符(见 [3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 然而,如果 默认REL 是指定的, REL 是默认的,除非覆盖的 腹肌 说明符, FS或 GS 使用部分覆盖,或另一个注册是有效地址的一部分。

特殊处理的 FS 和 GS 覆盖是由于这样的事实,这些部分是唯一的部分可以有非0的基地地址在64位模式下,因此通常用作线程指针或其他特殊功能。 与一个非零基础地址,生成 把 相对地址这些形式将极为混乱。  其他段寄存器等 DS 总是有基地址0,所以RIP-relative访问仍然是有意义的。

默认REL 被禁用, 默认的腹肌 。 汇编程序的默认模式启动 默认的腹肌 。

**5.3。 改变和定义部分**

**5.3.1。 部分 和 段**

的 部分 指令((( 段 )是一个完全等价的同义词)变化的输出文件您编写的代码将被组装成。  在某些对象文件格式,部分的数量和名称是固定的;另一方面,用户可能多达他们希望。  因此 部分 有时可能会给一个错误消息,也可能定义一个新的部分,如果你想切换到(还)不存在的部分。

**5.3.2。 标准化的部分名称**

Unix对象格式,和 本 对象格式,所有的支持 标准化的部分名称 。text , .  data 和 .bss 代码、数据和uninitialised-data部分。  的 obj 相比之下,格式不承认这些部分名称是特殊的,甚至会剥的主要时期的任何部分的名字有一个。

**5.3.3。 __SECT__ 宏**

的 部分 指令的不寻常之处在于,它的用户级函数形式不同于原始的形式。 原始的形式, (部分xyz) ,只需切换当前目标部分的。  用户级形式, 部分xyz 然而,首先定义了单行的宏 __SECT__ 是原始的 (部分) 指令它问题,然后问题。 因此,用户级指令

部分。

扩大到两行

%定义__SECT__(部分。text)

(部分。text)

用户可能会发现它有用的利用自己的宏。 例如, writefile NASM手册中定义的宏可以有效地重写在接下来的更复杂的形式:

%宏writefile 2 +

(. data部分)

% % str:db % 2

% % endstr:

__SECT__

mov dx,% % str

mov残雪,% % endstr - % % str

mov bx,% 1

mov啊,0 x40

int 0 x21

% endmacro

这种形式的宏,一旦一个字符串传递给输出,第一开关临时文件的数据部分,使用的原始形式 部分 指令,以免修改 __SECT__ 。 然后它声明其在数据部分的字符串,然后调用 __SECT__ 切换回 *无论* 部分用户以前的工作。 因此避免了需要在之前的版本的宏,包括 无条件转移指令 跳过数据指令,也不失败,如果在一个复杂 OBJ 格式模块,用户可能组装几个独立的代码的代码的任何部分。

**5.4。 绝对 :定义绝对标签**

的 绝对 指令可以被认为是另一种形式 部分 :它导致后续代码是针对没有物理部分,但在假想的部分从给定的绝对地址。 唯一可以使用在这种模式下的指令 RESB 家庭。

绝对 使用如下:

绝对0 x1a

kbuf_chr resw 1

kbuf_free resw 1

kbuf resw 16

这个例子描述了一个部分的PC BIOS数据区域,在段地址0 x40:上面的代码定义了 kbuf_chr 0 x1a, kbuf_free 0 x1c, kbuf 0 x1e。

用户级的形式 绝对 ,像这样的 部分 ,重新定义 __SECT__ 宏调用时。

STRUC 和 ENDSTRUC 被定义为宏使用哪一个 绝对 (还有 __SECT__ )。

绝对 没有绝对的常数作为参数:(实际上,它可以采取一个表达式 关键表达式:看 [3.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),它可以是一个值在一个段。 例如,TSR可以重用的设置代码运行时BSS是这样的:

org 100 h;它是一个。 COM程序

jmp设置,设置代码

,居民TSR是这里的一部分

设置:,现在写的代码安装临时避难所

绝对的设置

runtimevar1 resw 1

runtimevar2 resd 20

tsr_end:

这定义了一些变量“之上”的设置代码,这样设置后运行结束之后,它拿起可以重用为数据存储空间的临时避难所。 象征“tsr_end”可以用来计算的总大小的一部分需要居民的临时避难所。

**5.5。 走读生 :导入符号**

走读生 类似MASM指令实现吗 条EXTRN 和C字 走读生 :它是用于声明一个符号未定义任何模块组装,但被认为是定义在其他模块和需要被这一个。 并不是每一个对象文件格式可以支持外部变量: 本 格式不能。

的 走读生 你喜欢的指令需要尽可能多的参数。 每一个参数是一个符号的名称:

走读生_printf

走读生_sscanf,_fscanf

一些对象文件格式提供额外的功能 走读生 指令。 在所有情况下,使用额外的特性通过向一个冒号的符号名的对象格式特定的文本。 例如, obj 格式允许您声明默认段基地外部应该 dgroup 通过指令

走读生_variable:关于dgroup

的原始形式 走读生 不同于用户级形式只有在它一次只能带一个参数:支持多种参数预处理器级别的实现。

你可以声明变量一样 走读生 不止一次:NASM会悄悄地忽略第二,后来redeclarations。 你不可以声明一个变量走读生 不过,还有别的东西。

**5.6。 全球 :输出符号**

全球 的另一端吗 走读生 :如果一个模块声明作为一种象征 走读生 然后是指,为了防止链接错误,其他模块必须*定义* 象征和声明它 全球 。 一些汇编程序使用这个名字 公共 为这个目的。

的 全球 指令申请必须出现一个标志 *之前* 符号的定义。

全球 使用相同的语法 走读生 ,除了必须引用符号 *是* 在同一模块中定义 全球 指令。 例如:

全球_main

_main:;一些代码

全球 ,就像 走读生 ,允许对象格式定义私有扩展通过冒号。 的 精灵 对象格式,例如,允许您指定全球数据项是否函数或数据:

全球hashlookup:函数,散列表:数据

就像 走读生 的原始形式 全球 不同于用户级形式只有在它一次只能带一个参数。

**5.7。 常见的 :定义常见的数据区域**

的 常见的 指令用于申报的东西 *常见的变量* 。 常见的变量就像uninitialised数据部分中声明一个全局变量,所以

常见intvar 4

相似的函数吗

全球intvar

部分.bss

intvar resd 1

所不同的是,如果一个以上的模块定义了共同变量,然后在链接时这些变量 *合并后的* 和引用, intvar 在所有模块都指向同一块内存。

就像 全球 和 走读生 , 常见的 支持对象格式特定的扩展。 例如, obj 格式允许无论远近,常见的变量和 精灵格式允许您指定的对齐要求一个共同的变量:

常见commvar 4:附近;在OBJ工作

常见intarray 100:4;在精灵:4字节对齐

再次,像 走读生 和 全球 的原始形式 常见的 不同于用户级形式只有在它一次只能带一个参数。

**5.8。 CPU CPU:定义依赖关系**

的 CPU 指令限制汇编的指令可以在指定的CPU。 看到 [第六部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)为 CPU 选择不同的架构。

所有选择都不区分大小写。 说明将使只有适用于选定的cpu或更低。

**第三部分。 气体的语法**

本书的章节在这部分文档GNU兼容语法接受Yasm“气”解析器。

**表的内容**

[6。 TBD](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**第六章。TBD**

写的。

**第四部分。 对象的格式**

书的这一部分文档中的章节Yasm对各种对象文件格式的支持。

**第七章。 本 :平面形式的二进制输出**

的 本 “对象格式”并不产生对象文件:产生的输出文件只包含部分数据;不生成标题或搬迁。  输出可以被认为是“纯二进制”,用于操作系统和引导加载程序开发、生成ms -  dos com 可执行程序和 。系统 设备驱动程序,为嵌入式目标环境(如创建图像。 快闪记忆体)。

的 本 对象格式支持无限数量的指定部分。 看到 [7.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)获取详细信息。 唯一限制这些部分是输出文件的存储位置不能重叠。

当使用x86体系结构, 本 对象格式开始Yasm 16位模式。 为了编写本地32位或64位的代码,一个显式的 32位 或位64 指令分别是必需的。

本 产生一个输出文件没有扩展默认情况下,它只是条扩展从输入文件名称。 因此输入文件的默认输出文件名foo.asm 仅仅是 喷火 。

**7.1。 ORG 起源:二进制**

本 提供了 ORG 指令在NASM中语法允许设置输出文件的内存地址是最初加载。 的 ORG 指令只能使用一次(输出文件只能最初加载到一个位置)。 如果 ORG 没有指定, ORG 0 在默认情况下使用。

这使得NASM-syntax的操作 ORG 完全不同的操作 ORG 在其他装配工,通常只是将装配位置为给定的值。 本 提供了一个更强大的替代形式的扩展 部分 指令;参见 [7.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)获取详细信息。

当结合多个部分, ORG 也有违约的影响LMA的第一部分 ORG 价值的输出文件尽可能小。 如果这不是所需的行为,通过显式地指定一个LMA部分 开始 或 遵循 限定符的 部分 指令。

**7.2。 本 扩展 部分 指令**

的 本 对象格式允许使用任意名称的多个部分。 它还扩展了 部分 (或 段 )指令允许复杂的排序段的输出文件或初始加载地址(也称为LMA)和最终的执行地址(虚拟地址或VMA)。

的 的影响是执行地址。 Yasm计算绝对内存引用假设在一个部分程序代码在VMA而被执行。 另一方面,LMA指定一段在哪里 *最初* 加载,以及它的位置在输出文件中。

通常,VMA LMA一样。 然而,他们可能会有所不同,如果程序或另一段代码副本(凡)执行前一节。 在嵌入式系统中一个典型的例子是一段代码存储在ROM中,但快复制到RAM之前执行。 另一个例子是覆盖:部分加载到不同的文件位置的需求相同的执行位置。

的 本 扩展 部分 指令允许灵活的VMA和LMA的规范,包括一致性约束。 与其他对象格式,可以添加额外的属性后,部分名称。 可用的属性中列出 [表7.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**表7.1。 本** **部分属性**

| **属性**             | **显示部分**                                                 |
| -------------------- | ------------------------------------------------------------ |
| progbits             | 存储在磁盘映像,而在负荷分配和初始化。                        |
| nobits               | 分配和初始化加载(progbits相反)。 指定的progbits或nobits可能只有一个,他们是相互排斥的属性。 |
| 开始= *地址*         | LMA开始 *地址* 。 如果LMA对齐约束,它是针对所提供的地址和检查如果发出的一个警告 *地址* 不符合一致性约束。 |
| 遵循= *sectname*     | 应该遵循的部分命名 *sectname* 在输出文件中(LMA)。 如果LMA一致性约束,它是受人尊敬和插入一个缺口,部分满足一致性要求。 注意,LMA重叠是不允许的,通常只有一个部分可能遵循另一个。 |
| 对齐= *n*            | 需要一个LMA对齐的 *n* 字节。 的值 *n* 必须是2的幂。 如果未指定LMA对齐默认为4。 |
| 音速启动= *地址*     | 有一个VMA开始吗 *地址* 。 如果VMA一致性约束,它是针对所提供的地址和检查如果发出的一个警告 *地址* 不符合一致性约束。 |
| vfollows =*sectname* | 应该遵循的部分命名 *sectname* 在输出文件中(VMA)。 如果VMA一致性约束,这是尊重和插入一个缺口,部分满足一致性要求。 VMA允许重叠,所以多个部分可能遵循另一个(可能是有用的在覆盖的情况下)。 |
| valign = *n*         | 需要一个VMA对齐的 *n* 字节。 的值 *n* 必须是2的幂。 影响规律排列LMA对齐,如果未指定默认值。 |

只有一个 开始 或 遵循 可能为一段指定;同样的限制适用于 音速启动 和 vfollows 。

通过使用除非另有说明 遵循 或 开始 ,默认Yasm假定的隐式排序的顺序输入文件的各个部分中。 一段名为 。text 总是第一个部分。  任何代码之前,是显式的 部分 指令进入 。text 部分。 的 。text 部分属性可能会被给予一个明确的 部分。 指令与属性。

除非另有说明,Yasm默认设置VMA LMA =。 如果指定只是“valign,Yasm需要LMA和对齐到所需的对齐。 这可能推动下面的影响“VMAs non-LMA地址,以避免VMA重叠。

Yasm对待 nobits 部分在一个特殊的方式以减少输出文件的大小。 作为 nobits 部分可以0-sized  LMA领域,但不能如果位于两个其他部分(由于VMA =  LMA默认),Yasm移动 nobits 部分与未指明的LMA输出文件,在那里他们可以保存有0 LMA大小,因此在输出文件中不占用任何空间。  如果这种行为不需要, nobits 部分LMA(就像一个progbits 部分)可以指定使用 遵循 或 开始 部分属性。

**7.3。 本 特殊的符号**

为了方便编写代码,自己从一个位置复制到另一个(例如从LMA到VMA执行期间), 本 对象格式提供了一些特殊的符号定义的每一个部分。 每个特殊符号开始 部分。 其次是部分名称。 支持的特殊 本 符号:

部分。 *sectname* .start

设置为命名的LMA地址部分 *sectname* 。

部分。 *sectname* .vstart

设置为VMA地址的部分命名 *sectname* 。

部分。 *sectname* 长处

将部分的长度命名 *sectname* 。 长度被认为是运行时,因此“nobits部分”长度是他们的运行时,而不是0。

**7.4。 映射文件**

可能生成地图文件 本 通过使用 (地图) 指令。 地图文件名可以指定命令行选项( ——mapfile = *文件名* )或 (地图)指令。 如果要求但没有地图输出文件名,地图默认输出到标准输出。

如果没有 (地图) 指令中给出了输入文件,没有地图生成输出。 如果 (地图) 没有选择,给出一个简短的地图生成。 的 (地图) 指令接收以下选项来控制映射文件中包含什么内容。 可以指定多个选项。 以外的任何选项,下面是解释为输出文件名。

短暂的

包括输入和输出文件名、来源( ORG 值),做一个简要的介绍总结清单VMA和LMA启动和停止地址和每个部分的部分长度。

部分 , 段

包括一个章节的详细清单,包括VMA和LMA对齐,任何“跟随”设置,以及VMA和LMA的部分开始地址和长度。

符号

包括一个详细清单的所有装备的价值观和VMA和LMA符号位置,按截面分组。

所有

所有的上面。

**第八章。 coff :通用对象文件格式**

**第9章。 elf32 :可执行文件和可链接32位对象文件格式**

**表的内容**

[9.1。 调试格式支持](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.2。 精灵的部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.3。 精灵的指示](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[设备上装。 鉴别 :添加文件识别](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.3.2。 大小 :设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.3.3。 类型 :设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.3.4。 弱 :创建软弱的象征](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.4。 精灵的扩展 全球 指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.5。 精灵的扩展 常见的 指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[9.6。 elf32 特殊符号和 关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

可执行和可链接对象格式是许多操作系统包括的主要对象格式 FreeBSD和GNU / Linux。 它出现在三种形式:

- 共享对象文件(所以)
- 浮动对象文件(. o)
- 可执行文件(没有约定)

Yasm只有直接支持浮动对象文件。 其他工具,如GNU链接器 **ld** ,帮助把浮动对象文件转化为另一种格式。 Yasm支持32位和64位精灵的生成文件,调用 elf32 和 elf64 。 一个额外的格式,称为 elfx32 ,是一个32位精灵文件,支持64位执行(指令和寄存器),而限制指针大小为32位。

Yasm默认为 32位 当输出模式 elf32 对象的格式。

**9.1。 调试格式支持**

精灵支持两种调试格式: 刺穿了 (见 [第20章](http://www.tortall.net/projects/yasm/manual/html/manual.html)), dwarf2 (见 [第十九章](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 不同的调试器理解这些不同的格式,更新的调试格式 dwarf2 ,所以先试试。

**9.2。 精灵的部分**

精灵的section-based输出支持属性在每组的基础上。  这些属性包括 alloc , 执行 , 写 , progbits , 对齐 。  除了对齐,每个能否定在NASM中语法通过将“不”,例如,“noexec”。 属性是后来读由操作系统为每个部分中,选择合适的行为中所示的含义 [表9.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**表9.1。 精灵部分属性**

| **属性**  | **显示部分**                                                 |
| --------- | ------------------------------------------------------------ |
| alloc     | 在运行时被加载到内存中。 这是对代码和数据部分,假为元数据部分。 |
| 执行      | 已经许可运行可执行代码。                                     |
| 写        | 在运行时是可写的。                                           |
| progbits  | 存储在磁盘映像,而在负荷分配和初始化。                        |
| 对齐= *n* | 需要一个内存对齐的 *n* 字节。 的值 *n* 必须是2的幂。         |

在NASM中语法、属性 nobits 提供一个别名 noprogbits 。

标准的主要部分属性默认值根据其预期使用,和任何未知的部分都有自己的默认值,如图所示 [表9.2](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**表9.2。 精灵标准部分**

| **部分** | **alloc** | **执行** | **写** | **progbits** | **对齐** |
| -------- | --------- | -------- | ------ | ------------ | -------- |
| .bss     | alloc     |          | 写     |              | 4        |
| . data   | alloc     |          | 写     | progbits     | 4        |
| .rodata  | alloc     |          |        | progbits     | 4        |
| 。text   | alloc     | 执行     |        | progbits     | 16       |
| .comment |           |          |        | progbits     | 0        |
| 未知的   | alloc     |          |        | progbits     | 1        |

**9.3。 精灵的指示**

精灵增加了额外的汇编指令定义弱符号( 弱 ),设置符号大小( 大小 ),并表明是否符号是专门一个函数或一个对象( 类型)。 精灵还增加了一个指令,协助识别源文件或版本, 鉴别 。

**设备上装。 鉴别 :添加文件识别**

的 鉴别 指令允许任意的字符串数据添加到一个ELF对象文件,将被保存在对象和可执行文件,但不会加载到内存中的数据 . data 部分。 它通常用于储蓄 版本控制关键字等信息工具 **cvs** 或 **svn** 成文件,源创建对象的修改可以阅读使用 **鉴别** 大多数Unix系统上发现的命令。

该指令将一个或多个字符串参数。 每个参数保存在0-terminated字符串序列 .comment 部分对象的文件。 多种用途的 鉴别 指令是合法的,字符串将被保存到 .comment 节源文件中给出的顺序。

在NASM中语法,没有提供包装宏 鉴别 ,所以它必须用方括号。 示例使用在NASM中语法:

(识别“Id”美元)

**9.3.2。 大小 :设置符号大小**

精灵的符号表的存储大小的能力的象征。 这是常用的函数或数据对象。 虽然可以直接对特定的大小 常见的 符号, 大小 指令可以指定任何符号的大小,包括当地的象征。

指令接受两个参数,第一个参数是符号的名称,和第二个是大小。 可能是一个常量或表达式。 例子:

函数:

受潮湿腐烂

指标:最终

大小func func.end-func

**9.3.3。 类型 :设置符号类型**

精灵的符号表指示是否一个符号的能力是一个函数或数据。 虽然这可以直接指定 全球 指令(见 [9.4节](http://www.tortall.net/projects/yasm/manual/html/manual.html)), 类型 指令允许指定任何符号的符号类型,包括当地的符号。

指令接受两个参数,第一个参数是符号的名称,和第二个是符号类型。 符号类型必须 函数 或 对象 。 一个无法识别的类型会导致生成一个警告。 使用的例子:

函数:

受潮湿腐烂

类型函数函数

. data部分

var dd 4

类型变量对象

**9.3.4。 弱 :创建软弱的象征**

精灵允许某些符号定义为“软弱”。 弱符号类似于全局符号,除了在链接,弱符号只选择在全球和地方在符号解析符号。 与全球符号,多个目标文件可能声明相同的弱符号,并引用符号得到解决与弱符号只有在没有全局或局部符号有相同的名字。

这个功能主要是有用的库,希望提供常用功能但不与用户程序发生冲突。 例如,libc有一个系统调用(函数)称为“读”。  然而,实现一个线程的进程在用户空间使用POSIX线程,libpthread需要提供一个函数也称为“读”,它提供了一个阻塞接口,程序员,但实际上确实非阻塞调用内核。  允许一个应用程序与libc和libpthread(共享通用代码),libc需要其版本的系统调用与non-weak名称如“_sys_read”称为“读”的弱符号。 如果一个应用程序对有关libc,链接器找不到non-weak象征“读”,所以它将使用弱者。 如果相同的应用程序对libc有关 *和*libpthread,那么链接将链接“读”调用libpthread象征,忽略libc的弱者,不管库链接订单。 如果使用libc  non-weak名称,“读”功能程序最终可能取决于各种各样的因素;软弱的象征是一种告诉链接器符号是resolution-wise那么重要。

的 弱 指令需要一个参数,宣布弱符号名称。 例子:

weakfunc:

strongfunc:

受潮湿腐烂

弱weakfunc

全球strongfunc

**9.4。 精灵的扩展 全球 指令**

ELF目标文件可以包含更多信息的全局符号不仅仅是它的地址:它们可以包含符号的大小和它的类型。 这些不仅仅是调试器方便,但实际上是必要的程序编写时((共享库))。 Yasm因此支持一些扩展NASM语法 全球 指令(见 [5.6节](http://www.tortall.net/projects/yasm/manual/html/manual.html)),允许您指定这些特性。 Yasm还提供ELF-specific指令 [9.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)允许指定该信息非全局符号。

您可以指定一个全局变量是否是一个函数或一个数据对象通过向一个冒号和一词的名称 函数 或 数据 。 ((( 对象 )是同义词 数据 )。 例如:

全球hashlookup:函数,散列表:数据

出口全球的象征 hashlookup 作为一个函数, 哈希表 作为一个数据对象。

可选地,您可以控制的精灵能见度的象征。 能见度的添加一个关键词: 默认的 , 内部 , 隐藏的 ,或 受保护的 。 默认值是 默认的 ,当然。 例如, hashlookup 隐藏:

全球hashlookup:隐藏功能

您还可以指定与符号相关的数据的大小,作为一个数值表达式(可能涉及到标签,甚至向前引用)在类型说明符。 是这样的:

全球散列表:数据(散列表。 结束-散列表)

散列表:

db,一次;一些数据

指标:最终

这使得Yasm自动计算表的长度和位置信息到精灵符号表。 相同的信息可以得到更多的啰嗦地使用 类型 (见[9.3.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)), 大小 (见 [9.3.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html))指令如下:

全球散列表

哈希表对象类型

大小hashtable hashtable。 ——散列表

散列表:

db,一次;一些数据

指标:最终

声明全局符号的类型和尺寸是必要的写作时共享库代码。

**9.5。 精灵的扩展 常见的 指令**

精灵也允许您指定对齐要求共同的变量。 这是通过把数量(必须是2的幂)共同的变量的名称和大小后,(像往常一样)用冒号分开。 例如,一个双字将受益于4字节对齐的数组:

常见dwordarray 128:4

这个声明数组的总大小是128字节,并要求将其在一个4字节边界对齐。

**9.6。 elf32 特殊符号和 关于**

ELF规范包含足够的特性允许位置无关代码编写(PIC),这使得精灵共享库非常灵活。 然而,这也意味着Yasm必须能够生成各种奇怪的迁移类型在ELF对象文件中,如果它是一个可以编写汇编程序 照片。

因为精灵不支持segment-base引用 关于 运营商不用于其正常目的;因此Yasm elf32 利用输出格式 关于 不同的目的,即PIC-specific搬迁类型。

elf32 定义五个特殊符号,您可以使用右边的 关于 运营商获取图片迁移类型。 他们是 . . gotpc , . . gotoff , . .了 , . . plt 和 .信谊 。 它们的功能进行了总结:

. . gotpc

指的是符号标记 使用全局偏移表基地 关于. . gotpc 最终会让距离当前的开始部分全局偏移表。 (((_GLOBAL_OFFSET_TABLE_ )是标准的名称指象征 了)。 所以你将需要添加 $ $ 结果得到的真实地址。

. . gotoff

指的是在一个位置使用你自己的部分 关于. . gotoff 会给的距离到达指定位置,以便增加的地址有会给你想要的真正的地址位置。

. .了

指外部或全局符号使用 关于. .了 导致链接器建立一个条目 *在* 的包含了地址的象征,并参考了距离的开始要进入,所以您可以添加的地址,负载产生的地址和最终的地址的象征。

. . plt

指一个过程名称使用 关于. . plt 导致建立一个链接 过程符号链接表条目,并参考提供的地址 PLT条目。 你只能使用在上下文通常会生成一个PC-relative搬迁(即作为目的地 调用 或 无条件转移指令 ),因为精灵不包含搬迁绝对引用PLT条目类型。

.信谊

指的是使用符号名 关于……信谊 导致Yasm写一个普通的搬迁,而是让搬迁相对于部分的开始,然后添加偏移量的符号,它将编写一个搬迁记录直接针对问题的象征。 的区别是一个必要的动态链接器的特点。

**第十章。 elf64 :可执行文件和可链接格式64位对象文件**

**表的内容**

[10.1。 elf64 特殊符号和 关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的 elf64 对象格式是可执行的64位版本和可链接对象格式。 因为它有着许多相似之处 elf32 ,只有差异 elf32和 elf64 本章将描述。 有关 elf32 ,请参阅 [第九章](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

Yasm默认为 位64 当输出模式 elf64 对象的格式。

elf64 支持调试格式一样 elf32 然而, 刺穿了 调试格式是有限的32位地址,所以 dwarf2 (见 [第十九章](http://www.tortall.net/projects/yasm/manual/html/manual.html))推荐的调试格式。

elf64 还支持相同的部分,部分属性和指令 elf32 。 看到 [9.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)更多的细节部分属性和 [9.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)额外的指令精灵提供细节。

**10.1。 elf64 特殊符号和 关于**

的主要区别 elf32 和 elf64 (除了64位支持)是共享库的差异处理和位置无关代码。 作为 位64 允许使用的 把 相对寻址,大多数变量访问可以相对于RIP,允许简单的共享库的搬迁到不同的内存地址。

虽然RIP-relative寻址可用,它不能处理所有可能的变量访问模式,所以仍然需要有特殊符号,如 elf32 。 与elf32 , elf64 利用输出格式 关于 为利用 PIC-specific搬迁类型。

elf64 定义了四个特殊符号,您可以使用右边的 关于 运营商获取图片迁移类型。 他们是 . . gotpcrel , . .了 , . . plt 和 .信谊 。 它们的功能进行了总结:

. . gotpcrel

虽然RIP-relative寻址允许您编码指令指针相对数据参考 喷火 与 (rel  foo) ,有时需要编码RIP-relative引用指针linker-generated象征符号foo;这是使用 关于. .  gotpcrel ,如。 [rel foo关于. . gotpcrel] 。  不同于 elf32 ,这个搬迁,加上RIP-relative寻址,可以加载一个地址的((全局偏移表))使用一个单一的指令。  注意,因为RIP-relative引用仅限于一个签署了32位的位移, 有大小可以通过这种方法仅限于2 GB。

. .了

就像在 elf32 ,指外部或全局符号使用 关于. .了 导致链接器建立一个条目 *在* 的包含了地址的象征,并参考了距离的开始要进入,所以您可以添加的地址,负载产生的地址和最终的地址的象征。

. . plt

就像在 elf32 ,指的是一个过程名称使用 关于. . plt 导致建立一个链接 过程符号链接表条目,并参考提供的地址PLT条目。  你只能使用在上下文通常会生成一个PC-relative搬迁(即作为目的地 调用 或 无条件转移指令 ),因为精灵不包含搬迁绝对引用PLT条目类型。

.信谊

就像在 elf32 ,指的是一个符号名的使用 关于……信谊 导致Yasm写一个普通的搬迁,而是让搬迁相对于部分的开始,然后添加偏移量的符号,它将编写一个搬迁记录直接针对问题的象征。 的区别是一个必要的动态链接器的特点。

**第十一章。 elfx32 :精灵32位64位处理器对象文件**

**表的内容**

[11.1。 elfx32 特殊符号和 关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的 elfx32 对象格式的32位版本是64位的可执行文件和可链接对象格式执行。  类似于 elf64 ,它允许使用64位的寄存器和指令,但像 elf32 32位的指针大小,限制。  因为它有着许多相似之处 elf32 和 elf64 ,只有这些格式和之间的差异 elfx32 本章将描述。 有关 elf32 ,请参阅 [第九章](http://www.tortall.net/projects/yasm/manual/html/manual.html)为细节; elf64 ,请参阅 [第十章](http://www.tortall.net/projects/yasm/manual/html/manual.html)。 操作系统支持 elfx32 目前常见的比 elf64 。

Yasm默认为 位64 当输出模式 elfx32 对象的格式。

elfx32 支持相同的调试格式,部分,部分属性和指令 elf32 和 elf64 。 看到 [9.2节](http://www.tortall.net/projects/yasm/manual/html/manual.html)更多的细节部分属性和[9.3节](http://www.tortall.net/projects/yasm/manual/html/manual.html)额外的指令精灵提供细节。

**11.1。 elfx32 特殊符号和 关于**

由于RIP-relative寻址的可用性, elfx32 共享库处理和独立于位置的代码基本上是相同的 elf64 。

就像在 elf64 , elfx32 定义了四个特殊符号,您可以使用右边的 关于 运营商获取图片迁移类型。 他们是 . . gotpcrel , . .了 , . . plt 和 .信谊 和有相同的功能,因为他们做的 elf64 。 它们的功能进行了总结:

. . gotpcrel

虽然RIP-relative寻址允许您编码指令指针相对数据参考 喷火 与 (rel  foo) ,有时需要编码RIP-relative引用指针linker-generated象征符号foo;这是使用 关于. .  gotpcrel ,如。 [rel foo关于. . gotpcrel] 。  就像在 elf64 ,这个搬迁,加上RIP-relative寻址,可以加载一个地址的((全局偏移表))使用一个单一的指令。  注意,因为RIP-relative引用仅限于一个签署了32位的位移, 有大小可以通过这种方法仅限于2 GB。

. .了

就像在 elf64 ,指外部或全局符号使用 关于. .了 导致链接器建立一个条目 *在* 的包含了地址的象征,并参考了距离的开始要进入,所以您可以添加的地址,负载产生的地址和最终的地址的象征。

. . plt

就像在 elf64 ,指的是一个过程名称使用 关于. . plt 导致建立一个链接 过程符号链接表条目,并参考提供的地址PLT条目。  你只能使用在上下文通常会生成一个PC-relative搬迁(即作为目的地 调用 或 无条件转移指令 ),因为精灵不包含搬迁绝对引用PLT条目类型。

.信谊

就像在 elf64 ,指的是一个符号名的使用 关于……信谊 导致Yasm写一个普通的搬迁,而是让搬迁相对于部分的开始,然后添加偏移量的符号,它将编写一个搬迁记录直接针对问题的象征。 的区别是一个必要的动态链接器的特点。

**第十二章。 macho32 :32位马赫对象文件格式**

**第13章。 macho64 :马赫64位对象文件格式**

**第14章。 rdf :浮动动态对象文件格式**

**第15章。 win32 :微软Win32对象文件**

**表的内容**

[15.1。 win32 扩展 部分 指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[15.2。 win32 :安全结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的 win32 对象格式生成微软 Win32对象文件用于32位本地 Windows XP和Vista平台。 使用这个对象格式生成的对象文件可能与32位微软连接器等 Visual Studio生成32位 PE可执行文件。

的 win32 对象格式提供了一个默认的输出文件名扩展 .obj 。

注意,尽管微软说Win32对象文件遵循 COFF (通用对象文件格式)标准,Microsoft  Win32产生的对象文件编译器不兼容COFF连接器等DJGPP,反之亦然。 这是由于不同的意见在精确的语义PC-relative搬迁。  生产适合DJGPP COFF文件,使用 coff 输出格式;反之, coff 格式不产生目标文件,Win32连接器能产生正确的输出。

**15.1。 win32 扩展 部分 指令**

的 win32 对象格式允许您指定的附加信息 部分 指令,控制部分声明的类型和属性。 部分类型和属性自动生成Yasm标准部分的名字 。text , . data 和 .bss ,但仍有可能被这些限定符。

可用的限定符是:

代码 或 文本

定义了部分代码部分。 这标志着部分可读和可执行文件,但没有可写的,还指示链接器,部分代码的类型。

数据 或 bss

定义了部分数据部分,类似地 代码 。 数据部分被标记为可读和可写,但不是可执行的。 数据 声明了一个初始化数据部分,而 bss 声明了一个未初始化的数据部分。

rdata

声明了一个初始化数据段是可读的,但没有可写的。 微软编译器使用本节将常数。

信息

定义的部分 信息部分,其中不包括在链接器的可执行文件,但可能(例如)传递信息 *来* 链接器。 例如,声明一个 信息 类型章节.drectve 导致连接器部分的内容解释为命令行选项。

对齐= *n*

指定的对齐需求部分。 您可以指定的最高分数为8192:Win32对象文件格式包含不要求更大的部分对齐。  如果没有显式地指定对齐,默认值是16字节对齐的代码部分,8字节对齐rdata部分和4字节对齐的数据(BSS)部分。  信息部分的默认对齐方式1字节(没有对齐),虽然价值并不重要。 的定位必须是2的幂。

其他支持限定符控制特定部分标志: 丢弃 , 缓存 , 页面 , 分享 , 执行 , 读 , 写 , 基地 。 每集名称类似的部分标志,同时加上前缀 没有 清除相应的部分标志,如。 nodiscard 清除丢弃标志。

默认值由Yasm如果不指定上述限定符是:

部分。 对齐文本代码= 16

部分。 数据数据对齐= 4

部分。 rdata rdata对齐= 8

部分。 rodata rdata对齐= 8

部分。 rdata rdata对齐= 8美元

部分。 bss bss对齐= 4

部分。 drectve信息

部分.comment信息

其他部分的名字是默认处理 。text 。

**15.2。 win32 :安全结构化异常处理**

其他改进在Windows XP SP2和Windows Server 2003微软推出了“安全的概念结构化异常处理。  “总的想法是收集处理程序入口点在指定只读表,每个入口点验证之前对这个表异常控制被传递到处理程序。  为了创建一个可执行的安全异常处理程序表,链接器命令行上的每个对象文件必须包含一个特殊符号命名 @feat.00 。  如果任何对象文件传递给链接器没有这个符号,然后从可执行异常处理程序表省略,因此运行时检查为应用程序不会被执行。  默认情况下,表省略默默地从可执行文件如果发生这种情况,因此可以很容易被忽视。 用户可以指示链接器拒绝通过产生一个可执行的没有这个表 /  safeseh 命令行选项。

版本1.1.0,Yasm补充说这种特殊的符号 win32 对象文件,所以它的输出不无法链接 / safeseh 。

Yasm还指示来支持注册自定义的异常处理程序。 的 safeseh 指令指示汇编为安全生产适当格式化的输入数据异常处理程序表。 给出了一个典型的用例 [例15.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)。

**15.1 . Win32例子 safeseh** **例子**

部分。

走读生_MessageBoxA@16

safeseh处理程序,处理程序注册为“安全处理程序”

处理程序:

推动DWORD 1;MB_OKCANCEL

把字标题

推动DWORD文本

把字0

叫_MessageBoxA@16

子eax,1;顺便说一句适合作为返回值

,因为异常处理程序

受潮湿腐烂

全球_main

_main:

把字处理程序

推动DWORD[fs:0]

mov DWORD[fs:0],esp;异常处理程序

xor eax,eax

mov eax,DWORD[eax];导致异常

字流行[fs:0];解除异常处理程序

添加esp,4

受潮湿腐烂

文本:db”可以重新抛出,取消生成核心转储”,0

标题:db SEGV,0

部分。 drectve信息

db / defaultlib:user32。 lib / defaultlib:msvcrt。 自由的

如果应用程序有一个安全的异常处理程序表,试图执行任何程序未注册的异常处理程序将导致立即终止。 因此重要的是要登记每个异常处理程序的入口点 safeseh 指令。

所有提到的链接器在这部分参考微软链接器版本7。 x和。 的存在 @feat.00 符号和数据安全的异常处理程序表因为没有向后不兼容,因此“safeseh”生成对象文件仍然可以由链接器版本或非微软早些时候连接器连接起来。

**第十六章。 win64 :PE32 +(微软Win64)对象文件**

**表的内容**

[16.1。 win64 扩展 部分 指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2。 win64 结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.1。 约定x64堆栈、寄存器和功能参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.2。 类型的函数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.3。 框架功能结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.4。 堆栈框架的细节](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.5。 Yasm为解除操作原语](http://www.tortall.net/projects/yasm/manual/html/manual.html)

[16.2.6。 Yasm宏正式栈操作](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的 win64 或 x64 对象格式生成微软 Win64对象文件的64位本地使用 Windows XP x64(和Vista  x64)平台。 使用这个对象格式生成的对象文件可能与64位微软连接器等 Visual Studio 2005和2008年以生产64位 PE32  +可执行文件。

win64 提供了一个默认的输出文件名扩展 .obj 。

**16.1。 win64 扩展 部分 指令**

就像 win32 格式, win64 允许您指定的附加信息 部分 指令,控制部分声明的类型和属性。

**16.2。 win64 结构化异常处理**

大多数函数,利用堆栈的64位版本的Windows必须支持异常处理,即使他们没有内部使用的设施。 这是因为这些操作系统定位异常处理程序通过使用这一过程被称为“栈”,取决于功能提供数据,描述了他们如何使用堆栈。

当发生异常栈是“解除”工作向后通过函数调用链异常事件前确定功能是否具有适当的异常处理程序或他们是否有保存的非易失性寄存器的值需要恢复为了重建的执行上下文中的下一个更高的功能链。 这个过程取决于编译器和汇编程序提供“解除数据”功能。

以下部分详述的机制可用Yasm满足这些需求,从而允许函数写在汇编程序中使用的编码惯例遵守64位版本的Windows。 这些Yasm MASM。实现中提供设备遵循这些

**16.2.1。 约定x64堆栈、寄存器和功能参数**

[图16.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)显示了如何在函数调用栈通常使用。 当一个函数被调用时,一个8字节的返回地址是自动推到堆栈和函数然后保存它将使用的任何非易失性寄存器。 额外的空间也可以分配给本地变量和一个帧指针寄存器可以被指定。

**图16.1。 x64调用协定**

![img](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltYWdlczAuY25ibG9ncy5jb20vYmxvZy81MDI3MTEvMjAxNDA5LzA2MjIyMzU2NzAzOTYwNi5wbmc=.jpg)

前四个整数函数参数传递(从左到右顺序)寄存器RCX,RDX,R8和R9机型。  进一步的整数参数传递到栈上推在右到左的顺序(向左参数在低地址)。  栈空间是分配给四个寄存器参数(“影子空间”),但他们的价值观并不存储由调用函数调用的函数必须这样做,如果必要的。  调用函数有效地拥有这个空间,可以使用它为任何目的,因此,在返回调用函数不能依靠其内容。  注册参数占据最低有效位的寄存器和影子空间必须分配给四个寄存器参数,即使被调用的函数没有这么多参数。

前四个浮点参数传入XMM0 XMM3。 当混合整数和浮点参数时,参数和寄存器之间的通信是没有改变。 因此一个整数参数在两个浮点的R8 RCX和RDX未使用。

当它们通过值,结构和工会的尺寸8,16、32或64位传递相同大小的整数。 数组和更大的结构和工会作为指针传递给调用函数的内存分配和分配。

寄存器伸展,RCX,RDX、R8 R9机型,R10,R11是挥发性的,可以自由使用的调用函数没有保存它们的值(但是请注意,一些可用于传递参数)。 由于功能不能指望这些寄存器保存调用其他函数。

寄存器RBX RBP,肢体重复性劳损症,RDI R12,R13,R14、R15和XMM6 XMM15非易失性,必须保存和恢复功能,使用它们。

除了浮点值,返回XMM0,函数返回值,适合在递交的64位返回。 一些128位值也传入XMM0但较大的值返回调用程序在内存中分配的,并指出通过一个额外的“隐藏”函数参数,成为第一个参数和推动其他参数。 这个指针值也必须返回给调用程序在调用的程序返回时递交。

**16.2.2。 类型的函数**

函数分配堆栈空间,调用其他函数,非易失性寄存器保存或使用异常处理被称为“帧函数”;其他功能被称为“叶功能”。

帧函数使用一个区域在堆栈上称为“堆栈帧”和有一个定义的序言中设置。  通常他们在影子的位置保存寄存器参数(如果需要的话),他们使用的任何非易失性寄存器保存,为局部变量分配堆栈空间,并建立一个注册为一个堆栈帧指针。  他们还必须有一个或多个定义的尾声,免费的任何分配堆栈空间和恢复非易失性寄存器之前返回到调用函数。

除非动态分配堆栈空间,一个框架函数必须保持16字节对齐的堆栈指针,同时在其序言和后记外代码(在调用其他函数除外)。  一帧函数动态地分配堆栈空间必须首先需要分配任何固定的堆栈空间,然后分配和建立一个索引寄存器访问该地区。  这个区域的较低的基地址必须是16字节对齐的,必须提供注册而不管函数本身使得显式地使用它。  然后自由离开函数堆栈对齐在执行期间虽然必须重建16字节对齐如果或当它调用其他函数。

叶功能不需要定义序言或后记但他们不能调用其他函数;也不能改变任何非易失性寄存器和堆栈指针(这意味着他们不保持16字节的堆栈对齐在执行期间)。 不过,它们可以退出,跳转到另一个框架或叶的入口点函数提供了相应的堆参数是兼容的。

总结了这些规则 [表16.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)(函数的代码不属于序言或后记中提到表函数的身体)。

**表16.1。 功能结构化异常处理规则**

| **功能需求或可以:**                                          | **帧与帧指针寄存器函数**                                     | **框架没有帧指针寄存器函数** | **叶函数**                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| 序言和后记(s)                                                | 是的                                                         | 是的                         | 没有                                                         |
| 使用异常处理                                                 | 是的                                                         | 是的                         | 没有                                                         |
| 到栈上分配空间                                               | 是的                                                         | 是的                         | 没有                                                         |
| 保存或寄存器推压入堆栈                                       | 是的                                                         | 是的                         | 没有                                                         |
| 使用非易失性寄存器(在保存)                                   | 是的                                                         | 是的                         | 没有                                                         |
| 使用动态堆栈分配                                             | 是的                                                         | 没有                         | 没有                                                         |
| 在函数体改变堆栈指针                                         | 是的 ( [一个](http://www.tortall.net/projects/yasm/manual/html/manual.html)] | 没有                         | 没有                                                         |
| 在函数体对齐堆栈指针                                         | 是的 ( [一个](http://www.tortall.net/projects/yasm/manual/html/manual.html)] | 没有                         | 是的                                                         |
| 打电话给其他函数                                             | 是的                                                         | 是的                         | 没有                                                         |
| 使跳到其他功能                                               | 没有                                                         | 没有                         | 是的 ( [b](http://www.tortall.net/projects/yasm/manual/html/manual.html)] |
| ( [一个](http://www.tortall.net/projects/yasm/manual/html/manual.html)] 但16字节的堆栈对齐时,必须重新建立任何函数调用。 ( [b](http://www.tortall.net/projects/yasm/manual/html/manual.html)] 但是寄存器和堆栈的函数参数必须一致。 |                                                              |                              |                                                              |

**16.2.3。 框架功能结构**

已经表明,帧函数必须有一个良好定义的结构包括序言和一个或多个尾声,每个特定的形式。 一个函数中的代码不属于它的序言或其一个或多个结语将把这里称为函数的身体。

一个典型的功能序言的形式:

mov负责+ 8,rcx,参数存储在影子空间如果有必要

推动r14;保存任何要使用非易失性寄存器

推动r13;

子负责,大小;如果需要为局部变量分配堆栈

lea r13[偏差+负责];用r13帧指针偏移

当一个帧指针需要程序员可以选择使用哪个寄存器(“偏见”将在稍后解释)。 虽然它没有用于访问分配空间,它必须分配在序言和执行期间保持不变的主体功能。

如果使用大量的堆栈空间也是必要的电话 __chkstk 大小在递交之前分配堆栈空间为了如果需要内存页添加到堆栈(参见微软Visual Studio 2005文档详情)。

匹配的结语是:

lea负责,r13-bias;这不是官方后记的一部分

添加负责,大小;官方结语从这里开始

流行r13

流行r14

受潮湿腐烂

也可以使用以下规定,已经建立了一个帧指针寄存器:

lea负责,[r13 + size-bias]

流行r13

流行r14

受潮湿腐烂

这是只允许两种形式的尾声。 它必须与一个开始 添加负责,常量 指令或与 lea负责,(const +  fp_register) ;第一个表单可以使用有或没有一个帧指针寄存器但第二种形式要求。  这些指令然后其次是零个或多个8字节寄存器持久性有机污染物和返回指令(可替换为一组有限的跳转指令如微软文档中所述)。  结语形式高度受限,因为这允许例外调度代码来定位他们不需要解除数据除了提供序言。

每个函数的位置和数据长度的序幕,在任何固定堆栈分配和任何非易失性寄存器保存在对象代码是记录在特殊的部分。 Yasm提供宏创建这些数据,现在将描述(例子的方式使用它们)。

**16.2.4。 堆栈框架的细节**

有两种类型的堆栈帧,在创建解除数据需要考虑。

第一,左边所示 [图16.2](http://www.tortall.net/projects/yasm/manual/html/manual.html),仅涉及一个固定分配堆栈空间和结果的堆栈指针仍然固定在价值函数的身体除了在调用其他函数。 在这种类型的堆栈帧堆栈指针值的序言作为补偿的基础在后面描述的解除原语和宏。 它必须是16字节对齐的。

**图16.2。 x64详细的堆栈帧**

![img](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltYWdlczAuY25ibG9ncy5jb20vYmxvZy81MDI3MTEvMjAxNDA5LzA2MjIyMzU3MDMyMzgwNi5wbmc=.jpg)

第二种类型的框架,所示 [图16.2](http://www.tortall.net/projects/yasm/manual/html/manual.html)栈空间是动态分配的结果,堆栈指针的值是静态预测,不能作为解除补偿。 在这种情况下必须使用帧指针寄存器提供这个基地址。 解除补偿的基础是低端的固定在堆栈上分配区域,这是通常的价值框架时的堆栈指针寄存器分配。 一定是16字节对齐,必须分配任何解除之前宏与补偿。

为了让最大数量的数据访问单个字节偏移量\ + 127(-128)从帧指针寄存器,它是正常的,以抵消其价值对分配的中心区域(之前介绍的“偏见”)。 帧指针寄存器的身份,这抵消必须16字节的倍数,记录在解除数据允许堆栈帧基地址计算帧寄存器中的值。

**16.2.5。 Yasm为解除操作原语**

这是低水平设施Yasm提供创建解除数据。

proc_frame *的名字*

生成一个函数表条目 .pdata 和解除信息 .xdata 一个函数的结构化异常处理数据。

[pushreg *注册* ]

为指定的非易失性寄存器生成解除数据。 只使用非易失性整数寄存器;易失性寄存器使用 (allocstack 8) 代替。

[setframe *注册* , *抵消* ]

生成解除数据寄存器和堆栈帧偏移量。 偏移量必须16的倍数和小于或等于240。

[allocstack *大小* ]

生成撤销堆栈空间的数据。 尺寸必须是8的倍数。

[savereg *注册* , *抵消* ]

生成解除指定的寄存器和数据抵消;必须积极抵消8的倍数的基础相对于过程的框架。

[savexmm128 *注册* , *抵消* ]

生成解除数据指定XMM寄存器和抵消;必须积极抵消16的倍数的基础相对于过程的框架。

[pushframe *代码* ]

生成解除40或48字节的数据(使用一个可选的错误代码)框架用于存储硬件的结果异常或中断。

(endprolog)

信号的序言,必须在第一个255字节的函数。

endproc_frame

用在函数开始的结束 proc_frame 。

[例16.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)显示了如何使用这些原语(这是基于Microsoft Visual Studio 2005中提供一个例子文档)。

**例16.1。 Win64解除原语**

PROC_FRAME样本

db 0 x48;发出雷克斯前缀,使热修补

推动rbp;拯救未来的帧指针

[pushreg rbp];创建解除数据rbp寄存器推

子负责,0 x40;分配堆栈空间

[]allocstack 0 x40;创建解除数据堆栈分配

lea rbp,[负责+ 0 x20];分配帧指针的偏见32

[setframe rbp,0 x20);在rbp创建解除一帧数据寄存器

xmm7 movdqa(rbp),节省非易失性寄存器XMM

[savexmm128 xmm7,0 x20);创建解除XMM寄存器保存数据

mov(rbp + 0 x18),肢体重复性劳损症;保存肢体重复性劳损症

[savereg rsi,0 x38];创建解除数据保存肢体重复性劳损症

mov负责+ 0 x10,rdi;拯救rdi

(savereg rdi 0 x10);创建rdi的解除数据保存

(endprolog)

;我们可以改变堆栈指针之外的开场白,因为我们

,有一个帧指针。 如果我们没有一个这将是非法的。

;帧指针是必要的因为这个堆栈指针修改。

子负责,0 x60;我们可以自由修改堆栈指针

mov伸展,0;我们可以解除这个访问违例

mov伸展,伸展

movdqa xmm7(rbp);恢复寄存器没有保存

mov rsi(rbp + 0 x18);推动;这不是的一部分

mov rdi[rbp-0x10];官方跋

lea负责,(rbp + 0 x20),这是官方的尾声

流行rbp

受潮湿腐烂

ENDPROC_FRAME

**16.2.6。 Yasm宏正式栈操作**

从前面给出的YASM原语的描述可以看出,每个正常的堆栈操作之间有密切的关系和相关的原始需要生成其解除数据。 由于它是明智的,提供一组宏执行两个操作在一个宏调用。 Yasm提供了以下宏结合的两个操作。

proc_frame *的名字*

生成一个函数表条目 .pdata 和解除信息 .xdata 。

alloc_stack *n*

分配一个堆栈区 *n* 字节。

save_reg *注册* , *疯狂的*

节省了非易失性寄存器 *注册* 在抵消 *疯狂的* 在堆栈上。

push_reg *注册*

将非易失性寄存器 *注册* 在堆栈上。

rex_push_reg *注册*

将非易失性寄存器 *注册* 在堆栈上使用一个2字节指令。

save_xmm128 *注册* , *疯狂的*

节省了非易失性寄存器XMM *注册* 在抵消 *疯狂的* 在堆栈上。

set_frame *注册* , *疯狂的*

设置框架注册 *注册* 来抵消 *疯狂的* 在堆栈上。

push_eflags

推动eflags寄存器

push_rex_eflags

推动eflags寄存器使用一个2字节指令(允许热修补)。

push_frame *代码*

推动一个40字节帧和一个可选的8字节错误代码压入堆栈。

end_prologue , end_prolog

结束函数序言(这是另一种选择 (endprolog) )。

endproc_frame

使用的职能之一 proc_frame 。

[例16.2](http://www.tortall.net/projects/yasm/manual/html/manual.html)是 [例16.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)使用这些更高层次的宏。

**例16.2。 Win64解除宏**

PROC_FRAME样本;开始的序幕

rex_push_reg rbp;推动未来的帧指针

alloc_stack 0 x40;分配64字节的堆栈空间

set_frame rbp,0 x20;设置一个框架注册(负责+ 32)

save_xmm128 xmm7 0 x20;节省xmm7,肢体重复性劳损症和rdi到本地堆栈空间

save_reg肢体重复性劳损症,0 x38;放松基地地址:[rsp_after_entry - 72]

save_reg rdi,0 x10;框架寄存器值:[rsp_after_entry - 40]

END_PROLOGUE

子负责,0 x60;我们现在可以改变堆栈指针

mov伸展,0;解除这个访问违例

mov伸展,伸展,因为我们有一个帧指针

movdqa xmm7(rbp);恢复寄存器,没救了

mov rsi(rbp + 0 x18);推动(而不是官方跋的一部分)

mov rdi[rbp-0x10]

lea负责(rbp + 0 x20),官方的尾声

流行rbp

受潮湿腐烂

ENDPROC_FRAME

**第十七章。 xdf :扩展动态对象的格式**

**第五部分。 调试格式**

书的这一部分文档中的章节Yasm对各种调试格式的支持。

**第18章。 cv8 :VC8 CodeView调试格式**

**19章。 dwarf2 :DWARF2调试格式**

**第20章。 刺穿了 :刺穿了调试格式**

**第六部分架构。**

书的这一部分文档中的章节Yasm对各种指令集架构的支持。

**21章。 x86体系结构**

的 x86架构的通用名称是多供应商16位,32位,最近64位架构。  最初由英特尔的8086系列CPU,英特尔在80386年扩展到32位的CPU,并延长AMD Opteron 64位,Athlon 64 CPU。  而在2007年,英特尔和AMD是最高的x86处理器体积制造商,许多其他供应商也生产x86处理器。  通常制造商已经交换专利权(或复制)主要架构的改进,但也有一些独特的功能出现在许多的实现。

**21.1。说明**

x86体系结构有一个可变的指令大小,允许适度的代码压缩,同时允许非常复杂的操作数组合以及一个非常大的和许多扩展指令集的大小。 指令一般变化从零到三个操作数只有一个内存操作数允许的。

**21.1.1。 NOP填充**

不同的处理器有不同的建议 用于NOP(没有操作)指令 在代码中填充。  填充通常执行循环边界对齐最大化性能,是关键,填充物本身添加最小的开销。 而1字节NOP 90  h 是标准的x86实现,所有最近的一代又一代的处理器推荐不同时间填充序列最优性能。  大多数处理器声称686(例如后奔腾)一代或新featureset支持“长”NOP码 0 fh 1跳频 直到最近,尽管这操作码是非法。  旧处理器不支持这些专用长NOP操作码一般建议选择不再NOP序列;虽然这些序列做空操作,会导致在新处理器解码效率低下。

由于各种NOP建议,由Yasm生成的代码 对齐 指令取决于执行模式( 位 )设置和选择的处理器 CPU 指令(见 [21.2.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 [表21.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)列出了各种组合生成的空操作。

**表21.1。 x86 NOP填充模式**

| **位** | **CPU**                 | **填充**                   |
| ------ | ----------------------- | -------------------------- |
| 16     | 任何                    | 16位短空操作               |
| 32     | 没有,或少于686          | 32位短空操作(没有长空操作) |
| 32     | 686或最新的英特尔处理器 | 英特尔指南,使用长空操作    |
| 32     | 转K6或更新的AMD处理器   | AMD K10指南,使用长空操作   |
| 64年   | 没有一个                | 英特尔指南,使用长空操作    |
| 64年   | 686或最新的英特尔处理器 | 英特尔指南,使用长空操作    |
| 64年   | 转K6或更新的AMD处理器   | AMD K10指南,使用长空操作   |

此外,上述违约可能覆盖通过选项之一 [表21.2](http://www.tortall.net/projects/yasm/manual/html/manual.html)到 CPU 指令。

**表21.2。 x86 NOP CPU** **指令选项**

| **的名字** | **描述**                 |
| ---------- | ------------------------ |
| basicnop   | 长时间空操作不习惯       |
| intelnop   | 英特尔指南,使用长空操作  |
| amdnop     | AMD K10指南,使用长空操作 |

**21.2。 执行模式和扩展**

x86在许多方面已经扩展在整个历史进程中,剩下的大多是向后兼容的同时添加执行模式和大扩展指令集。现代x86处理器可以在四种主要模式:16位真正的模式中,16位保护模式,32位保护模式,和64位长模式。 的主要区别真实和保护模式的处理部分:在实模式段直接寻址内存16字节的页面,而在保护模式片段而不是索引到描述符表,其中包含的物理基础和大小。  32位保护模式允许分页和虚拟内存,以及一个32位的,而不是一个16位的偏移量。

16位和32位的操作模式都允许使用16位和32位寄存器通过设置操作和指令前缀地址大小16位或32位,与活跃的操作模式设置默认操作规模和“其他”大小被标记为一个前缀。  这些操作和地址大小也影响直接操作数的大小:例如,一个与一个32位的指令操作的大小和立即操作数将有一个32位值的编码指令,除了优化等符号扩展8位值。

与16位和32位模式不同的是,64位长模式更多的是一种打破“遗留”的模式。 长模式废止几个指令。  也是唯一的64位寄存器可用模式;64位寄存器不能从16位或32位访问模式。 与其他模式,大多数编码值模式是有限大小32位长。  一个小的子集 MOV 指令允许64位编码值,但在其他值大于32位指令必须来自一个寄存器。  部分原因是这个限制,也由于浮动的广泛使用共享库,长模式还增加了一个新的寻址模式: 把 相对。

**21.2.1。 CPU的选择**

NASM解析器允许设置哪些子集的指令和操作数被Yasm通过使用 CPU 指令(见 [5.8节](http://www.tortall.net/projects/yasm/manual/html/manual.html))。 随着x86架构的大量的扩展,“SSE3”等特定功能标志和CPU可以指定“P4”等名称。 功能标志均正常,“不”前缀版本打开或关闭一个特性,而CPU名称打开只列出的功能,关闭所有其他功能。 [表21.3](http://www.tortall.net/projects/yasm/manual/html/manual.html)列出了特征标志 [表21.4](http://www.tortall.net/projects/yasm/manual/html/manual.html)列出了CPU名称Yasm支持。 同时拥有功能标志和CPU名称允许组合等 CPU P3 nofpu 。 特色旗帜和CPU名称不区分大小写。

**表21.3。 x86处理器功能标志**

| **的名字**                  | **描述**                              |
| --------------------------- | ------------------------------------- |
| FPU                         | 浮点单元(FPU)指令                     |
| MMX                         | MMX SIMD指令                          |
| 上交所                      | 流SIMD扩展(SSE)指令                   |
| SSE2                        | 流SIMD指令扩展2                       |
| SSE3                        | 流SIMD指令扩展3                       |
| SSSE3                       | 补充流SIMD指令扩展3                   |
| SSE4.1                      | 流SIMD扩展4,彭林子集(47个指令)        |
| SSE4.2                      | 流SIMD扩展4,Nehalem子集(7指令)        |
| SSE4                        | 所有流SIMD指令扩展4(SSE4.1和SSE4.2)   |
| SSE4a                       | 4流SIMD扩展(AMD)                      |
| SSE5                        | 流SIMD扩展5                           |
| XSAVE                       | XSAVE指令                             |
| AVX                         | 先进的矢量扩展指令                    |
| 菲利普-马萨                 | 延时指令                              |
| AES                         | 高级加密标准说明                      |
| CLMUL,PCLMULQDQ             | PCLMULQDQ指令                         |
| 3 dnow                      | 3 dnow ! 指令                         |
| 新瑞仕                      | Cyrix-specific指令                    |
| AMD                         | 比转K6 AMD-specific指令(旧)           |
| 多发性骨髓瘤                | 系统管理模式指令                      |
| 普罗特,保护                 | 只保护模式指令                        |
| Undoc,无证                  | 非法指令                              |
| 奥林匹克广播服务公司,过时了 | 过时的指令                            |
| 我感到,特权                 | 特权指令                              |
| 支持向量机                  | 安全虚拟机指令                        |
| 挂锁                        | 通过挂锁指令                          |
| EM64T                       | 英特尔EM64T或更好的指令(不一定仅64位) |

**表21.4。 x86处理器的名字**

| **的名字**                                       | **功能标志**                                                 | **描述**                 |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------ |
| 8086年                                           | 我感到                                                       | 英特尔8086年             |
| 186年,80186年,i186                               | 我感到                                                       | 英特尔80186年            |
| 286年,80286年,i286                               | 我感到                                                       | 英特尔80286年            |
| 386年,80386年,i386                               | 多发性骨髓瘤,普罗特,我感到                                   | 英特尔80386年            |
| 486年,80486年,i486                               | FPU,多发性骨髓瘤,普罗特,我感到                               | 英特尔80486年            |
| 586年,i586奔腾,P5                                | FPU,多发性骨髓瘤,普罗特,我感到                               | 英特尔奔腾               |
| 686年,i686 P6、PPro PentiumPro                   | FPU,多发性骨髓瘤,普罗特,我感到                               | 英特尔奔腾               |
| P2,Pentium2、Pentium-2 PentiumII,奔腾ii          | MMX,FPU,多发性骨髓瘤,普罗特,我感到                           | 英特尔奔腾II             |
| P3、Pentium3奔腾3、PentiumIII pentium iii,卡特迈 | 上交所、MMX FPU,多发性骨髓瘤,普罗特,我感到                   | 英特尔奔腾III            |
| P4、Pentium4奔腾4,PentiumIV Williamette奔腾iv    | SSE2、SSE MMX,FPU,多发性骨髓瘤,普罗特,我感到                 | 英特尔奔腾4              |
| IA64,安腾ia - 64                                 | SSE2、SSE MMX,FPU,多发性骨髓瘤,普罗特,我感到                 | 英特尔安腾(x86)          |
| 转K6                                             | 3 dnow MMX,FPU,多发性骨髓瘤,普罗特,我感到                    | AMD转K6                  |
| 速龙,K7                                          | 上交所,3 dnow MMX,FPU,多发性骨髓瘤,普罗特,我感到             | 的AMD Athlon             |
| 锤,抓奏的,皓龙处理器、Athlon64速龙- 64           | SSE2、上交所、3 dnow MMX,FPU,多发性骨髓瘤,普罗特,我感到      | AMD Athlon64和皓龙处理器 |
| 普莱斯考特                                       | SSE3 SSE2,上交所MMX,FPU,多发性骨髓瘤,普罗特,我感到           | 英特尔代号普雷斯科特     |
| Conroe,嵌件                                      | SSSE3、SSE3 SSE2、SSE MMX,FPU,多发性骨髓瘤,普罗特,我感到     | 英特尔代号Conroe         |
| 彭林                                             | SSE4.1、SSSE3 SSE3 SSE2,SSE,MMX,FPU,多发性骨髓瘤,普罗特,我感到 | 英特尔代号彭林           |
| Nehalem,Corei7                                   | XSAVE、SSE4.2 SSE4.1、SSSE3 SSE3,SSE2,上交所,MMX,FPU,多发性骨髓瘤,普罗特,我感到 | 英特尔代号Nehalem        |
| Westmere                                         | CLMUL,AES、XSAVE SSE4.2、SSE4.1 SSSE3,SSE3,SSE2,上交所,MMX,FPU,多发性骨髓瘤,普罗特,我感到 | 英特尔代号Westmere       |
| Sandybridge                                      | AES,AVX,CLMUL XSAVE、SSE4.2 SSE4.1,SSSE3,SSE3,SSE2,上交所,MMX,FPU,多发性骨髓瘤,普罗特,我感到 | 英特尔代号桑迪大桥       |
| 威尼斯                                           | SSE3 SSE2,上交所、3 dnow MMX,FPU,多发性骨髓瘤,普罗特,我感到  | AMD代号威尼斯            |
| Family10h K10、杰出人才                          | SSE4a、SSE3 SSE2 SSE,3 dnow,MMX,FPU,多发性骨髓瘤,普罗特,我感到 | AMD代号K10               |
| 推土机                                           | SSE5、SSE4a SSE3 SSE2,SSE,3 dnow MMX,FPU,多发性骨髓瘤,普罗特,我感到 | AMD代号推土机            |

为了获得64位指令, *这两个* 必须选择一个64位CPU能力,和64位组装模式必须设置(在NASM中语法)通过使用 位64 (见 [5.1节](http://www.tortall.net/projects/yasm/manual/html/manual.html))或定位一个64位的对象格式等 elf64 。

默认为最新的处理器CPU设置和所有特性标志启用;例如x86指令的处理器,包括所有的指令集扩展和64位指令。

**21.3。寄存器**

64位x86寄存器设置由16个通用寄存器,其中只有8个16位和32位模式下可用。  8个16位寄存器是核心 斧头 , BX, 残雪 , DX , 如果 , 迪 , 英国石油公司 , SP 。  最重要的8位的第一个四个寄存器可通过 艾尔 , 提单 , CL, 戴斯。莱纳姆: 在所有的执行模式。  在64位模式下,最重要的8位其他四个寄存器也可访问;这些命名 银 , 迪勒 , SPL , 底保 。  最重要的8位前四16位寄存器也可用,虽然有一些限制时,可以用在64位模式;这些都是命名啊 , 黑洞 , CH , DH 。

80386年扩展这些寄存器为32位,同时保留所有的16位和8位的名字在16位模式下可用。 用添加一个新的扩展寄存器 *E* 前缀,因此核心八32位寄存器命名 EAX , EBX , 连成一片 , EDX , 应急服务国际公司 , EDI , EBP , ESP。 最初的8位和16位寄存器名称映射到最不重要的部分32位寄存器。

64位长模式进一步扩展这些寄存器的规模通过添加64位 *R* 16位的名字前缀;因此,基地8个64位寄存器命名 递交, RBX 等。长模式还添加了八个额外的寄存器指定数值 r8 通过 r15 。 最重要的32位的寄存器都可以通过 *d*后缀( r8d 通过 r15d ),最重要的16位通过 *w* 后缀( r8w 通过 r15w ),并通过一个最重要的8位 *b* 后缀( r8b通过 r15b )。

[图21.1](http://www.tortall.net/projects/yasm/manual/html/manual.html)总结了完整的64位x86通用寄存器组。

**图21.1。 x86通用寄存器**

![img](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltYWdlczAuY25ibG9ncy5jb20vYmxvZy81MDI3MTEvMjAxNDA5LzA2MjIyMzU3NTQ3ODk0OS5wbmc=.jpg)

**21.4。分割**

**指数**

**符号**

! =, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

美元

在这里, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

前缀, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html), [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

$ $, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%运算符, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% $, [Context-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% $ $, [Context-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% %, [Macro-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% %运算符, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% +, [连接一行宏观标记:% +](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% + 1, [码作为宏观参数条件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% 1, [码作为宏观参数条件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% 0, [默认宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html), [% 0:宏观参数计数器](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%分配, [预处理器变量:%分配](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%, [标准宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%定义, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elif, [有条件的组装](http://www.tortall.net/projects/yasm/manual/html/manual.html), [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifctx, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifdef, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifid, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifidn, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifidni, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifmacro, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnctx, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifndef, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnid, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnidn, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnidni, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnmacro, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnnum, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnstr, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifnum, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% elifstr, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%, [有条件的组装](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% endrep, [预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%的错误, [%错误:用户定义的错误报告](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% exitrep, [预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% iassign, [预处理器变量:%分配](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% idefine, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%,如果 [有条件的组装](http://www.tortall.net/projects/yasm/manual/html/manual.html), [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifctx, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html), [IFs示例使用上下文堆栈:块](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%如果定义了, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifid, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifidn, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifidni, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifmacro, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnctx, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%如果未定义, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnid, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnidn, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnidni, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnmacro, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnnum, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnstr, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifnum, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% ifstr, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% imacro, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%包括, [包括其他文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%的宏, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%的流行, [上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html), [%和%推流行:创建和删除背景](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%, [上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html), [%和%推流行:创建和删除背景](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%代表, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% repl, [% repl:重命名上下文](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%旋转, [%:旋转旋转宏观参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% strlen, [字符串长度:% strlen](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%的子串, [子:%的子串](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% undef, [它通过宏:% undef](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% xdefine, [提高%定义:% xdefine](http://www.tortall.net/projects/yasm/manual/html/manual.html)

% xidefine, [提高%定义:% xdefine](http://www.tortall.net/projects/yasm/manual/html/manual.html)

&运算符, [&:按位与运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

& &, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

*操作符, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

+修改器, [贪婪的宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

+操作符

二进制, [+和-:加法和减法运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

一元, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

——操作符

二进制, [+和-:加法和减法运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

一元, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

——mapfile, [映射文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

\- f, [关键字__YASM_OBJFMT__ __OUTPUT_FORMAT__:输出对象格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

\- p, [包括其他文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . @, [Macro-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . @符号前缀, [当地的标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. ., [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . gotoff, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . gotpc, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . gotpcrel, [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. . plt, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

. .信谊, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

com, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.comment, [鉴别:添加文件识别](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.drectve, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.nolist, [禁用清单扩张](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.obj, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.pdata, [win64结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

。系统, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

.xdata, [win64结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

/运营商, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

/ /运算符, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

16位模式

与32位模式下, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

32位, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

32位模式

对64位模式, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

32位的共享库, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

64位的, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html), [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

64位的共享库, [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

<, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

< <操作符, [< <和> >:移位操作符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

< =, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

< >, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

=、 [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

= =, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

\>, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

\> =, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

\> >操作符, [< <和> >:移位操作符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

?, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

(地图) [映射文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

^运营商, [^:按位异或运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

^ ^, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__FILE__, [__FILE__ __LINE__:文件名和行号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__LINE__, [__FILE__ __LINE__:文件名和行号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__OUTPUT_FORMAT__, [关键字__YASM_OBJFMT__ __OUTPUT_FORMAT__:输出对象格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__SECT__, [__SECT__宏](http://www.tortall.net/projects/yasm/manual/html/manual.html), [绝对绝对:定义标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_BUILD__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_MAJOR__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_MINOR__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_OBJFMT__, [关键字__YASM_OBJFMT__ __OUTPUT_FORMAT__:输出对象格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_SUBMINOR__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_VERSION_ID__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

__YASM_VER__, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

|运营商, [|:按位或运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

| |, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

~运营商, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**一个**

ABS、 [把相对寻址](http://www.tortall.net/projects/yasm/manual/html/manual.html), [默认值:改变汇编违约](http://www.tortall.net/projects/yasm/manual/html/manual.html)

绝对的, [绝对绝对:定义标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

另外, [+和-:加法和减法运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

地址大小前缀, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

%的信号后, [连接宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

代数, [有效的地址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

对齐, [对齐和ALIGNB:数据对齐](http://www.tortall.net/projects/yasm/manual/html/manual.html), [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

代码, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

ALIGNB, [对齐和ALIGNB:数据对齐](http://www.tortall.net/projects/yasm/manual/html/manual.html)

对齐

代码, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在win32中, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常见的变量, [精灵共同扩展指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

对齐的精灵, [精灵共同扩展指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

amd64, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html), [x86体系结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

amdnop, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

任意的数字表达式, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在宏观参数, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

汇编指令, [NASM汇编指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

组装, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在, [ISTRUC,IEND:声明的实例的结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**B**

basicnop, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

本, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

二进制, [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [+和-:加法和减法运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

二进制文件, [INCBIN:包括外部的二进制文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

二进制, [ORG:二进制起源](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的转变, [< <和> >:移位操作符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

位, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

位, [&:按位与运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

按位或 [|:按位或运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

按位异或, [^:按位异或运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

块IFs, [IFs示例使用上下文堆栈:块](http://www.tortall.net/projects/yasm/manual/html/manual.html)

牙套

%的信号后, [连接宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在宏观参数, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**C**

电话, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

区分大小写, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html), [提高%定义:% xdefine](http://www.tortall.net/projects/yasm/manual/html/manual.html), [预处理器变量:%分配](http://www.tortall.net/projects/yasm/manual/html/manual.html)

不区分大小写, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

区分大小写, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

变化的部分, [改变和定义部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符常量, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符常量, [字符常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

循环引用, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

代码, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

CodeView, [VC8 cv8:CodeView调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

coff, [coff:通用对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

COFF

调试, [刺穿:刺穿了调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

结肠癌、 [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常见的, [常见的:定义常见的数据区域](http://www.tortall.net/projects/yasm/manual/html/manual.html)

通用对象文件格式, [coff:通用对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常见的变量, [常见的:定义常见的数据区域](http://www.tortall.net/projects/yasm/manual/html/manual.html)

对齐的精灵, [精灵共同扩展指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

连接宏观参数, [连接宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

条件代码作为宏观参数, [码作为宏观参数条件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

有条件的大会, [有条件的组装](http://www.tortall.net/projects/yasm/manual/html/manual.html)

conditional-return宏, [码作为宏观参数条件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常数, [常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常数, [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

上下文堆栈, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

上下文堆栈, [上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html), [IFs示例使用上下文堆栈:块](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Context-Local标签, [Context-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Context-Local单行的宏, [Context-Local单行的宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

计算宏观参数, [% 0:宏观参数计数器](http://www.tortall.net/projects/yasm/manual/html/manual.html)

CPU、 [CPU:定义CPU依赖性](http://www.tortall.net/projects/yasm/manual/html/manual.html)

CPUID, [字符常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

创建上下文, [%和%推流行:创建和删除背景](http://www.tortall.net/projects/yasm/manual/html/manual.html)

关键表达式, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [装备:定义常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [预处理器变量:%分配](http://www.tortall.net/projects/yasm/manual/html/manual.html), [绝对绝对:定义标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

关键的表情, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

cv8, [VC8 cv8:CodeView调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**D**

数据, [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DB, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

弟弟, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DDQ, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DDQWORD, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

调试, [dwarf2:dwarf2调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [刺穿:刺穿了调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

声明的结构, [STRUC ENDSTRUC:声明数据类型结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

默认情况下, [把相对寻址](http://www.tortall.net/projects/yasm/manual/html/manual.html), [默认值:改变汇编违约](http://www.tortall.net/projects/yasm/manual/html/manual.html)

默认情况下, [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

默认的宏观参数, [默认宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

定义部分, [改变和定义部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的指令, [精灵的指示](http://www.tortall.net/projects/yasm/manual/html/manual.html)

禁用清单扩张, [禁用清单扩张](http://www.tortall.net/projects/yasm/manual/html/manual.html)

部门, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

做的, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DQ, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DT, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

复制品, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

DW, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

矮, [dwarf2:dwarf2调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

dwarf2, [dwarf2:dwarf2调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**E**

有效的地址, [有效的地址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

有效的地址, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

有效地址, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

精灵, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的指令, [精灵的指示](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elf32, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elf64, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elfx32, [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

节中, [精灵的部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

符号的大小, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html)

符号类型, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

弱引用, [弱势:创建软弱的象征](http://www.tortall.net/projects/yasm/manual/html/manual.html)

精灵

32位的共享库, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

64位的共享库, [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

调试, [dwarf2:dwarf2调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [刺穿:刺穿了调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x32共享库, [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elf32, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elf64, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

elfx32, [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

ENDSTRUC, [STRUC ENDSTRUC:声明数据类型结构](http://www.tortall.net/projects/yasm/manual/html/manual.html), [绝对绝对:定义标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

装备, [装备:定义常量](http://www.tortall.net/projects/yasm/manual/html/manual.html), [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

具体文本的身份, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

可执行的和可链接的格式, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

64位的, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x32, [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

导出符号, [全球:导出符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

表情, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

扩展动态对象, [xdf:扩展动态对象的格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

外来的, [走读生:导入符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**F**

指针, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

闪光灯, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

平面形式二进制, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

浮点数, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html), [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常数, [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

之前, [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

特定于格式的指令, [NASM汇编指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

向前引用, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

FreeBSD, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

函数, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**G**

gdb, [dwarf2:dwarf2调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [刺穿:刺穿了调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

全球性的, [全球:导出符号](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

全局偏移表, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

了, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

图形, [INCBIN:包括外部的二进制文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

贪婪的宏观参数, [贪婪的宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

组, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**H**

在这里, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

十六进制, [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

隐藏的, [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**我**

识别, [鉴别:添加文件识别](http://www.tortall.net/projects/yasm/manual/html/manual.html)

IEND, [ISTRUC,IEND:声明的实例的结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

直接的, [直接操作数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

导入符号, [走读生:导入符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在win32中, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

在win32中, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

INCBIN, [INCBIN:包括外部的二进制文件](http://www.tortall.net/projects/yasm/manual/html/manual.html), [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

包括其他文件, [包括其他文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

无限循环, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

信息部分, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

初始化, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

结构的实例, [ISTRUC,IEND:声明的实例的结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

英特尔数字格式, [浮点型常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

intelnop, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

内部的, [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

ISTRUC, [ISTRUC,IEND:声明的实例的结构](http://www.tortall.net/projects/yasm/manual/html/manual.html)

遍历宏观参数, [%:旋转旋转宏观参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**l**

标签前缀, [当地的标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

图书馆, [弱势:创建软弱的象征](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Linux

精灵, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x32, [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

低位优先, [字符常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

LMA, [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

当地的标签, [当地的标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

逻辑, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

逻辑或, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

逻辑异或, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**米**

Mac OSX, [macho32:马赫32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [macho64:马赫64位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Mach-O, [macho32:马赫32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html), [macho64:马赫64位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

男子气概

macho32, [macho32:马赫32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

macho64, [macho64:马赫64位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

macho32, [macho32:马赫32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

macho64, [macho64:马赫64位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

宏处理器, [NASM预处理器](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Macro-Local标签, [Macro-Local标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

宏, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

地图文件, [映射文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

内存引用, [有效的地址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

微软Visual Studio 2010中, [VSYASM——Yasm微软Visual Studio 2010](http://www.tortall.net/projects/yasm/manual/html/manual.html)

模运算符, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

MSBUILD, [VSYASM——Yasm微软Visual Studio 2010](http://www.tortall.net/projects/yasm/manual/html/manual.html)

多行宏观存在, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

多行宏, [多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

多行宏, [重载多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

乘法, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

multipush, [%:旋转旋转宏观参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**N**

NOP, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

NOSPLIT, [有效的地址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

数值常数, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

数字常量, [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**O**

对象, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

八进制 [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

常见的变量, [精灵共同扩展指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的符号, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html), [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

省略参数, [默认宏参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

一个人的补充, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

operand-size前缀, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

操作数, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

运营商, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

组织, [ORG:二进制起源](http://www.tortall.net/projects/yasm/manual/html/manual.html)

起源、 [ORG:二进制起源](http://www.tortall.net/projects/yasm/manual/html/manual.html)

orphan-labels, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

重叠的部分, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

重载

多行宏, [重载多行宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

单行的宏, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

OWORD, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**P**

填充, [NOP填充](http://www.tortall.net/projects/yasm/manual/html/manual.html)

悖论, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

通过, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

PE、 [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

PE32 +, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

期间, [当地的标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

图片, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

PIC-specific, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

PLT, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

位置无关代码, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

预定义, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

优先级, [表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

优先考虑, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

前缀, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html), [数字常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

预处理器, [装备:定义常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

预处理程序循环, [预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

预处理器变量, [预处理器变量:%分配](http://www.tortall.net/projects/yasm/manual/html/manual.html)

原始的指令, [NASM汇编指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

过程的联系表, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

处理器模式, [指定目标处理器模式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

保护, [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

伪指令, [伪指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

公开场合, [全球:导出符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

纯二进制, [本:平面形式二进制输出](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**问**

QWORD, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**R**

rdf, [rdf:浮动动态对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RDOFF, [rdf:浮动动态对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

REL, [把相对寻址](http://www.tortall.net/projects/yasm/manual/html/manual.html), [默认值:改变汇编违约](http://www.tortall.net/projects/yasm/manual/html/manual.html)

关系运算符, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

浮动动态对象文件格式, [rdf:浮动动态对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

搬迁

PIC-specific, [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

去除环境中, [%和%推流行:创建和删除背景](http://www.tortall.net/projects/yasm/manual/html/manual.html)

重命名上下文, [% repl:重命名上下文](http://www.tortall.net/projects/yasm/manual/html/manual.html)

重复, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

重复代码, [预处理程序循环](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RESB, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RESD, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RESDQ, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

资源, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RESQ, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

休息, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

RESW, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

雷克斯, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

撕开, [把相对寻址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

旋转宏观参数, [%:旋转旋转宏观参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**年代**

搜索包含文件, [包括其他文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

节中, [节,段](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵的部分](http://www.tortall.net/projects/yasm/manual/html/manual.html), [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

win32扩展, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

部分对齐

在win32中, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

section.length, [本特殊符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

section.start, [本特殊符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

section.vstart, [本特殊符号](http://www.tortall.net/projects/yasm/manual/html/manual.html)

凹陷, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html), [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

段的地址, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html), [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

段覆盖, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

分割

x86, [分割](http://www.tortall.net/projects/yasm/manual/html/manual.html)

段, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

改变命令, [%:旋转旋转宏观参数](http://www.tortall.net/projects/yasm/manual/html/manual.html)

签署部门, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

签署了模, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

单行的宏观的存在, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

单行的宏, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

单行的宏, [正常方式:%定义](http://www.tortall.net/projects/yasm/manual/html/manual.html)

大小

的符号, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的大小, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Solaris x86, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Solaris x86 - 64, [elf64:可执行文件和可链接格式64位对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

声音, [INCBIN:包括外部的二进制文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

指定, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html), [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

方括号, [有效的地址](http://www.tortall.net/projects/yasm/manual/html/manual.html)

刺穿了, [刺穿:刺穿了调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

标准的宏, [标准宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

标准节的名字, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

标准节的名字, [标准化的部分名称](http://www.tortall.net/projects/yasm/manual/html/manual.html)

严格的, [严格:抑制优化](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符串常量, [DB和朋友:宣布初始化数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符串常量, [字符串常量](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符串处理的宏, [字符串处理在宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

字符串长度, [字符串长度:% strlen](http://www.tortall.net/projects/yasm/manual/html/manual.html)

STRUC, [STRUC ENDSTRUC:声明数据类型结构](http://www.tortall.net/projects/yasm/manual/html/manual.html), [绝对绝对:定义标签](http://www.tortall.net/projects/yasm/manual/html/manual.html)

结构化的异常, [win64结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

子, [子:%的子串](http://www.tortall.net/projects/yasm/manual/html/manual.html)

减法, [+和-:加法和减法运算符](http://www.tortall.net/projects/yasm/manual/html/manual.html)

切换部分, [改变和定义部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

符号的大小, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html)

标志的尺寸

指定, [大小:设置符号大小](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

符号类型, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

符号类型

指定, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**T**

测试

任意的数字表达式, [%如果:测试任意数值表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

上下文堆栈, [% ifctx:测试上下文堆栈](http://www.tortall.net/projects/yasm/manual/html/manual.html)

具体文本的身份, [% ifidn和% ifidni:测试具体文本的身份](http://www.tortall.net/projects/yasm/manual/html/manual.html)

多行宏观存在, [% ifmacro:测试多行宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

单行的宏观的存在, [% ifdef:测试单行的宏观的存在](http://www.tortall.net/projects/yasm/manual/html/manual.html)

令牌类型, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

次, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html), [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

令牌类型, [ifid百分比、ifnum百分比,% ifstr:测试令牌类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

双行程汇编, [关键表达式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

TWORD, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

类型

的符号, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html), [精灵扩展全球指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

类型, [类型:设置符号类型](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**U**

一元, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

一元操作符, [一元运算符:+、-、~和凹陷](http://www.tortall.net/projects/yasm/manual/html/manual.html)

未初始化, [RESB和朋友:声明未初始化的数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

UnixWare, [elf32:可执行文件和可链接32位对象文件格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

展开循环, [时报》:重复指令或数据](http://www.tortall.net/projects/yasm/manual/html/manual.html)

无符号除法, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

无符号模, [*、/、/ /、%和% %:乘法和除法](http://www.tortall.net/projects/yasm/manual/html/manual.html)

平仓数据, [win64结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

USE16, [USE16、USE32 USE64](http://www.tortall.net/projects/yasm/manual/html/manual.html)

USE32, [USE16、USE32 USE64](http://www.tortall.net/projects/yasm/manual/html/manual.html)

USE64, [USE16、USE32 USE64](http://www.tortall.net/projects/yasm/manual/html/manual.html)

用户定义的错误, [%错误:用户定义的错误报告](http://www.tortall.net/projects/yasm/manual/html/manual.html)

用户级汇编指令, [标准宏](http://www.tortall.net/projects/yasm/manual/html/manual.html)

用户级的指令, [NASM汇编指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**V**

有效字符, [NASM源代码行布局](http://www.tortall.net/projects/yasm/manual/html/manual.html)

VALIGN, [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

版本控制, [鉴别:添加文件识别](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Yasm版本号, [__YASM_MAJOR__等:Yasm版本](http://www.tortall.net/projects/yasm/manual/html/manual.html)

与32位模式下, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

对64位模式, [位](http://www.tortall.net/projects/yasm/manual/html/manual.html)

VFOLLOWS, [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Vista, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Vista x64, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Visual Studio, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html), [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Visual Studio 2005中, [VC8 cv8:CodeView调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Visual Studio 2008中, [VC8 cv8:CodeView调试格式](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Visual Studio 2010中, [VSYASM——Yasm微软Visual Studio 2010](http://www.tortall.net/projects/yasm/manual/html/manual.html)

的影响, [本指令扩展部分](http://www.tortall.net/projects/yasm/manual/html/manual.html)

VSYASM, [VSYASM——Yasm微软Visual Studio 2010](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**W**

弱, [弱势:创建软弱的象征](http://www.tortall.net/projects/yasm/manual/html/manual.html)

弱引用, [弱势:创建软弱的象征](http://www.tortall.net/projects/yasm/manual/html/manual.html)

win32中, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

节中, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Win32中, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

win32扩展, [win32扩展部分指令](http://www.tortall.net/projects/yasm/manual/html/manual.html)

win64, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Win64, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

窗户

32位, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

64位的, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Windows XP, [win32:Microsoft win32对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

Windows XP x64, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

关于, [赛格和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html), [elf64特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**X**

x32, [elfx32:精灵32位64位处理器对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x32共享库, [elfx32特殊符号和关于](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x64, [win64:PE32 +(微软win64)对象文件](http://www.tortall.net/projects/yasm/manual/html/manual.html)

结构化的异常, [win64结构化异常处理](http://www.tortall.net/projects/yasm/manual/html/manual.html)

x86, [x86体系结构](http://www.tortall.net/projects/yasm/manual/html/manual.html), [Segmentation](http://www.tortall.net/projects/yasm/manual/html/manual.html)

xdf, [xdf: Extended Dynamic Object Format](http://www.tortall.net/projects/yasm/manual/html/manual.html)

**Y**

Yasm Version, [__YASM_MAJOR__, etc: Yasm Version](http://www.tortall.net/projects/yasm/manual/html/manual.html)

SRC=http://www.tortall.net/projects/yasm/manual/html/manual.html