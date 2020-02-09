# 内存检测工具AddressSanitizer

[![曾经浮华](https://pic1.zhimg.com/1ffbe89b041a25a4fa2467ee2dfa3b6f_xs.jpg)](https://www.zhihu.com/people/wang-cheng-87-67)

[曾经浮华](https://www.zhihu.com/people/wang-cheng-87-67)

装逼招雷劈

Google出品的内存检测工具AddressSanitizer介绍与分析

## 介绍

AddressSanitizer是Google旗下的一个内存问题检测工具，项目地址：[https://github.com/google/sanitizers/wiki/AddressSanitizer](https://link.zhihu.com/?target=https%3A//github.com/google/sanitizers/wiki/AddressSanitizer)

它与传统的内存问题检测工具，例如 `Valgrind` ，有何区别？

用过 `Valgrind` 的朋友应该都清楚，其会极大的降低程序运行速度，大约降低10倍，而 `AddressSanitizer` 大约只降低2倍，这是什么概念，果然是Google大法好！

## 具体使用

在LLVM及高版本编译器中已经自带了该工具，编译时添加 `-fsanitize=address` 选项。
正常运行程序，如有内存相关问题，即会打印异常信息。

## 工具原理

工具用法比较简单，这里想重点说说该工具的原理。

可参考文档：[https://github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm](https://link.zhihu.com/?target=https%3A//github.com/google/sanitizers/wiki/AddressSanitizerAlgorithm)

由于是内存检测工具，其需要对每一次内存读写操作进行检查：
`*address = ...;  // or: ... = *address;`

进行如下的逻辑判断：

```cpp
if (IsPoisoned(address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
*address = ...;  // or: ... = *address;
```

如果指针读写异常，则统计及打印异常信息，可见整个工具的关键在于 `IsPoisoned` 如何实现，该函数需要快速而且准确。

## 内存映射

其将内存分为两块：
主内存：程序常规使用
 影子内存：记录主内存是否可用等meta信息

如果有个函数 `MemToShadow` 可以根据主内存地址获取到对应的影子内存地址，那么内存检测的实现，可以改写为：

```cpp
shadow_address = MemToShadow(address);
if (ShadowIsPoisoned(shadow_address)) {
  ReportError(address, kAccessSize, kIsWrite);
}
```

## 影子内存

`AddressSanitizer` 用 1 byte 的影子内存，记录主内存中 8 bytes 的数据。

为什么是 8 bytes ，因为malloc分配内存是按照 8 bytes 对齐。

这样，8 bytes 的主内存，共构成 9 种不同情况：

- 8 bytes 的数据可读写，影子内存中的value值为 **0**
- 8 bytes 的数据不可读写，影子内存中的value值为 **负数**
- 前 k bytes 可读写，后 (8 - k) bytes 不可读写，影子内存中的value值为 **k** 。

如果 `malloc(13)` ，根据 8 bytes 字节对齐的原则，需要 2 bytes 的影子内存，第一个byte的值为 0，第二个byte的值为 5。

这时，整个判断流程，可改写为：

```cpp
byte *shadow_address = MemToShadow(address);
byte shadow_value = *shadow_address;
if (shadow_value) {
  if (SlowPathCheck(shadow_value, address, kAccessSize)) {
    ReportError(address, kAccessSize, kIsWrite);
  }
}

// Check the cases where we access first k bytes of the qword
// and these k bytes are unpoisoned.
bool SlowPathCheck(shadow_value, address, kAccessSize) {
  last_accessed_byte = (address & 7) + kAccessSize - 1;
  return (last_accessed_byte >= shadow_value);
}
```

## 主内存映射到影子内存

```
MemToShadow` 采用简单直接映射的方式
64-bit `Shadow = (Mem >> 3) + 0x7fff8000;`
32-bit `Shadow = (Mem >> 3) + 0x20000000;
```

## 例子

如何检测数组访问越界：

```cpp
void foo() {
  char a[8];
  ...
  return;
}
```

`AddressSanitizer` 将其改写为：

```cpp
void foo() {
  char redzone1[32];  // 32-byte aligned
  char a[8];          // 32-byte aligned
  char redzone2[24];
  char redzone3[32];  // 32-byte aligned
  int  *shadow_base = MemToShadow(redzone1);
  shadow_base[0] = 0xffffffff;  // poison redzone1
  shadow_base[1] = 0xffffff00;  // poison redzone2, unpoison 'a'
  shadow_base[2] = 0xffffffff;  // poison redzone3
  ...
  shadow_base[0] = shadow_base[1] = shadow_base[2] = 0; // unpoison all
  return;
}
```

如图：

![img](https://pic3.zhimg.com/80/v2-3857c140cd0ad2c3e6e8d61904d9ff3a_hd.jpg)


将 `char a[8]` 两侧用 `redzone` 包夹，这样数组访问越界时，立马能够侦测。