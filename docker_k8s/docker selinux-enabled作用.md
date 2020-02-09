### docker selinux-enabled作用

一.现象**

在docker中有一个运行选项是selinux-enabled。这个选项的作用是啥？

简而言之，它提供了对docker容器中进程的selinux的控制支持。下面举例说明。

首先按照官方文档的说明在docker的启动参数中加入selinux-enabled的支持，当然，前提是你系统的selinux已经打开(https://docs.docker.com/engine/reference/commandline/dockerd/)。

修改完参数，重新部署完毕后，执行以下指令，这条指令启动一个image为alpine的容器，并且将Host的根目录挂载到容器的/host目录。

```
[wlh@xiaomi ~]$ sudo docker run -ti -v /:/host alpine sh
/ # cd /host/home/
/host/home # ls
ls: can't open '.': Permission denied
/host/home # 
```

这里可以看到容器没有权限访问Host的/home目录。

然后，作为对比，我们再执行一条指令。

```
[wlh@xiaomi ~]$ sudo docker run -ti -v /:/host --security-opt label:disable alpine sh
[sudo] password for wlh: 
/ # cd /host/home/
/host/home # ls
lost+found  wlh
/host/home # 
```

这一次我们可以访问到宿主机的/home目录了。

 

**二. 推理**

那么造成访问权限不同的原因是什么呢？

这是第一次访问被拒绝的容器的sh进程的信息(用ps auxZ|grep sh获取)

```
system_u:system_r:container_t:s0:c157,c808 root 3289 1.0  0.0 1596  4 pts/2    Ss+  15:02   0:00 sh
```

这是第二次访问通过的容器的sh进程的信息(用ps auxZ|grep sh获取)

```
system_u:system_r:spc_t:s0      root      2904  0.0  0.0   1596     4 pts/2    Ss+  14:56   0:00 sh
```

我们可以看到，它们的selinux label是不同的。这些label决定了它们对主机上资源的访问权限。

第二个容器因为有--security-opt label:disable标签，所以其进程的selinux label和docker关闭了selinux-enabled时是一样的。

在如同第一个容器的默认情况下，每次创建一个新的容器，docker会根据`/etc/selinux/targeted/contexts/lxc_contexts`

文件的内容给容器的进程打上相应的label:

 

```
`process = ``"system_u:system_r:container_t:s0"``content = ``"system_u:object_r:virt_var_lib_t:s0"``file = ``"system_u:object_r:container_file_t:s0"``ro_file=``"system_u:object_r:container_ro_file_t:s0"``sandbox_kvm_process = ``"system_u:system_r:svirt_qemu_net_t:s0"``sandbox_kvm_process = ``"system_u:system_r:svirt_qemu_net_t:s0"``sandbox_lxc_process = ``"system_u:system_r:container_t:s0"`
```

 

 这里应该是按照sandbox_lxc_process来打Label的。(process和sandbox_lxc_process的内容是一致的，不确定是哪个)

并且随机生成一个Category，在我们的例子中，Category是c157,c808。因此整个Label为：

```
system_u:system_r:container_t:s0:c157,c808 
docker每创建一个容器，都会给它分配不同的Category，以此来确保容器之间的访问隔离。也就是说容器Ａ无法访问容器Ｂ的资源。
```

 

**三.验证**

回到我们前文举得例子，执行指令验证容器一的label能否访问home目录(sesearch指令可以通过sudo yum install setools-console.x86_64安装)：

宿主机的home目录的selinux label为：

 

```
`system_u:object_r:home_root_t:s0 home`
```

 

 下面我们验证一下selinux的规则对容器访问的隔离。简单说明一下这里指令的含义：

-A表示allow，意思是只输出allow的规则

-s表示的是source，意思是我想要查询的规则的source是container_t

-t表示的是type，意思是我想要查询的规则的object是home_root_t。

连在一起的意思是请返回允许domain 为container_t 的进程访问type为home_root_t的资源的规则。

```
`[root@xiaomi contexts]# sesearch -A -s container_t -t home_root_t |grep read``[root@xiaomi contexts]# sesearch -A -s spc_t -t home_root_t |grep read|grep dirallow files_unconfined_type file_type:dir { add_name append audit_access create execmod execute getattr ioctl link ``lock` `map mounton open quotaon read relabelfrom relabelto remove_name rename reparent rmdir search setattr swapon unlink write };allow named_filetrans_domain home_root_t:dir { add_name getattr ioctl ``lock` `open read remove_name search write };allow named_filetrans_domain mountpoint:dir { add_name getattr ioctl ``lock` `open read remove_name search write };`
```

从中可以看出容器一的lable是不具备读取宿主机home目录的权限的，而容器二的label是具备读取宿主机home目录的权限的。(关于这里为什么grep出来的结果不包含spc_t/home_root_t这里不展开了，和selinux的attribute机制有关，简而言之file_type是一个attribute，它是一组type的集合，其中包含了home_root_t)

因此，容器一访问宿主机的home目录失败了，容器二访问宿主机的home目录成功了。

 

**四.如何确保访问？**

那么问题来了，在一个开启了selinux的docker服务中，如果我们想要创建一个容器，给它挂载一个宿主机的目录，我们如何确保容器对该目录的访问呢?

做法是在容器启动时在需要挂载的目录后面加上z或者Z，例如：

```
`docker run -v /``var``/db:/``var``/db:Z rhel7 /bin/sh`
```

 这里加上大写Ｚ之后，docker服务会将docker容器一模一样的selinux lable打到挂载的目录上

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[wlh@xiaomi ~]$ sudo docker run -ti -v /home/wlh:/host alpine sh
[sudo] password for wlh: 
/ # cd /host/
/host # ls
ls: can't open '.': Permission denied
/host # exit
[wlh@xiaomi ~]$ sudo docker run -ti -v /home/wlh:/host:Z alpine sh
/ # cd /host/
/host # lsfile1 file2
/host # exit
[wlh@xiaomi ~]$ ls -Z /home/
              system_u:object_r:lost_found_t:s0 lost+found  system_u:object_r:container_file_t:s0:c498,c862 wlh
[wlh@xiaomi ~]$
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从中上面的例子可以看出，第一次启动容器没有添加大写Z，一如所料我们没有权限访问/home/wlh目录下的内容。

而加了大写Z后，我们有权限访问了，与此同时/home/wlh的selinux label也被改变了。

```
[root@xiaomi contexts]# sesearch -A -s container_t -t container_file_t |grep read|grep dir
allow container_domain container_file_t:dir { add_name create getattr ioctl link lock map open read relabelfrom relabelto remove_name rename reparent rmdir search setattr unlink write };
allow svirt_sandbox_domain container_file_t:dir { add_name create execmod getattr ioctl link lock map mounton open read relabelfrom relabelto remove_name rename reparent rmdir search setattr unlink write };
```

访问selinux policy,确实有规则负责这个事情。

而加上小写的z的话，情况为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[wlh@xiaomi ~]$ sudo docker run -ti -v /home/wlh:/host:z alpine sh
[sudo] password for wlh: 
/ # cd /host/
/host # ls
file1 file2/host # exit
[wlh@xiaomi ~]$ ls -Z /home/
    system_u:object_r:lost_found_t:s0 lost+found  system_u:object_r:container_file_t:s0 wlh
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

注意它和大写的Z的区别，小写的z没有添加category，这应该意味着如果添加的是小写ｚ，那么该目录可以被多个容器共享。

而如果添加的是大写的Ｚ，那么容器挂载的目录由当前容器独占的，其他容器无法访问。（当然，除非其他容器也用大写Z抢占了该目录。例如容器Ａ用大写Ｚ的方式挂载了/home目录，此时它是可以正常访问的，接着容器Ｂ用大写Ｚ的方式挂在了/home目录，那么/home目录的category会发生变化，由容器A的category转变为容器B的category，此时容器A失去了/home目录的访问权，转而容器Ｂ获得/home目录的访问权）

 

**五.总结**

根据前文的描述，我想selinux-enabled选项的基本功能已经比较明晰了，它主要是利用selinux机制限制docker容器内的进程访问宿主机/其它容器的资源。如果这个选项没有开启，那么容器自身的隔离机制是用户安全的唯一屏障，某些恶意程序在某些情况下可能会突破docker容器本身的资源隔离机制(以rootfs为主)，访问到宿主机的资源。开启了这个选项后，即使恶意程序突破了rootfs的限制进入到宿主机的文件系统中，它所造成的破坏也是有限的，因为selinux机制限制了这个进程对宿主机文件的访问。

 

ref: https://medium.com/lucjuggery/docker-selinux-30-000-foot-view-30f6ef7f621(这个例子简单的说明了如何开启selinux-enabled并且有一个简单的小例子说明selinux-enabled是干什么的)

　　https://prefetch.net/blog/2017/09/30/using-docker-volumes-on-selinux-enabled-servers/（这篇文章主要说明了大写Z和小写v的作用）

　　http://www.projectatomic.io/blog/2015/06/using-volumes-with-docker-can-cause-problems-with-selinux/（同上）