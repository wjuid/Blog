##  tmux常用命令

| Ctrl+b      | 激活控制台；此时以下按键生效                                 |                          |
| ----------- | ------------------------------------------------------------ | ------------------------ |
| 系统操作    | ?                                                            | 列出所有快捷键；按q返回  |
| d           | 脱离当前会话；这样可以暂时返回Shell界面，输入tmux attach能够重新进入之前的会话 |                          |
| D           | 选择要脱离的会话；在同时开启了多个会话时使用                 |                          |
| Ctrl+z      | 挂起当前会话                                                 |                          |
| r           | 强制重绘未脱离的会话                                         |                          |
| s           | 选择并切换会话；在同时开启了多个会话时使用                   |                          |
| :           | 进入命令行模式；此时可以输入支持的命令，例如kill-server可以关闭服务器 |                          |
| [           | 进入复制模式；此时的操作与vi/emacs相同，按q/Esc退出          |                          |
| ~           | 列出提示信息缓存；其中包含了之前tmux返回的各种提示信息       |                          |
| 窗口操作    | c                                                            | 创建新窗口               |
| &           | 关闭当前窗口                                                 |                          |
| 数字键      | 切换至指定窗口                                               |                          |
| p           | 切换至上一窗口                                               |                          |
| n           | 切换至下一窗口                                               |                          |
| l           | 在前后两个窗口间互相切换                                     |                          |
| w           | 通过窗口列表切换窗口                                         |                          |
| ,           | 重命名当前窗口；这样便于识别                                 |                          |
| .           | 修改当前窗口编号；相当于窗口重新排序                         |                          |
| f           | 在所有窗口中查找指定文本                                     |                          |
| 面板操作    | ”                                                            | 将当前面板平分为上下两块 |
| %           | 将当前面板平分为左右两块                                     |                          |
| x           | 关闭当前面板                                                 |                          |
| !           | 将当前面板置于新窗口；即新建一个窗口，其中仅包含当前面板     |                          |
| Ctrl+方向键 | 以1个单元格为单位移动边缘以调整当前面板大小                  |                          |
| Alt+方向键  | 以5个单元格为单位移动边缘以调整当前面板大小                  |                          |
| Space       | 在预置的面板布局中循环切换；依次包括even-horizontal、even-vertical、main-horizontal、main-vertical、tiled |                          |
| q           | 显示面板编号                                                 |                          |
| o           | 在当前窗口中选择下一面板                                     |                          |
| 方向键      | 移动光标以选择面板                                           |                          |
| {           | 向前置换当前面板                                             |                          |
| }           | 向后置换当前面板                                             |                          |
| Alt+o       | 逆时针旋转当前窗口的面板                                     |                          |
| Ctrl+o      | 顺时针旋转当前窗口的面板                                     |                          |

 

# Tmux 快捷键 & 速查表

启动新会话：

```
tmux [new -s 会话名 -n 窗口名]
```

恢复会话：

```
tmux at [-t 会话名]
```

列出所有会话：

```
tmux ls
```

关闭会话：

```
tmux kill-session -t 会话名
```

关闭所有会话：

```
tmux ls | grep : | cut -d. -f1 | awk '{print substr($1, 0, length($1)-1)}' | xargs kill
```

# 在 Tmux 中，按下 Tmux 前缀 `ctrl+b`，然后：

## 会话

```
:new<回车>  启动新会话
s           列出所有会话
$           重命名当前会话
```

## 窗口 (标签页)

```
c  创建新窗口
w  列出所有窗口
n  后一个窗口
p  前一个窗口
f  查找窗口
,  重命名当前窗口
&  关闭当前窗口
```

## 调整窗口排序

```
swap-window -s 3 -t 1  交换 3 号和 1 号窗口
swap-window -t 1       交换当前和 1 号窗口
move-window -t 1       移动当前窗口到 1 号
```

## 窗格（分割窗口）

```
%  垂直分割
"  水平分割
o  交换窗格
x  关闭窗格
⍽  左边这个符号代表空格键 - 切换布局
q 显示每个窗格是第几个，当数字出现的时候按数字几就选中第几个窗格
{ 与上一个窗格交换位置
} 与下一个窗格交换位置
z 切换窗格最大化/最小化
```

## 同步窗格

这么做可以切换到想要的窗口，输入 Tmux 前缀和一个冒号呼出命令提示行，然后输入：

```
:setw synchronize-panes
```

你可以指定开或关，否则重复执行命令会在两者间切换。 这个选项值针对某个窗口有效，不会影响别的会话和窗口。 完事儿之后再次执行命令来关闭。[帮助](http://blog.sanctum.geek.nz/sync-tmux-panes/)

## 调整窗格尺寸

如果你不喜欢默认布局，可以重调窗格的尺寸。虽然这很容易实现，但一般不需要这么干。这几个命令用来调整窗格：

```
PREFIX : resize-pane -D          当前窗格向下扩大 1 格
PREFIX : resize-pane -U          当前窗格向上扩大 1 格
PREFIX : resize-pane -L          当前窗格向左扩大 1 格
PREFIX : resize-pane -R          当前窗格向右扩大 1 格
PREFIX : resize-pane -D 20       当前窗格向下扩大 20 格
PREFIX : resize-pane -t 2 -L 20  编号为 2 的窗格向左扩大 20 格
```

## 文本复制模式：

按下**前缀 [**进入文本复制模式。可以使用方向键在屏幕中移动光标。默认情况下，方向键是启用的。在配置文件中启用 Vim  键盘布局来切换窗口、调整窗格大小。Tmux 也支持 Vi 模式。要是想启用 Vi 模式，只需要把下面这一行添加到 .tmux.conf 中：

```
setw -g mode-keys vi
```

启用这条配置后，就可以使用 h、j、k、l 来移动光标了。

想要退出文本复制模式的话，按下回车键就可以了。一次移动一格效率低下，在 Vi 模式启用的情况下，可以辅助一些别的快捷键高效工作。

例如，可以使用 w 键逐词移动，使用 b 键逐词回退。使用 f 键加上任意字符跳转到当前行第一次出现该字符的位置，使用 F 键达到相反的效果。

```
vi             emacs        功能
^              M-m          反缩进
Escape         C-g          清除选定内容
Enter          M-w          复制选定内容
j              Down         光标下移
h              Left         光标左移
l              Right        光标右移
L                           光标移到尾行
M              M-r          光标移到中间行
H              M-R          光标移到首行
k              Up           光标上移
d              C-u          删除整行
D              C-k          删除到行末
$              C-e          移到行尾
:              g            前往指定行
C-d            M-Down       向下滚动半屏
C-u            M-Up         向上滚动半屏
C-f            Page down    下一页
w              M-f          下一个词
p              C-y          粘贴
C-b            Page up      上一页
b              M-b          上一个词
q              Escape       退出
C-Down or J    C-Down       向下翻
C-Up or K      C-Up         向下翻
n              n            继续搜索
?              C-r          向前搜索
/              C-s          向后搜索
0              C-a          移到行首
Space          C-Space      开始选中
               C-t          字符调序
```

## 杂项：

```
d  退出 tmux（tmux 仍在后台运行）
t  窗口中央显示一个数字时钟
?  列出所有快捷键
:  命令提示符
```

## 配置选项：

```
# 鼠标支持 - 设置为 on 来启用鼠标
* setw -g mode-mouse off
* set -g mouse-select-pane off
* set -g mouse-resize-pane off
* set -g mouse-select-window off

# 设置默认终端模式为 256color
set -g default-terminal "screen-256color"

# 启用活动警告
setw -g monitor-activity on
set -g visual-activity on

# 居中窗口列表
set -g status-justify centre

# 最大化/恢复窗格
unbind Up bind Up new-window -d -n tmp \; swap-pane -s tmp.1 \; select-window -t tmp
unbind Down
bind Down last-window \; swap-pane -s tmp.1 \; kill-window -t tmp
```

## 配置文件（~/.tmux.conf）：

```
# 基础设置
set -g default-terminal "screen-256color"
set -g display-time 3000
set -g escape-time 0
set -g history-limit 65535
set -g base-index 1
set -g pane-base-index 1

# 前缀绑定 (Ctrl+a)
set -g prefix ^a
unbind ^b
bind a send-prefix

# 分割窗口
unbind '"'
bind - splitw -v
unbind %
bind | splitw -h

# 选中窗口
bind-key k select-pane -U
bind-key j select-pane -D
bind-key h select-pane -L
bind-key l select-pane -R

# copy-mode 将快捷键设置为 vi 模式
setw -g mode-keys vi

# 启用鼠标(Tmux v2.1)
set -g mouse on

# 更新配置文件
bind r source-file ~/.tmux.conf \; display "已更新"

#<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
# Tmux Plugin Manager(Tmux v2.1)
# Tmux Resurrect
set -g @plugin 'tmux-plugins/tmux-resurrect'

# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# Other examples:
# set -g @plugin 'github_username/plugin_name'
# set -g @plugin 'git@github.com/user/plugin'
# set -g @plugin 'git@bitbucket.com/user/plugin'

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
#>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```