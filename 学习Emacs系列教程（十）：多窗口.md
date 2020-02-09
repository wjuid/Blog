# 学习Emacs系列教程（十）：多窗口

​                                        转载                                                                                                                                            [weixin_30314631](https://me.csdn.net/weixin_30314631)                    最后发布于2010-03-24 16:37:00                                        阅读数 20                                                                                                                        收藏                                    

​                                                                展开                                    

　　首先先明确下Emacs窗口的概念，我们双击Emacs图标打开程序见到的Windows窗口叫做Frame，包含了标题栏，菜单栏，工具栏，最下面的Mode Line和回显区域，而中间一大块显示文本的区域则是Window，实际上每个窗口都有自己的Mode  Line。下文中我将称Frame为框，Window为窗口，这里和我们平时理解的Windows窗口有点区别。

　　在Emacs里面，一个框可以分割出多个窗口，多个窗口可以显示同一个或者不同Buffer，但是一个窗口只能属于一个Frame。一个窗口同时也只能显示一个Buffer，但是同时打开两个窗口也能显示一个Buffer的不同部分，这两个窗口是同步的，就是说如果在一个窗口中对Buffer做了修改，在另一个窗口中可以立即表现出来。但在一个窗口中移动光标之类的操作不会影响另一个窗口。多缓冲中有当前缓冲这个概念，同样对于多窗口也有当前窗口，对于当前选中的窗口其Mode Line相对于其它窗口颜色会深一些。

 

一、显示窗口

　　命令C-x 2 (split-window-vertically) 垂直拆分窗口，就是把一个窗口上下等分为两个，拆分出来的窗口默认是显示当前Buffer。可以多次使用这个命令，会把一个窗口不停的两等分。对应也有水平拆分窗口的命令C-x 3 (split-window-horizontally)，这两个命令也可以混合使用，如果你屏幕够大画个迷宫出来也不是不可能的。拆分命令还可以加参数，比如M-5 C-x 2就是说上面那个窗口只占5行，其余的位置都给下面的窗口。

　　C-x o (other-window) 可以在多个窗口中切换，从上到下一个一个的来。使用参数来控制选中下面第几个窗口，想往回选的话参数设为负数。

　　C-M-v (scroll-other-window)，用来滚动下一个窗口。

　　上一章我们看到有些命令加了C-x 4这个前缀，这一类命令都是用来操作多窗口的。

　　C-x 4 b *bufname* (switch-to-buffer-other-window) 在另一个窗口打开缓冲。

　　C-x 4 C-o *bufname* (display-buffer) 在另一个窗口打开缓冲，但不选中那个窗口。

　　C-x 4 f *filename* (find-file-other-window) 在另一个窗口打开文件。

　　C-x 4 d *directory* (dired-other-window) 在另一个窗口打开文件夹。

　　C-x 4 m (mail-other-window) 在另一个窗口写邮件。

　　C-x 4 r *filename* (find-file-read-only-other-window) 在另一个窗口以只读方式打开文件。

 　这一类的命令默认是垂直拆分窗口。

 

二、重排窗口

　　窗口排的密密麻麻看上去肯定不舒服，这时使用C-x 0 (delete-window) 来关闭当前窗口，需要注意的是窗口和缓冲是两个概念，关闭一个窗口对缓冲，或者我们正在编辑的文件没有任何影响。也可以使用C-x 1 (delete-other-windows) 关闭其它所有窗口。如果想连窗口打开的缓冲一并关掉使用C-x 4 0 (kill-buffer-and-window)。

　　我们还可以对窗口的大小做些改变：C-x ^ (enlarge-window)让窗口变得高点，C-x { (shrink-window-horizontally) 这个是把窗口变窄，变宽的话是C-x } (enlarge-window-horizontally) ，C-x - (shrink-window-if-larger-than-buffer)这个看字面意思就能理解，如果窗口比缓冲大就缩小点，C-x + (balance-windows)这个命令和前面那个没有任何关系是将所有窗口变得一样高。

　　最后再说一个在窗口中切换的命令，有时候窗口开的太多自己也记不住顺序，使用C-x o就会很麻烦。有一类命令能让你在上下左右切换当前窗口，M-x windmove-right 就是移到右边那个窗口，对应的"left"，"up“， "down"，向四个方向都能移。



　　不知不觉写了十章了，虽然慢了点，但还坚持下来了，继续努力！

 

小结：



| 按键      | 命令                                | 作用                                   |
| --------- | ----------------------------------- | -------------------------------------- |
| C-x 2     | split-window-vertically             | 垂直拆分窗口                           |
| C-x 3     | split-window-horizontally           | 水平拆分窗口                           |
| C-x o     | other-window                        | 选择下一个窗口                         |
| C-M-v     | scroll-other-window                 | 滚动下一个窗口                         |
| C-x 4 b   | switch-to-buffer-other-window       | 在另一个窗口打开缓冲                   |
| C-x 4 C-o | display-buffer                      | 在另一个窗口打开缓冲，但不选中         |
| C-x 4 f   | find-file-other-window              | 在另一个窗口打开文件                   |
| C-x 4 d   | dired-other-window                  | 在另一个窗口打开文件夹                 |
| C-x 4 m   | mail-other-window                   | 在另一个窗口写邮件                     |
| C-x 4 r   | find-file-read-only-other-window    | 在另一个窗口以只读方式打开文件         |
| C-x 0     | delete-window                       | 关闭当前窗口                           |
| C-x 1     | delete-other-windows                | 关闭其它窗口                           |
| C-x 4 0   | kill-buffer-and-window              | 关闭当前窗口和缓冲                     |
| C-x ^     | enlarge-window                      | 增高当前窗口                           |
| C-x {     | shrink-window-horizontally          | 将当前窗口变窄                         |
| C-x }     | enlarge-window-horizontally         | 将当前窗口变宽                         |
| C-x -     | shrink-window-if-larger-than-buffer | 如果窗口比缓冲大就缩小                 |
| C-x +     | balance-windows                     | 所有窗口一样高                         |
|           | windmove-right                      | 切换到右边的窗口(类似：up, down, left) |

![img](https://www.cnblogs.com/Emoticons/yoyocici/224023586.gif) 未完待续。。。

转载于:https://www.cnblogs.com/robertzml/archive/2010/03/24/1692737.html