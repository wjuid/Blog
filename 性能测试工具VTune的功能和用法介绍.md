# 性能测试工具VTune的功能和用法介绍

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

​                    [WY_Studying](https://me.csdn.net/WY_stutdy)                    2018-01-19 14:36:21                    ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)                    20336                                            ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)                                                收藏                                                    17                                                                

​                            分类专栏：                                [其它](https://blog.csdn.net/wy_stutdy/category_7160033.html)                            文章标签：                                [VTune安装   VTune下载  VTune使用](https://so.csdn.net/so/search/s.do?q=VTune安装   VTune下载  VTune使用&t=blog&o=vip&s=&l=&f=&viparticle=)                    

​                    版权                

# 1.VTune介绍

​    VTune可视化性能分析器（Intel VTune Performance  Analyzer）是一个用于分析和优化程序性能的工具，作为Intel为开发者提供的专门针对寻找软硬件性能瓶颈的一款分析工具，它能确定程序的热点（hotspot），找到导致性能不理想的原因，从而让开发者据此对程序进行优化。

VTune性能分析器能通过以下的手段发现和定位程序中的性能问题：

-    从当前系统中收集性能数据；  
-    从系统到源代码不同的层次上，以不同的互动形式来组织和展示数据；  
-    发现潜在的性能问题，并提出改进措施。  

2.VTune的下载和安装

  VTune的下载和安装比较繁琐，下面简单介绍VTune软件的下载过程和在Linux环境下的安装过程。

2.1  VTune的下载

  VTune的正式版的价格很贵，可以选择下载试用版——[下载链接](https://software.intel.com/en-us/intel-vtune-amplifier-xe)。下载试用版本需要注册账号，可以获得31天的免费试用。注册完成之后，注册邮箱里会收到一封邮件，其中包含软件的下载链接和注册码。

  点击邮箱里的下载链接，选择"Linux系统产品"并选择自己想要下载的软件版本号，本文档以"vtune_amplifier_xe_2013_update17.tar.gz"软件版本号为例。

2.2  VTune的安装

 将下载的软件安装包移动到Linux系统上，解压安装包：

tar zxvf vtune_amplifier_xe_2013_update17.tar.gz

 进入解压后的文件夹，执行"install.sh"脚本，全部按照默认设置，根据安装向导安装即可。

 安装完成后，需要先执行VTune安装成功后得到的文件：

source /home/…/intel/vtune_amplifier_xe_2017.1.0.486011/amplxe-vars.sh

 如图 2.1所示，使用"amplxe-gui"命令启动VTune软件。

![img](https://img-blog.csdn.net/20180119143614072)

图 2.1 VTune启动命令

\3. VTune的使用

 在Linux环境中，启动VTune性能分析器，如图 3.1所示，点击新建工程按钮，新建一个性能分析工程。

![img](https://img-blog.csdn.net/20180119143614237)

图 3.1 新建工程

 如图 3.2所示，选择要分析的目标文件，并填写分析的文件的执行参数。

![img](https://img-blog.csdn.net/20180119143615365)

图 3.2 目标文件选择

 如图 3.3所示，选中目标工程，右击，可以新建目标文件的分析类型。

![img](https://img-blog.csdn.net/20180119143615911)

图 3.3 新建分析类型

 如图 3.4所示，英特尔VTune性能分析器，可以分析的性能类型有："Algorithm Analysis"、"Microarchitecture  Analysis"、"Knights Corner Platform Analysis"和"Custom Analysis"四大类。

![img](https://img-blog.csdn.net/20180119143616289)

图 3.4 VTune分析类型

 如图 3.5所示，"Algorithm Analysis（算法分析）"是运用最广泛的分析类型。它包含"Basic  Hotspots（基础热点）"、"Advanced Hotspots（高级热点）"、"Concurrency（并发）"和"Locks and  Waits（资源锁和等待）"四种子分析类型。下面详细介绍"Basic Hotspots（基础热点）"的使用。

![img](https://img-blog.csdn.net/20180119143616461)

图 3.5 算法分析子类

 

# 4.（Basic Hotspots）基础性能热点分析

 如图 4.1所示，按照第3章节，选择要分析的目标程序，选择算法分析里的基础性能热点分析。设置CPU的采样间隔时间，点击右上角的"Start"按钮开始分析目标程序。

![img](https://img-blog.csdn.net/20180119143616982)

图 4.1 基础性能分析的建立（Basic Hotspots）

 如图 4.2所示，点击开始分析数据后，VTune就开始运行目标程序，并收集相关的性能数据，当收集完成之后，需要手动停止数据的收集。

![img](https://img-blog.csdn.net/20180119143617226)

图 4.2 停止收集数据（Basic Hotspots）

 如图 4.3所示，停止数据分析后，可以获得8类数据，分别为"Analysis Target"、"Analysis Type"、"Collection Log"、"Summary"、"Bottom-up"、"Caller/Callee"、"Top-down Tree"和"Tasks and  Frames"。其中"Analysis Target"、"Analysis Type"和"Collection  Log"三类数据，不做过多分析。主要分析其他几类数据所包含的内容。

![img](https://img-blog.csdn.net/20180119143617628)

图 4.3 收集的数据分类（Basic Hotspots）

4.1 Summary

 如图 4.4所示，Summary主要分析的数据有："Elapsed Time（经过的总时间）"、"Top Hotspots（高热点部分）"、"CPU Usage Histogram（CPU使用直方图）"和"Collection and Platform Info（收集信息和平台信息）"。

![img](https://img-blog.csdn.net/20180119143617821)

图 4.4 Summary的数据展示（Basic Hotspots）

 如图 4.5所示，Elapsed Time信息，主要有总线程数量、开销时间（花费在同步和线程库函数的时间）、自旋时间（CPU等待其它同步资源处理的自旋等待时间）、CPU时间（CPU运行程序所花费的总时间）和暂停时间。

![img](https://img-blog.csdn.net/20180119143618239)

图 4.5 Elapsed Time（Basic Hotspots）

 如图 4.6所示，Top Hotspots信息，会列举VTune分析的程序里的活跃度最高（最耗时）的部分，例如：自旋锁、函数等。

![img](https://img-blog.csdn.net/20180119143618440)

图 4.6 Top Hotspots（Basic Hotspots）

 如图 4.7所示，CPU Usage Histogram信息，显示CPU使用直方图。

![img](https://img-blog.csdn.net/20180119143618926)

图 4.7 CPU Usage Histogram（Basic Hotspots）

 如图 4.8所示，Collection and Platform Info信息，包含了应用程序命令行、操作系统、CPU等信息。

![img](https://img-blog.csdn.net/20180119143619255)

图 4.8 Collection and Platform info（Basic Hotspots）

## 4.2  Bottom-up

 如图 4.9所示，Bottom-up可以查看函数/模块/线程调用时间的耗费，主要分析的数据有：进程、线程、模块、函数和调用的堆栈信息。可以显示程序的进程、线程号，函数的开始地址，CPU开销时间，CPU自旋时间等信息。

![img](https://img-blog.csdn.net/20180119143619759)

图 4.9 Bottom-up（Basic Hotspots）

## 4.3  Caller/Callee

 如图 4.10所示，Caller/Callee主要分析的数据有：CPU总利用时间、各个函数自我利用时间、各个函数的自我开销时间、各个函数的调用者和被调用者等。

![img](https://img-blog.csdn.net/20180119143620047)

图 4.10 Caller/Callee（Basic Hotspots）

## 4.4  Top-down Tree

 如图 4.11所示，Top-down Tree是以树形结构展示每个调用所花费的时间及占比，可以从时间花费最多的地方往下一层一层的展开，找到关键函数分析其性能。其分析的内容和Caller/Callee基本相同。

![img](https://img-blog.csdn.net/20180119143620503)

图 4.11 Top-down Tree（Basic Hotspots）

## 4.5 Tasks and Frames

 如图 4.12所示，Tasks and Frames主要以直方图的形式详细展示了程序的分析时间、CPU使用时间以及进程和各个线程的运行时间。

![img](https://img-blog.csdn.net/20180119143620730)

图 4.12 Tasks and Frames（Basic Hotspots）

 

# 5.  总结

VTune可以帮助用户定位程序中的"热点"，所谓的"热点"就是程序中花费执行时间最长的代码段。VTune性能分析器能够收集应用程序和系统上的性能数据，然后以图形和表格的形式显示出来。从这些显示的数据中，用户能够分析应用程序的性能，从而知道程序中哪个部分执行的慢，为什么执行的慢。