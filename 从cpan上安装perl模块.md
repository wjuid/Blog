# 从cpan上安装perl模块

转载[dietan8353](https://me.csdn.net/dietan8353) 发布于2015-08-22 15:26:00 阅读数 36 收藏

展开

CPAN是perl的一个第三方源码模块库，里面有上百万的perl模块，用来支撑perl强大的功能，从生物工程到天文计算，从宏观世界到原子力学，无所不有。为了很方便的安装perl模块，有人做了一个CPAN模块，用cpan命令来管理和安装CPAN网站上的所有perl模块。


-------------------CPAN




m //这个是一个模式，用来查找perl模块，有点像perl正则里面的m//，用于匹配(match)，但这里m与//之间多了一个空格，且这里的m指'modules'，意思是根据模块名称查找，//内可以使用正则； 同理a(authors)、b(bundles)、d(distributions)和i(in all)都有这种模式。




CPAN配置
CPAN安装是网络安装，如果没有网络，使用cpan命令是无法安装的，所以CPAN安装的速度是与网络有直接关系；当然我们可以选择一个快速的镜像站点来安装CPAN模块，那如何配置呢？

如果是第一次使用CPAN，那么执行cpan命令的时候，cpan命令会给出配置的友好提示，不过很多人都对这个友好提示的繁冗程度显得很不耐烦，不过新版本的CPAN模块已经改善了这一缺点；如果不是第一次使用CPAN，而以前别人配置的CPAN镜像站点出了问题不能下载，或镜像站点太慢等等修改一下配置信息，可以重新配置CPAN：
CPAN>o conf init

若不是root用户，使用cpan命令或perl -MCPAN -e shell也可以进行配置后安装：
首先，配置好CPAN配置，这个配置可以使用非root用户配置好，配置好的内容一般在$HOME/.cpan/CPAN/MyConfig.pm文件内；

配置文件MyConfig.pm中配置信息中确定有 'makepl_arg'=>q[PREFIX=~/perl] 这一行，~/perl为你当前用户有权限读、写和执行的目录；

最后，设置 PERL5LIB 环境变量，把 ~/perl 下的相关目录添加进 PERL5LIB。以 bash shell 为例，在 ~/.bash_profile 里添加如下即可：
export PERL5LIB=~/perl/lib:\
~/perl/lib/perl5/5.10.1/i386-linux-thread-multi:\
~/perl/lib/perl5/5.10.1:\
~/perl/lib/perl5/site_perl/5.10.1/i386-linux-thread-multi:\
~/perl/lib/perl5/site_perl/5.10.1:\
${PERL5LIB}

模块检测
运行
$perl -e 'use Module'

如果没有任何输出，则表示模块 Module 成功安装

打印模块版本
$perl -MModule -e 'print $Module::VERSION;'

注意事项：
使用 CPAN 安装模块有时候会 make test 一步失败。可到 $HOME/.cpan/build 的相应目录下直接 make install；
第一次安装 CPAN 时，可以先安装：
install Bundle::CPAN
或
install Bundle::CPANxxl?

这样以后的安装包安装就会少需要的依赖，建议安装。


\-------------------
CPANPLUS

默认的cpan命令安装时，如果依赖到其它包时，cpan不能自己解决，而需要手动去解它们之间的依赖关系；在初次使用cpan时，有回答很多问题。如果使用CPANPLUS的话，它能自动给依赖的模块安装好，在 Perl5.10 中现在默认有 CPANPLUS Shell。

装好CPANPLUS模块后，在终端里输入：cpanp 即可进入CPANPLUS环境。下载后就能直接使用，不需要任何其它的模块(当内 Perl 本身的 Module::Build，ExtUtils::MakeMaker 和 C Compiler 还是要，这个是系统就有的)，有没有 Root 权限都不重要，非 root 会自动安装到当前用户的目录下。

包安装时取消安装测试(可选)
如果觉得每次测试太花时间,可以将测试取消：
\# 取消安装过程中的测试：s conf skiptest 1

设置镜象：s reconfigure

选择镜象地址
 选择7 Select mirrors
 选择 No 
 选择 1 镜象
 选择 3 Asia
 选择 9 China
 选择镜象地址,也一样按上面的数字,最后面一个是退出这个,记的退出时保存.
 选择 9 Save and exit

\# 取消提问回答是否按Y
s conf prereqs 1
s save #记的存一下

使用参考总结
\1. CPANPLUS 中安装模块,按i:CPAN Terminal> i Bundle::CPAN

\2. CPANPLUS 中删除模块,按u:CPAN Terminal> u YAML

\3. CPANPLUS 中查找模块,按m:CPAN Terminal>m Smart::Comments

\4. CPANPLUS 中查找作者的模块,按a:CPAN Terminal>a kai

\5. CPANPLUS 中更新所有有新版本的模块,按下o:
CPAN Terminal> o
aliased            0.30  0.31  O/OV/OVID/aliased-0.31.tar.gz
Any::Moose           0.13  0.21  S/SA/SARTAK/Any-Moose-0.21.tar.gz
AnyEvent            7.02  7.04  M/ML/MLEHMANN/AnyEvent-7.04.tar.gz
AnyEvent::HTTP         2.14  2.15  M/ML/MLEHMANN/AnyEvent-HTTP-2.15.tar.gz
Apache2::Cookie        2.12  2.13  I/IS/ISAAC/libapreq2-2.13.tar.gz
......

\6. 自我更新: CPAN Terminal>s selfupdate all
按x来更新包的索引缓存。


\-------------------
CPANM

'cpanm'是一个新近出现的能与'cpanp'不相上下的包安装工具，这个工具能克服cpan的一系列缺点。
cpan App::cpanminus

cpanm的安装
1)、单文件安装
\# wget https://raw.github.com/miyagawa/cpanminus/master/cpanm -O cpanm

2)、完整版本安装
\# wget https://raw.github.com/miyagawa/cpanminus/master/cpanm -O /usr/local/bin/cpanm
\# perl cpanm --self-upgrade --mirror http://mirrors.163.com/cpan

是在下载 cpanm 以后，直接用他来安装更新它自己，对应的模块名为：App::cpanminus。

cpanm的使用
使用方法很简单，命令行后直接跟包模块名即可，如：cpanm YAML::XS

这样它会从cpan镜像站上下载对应的tar包，后解压安装。至于它从哪里取得包，依据于当时的源设置，可以从本地磁盘上加载；它后面也可以跟包的url，如：'http://search.cpan.org/CPAN/authors/id/I/IN/INGY/YAML-LibYAML-0.41.tar.gz'，它会下载后自动安装，如何对应的包有依赖，它会自动解决。

可以对它进行重新配置时指定相关源，相对于修改其配置文件，具体可参考：使用minicpan创建本地CPAN http://www.freeoa.net/development/perl/diy-cpan-by-minicpan_1738.html

也可以在当前shell环境里，指定别名：
alias cpanm='cpanm --mirror http://mirrors.163.com/cpan'
alias cpanm='cpanm --mirror /data/cpan/ --mirror-only'

Then to install any module from CPAN
cpanm Module::Name

The latest and greatest answer to this question is to use cpanm instead (also referred to as App::cpanminus or cpanminus)!

DESCRIPTION
cpanminus is a script to get, unpack, build and install modules from CPAN and does nothing else.

It's dependency free (can bootstrap itself), requires zero configuration, and stands alone. When running, it requires only 10MB of RAM.

To bootstrap install it:
curl -L http://cpanmin.us | perl - --sudo App::cpanminus

or if you are using perlbrew simply
perlbrew install-cpanm

From then on install modules by executing (as root if necessary)
cpanm Foo::Bar

\-------------------
三者对比总结
CPAN是年代就为久远的，其成熟性和稳定性是不容置疑的，但同时它也有自身的缺点。

CPANPLUS是新出不久的，其特点是现代、智能、好用，与CPAN相同的是，它们都出现在'Core modules'中，估计以后打算用它做为CPAN的替代者。

cpanm也是初生牛犊，它具有与cpanplus相似的优点，但它更小巧、灵活，同时又不失其个性。

因此后两者是今后的主流，如何选择就要看个人喜好了。

\-------------------
pm模块管理
1、删除模块

上面介绍了多种安装perl模块的方法，如果我想删除我机器上的模块，应该怎么操作呢，下面介绍一个新模块：'App::pmuninstall'
安装好后，会在PATH中生成一个命令行工具：pm-uninstall，使用它便可删除相应的模块。

使用很简单   $ pm-uninstall YAML::XS  # 后跟模块的名字，任何你要删除的模块的名字都能加在其后

2、检查所有已安装的模块和版本
用于列出和检查本地已经安装的模块，查看具体版本信息，使用'App::cpanoutdated' 这个可以来实现：有那些可以更新，并会列出来，可以使用 cpanm或cpanp 来进行升级。
$ cpan-outdated --verbose --mirror file:///data/cpan/

我们可以将其结果作为参数传给 cpanm 来安装：
\# cpan-outdated | cpanm
\# cpan-outdated | xargs cpan -i

更多关于查看系统中perl模块安装情况时，可参考文章： 查看Perl模块安装路径的"查看系统中已经安装的Perl模块"段落。

3、查看具体模块的相关信息(安装位置、版本等) 
App::pmodinfo















原文网址：http://blog.chinaunix.net/uid-20367477-id-4249130.html

转载于:https://www.cnblogs.com/v-BigdoG-v/p/7398633.html