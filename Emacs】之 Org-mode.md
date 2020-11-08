# Emacs】之 Org-mode

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

​              Emacs最新版本（24.4）自带org-mode，这就意味着只要打开一个后缀名为org的文件就会自动进入org-mode。



### **（1）列提纲**

```
* 为一级标题
** 为二级标题
*** 为三级标题并以此类推123
```

#### `tab键` 对标题进行展开和关闭

#### `C-c C-t` 可以将一个条目转换成一个TODO事件（再按一次就变成 DONE）



### **（2）插入代码片段(Snippet)**

#### `<s` 然后 Tab键

```
#+BEGIN_SRC emacs-lisp
  ;; Your code goes here
  ;; 你的代码写在这里
#+END_SRC1234
```



### **（3）Org-mode文本内语法高亮**

```
(require 'org)
(setq org-src-fontify-natively t)12
```



### **（4）重置有序列表序号**

#### `M-<RET>` 表示回车



### **（5）Agenda的使用**

#### 配置：

```
;; 设置默认 Org Agenda 文件目录
(setq org-agenda-files '("~/org"))

;; 设置 org-agenda 打开快捷键
(global-set-key (kbd "C-c a") 'org-agenda)12345
```

> 只需要将 `*.org` 文件放入上面指定的文件夹中就可以使用Agenda模式了

#### `C-c C-s` 选择想要开始的时间

#### `C-c C-d` 选择想要结束的时间

#### `C-c a` 可以打开Agenda模式菜单并选择不同的可视方式（r）



### **（6） Capture模板**

#### 配置：

```
  (setq org-capture-templates
      '(("t" "Todo" entry (file+headline "~/.emacs.d/gtd.org" "工作安排")
         "* TODO [#B] %?\n  %i\n"
         :empty-lines 1)))
  )

  ;; r aka remeber
 (global-set-key (kbd "C-c r") 'org-capture)12345678
```

#### 步骤：

> \1. C-c r 
>  \2. ;; 输入你的任务 
>  \3. `C-c C-c` 完成

![这里写图片描述](https://img-blog.csdn.net/20180123211738296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmFuZmFuNDU2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



### **（7） 文档元数据**

```
#+TITLE:       the title to be shown (default is the buffer name)
#+AUTHOR:      the author (default taken from user-full-name)
#+DATE:        a date, an Org timestamp1, or a format string for format-time-string
#+EMAIL:       his/her email address (default from user-mail-address)
#+DESCRIPTION: the page description, e.g. for the XHTML meta tag
#+KEYWORDS:    the page keywords, e.g. for the XHTML meta tag
#+LANGUAGE:    language for HTML, e.g. ‘en’ (org-export-default-language)
#+TEXT:        Some descriptive text to be inserted at the beginning.
#+TEXT:        Several lines may be given.
#+OPTIONS:     H:2 num:t toc:t \n:nil @:t ::t |:t ^:t f:t TeX:t ...
#+BIND:        lisp-var lisp-val, e.g.: org-export-latex-low-levels itemize
               You need to confirm using these, or configure org-export-allow-BIND
#+LINK_UP:     the ``up'' link of an exported page
#+LINK_HOME:   the ``home'' link of an exported page
#+LATEX_HEADER: extra line(s) for the LaTeX header, like \usepackage{xyz}
#+EXPORT_SELECT_TAGS:   Tags that select a tree for export
#+EXPORT_EXCLUDE_TAGS:  Tags that exclude a tree from export
#+XSLT:        the XSLT stylesheet used by DocBook exporter to generate FO file123456789101112131415161718
```

#### 其中#+OPTIONS是复合的选项，包括：

```
H:         set the number of headline levels for export
num:       turn on/off section-numbers
toc:       turn on/off table of contents, or set level limit (integer)
\n:        turn on/off line-break-preservation (DOES NOT WORK)
@:         turn on/off quoted HTML tags
::         turn on/off fixed-width sections
|:         turn on/off tables
^:         turn on/off TeX-like syntax for sub- and superscripts.  If
           you write "^:{}", a_{b} will be interpreted, but
           the simple a_b will be left as it is.
-:         turn on/off conversion of special strings.
f:         turn on/off footnotes like this[1].
todo:      turn on/off inclusion of TODO keywords into exported text
tasks:     turn on/off inclusion of tasks (TODO items), can be nil to remove
           all tasks, todo to remove DONE tasks, or list of kwds to keep
pri:       turn on/off priority cookies
tags:      turn on/off inclusion of tags, may also be not-in-toc
<:         turn on/off inclusion of any time/date stamps like DEADLINES
*:         turn on/off emphasized text (bold, italic, underlined)
TeX:       turn on/off simple TeX macros in plain text
LaTeX:     configure export of LaTeX fragments.  Default auto
skip:      turn on/off skipping the text before the first heading
author:    turn on/off inclusion of author name/email into exported file
email:     turn on/off inclusion of author email into exported file
creator:   turn on/off inclusion of creator info into exported file
timestamp: turn on/off inclusion creation time into exported file
d:         turn on/off inclusion of drawers123456789101112131415161718192021222324252627
```

#### 举个栗子：

```
#+TITLE: 
#+AUTHOR:
#+EMAIL:
#+KEYWORDS: emacs, org-mode
#+OPTIONS:12345
```



### **（8） 内容元数据**

#### ① 分行区块

```
#+BEGIN_VERSE

#+END_VERSE123
```

#### ② 缩进区块

```
#+BEGIN_QUOTE
  缩进区块
#+END_QUOTE123
```

#### ③ 居中区块

```
#+BEGIN_CENTER

#+END_CENTER123
```

#### ④ 代码区块

```
#+BEGIN_SRC Java

#+END_SRC123
```

#### ⑤ 例子

```
: 单行的例子以冒号开头

#+BEGIN_EXAMPLE
 多行的例子
 使用区块
#+END_EXAMPLE123456
```

#### ⑥ 注释

```
#+BEGIN_COMMENT
  块注释
  ...
 #+END_COMMENT1234
```

#### ⑦ 表格与图片

```
;; 对于表格和图片，可以在前面增加标题和标签的说明，以方便交叉引用。
#+CAPTION: This is the caption for the next table (or link)
#+LABEL: tbl:table1

;; 则在需要的地方可以通过
\ref{table1}
;; 来引用该表格。1234567
```



### **（9） 常用快捷键**

#### 表格快捷键

| 快捷键       | 命令说明                                       |
| ------------ | ---------------------------------------------- |
| C-c 竖线     | 创建或转换成表格                               |
| C-c C-c      | 调整表格，不移动光标                           |
| TAB          | 移动到下一区域，必要时新建一行                 |
| S-TAB        | 移动到上一区域                                 |
| RET          | 移动到下一行，必要时新建一行                   |
| M-RET        | (Alt + 回车)插入同级列表项                     |
| M-S-RET      | (Alt + Shift + 回车)插入有checkbox的同级列表项 |
| M-left/right | (left: <- ; right: ->)改变列表顶层级关系       |
| M-up/down    | 上下移动列表项                                 |
| C-c C-c      | 改变checkbox状态                               |
| C-c -        | 添加水平分割线                                 |
| C-c RET      | 添加水平分割线并跳到下一行                     |
| C-c ^        | 根据当前列排序，可以选择排序方式               |

#### 水平分割线

> 五条短线或以上显示为分割线



### **（10） Tags**

> Tags只做一件事：标记这个项目是什么？

```
;; 例如：

** 跟特留尼西特握手                    :苦差:薪水:逃不掉:

#+FILETAGS: :work:12345
```

#### 配置：

```
;; 在.emacs中配置，即全局
(setq org-tag-alist '(("苦差" . ?k)
                     ("薪水" . ?s)))



;; 在文件开头添加，只对本文件有效
#+TAGS: @office(o) @home(h) @traffic(t)12345678
```

#### **添加标题操作：**正文部分：`C-c C-q`， 标题部分：`C-c C-c`

![这里写图片描述](https://img-blog.csdn.net/20180124105655714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmFuZmFuNDU2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

| KEYS    | COMMENT                                       |
| ------- | --------------------------------------------- |
| C-c \   | 按tag搜索标题                                 |
| C-c / m | 搜索并按树状结构显示                          |
| C-c a m | 按标签搜索多个文件（需要将文件加入全局agenda) |

------

#### **搜索查看Tags**

> 通过用agenda 来查看 tags

#### 配置

```
(setq org-agenda-custom-commands
      '(("k" "work haha"
     ((agenda "")
      (tags-todo "work")
      (tags-todo "支持")))))12345
;; 首先为 TODO
;; 打开agenda视图
C-c a  

;; 然后选择你所要的
k 123456
```

![这里写图片描述](https://img-blog.csdn.net/20180124114138576?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmFuZmFuNDU2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



### **（11） 任务状态**

#### 配置

```
(setq org-todo-keywords
      '((sequence "TODO(t!)" "NEXT(n)" "WAITTING(w)" "SOMEDAY(s)" "|" "DONE(d@/!)" "ABORT(a@/!)")
    ))123
```

#### `C-c C-t` 进行展开

![这里写图片描述](https://img-blog.csdn.net/20180124145801735?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZmFuZmFuNDU2OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


 
 

### 参考资料：

------

https://www.cnblogs.com/holbrook/archive/2012/04/12/2444992.html#sec-4-2 
 http://blog.51cto.com/darksun/971615 
 https://orgmode.org/worg/org-tutorials/org-custom-agenda-commands.html 
 http://www.360doc.com/content/16/0619/10/5633793_568957957.shtml