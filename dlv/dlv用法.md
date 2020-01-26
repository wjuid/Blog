# dlv用法

您可以多种方式调用Delve，具体取决于您的使用需求。 Delve使每个尝试都是用户友好的，确保用户必须做最少的工作可能开始调试他们的程序。

请参阅主要使用文档以进一步探索命令。

# 命令行界面

命令

| 命令             | 描述                        |
| ---------------- | --------------------------- |
| args             | 打印函数参数                |
| break            | 设置一个断点                |
| breakpoints      | 打印激活的断点信息          |
| clear            | 删除断点                    |
| clearall         | 删除所有的断点              |
| condition        | 设置断点条件                |
| continue         | 运行到断点或程序终止        |
| disassemble      | 拆解器                      |
| exit             | 退出debugger                |
| frame            | 在不同的框架上执行的命令    |
| funcs            | 打印函数列表                |
| goroutine        | 显示或更改当前goroutine     |
| goroutines       | 列出程序的全部goroutines    |
| help             | 打印出帮助信息              |
| list             | 显示源代码                  |
| locals           | 打印局部变量                |
| next             | 跳到下一行                  |
| on               | 在遇到断点时执行一个命令    |
| print            | 评估表达式                  |
| regs             | 打印CPU寄存器的内容         |
| restart          | 重启进程                    |
| set              | 更改变量的值                |
| source           | 执行包含delve命令列表的文件 |
| sources          | 打印源文件列表              |
| stack            | 打印堆栈跟踪                |
| step             | 单步执行程序                |
| step-instruction | 单步单个执行cpu指令         |
| thread           | 切换到指定的线程            |
| threads          | 打印每一个跟踪线程的信息    |
| trace            | 设置跟踪点                  |
| types            | 打印类型列表                |
| vars             | 打印某个包内的(全局)变量    |

http://mirror.centos.org/centos/8/BaseOS/x86_64/os/Packages/python3-hawkey-0.35.1-9.el8_1.x86_64.rpm