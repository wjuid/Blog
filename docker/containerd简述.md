#                  [containerd简述](https://www.cnblogs.com/embedded-linux/p/10850491.html)             

containerd是容器虚拟化技术，从docker中剥离出来，形成开放容器接口（OCI）标准的一部分。

![img](https://img2018.cnblogs.com/blog/791475/201905/791475-20190511230008829-783521255.png) 

docker对容器的管理和操作基本都是通过containerd完成的。**Containerd 是一个工业级标准的容器运行时，它强调简单性、健壮性和可移植性。Containerd 可以在宿主机中管理完整的容器生命周期：容器镜像的传输和存储、容器的执行和管理、存储和网络等。**详细点说，Containerd 负责干下面这些事情：

•管理容器的生命周期(从创建容器到销毁容器)

•拉取/推送容器镜像

•存储管理(管理镜像及容器数据的存储)

•调用 runC 运行容器(与 runC 等容器运行时交互)

•管理容器网络接口及网络

注意：**Containerd 被设计成嵌入到一个更大的系统中，而不是直接由开发人员或终端用户使用。**

我们可以从下面几点来理解为什么需要独立的 containerd：

•继续从整体 docker 引擎中分离出的项目(开源项目的思路)

•可以被 Kubernets CRI 等项目使用(通用化)

•为广泛的行业合作打下基础(就像 runC 一样)

**docker安装后containerd默认已安装，containerd包含如下命令组件：**

•containerd：高性能容器运行时。

•ctr：containerd的命令行客户端。

•runc：运行容器的命令行工具。

**docker、containerd、docker-shim、runC关系：**

docker：docker本身而言，包括了docker client和dockerd，dockerd实属是对容器相关操作的api的最上层封装，直接面向操作用户。

containerd：dockerd实际真实调用的还是containerd的api接口（rpc方式实现），containerd是dockerd和runC之间的一个中间交流组件。

docker-shim：一个真实运行容器的载体，每启动一个容器都会起一个新的docker-shim的进程。它通过指定三个参数：容器ID、boundle目录（containerd对应某个容器生成目录）、运行时二进制（默认是runC）来调用runC的api创建一个容器。

runC：一个命令行工具端，根据OCI的标准来创建和运行容器。

## containerd应用

docker镜像和containerd镜像通用，但组织方式和存放目录不同，导致docker与ctr命令不通用，各自管理自己的镜像和容器。

containerd的默认配置文件为/etc/containerd/config.toml，可通过命令：

```
containerd config default
```

输出默认配置，可参考文档https://github.com/containerd/containerd/blob/master/docs/ops.md

```
root = "/var/lib/containerd"
state = "/run/containerd"
oom_score = 0
……
```

root键值用于存储containerd持久化数据。

state键值用于存储containerd临时性数据，设备重启后数据丢失。

**显示containerd镜像**

```
sudo ctr images ls
```

**拉取hello-world镜像**

```
sudo ctr images pull docker.io/library/hello-world:latest
```

注：必须全路径，从dockerhub上下载默认hello-world镜像。

**运行容器**

```
sudo ctr run docker.io/library/hello-world:latestmy_hello-world
sudo ctr run -t docker.io/library/busybox:latestmybusybox_demosh
```

 

参考：

1．https://github.com/containerd/containerd

2．https://containerd.io/docs/getting-started/

3．[Containerd 简介](https://www.01hai.com/note/av139590)

4．[Docker组件介绍（一）：runc和containerd](https://www.colabug.com/5397423.html)