查看Docker容器使用资源情况--docker stats
原创置顶 爱若手握流沙 最后发布于2019-03-01 15:34:58 阅读数 3610  收藏
展开
在容器的使用过程中，如果能及时的掌握容器使用的系统资源，无论对开发还是运维工作都是非常有益的。幸运的是 docker 自己就提供了这样的命令：docker stats。

目录

docker stats (不带任何参数选项)

docker stats --no-stream  (只返回当前的状态)

docker stats   --no-stream   容器ID/Name (只输出指定的容器)

docker stats --format  “table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}" (格式化输出的结果)

总结

docker stats (不带任何参数选项)
该命令用来显示容器使用的系统资源。不带任何参数选项执行 docker stats 命令：

​            

默认情况下，stats 命令会每隔 1 秒钟刷新一次输出的内容直到你按下 ctrl + c。下面是输出的主要内容：

[CONTAINER]：以短格式显示容器的 ID。

[CPU %]：CPU 的使用情况。

[MEM USAGE / LIMIT]：当前使用的内存和最大可以使用的内存。

[MEM %]：以百分比的形式显示内存使用情况。

[NET I/O]：网络 I/O 数据。

[BLOCK I/O]：磁盘 I/O 数据。 

[PIDS]：PID 号。

docker stats --no-stream  (只返回当前的状态)
如果不想持续的监控容器使用资源的情况，可以通过 --no-stream 选项只输出当前的状态：

这样输出的结果就不会变化了，看起来省劲不少。

​           

docker stats   --no-stream   容器ID/Name (只输出指定的容器)
想查看某个容器的资源使用情况，可以为 docker stats 命令显式的指定目标容器的名称或者是 ID：

​            

           docker stats   --no-stream   registry   1493（指定多个容器的名称或ID，这里的 registry 和 1493 分别是容器的名称和ID）


​           

注意，多个容器的名称或者是ID之间需要用空格进行分割。

我们可能已经发现了，第一列不再显示默认的容器 ID，而是显示了我们传入的容器名称和 ID。

基于此，我们可以通过简单的方式使用容器的名称替代默认输出中的容器 ID：

           docker stats $(docker ps --format={{.Names}})


​            

docker stats --format "table{{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}" (格式化输出的结果)
上面把输出中的容器 ID 替换成了名称。其实 docker stats 命令支持我们通过 --format 选项自定义输出的内容和格式：

​            

上面的命令中我们只输出了 Name, CPUPerc 和 Memusage 三列。下面是自定义的格式中可以使用的所有占位符：

.Container    根据用户指定的名称显示容器的名称或 ID。

.Name           容器名称。

.ID                 容器 ID。

.CPUPerc       CPU 使用率。

.MemUsage  内存使用量。

.NetIO           网络 I/O。       

.BlockIO        磁盘 I/O。

.MemPerc     内存使用率。

.PIDs             PID 号。

有了这些信息我们就可以完全按照自己的需求或者是偏好来控制 docker stats 命令输出的内容了。

总结
通过 docker stats 命令我们可以看到容器使用系统资源的情况。这为我们进一步的约束容器可用资源或者是调查与资源相关的问题提供了依据。除了 docker 自带的命令，像 glances 等工具也已经支持查看容器使用的资源情况了，有兴趣的朋友可以去了解一下。
————————————————
版权声明：本文为CSDN博主「爱若手握流沙」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/QMW19910301/article/details/88058769