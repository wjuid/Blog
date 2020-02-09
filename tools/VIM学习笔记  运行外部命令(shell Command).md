# VIM学习笔记  运行外部命令(shell Command)



一个安静的家伙 对大多数事情都没有兴趣

## **执行外部命令**

使用`:!`命令，可以运行一个外部系统命令。例如，以下命令将打开终端窗口，并在其中显示当前日期：

```
:!date
```

使用`:!!`命令，可以重新执行最近一次运行过的命令。

使用`!!`命令，可以运行外部命令并将输出结果做为当前行的内容。例如，输入`!!date`命令，将会把date命令的输出结果插入到文件中，当前行中的原有内容将会被覆盖。

## **读取命令输出**

在常规模式下，使用`:read`命令，可以读取命令执行的输出结果。例如以下命令，将在当前行之下新增一行并插入当前日期。

`:read!date` (For Linux)

`:read!date /t` (For Windows)

## **调用命令终端**

使用`:shell`命令，不需要退出Vim，就可以打开操作系统的命令终端窗口，并在其中执行一个或多个Shell命令。在终端中使用`exit`命令，则可以退回到Vim。

![img](https://pic2.zhimg.com/80/v2-a76fa037ef473ec876b098afa4f37bf5_hd.jpg)

使用Vim8引入的`:terminal`命令，将在新建的水平分割窗口中进入命令终端。也可以使用`:vertical :term`命令，在新建的垂直分割窗口中进入命令终端。

![img](https://pic4.zhimg.com/80/v2-d992918b6c2fbb72b1b8528d545f13c7_hd.jpg)

如果无法正常调用:terminal命令，那么请使用`:version`命令，查看是否包含**+terminal**关键字，以确认在当前版本Vim中已启用此特性。

![img](https://pic1.zhimg.com/80/v2-80a5dc6e4b09520c1de170c4a506e1a0_hd.jpg)

在命令终端中，点击**Ctrl-\-N**快捷键，将从Terminal-Job模式切换至Terminal-Normal模式。在Terminal-Normal模式下，可以像在Vim常规模式下一样，使用光标键或命令来移动光标，也可以使用鼠标或命令来选择和复制文本，以便于将命令输出复制到其他文件。点击**i**键，则可以返回Terminal-Job模式，继续执行命令。

![img](https://pic2.zhimg.com/80/v2-00d9efec7e27947233c6cfd3828f4489_hd.jpg)

![img](https://pic1.zhimg.com/80/v2-b8accc791634a54b259523e548a15e90_hd.jpg)



- https://zhuanlan.zhihu.com/learn-vim)