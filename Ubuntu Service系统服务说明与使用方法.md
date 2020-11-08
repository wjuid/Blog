# Ubuntu Service系统服务说明与使用方法

date: 2015.07.12; modification:2019.04.15

目录:

- [1 什么是Ubuntu的Service](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#什么是ubuntu的service)
- [2 手动添加一个服务](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#手动添加一个服务)
- [3 service start / stop](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#service-start-stop)
- 4 控制服务的自启动
  - [4.1 说明](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#说明)
  - [4.2 使用update-rc.d添加自启动](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#使用update-rc.d添加自启动)
- 5 Service相关工具和命令
  - [5.1 列出系统所有服务](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#列出系统所有服务)
- 6 杂项
  - [6.1 关闭mysqld服务的开机自启动](http://www.mikewootc.com/wiki/linux/usage/ubuntu_service_usage.html#关闭mysqld服务的开机自启动)

# 1 什么是Ubuntu的Service

网上很多资料说, service就是linux中随开机自启动的, 并且在后台运行的程序. 个人认为, 至少对于Ubuntu来说, 这个说法是不太准确的, 这只不过是一种大家使用上的习俗而已. 起始, Service既不一定在后台运行, 也不一定随开机自启动.

举例说明: 比如我们在终端键入: sudo service apache2 restart来重启apache服务, 本文介绍的Service, 指的是这里的这个service.

那么什么是service呢?

按照man service的说明, service本身是个命令, 这个service命令是用来启动service服务的, 其语法格式为:

```
service SCRIPT COMMAND [OPTIONS]
```

其解释为: service运行一个位于/etc/init.d/下的脚本SCRIPT, 或者是一个位于/etc/init下upstart程序. upstart是ubuntu中用来代替以前的sysvinit的启动程序(笔者猜测可能是由于以前svsvinit中叫做startup, 所以现在较upstart).

本文先介绍/etc/init.d下的服务, 说明一下怎么手动的添加一个服务, 并且让它自启动(如果你需要的话). 本文这是简单并且直观的介绍一下service, 并不一定所有概念都准确, 如果读者想要更加准确的概念和更加全面的方法, 可以网上自己搜, 遍地都是.

# 2 手动添加一个服务

基于上面的解释, 其实添加一个服务很简单, 只需要添加一个脚本到/etc/init.d/并赋予它可执行权限即可. 如:

```
sudo touch /etc/init.d/hello
chmod +x /etc/init.d/hello
```

这是ubuntu就认为有个叫hello的服务了. 可以试试键入sudo service hell 再敲TAB键, 这时候应该就可以tab出来hello了, 这说明系统已经识别出来它是一个服务了. 如果此时报错: hello.service not found, 则可能需要执行一下:

```
sudo update-rc.d hello defaults
```

下面来测试一下, 在hello中加入一行:

```
#!/bin/bash
echo "hello"
```

第一行的"#!/bin/bash"一定要有, 否则有可能会报错.

然后运行命令:

```
sudo service hello start
```

这时便会打印输出hello(如果没有打印可以尝试用sudo systemctl status sss.service查看). 如果hello中的命令为echo "hello" $1, 则会打印hello start. 可见, 我们平时输入的sudo service xxx start中的start, 也就是man中说的COMMAND, 只不过是service传给xxx服务的第一个参数而已.

至此, 我们已经有了一个可以简单显示hello的服务, 但是它不会自动启动, 这就如前文所说的, 服务不一定非要随开机自启动的. 后文会介绍如何添加自启动.

# 3 service start / stop

下面我们介绍如何添加service的start / stop等, 其实很简单, 只需要在上文所建的/etc/init.d/hello加入:

```
case "$1" in
    start)
        echo start
        ;;
    stop)
        echo stop
        ;;
    restart)
        echo restart
        ;;
esac
```

在对应的case中进行想要的工作即可.

# 4 控制服务的自启动

## 4.1 说明

简单的说, 要让服务的自启动, 只需要在/etc/rc{RUNLEVEL}.d/中加入S12ServiceName的软链接, 指向/etc/init.d中对应的脚本(如本文的hello). 这里先且看说明, 稍后会介绍方法而不用手动一个个的添加:

说明:

- S12ServiceName中:
  - 表示该服务随启动自动启动, 如果是K, 则表示Kill(杀死进程);
  - 12表示优先级, 数越小, 越是先执行.
  - ServiceName即服务名, 起始叫什么都行, 真正起作用的是软链接的目标, 不过一般最好与服务同名.
- 其中的RUNLEVEL为系统的运行级别, 一般的linux分8个级别: 0-6和一个'S'级别.
  - 0代表关机(halt);
  - 6代表重启(restart);
  - 1级别是单用户模式(single),
  - 2-5各有不同. 但是在userlinux(包括ubuntu)中2-5级别是毫无差别的.
  - 'S'级别是一个比较特殊的级别, 他应该是先于其他级别运行的级别(这一点有待考证).

这里说明一下, 0-6级别的运行是互斥的, 而不是叠加运行, 也就是说如果进入(move into)4级别, 不是指0-3都要运行, 而只是完成4级别里所规定的服务.

如果要查看系统当前的运行级别可以使用命令:

```
runlevel
```

显示的数字就是当前运行级别, 一般ubuntu桌面版在我们平时使用时进入的应该是level 2.

## 4.2 使用update-rc.d添加自启动

虽然可以按照上文方法来手动添加, 但是更简单的是使用update-rc.d命令来添加. 如:

```
sudo update-rc.d hello defaults
```

如果要删除这个服务, 则:

```
sudo update-rc.d hello remove
```

可以看到, 运行添加时, 终端会显示:

```
update-rc.d: warning: /etc/init.d/hello missing LSB information
update-rc.d: see <http://wiki.debian.org/LSBInitScripts>
 Adding system startup for /etc/init.d/hello ...
   /etc/rc0.d/K20hello -> ../init.d/hello
   /etc/rc1.d/K20hello -> ../init.d/hello
   /etc/rc6.d/K20hello -> ../init.d/hello
   /etc/rc2.d/S20hello -> ../init.d/hello
   /etc/rc3.d/S20hello -> ../init.d/hello
   /etc/rc4.d/S20hello -> ../init.d/hello
   /etc/rc5.d/S20hello -> ../init.d/hello
```

然后就可以看到在上述列表中的各个级别下, 创建了对应的软链接.

remove方法如果/etc/init.d/脚本还存在, 则需要使用-f参数:

```
sudo update-rc.d -f hello remove
```

这样会删除各个软链接, 但是并不会删除/etc/init.d/下的脚本本身.

# 5 Service相关工具和命令

## 5.1 列出系统所有服务

```
service --status-all
```

# 6 杂项

## 6.1 关闭mysqld服务的开机自启动

首先尝试:

```
sudo update-rc.d mysql remove
```

如果不管用, 尝试:

```
sudo vim /etc/init/mysql.conf
将此行注释掉:
start on runlevel [2345]
```

如果还不管用, 尝试命令:

```
sudo systemctl disable mysql
或:
sudo echo "manual" > /etc/init/mysql.override 
```