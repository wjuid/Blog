# Ninja 构建系统

## 概述

Ninja 是一个构建系统，与 Make 类似。作为输入，你需要描述将源文件处理为目标文件这一过程所需的命令。 Ninja 使用这些命令保持目标处于最新状态。与其它一些构建系统不同，Ninja 的主要设计目标是速度。
我在参与 Google Chrome 项目时编写了 Ninja。一开始，我将 Ninja 视作一个实验——看看能不能让 Chrome 构建的更快。为了成功地构建 Chrome，Ninja 也有其它一些设计目标：Ninja 必须易于嵌入大型构建系统。
Ninja 获得了相当的成功，逐渐取代了 Chrome 所使用的构建系统。Ninja 公开后，一些人贡献了代码，使得流行的 CMake 构建系统能够生成 Ninja 文件。现在，Ninja 也被用来开发基于 CMake 的系统，如 LLVM 和 ReactOS。其它一些拥有定制构建系统的项目，如 TextMate，直接将 Ninja 作为其构建目标。
2007 年至 2012 年间，我参与了 Chrome 项目，Ninja 则开始于 2010 年。对于像 Chrome 这样的项目（如今有约 40000 个 C++ 代码文件，生成的二进制文件约有 90 MB），有许多因素能对构建性能造成影响。我当时也接触到了其中一些，涵盖多机器分布编译到链接技巧。Ninja 起初只针对一个领域——构建的前端。也即介于开始构建到第一个编译指令开始运行这一段时间的事。要理解这一阶段为何如此重要，有必要首先理解我们是怎样看待 Chrome 本身的性能的。

## Chrome 历史小介

对 Chrome 的目标进行全面讨论超出了本文的范畴。但速度是其中非常明确的一点。性能是整个计算机科学领域共同的追求。Chrome 几乎用到了所有可能的计巧，从缓存到并行再到 just-in-time 编译。还有启动速度——在点击了那个看起来有点无聊的图标后 Chrome 要花多久在屏幕上显示出来。
为什么要关心启动速度？对于浏览器而言，迅速的启动给人一种轻便的感觉，操作 Web 上的东西如同在本地打开一个文本文件一样便捷。其实，人机交互领域已经深入地研究了延迟对情绪和思维的影响。像 Google 或 Amazon 这样的互联网公司对延迟非常关注，他们在探知延迟的影响方面也有一定的优势。他们已有的实验结果显示，仅仅是毫秒级的延迟都会对人们使用网站或在网上购买商品的频率产生可见的影响。这是微小的挫折感在潜意识中累积的结果。
Chrome 的快速启动是通过 Chrome 最早期的某个工程师设计的技巧达成的。当他们刚搭建出程序框架，仅能在屏幕上显示一个窗口时，他们就建立了基准测试，并在隋后的构建过程中对其进行追踪。正如 Brett Wilson 所说：“规定非常简单：这个测试不能变慢。”[1](http://code.google.com/p/gyp/)随着 Chrome 的代码量不断增长，维持这一基准测试需要付出额外的努力[2](https://chromium.googlesource.com/chromium/src/+/master/docs/ninja_build.md)——有时，某些计算工作会被推迟到真正需要的时候，或者，启动时所需的数据是预先计算的。但是，对性能最首要的优化，也是给我印象最深刻的，是做得更少。
我起初加入 Chrome 团队时，并未想到在构建工具方面着手。我的背景和平台选择是 Linux，而且我想要成为一名 Linux 极客。为了限定工作范围，Chrome 项目起初是 Windows 专属的。我认为我的职则是帮助完成 Windows 版。如此，我再让它在 Linux 上运行。
当我们开始向其它平台移植时，第一个困难是选择构建系统。彼时，Chrome 已经非常庞大（补充，实际上，Windows 版的 Chrome 在 2008 年发布，当时任何移植都还没有开始），启图从基于 Visual Studio 的 Windows 平台构建系统大规模地切换到某个不同的构建悉统与当前的开发进程相冲突。这如同在使用一栋建筑的同时替换其根基。
Chrome 团队的某些成员想到了一个增量的解决方案，叫 GYP[3](https://cmake.org/[CMake])。用它可以以一次一个子组件的方式生成 Chrome 已经在用的 Visual Studio 构建文件，以及其它平台会用到的构建文件。
GYP 的输入非常简单：希望得到的输出文件名称，伴有纯文本描述的源文件列表，偶尔会有定制的规则，诸如“处理每个 IDL 文件，生成一个额外的源文件”，也有一些条件动作（俐如，只在特定平台使用某文件）。GYP 会通过这一高级描述生成各平台原生的构建文件。[4](https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files)
在 Mac 上，“原生构建”意味着 Xcode 工程文件。但是，在 Linux 上，并没有一个明确的优势选项。我首先尝试了 Scon，但我发现，通过 GYP 生成的 Scons 构建会在启动前花 30 秒计算文件变更情况。我以识到 Chrome 的规模与 Linux kernel 相当，在 Linux kernel 中使用的一些技巧也应该对 Chrome 管用。我撸起袖子干了起来，使 GYP 生成普通的 Makefiles 并使用了从 Kernel 的 Makefiles 借鉴的技巧。
于是，我无意间开始陷入构建系统的天坑。导致构建耗时的原因有许多，诸如缓慢的链接器以及缺乏并行能力，我把它们研究了个遍。使用 Makefile 的方法一开始相当快。但随着我们向 Linux 移植越来越多的部份，构建中使用的文件量不断增加，这个方法也变慢了。[5]
在我进行移植时，我发现构建过程的一部分让人非常不爽：可能我修改了某个文件，运行 make，意识到漏打了一个分号，补上后再次运行 make，这之中每次等待都长到足以使我忘记我原本在做什么。回想起我们对延迟的抗争，“怎么会这么久？”，我自问，“应该没有这么多工作要做。”我开始了 Ninja，作为一个实验，看看我能简化多少。

Ninja 的设计
在高层视角下，任何构建系统主要执行三项任务。(1)加载和分析构建目标，(2)计算出达到构建目标所需的步骤，(3)执行这些步骤。
为了使第一步迅速，Ninja 在加载构建文件时只做最少量的工作。构建系统一般是面向人的，这意味着它们会提供一个方便的、高层的语法来表达构建目标。这同样意味着在实际进行构建时，构建系统必须进一步处理指令：例如，在某一时刻，Visual Studio 必须基于其构建配置具体地决定输出文件的去向，或某个文件必须由 C++ 还是 C 编译器编译。
因此，GYP 在生成 Visual Studio 时的工作实际上被控制在了将源文件列表转译到 Visual Studio 语法，剩下的工作统统交给 Visual Studio。有了 Ninja，我看到了机会，可以在 GYP 中做尽可能多的工作。在某种意义上，当 GYP 生成 Ninja 构建文件时，GYP 会进行上述的计算。GYP 随后保存这一中间数据的一个快照——以一种 Ninja 可以快速读取的格式，供后续构建使用。
Ninja 的构建文件使用的语言因此简单到了不便于人类书写的程度。没有条件语句或基于文件拓展名的规则。相反，格式仅仅是一个列表，记录确切路径所产生的确切结果。这种文件可以被快速载入，几乎不需要解释。
最小化的设计产生了极大的灵活性。因为 Ninja 缺少对高层次构建概念的了解（如输出目录或当前配置），Ninja 易于嵌入各种更大型的系统（例入，CMake，我们一会就会看到）,这些系统对于构建该如何组织往往有不同的观点。例如，Ninja 对于构建输出是与源文件放在同一目录中（有人认为不怎么“卫生”），还是放在另一个构建输出目录（有人认为这样会使别人疑惑）。在 Ninja 发布很久以后，我终于想到了正确的比喻：如果将其它构建系统视为编译器，Ninja 则如同汇编器一般。

## Ninja 做了什么

如果 Ninja 将绝大部分工作推给了构建文件生成器，那自己还有什么事呢？上述想法理论看上很不错，但真实世界的需要永远更复杂。Ninja 在开发过程中添加（也失去）了很多特性。不论何时，重要的问题都是“我们能做得更少吗？”此处概述这如何运作。
在构建规则出错时需要人去调试（构建）文件，所以 .ninja 构建文件是普通文本，与 Makefiles 类似。为了增强可读性.ninja 也支持一些抽象。

第一种抽象是“rule”，可以代表单个命令行调用，rule 定义后在不同的构建步骤间共享。这是 Ninja 语法的一个例子，声明了一条名为“compile”的 rule——这条 rule 会调用 gcc 编译器，此外还有两条 build 语句对特定文件使用了 compile。

```
rule compile
  command = gcc -Wall -c $in -o $out
build out/foo.o: compile src/foo.c
build out/bar.o: compile src/bar.c1234
```

第二种抽象是变量。在上面的例子中，那些以”<script type="math/tex" id="MathJax-Element-1">"为前缀的标识符就是变量（</script>in 和 $out）。变量即可以表示命令的输入，也可以表示命令的输出，也可以给长字符串起一个短点的名字。这里有一个compile定义，用一个变量表示编译器的标志：

```
cflags = -Wall
rule compile
  command = gcc $cflags -c $in -o $out123
```

一条规则中使用的 变量 可以在单个 build 块中被缩进表述的新定义覆盖。继续上面的例子，cflags 的值可以在单个文件处调整：

```
build out/file_with_extra_flags.o: compile src/baz.c
  cflags = -Wall -Wextra12
```

rule 与函数很像，而且变量又酷似参数。这两个简单的功能使 Ninja 的语法与编程语言过于相似，这很危险——是“不做多余事”的对立面。但他们可以减少重复字符串，这不仅对人非常有用，也有利于电脑计算，因为减少了需要分析的文本量。

构建文件，一旦完成分析，就可以描绘出一幅依赖图：最终的二进制输出依赖于一组对象文件，这组对象文件中的每个都是编译源文件的结果。特别地，这是一幅二分图（bipartite graph），“结点”（输入文件）指向“边”（构建指令），构建指令再指向“结点”（输出文件）[6]。构建过程就是遍历这幅图。

给定一个构建目标，Ninja 首先遍历这幅图以确定每条“边”上输入文件的状态：即，输入文件是否存在，以及被修改的时间。Ninja 随即计算出一份计划。计划即是为了保证最终目标处于最新状态而必须执行的“边”的集合，依据中间文件的修改时间判断。最后，执行计划，遍历图并将“边”标记为已执行，至此顺利结束。

这些功能一就位，就可以建立起基于 Chrome 的参考基线：成功完成一次构建后再次运行 Ninja 所需的时间。即载入构建文件，验正构建状态，决定是否有工作要做所需的时间。这一基准只需要不到一秒。这是我的新启动指标。但是，随着 Chrome 日渐庞大，Ninja 必须持续变快，以保证这一指标不退化。

## 优化 Ninja

Ninja 最初的实现仔细地组织了数据结构，为快速构建创造条件。但从优化的角度来说这并不是个聪明的想法。在程序完成之际，我想到，一个分析器（profiler）可以揭示哪些代码对性能产生重要影响。[7]

这些年来，分析（profiling）的结果指向过程序的不同的区域。有时是单个热点程序，可以微优化（micro-optimized）。更多时候，分析会指向一些更广泛的问题，如，除非必要，不要分配或复制内存。也存在某些情型，采用更好的表示方法或数据结构可以获得最好的效果。接下来是对 Ninja 的实现的简单表述，以及围绕 Ninja 性能的有趣故事。

### 解析(Parsing)

起初，Ninja 使用的是手写的词法分析器和递归下降分析器。我以为语法足够简单了。事实证明，对于像 Chrome 这样够大的项目[8]，仅仅是解析构建文件（拓展名以 .ninja 结尾）所消耗的时间都十分令人吃。

很快，最初用来解析单个字符的函数很快出现在分析结果中：

```
static bool IsIdentifierCharacter(char c) {
  return
    ('a' <= c && c <= 'z') ||
    ('A' <= c && c <= 'Z') ||
    // and so on...
}123456
```

一个简单的调整就可以节省 200 毫秒——用一个有 256 个条目、以输入字符为索引的查找表替换这个函数。这样一张表用 Python 代码很好生成，像这样：

```
cs = set()
for c in string.ascii_letters + string.digits + r'+,-./\_$':
    cs.add(ord(c))
for i in range(256):
    print '%d,' % (i in cs),12345
```

这个技巧使 Ninja 在相当一段时间里保证了运行速度。最终我们转移到了更正式的工具：re2c，PHP所用的词法分析器（lexer）生成工具。它可以生成更复杂的查找表，和人无法理解的代码。例如：

```
if (yych <= 'b') {
    if (yych == '`') goto yy24;
    if (yych <= 'a') goto yy21;
    // and so on...1234
```

当初采用文本格式作为输入格式是否是一个好主意？这一点依然不明确。或许最终我们会采用某种机器友好的格式作为 Ninja 输入文件的格式，这也许可以避免绝大部分解析工作。

### 规范化（Canonicalization）

Ninja 避免用字符串识别路径。取而代之的是，Ninja 将它遇到的每个路径映射到唯一的 Node 对象，在后续代码中以 Node 对象表示这一路径。复用 Node 对象保证了一个给定的路径只在硬盘上检查一次，检查的结果（例如，修改时间）可在其它代码中复用。

指向 Node 对象的指针如同这一路径的唯一标识。如果想要测试两个 Node 是否指向同一路径，比较指针就足够了，不需要进行昂贵的字符串比较。例如，在 Ninja 遍历输入文件构成的图时，会保存一个 Node 依赖栈，以检查依赖是否有环：如果 A 依赖 B，B依赖 C，而 C 又依赖 A。构建就无法进行。这个栈，代表一组文件，可以通过一个指针数组实现，可以使用指针相等性判断检查是否有重复。

为了保证指向一个文件的路径总是指向同一 Node，Ninja 必须可靠地将一个文件所有可能的名字映射到同一 Node 对象。这需要对输入文件中提到的所有文件进行规范化（canonicalization），将像 foo/../bar.h 这样的路径转换为 bar.h。最初，Ninja 只是简单地要求所有路径以规范的形式给出，但由于几个原因，这最后还是不行。一个原因是用户指定的路径（例如，在命令行输入 ninja ./bar.h ）应该能正确工作。另一个原因是变量的组合可能产生出不规范的路径。最后，gcc 给出的依赖信息可能不规范。

于是，最终，Ninja 对路径进处理。这也导致路径功能成能成为分析结果中的另一处热点。原来的实现是以清晰而不是以性能为重点编写的，所以标准的优化技术，如移除双循环（removing a double loop）或避免内存分配，作用显著。

### 设计目标

这里是Ninja的设计目标：
\* 非常快（即瞬间）增量构建，即使是非常大的项目。
\* 代码如何构建的策略非常少。关于代码如何构建不同的项目和高级构建系统有不同的意见；例如，构建目标是应该与代码放在一起还是放在单独的目录里？是否有为项目构建一个可分发的“包”的规则？规避这些策略，而不是选择，否则只会变得复杂。
\* 获取正确的依赖关系，在某些特定的情况下Makefiles很难获取正确的依赖关系（例如：输出隐式依赖于命令行生成，构建c代码你需要使用gcc的`-M`标志去生成其头文件依赖）
\* 当目标和遍历冲突时我们选择速度.

一些明确的 *non-goals*:

- 方便人工编写构建文件的语法。你应该使用其它程序来生成你的构建文件，这是我们可以回避许多策略决定。
- built-in rules. _Out of the box, Ninja has no rules for
  e.g. compiling C code._
- build-time customization of the build. _Options belong in
  the program that generates the ninja files_.
- build-time decision-making ability such as conditionals or search
  paths. *Making decisions is slow.*

重申一下，Ninja比其它构建系统要快，因为它是异常的简单。在你为你的项目编写`.ninja`文件时你必须告诉Ninja要它做什么。

### 与Make比较

Ninja的定位非常清晰，就是达到更快的构建速度。

ninja的设计是对于make的缺陷的考虑，认为make有下面几点造成编译速度过慢：

- 隐式规则，make包含很多默认
- 变量计算，比如编译参与应该如何计算出来
- 依赖对象计算

ninja认为描述文件应该是这样的:

- 依赖必须显式写明(为了方便可以产生依赖描述文件)
- 没有任何变量计算
- 没有默认规则，没有任何默认值
  针对这点所以基本上可以认为ninja就是make的最最精简版。

ninja相对于make增加了下面这些功能：

- 如果构建命令发生变化，那么这个构建也会重新执行。
- 所依赖的目录在构建之前都已经创建了，如果不是这样的话，我们执行命令之前都要去生成目录。
- 每条构建规则，除了执行命令之外，还允许有一个描述，真正执行打印这个描述而不是实际执行命令。
- 每条规则的输出都是buffered的，也就是说并行编译，输入内容不会被搅和在一起。
  构建工具太多了，我个人觉得make主要偏大众化一点，可以进行各种隐式推导，比较灵活，每一条命令执行都有输出。

而Ninja主要的设计目的是为了像chromium这种大型项目，能够显著的提高编译速度，一方面它去掉了各种计算和推导，把一些耗时的需要计算的东西去掉了，只留下简单重要的部分，所以如果自己去写build.ninja文件的话比较繁琐，所以都是依赖于其它构建工具生成的，另一方面它每次输出只输出一个描述，而不是真正的命令执行输出，真正的命令执行再后台运行，只有警告和报错信息才会显示出来，这也提高了它的速度。

Make vs Ninja Performance Comparison这篇文章对Make接Ninja进行测试对比。

## 在你的项目中使用ninja

Ninja 目前在Windows和类Unix系统都支持，虽然大部分测试都是在linux上完成的（并且在Linux上性能也最好），不过在MAC OS X和FreeBSD 上也都能很好的工作。
如果你的项目很小，Ninja的速度优势可能不那么明显（然而，即使是小项目，Ninja的极其简洁的语法和极其简单的构建规则，也使你的项目能够更加快速的构建）。换一句话说，如果你对你的项目编辑-编译的循环时间感到满意，那么Ninja可能对你不会有太多的帮助。
还有许多其它的构建系统，比Ninja使用更加友好，功能也更加强大。作者觉得Ninja的设计受到了tup构建系统设计的影响，并认为重做的设计非常聪明。
Ninja的好处是可以将它与智能的元构建系统结合起来使用。
[gyp](http://code.google.com/p/gyp/)用于生成Google Chrome和相关项目（v8，node.js）的元构建文件建立系统，已被GN取代。 gyp可以为Chrome支持的所有平台生成Ninja文件。 有关详细信息，请参阅[Chromium Ninja文档](https://chromium.googlesource.com/chromium/src/+/master/docs/ninja_build.md)。
[CMake](https://cmake.org/[CMake])一个广泛使用的元构建系统，在Linux上CMake 2.8.8版本可以生成Ninja文件。 较新版本的CMake支持在Windows和Mac OS X上生成Ninja文件。
[其他](https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files):Ninja应该完善其他元构建软件的支持，例如premake。 如果你在做这项工作，请让我们知道！

### 运行Ninja

运行`ninja`。 默认情况下，它在当前目录中查找名为build.ninja的文件
并构建所有过期目标。 您可以在命令行参数中指定要构建的目标（文件）。

还有一个特殊的语法,`目标^`指定的目标作为第一个输出（如果存在）。 例如，如果您指定目标
`foo.c ^`, 那么foo.o将首先被构建（假设你的构建文件中存在这些目标）。

`ninja -h`打印帮助。 许多Ninja的flags与Make是匹配的; 例如ninja -C build -j 20改成构建目录并并行运行20个构建命令。 （注意
Ninja默认情况下以并行方式运行命令，所以通常你不需要传递-j。）
Ninja 默认基于系统中可用的 CPU 数量以并发方式执行指令。因为同时运行的命令们的输出可能混淆，Ninja 会在一个命令完成前缓存其输出。从结果看，如同命令是串行的。
这种对命令输出的控制使得 Ninja 可以小心控制总的输出。在构建过程中 Ninja 显示一行表示状态；如果构建顺利完成，Ninja 的全部输出就只有一行。这不会使 Ninja 运行得更快，但可以使人感觉 Ninja 很快，这几乎与在真实速度上的目标一样重要。

### 环境变量

Ninja支持用一个环境变量来控制其行为：
`NINJA STATUS`，在运行规则之前会打印进度状态。
下面是几个可用的占位符：

`%s`:: The number of started edges.
`%t`:: The total number of edges that must be run to complete the build.
`%p`:: The percentage of started edges.
`%r`:: The number of currently running edges.
`%u`:: The number of remaining edges to start.
`%f`:: The number of finished edges.
`%o`:: Overall rate of finished edges per second
`%c`:: Current rate of finished edges per second (average over builds
specified by `-j` or its default)
`%e`:: Elapsed time in seconds. *(Available since Ninja 1.2.)*
`%%`:: A plain `%` character.

默认进度状态为 `"[%f/%t] "` (
注意结尾空格以与构建规则分开). 另一个可能的进度状态的例子如下： `"[%u/%r/%f] "`.

### 扩展工具

在Ninja的开发过程中，命令行里使用`-t`可以运行一些非常有用的工具，目前有以下一些工具可以使用：

`query`:: dump指定target的输入和输出.

`browse`:: 在Web浏览器中浏览依赖关系图。 单击文件将焦点切到该文件上，会显示输入和输出。 这个
功能需要Python安装。 默认使用端口8000并打开Web浏览器。 可以按照如下方式修改：

```
ninja -t browse --port=8000 --no-browser mytarget1
```

`graph`::以自动图形布局工具`graphviz`的语法输出一个文件。 使用方式如下：

```
ninja -t graph mytarget | dot -Tpng -ograph.png1
```

+
在Ninja源代码树中，运行“ninja graph.png”命令将为Ninja本身生成一张图。 如果没有指定目标则将为
all目标生成。

`targets`:: output a list of targets either by rule or by depth. If used
like +ninja -t targets rule *name*+ it prints the list of targets
using the given rule to be built. If no rule is given, it prints the source
files (the leaves of the graph). If used like

```
+ninja -t targets depth _digit_+ it1
```

prints the list of targets in a depth-first manner starting by the root
targets (the ones with no outputs). Indentation is used to mark dependencies.
If the depth is zero it prints all targets. If no arguments are provided

```
+ninja -t targets depth 1+ is assumed. In this mode targets may be listed1
```

several times. If used like this +ninja -t targets all+ it
prints all the targets available without indentation and it is faster
than the *depth* mode.

`commands`:: given a list of targets, print a list of commands which, if
executed in order, may be used to rebuild those targets, assuming that all
output files are out of date.

`clean`:: remove built files. By default it removes all built files
except for those created by the generator. Adding the `-g` flag also
removes built files created by the generator (see <

## 编写你自己的Ninja文件

### 概述

Ninja和Make非常相似。他执行一个文件之间的依赖图，通过检测文件修改时间,运行必要的命令来更新你的构建目标。
一个构建文件（默认文件名为：build.ninja）提供一个rule（规则）表——长命令的简短名称，和运行编译器的方式。同时，附带提供build（构建）语句列表，表明通过rule如何构建文件——哪条规则应用于哪个输入产生哪一个输出。

从概念上讲，build语句描述项目的依赖图；而rule语句描述当给定一个图的一条边时，如何生成文件。

### 语法例子

这是一个用于验证绝大部分语法的.ninja文件，将作为后续描述相关的示例。具体内容，如下：

```
cflags = -Wall

rule cc
  command = gcc $cflags -c $in -o $out

build foo.o: cc foo.c123456
```

### 变量

ninja支持为字符串声明简短可读的名字。一个声明的语法，如下：

```
cflags = -g1
```

可以在=右边使用，并通过$进行引用（类似shell和perl的语法）。具体形式，如下：

```
rule cc
  command = gcc $cflags -c $in -o $out12
```

变量还可以用${in}($和成对的大括号)来引用。
当给定变量的值不能被修改，只能覆盖（shadowed）时,变量更恰当的叫法是绑定（”bindings”）。

### 规则

规则为命令行声明一个简短的名称。他们由关键字rule和一个规则名称打头的行开始，然后紧跟着一组带缩进格式的 variable = value行组成。
以上示例中声明了一个名为cc的rule，连同一个待运行的命令。在rule（规则）上下文中，command变量用于定义待执行的命令，$in展开（expands）为输入文件列表（foo.c）,而$out为命令的输出文件列表（foo.o）。 [参考手册](http://martine.github.io/ninja/manual.html#ref_rule)中罗列了所有特殊的变量。

### Build 语句

build语句声明输入和输出文件之间的一个关系。构建语句由关键字build开头，格式为

```
build outputs: rulename inputs1
```

这样的一个声明，所有的输出文件来源于输入文件。当缺输出文件或输入文件变更时，Ninja将会运行此规则来重新生成输出。
以上的简单示例，描述了使用cc规则如何构建foo.o文件。
在build block范围内（包括相关规则的执行），变量$in表示输入列表，$out表示输出列表。
一个构建语句，可以和rule一样，紧跟一组带缩进格式的key = value对。当在命令中变量执行时，这些变量将覆盖（shadow）任何变量。比如：

```
cflags = -Wall -Werror
rule cc
  command = gcc $cflags -c $in -o $out

# 如果没有指定，build的输出将是$cflags
build foo.o: cc foo.c

# 但是，你可以在特殊的build中覆盖cflags这样的变量
build special.o: cc special.c
  cflags = -Wall

# cflags变量仅仅覆盖了special.o的范围
# 以下的子序列build行得到的是外部的(原始的)cflags
build bar.o: cc bar.c1234567891011121314
```

如果你要从build语句传递更多的信息到rule规则（例如，如果规则需要知道”第一输入文件的扩展名”）,那么请通过扩展变量传递，就像`cflags`一样。

如果顶级Ninja文件使用build指定了任何输出，并且它又过期了，那么再为构建用户目标之前会先构建顶级文件里的目标。

### 根据代码生成Ninja文件

Ninja发行包中的misc/ninja_syntax.py是一个很小的python模块，用于生成Ninja文件。你可以使用python，执行如

```
ninja.rule(name='foo', command='bar',depfile='$out.d')1
```

的调用，生成合适的语法。如果这样还不错，可以将其整合到你的项目中。

## 更多细节

### `phony` 规则

可以使用phony创建其它target（编译构建目标）的别名。比如：

```
build foo: phony some/file/in/a/faraway/subdir/foo1
```

这样使得ninja foo构建更长的路径。从语义上讲，phony规则等同于一个没有做任何操作的普通规则，但是phony规则通过特殊的方式进行处理，这样当其运行时不会被打印，记日志，也不作为构建过程中打印出来的命令计数。
还可以用phony为构建时可能还不存在的文件创建dummy目标。

### 默认目标

默认情况下，如果没有在命令行中指定target，那么Ninja将构建任何地方没有作为输入命名的每一个输出。可以通过default目标语句来重写这个行为。一个default语句，让Ninja构建一个给定的输出文件子集，如果命令行中没有指定构建目标。
默认目标语句，由关键字default打头，并且采用default targets的格式。一个default目标语句必须出现在，声明这个目标作为一个输出文件的构建语句之后。他们是累积的（cumulative），所以可以使用多个default语句来扩展默认目标列表。比如：

```
default foo bar
default baz12
```

This causes Ninja to build the `foo`, `bar` and `baz` targets by
default.

### Ninja 日志

Ninja构建日志保存在构建过程的根目录或.ninja文件中builddir变量对应的目录的.ninja_log文件中。
一般而言，像上面这样的微优化不如改变算法或处理方式的结构性优化有效。Ninja 的构建日志就是这样一个例子。
Linux kernel 构建系统的一部分会追踪用于生成输出的命令。考虑一个启发性的例子：你将输入文件 foo.c 编译为输出文件 foo.o，随后修改了构建文件导致应该用不同的编译选项重新编译 foo.c。从构建系统的角度看，为了感知需要构建，必须要么注意到 foo.o 依赖于构建文件（构建文件依赖于项目的组织，这也许意味着对构建文件的修改将导致整个项目的重新构建），或记录生成每个输出的命令，在每次构建时进行比较。
kernel（以及 Chrome Makefiles 和 Ninja）采用后一种方法。在构建时，Ninja 写下一份构建日志，记录生成每个输出的完整命令。[9]在后续构建中，Ninja 载入之前的构建日志，通过比较当前命令与构建日志中的命令来发现变更。就像加载构建文件或路径规范化，这成为了分析结果中的又一处热点。
在进行了一些小优化后，Nico Weber，一个对 Ninja 贡献了很多代码的人，实现一种新的构建日志格式。比起通常很长且需要大量时间进行解析的记录命令，Ninja 取而代之以命令的哈希（hash）。在后续构建中，Ninja 比较将要执行的明令的哈希与记录中的哈希。如果两者不同，则可以确定输出已过期。这一方法很成功。使用哈希急剧降低了构建日志的大小——在 Mac OX X 上，从 200MB 降到 2MB——并使加载速度快了 20 倍。

### 版本兼容性

*Available since Ninja 1.2.*

Ninja version labels follow the standard major.minor.patch format,
where the major version is increased on backwards-incompatible
syntax/behavioral changes and the minor version is increased on new
behaviors. Your `build.ninja` may declare a 变量 named
`ninja_required_version` that asserts the minimum Ninja version
required to use the generated file. For example,

```
ninja_required_version = 1.11
```

declares that the build file relies on some feature that was
introduced in Ninja 1.1 (perhaps the `pool` syntax), and that
Ninja 1.1 or greater must be used to build. Unlike other Ninja
变量s, this version requirement is checked immediately when
the 变量 is encountered in parsing, so it’s best to put it
at the top of the build file.

Ninja always warns if the major versions of Ninja and the
`ninja_required_version` don’t match; a major version change hasn’t
come up yet so it’s difficult to predict what behavior might be
required.

### 文件依赖

还有另一种元数据（metadata）必需跨构建保存用。为了正确构建 C/C++ 代码，一个构建系统必需能感知头文件间的依赖。假定 foo.c 包含一行 #inclue “bar.h” 。而 bar.h 自身又包含一行 #include “bar.h”。所有的三个文件都会影响后续编译。例如，baz.h 的改变也会触发 foo.o 的重新构建。

一些构建系统使用一个“头文件扫描器”在构建时提取这部分依赖信息。但这个方法太慢，而且很难精确处理有 #ifdef 指令出现的情形。另一种选择是要求构建文件正确地报告所有依赖，包括头文件的依赖，但这对开发人员来说十分笨重：每次你添加或删除 #include 语句时，都需要修改或重新生成构建文件。

一个有用的方法依赖于这样的事实：在编译时，gcc （以及微软的 Visual Studio）可以给出在构建输出时用到了哪些头文件。这份信息，如同用于生成输出的信息，可以被构建系统记录和加载。由此，依赖可以被精确追踪。在第一次编译时，因为还未有输出，所有文件都会被编译，故不需头文件依赖。第一次编译后，对于被某个输出用到的任何文件如果发生更改（包括增加或删除额外的依赖），就会导致重新构建。这保证了依赖信息的更新。

在编译时，gcc 以 Makefile 的格式记下头文件依赖。Ninja 包括一个解析器处理这一Makefile 语法（的简化子集），并在下一次构建时载入这份依赖信息。在 Chrome 的最近一次构建，gcc 产生了共 90MB 的 Makefile，全部带有必须规范化的引用路径。

就像其它解析过程，通过使用 re2c 及尽可能地避免复制可以使性能有所提升，但就像 GYP 项目，这一解析工作可以不在关键时间路径上完成。近期，我们在 Ninja 上的工作（在写作本文时，这一工能已经完成，但还未发布）是让这一过程发生的早一些。

一旦 Ninja 开始执行构建指令，所有影响性能的工作都已完成，Ninja 在等待它启动的命令完成的过程中近乎闲置。在处理头文件依赖的新方法中，Ninja 利用这段时间处理 gcc 给出的 Makefile ，规范化路径，将依赖处理为一种可以快速识别的二进制格式。在下一次构建中，Ninja 只需要加载这一文件。改进非常剧烈，特别是在 Windows 上。（本章稍后讨论这个）

“依赖日志”需要储存上千条路径及路径间的依赖。载入日志和追加日志都必须迅速。追加日志操作应该是安全的，即使被打断，比如构建被取消。

在考虑了一些类似于数据库的方案后，我最终想到了一个简单的实现：文件由记录的序列组成，而记录要么是一个路径，要么是一个依赖列表。每个写入文件的路径都被赋于了一个整数序列号。故而依赖就是一列整数。为了向文件添加依赖，Ninja 首先记录下还没有序列号的路径，然后用这些序列号记录依赖。在后续的构建载入这一文件时，Ninja 可以简单地使用一个数组将序列号映射到对应的 Node 指针。

### C/C++ 头依赖

Ninja目前支持depfile和deps模式的C/C++头文件依赖生成。 如

```
rule cc
  depfile = $out.d
  command = gcc -MMD -MF $out.d [other gcc flags here]123
```

-MMD标识告诉gcc要生成头文件依赖，-MF则说明要写到哪里。
deps按照编译器的名词来管理。具体如下：（针对微软的VC：msvc）

```
rule cc
  deps = msvc
  command = cl /showIncludes -c $in /Fo$out123
```

### 工作池

为了支持并发作业，Ninja还支持pool的机制（和用-j并行模式一样）。此处不详细描述了。具体示例，如下：

```
# No more than 4 links at a time.
pool link_pool
  depth = 4

# No more than 1 heavy object at a time.
pool heavy_object_pool
  depth = 1

rule link
  ...
  pool = link_pool

rule cc
  ...

# The link_pool is used here. Only 4 links will run concurrently.
build foo.exe: link input.obj

# A build statement can be exempted from its rule's pool by setting an
# empty pool. This effectively puts the build statement back into the default
# pool, which has infinite depth.
build other.exe: link input.obj
  pool =

# A build statement can specify a pool directly.
# Only one of these builds will run at a time.
build heavy_object1.obj: cc heavy_obj1.cc
  pool = heavy_object_pool
build heavy_object2.obj: cc heavy_obj2.cc
  pool = heavy_object_pool
12345678910111213141516171819202122232425262728293031
```

#### `console` 池

这里有一个名为`console`深度为1的预定义池，池中的任何任务都可以直接访问标准输入、输出和错误流并提供给Ninja，通常是连接到用户的控制台。这对于交互式任务或运行时间较长的任务比较有用。可以在控制台上更新状态（例如测试套件）。
当’console’池中的任务正在运行时，Ninja的正常输出（如进度状态和并发任务的输出）将被缓冲起来直到控制台任务运行完成。

## Ninja 文件参考

一个Ninja文件是一系列声明，声明可以是下列之一：

1. 规则声明，以*rulename*开头,然后是一些列的变量的定义；
2. A build edge, which looks like +build *output1* *output2*:
   *rulename* *input1* *input2*+. +
   Implicit dependencies may be tacked on the end with +|
   *dependency1* *dependency2*+. +
   Order-only dependencies may be tacked on the end with +||
   *dependency1* *dependency2*+. (See <

### 语法

Ninja仅支持ASCII字符集。
注释以为#开始一直到行末。

新行是很重要的。像build foo bar的语句，是一堆空格分割分词（token），到换行结束。一个分词中的新行和空格必须进行转译。目前只有一个转译字符，$，其具有以下行为：

```
$ followed by a newline1
```

转译换行，让当前行一直扩展到下一行。

```
$ followed by text1
```

这是， 变量引用。

```
${varname}1
```

这是，另$varname的另一种语法。

```
$ followed by space1
```

这表示一个空格。(仅在path列表中，需要用空格分割文件名)

```
$:1
```

这表示一个冒号。（仅在build行中需要。此时冒号终止输出列表）

```
$$1
```

这个表示，字面值的$。

一个build或default语句，最先被解析，作为一个空格分割的文件名列表，然后每一个name都被展开。也就是说，变量中的一个空格将作为被展开后文件名中的一个空格。

```
spaced = foo bar
build $spaced/baz other$ file: ...12
```

在一个name = value语句中，value前的空白都会被去掉。出现跨行时，后续行起始的空白也会被去掉。

```
two_words_with_one_space = foo $
    bar
one_word_with_no_space = foo$
    bar1234
```

其他的空白，仅位于行开始处的很重要。如果一行的缩进比前一行多，那么被人为是其父边界的一部分。如果缩进比前一行少，那他就关闭前一个边界。

### 顶层变量

Ninja支持的顶层变量有builddir和ninja_required_version。具体说明，如下：

- builddir: 构建的一些输出文件的存放目录。
- ninja_required_version:指定满足构建需求的最小Ninja版本。

### Rule 变量

一个rule块包含一个key = value的列表声明，这直接影响规则的处理。以下是一些特殊的key：

- command (required)：
  待执行的命令。这个字符串 $variables被展开之后，被直接传递给sh -c，不经过Ninja翻译。每一个规则只能包含一条command声明。如果有多条命令，需要使用&&符号进行链接。
- depfile： 指向一个可选的Makefile，其中包含额外的隐式依赖。这个明确的为了支持C/C++的头文件依赖。
- deps: （1.3版本开始支持）如果存在，必须是gcc或msvc，来指定特殊的依赖。产生的数据库保存在builddir指定目录.ninja_deps文件中。
  msvc_deps_prefix： （1.5版本开始支持）定义必须从msvc的/showIncludes输出中去掉的字符串。仅在deps = msvc而且使用非英语的Visual Studio版本时使用。
- description： 命令的简短描述，作为命令运行时更好的打印输出。打印整行还是对应的描述，由-v标记控制。如果一个命令执行失败，整个命令行总是在命令输出之前打印。
- generator： 如果存在，指明这条规则是用来重复调用生成器程序。通过两种特殊的方式，处理使用生成器规则构建文件：首先，如果命令行修改了，他们不会重新构建；其次，默认不会被清除。
  in： 空格分割的文件列表被作为一个输入传递给引用此rule的构建行，如果出现在命令中需要使用${in}(shell-quoted)。（提供$in仅仅为了图个方便，如果你需要文件列表的子集或变种，请构建一个新的变量，然后传递新的变量。）
- in_newline： 和$in一样，只是分割符为换行而不是空格。（仅为了和$rspfile_content一起使用，解决MSVC - linker使用固定大小的缓冲区处理输入，而造成的一个bug。）
- out： 空格分割的文件列表被作为一个输出传递给引用此rule的构建行，如果出现在命令中需要使用${out}；
- restat: 如果存在，引发Ninja在命令行执行完之后，重新统计命令的输出。
- rspfile, rspfile_content： 如果存在（两个同时），Ninja将为给定命令提供一个响应文件，比如，在调用命令之前将选定的字符串(rspfile_content)写到给定的文件（rspfile），命令执行成功之后阐述文件。
  这个在Windows系统非常有用，因为此时命令行的最大长度非常受限，必须使用响应文件替代。具体使用方式，如下：

```
rule link
  command = link.exe /OUT$out [usual link flags here] @$out.rsp
  rspfile = $out.rsp
  rspfile_content = $in

build myapp.exe: link a.obj b.obj [possibly many other .obj files]123456
```

#### `command`变量的解释

在Unixes和Windows上命令行的行为是不同的。

在Unixes上，命令是参数数组。 Ninja命令变量直接传递给`sh -c`，然后负责
将该字符串解释为argv数组。 因此引用规则由shell决定，你可以使用所有正常的shell
运算符，如`&&`链接多个命令，或`VAR = value cmd` 来设置环境变量。

在Windows上，命令是字符串，因此Ninja直接将`command`字符串
传递给`CreateProcess`。 （在常见情况下编译器简单执行这意味着有更少的开销。）因此引用规则由被调用的程序确定，在Windows上通常由C库提供。 如果你需要shell解释命令（如使用`&&`来链接多个命令），使命令执行Windows shell前缀命令与`cmd / c`。

### 构建输出

有两种稍微有点区别的输出：

1. 显示输出, 在build行会列出来，在rule规则中可以通过$out变量访问。
   这是标准的输出使用形式，例如一个编译命令的目标文件。
2. 隐式输出, 在build行其语法格式如下，在build行的`:`前*out1* *out2*（在Ninja1.7版本开始支持）.语义与显式输出相同，唯一的区别是隐式输出不会出现在`$out`变量里。这是为了表示在命令行中没有指定输出的命令。

### 构建依赖

Ninja目前支持3种类型的构建依赖。分别是：

- 罗列在build行中的显式的依赖。他们可以作为规则中的$in变量。这是标准依赖格式。
- 从depfile属性或构建语句末尾的| dep1 dep2语法获得的隐式依赖。这个和显式依赖一样，但是不能在$in中使用（不可见）。
- 通过构建行末|| dep1 dep2语法表示的次序唯一（Order-only）依赖。他们过期的时候，输出不会被重新构建，直到他们被重建，但仅修改这种依赖不会引发输出重建。

### 变量展开

变量在路径（在build或default语句）和name = value右边被展开。
当name = value语句被执行，右手边的被立即展开（根据以下的规则），从此$name扩展为被展开结果的静态字符串。永远也不会存在，你将需要使用双转译（"double-escape"）来保护一 个值被第二次展开。
所有变量在解析过程，遇到的时候立即被展开，除了一个非常重要的例外：rule块中的变量仅在规则被使用的时候才被展开，而不是声明的时候。在以下的示例中，demo打印出"this is a demo of bar"而不是"this is a demo of $foo"。

```
rule demo
  command = echo "this is a demo of $foo"

build out: demo
  foo = bar12345
```

### 评估和边界

顶层（Top-level）变量声明的边界，是相关的文件。
subninja关键自，用于包含另一个.ninja文件，其表示新的边界。被包含的subninja文件可以使用父文件中的变量，在文件边界中覆盖他们的值，但是这不影响父文件中变量的值。
同时，可以用#include语句在当前边界内，引入另一个.ninja文件。这个有点像C中的#include语句。
构建块中声明的变量的边界，就是其所属的块。一个构建块中展开的变量的所有查询次序为：

- 特殊内建变量($in,$out)；
- build/rule块中构建层的变量；
- 构建行所在文件中的文件层变量（File-level）；
- 使用subninja关键字引入那个文件的（父）文件中的变量。