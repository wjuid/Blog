# GO语言调试利器——dlv



安装后直接运行dlv将会看到如下信息：

![img](https://upload-images.jianshu.io/upload_images/13918138-ffc2c58808e01132?imageMogr2/auto-orient/strip|imageView2/2/w/1080)

上面列举了dlv的一些命令，其中常用的有如help、attach、core、debug、trace、version等。

dlv help：上面的信息只是列出了命令列表，具体使用方法没有给出，这里可以运行dlv help查看具体命令的使用方法。也可以运行dlv help help查看help命令的使用说明。

![img](https://upload-images.jianshu.io/upload_images/13918138-957981b0455ee97c?imageMogr2/auto-orient/strip|imageView2/2/w/1080)

例如：查看core命令使用说明。

![img](https://upload-images.jianshu.io/upload_images/13918138-8264284e1595fbad?imageMogr2/auto-orient/strip|imageView2/2/w/1080)

dlv version：查看dlv工具版本信息

![img](https://upload-images.jianshu.io/upload_images/13918138-c7ce8fbb7c6630c2?imageMogr2/auto-orient/strip|imageView2/2/w/323)

进入调试模式的几种方法：

1、dlv attach pid：类似与gdb attach pid，可以对正在运行的进程直接进行调试(pid为进程号)。

2、dlv run|debug：run命令已被debug命令取代，运行dlv debug test.go会先编译go源文件，同时执行attach命令进入调试模式，该命令会在当前目录下生成一个名为debug的可执行二进制文件，退出调试模式会自动被删除。

3、dlv exec executable_file：直接从二进制文件启动调试模式。

4、dlv core executable_file core_file：以core文件启动调试，通常进行dlv的目的就是为了找出可执行文件core的原因，通过core文件可直接找出具体进程异常的信息。

dlv trace：该命令最直接的用途是可以追踪代码里函数的调用轨迹，可通过help查看其调用方式。

![img](https://upload-images.jianshu.io/upload_images/13918138-70e2e14b23b8f9a8?imageMogr2/auto-orient/strip|imageView2/2/w/944)

如下源代码，main函数里每隔1秒执行一次Afunc函数，现用trace命令跟踪其调用轨迹。

![img](https://upload-images.jianshu.io/upload_images/13918138-d4f985a68cae2dcc?imageMogr2/auto-orient/strip|imageView2/2/w/312)

![img](https://upload-images.jianshu.io/upload_images/13918138-2b087810f553bbb5?imageMogr2/auto-orient/strip|imageView2/2/w/946)

正式调试开始，为演示dlv的调试命令，这里写了一个简单的测试程序：启动两个协程对变量进行处理，然后输出处理后的结果。

![img](https://upload-images.jianshu.io/upload_images/13918138-622dc38f06c55c12?imageMogr2/auto-orient/strip|imageView2/2/w/554)

进入调试模式后，同样可以执行help查看所有的命令。

![img](https://upload-images.jianshu.io/upload_images/13918138-27b06de7a3f86216?imageMogr2/auto-orient/strip|imageView2/2/w/956)

大部分命令和gdb类似，break [name] ：设置断点，当需要设置多个断点时，为了断点可识别可进行自定义命名。进入调试模式后先打断点，b test.go:14，然后执行c（类似于gdb的run）运行到断点处。

![img](https://upload-images.jianshu.io/upload_images/13918138-be25400b53f2805f?imageMogr2/auto-orient/strip|imageView2/2/w/722)

bp：查看所有断点。

![img](https://upload-images.jianshu.io/upload_images/13918138-0e276fe473ed8915?imageMogr2/auto-orient/strip|imageView2/2/w/324)

on   ：当运行到某断点时执行相应命令，断点可以是名称（在设置断点时可命名断点）或者编号，例如on 1 p a表示运行到断点1时打印变量a。

![img](https://upload-images.jianshu.io/upload_images/13918138-7dfefc81e7fb7fe0?imageMogr2/auto-orient/strip|imageView2/2/w/842)

condition  ：有条件的中断断点，当expression为true时指定的断点将被中断，程序将继续执行。

next|step：逐行执行代码，区别和gdb类似，next遇到函数调用时不会进入该函数，step则会进入函数，如果需要查看函数的具体执行过程则用step，否则用next，调试过程中一般这两个命令会结合使用，对于用户自定义的函数可能需要进入函数内部查看每步执行情况，对于系统函数则没有必要。

step-instruction |si：单步单核执行代码，如果不希望多协程并发执行可以使用该命令，这在多协程调试时极为方便。

stepout：当使用s命令进入某个函数后又想退出时可用此命令。

args：查看函数参数，调试时可以用此命令查看被调用函数所传入的参数值。

locals：查看所有局部变量，locals var_name：查看具体某个变量，var_name可以是正则表达式。

![img](https://upload-images.jianshu.io/upload_images/13918138-2f80cdf2ad7fbe7c?imageMogr2/auto-orient/strip|imageView2/2/w/266)

clear：清除单个断点；clearall：清除所有断点。

![img](https://upload-images.jianshu.io/upload_images/13918138-8962c045feb52c40?imageMogr2/auto-orient/strip|imageView2/2/w/343)

list：打印当前断点位置的源代码，list后面加行号可以展示该行附近的源代码，如list test.go:10将会展示test.go文件第10行附近的代码，值得注意的是该行必须是代码行而不能是空行。

![img](https://upload-images.jianshu.io/upload_images/13918138-f11afa8e565ead36?imageMogr2/auto-orient/strip|imageView2/2/w/343)

bt：打印当前栈信息。

frame：切换栈。

regs：打印寄存器内容。

go语言的优势就是它的协程，为演示多协程下如何调试go进程，对Func函数做如下改动，增加一个for死循环，每休眠一秒输出变量a。

![img](https://upload-images.jianshu.io/upload_images/13918138-f5dba477cf7370b4?imageMogr2/auto-orient/strip|imageView2/2/w/1080)

goroutines：显示所有协程。编号为1的是主协程，编号为5、6的为自己启动的协程。

![img](https://upload-images.jianshu.io/upload_images/13918138-3183757d5054e1cb?imageMogr2/auto-orient/strip|imageView2/2/w/332)

goroutine：协程切换。dlv默认会一直在主协程上执，为打印Func函数中的临时变量a，需要进行协程切换，先执行goroutine 5表示切换到5号协程上，然后bt命令查看当前协程的栈状态，执行frame 3 locals切换到3号栈上并打印栈上的变量。

![img](https://upload-images.jianshu.io/upload_images/13918138-4b074b9ab8cdf717?imageMogr2/auto-orient/strip|imageView2/2/w/240)

sources：打印所有源代码文件路径。

source：执行一个含有dlv命令的文件，source命令允许将dlv命令放在一个文件中，然后逐行执行文件内的命令。

threads：显示所有线程，thread：线程切换，这两个命令与goroutines、goroutine类似，不过在go语言中一般很少使用。

trace：类似于打断点，但程序运行到该处时不会中断而是继续执行，同时会输出一行提示信息。

![img](https://upload-images.jianshu.io/upload_images/13918138-526f3084da3079ce?imageMogr2/auto-orient/strip|imageView2/2/w/387)

总结：

利用调试工具进行代码流程分析固然是一种很好的手段，但是在实际项目开发中最好在代码里多加一些有意义的log输出，在遇到问题时能快速定位分析