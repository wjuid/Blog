

# 1 安装[firewall](https://www.stubbornhuang.com/tag/firewall/)

> yum install firewalld firewalld-config

# 2 firewall[防火墙](https://www.stubbornhuang.com/tag/防火墙/)基本操作

## 2.1 查看防火墙状态

防火墙是否开启或者关闭

> firewall-cmd --state

## 2.2 开启防火墙

> systemctl start firewalld.service

设置防火墙开机自启

> systemctl enable firewalld.service

## 2.3 重启防火墙

> systemctl restart firewalld.service

## 2.4 查看防火墙设置开机自启是否成功

> systemctl is-enabled firewalld.service;echo $?

结果为enable则为设置开机自启成功。

## 2.5 停止防火墙并禁用开机启动

> systemctl disable firewalld

# 3 配置防火墙

## 3.1 查看版本

> firewall-cmd --version

## 3.2 查看帮助

> firewall-cmd --help

## 3.3 显示状态

> firewall-cmd --state

## 3.4 查看所有打开的端口

> firewall-cmd --zone=public --list-ports

## 3.5 更新防火墙规则

> firewall-cmd --reload

## 3.6 查看区域信息

> firewall-cmd --get-active-zones

## 3.7 查看指定接口所属区域

> firewall-cmd --get-zone-of-interface=eth0

## 3.8 拒绝所有包

> firewall-cmd --panic-on

## 3.9 取消拒绝状态

> firewall-cmd --panic-off

## 3.10 查看是否拒绝

> firewall-cmd --query-panic

# 4 firewall防火墙开启特定端口

## 4.1 开启特定端口

在开启防火墙之后，我们有些服务就会访问不到，是因为服务的相关端口没有打开。
 在此以永久打开80端口为例

```cpp
开端口命令：firewall-cmd --zone=public --add-port=80/tcp --permanent
重启防火墙：systemctl restart firewalld.service

命令含义：

--zone #作用域

--add-port=80/tcp  #添加端口，格式为：端口/通讯协议

--permanent   #永久生效，没有此参数重启后失效
```

C++

在使用

> firewall-cmd --zone=public --add-port=80/tcp --permanent

添加端口之后需要使用

> systemctl restart firewalld.service

重启防火墙，使得出入规则生效。

## 4.2 关闭特定端口

关闭80端口

```cpp
firewall-cmd --zone=public --remove-port=80/tcp --permanent
firewall-cmd --zone=public --remove-port=80/udp --permanent
```

C++

## 4.3 查看所有开启的端口

> netstat -ntlp

或者

> firewall-cmd --list-ports

## 4.4 批量添加区间端口

```cpp
firewall-cmd --zone=public --add-port=4400-4600/udp --permanent
firewall-cmd --zone=public --add-port=4400-4600/tcp --permanent
```