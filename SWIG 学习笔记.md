SWIG 学习笔记
原创 心中要有一片海 最后发布于2019-01-26 17:45:57 阅读数 1180 收藏
展开
查看SWIG帮助

swig - -help 可以看到语言特有的选项
例如：swig -java -help

常规选项
     -addextern - 添加额外的extern声明
     -c ++ - 启用C ++处理
     -co <file> - 检查SWIG库中的<file>
     -copyctor - 尽可能自动生成复制构造函数
     -cpperraswarn - 将预处理器#error语句视为#warning（默认）
     -cppext <ext> - 将生成的C ++文件的文件扩展名更改为<ext>
                       （默认为cxx，但使用cpp的PHP除外）
     -copyright - 显示版权声明
     -debug-classes - 显示有关在接口中找到的类的信息
     -debug-module <n> - 在1-4阶段显示模块解析树，<n>是csv阶段列表
     -debug-symtabs - 显示符号表信息
     -debug-symbols - 在符号表中显示目标语言符号
     -debug-csymbols - 在符号表中显示C符号
     -debug-lsymbols - 显示目标语言层符号
     -debug-tags - 显示有关在界面中找到的标签的信息
     -debug-template - 显示调试模板的信息
     -debug-top <n> - 在1-4阶段显示整个解析树，<n>是csv阶段列表
     -debug-typedef - 显示有关接口中类型和typedef的信息
     -debug-typemap - 显示类型映射调试信息
     -debug-tmsearch - 显示类型映射搜索调试信息
     -debug-tmused - 显示使用调试信息的类型映射
     -directors - 打开所有类的导演模式，主要用于测试
     -dirprot - 打开导向器类的受保护成员包装（默认）
     -D <symbol> - 定义符号<symbol>（用于条件编译）
     -E - 仅预处理，不生成包装器代码
     -external-runtime [file] - 导出SWIG运行时堆栈
     -fakeversion <v> - 使SWIG伪造程序版本号为<v>
     -fcompact - 在紧凑模式下编译
     -features <list> - 设置全局功能，其中<list>是逗号分隔的列表
                       功能，例如-features director，autodoc = 1
                       如果未对该功能赋予显式值，则使用默认值1
     -fastdispatch - 启用快速调度模式以生成更快的过载调度程序代码
     -Fmicrosoft - 以Microsoft格式显示错误/警告消息
     -Fstandard - 以常用格式显示错误/警告消息
     -fvirtual - 在virtual排除模式下编译。
     -help - 此输出
     -I- - 不要搜索当前目录
     -I <dir> - 在目录<dir>中查找SWIG文件
     -ignoremissing - 忽略丢失的包含文件
     -importall - 将所有#include语句作为导入
     -includeall - 关注所有#include语句
     -l <ifile> - 包含SWIG库文件<ifile>
     -macroerrors - 报告宏内部的错误
     -makedefault - 创建默认构造函数/析构函数（默认值）
     -M - 列出所有依赖项
     -MD - 相当于`-M -MF <file>'，但不暗示`-E'
     -MF <file> - 生成<file>中的依赖项并继续生成包装器
     -MM - 列出依赖项，但省略SWIG库中的文件
     -MMD - 与`-MD'类似，但省略SWIG库中的文件
     -module <name> - 将模块名称设置为<name>
     -MP - 为所有依赖项生成虚假目标
     -MT <target> - 设置依赖关系生成发出的规则的目标
     -nocontract - 关闭合同检查
     -nocpperraswarn - 不要将预处理器#error语句视为#warning
     -nodefault - 不生成默认构造函数也不生成默认析构函数
     -nodefaultctor - 不生成隐式默认构造函数
     -nodefaultdtor - 不生成隐式默认析构函数
     -nodirprot - 不要包装director受保护的成员
     -noexcept - 不要包装异常说明符
     -nofastdispatch - 禁用快速调度模式（默认）
     -nopreprocess - 跳过预处理器步骤
     -notemplatereduce - 禁用 简化模板中的typedef
     -O - 启用优化选项：
                        -fastdispatch -fvirtual
     -o <outfile> - 将C / C ++输出文件的名称设置为<outfile>
     -oh <headfile> - 将控制器的C ++输出头文件的名称设置为<headfile>
     -outcurrentdir - 将默认输出目录设置为当前目录而不是输入文件的路径
     -outdir <dir> - 将语言特定文件输出目录设置为<dir>
     -pcreversion - 显示PCRE版本信息
     -small - 在虚拟消除和紧凑模式下编译
     -swiglib - 报告SWIG库的位置并退出
     -templatereduce - 减少模板中的所有typedef
     -v - 以详细模式运行
     -version - 显示SWIG版本号
     -Wall - 删除所有警告抑制，也暗示-Wextra
     -Wallkw - 为所有支持的语言启用关键字警告
     -Werror - 将警告视为错误
     -Wextra - 添加以下附加警告：202,309,403,512,321,322
     -w <list> - 抑制/添加警告消息，例如-w401，+ 321 - 请参阅Warnings.html
     -xmlout <file> - 正常处理后，将解析树的XML版本写入<file>
Java选项（与-java一起提供）
      -nopgcpp - 抑制过早的垃圾收集保护参数
      -noproxy - 生成低级函数接口而非代理类
      -oldvarnames - 变量包装器使用旧中间方法名称
      -package <name> - 将Java包的名称设置为<name>

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
    21
    22
    23
    24
    25
    26
    27
    28
    29
    30
    31
    32
    33
    34
    35
    36
    37
    38
    39
    40
    41
    42
    43
    44
    45
    46
    47
    48
    49
    50
    51
    52
    53
    54
    55
    56
    57
    58
    59
    60
    61
    62
    63
    64
    65
    66
    67
    68
    69
    70
    71
    72
    73
    74
    75
    76
    77
    78
    79
    80
    81
    82
    83
    84
    85
    86

SWIG接口文件语法

    简单例子
    
    %module mymodule #指定生成的目标语言文件名
    %{                              #%{%}包含的部分，将会插入到wrapper中
    #include "myheader.h"
    %}
    // Now list ANSI C/C++ declarations
    int foo;                      #swig预处理器根据这里的函数声明生成相应的包装方法
    int bar(int x);
        1
        2
        3
        4
        5
        6
        7
    
    初始化块（%init指令）
    
    %init %{ #在module加载的时候调用，测试了一下，java没有用，但是python有效果
    init_variables();
    %}
        1
        2
        3
    
    %inline 指令
    
    %inline %{ #也就是说块中的代码既会添加到生成的接口文件（xx_wrap.cxx中）又会被swig预处理器解析从而生成相应的包装方法
    /* Create a new vector */
    Vector *new_Vector() {
      return (Vector *) malloc(sizeof(Vector));
    }
    %}
        1
        2
        3
        4
        5
        6
    
    等效于
    
    %{
    /* Create a new vector */
    Vector *new_Vector() {
      return (Vector *) malloc(sizeof(Vector));
    }
    %}
    
    Vector *new_Vector() {
      return (Vector *) malloc(sizeof(Vector));
    }
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
    
    代码插入
    
    %begin %{
    ... code in begin section ... xx_wrap.cxx 最前面，一般是放入宏，好让后面都能用到
    %}
    
    %runtime %{
    ... code in runtime section ... 在%header前面
    %}
    
    %header %{
    ... code in header section ... 等效 %{%}
    %}
    
    %wrapper %{
    ... code in wrapper section ... 放在SWIG生成接口代码的地方
    %}
    
    %init %{
    ... code in init section ... 放在模块加载的地方，目标语言是java就会生成在xx_wrap.cxx文件最后面，
    python的话在SWIG_INIT方法内最后一行
    
    %}
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
    
    SWIG预处理器是C/C++预处理器的增强版本。除了#include之外，其它的C/C++语法都会被SWIG解析并生成相应的包装代码。
    基于这个原因，SWIG直接解析头文件来生成包装代码 swig -java -module example example.h
    
    %module base_module
    %{
    #include "base.h"
    %}
    %include "base.h"//SWIG默认不解析#include，所以当需要让SWIG预处理解析#include需要通过%include指令来达到
    #define STATUS 50//这些C语言语法会被解析，生成相应的包装（get/set）方法
    #define VERSION "1.1"
        1
        2
        3
        4
        5
        6
        7
    
    使用宏在C/C++头文件中添加SWIG指令。因为SWIG指令语法不是标准的C语法所以需要通过宏来隔离，避免编译时语法错误
    
    #ifdef SWIG //swig解析时会定义这个宏
    %module foo
    #endif
        1
        2
        3
    
    %import指令
    SWIG使用％import 指令提供另一个文件包含指令。例如：
    ％import“foo.i”
    ％import的目的是从另一个SWIG接口文件或头文件中收集某些信息，而不实际生成任何包装器代码。
    此类信息通常包括类型声明（例如，typedef）以及可能用作接口中类声明的基类的C ++类。当SWIG用于生成扩展作为相关模块的集合时，
    使用％import也很重要。这是一个高级主题，稍后将在使用模块章节中进行介绍。
    该-importall指令告诉SWIG将#include作为%import。如果要从系统头文件中提取类型定义而不生成任何包装器，这可能很有用。
    %immutable指令
    将变量标记为不可修改，这样就只会在生成的接口文件生成只读方法get而已，不会生成set方法
    
    // File : interface.i
    int a;       // Can read/write
    %immutable;
    int b, c, d;   // Read only variables
    %mutable;
    double x, y;  // read/write
        1
        2
        3
        4
        5
        6
    
    链接char* 全局变量
    当出现char *类型的全局变量时，SWIG使用malloc（）或 new为新值分配内存。具体来说，如果你有这样的变量
    
    char * foo;
    //SWIG生成以下代码：
    / * C模式* /
    void foo_set（char * value）{
      if（foo）free（foo）;
      foo =（char *）malloc（strlen（value）+1）;
      strcpy（foo，value）;
    }
    / * C ++模式。使用-c++选项时* /
    void foo_set（char * value）{
      if（foo）delete [] foo;
      foo = new char [strlen（value）+1];
      strcpy（foo，value）;
    }
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
    
    如果这不是您想要的行为，请考虑使用％immutable指令将变量设置为只读 。或者，您可以编写一个简短的辅助函数来完全按照您的意愿设置值。例如：
    
    ％inline ％{
      void set_foo（char * value）{
        strncpy（foo，value，50）;
      }
    ％}
        1
        2
        3
        4
        5
    
    注意：如果你编写这样的辅助函数，你将不得不从目标脚本语言中将其称为函数（它不像变量那样工作）。例如，在Python中你必须写：
    
    >>> set_foo（“Hello World”）
    
        1
        2
    
    char *变量的 常见错误是链接到声明如下的变量：
    char * VERSION =“1.0”;
    在这种情况下，变量将是可读的，但任何更改值的尝试都会导致段错误或一般保护错误。这是因为当使用malloc（）或new分配给变量的字符串文字值时，SWIG尝试使用free或delete释放旧值 。要解决此问题，您可以将变量标记为只读，编写类型映射，或者编写一个特殊的set函数。另一种方法是将变量声明为数组：
    char VERSION [64] =“1.0”;
    当声明类型为const char *的变量时，SWIG仍会生成用于设置和获取值的函数。但是，默认行为并没有释放以前的内容（从而可能导致内存泄漏）。实际上，在包装这样的变量时，您可能会收到一条警告消息：example.i：20。类型图警告。设置const char *变量可能会泄漏内存。这种行为的原因是const char *变量通常用于指向字符串文字。例如：const char * foo =“Hello World \ n”;因此，在这样的指针上调用free（）是一个非常糟糕的主意。另一方面，将指针更改为指向其他值是合法的。设置此类型的变量时，SWIG会分配一个新字符串（使用malloc或new）并将指针更改为指向新值。但是，重复修改该值将导致内存泄漏，因为旧值未释放。
    %constant、%callback/%nocallback 回调函数指针
    
    / *带回调函数* /
    int binary_op（int a，int b，int（* op）（int，int））;
    
    / *一些回调函数* /
    ％constant int add（int，int）;
    ％constant int sub（int，int）;
    ％constant int mul（int，int）;
        1
        2
        3
        4
        5
        6
        7
    
    在这种情况下，add，sub和mul成为目标脚本语言中的函数指针常量。这允许您按如下方式使用它们：
    
    >>> binary_op（3,4，add）
    7
    >>> binary_op（3,4，mul）
    12
    >>>
        1
        2
        3
        4
        5
    
    不幸的是，通过将回调函数声明为常量，它们不再可以作为函数访问。如果要将函数作为回调函数和函数使用，可以使用％callback和％nocallback指令，如下所示：
    
    / *带回调函数* /
    int binary_op（int a，int b，int（* op）（int，int））;
    
    / *一些回调函数* /
    ％callback（ “％s_cb”）;
    int add（int，int）;
    int sub（int，int）;
    int mul（int，int）;
    ％nocallback;
        1
        2
        3
        4
        5
        6
        7
        8
        9
    
    ％callback 的参数是一个printf样式的格式字符串，它指定回调常量的命名约定（％s被函数名替换）。回调模式保持有效，直到使用％nocallback显式禁用它。执行此操作时，界面现在的工作方式如下：
    
    >>> binary_op（3,4，add_cb）
    7
    >>> binary_op（3,4，mul_cb）
    12
    >>> add（3,4）
    7
    >>> mul（3,4）
    12
        1
        2
        3
        4
        5
        6
        7
        8
    
    全部转为大写来作为回调指针
    
    /* Some callback functions */
    %callback("%(uppercase)s");
    int add(int, int);
    int sub(int, int);
    int mul(int, int);
    %nocallback;
        1
        2
        3
        4
        5
        6
    
    使用%extend扩展c语言struct结构体，将方法绑定到结构体上。这样可以实现类似C++的类的效果
    
    /* file : vector.h */
    ...
    typedef struct Vector {
    double x, y, z;
    } Vector
    
    // file : vector.i
    %module mymodule
    %{
    #include "vector.h"
    %}
    %include "vector.h"
    // Just grab original C header file
    %extend Vector {
    // Attach these functions to struct Vector
    Vector(double x, double y, double z) {
    Vector *v;
    v = (Vector *) malloc(sizeof(Vector));
    v->x = x;
    v->y = y;
    v->z = z;
    return v;
    }
    ~Vector() {
    free($self);
    }
    double magnitude() {
    return sqrt($self->x*$self->x+$self->y*$self->y+$self->z*$self->z);
    }
    void print() {
    printf("Vector [%g, %g, %g]\n", $self->x, $self->y, $self->z);
    }
    };
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
        22
        23
        24
        25
        26
        27
        28
        29
        30
        31
        32
        33
    
    %ignore忽略指定的名字，使得SWIG不会解析指定名字和生成代码
    
    %ignore foo(double); //Ignore all foo(double)
    %ignore Spam::foo; //Ignore foo in class Spam
    %ignore Spam::foo(double);//Ignore foo(double) in class Spam
    %ignore *::foo(double);//Ignore foo(double) in all classes
        1
        2
        3
        4
    
    %rename 重命名
    所有类的成员函数重命名
    
    %rename(foo_i) *::foo(int);
        1
    
    指定的类的成员函数重命名
    
    %rename(foo_i) Spam::foo(int);
    %rename(foo_d) Spam::foo(double);//子类中的也会被重命名
        1
        2
    
    全局函数重命名
    
    %rename(foo_i) ::foo(int);
        1
    
    重命名所有的函数（类成员和非类成员）
    
    %rename(foo_i) foo;
        1
    
    %template 实例化模板
    
    %template(intList) vector<int>;
    typedef int Integer;
    ...
    void foo(vector<Integer> *x);
    
    // 实例化 traits<double, double>, but don't 生成包装代码
    %template() traits<double, double>;
        1
        2
        3
        4
        5
        6
        7
    
    批量实例化
    
    %define TEMPLATE_WRAP(prefix, T...) //这里使用...，是因为不这样做的话， 
    //std::pair<string, int>中的逗号就不能使用了。在宏这里变量是以逗号作为每个单位的分隔符。
    %template(prefix ## Foo) Foo<T >;
    %template(prefix ## Bar) Bar<T >;
    ...
    %enddef
    TEMPLATE_WRAP(int, int)
    TEMPLATE_WRAP(double, double)
    TEMPLATE_WRAP(String, char *)
    TEMPLATE_WRAP(PairStringInt, std::pair<string, int>)
    ...
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
    
    强制模板实例生成默认构造函数
    
    template<class T1, class T2> struct pair {
    T1 first;
    T2 second;
    pair() : first(T1()), second(T2()) { }
    pair(const T1 &x, const T2 &y) : first(x), second(y) { }
    template<class U1, class U2> pair(const pair<U1, U2> &x)
    : first(x.first), second(x.second) { }
    };
    
    %extend pair {
    %template(pair) pair<T1, T2>; // 生成默认拷贝构造函数
    };
    
    // Instantiate a few versions
    %template(pairii) pair<int, int>;
    %template(pairdd) pair<double, double>;
    // Create a default constructor only
    %extend pair<int, int> {
    %template(paird) pair<int, int>; // Default constructor
    };
    // Create default and conversion constructors
    %extend pair<double, double> {
    %template(paird) pair<double, dobule>;// Default constructor
    %template(pairc) pair<int, int>;// Conversion constructor
    }
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
        22
        23
        24
        25
    
    %feature ref unref 生成引用计数类的包装代码
    
    class RCObj {
    // implement the ref counting mechanism
    int add_ref();
    int del_ref();
    int ref_count();
    public:
    virtual ~RCObj() = 0;
    int ref() const {
    return add_ref();
    }
    int unref() const {
    if (ref_count() == 0 || del_ref() == 0 ) {
    delete this;
    return 0;
    }
    return ref_count();
    }
    };
    class A : RCObj {
    public:
    A();
    int foo();
    };
    class B {
    A *_a;
    public:
    B(A *a) : _a(a) {
    a->ref();
    }
    ~B() {
    a->unref();
    }
    };
    
    %module example
    ...
    %feature("ref") RCObj "$this->ref();"
    %feature("unref") RCObj "$this->unref();"
    %include "rcobj.h"
    %include "A.h"
    ...
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
        11
        12
        13
        14
        15
        16
        17
        18
        19
        20
        21
        22
        23
        24
        25
        26
        27
        28
        29
        30
        31
        32
        33
        34
        35
        36
        37
        38
        39
        40
        41
    
    使用％newobject
    ％newobject功能旨在表明它应当采取返回对象的所有权的目标语言。当与具有与之关联的“ref”特性的类型一起使用时，它还将“ref”特征中的代码发送到C ++包装器中。除上述内容外，请考虑包装以下工厂方法：
    
    %newobject AFactory;
    A *AFactory() {
    return new A();
    }
        1
        2
        3
        4
    
    %newobject 标识那些返回堆对象的方法。
    这样返回的C/C++堆对象生命周期就完全由目标语言来管理
    
    %newobject derived::bar;
    
    %inline %{
    class derived : public base {
    public:
    derived * bar(){
    return new derived();
    }
    };
    %}
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
    
    typemap
    编写格式编写格式
    
    %typemap(method [, modifiers]) typelist code ;
    
    typelist : typepattern [, typepattern, typepattern, ... ] ;
    typepattern : type [ (parms) ]               int/*声明匹配的类型*/ (int temp/*声明在wrapper中使用的临时变量*/)
    | type name [ (parms) ]                         int len/*声明匹配的类型*/ (int temp/*声明在wrapper中使用的临时变量*/)
    | ( typelist ) [ (parms) ]                         （char *str, int len)/*声明匹配的类型*/ (int temp/*声明在wrapper中使用的临时变量*/)
    
    code       : { ... } //生成 {}
               | " ... " //不生成 {}
               | %{ ... %}//不生成 {}
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10
    
    typemap作用域
    
    namespace Foo {
      class string;
      %typemap(in) string {          /* Foo::string */
        ...
      }
    }
        1
        2
        3
        4
        5
        6
    
    typemap_example.i
    
    %module TypemapExample //控制java类名
    //argxx都是针对wrapper局部变量做操作
    
    //针对重载方法参数检查，只有在目标语言不支持重载的情况下有用。
    //因为在不支持重载的目标语言中，是通过调用同一个函数，然后参数被包装成对象数组来实现调用C/C++重载的（当然也可以通过%rename来生成函数各不相同的函数来避免重载）。这个时候就需要根据数组的元素个数，以及检查数组参数类型来得知具体是调用哪个重载函数
    %typemap(typecheck , precedence=SWIG_TYPECHECK_INTEGER) int {
    $1 = Custom_Check_Int($input) ? 1 : 0;//检查是否是int类型
    }
    //另一种写法
    /* Required for C++ method overloading */
    /*%typecheck(SWIG_TYPECHECK_STRING_ARRAY) (int argc, char *argv[]) {
    $1 = PyList_Check($input) ? 1 : 0;
    }*/
    
    //处理局部变量（传给C/C++方法参数）
    %typemap(arginit) int flags {
    //arginit，初始化wrapper方法局部变量
    $1 = NULL;
    }
    
    %typemap(default) int flags {
    //default，设置wrapper方法局部变量初始值
    $1 = DEFAULT_FLAGS;
    }
    
    %typemap(in) int flags {
    //in，将java对象转换成c/c++的int
    $1 = JavaObj_to_int($input);
    }


    %typemap(check) int flags {
    //check，检查参数合法性
    if ($1 <= 0) {
    SWIG_exception(SWIG_ValueError, "Expected positive value.");
    }
    }
    
    %typemap(out) int foo {
    //out，C/C++函数返回值进行处理——转换成目标语言的结果类型
    //\$\1，C/C++函数返回值
    $result = To_Java_Int($1);
    }
    
    %typemap(argout) int flags {
    //argout，将参数添加到返回值
    $result += $1
    }
    
    %typemap(freearg) int flags {
    //freearg
    free($1);
    }
    
    %typemap(newfree) int {
    //newfree， 释放C/C++函数返回的堆对象。需要配合（%newobject）一起使用
    delete $1;
    }
    
    %typemap(varin) int {
    //varin，全局变量赋值。目前只在python中看见有效
    $1 = PythonInt_To_C($input);
    }
    
    %typemap(varout) int {
    //varout，获取全局变量。目前只在python中看见有效
    $1 = C_To_PythonInt($result);
    }
    
    int ALL_TIMES;
    
    %typemap(ret) stringheap_t %{
    //ret，针对C/C++函数返回值进行操作
    free($1);
    %}
    
    typedef char * string_t;
    typedef char * stringheap_t;
    
    string_t MakeString1();
    stringheap_t MakeString2();
    
    //numinputs只能取 0 或 1
    //0表示wrapper方法不会声明这个参数，也就是不要求目标语言传递这个参数，$input也就无效
    //1表示wrapper方法会声明这个参数，也就是要求目标语言需要传递这个参数，$input有效
    /*
    %typemap(in, numinputs=0) int flags (int temp) {
    $1 = &temp;
    }*/
    
    %newobject foo;
    int foo(int x, int y, int flags);



    /*%typemap(memberin) int x[20] {
    //memberin，
    memmove($1, $input, 4*sizeof(int));
    }*/
    
    %typemap(memberin) int x[ANY] {
    //memberin，结构体或者类中的成员set函数中将目标语言传入的参数转换成C/C++类成员变量
    memmove($1, $input, $1_dim0*sizeof(int));
    }
    
    struct A {
    int x[4];
    };


    %fragment("AsMyClassFragment", "header") {
    //%fragment("AsMyClassFragment", "header")
    MyClass *AsMyClassA(PyObject *obj) {
    MyClass *value = 0;
    
    return value;
    }
    }
    
    %typemap(in, fragment="AsMyClassFragment") MyClass * {
    //in，当需要将目标语言参数转换成MyClass *对象时，会在wrapper的header那里插入fragment代码
    $result = AsMyClassA($input);
    }
    
    %typemap(varin, fragment="AsMyClassFragment") MyClass * {
    //varin
    $result = AsMyClassA($input);
    }
    
    void foo(MyClass *a, MyClass *b);
    
    %fragment("<limits.h>", "header") {
    %#include <limits.h>
    }
    
    %fragment("AsMyClass", "header", fragment="<limits.h>") {//fragment依赖fragment
    MyClass *AsMyClass(PyObject *obj) {
    MyClass *value = 0;
    
    ... some marshalling code ...
    
    if (ival < CHAR_MIN /*defined in <limits.h>*/) {
    ...
    } else {
    ...
    }
    ...
    return value;
    }
    }
    
    //%fragment("bigfragment", "header", fragment="frag1", fragment="frag2", fragment="frag3") "";//依赖多个fragment
    
    //typemap依赖多个fragment
    //%typemap(in, fragment="frag1, frag2, frag3") {...}
    //等于
    //%typemap(in, fragment="bigfragment") {...}
    
    //强制插入fragment代码片段
    //%fragment("bigfragment");
    
    //目标语言一个参数对应C/C++函数中的两个参数
    //jstring jarg1 -> char *str, int len
    %typemap(in) (char *str, int len) {
    $1 = PyString_AsString($input);
    /* char *str */
    $2 = PyString_Size($input);
    /* int len
    */
    }
    
    int foo(char *str, int len);



    %typemap(in) const int* bar{
    $symname
    $argnum
    $1_name
    
    $1_type
    $*1_type
    $&1_type
    
    $1_ltype
    $*1_ltype
    $&1_ltype
    
    $1_basetype
    
    $1_descriptor
    $*1_descriptor
    $&1_descriptor
    
    $1_mangle
    $*1_mangle
    $&1_mangle
    
    $descriptor(std :: vector <int> *)
    }
    int barf( const int *bar);



    %typemap(in) (int argc, char *argv[]) {
    int i;
    if (!PyList_Check($input)) {
    PyErr_SetString(PyExc_ValueError, "Expecting a list");
    SWIG_fail;
    }
    $1 = PyList_Size($input);
    $2 = (char **) malloc(($1+1)*sizeof(char *));
    for (i = 0; i < $1; i++) {
    PyObject *s = PyList_GetItem($input, i);
    if (!PyString_Check(s)) {
    free($2);
    PyErr_SetString(PyExc_ValueError, "List items must be strings");
    SWIG_fail;
    }
    $2[i] = PyString_AsString(s);
    }
    $2[i] = 0;
    }
    
    %typemap(freearg) (int argc, char *argv[]) {
    if ($2) free($2);
    }
    
    %fragment("incode"{float}, "header") {
    float in_method_float(PyObject *obj) {
    }
    }
    
    %fragment("incode"{long}, "header") {
    float in_method_long(PyObject *obj) {
    }
    }
    
    // %my_typemaps macro definition
    %define %my_typemaps(Type) 
    %typemap(in, fragment="incode"{Type}) Type {
    value = in_method_##Type(obj);
    }
    %enddef
    
    %my_typemaps(float);
    %my_typemaps(long);
    
    int foo(int argc, char *argv[]);
    
    int agoda(long argc, char *argv[]);

————————————————
版权声明：本文为CSDN博主「心中要有一片海」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/lylwo317/article/details/86659030

https://codeload.github.com/swig/swig/tar.gz/rel-4.0.1



https://codeload.github.com/swig/swig/tar.gz/rel-4.0.1

