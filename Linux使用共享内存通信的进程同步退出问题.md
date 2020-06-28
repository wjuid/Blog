# Linux使用共享内存通信的进程同步退出问题

| [日期：2015-04-04] | 来源：Linux社区 				作者：coding-my-life | [字体：[大](javascript:ContentSize(16)) [中](javascript:ContentSize(0)) [小](javascript:ContentSize(12))] |
| ------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
|                    |                                                      |                                                              |

  

　　两个甚至多个进程使用共享内存(shm)通信，总遇到同步问题。这里的“同步问题”不是说进程读写同步问题，这个用信号量就好了。这里的同步问题说的是同步退出问题，到底谁先退出，怎么知道对方退出了。举个例子：进程负责读写数据库Ａ，进程Ｂ负责处理数据。那么进程Ａ得比进程Ｂ晚退出才行，因为要保存进程Ｂ处理完的数据。可是Ａ不知道Ｂ什么时候退出啊。Ａ、Ｂ是无关联的进程，也不知道对方的pid。它们唯一的关联就是读写同一块共享内存。正常情况下，进程Ｂ在共享内存中写个标识：进程Ａ你可以退出了，也是可以的。不过进程Ｂ可能是异常退出，连标识都来不及写。其次，共享内存用来做数据通信的，加这么个标识感觉不太好，有滥用的感觉。

　　采用socket通信没有这个问题，因为进程Ｂ退出怎么也会导致socket断开，哪怕是超时。但shm却没有协议来检测这些行为，如果自己也做一个未免太麻烦。那就从共享内存下手吧。

　　共享内存是由内核来管理的，一个进程删除本身打开的共享内存并不影响另一个进程的共享内存，哪怕都是同一块共享内存。这是因为共享内存在内核中一个引用计数，一个进程使用该共享内存就会导致引用计数加１。如果其中一个进程调用了删除函数，只有这个计数为０才会真正删除共享内存。那么，需要最后才退出的进程检测这个计数就可以了。

　　在System V的共享内存中，创建一个共享内存会初始化一个结构：

struct shmid_ds {
        struct ipc_perm shm_perm;  /* Ownership and permissions */
        size_t     shm_segsz;  /* Size of segment (bytes) */
        time_t     shm_atime;  /* Last attach time */
        time_t     shm_dtime;  /* Last detach time */
        time_t     shm_ctime;  /* Last change time */
        pid_t      shm_cpid;  /* PID of creator */
        pid_t      shm_lpid;  /* PID of last shmat(2)/shmdt(2) */
        shmatt_t    shm_nattch; /* No. of current attaches */
        ...
      };

使用shmctl函数可以读取该结构体，其中的shm_nattch就是使用该共享内存的进程数。

不过，现在有了新的POSIX标准，当然要用新标准了。shm_open创建的共享内存也具有“一个进程删除本身打开的共享内存并不影响另一个进程的共享内存”的特点。可是用shm_open创建的共享内存不再有上面的结构，那么，内核是怎么管理shm_open创建共享内存？？看下面的源码：

/* shm_open - open a shared memory file */

/* Copyright 2002, [Red Hat](https://www.linuxidc.com/topicnews.aspx?tid=10) Inc. */

\#include <sys/types.h>
\#include <sys/mman.h>
\#include <unistd.h>
\#include <string.h>
\#include <fcntl.h>
\#include <limits.h>

int
shm_open (const char *name, int oflag, mode_t mode)
{
 int fd;
 char shm_name[PATH_MAX+20] = "/dev/shm/";

 /* skip opening slash */
 if (*name == '/')
  ++name;

 /* create special shared memory file name and leave enough space to
   cause a path/name error if name is too long */
 strlcpy (shm_name + 9, name, PATH_MAX + 10);

 fd = open (shm_name, oflag, mode);

 if (fd != -1)
  {
   /* once open we must add FD_CLOEXEC flag to file descriptor */
   int flags = fcntl (fd, F_GETFD, 0);

   if (flags >= 0)
    {
     flags |= FD_CLOEXEC;
     flags = fcntl (fd, F_SETFD, flags);
    }

   /* on failure, just close file and give up */
   if (flags == -1)
    {
     close (fd);
     fd = -1;
    }
  }

 return fd;
}

我嚓，这就是创建一个普通的文件啊，只是创建的位置在/dev/shm下（也就是RAM上）。再来看看删除共享内存的函数shm_unlink：

/* shm_unlink - remove a shared memory file */

/* Copyright 2002, Red Hat Inc. */

\#include <sys/types.h>
\#include <sys/mman.h>
\#include <unistd.h>
\#include <string.h>
\#include <limits.h>

int
shm_unlink (const char *name)
{
 int rc;
 char shm_name[PATH_MAX+20] = "/dev/shm/";

 /* skip opening slash */
 if (*name == '/')
  ++name;

 /* create special shared memory file name and leave enough space to
   cause a path/name error if name is too long */
 strlcpy (shm_name + 9, name, PATH_MAX + 10);

 rc = unlink (shm_name);

 return rc;
}

这也只是一个普通的unlink函数。也就是说，POSIX标准的共享内存就是一个文件。所谓的“一个进程删除本身打开的共享内存并不影响另一个进程的共享内存”就相当于你用fstream对象打开了一个文件，然后去文件夹把文件删除了(也就是对文件进行了unlink操作)，可是fstream对象还可以正常读写文件，并没有什么引用计数。这下好了，进程退出时又没法同步了。

　　不过，在linux下怎么会有解决不了的问题呢？解决不了只能说明自己太菜。既然是文件，那就从文件下手。那文件有什么是原子操作，又可以计数的呢。答案：硬链接。比如：

linuxidc@www.linuxidc.com:/dev/shm$ stat abc
 文件："abc"
 大小：4       块：8     IO 块：4096  普通文件
设备：15h/21d  Inode：5743159   硬链接：1
权限：(0664/-rw-rw-r--) Uid：( 1000/   linuxidc)  Gid：( 1000/   linuxidc)
最近访问：2015-01-25 21:27:00.961053098 +0800
最近更改：2015-01-25 21:27:00.961053098 +0800
最近改动：2015-01-25 21:27:00.961053098 +0800
创建时间：-
linuxidc@www.linuxidc.com:/dev/shm$

这个硬链接可以通过fstat函数获取。可是要这样实现的话，意味着需要先创建一块共享内存，每个进程引用的时候需要调用link函数来创建一个硬链接。问题解决了，可是这样会在/dev/shm下多个Ｎ多个文件。这可是RAM啊，虽然现在的服务器都比较牛，但这样做也不太好吧。好吧，还有一个flock文件锁。flock使用LOCK_SH参数多个进程对同一个文件加锁。这样，进程Ｂ初始化共享内存时加锁(可以有多个这样的进程)，在退出(包括异常退出)时解锁。进程Ａ在退出时检测这个锁。当发现无锁时说明可以安全退出了。

　　同步退出的问题基本解决了。来不及写代码去验证，下次吧。

PS：内核unlink时应该也是有计数才知道当前有没有进程打开文件，在什么时候应该删除文件。这个还得去查资料，看用不用得上。另外lsof这个工具是可以检测到所有打开该共享内存的进程及相应的状态。这个应该也是有对应的api的，只是现在还没搞懂