# container 监控数据

[![李志华](https://pic3.zhimg.com/v2-e69faa2cca75dba6960820ffd2f1f513_xs.jpg)](https://www.zhihu.com/people/lizhihua0925)

[李志华](https://www.zhihu.com/people/lizhihua0925)[](https://www.zhihu.com/question/48510028)



高级开发工程师

目前容器技术不止是docker，而且出现了不少的变种，如rtk、containerd、gvisor等，但原理其实都一样，只是实现上略有差异。要对容器监控，我们就需要掌握容器原理和监控数据的信息，以便多维度获取需要监控的数据。

## **容器技术主要由三个基础：namespace、cgroup、rootfs，共用kernel**

从容器角度看，每个容器都是一个或者多个特殊的隔离的进程，运行在受限的隔离的资源中，并且各自有自己的文件系统；但从操作系统的上帝角度看，每个容器都是一个进程，只不过这种进程拥有自己的特殊的子namespace、cgroup设置和rootfs挂载，所有的信息都在操作系统的掌控之中。

监控数据也是一样，也在操作系统的掌控之下，今天探索容器的监控数据，其实也就是研究操作系统对进程数据的管理和配置信息。

## **容器监控数据：cpu、mem、io、port and connects**

**linux管理进程数据的地方在/proc，其中存放了每个进程的cpu、mem、net、io、cgroup等各种信息，我们所有的监控数据都基于/proc获取**

[https://www.kernel.org/doc/Documentation/filesystems/proc.txt](https://link.zhihu.com/?target=https%3A//www.kernel.org/doc/Documentation/filesystems/proc.txt)

[www.kernel.org](https://link.zhihu.com/?target=https%3A//www.kernel.org/doc/Documentation/filesystems/proc.txt)

**cpu [/proc/pid/stat]**

```text
 # cat /proc/38901/stat
38901 (all-in-one-linu) S 38885 38901 38901 0 -1 1077944576 12824 0 24 0 2396 173 0 0 20 0 16 0 7618753648 144478208 6605 18446744073709551615 4194304 18183813 140731019730784 140731019730352 4591779 0 1006254592 0 2143420159 18446744073709551615 0 0 17 4 0 0 2 0 0 41541632 41986400 66891776 140731019738887 140731019738976 140731019738976 140731019739103 0
```

cpu的计算方式是counter，随时间永远增加，所以计算方式就是根据一段时间内的差值/时间

cpu metrics:

```text
container_cpu_user_seconds_total
container_cpu_system_seconds_total
container_cpu_usage_seconds_total (user+system)
```

prometheus表达式示例：

```text
rate(container_cpu_usage_seconds_total{image=""}[3m])  / ignoring(cpu) container_spec_cpu_quota
```

**mem [/sys/fs/cgroup/docker/memory/xxxxx]**

```text
# cat memory.usage_in_bytes                    # container_memory_usage_bytes 
4294660096
# cat memory.stat
cache 2541522944                               # container_memory_cache
rss 1752494080                                 # container_memory_rss
rss_huge 0
mapped_file 6492160
swap 0
pgpgin 53067055
pgpgout 52018711
pgfault 3071556847
pgmajfault 1007
inactive_anon 0
active_anon 1752489984
inactive_file 1270788096
active_file 1270521856
unevictable 0
hierarchical_memory_limit 4294967296
hierarchical_memsw_limit 4294967296
total_cache 2541522944
total_rss 1752494080
total_rss_huge 0
total_mapped_file 6492160
total_swap 0
total_pgpgin 53067055
total_pgpgout 52018711
total_pgfault 3071556847
total_pgmajfault 1007
total_inactive_anon 0
total_active_anon 1752489984
total_inactive_file 1270788096
total_active_file 1270521856
total_unevictable 0
```

memory metrics

```text
container_memory_cache
container_memory_rss
container_memory_swap
container_memory_usage_bytes (cache+rss+swap,在kernel4.4+，还包括kmem)
container_memory_working_set_bytes   
container_spec_memory_limit_bytes
```

1. container_memory_working_set_bytes = container_memory_usage_bytes  - total_inactive_anon - total_inactive_file 
2. rss = total_inactive_anon + total_active_anon 
3. cache = total_inactive_file + total_active_file 
4. kubelet比较container_memory_working_set_bytes和container_spec_memory_limit_bytes来决定oom container

[https://github.com/google/cadvisor/blob/4a9a57cdd061e241a974602009f26729f719a862/container/libcontainer/helpers.go#L191](https://link.zhihu.com/?target=https%3A//github.com/google/cadvisor/blob/4a9a57cdd061e241a974602009f26729f719a862/container/libcontainer/helpers.go%23L191)[github.com](https://link.zhihu.com/?target=https%3A//github.com/google/cadvisor/blob/4a9a57cdd061e241a974602009f26729f719a862/container/libcontainer/helpers.go%23L191)[qingwave.github.io![图标](https://pic4.zhimg.com/v2-4259206133cb762dfe6ec393804c8e03_ipico.jpg)](https://link.zhihu.com/?target=https%3A//qingwave.github.io/2018/11/15/Pod-memory-usage-in-k8s/%23Cgroup%E4%B8%AD%E5%85%B3%E4%BA%8Emem%E6%8C%87%E6%A0%87)[github.com](https://link.zhihu.com/?target=https%3A//github.com/google/cadvisor/issues/2138)

```text
# cat /proc/38901/io
rchar: 59318
wchar: 1372447
syscr: 162
syscw: 408
read_bytes: 37900800
write_bytes: 0
cancelled_write_bytes: 0
```

io metrics

```text
container_fs_writes_bytes_total  磁盘写总量
container_fs_reads_bytes_total   磁盘读总量
```

**net [/proc/pid/net/[tcp|tcp6]] get port and connects**

```text
# # cat /proc/38901/net/tcp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
   0: 020011AC:412E 21CD810A:F047 03 00000000:00000000 01:00000304 00000003     0        0 0 2 ffff880435842500
   1: 020011AC:412E 21CD810A:F048 03 00000000:00000000 01:00000318 00000003     0        0 0 2 ffff880435842700
   2: 0100007F:9A65 0100007F:37AA 01 00000000:00000000 02:000002CB 00000000     0        0 4261696987 2 ffff88011244a580 20 4 26 10 -1
```

net metrics

```text
container_network_receive_bytes_total
container_network_transmit_bytes_total
container_network_receive_packets_dropped_total
container_network_transmit_packets_dropped_total
container_network_receive_errors_total
container_network_transmit_errors_total
```