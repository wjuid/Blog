# Emacs入门系列：定制Emacs的Theme

[![img](https://upload.jianshu.io/users/upload_avatars/258656/ba2e2596c3a5?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)](https://www.jianshu.com/u/df2c2942c0e3)

[麦满屯](https://www.jianshu.com/u/df2c2942c0e3)

0.122016.04.28 10:26:29字数 1,317阅读 10,747

距离上一次介绍定制Emacs的基本外观，已经过去了颇长一段时间。主要是由于工作上出了一些情况，突然的工作量大了不少。到最近才相对轻松了一些。所以，我打算继续写这个专题。欢迎大家继续关注我的文章！

# 1. Emacs 的Theme

Emacs是“上古的神器”，按照年龄算，估计要比好多人都大不少。虽然是“老一辈”的神器，不过这些年Emacs的发展一点也没有停止，Emacs与Vi（m）的圣战也在持续。作为神器，华丽的外表必不可少。而Emacs强大的自定义灵活性，也造就了他在外观上不输于现代编辑器的强大能力。

在Emacs中，可以通过灵活的定义颜色、显示方式和内容、字体大小等内容以达到视觉效果的灵活定义。这类定义Emacs外观显示参数的一系列代码就是我们说的Theme。也庆幸Theme机制的存在，我们可以轻松的获得Emacs视觉达人设计的外观样式，让我们的Emacs千变万化，总是充满蓬勃的朝气。

# 2. 在Emacs中加载Theme

随着Emacs发行版一起安装的有几个Theme。我们可以随时切换Emacs的外观。使用组合键 `Meta-x` 调用万能的函数，如图:

![img](https://upload-images.jianshu.io/upload_images/258656-7c994d888a5980f5.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/170)

图1: 加载Theme的命令

调用 `load-theme`  之后，Emacs会问：要加载哪个主题呢？你可以输入需要的主题名称（通常是主题文件的文件名），不过我更多的是使用 `Tab` 键来进行补全。按下  `Tab` 键之后，Emacs会吧当前可以使用的所有Theme名称都列出来。自由的选择吧！

多方便！：D

# 3.安装新的Theme

打开一个Theme定义文件，你看到的一定是这样的：

![img](https://upload-images.jianshu.io/upload_images/258656-68ce295837ed6404.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/660)

图2: Theme文件的内容

> 哦，我的天！我好像掉进了兔子洞，已经分不清东南西北啦！

好在，如果不想研究Theme的编写的话，你完全没必要打开Theme的定义文件（Elisp文件），直接使用大牛们编写好的就可以了。

使用 `package-list-packages` 你可以找到绝大多数的Theme包。感谢Emacs大神，他们将Emacs的所有扩展插件都集中管理了起来，让我们可以从一个统一的数据源安装扩展插件。

我们先来做一些准备工作

## 3.1添加Emacs的扩展安装源

从Emacs 24.1 开始，Emacs有了自己的扩展插件——包（package）管理系统ELPA（Emacs Lisp Package  Archive）。这个管理系统可以与互联网上指定的服务器联系，方便的管理Emacs的各种扩展插件，进行安装、更新等操作。

默认情况下，Emacs只是从ELPA（Emacs的官方扩展插件）获取相关的可用插件信息。而MELPA则是另一处知名的扩展插件聚集地。我们使用如下代码先将MELPA的信息添加到Emacs的配置文件中:

```rust
(require 'package)
(add-to-list 'package-archives 
             '("melpa" . "http://melpa.org/packages/"))
(package-initialize)
```

## 3.2选择和安装Theme

使如上代码生效后，我们可以使用 `package-list-packages` 命令获取当前可用的所有扩展插件列表：

![img](https://upload-images.jianshu.io/upload_images/258656-6b6e0699f3485b77.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/660)

图3: Emacs上的扩展列表

检索关键字 `theme` ，你可以找到大量的结果。选择你喜欢的安装上吧。

## 3.3让Theme生效

好不容易，我们已经安装好了一个Theme，如何生效呢？这里有两个办法：

1. 临时激活Theme，看看效果再说

   Theme扩展安装完了，我们可以在当前Emacs的Session中先激活一个Theme。这个命令可以做到：

   `M-x load-theme`

   在使用按下 `Tab` 键获取Theme列表后，我们选择了一个。嗯，看起来有点意思。

2. 正式启用制定的Theme

   经过一番试用，我们终于有了Chosen One，要让Emacs每次启动时自动加载该Theme，可以在配置文件中使用如下代码（下面的代码中以 “monokai” Theme作为例子，大家可以换成自己喜欢的其他Theme名字）：

   

```rust
(load-theme 'monokai t)
```

1.  

## 3.4让生活更轻松

> “我的天啊，这么多的Package呐。要选一个心仪的Theme，也太耗费精力了。”
>
> “选一个Theme，在一个个的下载，试用。这么费心费力？！唉，算了，我还是玩别的去吧，神器不可近观！”
>
> ……

停，先别着急下结论。我们有更简单的方法。现在隆重介绍Emacs Theme展示板：

![img](https://upload-images.jianshu.io/upload_images/258656-bff56991514c846e.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/553)

图4: Emacs Theme展示

在这里你可以看到几乎所有的Emacs Themes。所以在这里淘换吧！遇到喜欢的，再安装试用就好。

怎样，是不是轻松了很多？

哦，对了，上面那个网站的地址在这里： [https://emacsthemes.com/](https://link.jianshu.com?t=https://emacsthemes.com/)

祝生活愉快！