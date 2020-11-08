

vsftpd 是一个 UNIX  类操作系统上运行的服务器的名字，它可以运行在诸如 Linux, BSD, Solaris, HP-UX 以及 IRIX 上面。它支持很多其他的  FTP 服务器不支持的特征。此外，本文还介绍了FTP基本原理，以及FTP用户管理方面的基础知识。 [1] 





- 外文名

  very secure FTP daemon

- 缩  写

  vsftpd

- 特  点

  安全性

- 性  质

  服务器的名字

## 目录

1. 1 [简介](https://baike.baidu.com/item/vsftpd/5254770#1)
2. 2 [特点](https://baike.baidu.com/item/vsftpd/5254770#2)
3. 3 [配置](https://baike.baidu.com/item/vsftpd/5254770#3)

1. 4 [使用技巧](https://baike.baidu.com/item/vsftpd/5254770#4)
2. ▪ [安装](https://baike.baidu.com/item/vsftpd/5254770#4_1)
3. ▪ [文件清单](https://baike.baidu.com/item/vsftpd/5254770#4_2)

1. 5 [版本发布](https://baike.baidu.com/item/vsftpd/5254770#5)



## 简介

[编辑](javascript:;)

[ ![vsftpd](https://bkimg.cdn.bcebos.com/pic/4034970a304e251f2ffd258fae86c9177f3e53bb?x-bce-process=image/resize,m_lfit,w_220,h_220,limit_1) ](https://baike.baidu.com/pic/vsftpd/5254770/0/4034970a304e251f2ffd258fae86c9177f3e53bb?fr=lemma&ct=single) vsftpd

vsftpd 是“very secure FTP daemon”的缩写，安全性是它的一个最大的特点。vsftpd 是一个 UNIX  类操作系统上运行的服务器的名字，它可以运行在诸如 Linux、BSD、Solaris、  HP-UNIX等系统上面，是一个完全免费的、开放源代码的ftp服务器软件，支持很多其他的 FTP  服务器所不支持的特征。比如：非常高的安全性需求、带宽限制、良好的可伸缩性、可创建虚拟用户、支持IPv6、速率高等。 [2] 

vsftpd是一款在Linux发行版中最受推崇的[FTP服务器](https://baike.baidu.com/item/FTP服务器)程序。特点是小巧轻快，安全易用。

在开源操作系统中常用的FTPD套件主要还有ProFTPD、PureFTPd和wuftpd等



## 特点

[编辑](javascript:;)

**①**vsftpd 是以一般身份启动服务，所以对于 Linux 系统的使用权限较低，对于Linux 系统的危害就相对的减低了。此外， vsftpd 亦利用 chroot() 这个函式进行改换根目录的动作，使得系统工具不会被vsftpd 这支服务所误用；

**②**任何需要具有较高执行权限的 vsftpd 指令均以一支特殊的上层程序( parent process ) 所控制 ，该上层程序享有的较高执行权限功能已经被限制的相当的低，并以不影响Linux 本身的系统为准；

**③**所有来自 clients 端，想要使用这支上层程序所提供的较高执行权限之vsftpd 指令的需求，均被视为『不可信任的要求』来处理，必需要经过相当程度的身份确认后，方可利用该上层程序的功能。例如chown(), Login 的要求等等动作；

**④**此外，上面提到的上层程序中，依然使用 chroot() 的功能来限制使用者的执行权限。



## 配置

[编辑](javascript:;)

**vsftpd配置**

关于[主机](https://baike.baidu.com/item/主机)的设定值

connect_from_port_20=YES(NO)

还记得 wuftp 那篇文章提到的，关于主动联机的 ftp-data 吗？

这个设定项目在启动主动联机的port 20 咯！

listen_port=21

使用的 vsftpd命令通道的 port number 设定，如果您想要使用非

正规的 ftpport，在这个设定项目修改吧！

dirmessage_enable=YES(NO)

当使用者进入某个目录时，会显示该目录需要注意的内容，显示的

档案预设是.message ，当然，可以使用底下的设定项目来修订！

message_file=.message

当 dirmessage_enable=YES时，可以设定这个项目来让 vsftpd

寻找该档案来显示讯息！您也可以设定其它档名喔！

listen=YES(NO)

若设定为YES 表示 vsftpd 是以 standalone 的方式来启动的！

pasv_enable=YES(NO)

启动被动式联机(passivemode)，一定要设定为 YES 的啦！

use_localtime=YES(NO)

是否使用[主机](https://baike.baidu.com/item/主机)的时间？！预设使用GMT 时间(格林威治)，会比北京

时间晚 8小时，一般来说，建议设定为 YES 吧！

write_enable=YES(NO)

是否允许使用者具有写入的权限？！这包括删除与修改等功能喔！

connect_timeout=60

单位是秒，如果client 尝试连接我们的 vsftpd 命令通道超过 60 秒，

则不等待，强制断线咯。

accept_timeout=60

当使用者以被动式PASV 来进行数据传输时，如果[主机](https://baike.baidu.com/item/主机)启用 passive port

并等待 client超过60 秒，那么就给他强制断线！您可以修改 60 这个数值。

data_connection_timeout=300

如果 client与 Server 间的[数据传送](https://baike.baidu.com/item/数据传送)在 300 秒内都无法传送成功，

那 Client的联机就会被我们的 vsftpd 强制剔除！

idle_session_timeout=300

如果使用者在300 秒内都没有命令动作，强制离线！

max_clients=0

如果 vsftpd是以 stand alone 方式启动的，那么这个设定项目可以设定

同一时间，最多有多少client 可以同时连上 vsftpd 哩！？

max_per_ip=0

与上面 max_clients类似，这里是同一个 IP 同一时间可允许多少联机？

pasv_max_port=0

pasv_min_port=0

上面两个是与passive mode 使用的 port number 有关，如果您想要使用

65400 到65410 这 11 个 port 来进行被动式资料的连接，可以这样设定

pasv_max_port=65410以及 pasv_min_port=65400

ftpd_banner=一些文字说明

当使用者无法顺利连上我们的主机，例如联机数量已经超过max_clients

的设定了，那么client 的画面就会显示『一些文字说明』的字样，您可以修改

关于实体用户登入者的设定值

guest_enable=YES(NO)

若这个值设定为YES 时，那么任何非 anonymous 登入的账号，均会被

假设成为guest (访客) 喔！

local_enable=YES(NO)

这个设定值必须要为YES 时，在 /etc/passwd 内的账号才能以

实体用户的方式登入我们的vsftpd [主机](https://baike.baidu.com/item/主机)喔！

local_max_rate=0

实体用户的传输速度限制，单位为bytes/second， 0 为不限制。

chroot_local_user=YES(NO)

将使用者限制在自己的家目录之内(chroot)！这个设定在vsftpd

当中预设是NO，因为有底下两个设定项目的辅助喔！

所以不需要启动他！

chroot_list_enable=YES(NO)

是否启用将某些实体用户限制在他们的家目录内？！预设是NO ，

不过，如果您想要让某些使用者无法离开他们的家目录时，

可以考虑将这个设定为YES ，并且规划下个设定值

chroot_list_file=/etc/vsftpd.chroot_list

如果 chroot_list_enable=YES那么就可以设定这个项目了！他里面可以规定

那一个实体用户会被限制在自己的家目录内而无法离开！(chroot)

一行一个账号即可！<= 这个应该是白名单，即写在这个文件中实体用户可以不受家目录限制。

userlist_deny=YES(NO)

若此设定值为YES 时，则当使用者账号被列入到某个档案时，在该档案内

的使用者将无法登入vsftpd 服务器！该档案文件名与下列设定项目有关。

userlist_file=/etc/vsftpd.user_list

若上面 userlist_deny=YES时，则这个档案就有用处了！在这个档案内的

账号都无法使用vsftpd 喔！

关于匿名者登入的设定值

anonymous_enable=YES(NO)

设定为允许anonymous 登入我们的 vsftpd [主机](https://baike.baidu.com/item/主机)！预设是 YES ，底下的所有

相关设定都需要将这个设定为anonymous_enable=YES 之后才会生效！

anon_world_readable_only=YES(NO)

仅允许 anonymous具有下载可读档案的权限，预设是 YES。

anon_other_write_enable=YES(NO)

是否允许anonymous 具有写入的权限？预设是 NO！如果要设定为 YES，

那么开放给anonymous 写入的目录亦需要调整权限，让 vsftpd 的 PID

拥有者可以写入才行！

anon_mkdir_write_enable=YES(NO)

是否让anonymous 具有建立目录的权限？默认值是 NO！如果要设定为 YES，

那么 anony_other_write_enable必须设定为 YES ！

anon_upload_enable=YES(NO)

是否让anonymous 具有上传数据的功能，预设是 NO，如果要设定为 YES ，

则 anon_other_write_enable=YES必须设定。

deny_email_enable=YES(NO)

将某些特殊的email address 抵挡住，不让那些 anonymous 登入！

如果以 anonymous登入[主机](https://baike.baidu.com/item/主机)时，不是会要求输入密码吗？密码不是要您

输入您的email address 吗？如果你很讨厌某些 email address ，

就可以使用这个设定来将他取消登入的权限！需与下个设定项目配合：

banned_email_file=/etc/vsftpd.banned_emails

如果 deny_email_enable=YES时，可以利用这个设定项目来规定那个

email address不可登入我们的 vsftpd 喔！在上面设定的档案内，

一行输入一个email address 即可！

no_anon_password=YES(NO)

当设定为YES 时，表示 anonymous 将会略过密码检验步骤，

而直接进入vsftpd 服务器内喔！所以一般预设都是 NO 的！

anon_max_rate=0

这个设定值后面接的数值单位为bytes/秒 ，限制 anonymous 的传输速度，

如果是 0则不限制(由最大频宽所限制)，如果您想让 anonymous 仅有

30 KB/s 的速度，可以设定『anon_max_rate=30000』

anon_umask=077

限制 anonymous的权限！如果是 077 则 anonymous 传送过来的档案

权限会是-rw------- 喔！

关于系统安全的设定值：

ascii_download_enable=YES(NO)

如果设定为YES ，那么 client 就可以使用 ASCII 格式下载档案。

一般来说，由于启动了这个设定项目可能会导致DoS 的攻击，因此预设是NO。

ascii_upload_enable=YES(NO)

与上一个设定类似的，只是这个设定针对上传而言！预设是NO。

async_abor_enable=YES(NO)

如果您的FTP client 会下达 "async ABOR" 这个指令时，这个设定才需要启用

一般来说，由于这个设定并不安全，所以通常都是将他取消的！

check_shell=YES(NO)

如果您想让拥有任何奇怪的shell 的使用者(在 /etc/passwd 的 shell 字段)

可以使用vsftpd 的话，这个设定可以设定为 NO 喔！

one_process_model=YES(NO)

这个设定项目比较危险一点～当设定为YES 时，表示每个建立的联机

都会拥有一支process 在负责，可以增加 vsftpd 的效能。不过，

除非您的系统比较安全，而且硬件配备比较高，否则容易耗尽系统资源喔！

一般建议设定为NO 的啦！

tcp_wrappers=YES(NO)

当然我们都习惯支持TCP Wrappers 的啦！所以设定为 YES 吧！

xferlog_enable=YES(NO)

当设定为YES 时，使用者上传与下载档案都会被纪录起来。记录档案

与下一个设定项目有关：

xferlog_file=/var/log/vsftpd.log

如果上一个xferlog_enable=YES 的话，这里就可以设定了！

这个是登录档的档名啦！

xferlog_std_format=YES(NO)

是否设定为wu ftp 相同的登录档格式？！预设为 NO ，因为登录档会比较容易读！

不过，如果您有使用wu ftp 登录文件的分析软件，这里才需要设定为 YES

nopriv_user=nobody

我们的 vsftpd预设以 nobody 作为此一服务执行者的权限。因为 nobody 的权限

相当的低，因此即使被入侵，入侵者仅能取得nobody 的权限喔！

pam_service_name=vsftpd

这个是 pam模块的名称，我们放置在 /etc/pam.d/vsftpd 即是这个咚咚！

**vsftpd.conf配置：**

**1、默认配置：**

anonymous_enable=YES #允许匿名用户访问

local_enable=YES #允许[本地用户](https://baike.baidu.com/item/本地用户)访问

write_enable=YES #具有写权限

local_umask=022 #本地用户创建文件或目录的掩码

connect_from_port_20=YES #开启20端口

**2、允许匿名用户具有写权限（上传/创建目录）**

在默认配置下添加以下内容：

anon_upload_enable=YES

anon_mkdir_write_enable=YES

anon_world_readable_only=NO 允许匿名帐号写 另外还需具有所有权限的目录

**3、屏蔽本地所有用户浏览其他目录的权限**（除了home目录，匿名用户本身只能访问home目录）

在默认配置下添加以下内容：

chroot_local_user=YES

**4、屏蔽部分[本地用户](https://baike.baidu.com/item/本地用户)浏览其他目录的权限**

在默认配置下添加以下内容：

chroot_local_user=NO

chroot_list_enable=YES

chroot_list_file=/etc/vsftpd.chroot_list

另外再创建文件/etc/vsftpd.chroot_list，并添加需要屏蔽的用户。

**5、性能选项**

idle_session_timeout=600

data_connection_timeout=120

local_max_rate=50000 　#本地用户的最高速率

anon_max_rate=30000　#匿名用户的最高速率

修改/etc/passwd文件的用户家目录可以改变用户登录的目录

修改/etc/passwd文件的用户的登录shell为/sbin/nologin，则不能用于本地登录，可以用于ftp登录。

/etc/xinetd.d/vsftpd文件的主要内容：（“=”前后有空格）

only_from = 192.168.1.1|192.168.1.0/24 #只接收来至某ip或[网段](https://baike.baidu.com/item/网段)

no_access = 192.168.3.2|192.168.3.0/24 #拒绝接收来至某ip或网段

access_times = 8:00-17:00 #设置访问时间

instances = 200 #设置最大连接数

per_source = 5 #设置每个ip可有几个连接

/etc/vsftpd/vsftpd.conf 主配置文件

/etc/vsftpd.ftpusers 阻止用户访问FTP服务器的用户名称清单

/etc/vsftpd.userlist 控制用户访问FTP服务器的用户名称清单，由/etc/vsftpd/vsftpd.conf中的

userlist_deny参数决这是允许还是拒绝 [3] 



## 使用技巧

[编辑](javascript:;)

**限制不同下载速度**

操作步骤

1.安装vsftp，并启用

2.编辑: sudo vim /etc/vsftpd/vsftpd.conf

(就是对vsftpd进行配置）

可以通过命令：lftp 172.18.176.12 来查看。

如： yu@yu-laptop:/home/ftp$ lftp 172.17.184.24

lftp 172.17.184.24:~> ls (查看)

-rw-r--r-- 1 1000 1000 83643 Jul 12 10:34 023w.jpg

ftp 172.17.184.24:~> bye (退出)

use_config_dir=/etc/vsftpd/[userconf](https://baike.baidu.com/item/userconf)

3.新增/etc/vsftpd/userconf

4./etc/vsftpd/userconf下增加test1

编辑test1

test1 local_max_rate=25000 （下载速度单位为字节 B）

5./etc/vsftpd/userconf下增加test2

编辑test2

test2 local_max_rate=30000

6.service vsftpd restart

**与Tcp_wrapper结合**

1.编辑/etc/vsftpd/vsftpd.conf

tcp_wrapper=yes

2.编辑/etc/hosts.deny

vsftpd:192.168.0 10.0.0 192.168.1.3 :deny

ALL:ALL:ALLOW

3.效果 192.168.0段的和10.0.0[网段](https://baike.baidu.com/item/网段) 及192.168.1.3不能访问当前[ftp服务器](https://baike.baidu.com/item/ftp服务器)。其他地址的可以访问

**并入xinetd 配置方法**

1.编辑/etc/vsftpd/vsftpd.conf

listen=no

2.新增/etc/xinetd.d/vsftpd

service vsftpd

{

disable=no

socket_type=stream

wait=no

user=root

service=/usr/sbin/vsftpd

port=21

log_no_success+=PID HOST DURATION

log_no_failure+=HOST

}

3.service xinetd restart

**vsftpd 设置用户目录**

增加一个用户ftpuser并设置其目录为/opt/ftp：

1 增加组 [groupadd](https://baike.baidu.com/item/groupadd) ftpgroup

2 修改vsftpd.conf

vi /etc/vsftpd/vsftpd.conf

将底下三行

\#chroot_list_enable=YES

\# (default follows)

\#chroot_list_file=/etc/vsftpd/chroot_list

改为

chroot_list_enable=YES

\# (default follows)

chroot_list_file=/etc/vsftpd/chroot_list

3 增加用户ftpuser并设置其目录为/opt/ftp

useradd -g ftpgroup -d /opt/ftp -M ftpuser

4 设置用户口令 passwd ftpuser

5 编辑chroot_list文件:

vi /etc/vsftpd/chroot_list

内容为ftp用户名,每个用户占一行,如：

ftpuser

6 重新启动vsftpd：

/sbin/service vsftpd restart

注：可能需要设置/opt/ftp文件夹的权限：chmod 777 /opt/ftp



### 安装

ubuntu系统如下：

sudo [apt-get](https://baike.baidu.com/item/apt-get) install vsftpd

centos系统如下：

yum -y install vsftpd

**以Tarball 安装**

可自行找寻版本安装。这里以 1.2.0 这一版来安装 vsftpd在 Mandrake 9.0 上面(注：如果是 Red Hat 的系统，原本就有 vsftpd了，所以使用 RPM 安装比较好！至于其它没有提供  vsftpd RPM 档案的 distribution就可以使用 Tarball )1. 下载与[解压缩](https://baike.baidu.com/item/解压缩)：

[root@testroot]# wget \

\> ftp://XX/users/cevans/vsftpd-1.2.0.tar.gz

[root@testroot]#cd /usr/local/src

[root@testroot]# tar -zxvf /root/vsftpd-1.2.0.tar.gz

[root@testroot]#cd vsftpd-1.2.0/

\# 在这个目录下有个INSTALL 与 README 请务必察看喔！

\2. 开始编译与安装

\# vsftpd 预设安装的路径为：

\# 所有可执行档放置在/usr/local/sbin 里面；

\# man page放置在 /usr/local/man/man5 与 /usr/local/man/man8

\# 若 superdaemon 为 xinetd 时，会复制一份启动档案到 /etc/xinetd.d 去！

[root@testvsftpd-1.2.0]#make

\# 编译的过程可能有warning 的讯息，只要不是 Error 就可以不理他！

[root@testvsftpd-1.2.0]# make install

[root@testvsftpd-1.2.0]#cp vsftpd.conf /etc

\# 将 PAM 身份认证模块给他放进去系统里面！

[root@testvsftpd-1.2.0]#cp RedHat/vsftpd.pam /etc/pam.d/vsftpd

\# 建立 ftp这个使用者以及他的家目录：

\# 若本来就存在ftp 这个使用者，那就不需要进行新增！

[root@testvsftpd-1.2.0]# useradd -M ftp -d /var/ftp

[root@testvsftpd-1.2.0]# mkdir -p /var/ftp

[root@testvsftpd-1.2.0]#chown root:root /var/ftp

[root@testvsftpd-1.2.0]#chown 755 /var/ftp

\# 建立 vsftpd需要的特殊目录

[root@testvsftpd-1.2.0]#mkdir -p /usr/share/empty

\3. 如果需要移除时：

\# 如果想要移除vsftp 时，可以这样做

[root@testvsftpd-1.2.0]# rm /usr/local/sbin/vsftpd

[root@testvsftpd-1.2.0]# rm /usr/local/man/man5/vsftpd.conf.5

[root@testvsftpd-1.2.0]# rm /usr/local/man/man8/vsftpd.8

[root@testvsftpd-1.2.0]# rm /etc/xinetd.d/vsftpd

[root@testvsftpd-1.2.0]# rm /etc/vsftpd.conf

\# 因为刚刚安装只有安装这几个档案而已说！所以啦，vsftpd 真的是挺安全的说！

\4. 测试：

\# 先确认一下xinetd.d 有没有问题再说：

[root@testroot]#vi /etc/xinetd.d/vsftpd

service ftp

{

socket_type = stream

wait = no

user = root

server = /usr/local/sbin/vsftpd

log_on_success +=DURATION USERID

log_on_failure +=USERID

nice = 10

**disable = no**

}

[root@testroot]#/etc/rc.d/init.d/xinetd restart

[root@testroot]# ftp localhost

ftp localhost

Connected tolocalhost.

220 (vsFTPd1.2.0)

530 Pleaselogin with USER and PASS.

530 Pleaselogin with USER and PASS.

KERBEROS_V4rejected as an authentication type

Name (localhost:root):anonymous

\# 这样就表示vsftpd 已经可以正确的启动了，不过因为我们还没有设定好

\# /etc/vsftpd.conf，所以会有无法登入的问题！没关系，

\# 等一下设定好就OK 了！

安装的过程真的是很简单，不过， vsftpd.conf  这个档案放置的地点在 RPM与 Tarball 则可能有点不一样，需要给他特别留意呢！例如 Red Hat 9  预设放置在/etc/vsftpd/vsftpd.conf ，而 Tarball 则预设放置在 /etc/vsftpd.conf 里面说！



### 文件清单

/etc/sbin/vsftpd：服务文件。redhat 9.0为/etc/rc.d/init.d/vsftpd

/etc/vsftpd.conf：配置文件。redhat 9.0为/etc/vsftpd/vsftpd.conf

/etc/vsftpd.ftpusers：不能用于ftp登录的用户。

/var/ftp：默认的匿名用户（anonymous或ftp，无密码）登录的目录。



## 版本发布

[编辑](javascript:;)

2011年03月13日，vsftpd 2.3.4 发布，Linux的FTP服务器，该版本修复了一个潜在的 DoS 漏洞 (CVE-2011-0762) ，该漏洞可能导致客户端致使服务器耗尽CPU的处理时间，还包括其他的一些小bug修复。 [4] 

2012年04月10日，vsftpd 3.0.0 正式版发布。 [5] 