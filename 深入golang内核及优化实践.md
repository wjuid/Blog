# 深入golang内核及优化实践



[buptbill220](https://www.zhihu.com/people/ming-fang-26)・

![深入golang内核及优化实践](https://pic3.zhimg.com/4b70deef7_t.jpg)

## 最新文章

[关于thrift协议改进畅想](https://zhuanlan.zhihu.com/p/103282810)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

17 天前

[Thrift简介thrift协议你主要提供3种编解码格式协议，Binary、Compact、SimpleJson（还有debug、virtual）；其中最常用的是Binary协议，其传输层一般以BufferedTransport、BufferedTransport居多；这里讨论主要以Binary协议为基础。本身来说… 阅读全文](https://zhuanlan.zhihu.com/p/103282810)



[用protobuf生成json结构插件实现](https://zhuanlan.zhihu.com/p/66793878)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

8 个月前

[![cover](https://pic3.zhimg.com/v2-0bc7b57f679fa2837a7e0d141bad3c34_400x224.jpg)背景json格式不便描述/统一管理在日常中json数据格式应用场景很多。比如在restful请求/返回、业务通信协议、消息（nsq/kafka），广泛使用json。但是目前存在如下问题json数据协议分散在代码里，没有一个统一的描述方式json使用往往使用map结… 阅读全文](https://zhuanlan.zhihu.com/p/66793878)

[golang内核系列--深入理解函数闭包](https://zhuanlan.zhihu.com/p/56750616)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

11 个月前

[![cover](https://pic3.zhimg.com/v2-0bc7b57f679fa2837a7e0d141bad3c34_400x224.jpg)问题闭包 是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。 “官方”的解释是：所谓“闭包”，指的是一个拥有许多变量和绑定了这些变量的环境的表达式（通常是一个函数），因而这些变量也是该表达式的一部分。比如下面“… 阅读全文](https://zhuanlan.zhihu.com/p/56750616)

[golang内核系列--深入理解plan9汇编&实践](https://zhuanlan.zhihu.com/p/56750445)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

11 个月前

[![cover](https://pic3.zhimg.com/v2-0bc7b57f679fa2837a7e0d141bad3c34_400x224.jpg)对于写业务的同学来说，学习汇编可能没必要，仅仅关注业务逻辑即可。   但是当你要深入去优化代码结构、系统架构，就不得不去深入了解golang这门语言，去了解golang内核实现：比如goroutine调度、io调度、map实现、string实现。当然，golang内… 阅读全文](https://zhuanlan.zhihu.com/p/56750445)

[context库--如何获取goroutine id、实现gls](https://zhuanlan.zhihu.com/p/46440890)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

1 年前

[![cover](https://pic3.zhimg.com/v2-0bc7b57f679fa2837a7e0d141bad3c34_400x224.jpg)本文主要讲解context库实现，以及如何实现类似的gls（golang  local  storage）相关代码文件：src/context/context.gosrc/runtime/runtime2.gosrc/runtime/proc.gosrc/runtime/stubs.go参考文献：https://golang.org/doc/asm测试代码：buptbil… 阅读全文](https://zhuanlan.zhihu.com/p/46440890)

[runtime库--io模型1--netpoll](https://zhuanlan.zhihu.com/p/46128288)

[![buptbill220](https://pic1.zhimg.com/v2-23c9d1cfb9ab7ddbaad52895e493ffb1_s.jpg)](https://www.zhihu.com/people/ming-fang-26)

[buptbill220](https://www.zhihu.com/people/ming-fang-26)

1 年前

[![cover](https://pic3.zhimg.com/v2-0bc7b57f679fa2837a7e0d141bad3c34_400x224.jpg)本文主要讲解epoll初始化、epoll调度、poll内部结构核心代码文件：src/runtime/proc.gosrc/runtime/netpoll.gosrc/runtime/netpollxxx.go(netpoll_epoll.go,  netpoll_kqueue.go等；这里只关注netpoll_epoll.go实现)src/internal/poll/fd_pol… 阅读全文](https://zhuanlan.zhihu.com/p/46128288)