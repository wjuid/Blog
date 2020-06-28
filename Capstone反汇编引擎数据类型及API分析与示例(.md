Capstone反汇编引擎数据类型及API分析与示例(一)                

​                                    [                          kabeor](https://xz.aliyun.com/u/20703)  /                      2019-07-26 09:44:00 /                     浏览数 6836                                                                                                                                    [安全技术](https://xz.aliyun.com/tab/1)                                                    [二进制安全](https://xz.aliyun.com/node/23)                                                                            [              顶(0)](javascript:)             [              踩(0)](javascript:)                

------



最近准备用开源的反汇编引擎做个项目，研究了OllyDebug的ODDisasm，disasm与assembl部分代码的思想都很值得学习，但毕竟是2000年的产物，指令集只支持x86，也没有对语义的深度分析，于是转向了对Capstone的研究。

Capstone反汇编引擎可以说是如今世界上最优秀的反汇编引擎，IDA，Radare2，Qemu等著名项目都使用了Capstone Engine，所以选择它来开发是一个不错的选择。
 但在开发时发现官方并未给出详细API文档，网上也没有类似的分析，因此想到自己阅读源码和试验，由此写出了一个简单的非官方版本的API手册，希望能与大家分享。

个人博客： kabeor.cn

## 0x0 开发准备

Capstone官网： http://www.capstone-engine.org

### 自行编译lib和dll方法

源码： https://github.com/aquynh/capstone/archive/4.0.1.zip

下载后解压
  文件结构如下：

> .                   <- 主要引擎core engine + README + 编译文档COMPILE.TXT 等
>  ├── arch            <- 各语言反编译支持的代码实现
>  │   ├── AArch64     <- ARM64 (aka ARMv8) 引擎
>  │   ├── ARM         <- ARM 引擎
>  │   ├── EVM         <- Ethereum 引擎
>  │   ├── M680X       <- M680X 引擎
>  │   ├── M68K        <- M68K 引擎
>  │   ├── Mips        <- Mips 引擎
>  │   ├── PowerPC     <- PowerPC 引擎
>  │   ├── Sparc       <- Sparc 引擎
>  │   ├── SystemZ     <- SystemZ 引擎
>  │   ├── TMS320C64x  <- TMS320C64x 引擎
>  │   ├── X86         <- X86 引擎
>  │   └── XCore       <- XCore 引擎
>  ├── bindings        <- 中间件
>  │   ├── java        <- Java 中间件 + 测试代码
>  │   ├── ocaml       <- Ocaml 中间件 + 测试代码
>  │   └── python      <- Python 中间件 + 测试代码
>  ├── contrib         <- 社区代码
>  ├── cstool          <- Cstool 检测工具源码
>  ├── docs            <- 文档，主要是capstone的实现思路
>  ├── include         <- C头文件
>  ├── msvc            <- Microsoft Visual Studio 支持（Windows）
>  ├── packages        <- Linux/OSX/BSD包
>  ├── windows         <- Windows 支持(Windows内核驱动编译)
>  ├── suite           <- Capstone开发测试工具
>  ├── tests           <- C语言测试用例
>  └── xcode           <- Xcode 支持 (MacOSX 编译)

下面演示Windows10使用Visual Studio2019编译

复制msvc文件夹到一个比较清爽的位置（强迫症专用），内部结构如下：

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183420-eea8a21a-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183420-eea8a21a-aad9-1.png)

VS打开capstone.sln项目文件，解决方案自动载入这些

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183425-f20e2682-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183425-f20e2682-aad9-1.png)

可以看到支持的所有语言都在这里了，如果都需要的话，直接编译就好了，只需要其中几种，则右键解决方案->属性->配置属性  如下

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183429-f486f9a2-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183429-f486f9a2-aad9-1.png)

生成选项中勾选你需要的支持项即可
 编译后会在当前文件夹Debug目录下生成capstone.lib静态编译库和capstone.dll动态库这样就可以开始使用Capstone进行开发了

如果不想自己编译，官方也提供了官方编译版本
 Win32： https://github.com/aquynh/capstone/releases/download/4.0.1/capstone-4.0.1-win32.zip
 Win64： https://github.com/aquynh/capstone/releases/download/4.0.1/capstone-4.0.1-win64.zip

选x32或x64将影响后面开发的位数

### 引擎调用测试

新建一个VS项目，将..\capstone-4.0.1\include\capstone中的头文件以及编译好的lib和dll文件全部拷贝到新建项目的主目录下

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183430-f4d5c294-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183430-f4d5c294-aad9-1.png)

在VS解决方案中，头文件添加现有项capstone.h，资源文件中添加capstone.lib，重新生成解决方案

[![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183431-f5b4c9e4-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183431-f5b4c9e4-aad9-1.png)

那么现在来测试一下我们自己的capstone引擎吧

主文件写入如下代码

```
#include <iostream>
#include <stdio.h>
#include <cinttypes>  
#include "capstone.h"
using namespace std;

#define CODE "\x55\x48\x8b\x05\xb8\x13\x00\x00"

int main(void)
{
    csh handle;
    cs_insn* insn;
    size_t count;

    if (cs_open(CS_ARCH_X86, CS_MODE_64, &handle)) {
        printf("ERROR: Failed to initialize engine!\n");
        return -1;
    }

    count = cs_disasm(handle, (unsigned char*)CODE, sizeof(CODE) - 1, 0x1000, 0, &insn);
    if (count) {
        size_t j;

        for (j = 0; j < count; j++) {
            printf("0x%""Ix"":\t%s\t\t%s\n", insn[j].address, insn[j].mnemonic, insn[j].op_str);
        }

        cs_free(insn, count);
    }
    else
        printf("ERROR: Failed to disassemble given code!\n");

    cs_close(&handle);

    return 0;
}
```

事实上这是官方给出的C语言开发唯一几个例子之一，但注意到代码cs_open(CS_ARCH_X86, CS_MODE_64,  &handle)，测试的是archx64的反编译，因此编译选项也需要设置为x64，除此以外，如果你的项目像我一样是c++开发，那么printf("0x%""Ix"":\t%s\t\t%s\n", insn[j].address, insn[j].mnemonic,  insn[j].op_str);处官方给出的"0x%"PRIx64":\t%s\t\t%s\n"应修改为我这里的"0x%""Ix"":\t%s\t\t%s\n"，这是inttypes支持问题。

运行结果
 [![img](https://xzfile.aliyuncs.com/media/upload/picture/20190720183432-f614cfa6-aad9-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20190720183432-f614cfa6-aad9-1.png)

## 0x1 数据类型及API分析

### 数据类型

#### csh

用于生成调用capstone API的句柄
 `size_t csh`

> 用法： `csh handle;`

#### cs_arch

架构选择

```
enum cs_arch {
    CS_ARCH_ARM = 0,    ///< ARM 架构 (包括 Thumb, Thumb-2)
    CS_ARCH_ARM64,      ///< ARM-64, 也叫 AArch64
    CS_ARCH_MIPS,       ///< Mips 架构
   CS_ARCH_X86,     ///< X86 架构 (包括 x86 & x86-64)
    CS_ARCH_PPC,        ///< PowerPC 架构
    CS_ARCH_SPARC,      ///< Sparc 架构
    CS_ARCH_SYSZ,       ///< SystemZ 架构
    CS_ARCH_XCORE,      ///< XCore 架构
    CS_ARCH_M68K,       ///< 68K 架构
    CS_ARCH_TMS320C64X, ///< TMS320C64x 架构
    CS_ARCH_M680X,      ///< 680X 架构
    CS_ARCH_EVM,        ///< Ethereum 架构
    CS_ARCH_MAX,
    CS_ARCH_ALL = 0xFFFF, // All 架构 - for cs_support()
} cs_arch;
```

> 用法：API中cs_arch参数填入枚举内容，如API中cs_open(cs_arch arch, cs_mode mode, csh *handle);第一个参数填CS_ARCH_X86则支持X86 架构

#### cs_mode

模式选择

```
enum cs_mode {
    CS_MODE_LITTLE_ENDIAN = 0,  ///< little-endian 模式 (default 模式)
    CS_MODE_ARM = 0,    ///< 32-bit ARM
    CS_MODE_16 = 1 << 1,    ///< 16-bit 模式 (X86)
    CS_MODE_32 = 1 << 2,    ///< 32-bit 模式 (X86)
    CS_MODE_64 = 1 << 3,    ///< 64-bit 模式 (X86, PPC)
    CS_MODE_THUMB = 1 << 4, ///< ARM's Thumb 模式, including Thumb-2
    CS_MODE_MCLASS = 1 << 5,    ///< ARM's Cortex-M series
    CS_MODE_V8 = 1 << 6,    ///< ARMv8 A32 encodings for ARM
    CS_MODE_MICRO = 1 << 4, ///< MicroMips 模式 (MIPS)
    CS_MODE_MIPS3 = 1 << 5, ///< Mips III ISA
    CS_MODE_MIPS32R6 = 1 << 6, ///< Mips32r6 ISA
    CS_MODE_MIPS2 = 1 << 7, ///< Mips II ISA
    CS_MODE_V9 = 1 << 4, ///< SparcV9 模式 (Sparc)
    CS_MODE_QPX = 1 << 4, ///< Quad Processing eXtensions 模式 (PPC)
    CS_MODE_M68K_000 = 1 << 1, ///< M68K 68000 模式
    CS_MODE_M68K_010 = 1 << 2, ///< M68K 68010 模式
    CS_MODE_M68K_020 = 1 << 3, ///< M68K 68020 模式
    CS_MODE_M68K_030 = 1 << 4, ///< M68K 68030 模式
    CS_MODE_M68K_040 = 1 << 5, ///< M68K 68040 模式
    CS_MODE_M68K_060 = 1 << 6, ///< M68K 68060 模式
    CS_MODE_BIG_ENDIAN = 1 << 31,   ///< big-endian 模式
    CS_MODE_MIPS32 = CS_MODE_32,    ///< Mips32 ISA (Mips)
    CS_MODE_MIPS64 = CS_MODE_64,    ///< Mips64 ISA (Mips)
    CS_MODE_M680X_6301 = 1 << 1, ///< M680X Hitachi 6301,6303 模式
    CS_MODE_M680X_6309 = 1 << 2, ///< M680X Hitachi 6309 模式
    CS_MODE_M680X_6800 = 1 << 3, ///< M680X Motorola 6800,6802 模式
    CS_MODE_M680X_6801 = 1 << 4, ///< M680X Motorola 6801,6803 模式
    CS_MODE_M680X_6805 = 1 << 5, ///< M680X Motorola/Freescale 6805 模式
    CS_MODE_M680X_6808 = 1 << 6, ///< M680X Motorola/Freescale/NXP 68HC08 模式
    CS_MODE_M680X_6809 = 1 << 7, ///< M680X Motorola 6809 模式
    CS_MODE_M680X_6811 = 1 << 8, ///< M680X Motorola/Freescale/NXP 68HC11 模式
    CS_MODE_M680X_CPU12 = 1 << 9, ///< M680X Motorola/Freescale/NXP CPU12
                    ///< 用于 M68HC12/HCS12
    CS_MODE_M680X_HCS08 = 1 << 10, ///< M680X Freescale/NXP HCS08 模式
} cs_mode;
```

> 用法：API中cs_mode参数填入枚举内容，如API中cs_open(cs_arch arch, cs_mode mode, csh *handle);第二个参数填CS_MODE_64则支持X64模式

#### cs_opt_mem

内存操作

```
struct cs_opt_mem {
    cs_malloc_t malloc;
    cs_calloc_t calloc;
    cs_realloc_t realloc;
    cs_free_t free;
    cs_vsnprintf_t vsnprintf;
} cs_opt_mem;
```

> 用法：可使用用户自定义的malloc/calloc/realloc/free/vsnprintf()函数，默认使用系统自带malloc(), calloc(), realloc(), free() & vsnprintf()

#### cs_opt_mnem

自定义助记符

```
struct cs_opt_mnem {
    /// 需要自定义的指令ID
    unsigned int id;
    /// 自定义的助记符
    const char *mnemonic;
} cs_opt_mnem;
```

#### cs_opt_type

反编译的运行时选项

```
enum cs_opt_type {
    CS_OPT_INVALID = 0, ///< 无特殊要求
    CS_OPT_SYNTAX,  ///< 汇编输出语法
    CS_OPT_DETAIL,  ///< 将指令结构分解为多个细节
    CS_OPT_MODE,    ///< 运行时改变引擎模式
    CS_OPT_MEM, ///< 用户定义的动态内存相关函数
    CS_OPT_SKIPDATA, ///< 在反汇编时跳过数据。然后引擎将处于SKIPDATA模式
    CS_OPT_SKIPDATA_SETUP, ///< 为SKIPDATA选项设置用户定义函数
    CS_OPT_MNEMONIC, ///<自定义指令助记符
    CS_OPT_UNSIGNED, ///< 以无符号形式打印立即操作数
} cs_opt_type;
```

> 用法：API cs_option(csh handle, cs_opt_type type, size_t value);中第二个参数

#### cs_opt_value

运行时选项值(与cs_opt_type关联)

```
enum cs_opt_value {
    CS_OPT_OFF = 0,  ///< 关闭一个选项 - 默认为CS_OPT_DETAIL, CS_OPT_SKIPDATA, CS_OPT_UNSIGNED.
    CS_OPT_ON = 3, ///< 打开一个选项 (CS_OPT_DETAIL, CS_OPT_SKIPDATA).
    CS_OPT_SYNTAX_DEFAULT = 0, ///< 默认asm语法 (CS_OPT_SYNTAX).
    CS_OPT_SYNTAX_INTEL, ///< X86 Intel asm语法 - 默认开启 X86 (CS_OPT_SYNTAX).
    CS_OPT_SYNTAX_ATT,   ///< X86 ATT 汇编语法 (CS_OPT_SYNTAX).
    CS_OPT_SYNTAX_NOREGNAME, ///< 只打印寄存器名和编号 (CS_OPT_SYNTAX)
    CS_OPT_SYNTAX_MASM, ///< X86 Intel Masm 语法 (CS_OPT_SYNTAX).
} cs_opt_value;
```

> 用法：API cs_option(csh handle, cs_opt_type type, size_t value);中第三个参数

#### cs_op_type

通用指令操作数类型，在所有架构中保持一致

```
enum cs_op_type {
    CS_OP_INVALID = 0,  ///< 未初始化/无效的操作数
    CS_OP_REG,          ///< 寄存器操作数
    CS_OP_IMM,          ///< 立即操作数
    CS_OP_MEM,          ///< 内存操作数
    CS_OP_FP,           ///< 浮点数
} cs_op_type;
```

> 目前开放的API中未调用

#### cs_ac_type

通用指令操作数访问类型，在所有架构中保持一致
 可以组合访问类型，例如:CS_AC_READ | CS_AC_WRITE

```
enum cs_ac_type {
    CS_AC_INVALID = 0,        ///< 未初始化/无效的访问类型
    CS_AC_READ    = 1 << 0,   ///< 操作数从内存或寄存器中读取
    CS_AC_WRITE   = 1 << 1,   ///< 操作数从内存或寄存器中写入
} cs_ac_type;
```

> 目前开放的API中未调用

#### cs_group_type

公共指令组，在所有架构中保持一致

```
cs_group_type {
    CS_GRP_INVALID = 0,  ///< 未初始化/无效指令组
    CS_GRP_JUMP,    ///< 所有跳转指令(条件跳转+直接跳转+间接跳转)
    CS_GRP_CALL,    ///< 所有调用指令
    CS_GRP_RET,     ///< 所有返回指令
    CS_GRP_INT,     ///< 所有中断指令(int+syscall)
    CS_GRP_IRET,    ///< 所有中断返回指令
    CS_GRP_PRIVILEGE,    ///< 所有特权指令
    CS_GRP_BRANCH_RELATIVE, ///< 所有相关分支指令
} cs_group_type;
```

> 目前开放的API中未调用

#### cs_opt_skipdata

用户自定义设置SKIPDATA选项

```
struct cs_opt_skipdata {
    /// Capstone认为要跳过的数据是特殊的“指令”
    /// 用户可以在这里指定该指令的“助记符”字符串
    /// 默认情况下(@mnemonic为NULL)， Capstone使用“.byte”
    const char *mnemonic;

    /// 用户定义的回调函数，当Capstone命中数据时调用
    /// 如果这个回调返回的值是正数(>0)，Capstone将跳过这个字节数并继续。如果回调返回0,Capstone将停止反汇编并立即从cs_disasm()返回
    /// 注意:如果这个回调指针为空，Capstone会根据架构跳过一些字节，如下所示:
    /// Arm:     2 bytes (Thumb mode) or 4 bytes.
    /// Arm64:   4 bytes.
    /// Mips:    4 bytes.
    /// M680x:   1 byte.
    /// PowerPC: 4 bytes.
    /// Sparc:   4 bytes.
    /// SystemZ: 2 bytes.
    /// X86:     1 bytes.
    /// XCore:   2 bytes.
    /// EVM:     1 bytes.
    cs_skipdata_cb_t callback;  // 默认值为 NULL

    /// 用户自定义数据将被传递给@callback函数指针
    void *user_data;
} cs_opt_skipdata;
```

> 目前开放的API中未调用
>
> #### cs_detail
>
> 注意:只有当CS_OPT_DETAIL = CS_OPT_ON时，cs_detail中的所有信息才可用

在arch/ARCH/ARCHDisassembler.c的ARCH_getInstruction中初始化为memset(., 0, offsetof(cs_detail, ARCH)+sizeof(cs_ARCH))

如果cs_detail发生了变化，特别是在union之后添加了字段，那么相应地更新arch/ arch/ archdisassembly .c

```
struct cs_detail {
    uint16_t regs_read[12]; ///< 这个参数读取隐式寄存器列表
    uint8_t regs_read_count; ///< 这个参数读取隐式寄存器计数

    uint16_t regs_write[20]; ///< 这个参数修改隐式寄存器列表
    uint8_t regs_write_count; ///< 这个参数修改隐式寄存器计数

    uint8_t groups[8]; ///< 此指令所属的指令组的列表
    uint8_t groups_count; ///< 此指令所属的组的数

    /// 特定于体系结构的信息
    union {
        cs_x86 x86;     ///< X86 架构, 包括 16-bit, 32-bit & 64-bit 模式
        cs_arm64 arm64; ///< ARM64 架构 (aka AArch64)
        cs_arm arm;     ///< ARM 架构 (包括 Thumb/Thumb2)
        cs_m68k m68k;   ///< M68K 架构
        cs_mips mips;   ///< MIPS 架构
        cs_ppc ppc;     ///< PowerPC 架构
        cs_sparc sparc; ///< Sparc 架构
        cs_sysz sysz;   ///< SystemZ 架构
        cs_xcore xcore; ///< XCore 架构
        cs_tms320c64x tms320c64x;  ///< TMS320C64x 架构
        cs_m680x m680x; ///< M680X 架构
        cs_evm evm;     ///< Ethereum 架构
    };
} cs_detail;
```

#### cs_insn

指令的详细信息

```
struct cs_insn {
    /// 指令ID(基本上是一个用于指令助记符的数字ID)
    /// 应在相应架构的头文件中查找'[ARCH]_insn' enum中的指令id，如ARM.h中的'arm_insn'代表ARM, X86.h中的'x86_insn'代表X86等…
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    /// 注意:在Skipdata模式下，这个id字段的“data”指令为0
    unsigned int id;

    /// 指令地址 (EIP)
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    uint64_t address;

    /// 指令长度
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    uint16_t size;

    /// 此指令的机器码，其字节数由上面的@size表示
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    uint8_t bytes[16];

    /// 指令的Ascii文本助记符
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    char mnemonic[CS_MNEMONIC_SIZE];

    /// 指令操作数的Ascii文本
    /// 即使在CS_OPT_DETAIL = CS_OPT_OFF时也可以使用此信息
    char op_str[160];

    /// cs_detail指针
    /// 注意:只有同时满足以下两个要求时，detail指针才有效:
    /// (1) CS_OP_DETAIL = CS_OPT_ON
    /// (2) 引擎未处于Skipdata模式(CS_OP_SKIPDATA选项设置为CS_OPT_ON)
    ///
    /// 注意2:当处于Skipdata模式或detail模式关闭时，即使这个指针不是NULL，它的内容仍然是不相关的。
    cs_detail *detail;
} cs_insn;
```

#### cs_err

Capstone API遇到的各类型的错误时cs_errno()的返回值

```
typedef enum cs_err {
    CS_ERR_OK = 0,   ///< 无错误
    CS_ERR_MEM,      ///< 内存不足: cs_open(), cs_disasm(), cs_disasm_iter()
    CS_ERR_ARCH,     ///< 不支持的架构: cs_open()
    CS_ERR_HANDLE,   ///<句柄不可用: cs_op_count(), cs_op_index()
    CS_ERR_CSH,      ///< csh参数不可用: cs_close(), cs_errno(), cs_option()
    CS_ERR_MODE,     ///< 无效的或不支持的模式: cs_open()
    CS_ERR_OPTION,   ///< 无效的或不支持的选项: cs_option()
    CS_ERR_DETAIL,   ///< 信息不可用，因为detail选项是关闭的
    CS_ERR_MEMSETUP, ///< 动态内存管理未初始化(见 CS_OPT_MEM)
    CS_ERR_VERSION,  ///< 不支持版本 (bindings)
    CS_ERR_DIET,     ///< 在“diet”引擎中访问不相关的数据
    CS_ERR_SKIPDATA, ///< 在SKIPDATA模式下访问与“数据”指令无关的数据
    CS_ERR_X86_ATT,  ///< X86 AT&T 语法不支持(在编译时退出)
    CS_ERR_X86_INTEL, ///< X86 Intel 语法不支持(在编译时退出)
    CS_ERR_X86_MASM, ///< X86 Intel 语法不支持(在编译时退出)
} cs_err;
```