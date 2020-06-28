##  			 [posix 共享内存  shm_open shm_unlink mmap munmap msync](https://www.cnblogs.com/my_life/articles/4532808.html) 			

 			Posted on  2015-05-27 10:53[bw_0927](https://www.cnblogs.com/my_life/)11690[编辑](https://i.cnblogs.com/EditArticles.aspx?postid=4532808)[收藏](javascript:void(0))

http://blog.csdn.net/laojing123/article/details/6109154

 POSIX 为创建、映射、同步和取消共享内存段提供五个入口点：

- **`shm_open()`**：创建共享内存段或连接到现有的已命名内存段。这个系统调用返回一个文件描述符。
- **`shm_unlink()`**：根据（`shm_open()` 返回的）文件描述符，删除共享内存段。实际上，这个内存段直到访问它的所有进程都退出时才会删除，这与在 UNIX 中删除文件很相似。但是，调用 `shm_unlink()` （通常由原来创建共享内存段的进程调用）之后，其他进程就无法访问这个内存段了。
- **`mmap()`**：把共享内存段映射到进程的内存。这个系统调用需要 `shm_open()` 返回的文件描述符，它返回指向内存的指针。（在某些情况下，还可以把一般文件或另一个设备的文件描述符映射到内存。对这些操作的讨论超出了本文的范围；具体方法请查阅操作系统的 `mmap()` 文档。）
- **`munmap()`**：作用与 `mmap()` 相反。
- **`msync()`**：用来让共享内存段与文件系统同步 — 当把文件映射到内存时，这种技术有用。

使用共享内存的过程是，用 `shm_open()` 创建内存段，用 `write()` 或 `ftruncate()` 设置它的大小，用 `mmap()` 把它映射到进程内存，执行其他参与者需要的操作。当使用完时，原来的进程调用 `munmap()` 和 `shm_unlink()`，然后退出。

在共享内存不使用的时候，通过close关闭，和普通文件关闭的接口是同一个。关闭不会删除共享内存文件，即使最后一个打开被close了也不会删除，之后还可以再打开。只有调用shm_unlink才会删除共享内存文件。

 

怎么样？简单吧，就像文件操作一样，打开，映射得到一个指针，对指针操作，然后撤销映射，关闭！

下面给出两段示例代码，以示用来进行无血缘关系的进程间通信

```
`  ``#include `` ``6 #include `` ``7 #include `` ``8 #include `` ``9 #include ``10 #include ``"util.h"``11 ``12 ``void` `error_and_die(``const` `char` `*msg) {``13   ``perror``(msg);``14   ``exit``(EXIT_FAILURE);``15 }``16 ``17 ``int` `main()``18 {``19   ``int` `ret;``20   ``const` `size_t` `region_size = sysconf(_SC_PAGE_SIZE);``21   ``const` `char` `*string = ``"This is a test for share memory!/n"``;``22   ``int` `fd = shm_open(SHARE_MEMORY, O_CREAT | O_TRUNC | O_RDWR, 0666);``23   ``if` `(-1 == fd)``24   {``25     error_and_die(``"shm_open"``);``26   }``27 ``28   ret = ftruncate(fd, region_size);``29   ``if` `(ret)``30   {``31     error_and_die(``"ftruncate"``);``32   }``33 ``34   ``void` `*ptr = mmap(0, region_size, PROT_READ | PROT_WRITE , MAP_SHARED, fd, 0);``35   ``if` `(ptr == MAP_FAILED)``36   {``37     error_and_die(``"mmap"``);``38   }``39 ``40   close(fd);``41   ``//*(int *)ptr = 0x55aa;``42   ``memcpy``(ptr, string, (``strlen``(string) + 1));``43 ``44   sleep(10);``45 ``46   ret = munmap(ptr, region_size);``47   ``if` `(ret)``48     error_and_die(``"munmap"``);``49 ``50   ret = shm_unlink(SHARE_MEMORY);``51   ``if` `(ret)``52     error_and_die(``"shm_unlink"``);``53   ``return` `0;``54 ``55 ``56 }`
```

　　

 

```
`1 #include `` ``2 #include `` ``3 #include `` ``4 #include `` ``5 #include `` ``6 #include `` ``7 #include `` ``8 #include `` ``9 #include `` ``10 #include ``"util.h"`` ``11 `` ``12 ``void` `error_and_die(``const` `char` `*msg) {`` ``13   ``perror``(msg);`` ``14   ``exit``(EXIT_FAILURE);`` ``15 }`` ``16 `` ``17 ``int` `main()`` ``18 {`` ``19   ``int` `ret;`` ``20   ``const` `size_t` `region_size = sysconf(_SC_PAGE_SIZE);`` ``21   ``int` `fd = shm_open(SHARE_MEMORY, O_CREAT | O_RDWR, 0666);`` ``22   ``if` `(-1 == fd)`` ``23   {`` ``24     error_and_die(``"shm_open"``);`` ``25   }`` ``26 `` ``27   ret = ftruncate(fd, region_size);`` ``28   ``if` `(ret)`` ``29   {`` ``30     error_and_die(``"ftruncate"``);`` ``31   }`` ``32 `` ``33   ``void` `*ptr = mmap(0, region_size, PROT_READ | PROT_WRITE , MAP_SHARED, fd, 0);`` ``34   ``if` `(ptr == MAP_FAILED)`` ``35   {`` ``36     error_and_die(``"mmap"``);`` ``37   }`` ``38 `` ``39   close(fd);`` ``40 `` ``41 `` ``42   ``printf``(``"%s"``, (``char` `*)ptr);`` ``43   ret = munmap(ptr, region_size);`` ``44   ``if` `(ret)`` ``45     error_and_die(``"munmap"``);`` ``46 `` ``47   ret = shm_unlink(SHARE_MEMORY);`` ``48   ``if` `(ret)`` ``49     error_and_die(``"shm_unlink"``);`` ``50 `` ``51   ``return` `0;`` ``52 }`
```

　　

代码很简单，唯一注意的地方是，第二个文件中shm_open的时候O_TUNC参数要去掉，否则内存又被截断为0，读不到东西的。

 

=========================

1、概述

　Posix提供了两种在无亲缘关系进程间共享内存区的方法：

（1）内存映射文件：先有open函数打开，然后调用mmap函数把得到的描述符映射到当前进程地址空间中的一个文件（上一篇笔记所用到的就是）。

（2）共享内存区对象：先有shm_open打开一个Posix IPC名字（也可以是文件系统中的一个路径名），然后调用mmap将返回的描述符映射到当前进程的地址空间。

这两种方法都需要调用mmap，差别在于作为mmap的参数之一的描述符的获取手段