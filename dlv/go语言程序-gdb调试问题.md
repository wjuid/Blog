# go语言程序-gdb调试问题



  以前经常用gdb调试C/C++程序，当学习golang的时候，发现golang的程序也是支持gdb调试的，然而还是遇到一些问题。比如说调试etcd程序就遇到如下问题：

# 【问题1】 info goroutines，提示找不到Undefined info command: "goroutines". Try "help info".

解决方法：启动gdb后，运行目标程序之前（输入r之前），手动加载runtime/runtime-gdb.py，执行如下命令：source /usr/lib/golang/src/runtime/runtime-gdb.py

原因：对于golang相关的特性，gdb本身并不熟知，因此需要通过插件方式使gdb感知到。

验证：

(gdb) b 28

Breakpoint 1 at 0xcc032b: file  /root/etcd/src/github.com/coreos/etcd/gopath/src/github.com/coreos/etcd/cmd/etcd/main.go, line 28.
 (gdb) r
 Starting program: /root/etcd/src/github.com/coreos/etcd/bin/etcd
(gdb) info goroutines
 \* 1 running runtime.systemstack_switch
  2 waiting runtime.gopark
  17 waiting runtime.gopark
  33 waiting runtime.gopark
 \* 49 running runtime.gopark
  42 runnable runtime.gopark
 \* 40 syscall runtime.notetsleepg
  43 runnable runtime.gopark
 \* 74 syscall runtime.notetsleepg
 (gdb) n
 39       startEtcdOrProxyV2()
 (gdb) s
 github.com/coreos/etcd/cmd/vendor/github.com/coreos/etcd/etcdmain.startEtcdOrProxyV2 ()
   at  /root/etcd/src/github.com/coreos/etcd/gopath/src/github.com/coreos/etcd/cmd/vendor/github.com/coreos/etcd/etcdmain/etcd.go:57
 57   func startEtcdOrProxyV2() {
 (gdb) n
 58       grpc.EnableTracing = false
 (gdb) p grpc
 No symbol "grpc" in current context.
 

# 【问题2】打印全局变量是提示No symbol "XXX" in current context.

原因：对于golang语言的程序，并不能像C/C++直接打印全局变量，需要通过路径+变量名字才可以打印出来

解决方法：

1）先查看全局变量所在位置，通过info variables XXX

(gdb) info variables grpc.EnableTracing
 All variables matching regular expression "grpc.EnableTracing":


 File go:

bool github.com/coreos/etcd/cmd/vendor/google.golang.org/grpc.EnableTracing;

2）打印全局变量

(gdb) p 'github.com/coreos/etcd/cmd/vendor/google.golang.org/grpc.EnableTracing'

$1 = true
 注意打印一定要有双引号/单引号

# 【问题3】打印变量长度，不能p len(XXX)必须是p $len(XXX)

# 【问题4】gdb 调试golang程序提示没有no debugging symbols found

主要原因是在编译过程中没有生成符号表，因此需要修改makefile。以flannel编译makefile举例说明：

修改前命令：

```
 dist/flanneld: $(shell find . -type f  -name '*.go')        go build -o dist/flanneld \           -ldflags '-s -w -X github.com/coreos/flannel/version.Version=$(TAG) -extldflags "-static"'
```

修改后命令：

```
 dist/flanneld: $(shell find . -type f  -name '*.go')        go build -o dist/flanneld \           -gcflags "-N -l"  -ldflags '-X github.com/coreos/flannel/version.Version=$(TAG) -extldflags "-static"'
```

删除-ldflags中"-s -w"，增加-gcflags "-N -l"。注意：golang 1.10版本gcflag改成-gcflags "all -N -l"。