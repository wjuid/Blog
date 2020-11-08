# failed to find an available destination > EOF

![img](https://www.stubbornhuang.com/wp-content/themes/Grace8.2CrackedVersion/avatar/3.jpg)[StubbornHuang](https://www.stubbornhuang.com/author/stubbornhuang/) [VPS](https://www.stubbornhuang.com/category/建站/vps/) 2020-01-13 851 [0](https://www.stubbornhuang.com/661/#respond) 3
[百度已收录](http://www.baidu.com/s?wd=failed to find an available destination > EOF)
 本文共832个字，阅读需要3分钟。	 

感谢来自使用操作系统为：other
浏览器为：unknown
IP地址为：172.96.247.223
来源网址为：https://www.stubbornhuang.com/661/的朋友，谢谢您的访问！！！

原创文章，作者：[StubbornHuang](https://www.stubbornhuang.com/author/stubbornhuang/)，如若转载，请注明出处：《failed to find an available destination > EOF》https://www.stubbornhuang.com/661/

使用一键脚本：

> bash <(curl -s -L [https://git.io/](https://git.io/v2ray.sh)[v2ray](https://www.stubbornhuang.com/tag/v2ray/).sh)

配置v2ray的时候经常配置成功，但是在连接的时候报failed to find an available destination > EOF错误，这种情况下一般有两种原因:

# 1 原因1：防火墙相应端口未开

该一键脚本不会自动配置防护墙，这也是根据不同的vps所不同，有的可以开启响应端口，有的不行，所以出现上述问题应该马上检测防火墙中响应的端口是否开启，例如所设置端口为1000，那么使用下面的命令开启端口：

```cpp
firewall-cmd --zone=public --add-port=1000/tcp --permanent
firewall-cmd --zone=public --add-port=1000/udp --permanent
```

C++

然后使用：

```cpp
systemctl restart firewalld.service
```

C++

重启防火墙。
 其他防火墙的操作请参考我的另一篇博文：https://www.stubbornhuang.com/642/

我自己连不上就是这个问题，所以必须要好好检测。

# 2 原因2：客户端服务端时间未同步

可以安装ntp进行时间同步，相关的命令如下：

```cpp
date // 查看系统时间
hwclock // 查看硬件时间
yum -y install ntp ntpdate 安装ntpdate工具
ntpdate cn.pool.ntp.org  设置系统时间与网络时间同步
hwclock --systohc  将系统时间写入硬件时间
timedatectl # 查看系统时间方面的各种状态
timedatectl list-timezones # 列出所有时区
timedatectl set-local-rtc 1 # 将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
timedatectl set-timezone Asia/Shanghai # 设置系统时区为上海
```

C++

也可自行百度配置时间同步。

# 3 重启服务