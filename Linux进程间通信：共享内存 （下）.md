# Linux进程间通信：共享内存 （下）

 2017-07-27阅读 1.3K0

接[Linux进程间通信:共享内存 （上）](https://www.qcloud.com/community/article/347803)

## POSIX共享内存

POSIX共享内存实际上毫无新意，它本质上就是mmap对文件的共享方式映射，只不过映射的是tmpfs文件系统上的文件。

什么是tmpfs？Linux提供一种“临时”文件系统叫做tmpfs，它可以将内存的一部分空间拿来当做文件系统使用，使内存空间可以当做目录文件来用。现在绝大多数Linux系统都有一个叫做/dev/shm的tmpfs目录，就是这样一种存在。具体使用方法，大家可以参考我的另一篇文章《Linux内存中的Cache真的能被回收么？》。

Linux提供的POSIX共享内存，实际上就是在/dev/shm下创建一个文件，并将其mmap之后映射其内存地址即可。我们通过它给定的一套参数就能猜到它的主要函数shm_open无非就是open系统调用的一个封装。大家可以通过man shm_overview来查看相关操作的方法。使用代码如下：

```javascript
[root@zorrozou-pc0 sharemem]# cat racing_posix_shm.c 
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <errno.h>
#include <fcntl.h>
#include <string.h>
#include <sys/file.h>
#include <wait.h>
#include <sys/mman.h>

#define COUNT 100
#define SHMPATH "shm"

int do_child(char * shmpath)
{
    int interval, shmfd, ret;
    int *shm_p;
    /* 使用shm_open访问一个已经创建的POSIX共享内存 */
    shmfd = shm_open(shmpath, O_RDWR, 0600);
    if (shmfd < 0) {
        perror("shm_open()");
        exit(1);
    }

    /* 使用mmap将对应的tmpfs文件映射到本进程内存 */
    shm_p = (int *)mmap(NULL, sizeof(int), PROT_WRITE|PROT_READ, MAP_SHARED, shmfd, 0);
    if (MAP_FAILED == shm_p) {
        perror("mmap()");
        exit(1);
    }
    /* critical section */
    interval = *shm_p;
    interval++;
    usleep(1);
    *shm_p = interval;
    /* critical section */
    munmap(shm_p, sizeof(int));
    close(shmfd);

    exit(0);
}

int main()
{
    pid_t pid;
    int count, shmfd, ret;
    int *shm_p;

    /* 创建一个POSIX共享内存 */
    shmfd = shm_open(SHMPATH, O_RDWR|O_CREAT|O_TRUNC, 0600);
    if (shmfd < 0) {
        perror("shm_open()");
        exit(1);
    }

    /* 使用ftruncate设置共享内存段大小 */
    ret = ftruncate(shmfd, sizeof(int));
    if (ret < 0) {
        perror("ftruncate()");
        exit(1);
    }

    /* 使用mmap将对应的tmpfs文件映射到本进程内存 */
    shm_p = (int *)mmap(NULL, sizeof(int), PROT_WRITE|PROT_READ, MAP_SHARED, shmfd, 0);
    if (MAP_FAILED == shm_p) {
        perror("mmap()");
        exit(1);
    }

    *shm_p = 0;

    for (count=0;count<COUNT;count++) {
        pid = fork();
        if (pid < 0) {
            perror("fork()");
            exit(1);
        }

        if (pid == 0) {
            do_child(SHMPATH);
        }
    }

    for (count=0;count<COUNT;count++) {
        wait(NULL);
    }

    printf("shm_p: %d\n", *shm_p);
    munmap(shm_p, sizeof(int));
    close(shmfd);
    //sleep(3000);
    shm_unlink(SHMPATH);
    exit(0);
}
```

编译执行这个程序需要指定一个额外rt的库，可以使用如下命令进行编译：

```javascript
[root@zorrozou-pc0 sharemem]# gcc -o racing_posix_shm -lrt racing_posix_shm.c
```

对于这个程序，我们需要解释以下几点：

1. shm_open的SHMPATH参数是一个路径，这个路径默认放在系统的/dev/shm目录下。这是shm_open已经封装好的，保证了文件一定会使用tmpfs。
2. shm_open实际上就是open系统调用的封装。我们当然完全可以使用open的方式模拟这个方法。
3. 使用ftruncate方法来设置“共享内存”的大小。其实就是更改文件的长度。
4. 要以共享方式做mmap映射，并且指定文件描述符为shmfd。
5. shm_unlink实际上就是unlink系统调用的封装。如果不做unlink操作，那么文件会一直存在于/dev/shm目录下，以供其它进程使用。
6. 关闭共享内存描述符直接使用close。

以上就是POSIX共享内存。其本质上就是个tmpfs文件。那么从这个角度说，mmap匿名共享内存、XSI共享内存和POSIX共享内存在内核实现本质上其实都是tmpfs。如果我们去查看POSIX共享内存的free空间占用的话，结果将跟mmap和XSI共享内存一样占用shared和buff/cache，所以我们就不再做这个测试了。这部分内容大家也可以参考《Linux内存中的Cache真的能被回收么？》。

根据以上例子，我们整理一下POSIX共享内存的使用相关方法：

```javascript
#include <sys/mman.h>
#include <sys/stat.h>        /* For mode constants */
#include <fcntl.h>           /* For O_* constants */

int shm_open(const char *name, int oflag, mode_t mode);

int shm_unlink(const char *name);
```

使用shm_open可以创建或者访问一个已经创建的共享内存。上面说过，实际上POSIX共享内存就是在/dev/shm目录中的的一个tmpfs格式的文件，所以shm_open无非就是open系统调用的封装，所以起函数使用的参数几乎一样。其返回的也是一个标准的我呢间描述符。

shm_unlink也一样是unlink调用的封装，用来删除文件名和文件的映射关系。在这就能看出POSIX共享内存和XSI的区别了，一个是使用文件名作为全局标识，另一个是使用key。

映射共享内存地址使用mmap，解除映射使用munmap。使用ftruncate设置共享内存大小，实际上就是对tmpfs的文件进行指定长度的截断。使用fchmod、fchown、fstat等系统调用修改和查看相关共享内存的属性。close调用关闭共享内存的描述符。实际上，这都是标准的文件操作