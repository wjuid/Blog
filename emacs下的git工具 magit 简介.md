# emacs下的git工具 magit 简介

## Table of Contents

- [简介](http://jixiuf.github.io/blog/000100-emacs-magit.html/#orga5849dd)
- [基本界面](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org7da76bf)
- [(?)maigt 提示](http://jixiuf.github.io/blog/000100-emacs-magit.html/#orgd7fd8d8)
- [Stage (git add )与 Unstage](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org8de09cc)
- [Commit(c)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#orgddaa89d)
- [Pull(F)(git pull)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org8b35e97)
- [Push(p)(git push)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#orgb8ec7e8)
- [Log(l)查看日志](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org4ef2ad5)
- [cherry picking(a A)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#orgddce816)
- [Stashing(z)(git stash)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org775130a)
- [Discard or Delete(k 如果用evil-magit则为x)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org54432e5)
- [Resetting (x or evil-magit:o)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org9d8430f)
- [Merge(m)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org23f4860)
- Rebase(r)
  - [ri 交互式rebase (git rebase -i)](http://jixiuf.github.io/blog/000100-emacs-magit.html/#org963b697)

## 简介

[magit](https://github.com/magit/magit) 是emacs下对git 的封装,让你基本在emacs中即可完成对git仓库的管理
 magit 并不是emacs自带的，需要单独安装

## 基本界面

运行 M-x:magit-status 打开当前git仓库 (如果当前没仓库会让你选择相应的路径)
 可以看到以下内容

| 当前处于哪个branch                         | Head             |
| ------------------------------------------ | ---------------- |
| 要push 到哪个远程branch                    | Push             |
| 哪些文件未被git管理                        | Untracked files  |
| 哪些文件修改了未stage                      | Unstaged Changes |
| 哪些文件处于staged状态（即运行了git add ） | Staged Changes   |
| 哪些commit 未push到远程分支                | Unpushed to      |
| 哪些提交 未拉取到本地                      | Unpulled  from   |

甚至可以查看具体文件修改了哪些内容
 在相应的文件上回车则打开那个文件,在相应的修改部位回车则直接跳转到文件修改的位置
 在相应的commit 上回车则能查看此次commit 都修改了哪些文件，具体的文件进行了哪些修改

![magit-status-tab.gif](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-status-tab.gif)

## (?)maigt 提示

magit status 中 ？键可提示magit命令绑定在哪些按键上
 如c 为commit相关,p如push相关等
 ![magit-help.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-help.png)

## Stage (git add )与 Unstage

如果你修改了一个git管理的文件，但是未运行git add  则当前文件处于Unstaged状态
 运行git add 之后则处于Stage状态。git commit 之后，则变成了一个commit 会处于 Unpushed 状态

| s    | Stage   | git add filename        | 从unstaged变成staged  |
| ---- | ------- | ----------------------- | --------------------- |
| u    | Unstage | git reset HEAD filename | 从staged 变成Unstaged |

s 与u 命令可以作用于文件，也可以作用于整个Unstaged changes与Staged changes标题上
 也可以作用于某个文件的一部分(展开一个文件你会看到@@开头的内容)
 甚至你可以选中某个区域只提交选中区域的部分变更
 操作时把光标移动到相应的位置即可

![magit-stage-unstage-s-u.gif](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-stage-unstage-s-u.gif)

## Commit(c)

在magit status buffer中c键为git commit 相关操作
 c后会弹出如下界面
 ![magit-commit-popup.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-commit-popup.png)
 最常用的操作是cc 即普通的Commit
 我最常用的这三个命令

| cc   | Commit | 最普通的                           | git commit                 |
| ---- | ------ | ---------------------------------- | -------------------------- |
| ce   | Extend | 当前Staged的文件合并到上一次提交中 | git commit –amend –no-edit |
| ca   | Amend  | 只修改上次提交的日志               | git commit –amend          |

执行相应的按键后会提示你输入日志

以下动图中依次展示了cc/ce/ca三个命令，每次执行完命令展示了对应的git commit($可查看magit执行了哪些命令)
 ![magit-commit.gif](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-commit.gif)

## Pull(F)(git pull)

![magit-pull-popup.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-pull-popup.png)

| Fu   | pull from upstream                 |
| ---- | ---------------------------------- |
| Fp   | pull from pushremote               |
| Fe   | pull from elsewhere 提示你从哪pull |

如何理解Fu Fp的不同
 比如github上userA有一个仓库r,userB fork了这个仓库
 则对于userB来说 userA/r 是upstream ,而 userB/r 则为pushremote
 即一般来说我们把代码push 到pushremote内，而不是直接push 到upstream上
 通常的应用场景是我们把未成型的代码临时push 到pushremote上，等这个功能彻底完善后才push到upstream上

另外只有设置了pushremote分支，magit status buffer 才会展示有哪些commit未pull 或未push
 将相应的分支设为upstream 或pushremote需要在branch管理内设置(快捷键b)(对应bu bp来设置,如下图)
 ![magit-branch.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-branch.png)

## Push(p)(git push)

| pu   | push to upstram        | 最普通的git push           |
| ---- | ---------------------- | -------------------------- |
| pe   | push to elsewhere      | 会提示你push到哪个远程分支 |
| po   | push another branch to | 会提示你push哪个分支       |
| pT   | push a tag             | push 一个tag标签           |
| pt   | push all tag           | push 所有tag标签           |

同时可以加相应的switch 选项，比如想强制push 上去加-F ，即p-Fu 或 p-Fe 等
  ![magit-push-popup.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-push-popup.png)

## Log(l)查看日志

查看日志相关操作绑定在l上
 如查看当前分支的日志为ll

| ll   | 查看当前分支的日志           |
| ---- | ---------------------------- |
| lo   | log other 查看其他分支的日志 |

![magit-log-popup.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-log-popup.png)
 ![magit-log.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-log.png)
 在具体的commit 上回车则能查看此次commit的提交内容
 ![magit-log-diff.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-log-diff.png)

## cherry picking(a A)

 具体的作用是把某一次commit在当前分支重新commit一次
 比如你想把另一个分支上的某一次提交在当前分支也重做一次,但又不想整个merge那个分支，则可以用此功能
 lo 展示别的分支的日志，找到相应的commit,然后按a或A 来cherry pick

## Stashing(z)(git stash)

 把临时未commit 的更改暂存起来
 我常用的

| zz   | git stash     | 暂存 |
| ---- | ------------- | ---- |
| A    | git stash pop | 找回 |

![magit-stash.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-stash.png)

## Discard or Delete(k 如果用evil-magit则为x)

跟删除相关的操作都可以用这个按键
 如删除一个文件，删除一个@@,删除一个stash等

## Resetting (x or evil-magit:o)

放弃最近的n次提交，这n次的提交内容变成staged状态，之后可以进行合并提交或者丢弃
 只需要在日志日光标定位到想要丢弃的log,即可回滚到这一次的提交状态

## Merge(m)

在magit 中的操作绑定在m上
 常用的操作为 mm之后会提示选择与哪个分支进行merge

## Rebase(r)

| ru   | rebase on upstream                 |
| ---- | ---------------------------------- |
| rp   | rebase on pushremote               |
| re   | 会提示你以哪个分支为基点进行rebase |

![magit-rebase.png](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-rebase.png)

### ri 交互式rebase (git rebase -i)

可以实现调整commit 的顺序，合并commit,放弃某个commit等
 下面这些按键绑定是使用evil-magit后的按键绑定，这些绑定不必担心记不住，用到的时候magit会以注释的形式展示给你

| p       | pick = use commit                                            | 保留此commit                                             |
| ------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| r       | reword = use commit, but edit the commit message             | 只修改此次commit 的日志                                  |
| e       | edit = use commit, but stop for amending                     |                                                          |
| s       | squash = use commit, but meld into previous commit           | 把这条commit与上一条commit合并(会提示输入合并之后的日志) |
| f       | fixup = like "squash", but discard this commit's log message | 同squash,只是合并后的那条commit直接使用上条commit的日志  |
| x       | exec = run command (the rest of the line) using shell        |                                                          |
| d       | drop = remove commit                                         | 丢弃此条commit                                           |
| u       | undo last change                                             |                                                          |
| C-c C-c | tell Git to make it happen                                   | 执行更改                                                 |
| C-c C-k | tell Git that you changed your mind, i.e. abort              | 放弃更改                                                 |
| k       | move point to previous line                                  |                                                          |
| j       | move point to next line                                      |                                                          |
| M-k     | move the commit at point up                                  |                                                          |
| M-j     | move the commit at point down                                |                                                          |
| RET     | show the commit at point in another buffer                   |                                                          |

demo中演示了将commit 1 与update 进行合并，修改commit2 的日志为commit22222,同时调整commit 3 与commit 4 的顺序，
 最后C-c C-c 执行上述变更
 ![magit-rebase-i.gif](http://jixiuf.github.io/assets/blog/000100-emacs-magit.html/magit-rebase-i.gif)