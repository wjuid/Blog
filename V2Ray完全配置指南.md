# V2Ray完全配置指南

[ 2018年3月20日](https://ailitonia.com/archives/v2ray完全配置指南/) [Ailitonia](https://ailitonia.com/archives/author/ailitonia/) 

 [小本本](https://ailitonia.com/topics/notebook/) 

 [32 Comments](https://ailitonia.com/archives/v2ray完全配置指南/#comments) 

[V2Ray](https://ailitonia.com/tags/v2ray/)

本文参考于[V2Ray官网](https://www.v2ray.com/)和[V2Ray 配置指南](https://guide.v2fly.org/)



本文更新于2019年3月27日，对应版本v4.18.0，请注意文章时效性。



## 兼容性保证

- 配置文件向后兼容至少一个大版本，即 V2Ray 4.x 可以正常加载 3.x 的配置文件。
- 所有基于 Protobuf 的通信协议，如 Api，向后兼容至少一个大版本。
- 所有基于二进制的通信协议，如 Shadowsocks 和 VMess。当服务器版本不低于客户端版本时，保持永久兼容；当客户端版本超过服务器版本时，保持至少 12 个小版本的兼容性。

# **下载安装**

## 平台支持

V2Ray 在以下平台中可用：

- Windows 7 及之后版本（x86 / amd64）；
- Mac OS X 10.10 Yosemite 及之后版本（amd64）；
- Linux 2.6.23 及之后版本（x86 / amd64 / arm / arm64 / mips64 / mips）；
  - 包括但不限于 Debian 7 / 8、Ubuntu 12.04 / 14.04 及后续版本、CentOS 6 / 7、Arch Linux；
- FreeBSD (x86 / amd64)；
- OpenBSD (x86 / amd64)；
- Dragonfly BSD (amd64)；

## 硬件要求

至少 32MB 空闲内存，推荐 64MB 或更多。

## 下载 V2Ray

预编译的压缩包可以在如下几个站点找到：

1. Github Release: [github.com/v2ray/v2ray-core](https://github.com/v2ray/v2ray-core/releases)
2. Github 分流: [v2ray.com/download](https://www.v2ray.com/download/)
3. Homebrew: [github.com/v2ray/homebrew-v2ray](https://github.com/v2ray/homebrew-v2ray)
4. Arch Linux: [packages/community/x86_64/v2ray/](https://www.archlinux.org/packages/community/x86_64/v2ray/)
5. Snapcraft: [snapcraft.io/v2ray-core](https://snapcraft.io/v2ray-core)

压缩包均为 zip 格式，找到对应平台的压缩包，下载解压即可使用。

## 验证安装包

V2Ray 提供两种验证方式：

1. 安装包 zip 文件的 SHA1 / SHA256 摘要，在每个安装包对应的.dgst文件中可以找到。
2. 可运行程序（v2ray 或 v2ray.exe）的 gpg 签名，文件位于安装包中的 v2ray.sig 或 v2ray.exe.sig。签名公钥可以[在代码库中](https://raw.githubusercontent.com/v2ray/v2ray-core/master/release/verify/official_release.asc)找到。

## Windows 和 Mac OS 安装方式

通过上述方式下载的压缩包，解压之后可看到 v2ray 或 v2ray.exe。直接运行即可。

## Linux 安装脚本

V2Ray 提供了一个在 Linux 中的自动化安装脚本。这个脚本会自动检测有没有安装过 V2Ray，如果没有，则进行完整的安装和配置；如果之前安装过 V2Ray，则只更新 V2Ray 二进制程序而不更新配置。

以下指令假设已在 su 环境下，如果不是，请先运行 sudo su。

运行下面的指令下载并安装 V2Ray。当 yum 或 apt-get 可用的情况下，此脚本会自动安装 unzip 和 daemon。这两个组件是安装 V2Ray 的必要组件。如果你使用的系统不支持 yum 或 apt-get，请自行安装 unzip 和 daemon。

```
bash <(curl -L -s https://install.direct/go.sh)
```

 

此脚本会自动安装以下文件：

```
/usr/bin/v2ray/v2ray：V2Ray 程序；
/usr/bin/v2ray/v2ctl：V2Ray 工具；
/etc/v2ray/config.json：配置文件；
/usr/bin/v2ray/geoip.dat：IP 数据文件
/usr/bin/v2ray/geosite.dat：域名数据文件
```

 

此脚本会配置自动运行脚本。自动运行脚本会在系统重启之后，自动运行 V2Ray。目前自动运行脚本只支持带有 Systemd 的系统，以及 Debian / Ubuntu 全系列。

运行脚本位于系统的以下位置：

- /etc/systemd/system/v2ray.service: Systemd
- /etc/init.d/v2ray: SysV

脚本运行完成后，你需要：

1. 编辑 /etc/v2ray/config.json 文件来配置你需要的代理方式；
2. 运行 service v2ray start 来启动 V2Ray 进程；
3. 之后可以使用 service v2ray start|stop|status|reload|restart|force-reload 控制 V2Ray 的运行。

## go.sh 参数

go.sh 支持如下参数，可在手动安装时根据实际情况调整：

- -p 或 –proxy: 使用代理服务器来下载 V2Ray 的文件，格式与 curl 接受的参数一致，比如 “socks5://127.0.0.1:1080” 或 “http://127.0.0.1:3128“。
- -f 或 –force: 强制安装。在默认情况下，如果当前系统中已有最新版本的 V2Ray，go.sh 会在检测之后就退出。如果需要强制重装一遍，则需要指定该参数。
- –version: 指定需要安装的版本，比如 “v1.13“。默认值为最新版本。
- –local: 使用一个本地文件进行安装。如果你已经下载了某个版本的 V2Ray，则可通过这个参数指定一个文件路径来进行安装。

示例：

- 使用地址为 127.0.0.1:1080 的 SOCKS 代理下载并安装最新版本：./go.sh -p socks5://127.0.0.1:1080
- 安装本地的 v1.13 版本：./go.sh –version v1.13 –local /path/to/v2ray.zip

## Docker

V2Ray 提供了两个预编译的 Docker image：

- [v2ray/official](https://hub.docker.com/r/v2ray/official/): 包含最新发布的版本，每周跟随新版本更新；
- [v2ray/dev](https://hub.docker.com/r/v2ray/dev/): 包含由最新的代码编译而成的程序文件，随代码库更新；

两个 image 的文件结构相同：

- /etc/v2ray/config.json: 配置文件
- /usr/bin/v2ray/v2ray: V2Ray 主程序
- /usr/bin/v2ray/v2ctl: V2Ray 辅助工具
- /usr/bin/v2ray/geoip.dat: IP 数据文件
- /usr/bin/v2ray/geosite:dat: 域名数据文件

------

# **命令行参数**

## V2Ray

V2Ray 的程序文件的命令行参数如下：

```
v2ray [-version] [-test] [-config=config.json] [-format=json]
```

 

其中：

- -version: 只输出当前版本然后退出，不运行 V2Ray 主程序。
- -test: 测试配置文件有效性，如果有问题则输出错误信息，不运行 V2Ray 主程序。
- -config: 配置文件路径，可选的形式如下:
  - 本地路径，可以是一个绝对路径，或者相对路径。
  - “stdin:“: 表示将从标准输入读取配置文件内容，调用者必须在输入完毕后关闭标准输入流。
  - 以http://或https://(均为小写)开头: V2Ray 将尝试从这个远程地址加载配置文件。
- -format: 配置文件格式，可选的值有：
  - json: JSON 格式；
  - pb 或 protobuf: Protobuf 格式；



当-config没有指定时，V2Ray 将先后尝试从以下路径加载config.json:

- 工作目录（Working Directory）
- 环境变量中v2ray.location.asset所指定的路径

## V2Ctl

V2Ctl 命令行参数如下：

```
v2ctl <command> <options>
```

 

其中：

- **Command**: 子命令，有以下选项:
  - api: 调用 V2Ray 进程的远程控制指令。
  - config: 从标准输入读取 JSON 格式的配置，然后从标准输出打印 Protobuf 格式的配置。
  - cert: 生成 TLS 证书。
  - fetch: 抓取远程文件。
  - tlsping: (V2Ray 4.17+) 尝试进行 TLS 握手。
  - verify: 验证文件是否由 Project V 官方签名。
  - uuid: 输出一个随机的 UUID。

## V2Ctl Api

```
v2ctl api [--server=127.0.0.1:8080] <Service.Method> <Request>
```

 

调用 V2Ray 进程的远程控制指令。示例：

```
v2ctl api --server=127.0.0.1:8080 LoggerService.RestartLogger ''
```

 

## V2Ctl Config

```
v2ctl config
```

 

此命令没有参数。它从标准输入读取 JSON 格式的配置，然后从标准输出打印 Protobuf 格式的配置。

## V2Ctl Cert

```
v2ctl cert [--ca] [--domain=v2ray.com] [--expire=240h] [--name="V2Ray Inc"] [--org="V2Ray Inc] [--json] [--file=v2ray]
```

 

生成一个 TLS 证书。

- –ca
  - 如果指定此选项，将会生成一个 CA 证书。
- –domain
  - 证书的 Alternative Name 项。该参数可以多次使用，来指定多个域名。比如–domain=v2ray.com –domain=v2ray.cool。
- –expire
  - 证书有效期。格式为 Golang 的时间长度。
- –name
  - 证书的 Command Name 项。
- –org
  - 证书的 Orgnization 项。
- –json
  - 将生成的证书以 V2Ray 支持的 JSON 格式输出到标准输出。默认开启。
- –file
  - 将证书以 PEM 格式输出到文件。当指定 –file=a 时，将生成 a_cert.pem 和 a_key.pem 两个文件。

## V2Ctl Fetch

```
v2ctl fetch <url>
```

 

抓取指定的 URL 的内容并输出，只支持 HTTP 和 HTTPS。

## V2Ctl TlsPing

```
v2ctl tlsping <domain> --ip=[ip]
```

 

向指定的域名发起 TLS 握手。

- domain
  - 目标域名
- –ip
  - 此域名的 IP 地址。如果未指定此参数，V2Ctl 将使用系统的 DNS 进行域名解析。

 

## Verify

```
v2ctl verify [--sig=/path/to/sigfile] <filepath>
```

 

此命令用于验证一个文件是否由 Project V 官方签名。

- -sig
  - 签名文件路径，默认值为待验证文件加入’.sig’后缀。
- filepath
  - 待验证文件路径。

## V2Ctl UUID

```
v2ctl uuid
```

 

此命令没有参数。每次运行都会输出一个新的 UUID。

------

# **配置文件**

V2Ray 本身使用基于 [Protobuf](https://developers.google.com/protocol-buffers/) 的配置。由于 Protobuf 的文本格式不方便阅读，V2Ray 同时也支持 JSON 格式的配置。在运行之前，V2Ray 会自动将 JSON 转换为对应的 Protobuf。换言之，V2Ray 将来也可能会支持其它格式的配置。

以下介绍一下基于 JSON 格式的配置。

JSON，全称 [JavaScript Object Notation](https://en.wikipedia.org/wiki/JSON)，简而言之是 Javascript 中的对象（Object）。一个 JSON 文件包含一个完整的对象，以大括号“{”开头，大括号“}”结束。

一个 JSON 对象包含一系列的键值对（Key-Value Pair），一个键是一个字符串（String），而值有多种类型，常见的有字符串（String）、数字（Number）、布尔（Bool）、数组（Array）和对象（Object）。下面是一个简单的 JSON 对象示例：

```
{
  "stringValue": "This is a string.",
  "numberValue": 42,
  "boolValue": true,
  "arrayValue": ["this", "is", "a", "string", "array"],
  "objectValue": {
    "another": "object"
  }
}
```

## JSON 数据类型

这里介绍一下常用的数据类型，在之后其它的配置中会用到。

```
boolean: true | false
#布尔值，只有true和false两种取值，不带引号。

number
#数字，在 V2Ray 的使用中通常为非负整数，即0、53…… 数字在 JSON 格式中不带引号。

string
#字符串，由引号包含的一串字符，如无特殊说明，字符的内容不限。

array: []
#数组，由方括号包含的一组元素，如字符串数组表示为[string]。

object: {}
#对象，一组键值对。样例见本文上方的示例。
```

● 通常一个键值对的后面需要有一个逗号”,”，但如果这个键值对后面紧跟一个大括号”｝”的话，则一定不能有逗号。

## V2Ray 常用数据类型

```
map: object {string:string}
#一组键值对，其类型在括号内指出。每一个键和值的类型对应相同。

address: string
#字符串，表示一个 IP 地址或域名，形如："8.8.8.8" 或 "www.v2ray.com"

address_port: string
#字符串，表示一个地址和端口，常见的形式如："8.8.8.8:53"，或者 "www.v2ray.com:80"。在一部分配置中，地址部分可以省略，如":443"。
```

## 配置文件格式

请先注意：

- JSON 所有标点符号都要用半角符号（英文符号）
- 所有字符串都要加双引号 ” “，键是字符串，所以键也要加双引号，数字不用加双引号
- 布尔类型也不用加双引号，布尔值只有两个就是 true 和 false，意思就是真和假
- 对象没有顺序，即大括号 {} 括起来的内容顺序是怎么样都没关系

V2Ray 的配置文件形式如下，客户端和服务器通用一种形式，只是实际的配置不一样。

```
{
  "log": {},
  "api": {},
  "dns": {},
  "stats": {},
  "routing": {},
  "policy": {},
  "reverse": {},
  "inbounds": [],
  "outbounds": [],
  "transport": {}
}
```

● V2Ray 的 JSON 格式支持注释，可使用“//”或者“/* */”来进行注释。在不支持注释的编辑器中可能被显示为“错误”，但实际上是可以正常使用的。



在v2ray的配置文件中：

- log: LogObject，日志配置。表示 V2Ray 如何输出日志，详见[日志配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#log)；
- api: ApiObject，内置的远程控置 API，详见[API 配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#api)；
- dns: DnsObject，内置的 DNS 服务器，若此项不存在，则默认使用本机的 DNS 设置。详见[DNS 配置（dns）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#dns)；
- routing: RoutingObject，路由配置，详见[路由配置（routing）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#routing)；
- policy：PolicyObject，本地策略可进行一些权限相关的配置，详见[本地策略](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#policy)；
- inbounds: [InboundObject]，传入连接配置，一个数组，每个元素是一个入站连接配置。详见[入站连接配置（inbound）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#inbound)；
  - 具体配置参见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)；
- outbounds: [OutboundObject]，传出连接配置，一个数组，每个元素是一个出站连接配置。列表中的第一个元素作为主出站协议。当路由匹配不存在或没有匹配成功时，流量由主出站协议发出。详见[出站连接配置（outbound）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#outbound)；
  - 具体配置参见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)；
- inboundDetour: 【旧版（3.x）配置，在新版（4+）中已被取消】额外的传入连接配置，详见[额外的传入连接配置（inbound detour）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#extrin)；
- outboundDetour: 【旧版（3.x）配置，在新版（4+）中已被取消】额外的传出连接配置，详见[额外的传出连接配置（outbound detour）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#extrout)；
- transport: TransportObject，底层传输配置，用于配置 V2Ray 如何与其它服务器建立和使用网络连接。详见[底层传输配置（transport）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#transport)；
  - [TCP 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TCP)
  - [mKCP 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#mKCP)
  - [WebSocket 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#WebSocket)
  - [HTTP/2 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTP2)
  - [DomainSocket 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#domainsocket)
  - [QUIC 传输方式](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#quic)
- stats: StatsObject，当此项存在时，开启[统计信息](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#stats)。
- reverse: ReverseObject，[反向代理配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#reverse)。
- [环境变量](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#env)：V2Ray 的一些底层配置。





## 配置实例：

- [配置实例](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#eg)
- 高级功能
  - [均衡负载](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#balance)
  - [动态端口](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#dynport)
  - [代理转发](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#outboundproxy)
  - [TLS](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#tls)
  - [WebSocket+TLS+Web](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#WTW)
  - [CDN](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#cdn)
  - [反向代理](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#reverse_eg1)

------



# **日志配置**

配置如下：

```
{
  "access": "文件地址",
  "error": "文件地址",
  "loglevel": "warning"
}
```

 

其中：

- **access**: string
  - 访问日志的文件地址，其值是一个合法的文件地址，如”/tmp/v2ray/_access.log”（Linux）或者”C:\\Temp\\v2ray\\_access.log”（Windows）。当此项不指定或为空值时，表示将日志输出至 stdout。
- **error**: string
  - 错误日志的文件地址，其值是一个合法的文件地址，如”/tmp/v2ray/_error.log”（Linux）或者”C:\\Temp\\v2ray\\_error.log”（Windows）。当此项不指定或为空值时，表示将日志输出至 stdout。
- **loglevel**: “debug” | “info” | “warning” | “error” | “none”
  - 错误日志的级别。默认值为”warning”。
    - “debug“: 只有开发人员能看懂的信息。同时包含所有”info”内容。
    - “info“: V2Ray 在运行时的状态，不影响正常使用。同时包含所有”warning”内容。
    - “warning“: V2Ray 遇到了一些问题，通常是外部问题，不影响 V2Ray 的正常运行，但有可能影响用户的体验。同时包含所有”error”内容。
    - “error“: V2Ray 遇到了无法正常运行的问题，需要立即解决。
    - “none“: 不记录任何内容。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **远程控制**

V2Ray 中可以开放一些 API 以便远程调用。这些 API 都基于 [gRPC](https://grpc.io/)。

当远程控制开启时，当远程控制开启时，V2Ray 会自建一个出站代理，以tag配置的值为标识。用户必须手动将所有的 gRPC 入站连接通过[路由](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#routing)指向这一出站代理。

## ApiObject

ApiObject对应配置文件中的api项。

```
{
    "tag": "api",
    "services": [
        "HandlerService",
        "LoggerService",
        "StatsService"
    ]
}
```

 

其中：

- **tag**: string
  - 传出代理标识。
- **services**: [string]
  - 开启的 API 列表，可选的值见[API 列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#api-list)。

## 支持的 API 列表

### HandlerService

一些对于传入传出代理进行修改的 API，可用的功能如下：

- 添加一个新的传入代理；
- 添加一个新的传出代理；
- 删除一个现有的传入代理；
- 删除一个现有的传出代理；
- 在一个传入代理中添加一个用户（仅支持 VMess）；
- 在一个传入代理中删除一个用户（仅支持 VMess）；

### LoggerService

支持对内置 Logger 的重启，可配合 logrotate 进行一些对日志文件的操作。

### StatsService

内置的数据统计服务，详见[统计信息](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#stats)。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **DNS 服务器**

V2Ray 内置了一个 DNS 服务器，可以将 DNS 查询根据路由设置转发到不同的远程服务器中。由此 DNS 服务器所发出的 DNS 查询请求，会自动根据路由配置进行转发，无需额外配置。若此项不存在，则默认使用本机的 DNS 设置。

● 由于 DNS 协议的复杂性，V2Ray 只支持最基本的 IP 查询（A 和 AAAA 记录）。推荐使用本机 DNS 配合一个额外的 DNS 服务器来做 DNS 查询，如[CoreDNS](https://coredns.io/)，以使用完整的 DNS 功能。

## DnsObject

DnsObject对应配置文件中的dns项。

```
{
  "hosts": {
    "baidu.com": "127.0.0.1"
  },
  "servers": [
      {
        "address": "1.2.3.4",
        "port": 5353,
        "domains": [
          "domain:v2ray.com"
        ],
      },
      "8.8.8.8",
      "8.8.4.4",
      "localhost"
  ],
  "clientIp": "1.2.3.4",
  "tag": "dns_inbound"
}
```

 

其中：

- **hosts**: map{string: address}
  - 静态 IP 列表，其值为一系列的”域名”:”IP”。IP 可以是 IPv4 或者 IPv6。在解析域名时，如果域名匹配这个列表中的某一项，则解析结果为该项的 IP，而不会使用下述的 servers 进行解析。
  - 域名的格式和[路由](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#routing)中的域名格式一样。
- **servers**: [string | [ServerObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ServerObject_DnsObject) | “localhost” ]
  - 指定一个 DNS 服务器列表，支持的类型有三种：IP 地址（字符串形式），ServerObject，或者”localhost”。
  - 当它的值是一个 IP 地址时，如”8.8.8.8″，V2Ray 会使用此地址的 53 端口进行 DNS 查询。
  - 当值为”localhost”时，表示使用本机预设的 DNS 配置。

● 当使用 localhost 时，本机的 DNS 请求不受 V2Ray 控制，需要额外的配置才可以使 DNS 请求由 V2Ray 转发。

- **clientIp**: string
  - 当前系统的 IP 地址，用于 DNS 查询时，通知服务器客户端的所在位置。不能是私有地址。
- **tag**: string
  - (V2Ray 4.13+) 由此 DNS 发出的查询流量，除localhost外，都会带有此标识，可在路由使用inboundTag进行匹配。

## ServerObject

```
{
  "address": "1.2.3.4",
  "port": 5353,
  "domains": [
    "domain:v2ray.com"
  ]
}
```

 

其中：

- **address**: address
  - DNS 服务器地址，如”8.8.8.8“。目前只支持 UDP 协议的 DNS 服务器。
- **port**: number
  - DNS 服务器端口，如53。
- **domains**: [string]
  - 一个域名列表，此列表包含的域名，将优先使用此服务器进行查询。域名格式和[路由配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#routing)中相同

若要使 DNS 服务生效，需要配置路由功能中的 domainStrategy。

当某个 DNS 服务器指定的域名列表匹配了当前要查询的域名，V2Ray 会优先使用这个 DNS 服务器进行查询，否则按从上往下的顺序进行查询。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **路由功能**

V2Ray 内建了一个简单的路由功能，可以将传入数据按需求由不同的传出连接发出，以达到按需代理的目的。这一功能的常见用法是分流国内外流量，V2Ray 可以通过内部机制判断不同地区的流量，然后将它们发送到不同的传出代理。

## RoutingObject

RoutingObject 对应主配置文件中的routing项。

```
{
  "domainStrategy": "AsIs",
  "rules": [],
  "balancers": []
}
```

 

其中：

- **domainStrategy:** “AsIs” | “IPIfNonMatch” | “IPOnDemand”
  - 域名解析策略，根据不同的设置使用不同的策略。
    - “AsIs“: 只使用域名进行路由选择。默认值。
    - “IPIfNonMatch“: 当域名没有匹配任何规则时，将域名解析成 IP（A 记录或 AAAA 记录）再次进行匹配；
      - 当一个域名有多个 A 记录时，会尝试匹配所有的 A 记录，直到其中一个与某个规则匹配为止；
      - 解析后的 IP 仅在路由选择时起作用，转发的数据包中依然使用原始域名；
    - “IPOnDemand“: 当匹配时碰到任何基于 IP 的规则，将域名立即解析为 IP 进行匹配；
- **rules**: [[RuleObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#RuleObject)]
  - 对应一个数组，数组中每个元素是一个规则。对于每一个连接，路由将根据这些规则依次进行判断，当一个规则生效时，即将这个连接转发至它所指定的outboundTag(或balancerTag，V2Ray 4.4+)。当没有匹配到任何规则时，流量默认由主出站协议发出。
- **balancers**: [ [BalancerObject ](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#BalancerObject)]
  - (V2Ray 4.4+)一个数组，数组中每个元素是一个负载均衡器的配置。当一个规则指向一个负载均衡器时，V2Ray 会通过此负载均衡器选出一个出站协议，然后由它转发流量。

旧版（3.x）配置已失效并被折叠，新版（4+）请忽略此内容

## RuleObject

```
{
  "type": "field",
  "domain": [
    "baidu.com",
    "qq.com",
    "geosite:cn"
  ],
  "ip": [
    "0.0.0.0/8",
    "10.0.0.0/8",
    "fc00::/7",
    "fe80::/10",
    "geoip:cn"
  ],
  "port": "0-100",
  "network": "tcp",
  "source": [
    "10.0.0.1"
  ],
  "user": [
    "love@v2ray.com"
  ],
  "inboundTag": [
    "tag-vmess"
  ],
  "protocol":["http", "tls", "bittorrent"],
  "attrs": "attrs[':method'] == 'GET'",
  "outboundTag": "direct",
  "balancerTag": "balancer"
}
```

● 当多个属性同时指定时，这些属性需要同时满足，才可以使当前规则生效。如果多个规则分别使用了domain或者ip，需要对应添加多条规则。

其中：

- **type**: “field”
  - 目前只支持”field“这一个选项。
- **domain**: [string]
  - 一个数组，数组每一项是一个域名的匹配。有以下几种形式：
    - 纯字符串: 当此字符串匹配目标域名中任意部分，该规则生效。比如”sina.com”可以匹配”sina.com”、”sina.com.cn”和”www.sina.com”，但不匹配”sina.cn”。
    - 正则表达式: 由”regexp:“开始，余下部分是一个正则表达式。当此正则表达式匹配目标域名时，该规则生效。例如”regexp:\\.goo.*\\.com$”匹配”www.google.com”、”fonts.googleapis.com”，但不匹配”google.com”。
    - 子域名 (推荐): 由”domain:“开始，余下部分是一个域名。当此域名是目标域名或其子域名时，该规则生效。例如”domain:v2ray.com”匹配”www.v2ray.com”、”v2ray.com”，但不匹配”xv2ray.com”。
    - 完整匹配: 由”full:“开始，余下部分是一个域名。当此域名完整匹配目标域名时，该规则生效。例如”full:v2ray.com”匹配”v2ray.com”但不匹配”www.v2ray.com”。
    - 预定义域名列表：由”geosite:“开头，余下部分是一个名称，如geosite:google或者geosite:cn。名称及域名列表参考[预定义域名列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#dnsdlc)。
    - 从文件中加载域名: 形如”ext:file:tag“，必须以ext:（小写）开头，后面跟文件名和标签，文件存放在[资源目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#env)中，文件格式与geosite.dat相同，标签必须在文件中存在。
- **ip**: [string]
  - 一个数组，数组内每一个元素代表一个 IP 范围。当某一元素匹配目标 IP 时，此规则生效。有以下几种形式：
    - IP: 形如”127.0.0.1“。
    - [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing): 形如”10.0.0.0/8“.
    - GeoIP: 形如”geoip:cn“，必须以geoip:（小写）开头，后面跟双字符国家代码，支持几乎所有可以上网的国家。
      - 特殊值：”geoip:private” (V2Ray 3.5+)，包含所有私有地址，如127.0.0.1。
    - 从文件中加载 IP: 形如”ext:file:tag“，必须以ext:（小写）开头，后面跟文件名和标签，文件存放在资源目录中，文件格式与geoip.dat相同标签必须在文件中存在。

● “ext:geoip.dat:cn“等价于”geoip:cn“

- **port**: number | string
  - 端口范围，有两种形式：
    - “a-b“: a 和 b 均为正整数，且小于 65536。这个范围是一个前后闭合区间，当目标端口落在此范围内时，此规则生效。
    - a: a 为正整数，且小于 65536。当目标端口为 a 时，此规则生效。
    - (V2Ray 4.18+) 以上两种形式的混合，以逗号”,”分隔。形如：”53,443,1000-2000″。
- **network**: “tcp” | “udp” | “tcp,udp”
  - 可选的值有”tcp“、”udp“或”tcp,udp“，当连接方式是指定的方式时，此规则生效。
- **source**: [string]
  - 一个数组，数组内每一个元素是一个 IP 或 CIDR。当某一元素匹配来源 IP 时，此规则生效。
- **user**: [string]
  - 一个数组，数组内每一个元素是一个邮箱地址。当某一元素匹配来源用户时，此规则生效。当前 Shadowsocks 和 VMess 支持此规则。
- **inboundTag**: [string]
  - 一个数组，数组内每一个元素是一个标识。当某一元素匹配入站协议的标识时，此规则生效。
- **protocol**: [ “http” | “tls” | “bittorrent” ]
  - 一个数组，数组内每一个元素表示一种协议。当某一个协议匹配当前连接的流量时，此规则生效。必须开启入站代理中的sniffing选项。
- **attrs**: string
  - (V2Ray 4.18+) 一段脚本，用于检测流量的属性值。当此脚本返回真值时，此规则生效。
  - 脚本语言为 Starlark，它的语法是 Python 的子集。脚本接受一个全局变量attrs，其中包含了流量相关的属性。
  - 目前只有 http 入站代理会设置这一属性。
  - 示例：
    - 检测 HTTP GET: “attrs[‘:method’] == ‘GET'”
    - 检测 HTTP Path: “attrs[‘:path’].startswith(‘/test’)”
    - 检测 Content Type: “attrs[‘accept’].index(‘text/html’) >= 0”
- **outboundTag**: string
  - 对应一个[额外出站连接配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)的标识。
- **balancerTag**: string
  - 对应一个负载均衡器的标识。balancerTag和outboundTag须二选一。当同时指定时，outboundTag生效。

## BalancerObject

负载均衡器配置。当一个负载均衡器生效时，它会从指定的出站协议中，按配置选出一个最合适的出站协议，进行流量转发。

```
{
  "tag": "balancer",
  "selector": []
}
```

 

其中：

- **tag**: string
  - 此负载均衡器的标识，用于匹配RuleObject中的balancerTag。
- **selector**: [ string ]
  - 一个字符串数组，其中每一个字符串将用于和出站协议标识的前缀匹配。在以下几个出站协议标识中：[ “a“, “ab“, “c“, “ba” ]，”selector“: [“a“]将匹配到[ “a“, “ab” ]。
  - 如果匹配到多个出站协议，负载均衡器目前会从中随机选出一个作为最终的出站协议。

## 预定义域名列表

此列表由[domain-list-community](https://github.com/v2ray/domain-list-community)项目维护，预置于每一个 V2Ray 的安装包中，文件名为geosite.dat。

这个文件包含了一些常见的域名，可用于路由和 DNS 筛选。常用的域名有：

- category-ads: 包含了常见的广告域名。
- category-ads-all: 包含了常见的广告域名，以及广告提供商的域名。
- cn: 相当于 geolocation-cn 和 tld-cn 的合集。
- google: 包含了 Google 旗下的所有域名。
- facebook: 包含了 Facebook 旗下的所有域名。
- geolocation-cn: 包含了常见的国内站点的域名。
- geolocation-!cn: 包含了常见的非国内站点的域名。
- speedtest: 包含了所有 Speedtest 所用的域名。
- tld-cn: 包含了所有 .cn 和 .中国 结尾的域名。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **本地策略**

本地策略可以配置一些用户相关的权限，比如连接超时设置。V2Ray 处理的每一个连接，都对应到一个用户，按照这个用户的等级（level）应用不同的策略。本地策略可按照等级的不同而变化。

## PolicyObject

PolicyObject对应配置文件中的policy项。

```
{
  "levels": {
    "0": {
      "handshake": 4,
      "connIdle": 300,
      "uplinkOnly": 2,
      "downlinkOnly": 5,
      "statsUserUplink": false,
      "statsUserDownlink": false,
      "bufferSize": 10240
    }
  },
  "system": {
    "statsInboundUplink": false,
    "statsInboundDownlink": false
  }
}
```

 

其中：

- **level**: map{string: [LevelPolicyObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#LevelPolicyObject)}
  - 一组键值对，一组键值对，每个键是一个字符串形式的数字（JSON 的要求），比如 “0”、”1″ 等，双引号不能省略，这个数字对应用户等级。每一个值是一个 [LevelPolicyObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#LevelPolicyObject)。

● 每个入站出站代理现在都可以设置用户等级，V2Ray 会根据实际的用户等级应用不同的本地策略。

- **system**: [SystemPolicyObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#SystemPolicyObject)
  - V2Ray 系统的策略

## LevelPolicyObject

```
{
  "handshake": 4,
  "connIdle": 300,
  "uplinkOnly": 2,
  "downlinkOnly": 5,
  "statsUserUplink": false,
  "statsUserDownlink": false,
  "bufferSize": 10240
}
```

 

其中：

- **handshake**: number
  - 连接建立时的握手时间限制。单位为秒。默认值为4。在入站代理处理一个新连接时，在握手阶段（比如 VMess 读取头部数据，判断目标服务器地址），如果使用的时间超过这个时间，则中断该连接。
- **connIdle**: number
  - 连接空闲的时间限制。单位为秒。默认值为300。在入站出站代理处理一个连接时，如果在 connIdle 时间内，没有任何数据被传输（包括上行和下行数据），则中断该连接。
- **uplinkOnly**: number
  - 当连接下行线路关闭后的时间限制。单位为秒。默认值为2。当服务器（如远端网站）关闭下行连接时，出站代理会在等待 uplinkOnly 时间后中断连接。
- **downlinkOnly**: number
  - 当连接上行线路关闭后的时间限制。单位为秒。默认值为5。当客户端（如浏览器）关闭上行连接时，入站代理会在等待 downlinkOnly 时间后中断连接。

● 在 HTTP 浏览的场景中，可以将uplinkOnly和downlinkOnly设为0，以提高连接关闭的效率。

- **statsUserUplink**: true | false
  - 当值为true时，开启当前等级的所有用户的上行流量统计。
- **statsUserDownlink**: true | false
  - 当值为true时，开启当前等级的所有用户的下行流量统计。
- **bufferSize**: number
  - 每个连接的内部缓存大小。单位为 kB。当值为0时，内部缓存被禁用。
  - 默认值 (V2Ray 4.4+):
    - 在 ARM、MIPS、MIPSLE 平台上，默认值为0。
    - 在 ARM64、MIPS64、MIPS64LE 平台上，默认值为4。
    - 在其它平台上，默认值为512。
  - 默认值 (V2Ray 4.3-):
    - 在 ARM、MIPS、MIPSLE、ARM64、MIPS64、MIPS64LE 平台上，默认值为16。
    - 在其它平台上，默认值为2048。

● bufferSize 选项会覆盖环境变量中v2ray.ray.buffer.size的设定。

## SystemPolicyObject

```
{
  "statsInboundUplink": false,
  "statsInboundDownlink": false
}
```

 

其中：

- **statsInboundUplink**: true | false
  - 当值为true时，开启所有入站代理的上行流量统计。
- **statsInboundDownlink**: true | false
  - 当值为true时，开启所有入站代理的下行流量统计。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **入站连接配置**

入站连接用于接收从客户端（浏览器或上一级代理服务器）发来的数据，可用的协议请见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)。

## InboundObject

```
{
  "port": 1080,
  "listen": "127.0.0.1",
  "protocol": "协议名称",
  "settings": {},
  "streamSettings": {},
  "tag": "标识",
  "sniffing": {
    "enabled": false,
    "destOverride": ["http", "tls"]
  },
  "allocate": {
    "strategy": "always",
    "refresh": 5,
    "concurrency": 3
  }
}
```

 

其中：

- **port**: number | “env:variable” | string
  - 端口。接受的格式如下:
    - 整型数值: 实际的端口号。
    - 环境变量: 以”env:“开头，后面是一个环境变量的名称，如”env:PORT“。V2Ray 会以字符串形式解析这个环境变量。
    - 字符串: 可以是一个数值类型的字符串，如”1234“；或者一个数值范围，如”5-10“表示端口 5 到端口 10 这 6 个端口。
- **listen**: address
  - 监听地址，只允许 IP 地址，默认值为”0.0.0.0“，表示接收所有网卡上的连接。除此之外，必须指定一个现有网卡的地址。
- **protocol**: string
  - 连接协议名称，可选的值见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)。
- **settings**:InboundConfigurationObject
  - 具体的配置内容，视协议不同而不同。详见每个[协议](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)中的InboundConfigurationObject。
- **streamSettings**: StreamSettingsObject
  - [底层传输配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#transport)。
- **tag**: string
  - 此此入站连接的标识，用于在其它的配置中定位此连接。当其不为空时，其值必须在所有tag中唯一。

- **sniffing**: [SniffingObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#SniffingObject)
  - 尝试探测流量类型。
- **allocate**: [AllocateObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#AllocateObject)
  - 端口分配设置

## SniffingObject

```
{
  "enabled": false,
  "destOverride": ["http", "tls"]
}
```

 

其中：

- **enabled**: true | false
  - 是否开启流量探测。
- **destOverride**: [“http” | “tls”]
  - 当流量为指定类型时，按其中包括的目标地址重置当前连接的目标。

## AllocateObject

```
{
  "strategy": "always",
  "refresh": 5,
  "concurrency": 3
}
```

 

其中：

- **strategy**: “always” | “random”
  - 端口分配策略。”always“表示总是分配所有已指定的端口，port中指定了多少个端口，V2Ray 就会监听这些端口。”random“表示随机开放端口，每隔refresh分钟在port范围中随机选取concurrency个端口来监听。
- **refresh**: number
  - 随机端口刷新间隔，单位为分钟。最小值为2，建议值为5。这个属性仅当strategy = random时有效。
- **concurrency**: number
  - 随机端口数量。最小值为1，最大值为port范围的三分之一。建议值为3。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **出站连接配置**

出站连接用于向远程网站或下一级代理服务器发送数据，可用的协议请见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)。

## OutboundObject

```
{
  "sendThrough": "0.0.0.0",
  "protocol": "协议名称",
  "settings": {},
  "tag": "标识",
  "streamSettings": {},
  "proxySettings": {
    "tag": "another-outbound-tag"
  },
  "mux": {}
}
```

 

其中：

- **sendThrough**: address
  - 用于发送数据的 IP 地址，当主机有多个 IP 地址时有效，默认值为”0.0.0.0“。
- **protocol**: string
  - 连接协议名称，可选的值见[协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)。
- **settings**: OutboundConfigurationObject
  - 具体的配置内容，视协议不同而不同。详见每个[协议](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)中的OutboundConfigurationObject。
- **tag**: string
  - 此出站连接的标识，用于在其它的配置中定位此连接。当其值不为空时，必须在所有 tag 中唯一。
- **streamSettings**: StreamSettingsObject
  - [底层传输配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#transport)。
- **proxySettings**: [ProxySettingsObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ProxySettingsObject)
  - 传出代理配置。出站代理配置。当出站代理生效时，此出站协议的streamSettings将不起作用。
- **mux**: MuxObject
  - [Mux 配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#mux)。

## ProxySettingsObject

```
{
  "tag": "another-outbound-tag"
}
```

 

其中：

- **tag**: string
  - 当指定另一个出站协议的标识时，此出站协议发出的数据，将被转发至所指定的出站协议发出。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **协议列表**

V2Ray 支持以下协议：

- [Blackhole](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Blackhole)
- [Dokodemo-door](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Dokodemo-door)
- [Freedom](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Freedom)
- [DNS](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#dns_protocol)
- [HTTP](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTP)
- [MTProto](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#MTProto)
- [Shadowsocks](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Shadowsocks)
- [Socks](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Socks)
- [VMess](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#VMess)

## **Blackhole**

- 名称：blackhole
- 类型：**Outbound / 出站协议**

Blackhole（黑洞）是一个**出站**数据协议，它会阻碍所有数据的传出，配合[路由（Routing）](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#routing)一起使用，可以达到禁止访问某些网站的效果。

OutboundConfigurationObject



```
{
  "response": {
    "type": "none"
  }
}
```

 

其中：

- **response**: [ResponseObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ResponseObject)
  - 配置黑洞的响应数据。Blackhole 会在收到待转发数据之后，发送指定的响应数据，然后关闭连接。待转发的数据将被丢弃。如不指定此项，Blackhole 将直接关闭连接。



ResponseObject

```
{
  "type": "none"
}
```

 

其中：

- **type**: “http” | “none”
  - 当type为”none“（默认值）时，Blackhole将直接关闭连接。当type为”http“时，Blackhole会发回一个简单的 HTTP 403 数据包，然后关闭连接。

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **Dokodemo-door**

- 名称：dokodemo-door
- 类型：**Inbound / 入站协议**

Dokodemo door（任意门）是一个**入站**数据协议，它可以监听一个本地端口，并把所有进入此端口的数据发送至指定服务器的一个端口，从而达到端口映射的效果。

InboundConfigurationObject



```
{
  "address": "8.8.8.8",
  "port": 53,
  "network": "tcp",
  "timeout": 0,
  "followRedirect": false,
  "userLevel": 0
}
```

 

其中：

- **address**: address
  - 将流量转发到此地址。可以是一个 IP 地址，形如”1.2.3.4“，或者一个域名，形如”v2ray.com“。字符串类型。
  - 当 followRedirect（见下文）为 true 时，address 可为空。
- **port**: number
  - 将流量转发到目标地址的指定端口，范围[1, 65535]，数值类型。必填参数。
- **network**: “tcp” | “udp” | “tcp,udp”
  - 可接收的网络协议类型。比如当指定为”tcp“时，任意门仅会接收 TCP 流量。默认值为”tcp“。
- **timeout**: number
  - 入站数据的时间限制（秒），默认值为 300。
  - V2Ray 3.1 后等价于对应用户等级的 connIdle 策略
- **followRedirect**: true | false
  - 当值为 true 时，dokodemo-door 会识别出由 iptables 转发而来的数据，并转发到相应的目标地址。详见[传输配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#transport)中的 tproxy 设置。
- **userLevel**: number
  - 用户等级，所有连接都会使用这个用户等级。



透明代理配置样例



V2Ray 中增加一个 dokodemo-door 的传入协议：

```
{
  "network": "tcp,udp",
  "timeout": 30,
  "followRedirect": true
}
```

 

配置 iptables：

```
# Create new chain
iptables -t nat -N V2RAY
iptables -t mangle -N V2RAY
iptables -t mangle -N V2RAY_MARK

# Ignore your V2Ray server's addresses
# It's very IMPORTANT, just be careful.
iptables -t nat -A V2RAY -d 123.123.123.123 -j RETURN

# Ignore LANs and any other addresses you'd like to bypass the proxy
# See Wikipedia and RFC5735 for full list of reserved networks.
iptables -t nat -A V2RAY -d 0.0.0.0/8 -j RETURN
iptables -t nat -A V2RAY -d 10.0.0.0/8 -j RETURN
iptables -t nat -A V2RAY -d 127.0.0.0/8 -j RETURN
iptables -t nat -A V2RAY -d 169.254.0.0/16 -j RETURN
iptables -t nat -A V2RAY -d 172.16.0.0/12 -j RETURN
iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN
iptables -t nat -A V2RAY -d 224.0.0.0/4 -j RETURN
iptables -t nat -A V2RAY -d 240.0.0.0/4 -j RETURN

# Anything else should be redirected to Dokodemo-door's local port
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345

# Add any UDP rules
ip route add local default dev lo table 100
ip rule add fwmark 1 lookup 100
iptables -t mangle -A V2RAY -p udp --dport 53 -j TPROXY --on-port 12345 --tproxy-mark 0x01/0x01
iptables -t mangle -A V2RAY_MARK -p udp --dport 53 -j MARK --set-mark 1

# Apply the rules
iptables -t nat -A OUTPUT -p tcp -j V2RAY
iptables -t mangle -A PREROUTING -j V2RAY
iptables -t mangle -A OUTPUT -j V2RAY_MARK
```

 

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **Freedom**

- 名称：freedom
- 类型：**Outbound / 出站协议**

Freedom 是一个**出站**协议，可以用来向任意网络发送（正常的） TCP 或 UDP 数据。

OutboundConfigurationObject



```
{
  "domainStrategy": "AsIs",
  "redirect": "127.0.0.1:3366",
  "userLevel": 0
}
```

 

其中：

- **domainStrategy**: “AsIs” | “UseIP” | “UseIPv4” | “UseIPv6”
  - 域名解析策略，在目标地址为域名时，Freedom 可以直接向此域名发出连接（”AsIs“），或者将域名解析为 IP 之后再建立连接（”UseIP“、”UseIPv4“、”UseIPv6“）。解析 IP 的步骤会使用 V2Ray 内建的 [DNS](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#dns)。默认值为”AsIs“。
  - (V2Ray 4.6+) 当使用”UseIP“模式，并且[出站连接配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#outbound)中指定了sendThrough时，Freedom 会根据sendThrough的值自动判断所需的IP类型，IPv4 或 IPv6。
  - (V2Ray 4.7+) 当使用”UseIPv4“或”UseIPv6“模式时，Freedom 会只使用对应的 IPv4 或 IPv6 地址。当sendThrough指定了不匹配的本地地址时，将导致连接失败。
- **redirect**: address_port
  - Freedom 会强制将所有数据发送到指定地址（而不是入站协议指定的地址）。其值为一个字符串，样例：”127.0.0.1:80“, “:1234“。当地址不指定时，如”:443“，Freedom 不会修改原先的目标地址。当端口为0时，如”v2ray.com:0“，Freedom 不会修改原先的端口。
- **userLevel**: number
  - 用户等级，所有连接都使用这一等级。

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **DNS**

- 名称：dns
- 类型：**Outbound** / **出站协议**

DNS 是一个出站协议，主要用于拦截和转发 DNS 查询。此出站协议只能接收 DNS 流量（包含基于 UDP 和 TCP 协议的查询），其它类型的流量会导致错误。

在处理 DNS 查询时，此出站协议会将 IP 查询（即 A 和 AAAA）转发给内置的 DNS 服务器。其它类询的流量将被转发至它们原本的目标地址。

DNS 出站协议在 V2Ray 4.15 中引入。

OutboundConfigurationObject



```
{
    "network": "tcp",
    "address": "1.1.1.1",
    "port": 53
}
```

 

其中：

- **network**: “tcp” | “udp”
  - (V2Ray 4.16+) 修改 DNS 流量的传输层协议，可选的值有”tcp”和”udp”。当不指定时，保持来源的传输方式不变。
- **address**: address
  - (V2Ray 4.16+) 修改 DNS 服务器地址。当不指定时，保持来源中指定的地址不变。
- **port**: number
  - (V2Ray 4.16+) 修改 DNS 服务器端口。当不指定时，保持来源中指定的端口不变。

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **HTTP**

- 名称：http
- 类型：**Inbound / 入站协议**

HTTP 是一个**入站**数据协议，兼容 HTTP 1.x 代理。

InboundConfigurationObject



```
{
  "timeout": 0,
  "accounts": [
    {
      "user": "my-username",
      "pass": "my-password"
    }
  ],
  "allowTransparent": false,
  "userLevel": 0
}
```

 

其中：

- **timeout**: number
  - 从客户端读取数据的超时设置（秒），0 表示不限时。默认值为 300。 V2Ray 3.1 后等价于对应用户等级的 connIdle 策略。
- **accounts**: [[AccountObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#AccountObject)]
  - 一个数组，数组中每个元素为一个用户帐号。默认值为空。
  - 当 accounts 非空时，HTTP 代理将对入站连接进行 Basic Authentication 验证。
- **allowTransparent**: true | false
  - 当为true时，会转发所有 HTTP 请求，而非只是代理请求。若配置不当，开启此选项会导致死循环。
- **userLevel**: number
  - 用户等级，所有连接使用这一等级。



AccountObject



```
{
  "user": "my-username",
  "pass": "my-password"
}
```

 

其中：

- **user**: string
  - 用户名，字符串类型。必填。
- **pass**: string
  - 密码，字符串类型。必填。

● 在 Linux 中使用以下环境变量即可在当前 session 使用全局 HTTP 代理（很多软件都支持这一设置，也有不支持的）。



- - export http_proxy=http://127.0.0.1:8080/ (地址须改成你配置的 HTTP 传入代理地址)
  - export https_proxy=$http_proxy

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **MTProto**

协议描述：

- 名称：mtproto
- 类型：**Inbound 入站协议 / Outbound 出站协议**

MTProto 是一个 Telegram 专用的代理协议。在 V2Ray 中可使用一组入站出站代理来完成 Telegram 数据的代理任务。

目前只支持转发到 Telegram 的 IPv4 地址。

InboundConfigurationObject



```
{
  "users": [{
    "email": "love@v2ray.com",
    "level": 0,
    "secret": "b0cbcef5a486d9636472ac27f8e11a9d"
  }]
}
```

 

其中：

- users: [[UserObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#UserObject_MTProto)]
  - 一个数组，其中每一个元素表示一个用户。**目前只有第一个用户会生效**。



UserObject



```
{
  "email": "love@v2ray.com",
  "level": 0,
  "secret": "b0cbcef5a486d9636472ac27f8e11a9d"
}
```

 

其中：

- **email**: string
  - 用户邮箱，用于统计流量等辅助功能
- **level**: number
  - 用户等级。
- **secret**: string
  - 用户密钥。必须为 32 个字符，仅可包含0到9和a到f之间的字符。

● 使用此命令生成 MTProto 代理所需要的用户密钥：openssl rand -hex 16

OutboundConfigurationObject



```
{
}
```

 

样例配置

MTProto 仅可用于 Telegram 数据。你可能需要一个路由来绑定对应的入站出站代理。以下是一个不完整的示例：

入站代理：

```
{
  "tag": "tg-in",
  "port": 443,
  "protocol": "mtproto",
  "settings": {
    "users": [{"secret": "b0cbcef5a486d9636472ac27f8e11a9d"}]
  }
}
```

 

出站代理：

```
{
  "tag": "tg-out",
  "protocol": "mtproto",
  "settings": {}
}
```

 

路由：

```
{
  "type": "field",
  "inboundTag": ["tg-in"],
  "outboundTag": "tg-out"
}
```

 

然后使用 Telegram 连接这台机器的 443 端口即可。

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **Shadowsocks**

- 名称：shadowsocks
- 类型：**Inbound 入站协议 /** **Outbound 出站协议**

[Shadowsocks](https://zh.wikipedia.org/wiki/Shadowsocks) 协议，包含**入站**和**出站**两部分，兼容大部分其它版本的实现。

与官方版本的兼容性：

- 支持 TCP 和 UDP 数据包转发，其中 UDP 可选择性关闭；
- 支持 [OTA](https://web.archive.org/web/20161221022225/https://shadowsocks.org/en/spec/one-time-auth.html)；
  - 客户端可选开启或关闭；
  - 服务器端可强制开启、关闭或自适应；
- 加密方式（其中 [AEAD](https://shadowsocks.org/en/spec/AEAD-Ciphers.html) 加密方式在 V2Ray 3.0 中加入）：
  - aes-256-cfb
  - aes-128-cfb
  - chacha20
  - chacha20-ietf
  - aes-256-gcm
  - aes-128-gcm
  - chacha20-poly1305 或称 chacha20-ietf-poly1305
- 插件：
  - 通过 Standalone 模式支持 obfs

Shadowsocks 的配置分为两部分，InboundConfigurationObject 和 OutboundConfigurationObject，分别对应入站和出站协议配置中的 settings 项。

InboundConfigurationObject

```
{
  "email": "love@v2ray.com",
  "method": "aes-128-cfb",
  "password": "密码",
  "level": 0,
  "ota": true,
  "network": "tcp"
}
```

 

其中：

- **email**: string
  - 邮件地址，可选，用于标识用户
- **method**: string
  - 必填。可选的值见[加密方式列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#encryption-list)
- **password**: string
  - 必填，任意字符串。Shadowsocks 协议不限制密码长度，但短密码会更可能被破解，建议使用 16 字符或更长的密码。
- **level**: number
  - 用户等级，默认值为 0。详见[本地策略](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#policy)。
- **ota**: true | false
  - 是否强制 OTA，如果不指定此项，则自动判断。强制开启 OTA 后，V2Ray 会拒绝未启用 OTA 的连接。反之亦然。
  - 当使用 AEAD 时，ota 设置无效
- **network**: “tcp” | “udp” | “tcp,udp”
  - 可接收的网络连接类型，默认值为”tcp”。

OutboundConfigurationObject

```
{
  "servers": [
    {
      "email": "love@v2ray.com",
      "address": "127.0.0.1",
      "port": 1234,
      "method": "加密方式",
      "password": "密码",
      "ota": false,
      "level": 0
    }
  ]
}
```

 

其中：

- **servers**: [[ServerObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ServerObject_Shadowsocks)]
  - 一个数组，其中每一项是一个 [ServerObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ServerObject_Shadowsocks)。



ServerObject



```
{
  "email": "love@v2ray.com",
  "address": "127.0.0.1",
  "port": 1234,
  "method": "加密方式",
  "password": "密码",
  "ota": false,
  "level": 0
}
```

 

其中：

- **email**: string
  - 邮件地址，可选，用于标识用户
- **address**: address
  - Shadowsocks 服务器地址，支持 IPv4、IPv6 和域名。必填。
- **port**: number
  - Shadowsocks 服务器端口。必填。
- **method**: string
  - 必填。可选的值见[加密方式列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#encryption-list)
- **password**: string
  - 必填。任意字符串。Shadowsocks 协议不限制密码长度，但短密码会更可能被破解，建议使用 16 字符或更长的密码。
- **ota**: true | false
  - 是否开启 Shadowsocks 的一次验证（One time auth），默认值为false。
  - 当使用 AEAD 时，ota 设置无效。
- **level**: number
  - 用户等级



加密方式列表



- “aes-256-cfb”
- “aes-128-cfb”
- “chacha20”
- “chacha20-ietf”
- “aes-256-gcm”
- “aes-128-gcm”
- “chacha20-poly1305” 或 “chacha20-ietf-poly1305”

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **Socks**

- 名称：socks
- 类型：**Inbound 入站协议 /** **Outbound 出站协议**

标准 Socks 协议实现，兼容 [Socks 4](http://ftp.icm.edu.pl/packages/socks/socks4/SOCKS4.protocol)、Socks 4a 和 [Socks 5](http://ftp.icm.edu.pl/packages/socks/socks4/SOCKS4.protocol)。

Socks 的配置分为两部分，InboundConfigurationObject 和 OutboundConfigurationObject，分别对应入站和出站协议配置中的 settings 项。

OutboundConfigurationObject



```
{
  "servers": [{
    "address": "127.0.0.1",
    "port": 1234,
    "users": [
      {
        "user": "test user",
        "pass": "test pass",
        "level": 0
      }
    ]
  }]
}
```

 

其中：

- **servers**: [ [ServerObject ](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ServerObject_Socks)]
  - 
  - Socks 服务器列表，其中每一项是一个服务器配置。



ServerObject



```
{
  "address": "127.0.0.1",
  "port": 1234,
  "users": [
    {
      "user": "test user",
      "pass": "test pass",
      "level": 0
    }
  ]
}
```

 

其中：

- **address**: address
  - 服务器地址。

● 仅支持连接到 Socks 5 服务器。

- **port**: number
  - 服务器端口
- **users**: [ [UserObject ](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#UserObject_Socks)]
  - 用户列表，其中每一项一个用户配置。当列表不为空时，Socks 客户端会使用此用户信息进行认证；如未指定，则不进行认证。



UserObject



```
{
  "user": "test user",
  "pass": "test pass",
  "level": 0
}
```

 

其中：

- **user**: string
  - 用户名
- **pass**: string
  - 密码
- **level**: number
  - 用户等级

InboundConfigurationObject



```
{
  "auth": "noauth",
  "accounts": [
    {
      "user": "my-username",
      "pass": "my-password"
    }
  ],
  "udp": false,
  "ip": "127.0.0.1",
  "userLevel": 0
}
```

 

其中：

- **auth**: “noauth” | “password”
  - Socks 协议的认证方式，支持”noauth“匿名方式和”password“用户密码方式。默认值为”noauth“。
- **accounts**: [ [AccountObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#AccountObject_Socks) ]
  - 一个数组，数组中每个元素为一个用户帐号。默认值为空。此选项仅当 auth 为 password 时有效。
- **udp**: true | false
  - 是否开启 UDP 协议的支持，默认值为 false。
- **ip**: address
  - 当开启 UDP 时，V2Ray 需要知道本机的 IP 地址。默认值为”127.0.0.1“。
- **userLevel**: number
  - 用户等级，所有连接使用这一等级。



AccountObject



```
{
  "user": "my-username",
  "pass": "my-password"
}
```

 

其中：

- **user**: string
  - 用户名
- **pass**: string
  - 密码

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

## **VMess**

- 名称：vmess
- 类型：**Inbound 入站协议 /** **Outbound 出站协议**

[VMess](https://www.v2ray.com/developer/protocols/vmess.html) 是一个加密传输协议，它分为**入站**和**出站**两部分，通常作为 V2Ray 客户端和服务器之间的桥梁。

VMess 依赖于系统时间，请确保使用 V2Ray 的系统 UTC 时间误差在 90 秒之内，时区无关。在 Linux 系统中可以安装ntp服务来自动同步系统时间。

VMess 的配置分为两部分，InboundConfigurationObject 和 OutboundConfigurationObject，分别对应入站和出站协议配置中的 settings 项。

OutboundConfigurationObject



```
{
  "vnext": [
    {
      "address": "127.0.0.1",
      "port": 37192,
      "users": [
        {
          "id": "27848739-7e62-4138-9fd3-098a63964b6b",
          "alterId": 16,
          "security": "auto",
          "level": 0
        }
      ]
    }
  ]
}
```

 

其中：

- **vnext**: [ [ServerObject ](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ServerObject_VMess)]
  - 一个数组，包含一系列的服务器配置。



ServerObject

```
{
  "address": "127.0.0.1",
  "port": 37192,
  "users": []
}
```

 

其中：

- **address**: address
  - 服务器地址，支持 IP 地址或者域名。
- **port**: number
  - 服务器端口号。
- **users**: [ [UserObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#UserObject_VMess) ]
  - 一组服务器认可的用户



UserObject

```
{
  "id": "27848739-7e62-4138-9fd3-098a63964b6b",
  "alterId": 16,
  "security": "auto",
  "level": 0
}
```

 

其中：

- **id**: string
  - VMess 用户的主 ID。必须是一个合法的 UUID。
- **alterId**: number
  - 为了进一步防止被探测，一个用户可以在主 ID 的基础上，再额外生成多个 ID。这里只需要指定额外的 ID 的数量，推荐值为 16。不指定的话，默认值是 0。最大值 65535。这个值不能超过服务器端所指定的值。
- **level**: number
  - 用户等级
- **security**: “aes-128-gcm” | “chacha20-poly1305” | “auto” | “none”
  - 加密方式，客户端将使用配置的加密方式发送数据，服务器端自动识别，无需配置。
    - “aes-128-gcm“：推荐在 PC 上使用
    - “chacha20-poly1305“：推荐在手机端使用
    - “auto“：默认值，自动选择（运行框架为 AMD64、ARM64 或 s390x 时为aes-128-gcm加密方式，其他情况则为 Chacha20-Poly1305 加密方式）
    - “none“：不加密

● 推荐使用”auto“加密方式，这样可以永久保证安全性和兼容性。

InboundConfigurationObject

```
{
  "clients": [
    {
      "id": "27848739-7e62-4138-9fd3-098a63964b6b",
      "level": 0,
      "alterId": 16,
      "email": "love@v2ray.com"
    }
  ],
  "default": {
    "level": 0,
    "alterId": 32
  },
  "detour": {
    "to": "tag_to_detour"
  },
  "disableInsecureEncryption": false
}
```

 

其中：

- **clients**：[ [ClientObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#ClientObject_VMess) ]
  - 一组服务器认可的用户。clients 可以为空。当此配置用作动态端口时，V2Ray 会自动创建用户。
- **detour**: [DetourObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#DetourObject_VMess)
  - 指示对应的出站协议使用另一个服务器。
- **default**: [DefaultObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#DefaultObject_VMess)
  - 可选，clients 的默认配置。仅在配合detour时有效。
- **disableInsecureEncryption**: true | false
  - 禁止客户端使用不安全的加密方式，当下列客户端指定下列加密方式时，服务器会主动断开连接。默认值为false。
    - “none”
    - “aes-128-cfb”



ClientObject

```
{
  "id": "27848739-7e62-4138-9fd3-098a63964b6b",
  "level": 0,
  "alterId": 16,
  "email": "love@v2ray.com"
}
```

 

其中：

- **id**: string
  - VMess 的用户 ID。必须是一个合法的 UUID。
- **level**: number
  - 用户等级，详见[本地策略](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#policy)
- **alterId**: number
  - 与上文出站协议中的含义相同。
- **email**: string
  - 用户邮箱地址，用于区分不同用户的流量。

● alterId 取值的大小和流量特征没有必然联系。对于日常使用，64 以内的值已经够用了。



DetourObject

```
{
  "to": "tag_to_detour"
}
```

 

其中：

- **to**: string
  - 一个入站协议的tag，详见配置文件。指定的入站协议必须是一个 VMess



DefaultObject

```
{
  "level": 0,
  "alterId": 32
}
```

 

其中：

- **level**: number
  - 用户等级，意义同上。默认值为0。
- **alterId**: number
  - 和 ClientObject 中的 alterId 相同，默认值为64。推荐值16。

[↑返回协议列表](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#protocols)

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **Mux 多路复用**

Mux 功能是在一条 TCP 连接上分发多个 TCP 连接的数据。实现细节详见[Mux.Cool](https://www.v2ray.com/eng/protocols/muxcool.html)。Mux 是为了减少 TCP 的握手延迟而设计，而非提高连接的吞吐量。使用 Mux 看视频、下载或者测速通常都有反效果。Mux 只需要在客户端启用，服务器端自动适配。

## MuxObject

MuxObject对应OutboundObject中的mux项。

```
{
  "enabled": false,
  "concurrency": 8
}
```

 

其中：

- **enabled**: true | false
  - 是否启用 Mux
- **concurrency**: number
  - 最大并发连接数。最小值1，最大值1024，默认值8。这个数值表示了一个 TCP 连接上最多承载的 Mux 连接数量。当客户端发出了 8 个 TCP 请求，而concurrency=8时，V2Ray 只会发出一条实际的 TCP 连接，客户端的 8 个请求全部由这个 TCP 连接传输。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **额外的传入连接配置（inbound detour）**

**此项为旧版（3.x）配置，已失效且被折叠，新版（4+）请忽略此部分**

点击展开

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **额外的传出连接配置（outbound detour）**

**此项为旧版（3.x）配置，已失效且被折叠，新版（4+）请忽略此部分**

点击展开

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **底层传输配置（transport）**

底层传输方式（transport）是当前 V2Ray 节点和其它节点对接的方式。底层传输方式提供了稳定的数据传输通道。通常来说，一个网络连接的两端需要有对称的传输方式。比如一端用了 WebSocket，那么另一个端也必须使用 WebSocket，否则无法建立连接。

底层传输（transport）配置分为两部分，一是全局设置([TransportObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TransportObject))，二是分协议配置([StreamSettingsObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#StreamSettingsObject))。分协议配置可以指定每个单独的入站出站协议用怎样的方式传输。通常来说客户端和服务器对应的出站入站协议需要使用同样的传输方式。当分协议传输配置指定了一种传输方式，但没有填写其设置时，此传输方式会使用全局配置中的设置。

## TransportObject

TransportObject对应配置文件的transport项。

```
{
  "tcpSettings": {},
  "kcpSettings": {},
  "wsSettings": {},
  "httpSettings": {},
  "dsSettings": {},
  "quicSettings": {}
}
```

 

其中：

- **tcpSettings**: TcpObject
  - 针对 [TCP 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TCP)。
- **kcpSettings**: KcpObject
  - 针对 [mKCP 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#mKCP)。
- **wsSettings**: WebSocketObject
  - 针对 [WebSocket 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#WebSocket)。
- **httpSettings**: HttpObject
  - 针对 [HTTP/2 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTP2)。
- **dsSettings**: DomainSocketObject
  - 针对[Domain Socket 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#domainsocket)。
- **quicSettings**: QUICObject
  - (V2Ray 4.7+) 针对[QUIC 连接的配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#quic)。

## StreamSettingsObject

TransportObject对应出站入站协议中的streamSettings项。每一个入站、出站连接都可以分别配置不同的传输配置，都可以设置streamSettings来进行一些传输的配置。

```
{
  "network": "tcp",
  "security": "none",
  "tlsSettings": {},
  "tcpSettings": {},
  "kcpSettings": {},
  "wsSettings": {},
  "httpSettings": {},
  "dsSettings": {},
  "quicSettings": {},
  "sockopt": {
    "mark": 0,
    "tcpFastOpen": false,
    "tproxy": "off"
  }
}
```

 

其中：

- **network**: “tcp” | “kcp” | “ws” | “http” | “domainsocket” | “quic”
  - 数据流所使用的网络类型，默认值为 “tcp“
- **security**: “none” | “tls”
  - 是否启入传输层加密，支持的选项有 “none” 表示不加密（默认值），”tls” 表示使用 [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security)。
- **tlsSettings**: [TLSObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TLSObject_transport)
  - [TLS 配置](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TLSObject_transport)。TLS 由 Golang 提供，支持 TLS 1.2，不支持 DTLS。
- **tcpSettings**: TcpObject
  - 当前连接的 TCP 配置，仅当此连接使用 [TCP](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#TCP) 时有效。配置内容与上面的全局配置相同。
- **kcpSettings**: KcpObject
  - 当前连接的 mKCP 配置，仅当此连接使用 [mKCP](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#mKCP) 时有效。配置内容与上面的全局配置相同。
- **wsSettings**: WebSocketObject
  - 当前连接的 WebSocket 配置，仅当此连接使用 [WebSocket](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#WebSocket) 时有效。配置内容与上面的全局配置相同。
- **httpSettings**: HttpObject
  - 当前连接的 HTTP/2 配置，仅当此连接使用 [HTTP/2](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTP2) 时有效。配置内容与上面的全局配置相同。
- **dsSettings**: DomainSocketObject
  - 当前连接的 Domain socket 配置，仅当此连接使用 [Domain socket](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#domainsocket) 时有效。配置内容与上面的全局配置相同。
- **quicSettings**: QUICObject
  - (V2Ray 4.7+) 当前连接的 QUIC 配置，仅当此连接使用 [QUIC](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#quic) 时有效。配置内容与上面的全局配置相同。
- **sockopt**: [SockoptObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#SockoptObject_transport)
  - 连接选项。



TLSObject



```
{
  "serverName": "v2ray.com",
  "allowInsecure": false,
  "allowInsecureCiphers": false,
  "alpn": ["http/1.1"],
  "certificates": [],
  "disableSystemRoot": false
}
```

 

其中：

- **serverName**: string
  - 指定服务器端证书的域名，在连接由 IP 建立时有用。
- **alpn**: [ string ]
  - 一个字符串数组，指定了 TLS 握手时指定的 ALPN 数值。默认值为[“http/1.1“]。
- **allowInsecure**: true | false
  - 是否允许不安全连接（用于客户端）。当值为 true 时，V2Ray 不会检查远端主机所提供的 TLS 证书的有效性。
- **allowInsecureCiphers**: true | false
  - 是否允许不安全的加密方式。默认情况下 TLS 只使用 TLS 1.3 推荐的加密算法套件，开启这一选项会增加一些与 TLS 1.2 兼容的加密套件。
- **certificates**: [ [CertificateObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#CertificateObject_transport) ]
  - 证书列表，其中每一项表示一个证书
- **disableSystemRoot**: true | false
  - (V2Ray 4.18+) 是否禁用操作系统自带的 CA 证书。默认值为false。当值为true时，V2Ray 只会使用certificates中指定的证书进行 TLS 握手。



CertificateObject



```
{
  "usage": "encipherment",

  "certificateFile": "/path/to/certificate.crt",
  "keyFile": "/path/to/key.key",

  "certificate": [
    "-----BEGIN CERTIFICATE-----",
    "MIICwDCCAaigAwIBAgIRAO16JMdESAuHidFYJAR/7kAwDQYJKoZIhvcNAQELBQAw",
    "ADAeFw0xODA0MTAxMzU1MTdaFw0xODA0MTAxNTU1MTdaMAAwggEiMA0GCSqGSIb3",
    "DQEBAQUAA4IBDwAwggEKAoIBAQCs2PX0fFSCjOemmdm9UbOvcLctF94Ox4BpSfJ+",
    "3lJHwZbvnOFuo56WhQJWrclKoImp/c9veL1J4Bbtam3sW3APkZVEK9UxRQ57HQuw",
    "OzhV0FD20/0YELou85TwnkTw5l9GVCXT02NG+pGlYsFrxesUHpojdl8tIcn113M5",
    "pypgDPVmPeeORRf7nseMC6GhvXYM4txJPyenohwegl8DZ6OE5FkSVR5wFQtAhbON",
    "OAkIVVmw002K2J6pitPuJGOka9PxcCVWhko/W+JCGapcC7O74palwBUuXE1iH+Jp",
    "noPjGp4qE2ognW3WH/sgQ+rvo20eXb9Um1steaYY8xlxgBsXAgMBAAGjNTAzMA4G",
    "A1UdDwEB/wQEAwIFoDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAA",
    "MA0GCSqGSIb3DQEBCwUAA4IBAQBUd9sGKYemzwPnxtw/vzkV8Q32NILEMlPVqeJU",
    "7UxVgIODBV6A1b3tOUoktuhmgSSaQxjhYbFAVTD+LUglMUCxNbj56luBRlLLQWo+",
    "9BUhC/ow393tLmqKcB59qNcwbZER6XT5POYwcaKM75QVqhCJVHJNb1zSEE7Co7iO",
    "6wIan3lFyjBfYlBEz5vyRWQNIwKfdh5cK1yAu13xGENwmtlSTHiwbjBLXfk+0A/8",
    "r/2s+sCYUkGZHhj8xY7bJ1zg0FRalP5LrqY+r6BckT1QPDIQKYy615j1LpOtwZe/",
    "d4q7MD/dkzRDsch7t2cIjM/PYeMuzh87admSyL6hdtK0Nm/Q",
    "-----END CERTIFICATE-----"
  ],
  "key": [
    "-----BEGIN RSA PRIVATE KEY-----",
    "MIIEowIBAAKCAQEArNj19HxUgoznppnZvVGzr3C3LRfeDseAaUnyft5SR8GW75zh",
    "bqOeloUCVq3JSqCJqf3Pb3i9SeAW7Wpt7FtwD5GVRCvVMUUOex0LsDs4VdBQ9tP9",
    "GBC6LvOU8J5E8OZfRlQl09NjRvqRpWLBa8XrFB6aI3ZfLSHJ9ddzOacqYAz1Zj3n",
    "jkUX+57HjAuhob12DOLcST8np6IcHoJfA2ejhORZElUecBULQIWzjTgJCFVZsNNN",
    "itieqYrT7iRjpGvT8XAlVoZKP1viQhmqXAuzu+KWpcAVLlxNYh/iaZ6D4xqeKhNq",
    "IJ1t1h/7IEPq76NtHl2/VJtbLXmmGPMZcYAbFwIDAQABAoIBAFCgG4phfGIxK9Uw",
    "qrp+o9xQLYGhQnmOYb27OpwnRCYojSlT+mvLcqwvevnHsr9WxyA+PkZ3AYS2PLue",
    "C4xW0pzQgdn8wENtPOX8lHkuBocw1rNsCwDwvIguIuliSjI8o3CAy+xVDFgNhWap",
    "/CMzfQYziB7GlnrM6hH838iiy0dlv4I/HKk+3/YlSYQEvnFokTf7HxbDDmznkJTM",
    "aPKZ5qbnV+4AcQfcLYJ8QE0ViJ8dVZ7RLwIf7+SG0b0bqloti4+oQXqGtiESUwEW",
    "/Wzi7oyCbFJoPsFWp1P5+wD7jAGpAd9lPIwPahdr1wl6VwIx9W0XYjoZn71AEaw4",
    "bK4xUXECgYEA3g2o9WqyrhYSax3pGEdvV2qN0VQhw7Xe+jyy98CELOO2DNbB9QNJ",
    "8cSSU/PjkxQlgbOJc8DEprdMldN5xI/srlsbQWCj72wXxXnVnh991bI2clwt7oYi",
    "pcGZwzCrJyFL+QaZmYzLxkxYl1tCiiuqLm+EkjxCWKTX/kKEFb6rtnMCgYEAx0WR",
    "L8Uue3lXxhXRdBS5QRTBNklkSxtU+2yyXRpvFa7Qam+GghJs5RKfJ9lTvjfM/PxG",
    "3vhuBliWQOKQbm1ZGLbgGBM505EOP7DikUmH/kzKxIeRo4l64mioKdDwK/4CZtS7",
    "az0Lq3eS6bq11qL4mEdE6Gn/Y+sqB83GHZYju80CgYABFm4KbbBcW+1RKv9WSBtK",
    "gVIagV/89moWLa/uuLmtApyEqZSfn5mAHqdc0+f8c2/Pl9KHh50u99zfKv8AsHfH",
    "TtjuVAvZg10GcZdTQ/I41ruficYL0gpfZ3haVWWxNl+J47di4iapXPxeGWtVA+u8",
    "eH1cvgDRMFWCgE7nUFzE8wKBgGndUomfZtdgGrp4ouLZk6W4ogD2MpsYNSixkXyW",
    "64cIbV7uSvZVVZbJMtaXxb6bpIKOgBQ6xTEH5SMpenPAEgJoPVts816rhHdfwK5Q",
    "8zetklegckYAZtFbqmM0xjOI6bu5rqwFLWr1xo33jF0wDYPQ8RHMJkruB1FIB8V2",
    "GxvNAoGBAM4g2z8NTPMqX+8IBGkGgqmcYuRQxd3cs7LOSEjF9hPy1it2ZFe/yUKq",
    "ePa2E8osffK5LBkFzhyQb0WrGC9ijM9E6rv10gyuNjlwXdFJcdqVamxwPUBtxRJR",
    "cYTY2HRkJXDdtT0Bkc3josE6UUDvwMpO0CfAETQPto1tjNEDhQhT",
    "-----END RSA PRIVATE KEY-----"
  ]
}
```

 

其中：

- **usage**: “encipherment” | “verify” | “issue”
  - 证书用途，默认值为”encipherment“
    - “encipherment“: 证书用于 TLS 认证和加密。
    - “verify“: 证书用于验证远端 TLS 的证书。当使用此项时，当前证书必须为 CA 证书。
    - “issue“: 证书用于签发其它证书。当使用此项时，当前证书必须为 CA 证书。

● 在 Windows 平台上可以将自签名的 CA 证书安装到系统中，即可验证远端 TLS 的证书。

● 当有新的客户端请求时，假设所指定的serverName为”v2ray.com“，V2Ray 会先从证书列表中寻找可用于”v2ray.com“的证书，如果没有找到，则使用任一usage为”issue“的证书签发一个适用于”v2ray.com“的证书，有效期为一小时。并将新的证书加入证书列表，以供后续使用。

- **certificateFile**: string
  - 证书文件路径，如使用 OpenSSL 生成，后缀名为 .crt。

● 使用v2ctl cert -ca可以生成自签名的 CA 证书。

- **certificate**: [ string ]
  - 一个字符串数组，表示证书内容，格式如样例所示。certificate和certificateFile二者选一。
- **keyFile**: string
  - 密钥文件路径，如使用 OpenSSL 生成，后缀名为 .key。目前暂不支持需要密码的 key 文件。
- **key**: [ string ]
  - 一个字符串数组，表示密钥内容，格式如样例如示。key和keyFile二者选一。
  - 当certificateFile和certificate同时指定时，V2Ray 优先使用certificateFile。keyFile和key也一样。

● 当usage为”verify“时，keyFile和key可均为空。



SockoptObject

```
{
  "mark": 0,
  "tcpFastOpen": false,
  "tproxy": "off"
}
```

 

其中：

- **mark**: number
  - 一个整数。当其值非零时，在出站连接上标记 SO_MARK。
    - 仅适用于 Linux 系统。
    - 需要 CAP_NET_ADMIN 权限。
- **tcpFastOpen**: true | false
  - 是否启用 [TCP Fast Open](https://zh.wikipedia.org/wiki/TCP快速打开)。当其值为true时，强制开启TFO；当其它为false时，强制关闭TFO；当此项不存在时，使用系统默认设置。可用于入站出站连接。
    - 仅在以下版本（或更新版本）的操作系统中可用:
      - Windows 10 (1604)
      - Mac OS 10.11 / iOS 9
      - Linux 3.16: 系统已默认开启，无需要配置。
- **tproxy**: “redirect” | “tproxy” | “off”
  - 是否开启透明代理 (仅适用于 Linux)。
    - “redirect“: 使用 Redirect 模式的透明代理。仅支持 TCP/IPv4 和 UDP 连接。
    - “tproxy“: 使用 TProxy 模式的透明代理。支持 TCP 和 UDP 连接。
    - “off“: 关闭透明代理。

透明代理需要 Root 或 CAP_NET_ADMIN 权限。

● 当 [Dokodemo-door](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#Dokodemo-door) 中指定了followRedirect，且sockopt.tproxy为空时，sockopt.tproxy的值会被设为”redirect“。

## **TCP 传输方式**

TcpObject



```
{
  "header": {
    "type": "none"
  }
}
```

 

其中：

- **header**: NoneHeaderObject | HttpHeaderobject
  - 数据包头部伪装设置，默认值为NoneHeaderObject。

NoneHeaderObject

不进行伪装

```
{
  "type": "none"
}
```

 

其中：

- type: “none”
  - 指定不进行伪装

HttpHeaderObject

HTTP 伪装配置必须在对应的入站出站连接上同时配置，且内容必须一致。

```
{
  "type": "http",
  "request": {},
  "response": {}
}
```

 

其中：

- **type**: “http”
  - 指定进行 HTTP 伪装
- **request**: [HTTPRequestObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTPRequestObject)
  - HTTP 请求
- **response**: [HTTPResponseObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HTTPResponseObject)
  - HTTP 响应



HTTPRequestObject

```
{
  "version": "1.1",
  "method": "GET",
  "path": ["/"],
  "headers": {
    "Host": ["www.baidu.com", "www.bing.com"],
    "User-Agent": [
      "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safari/537.36",
      "Mozilla/5.0 (iPhone; CPU iPhone OS 10_0_2 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/53.0.2785.109 Mobile/14A456 Safari/601.1.46"
    ],
    "Accept-Encoding": ["gzip, deflate"],
    "Connection": ["keep-alive"],
    "Pragma": "no-cache"
  }
}
```

 

其中：

- **version**: string
  - HTTP 版本，默认值为”1.1“。
- **method**: string
  - HTTP 方法，默认值为”GET“。
- **path**: [ string ]
  - 路径，一个字符串数组。默认值为[“/“]。当有多个值时，每次请求随机选择一个值。
- **headers**: map{ string, [ string ]}
  - HTTP 头，一个键值对，每个键表示一个 HTTP 头的名称，对应的值是一个数组。每次请求会附上所有的键，并随机选择一个对应的值。默认值见上方示例。



HTTPResponseObject

```
{
  "version": "1.1",
  "status": "200",
  "reason": "OK",
  "headers": {
    "Content-Type": ["application/octet-stream", "video/mpeg"],
    "Transfer-Encoding": ["chunked"],
    "Connection": ["keep-alive"],
    "Pragma": "no-cache"
  }
}
```

 

其中：

- **version**: string
  - HTTP 版本，默认值为”1.1“。
- **status**: string
  - HTTP 状态，默认值为”200“。
- **reason**: string
  - HTTP 状态说明，默认值为”OK“。
- **headers**: map{string, [ string ]}
  - HTTP 头，一个键值对，每个键表示一个 HTTP 头的名称，对应的值是一个数组。每次请求会附上所有的键，并随机选择一个对应的值。默认值见上方示例。

## **mKCP 传输方式**

[mKCP](https://www.v2ray.com/eng/protocols/mkcp.html) 是流式传输协议，由 [KCP 协议](https://github.com/skywind3000/kcp)修改而来，可以按顺序传输任意的数据流。

mKCP 使用 UDP 来模拟 TCP 连接，请确定主机上的防火墙配置正确。mKCP 牺牲带宽来降低延迟。传输同样的内容，mKCP 一般比 TCP 消耗更多的流量。

KcpObject

```
{
  "mtu": 1350,
  "tti": 20,
  "uplinkCapacity": 5,
  "downlinkCapacity": 20,
  "congestion": false,
  "readBufferSize": 1,
  "writeBufferSize": 1,
  "header": {
    "type": "none"
  }
}
```

 

其中：

- **mtu**: number
  - 最大传输单元（maximum transmission unit），请选择一个介于 576 – 1460 之间的值。默认值为 1350。
- **tti**: number
  - 传输时间间隔（transmission time interval），单位毫秒（ms），mKCP 将以这个时间频率发送数据。请选译一个介于 10 – 100 之间的值。默认值为 50。
- **uplinkCapacity**: number
  - 上行链路容量，即主机发出数据所用的最大带宽，单位 MB/s，默认值 5。注意是 Byte 而非 bit。可以设置为 0，表示一个非常小的带宽。
- **downlinkCapacity**: number
  - 下行链路容量，即主机接收数据所用的最大带宽，单位 MB/s，默认值 20。注意是 Byte 而非 bit。可以设置为 0，表示一个非常小的带宽。

● uplinkCapacity 和 downlinkCapacity 决定了 mKCP 的传输速度。以客户端发送数据为例，客户端的 uplinkCapacity 指定了发送数据的速度，而服务器端的 downlinkCapacity 指定了接收数据的速度。两者的值以较小的一个为准。推荐把 downlinkCapacity 设置为一个较大的值，比如 100，而 uplinkCapacity 设为实际的网络速度。当速度不够时，可以逐渐增加 uplinkCapacity 的值，直到带宽的两倍左右。

- **congestion**: true | false
  - 是否启用拥塞控制，默认值为 false。开启拥塞控制之后，V2Ray 会自动监测网络质量，当丢包严重时，会自动降低吞吐量；当网络畅通时，也会适当增加吞吐量。
- **readBufferSize**: number
  - 单个连接的读取缓冲区大小，单位是 MB。默认值为 2。
- **writeBufferSize**: number
  - 单个连接的写入缓冲区大小，单位是 MB。默认值为 2。

● readBufferSize 和 writeBufferSize 指定了单个连接所使用的内存大小。在需要高速传输时，指定较大的 readBufferSize 和 writeBufferSize 会在一定程度上提高速度，但也会使用更多的内存。在网速不超过 20MB/s 时，默认值 1MB 可以满足需求；超过之后，可以适当增加 readBufferSize 和 writeBufferSize 的值，然后手动平衡速度和内存的关系。

- **header**: [HeaderObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HeaderObject_mKCP)
  - 数据包头部伪装设置。



HeaderObject

```
{
  "type": "none"
}
```

 

其中：

- **type**: string
  - 伪装类型，可选的值有：
    - “none“: 默认值，不进行伪装，发送的数据是没有特征的数据包。
    - “srtp“: 伪装成 SRTP 数据包，会被识别为视频通话数据（如 FaceTime）。
    - “utp“: 伪装成 uTP 数据包，会被识别为 BT 下载数据。
    - “wechat-video“: 伪装成微信视频通话的数据包。
    - “dtls“: 伪装成 DTLS 1.2 数据包。
    - “wireguard“: 伪装成 WireGuard 数据包。(并不是真正的 WireGuard 协议)

鸣谢

- @skywind3000 发明并实现了 KCP 协议；
- @xtaci 将 KCP 由 C 语言实现翻译成 Go；
- @xiaokangwang 测试 KCP 与 V2Ray 的整合并提交了最初的 PR。

对 KCP 协议的改进

**更小的协议头**

原生 KCP 协议使用了 24 字节的固定头部，而 mKCP 修改为数据包 18 字节，确认（ACK）包 16 字节。更小的头部有助于躲避特征检查，并加快传输速度。

另外，原生 KCP 的单个确认包只能确认一个数据包已收到，也就是说当 KCP 需要确认 100 个数据已收到时，它会发出 24 * 100 = 2400 字节的数据。其中包含了大量重复的头部数据，造成带宽的浪费。mKCP 会对多个确认包进行压缩，100 个确认包只需要 16 + 2 + 100 * 4 = 418 字节，相当于原生的六分之一。

**确认包重传**

原生 KCP 协议的确认（ACK）包只发送一次，如果确认包丢失，则一定会导致数据重传，造成不必要的带宽浪费。而 mKCP 会以一定的频率重发确认包，直到发送方确认为止。单个确认包的大小为 22 字节，相比起数据包的 1000 字节以上，重传确认包的代价要小得多。

**连接状态控制**

mKCP 可以有效地开启和关闭连接。当远程主机主动关闭连接时，连接会在两秒钟之内释放；当远程主机断线时，连接会在最多 30 秒内释放。

原生 KCP 不支持这个场景。

## **WebSocket 传输方式**

使用标准的 WebSocket 来传输数据。WebSocket 连接可以被其它 HTTP 服务器（如 NGINX）分流。

● Websocket 会识别 HTTP 请求的 X-Forwarded-For 头来用做流量的源地址。

WebSocketObject

```
{
  "path": "/",
  "headers": {
    "Host": "v2ray.com"
  }
}
```

 

其中：

- **path**: string
  - WebSocket 所使用的 HTTP 协议路径，默认值为 “/”。
- **headers**: map{string: string}
  - 自定义 HTTP 头，一个键值对，每个键表示一个 HTTP 头的名称，对应的值是字符串。默认值为空。

## **HTTP/2 传输方式**

V2Ray 3.17 中加入了基于 HTTP/2 的传输方式。它完整按照 HTTP/2 标准实现，可以通过其它的 HTTP 服务器（如 Nginx）进行中转。

由 HTTP/2 的建议，客户端和服务器必须同时开启 TLS 才可以正常使用这个传输方式。

HttpObject

HttpObject对应传输配置中的httpSettings项。

```
{
  "host": ["v2ray.com"],
  "path": "/random/path"
}
```

 

其中：

- **host**: [string]
  - 一个字符串数组，每一个元素是一个域名。客户端会随机从列表中选出一个域名进行通信，服务器会验证域名是否在列表中。
- **path**: string
  - HTTP 路径，由/开头。客户端和服务器必须一致。可选参数，默认值为”/“。

## **DomainSocket 传输方式**

Domain Socket 使用标准的 Unix domain socket 来传输数据。它的优势是使用了操作系统内建的传输通道，而不会占用网络缓存。相比起本地环回网络（local loopback）来说，Domain socket 速度略快一些。

目前仅可用于支持 Unix domain socket 的平台，如 macOS 和 Linux。在 Windows 上不可用。

● 如果指定了 domain socket 作为传输方式，在入站出站代理中配置的端口和 IP 地址将会失效，所有的传输由 domain socket 取代。

DomainSocketObject

DomainSocketObject对应传输配置中的dsSettings项。

```
{
  "path": "/path/to/ds/file"
}
```

 

其中：

- **path**: string
  - 一个合法的文件路径。在运行 V2Ray 之前，这个文件必须不存在。

## **QUIC 传输方式**

QUIC 全称 Quick UDP Internet Connection，是由 Google 提出的使用 UDP 进行多路并发传输的协议。其主要优势是:

1. 减少了握手的延迟（1-RTT 或 0-RTT）
2. 多路复用，并且没有 TCP 的阻塞问题
3. 连接迁移，（主要是在客户端）当由 Wifi 转移到 4G 时，连接不会被断开。

QUIC 目前处于实验期，使用了正在标准化过程中的 IETF 实现，不能保证与最终版本的兼容性。

版本历史

V2Ray 4.7:

- 开始支持 QUIC。
- 默认设定:
  - 12 字节的 Connection ID
  - 30 秒没有数据通过时自动断开连接 (可能会影响一些长连接的使用)

QuicObject

QUIC 的配置对应传输配置中的 quicSettings 项。对接的两端的配置必须完全一致，否则连接失败。QUIC 强制要求开启 TLS，在传输配置中没有开启 TLS 时，V2Ray 会自行签发一个证书进行 TLS 通讯。在使用 QUIC 传输时，可以关闭 VMess 的加密。

```
{
  "security": "none",
  "key": "",
  "header": {
    "type": "none"
  }
}
```

 

其中：

- **security**: “none” | “aes-128-gcm” | “chacha20-poly1305”
  - 加密方式。默认值为不加密。
  - 此加密是对 QUIC 数据包的加密，加密后数据包无法被探测。
- **key**: string
  - 加密时所用的密钥。可以是任意字符串。当security不为”none”时有效。
- **header**: [HeaderObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#HeaderObject_quic)
  - 数据包头部伪装设置



HeaderObject

```
{
  "type": "none"
}
```

 

其中：

- **type**: string
  - 伪装类型，可选的值有：
    - “none“: 默认值，不进行伪装，发送的数据是没有特征的数据包。
    - “srtp“: 伪装成 SRTP 数据包，会被识别为视频通话数据（如 FaceTime）。
    - “utp“: 伪装成 uTP 数据包，会被识别为 BT 下载数据。
    - “wechat-video“: 伪装成微信视频通话的数据包。
    - “dtls“: 伪装成 DTLS 1.2 数据包。
    - “wireguard“: 伪装成 WireGuard 数据包。(并不是真正的 WireGuard 协议)

● 当加密和伪装都不启用时，数据包即为原始的 QUIC 数据包，可以与其它的 QUIC 工具对接。为了避免被探测，建议加密或伪装至少开启一项。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **统计信息**

V2Ray 提供了一些关于其运行状况的统计信息。

## StatsObject

StatsObject 对应配置文件中的stats项。

```
{
}
```

 

目前统计信息没有任何参数，只要StatsObject项存在，内部的统计即会开启。同时你还需要在[本地策略](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#policy)中开启对应的项，才可以统计对应的数据。

目前已有的统计信息如下：

## 用户数据

- user>>>[email]>>>traffic>>>uplink
  - 特定用户的上行流量，单位字节。
- user>>>[email]>>>traffic>>>downlink
  - 特定用户的下行流量，单位字节。

● 如果对应用户没有指定 Email，则不会开启统计。

## 全局数据

- inbound>>>[tag]>>>traffic>>>uplink
  - 特定入站代理的上行流量，单位字节。
- inbound>>>[tag]>>>traffic>>>downlink
  - 特定入站代理的下行流量，单位字节。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------

# **反向代理**

反向代理是一个 V2Ray 的附加功能，可以把服务器端的流量向客户端转发，即逆向流量转发。

反向代理功能在 V2Ray 4.0+ 可用。目前处于测试阶段，可能会有一些问题。

反向代理的大致工作原理如下:

- 假设在主机 A 中有一个网页服务器，这台主机没有公网 IP，无法在公网上直接访问。另有一台主机 B，它可以由公网访问。现在我们需要把 B 作为入口，把流量从 B 转发到 A。
- 在主机 A 中配置一个 V2Ray，称为bridge，在 B 中也配置一个 V2Ray，称为portal。
- bridge会向portal主动建立连接，此连接的目标地址可以自行设定。portal会收到两种连接，一是由bridge发来的连接，二是公网用户发来的连接。portal会自动将两类连接合并。于是bridge就可以收到公网流量了。
- bridge在收到公网流量之后，会将其原封不动地发给主机 A 中的网页服务器。当然，这一步需要路由的协作。
- bridge会根据流量的大小进行动态的负载均衡。

反向代理默认已开启 [Mux](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#mux)，请不要在其用到的出站代理上再次开启 Mux。

## ReverseObject

ReverseObject对应配置文件中的reverse项。

```
{
  "bridges": [{
    "tag": "bridge",
    "domain": "test.v2ray.com"
  }],
  "portals": [{
    "tag": "portal",
    "domain": "test.v2ray.com"
  }]
}
```

 

其中：

- **bridges**: [[BridgeObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#BridgeObject_reverse)]
  - 一个数组，每一项表示一个bridge。每个bridge的配置是一个 BridgeObject。
- **portals**: [[PortalObject](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#PortalObject_reverse)]
  - 一个数组，每一项表示一个portal。每个portal的配置是一个 PortalObject。



BridgeObject



```
{
  "tag": "bridge",
  "domain": "test.v2ray.com"
}
```

 

其中：

- **tag**: string
  - 一个标识，所有由 bridge 发出的连接，都会带有这个标识。可以在路由中使用inboundTag进行识别。
- **domain**: string
  - 一个域名。bridge 向 portal 建立的连接，都会使用这个域名进行发送。这个域名只作为 bridge 和 portal 的通信用途，不必真实存在。



PortalObject

- **tag**: string
  - portal 的标识。在路由中使用 outboundTag 将流量转发到这个 portal。
- **domain**: string
  - 一个域名。当 portal 接收到流量时，如果流量的目标域名是此域名，则 portal 认为当前连接上 bridge 发来的通信连接。而其它流量则会被当成需要转发的流量。portal 所做的工作就是把这两类连接进行识别并拼接。

● 和其它配置一样，一个 V2Ray 既可以作为bridge，也可以作为portal，也可以同时两者，以适用于不同的场景需要。

## 完整配置

bridge通常需要两个出站代理，一个用于连接portal，另一个用于发送实际的流量。也就是说，你需要用路由区分两种流量。

反向代理配置:

```
{
  "bridges": [{
    "tag": "bridge",
    "domain": "test.v2ray.com"
  }]
}
```

 

出站代理:

```
{
  "tag": "out",
  "protocol": "freedom",
  "settings": {
    "redirect": "127.0.0.1:80" // 将所有流量转发到网页服务器
  }
},
{
  "protocol": "vmess",
  "settings": {
    "vnext": [{
      "address": "portal的IP地址",
      "port": 1024,
      "users": [{"id": "27848739-7e62-4138-9fd3-098a63964b6b"}]
    }]
  },
  "tag": "interconn"
}
```

 

路由配置:

```
"routing": {
  "strategy": "rules",
  "settings": {
    "rules": [{
      "type": "field",
      "inboundTag": ["bridge"],
      "domain": ["full:test.v2ray.com"],
      "outboundTag": "interconn"
    },{
      "type": "field",
      "inboundTag": ["bridge"],
      "outboundTag": "out"
    }]
  }
}
```

 

portal通常需要两个入站代理，一个用于接收bridge的连接，另一个用于接收实际的流量。同时你也需要用路由区分两种流量。

反向代理配置:

```
{
  "portals": [{
    "tag": "portal",
    "domain": "test.v2ray.com"  // 必须和 bridge 的配置一样
  }]
}
```

 

入站代理:

```
{
  "tag": "external",
  "port": 80,  // 开放 80 端口，用于接收外部的 HTTP 访问
  "protocol": "dokodemo-door",
  "settings": {
    "address": "127.0.0.1",
    "port": 80,
    "network": "tcp"
  }
},
{
  "port": 1024, // 用于接收 bridge 的连接
  "tag": "interconn",
  "protocol": "vmess",
  "settings": {
    "clients": [{"id": "27848739-7e62-4138-9fd3-098a63964b6b"}]
  }
}
```

 

路由配置:

```
"routing": {
  "strategy": "rules",
  "settings": {
    "rules": [{
      "type": "field",
      "inboundTag": ["external"],
      "outboundTag": "portal"
    },{
      "type": "field",
      "inboundTag": ["interconn"],
      "outboundTag": "portal"
    }]
  }
}
```

在运行过程中，建议先启用bridge，再启用portal。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **环境变量**

V2Ray 提供以下环境变量以供修改 V2Ray 的一些底层配置。

## 每个连接的缓存大小

- 名称: v2ray.ray.buffer.size 或 V2RAY_RAY_BUFFER_SIZE
- 单位: MBytes
- 默认值: 在 x86、amd64、arm64、s390x 上为 2，其它平台上禁用该缓存。
- 特殊值: 0 表示缓存无上限

**已过时，请使用本地策略中的 bufferSize**

对于一个代理连接，当上下游网络速度有差距时，V2Ray 会缓存一部分数据，以减小对网络传输的影响。这个配置设置了缓存的大小，越大的缓存会占用更多的内存，也会使网络性能越好。

## 资源文件路径

- 名称: v2ray.location.asset 或 V2RAY_LOCATION_ASSET
- 默认值: 和 v2ray 文件同路径

这个环境变量指定了一个文件夹位置，这个文件夹应当包含 geoip.dat 和 geosite.dat 文件。

## 配置文件位置

- 名称: v2ray.location.config 或 V2RAY_LOCATION_CONFIG
- 默认值: 和 v2ray 文件同路径

这个环境变量指定了一个文件夹位置，这个文件夹应当包含 config.json 文件。

## 分散读取

- 名称：v2ray.buf.readv 或 V2RAY_BUF_READV
- 默认值：auto

V2Ray 3.37 开始使用 Scatter/Gather IO，这一特性可以在大流量（超过 100 MByte/s）的时候依然使用较低的内存。可选的值有auto、enable和disable。

- **enable**: 强制开启分散读取特性。
- **disable**: 强制关闭分散读取特性
- **auto**: 仅在 Windows、MacOS、Linux 并且 CPU 平台为 x86、AMD64、s390x 时，开启此特性。

在流量没有达到 100 MByte/s 时，开启与否在内存使用上没有明显的差异。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **配置实例**

这里有一些例子以供参考

## V2Ray使用Shadowsocks协议

客户端配置：

```
{
  "inbounds": [
    {
      "port": 1080, // 监听端口
      "protocol": "socks", // 入口协议为 SOCKS 5
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth"  // 不认证
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "shadowsocks",
      "settings": {
        "servers": [
          {
            "address": "serveraddr.com", // Shadowsocks 的服务器地址
            "method": "aes-128-gcm", // Shadowsocks 的加密方式
            "ota": true, // 是否开启 OTA，true 为开启
            "password": "sspasswd", // Shadowsocks 的密码
            "port": 1024  
          }
        ]
      }
    }
  ]
}
```

 

服务器配置：

```
{
  "inbounds": [
    {
      "port": 1024, // 监听端口
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-128-gcm",
        "ota": true, // 是否开启 OTA
        "password": "sspasswd"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",  
      "settings": {}
    }
  ]
}
```

 

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **mKCP**

客户端配置：

```
{
  "inbounds": [
    {
      "port": 1080,
      "protocol": "socks",
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "serveraddr.com",
            "port": 16823,
            "users": [
              {
                "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "mkcp",
        "kcpSettings": {
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": true,
          "header": {
            "type": "none"
          }
        }
      }
    }
  ]
}
```

 

服务器配置：

```
{
  "inbounds": [
    {
      "port": 16823,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "mkcp", //此处的 mkcp 也可写成 kcp，两种写法是起同样的效果
        "kcpSettings": {
          "uplinkCapacity": 5,
          "downlinkCapacity": 100,
          "congestion": true,
          "header": {
            "type": "none"
          }
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

 

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **国内直连**

客户端配置：

```
{
  "log": {
    "loglevel": "warning",
    "access": "D:\\v2ray\\access.log",
    "error": "D:\\v2ray\\error.log"
  },
  "inbounds": [
    {
      "port": 1080,
      "protocol": "socks",
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth",
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "serveraddr.com",
            "port": 16823,  
            "users": [
              {
                "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
                "alterId": 64
              }
            ]
          }
        ]
      }
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct" //如果要使用路由，这个 tag 是一定要有的，在这里 direct 就是 freedom 的一个标号，在路由中说 direct V2Ray 就知道是这里的 freedom 了
    }    
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "outboundTag": "direct",
        "domain": ["geosite:cn"] // 中国大陆主流网站的域名
      },
      {
        "type": "chinaip",
        "outboundTag": "direct",
        "ip": [
          "geoip:cn"，// 中国大陆的 IP
          "geoip:private" // 私有地址 IP，如路由器等
        ]
      }
    ]
  }
}
```

 

服务器配置：

```
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": 16823,
      "protocol": "vmess",    
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811"
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

 

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **广告过滤**

客户端配置：

```
{
  "log": {
    "loglevel": "warning",
    "access": "D:\\v2ray\\access.log",
    "error": "D:\\v2ray\\error.log"
  },
  "inbounds": [
    {
      "port": 1080,
      "protocol": "socks",
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth",
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "serveraddr.com",
            "port": 16823,
            "users": [
              {
                "id": "2b831381d-6324-4d53-ad4f-8cda48b30811",  
                "alterId": 64
              }
            ]
          }
        ]
      }
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"//如果要使用路由，这个 tag 是一定要有的，在这里 direct 就是 freedom 的一个标号，在路由中说 direct V2Ray 就知道是这里的 freedom 了
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "adblock"//同样的，这个 tag 也是要有的，在路由中说 adblock 就知道是这里的 blackhole（黑洞） 了
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "domain": [
          "tanx.com",
          "googeadsserving.cn",
          "baidu.com"
        ],
        "type": "field",
        "outboundTag": "adblock"       
      },
      {
        "domain": [
          "amazon.com",
          "microsoft.com",
          "jd.com",
          "youku.com",
          "baidu.com"
        ],
        "type": "field",
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "outboundTag": "direct"，
        "domain": ["geosite:cn"]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "ip": [
          "geoip:cn",
          "geoip:private"
        ]
      }
    ]
  }
}
```

 

服务器配置：

```
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": 16823,
      "protocol": "vmess",    
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

 

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

------



# **高级功能**

本章节介绍了 V2Ray 的高级功能。

## **均衡负载**

相较于 Shadowsocks 等其它的代理，V2Ray 可以配置多服务器实现负载均衡。此处的负载均衡并非是自动选择一个延迟或网速最好的服务器进行连接，而是指多个服务器共同承担网络流量，从而减小单个服务器的资源占用及提高每个服务器的利用率。

服务器配置：

实现负载均衡很简单，在客户端配置同一个 outbound 的 vnext 写入各个服务器的配置即可(前提是服务器已经部署好并能正常使用)。形如：

```
{
  "inbound": {
  ...
  },
  "outbound": {
    "protocol": "vmess",
    "settings": {
      "vnext": [
        {
          "address": "address_of_vps1",
          "port": 8232,
          "users": [
            {
              "id": "1ce383ea-13e9-4939-9b1d-20d67135969a",
              "alterId": 64
            }
          ]
        },
        {
          "address": "address_of_vps2",
          "port": 4822,
          "users": [
            {
              "id": "bc172445-4b5e-49b2-a712-12c5295fd26b",
              "alterId": 64
            }
          ]
        }
        // 如果还有更多服务器可继续添加
      ]
    },
    "streamSettings": {
    ...
    }
  },
  ...
}
```

 

原理：V2Ray 是以轮询的方式均衡负载，也就是说当有流量需要通过代理时，首先走第一个 vnext 配置的服务器，有第二个连接就走第二个服务器，接着第三个，以此类推。轮询完一遍又头开始轮询。这样的方式虽然简单粗暴，特别是对于拥有多个性能差的 VPS 的人来说比较有用。另外**也许**能减小长时间大流量连接单个 IP 的特征，给自己一个心理安慰。

注意事项：如端口、id 这些在 vnext 数组内的配置项可以各不相同，但是它们的传输层配置（streamSettings）必须一致。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **动态端口**

通常情况下，V2Ray 的服务器端使用一个固定的端口来接收客户端的数据。这个端口由配置文件中的 port 属性指定。如果同一端口通信时间过长，或流量过大，则有可能被服务商限速。于是 V2Ray 提供了一个功能来动态调整通信端口。

在新的配置中，V2Ray 服务器端依然使用一个主端口（即上文的 port）接收请求，但可以配置一个绕路（detour）的特性。配置之后，服务器会主动告诉客户端，使用一个新的端口 X 来通信，X 是一个范围（可配置）内随机选取的值。此端口的有效期为 Y 分钟，客户端和服务器都会遵守这个时间，到期之后，客户端会继续向服务器请求新的端口来通信。以此循环。

要启用动态端口，需要在现有的服务器配置文件中进行一些修改，主要有以下两项。客户端配置不用更改，客户端会自动接收服务器的配置。

到目前为止没有证据可以动态端口对于科学上网是加分项还是减分项。

基本动态端口



服务器 inbound 的端口作为主端口，在 inboundDetour 开动态监听的端口，客户端不用额外设定，客户端会先与服务器的主端口通信协商下一个使用的端口号。

服务器配置：

```
{
  "inbounds":[
  { //主端口配置
      "port": 37192,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "d17a1af7-efa5-42ca-b7e9-6a35282d737f",
            "alterId": 64
          }
        ],
        "detour": { //绕行配置，即指示客户端使用 dynamicPort 的配置通信
          "to": "dynamicPort"   
        }
      }
    },
    {
      "protocol": "vmess",
      "port": "10000-20000", // 端口范围
      "tag": "dynamicPort",  // 与上面的 detour to 相同
      "settings": {
        "default": {
          "alterId": 64
        }
      },
      "allocate": {            // 分配模式
        "strategy": "random",  // 随机开启
        "concurrency": 2,      // 同时开放两个端口,这个值最大不能超过端口范围的 1/3
        "refresh": 3           // 每三分钟刷新一次
      }
    }
  ]
}
```

 

客户端配置：

```
{
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "1.2.3.4",
            "port": 37192,
            "users": [
              {
                "id": "d17a1af7-efa5-42ca-b7e9-6a35282d737f",
                "alterId": 64
              }
            ]
          }
        ]
      }
    }
  ]
}
```

 

动态端口使用 mKCP



在对应的 inbounds 和 outbounds 加入 streamSettings 并将 network 设置为 kcp 即可。

服务器配置：

```
{
  "inbounds": [
    {
      "port": 37192,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "d17a1af7-efa5-42ca-b7e9-6a35282d737f",
            "level": 1,
            "alterId": 64
          }
        ],
        "detour": {        
          "to": "dynamicPort"   
        }
      },
      "streamSettings": {
        "network": "kcp"
      }
    },
    {
      "protocol": "vmess",
      "port": "10000-20000", // 端口范围
      "tag": "dynamicPort",       
      "settings": {
        "default": {
          "level": 1,
          "alterId": 32
        }
      },
      "allocate": {            // 分配模式
        "strategy": "random",  // 随机开启
        "concurrency": 2,      // 同时开放两个端口
        "refresh": 3           // 每三分钟刷新一次
      },
      "streamSettings": {
        "network": "kcp"
      }
    }
  ]
}
```

 

客户端配置：

```
{
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "1.2.3.4",
            "port": 37192,
            "users": [
              {
                "id": "d17a1af7-efa5-42ca-b7e9-6a35282d737f",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "kcp"
      }
    }
  ]
}
```

 

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## 代理转发

通常，如果用了一台 VPS 进行流量中转，我们都要有这台 VPS 操作权限对 VPS 进行一定设置才能将客户端发来的数据包中转到目的 VPS。但是，很多情况下，我们只有别人提供的 Shadowsocks 账户而不具有 VPS 操作权限，除非是这台 VPS 的所有者进行相关设置，一般我们是无法利用该 VPS 作为中转服务器。

V2Ray 自 v2.6 起新增了一个传出代理功能，使用代理转发可以实现由一个 Shadowsocks 服务器或者 V2Ray(VMess) 服务器来中转你的网络流量。传出代理解决了两个问题：一是可以别人使用提供的 Shadowsocks 帐户所属的 VPS 作为中转服务器（也就是说不需要对服务器进行额外的配置或设定）；二是基于一的情况下该 VPS 中转的数据包是经过 V2Ray 加密后的数据包，无法获知你访问了什么网站的数据，只能得到目的 VPS 的 IP 以及客户端 IP，有效地保证隐私安全。

**原理**

参考这个 [issue](https://github.com/v2ray/v2ray-core/issues/306)

**使用方法**

**所有服务器都不需要额外的设定，只需修改客户端的配置文件**

客户端配置：

```
{
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": { // settings 的根据实际情况修改
        "vnext": [
          {
            "address": "1.1.1.1",
            "port": 8888,
            "users": [
              {
                "alterId": 64,
                "id": "b12614c5-5ca4-4eba-a215-c61d642116ce"
              }
            ]
          }
        ]
      },
      "proxySettings": {
          "tag": "transit"  // 这里的 tag 必须跟作为代理 VPS 的 tag 一致，这里设定的是 "transit"
        }
    },
    {
      "protocol": "shadowsocks",
      "settings": {
        "servers": [
          {
            "address": "2.2.2.2",
            "method": "aes-256-cfb",
            "ota": false,
            "password": "password",
            "port": 1024
          }
        ]
      },
      "tag": "transit"
    }
  ]
}
```

 

**链式代理转发**

如果你有多个 Shadowsocks 或 VMess 账户，那么你可以这样:

```
{
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": { // settings 的根据实际情况修改
        "vnext": [
          {
            "address": "1.1.1.1",
            "port": 8888,
            "users": [
              {
                "alterId": 64,
                "id": "b12614c5-5ca4-4eba-a215-c61d642116ce"
              }
            ]
          }
        ]
      },
      "tag": "DOUS",
      "proxySettings": {
          "tag": "DOSG"  
        }
    },
    {
      "protocol": "shadowsocks",
      "settings": {
        "servers": [
          {
            "address": "2.2.2.2",
            "method": "aes-256-cfb",
            "ota": false,
            "password": "password",
            "port": 1024
          }
        ]
      },
      "tag": "AliHK"
    },
    {
      "protocol": "shadowsocks",
      "settings": {
        "servers": [
          {
            "address": "3.3.3.3",
            "method": "aes-256-cfb",
            "ota": false,
            "password": "password",
            "port": 3442
          }
        ]
      },
      "tag": "AliSG",
      "proxySettings": {
          "tag": "AliHK"  
      }
    },
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "4.4.4.4",
            "port": 8462,
            "users": [
              {
                "alterId": 64,
                "id": "b27c24ab-2b5a-433e-902c-33f1168a7902"
              }
            ]
          }
        ]
      },
      "tag": "DOSG",
      "proxySettings": {
          "tag": "AliSG"  
      }
    },
  ]
}
```

 

那么数据包经过的节点依次为： PC -> AliHK -> AliSG -> DOSG -> DOUS -> 目标网站

这样的代理转发形成了一条链条，被称之为链式代理转发。

**注意：如果你打算配置(动态)链式传出代理，应当明确几点：**

- **性能**。链式代理使用了多个节点，可能会造成延时、带宽等网络性能问题，并且客户端对每一个加解密的次数取决于代理链的长度，理论上也会有一定的影响。
- **安全**。前文提到，传出代理会一定程度上提高安全性，但安全取决于最弱一环，并不意味着代理链越长就会越安全。如果你需要匿名，请考虑成熟的匿名方案。 另外，使用了传出代理 streamSettings 会失效，即只能是非 TLS、无 HTTP 伪装的 TCP 传输协议。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **TLS**

申请SSL证书：

参考[申请Let‘s Encrypt免费SSL证书](https://ailitonia.com/archives/申请lets-encrypt免费ssl证书/)，或者[这篇文章](https://ailitonia.com/archives/v2ray利用websockettlsnginx实现流量伪装/)中的申请证书部分。

服务器配置：

```
{
  "inbounds": [
    {
      "port": 443, // 建议使用 443 端口
      "protocol": "vmess",    
      "settings": {
        "clients": [
          {
            "id": "23ad6b10-8d1a-40f7-8ad0-e3e35cd38297",  
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls", // security 要设置为 tls 才会启用 TLS
        "tlsSettings": {
          "certificates": [
            {
              "certificateFile": "/etc/v2ray/v2ray.crt", // 证书文件
              "keyFile": "/etc/v2ray/v2ray.key" // 密钥文件
            }
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

 

客户端配置：

```
{
  "inbounds": [
    {
      "port": 1080,
      "protocol": "socks",
      "domainOverride": ["tls","http"],
      "settings": {
        "auth": "noauth"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "mydomain.me", // tls 需要域名，所以这里应该填自己的域名
            "port": 443,
            "users": [
              {
                "id": "23ad6b10-8d1a-40f7-8ad0-e3e35cd38297",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "tls" // 客户端的 security 也要设置为 tls
      }
    }
  ]
}
```

 

温馨提醒：不要想当然地把在 SS 和 SSR 的观念带过来，V2Ray 的 TLS 不是伪装，也不是混淆，这是**完整、真正的 TLS**。因此才需要域名和证书。下面用的的 WS(WebSocket) 也不是伪装。

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **WebSocket+TLS+Web**

请参见：[V2Ray利用WebSocket+TLS+nginx+CDN实现流量伪装](https://ailitonia.com/archives/v2ray利用websockettlsnginx实现流量伪装/)，密码V2

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **CDN**

请参见：[V2Ray利用WebSocket+TLS+nginx+CDN实现流量伪装](https://ailitonia.com/archives/v2ray利用websockettlsnginx实现流量伪装/)，密码同上

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)

## **反向代理**

请参见：[使用v2ray进行反向代理/内网穿透](https://ailitonia.com/archives/使用v2ray进行反向代理-内网穿透/)

[↑返回目录](https://ailitonia.com/archives/v2ray完全配置指南/comment-page-1/#index)