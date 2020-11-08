# Go  go.mod入门

![img](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

​                    [SanfordZhu](https://me.csdn.net/weixin_39003229)                    2019-07-29 14:20:32                    ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)                    22651                                            ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)                                                收藏                                                    32                                                                

​                            分类专栏：                                [Go](https://blog.csdn.net/weixin_39003229/category_8328481.html)                    

​                    版权                

### 什么是go.mod?

Go.mod是Golang1.11版本新引入的官方包管理工具用于解决之前没有地方记录依赖包具体版本的问题，方便依赖包的管理。

Go.mod其实就是一个Modules，关于Modules的官方定义为：

Modules是相关Go包的集合，是源代码交换和版本控制的单元。go命令直接支持使用Modules，包括记录和解析对其他模块的依赖性。Modules替换旧的基于GOPATH的方法，来指定使用哪些源文件。

Modules和传统的GOPATH不同，不需要包含例如src，bin这样的子目录，一个源代码目录甚至是空目录都可以作为Modules，只要其中包含有go.mod文件。

###   如何使用go.mod?

1.首先将go的版本升级为1.11以上

2.设置GO111MODULE

### GO111MODULE

`GO111MODULE`有三个值：`off`, `on`和`auto（默认值）`。

- `GO111MODULE=off`，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。

- `GO111MODULE=on`，go命令行会使用modules，而一点也不会去GOPATH目录下查找。

- ```
  GO111MODULE=auto
  ```

  ，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：   

  - 当前目录在GOPATH/src之外且该目录包含go.mod文件
  - 当前文件在包含go.mod文件的目录下面。

### go mod命令：

golang 提供了 `go mod`命令来管理包。go mod 有以下命令：

![img](https://img-blog.csdnimg.cn/20190729135250528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTAwMzIyOQ==,size_16,color_FFFFFF,t_70)

 

### go.mod如何在项目中使用？

1.首先我们要在GOPATH/src 目录之外新建工程，或将老工程copy到GOPATH/src 目录之外。

PS：go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。

go.mod 提供了`module`, `require`、`replace`和`exclude`四个命令

- `module`语句指定包的名字（路径）
- `require`语句指定的依赖项模块
- `replace`语句可以替换依赖项模块
- `exclude`语句可以忽略依赖项模块

下面是我们建立了一个hello.go的文件

```Go
package main



 



import (



	"fmt"



)



 



func main() {



    fmt.Println("Hello, world!")



}
```

2.在当前目录下，命令行运行 go mod init + 模块名称 初始化模块

即go mod init hello

运行完之后，会在当前目录下生成一个go.mod文件，这是一个关键文件，之后的包的管理都是通过这个文件管理。

官方说明：除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希 
 go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 go.mod和go.sum都应检入版本控制。 
 go.sum 不需要手工维护，所以可以不用太关注。

**注意**：子目录里是不需要init的，所有的子目录里的依赖都会组织在根目录的go.mod文件里

接下来，让我们的项目依稀一下第三方包

如修改hello.go文件如下，按照过去的做法，要运行hello.go需要执行`go get` 命令 下载gorose包到 $GOPATH/src

```Go
package main



 



import (



	"fmt"



	"github.com/gohouse/gorose"



)



 



func main() {



    fmt.Println("Hello, world!")



}
```

**但是，使用了新的包管理就不在需要这样做了**

直接 `go run hello.go`

稍等片刻… go 会自动查找代码中的包，下载依赖包，并且把具体的依赖关系和版本写入到go.mod和go.sum文件中。
 查看go.mod，它会变成这样：

```Go
module test



 



require (



	github.com/gohouse/gorose v1.0.5



)
```

`require `关键字是引用，后面是包，最后v1.11.1 是引用的版本号

这样，一个使用Go包管理方式创建项目的小例子就完成了。

### 问题一：依赖的包下载到哪里了？还在GOPATH里吗？

**不在。**
 使用Go的包管理方式，依赖的第三方包被下载到了`$GOPATH/pkg/mod`路径下。

### 问题二： 依赖包的版本是怎么控制的？

在上一个问题里，可以看到最终下载在`$GOPATH/pkg/mod` 下的包中最后会有一个版本号 v1.0.5，也就是说，`$GOPATH/pkg/mod`里可以保存相同包的不同版本。

版本是在go.mod中指定的。如果，在go.mod中没有指定，go命令会自动下载代码中的依赖的最新版本，本例就是自动下载最新的版本。如果，在go.mod用require语句指定包和版本 ，go命令会根据指定的路径和版本下载包，
 指定版本时可以用`latest`，这样它会自动下载指定包的最新版本；

### 问题三： 可以把项目放在$GOPATH/src下吗？

**可以。**但是go会根据GO111MODULE的值而采取不同的处理方式，默认情况下，`GO111MODULE=auto` 自动模式

1.auto 自动模式下，项目在`$GOPATH/src`里会使用`$GOPATH/src`的依赖包，在$GOPATH/src外，就使用go.mod 里 require的包

2.on 开启模式，1.12后，无论在`$GOPATH/src`里还是在外面，都会使用go.mod 里 require的包

3.off 关闭模式，就是老规矩。

### 问题三： 依赖包中的地址失效了怎么办？比如 golang.org/x/… 下的包都无法下载怎么办？

在go快速发展的过程中，有一些依赖包地址变更了。以前的做法：

1.修改源码，用新路径替换import的地址

2.git clone 或 go get 新包后，copy到$GOPATH/src里旧的路径下

无论什么方法，都不便于维护，特别是多人协同开发时。

使用go.mod就简单了，在go.mod文件里用 replace 替换包，例如

```
replace golang.org/x/text => github.com/golang/text latest
```

这样，go会用 github.com/golang/text 替代golang.org/x/text，原理就是下载github.com/golang/text 的最新版本到 `$GOPATH/pkg/mod/golang.org/x/text`下。

### 问题四： init生成的go.mod的模块名称有什么用？

本例里，用 go mod init hello 生成的go.mod文件里的第一行会申明`module hello`

因为我们的项目已经不在`$GOPATH/src`里了，那么引用自己怎么办？就用模块名+路径。

例如，在项目下新建目录 utils，创建一个tools.go文件:

```Go
package utils



 



import “fmt”



 



func PrintText(text string) {



    fmt.Println(text)



}
```

在根目录下的hello.go文件就可以 import “hello/utils” 引用utils

```Go
package main



 



import (



"hello/utils"



 



"github.com/astaxie/beego"



)



 



func main() {



 



    utils.PrintText("Hi")



 



    beego.Run()



}
```

### 问题五：以前老项目如何用新的包管理

1. 如果用auto模式，把项目移动到`$GOPATH/src`外
2. 进入目录，运行 `go mod init + 模块名称`
3. `go build` 或者 `go run` 一次