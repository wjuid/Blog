### [CentOS上手工部署kubernetes集群]

## 环境说明

在下面的步骤中，我们将在三台CentOS系统的物理机上部署具有三个节点的kubernetes1.7.0集群。

角色分配如下：

**镜像仓库**：172.16.138.100，域名为 `harbor.suixingpay.com`，为私有镜像仓库，请替换为公共仓库或你自己的镜像仓库地址。

**Master**：172.16.138.171

**Node**：172.16.138.172，172.16.138.173

注意：172.16.138.171这台主机master和node复用。所有生成证书、执行kubectl命令的操作都在这台节点上执行。一旦node加入到kubernetes集群之后就不需要再登陆node节点了。

## 安装前的准备

1、在node节点上安装docker1.17.03.2.ce

2、关闭所有节点的SELinux

**永久方法 – 需要重启服务器**

修改`/etc/selinux/config`文件中设置SELINUX=disabled ，然后重启服务器。

**临时方法 – 设置系统参数**

使用命令`setenforce 0`

3、准备harbor私有镜像仓库

参考：https://github.com/vmware/harbor

## 提示

由于启用了 TLS 双向认证、RBAC 授权等严格的安全机制，建议**从头开始部署**，而不要从中间开始，否则可能会认证、授权等失败！

## 1、创建TLS证书和密钥

`kubernetes` 系统的各组件需要使用 `TLS` 证书对通信进行加密，本文档使用 `CloudFlare` 的 PKI 工具集 [cfssl](https://github.com/cloudflare/cfssl) 来生成 Certificate Authority (CA) 和其它证书；

**生成的 CA 证书和秘钥文件如下：**

- ca-key.pem
- ca.pem
- kubernetes-key.pem
- kubernetes.pem
- kube-proxy.pem
- kube-proxy-key.pem
- admin.pem
- admin-key.pem

**使用证书的组件如下：**

- etcd：使用 ca.pem、kubernetes-key.pem、kubernetes.pem；
- kube-apiserver：使用 ca.pem、kubernetes-key.pem、kubernetes.pem；
- kubelet：使用 ca.pem；
- kube-proxy：使用 ca.pem、kube-proxy-key.pem、kube-proxy.pem；
- kubectl：使用 ca.pem、admin-key.pem、admin.pem；
- kube-controller-manager：使用 ca-key.pem、ca.pem;

**注意：以下操作都在 master 节点即 172.16.138.171 这台主机上执行，证书只需要创建一次即可，以后在向集群中添加新节点时只要将 /etc/kubernetes/ 目录下的证书拷贝到新节点上即可。**

## 安装 `CFSSL`

**直接使用二进制源码包安装**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
chmod +x cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 创建 CA (Certificate Authority)

**创建 CA 配置文件**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
mkdir /root/ssl
cd /root/ssl
cfssl print-defaults config > config.json
cfssl print-defaults csr > csr.json
# 根据config.json文件的格式创建如下的ca-config.json文件
# 过期时间设置成了 87600h
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}EOF
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

字段说明

- `ca-config.json`：可以定义多个 profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile；
- `signing`：表示该证书可用于签名其它证书；生成的 ca.pem 证书中 `CA=TRUE`；
- `server auth`：表示client可以用该 CA 对server提供的证书进行验证；
- `client auth`：表示server可以用该CA对client提供的证书进行验证；

**创建 CA 证书签名请求**

创建 `ca-csr.json` 文件，内容如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ],
    "ca": {
       "expiry": "87600h"
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- "CN"：`Common Name`，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法；
- "O"：`Organization`，kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)；

**生成 CA 证书和私钥**

```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
$ ls ca*
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

## 创建 kubernetes 证书

创建 kubernetes 证书签名请求文件 `kubernetes-csr.json`：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "172.16.138.100",
      "172.16.138.171",
      "172.16.138.172",
      "172.16.138.173",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 如果 hosts 字段不为空则需要指定授权使用该证书的 **IP 或域名列表**，由于该证书后续被 `etcd` 集群和 `kubernetes master` 集群使用，所以上面分别指定了 `etcd` 集群、`kubernetes master` 集群的主机 IP 和 **`kubernetes` 服务的服务 IP**（一般是 `kube-apiserver` 指定的 `service-cluster-ip-range` 网段的第一个IP，如 10.254.0.1）。
- 这是最小化安装的kubernetes集群，包括一个私有镜像仓库，三个节点的kubernetes集群，以上物理节点的IP也可以更换为主机名。

**生成 kubernetes 证书和私钥**

```
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
$ ls kubernetes*
kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
```

## 创建 admin 证书

创建 admin 证书签名请求文件 `admin-csr.json`：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 后续 `kube-apiserver` 使用 `RBAC` 对客户端(如 `kubelet`、`kube-proxy`、`Pod`)请求进行授权；
- `kube-apiserver` 预定义了一些 `RBAC` 使用的 `RoleBindings`，如 `cluster-admin` 将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` 的**所有 API**的权限；
- O 指定该证书的 Group 为 `system:masters`，`kubelet` 使用该证书访问 `kube-apiserver` 时 ，由于证书被 CA 签名，所以认证通过，同时由于证书用户组为经过预授权的 `system:masters`，所以被授予访问所有 API 的权限；

**注意**：这个admin 证书，是将来生成管理员用的kube config 配置文件用的，现在我们一般建议使用RBAC 来对kubernetes 进行角色权限控制， kubernetes 将证书中的CN 字段 作为User， O 字段作为 Group

在搭建完 kubernetes 集群后，我们可以通过命令: `kubectl get clusterrolebinding cluster-admin -o yaml` ,查看到 `clusterrolebinding cluster-admin` 的 subjects 的 kind 是 Group，name 是 `system:masters`。 `roleRef` 对象是 `ClusterRole cluster-admin`。 意思是凡是 `system:masters Group` 的 user 或者 `serviceAccount` 都拥有 `cluster-admin` 的角色。 因此我们在使用 kubectl 命令时候，才拥有整个集群的管理权限。可以使用 `kubectl get clusterrolebinding cluster-admin -o yaml` 来查看。

生成 admin 证书和私钥：

```
$ cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
$ ls admin*
admin.csr  admin-csr.json  admin-key.pem  admin.pem
```

## 创建 kube-proxy 证书

创建 kube-proxy 证书签名请求文件 `kube-proxy-csr.json`：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- CN 指定该证书的 User 为 `system:kube-proxy`；
- `kube-apiserver` 预定义的 RoleBinding `cluster-admin` 将User `system:kube-proxy` 与 Role `system:node-proxier` 绑定，该 Role 授予了调用 `kube-apiserver` Proxy 相关 API 的权限；

生成 kube-proxy 客户端证书和私钥

## 校验证书

以 kubernetes 证书为例

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ openssl x509  -noout -text -in  kubernetes.pem
.......
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=BeiJing, L=BeiJing, O=k8s, OU=System, CN=kubernetes
        Validity
            Not Before: May  8 07:32:00 2018 GMT
            Not After : May  5 07:32:00 2028 GMT
        Subject: C=CN, ST=BeiJing, L=BeiJing, O=k8s, OU=System, CN=kubernetes
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
.........   
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                E8:56:76:B4:91:C6:E2:62:BA:9D:31:48:30:B8:EA:B8:11:C9:24:A8
            X509v3 Authority Key Identifier: 
.........
   
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 确认 `Issuer` 字段的内容和 `ca-csr.json` 一致；
- 确认 `Subject` 字段的内容和 `kubernetes-csr.json` 一致；
- 确认 `X509v3 Subject Alternative Name` 字段的内容和 `kubernetes-csr.json` 一致；
- 确认 `X509v3 Key Usage、Extended Key Usage` 字段的内容和 `ca-config.json` 中 `kubernetes` profile 一致；

### 使用 `cfssl-certinfo` 命令

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
cfssl-certinfo -cert kubernetes.pem
{
  "subject": {
    "common_name": "kubernetes",
    "country": "CN",
    "organization": "k8s",
    "organizational_unit": "System",
    "locality": "BeiJing",
    "province": "BeiJing",
    "names": [
      "CN",
      "BeiJing",
      "BeiJing",
      "k8s",
      "System",
      "kubernetes"
    ]
  },
  "issuer": {
    "common_name": "kubernetes",
    "country": "CN",
    "organization": "k8s",
    "organizational_unit": "System",
    "locality": "BeiJing",
    "province": "BeiJing",
    "names": [
      "CN",
      "BeiJing",
      "BeiJing",
      "k8s",
      "System",
      "kubernetes"
    ]
  },
  "serial_number": "149579304534773967289155011801948025930902452922",
  "sans": [
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local",
    "127.0.0.1",
    "172.16.138.100",
    "172.16.138.171",
    "172.16.138.172",
    "172.16.138.173",
    "10.254.0.1"
  ],
  "not_before": "2018-05-08T07:32:00Z",
  "not_after": "2028-05-05T07:32:00Z",
  "sigalg": "SHA256WithRSA",
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 分发证书

将生成的证书和秘钥文件（后缀名为`.pem`）拷贝到所有机器的 `/etc/kubernetes/ssl` 目录下备用；

```
 mkdir -p /etc/kubernetes/ssl
 cp *.pem /etc/kubernetes/ssl
```

 

## 2、安装kubectl命令行工具

## 下载 kubectl

注意请下载对应的Kubernetes版本的安装包。

```
wget https://dl.k8s.io/v1.6.0/kubernetes-client-linux-amd64.tar.gz
tar -xzvf kubernetes-client-linux-amd64.tar.gz
cp kubernetes/client/bin/kube* /usr/bin/
chmod a+x /usr/bin/kube*
```

## 3、创建 kubeconfig 文件

## 创建 TLS Bootstrapping Token

**Token auth file**

Token可以是任意的包含128 bit的字符串，可以使用安全的随机数发生器生成。

```
export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
cat > token.csv <<EOF
${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
```

**注意：在进行后续操作前请检查 `token.csv` 文件，确认其中的 `${BOOTSTRAP_TOKEN}` 环境变量已经被真实的值替换。**

***\*BOOTSTRAP_TOKEN\** 将被写入到 kube-apiserver 使用的 token.csv 文件和 kubelet 使用的 `bootstrap.kubeconfig` 文件，如果后续重新生成了 BOOTSTRAP_TOKEN，则需要：**

1. 更新 token.csv 文件，分发到所有机器 (master 和 node）的 /etc/kubernetes/ 目录下，分发到node节点上非必需；
2. 重新生成 bootstrap.kubeconfig 文件，分发到所有 node 机器的 /etc/kubernetes/ 目录下；
3. 重启 kube-apiserver 和 kubelet 进程；
4. 重新 approve kubelet 的 csr 请求；

```
cp token.csv /etc/kubernetes/
```

## 创建 kubelet bootstrapping kubeconfig 文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 cd /etc/kubernetes
 export KUBE_APISERVER="https://172.16.138.171:6443"

# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kubelet-bootstrap \
  --token=${BOOTSTRAP_TOKEN} \
  --kubeconfig=bootstrap.kubeconfig

# 设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kubelet-bootstrap \
  --kubeconfig=bootstrap.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- `--embed-certs` 为 `true` 时表示将 `certificate-authority` 证书写入到生成的 `bootstrap.kubeconfig` 文件中；
- 设置客户端认证参数时**没有**指定秘钥和证书，后续由 `kube-apiserver` 自动生成；

## 创建 kube-proxy kubeconfig 文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
export KUBE_APISERVER="https://172.16.138.171:6443"
# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kube-proxy \
  --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem \
  --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

#  设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 设置集群参数和客户端认证参数时 `--embed-certs` 都为 `true`，这会将 `certificate-authority`、`client-certificate` 和 `client-key` 指向的证书文件内容写入到生成的 `kube-proxy.kubeconfig` 文件中；
- `kube-proxy.pem` 证书中 CN 为 `system:kube-proxy`，`kube-apiserver` 预定义的 RoleBinding `cluster-admin` 将User `system:kube-proxy` 与 Role `system:node-proxier` 绑定，该 Role 授予了调用 `kube-apiserver` Proxy 相关 API 的权限；

## 安装kubectl命令行工具

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
export KUBE_APISERVER="https://172.16.138.171:6443"
# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER}

# 设置客户端认证参数
kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/ssl/admin.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/ssl/admin-key.pem

# 设置上下文参数
kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin

# 设置默认上下文
kubectl config use-context kubernetes
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- `admin.pem` 证书 OU 字段值为 `system:masters`，`kube-apiserver` 预定义的 RoleBinding `cluster-admin` 将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` 相关 API 的权限；
- 生成的 kubeconfig 被保存到 `~/.kube/config` 文件；

**注意：**`~/.kube/config`文件拥有对该集群的最高权限，请妥善保管。

 

## 分发 kubeconfig 文件

将两个 kubeconfig 文件分发到所有 Node 机器的 `/etc/kubernetes/` 目录

```
cp bootstrap.kubeconfig kube-proxy.kubeconfig /etc/kubernetes/
```

## 4、创建高可用 etcd 集群

## TLS 认证文件

需要为 etcd 集群创建加密通信的 TLS 证书，这里复用以前创建的 kubernetes 证书

```
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/kubernetes/ssl
```

- kubernetes 证书的 `hosts` 字段列表中包含上面三台机器的 IP，否则后续证书校验会失败；

## 下载二进制文件

到 `https://github.com/coreos/etcd/releases` 页面下载最新版本的二进制文件

```
wget https://github.com/coreos/etcd/releases/download/v3.1.5/etcd-v3.1.5-linux-amd64.tar.gz
tar -xvf etcd-v3.1.5-linux-amd64.tar.gz
mv etcd-v3.1.5-linux-amd64/etcd* /usr/local/bin
```

## 创建 etcd 的 systemd unit 文件

在/usr/lib/systemd/system/目录下创建文件etcd.service，内容如下。注意替换IP地址为你自己的etcd集群的主机IP。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --peer-cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --peer-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster infra1=https://172.16.138.171:2380,infra2=https://172.16.138.172:2380,infra3=https://172.16.138.173:2380 \
  --initial-cluster-state new \
  --data-dir=${ETCD_DATA_DIR}
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 指定 `etcd` 的工作目录为 `/var/lib/etcd`，数据目录为 `/var/lib/etcd`，需在启动服务前创建这个目录，否则启动服务的时候会报错“Failed at step CHDIR spawning /usr/bin/etcd: No such file or directory”；
- 为了保证通信安全，需要指定 etcd 的公私钥(cert-file和key-file)、Peers 通信的公私钥和 CA 证书(peer-cert-file、peer-key-file、peer-trusted-ca-file)、客户端的CA证书（trusted-ca-file）；
- 创建 `kubernetes.pem` 证书时使用的 `kubernetes-csr.json` 文件的 `hosts` 字段**包含所有 etcd 节点的IP**，否则证书校验会出错；
- `--initial-cluster-state` 值为 `new` 时，`--name` 的参数值必须位于 `--initial-cluster` 列表中；

环境变量配置文件`/etc/etcd/etcd.conf`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# [member]
ETCD_NAME=infra1
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="https://172.16.138.171:2380"
ETCD_LISTEN_CLIENT_URLS="https://172.16.138.171:2379"

#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://172.16.138.171:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://172.16.138.171:2379"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

这是172.16.138.171节点的配置，其他两个etcd节点只要将上面的IP地址改成相应节点的IP地址即可。ETCD_NAME换成对应节点的infra1/2/3。

## 启动 etcd 服务

```
mv etcd.service /usr/lib/systemd/system/
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
systemctl status etcd
```

在所有的 kubernetes master 节点重复上面的步骤，直到所有机器的 etcd 服务都已启动。

## 验证服务

在任意 kubernetes master 机器上执行如下命令：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ etcdctl \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  cluster-health
2018-05-08 05:55:53.668852 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
2018-05-08 05:55:53.670937 I | warning: ignoring ServerName for user-provided CA for backwards compatibility is deprecated
member ab044f0f6d623edf is healthy: got healthy result from https://172.16.138.173:2379
member cf3528b42907470b is healthy: got healthy result from https://172.16.138.172:2379
member eab584ea44e13ad4 is healthy: got healthy result from https://172.16.138.171:2379
cluster is healt
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 5、 部署master节点

kubernetes master 节点包含的组件：

- kube-apiserver
- kube-scheduler
- kube-controller-manager

目前这三个组件需要部署在同一台机器上。

- `kube-scheduler`、`kube-controller-manager` 和 `kube-apiserver` 三者的功能紧密相关；
- 同时只能有一个 `kube-scheduler`、`kube-controller-manager` 进程处于工作状态，如果运行多个，则需要通过选举产生一个 leader；

## TLS 证书文件

```
$ ls /etc/kubernetes/ssl
admin-key.pem  admin.pem  ca-key.pem  ca.pem  kube-proxy-key.pem  kube-proxy.pem  kubernetes-key.pem  kubernetes.pem
```

## 下载最新版本的二进制文件

从[changelog](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md)``下载 `client` 或 `server` tar包 文件

`server` 的 tarball `kubernetes-server-linux-amd64.tar.gz` 已经包含了 `client`(`kubectl`) 二进制文件，所以不用单独下载`kubernetes-client-linux-amd64.tar.gz`文件；

```
wget https://dl.k8s.io/v1.7.16/kubernetes-server-linux-amd64.tar.gz
tar -xzvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes
tar -xzvf  kubernetes-src.tar.gz
```

将二进制文件拷贝到指定路径

```
cp -r server/bin/{kube-apiserver,kube-controller-manager,kube-scheduler,kubectl,kube-proxy,kubelet} /usr/local/bin/
```

## 配置和启动 kube-apiserver

**创建 kube-apiserver的service配置文件**

service配置文件`/usr/lib/systemd/system/kube-apiserver.service`内容：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Kubernetes API Service
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
After=etcd.service

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
ExecStart=/usr/local/bin/kube-apiserver \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_ETCD_SERVERS \
        $KUBE_API_ADDRESS \
        $KUBE_API_PORT \
        $KUBELET_PORT \
        $KUBE_ALLOW_PRIV \
        $KUBE_SERVICE_ADDRESSES \
        $KUBE_ADMISSION_CONTROL \
        $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`/etc/kubernetes/config`文件的内容为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=true"

# How the controller-manager, scheduler, and proxy find the apiserver

KUBE_MASTER="--master=http://172.16.138.171:8080"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

该配置文件同时被kube-apiserver、kube-controller-manager、kube-scheduler、kubelet、kube-proxy使用。

apiserver配置文件`/etc/kubernetes/apiserver`内容为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
###
## kubernetes system config
##
## The following values are used to configure the kube-apiserver
##
#
## The address on the local server to listen to.
#KUBE_API_ADDRESS="--insecure-bind-address=test-001.jimmysong.io"
KUBE_API_ADDRESS="--advertise-address=172.16.138.171 --bind-address=172.16.138.171 --insecure-bind-address=172.16.138.171"
#
## The port on the local server to listen on.
#KUBE_API_PORT="--port=8080"
#
## Port minions listen on
#KUBELET_PORT="--kubelet-port=10250"
#
## Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=https://172.16.138.171:2379,https://172.16.138.172:2379,https://172.16.138.173:2379"
#
## Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
#
## default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=ServiceAccount,NamespaceLifecycle,NamespaceExists,LimitRanger,ResourceQuota"
#
## Add your own!
KUBE_API_ARGS="--authorization-mode=RBAC --runtime-config=rbac.authorization.k8s.io/v1beta1 --kubelet-https=true --experimental-bootstrap-token-auth --token-auth-file=/etc/kubernetes/token.csv --service-node-por
t-range=30000-32767 --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem --client-ca-file=/etc/kubernetes/ssl/ca.pem --service-account-key-file=/etc/ku
bernetes/ssl/ca-key.pem --etcd-cafile=/etc/kubernetes/ssl/ca.pem --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem --enable-swagger-ui=true --apiserver-coun
t=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/lib/audit.log --event-ttl=1h"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

-  `--experimental-bootstrap-token-auth` Bootstrap Token Authentication在1.9版本已经变成了正式feature，参数名称改为`--enable-bootstrap-token-auth`
- 如果中途修改过`--service-cluster-ip-range`地址，则必须将default命名空间的`kubernetes`的service给删除，使用命令：`kubectl delete service kubernetes`，然后系统会自动用新的ip重建这个service，不然apiserver的log有报错`the cluster IP x.x.x.x for service kubernetes/default is not within the service CIDR x.x.x.x/16; please recreate`
- `--authorization-mode=RBAC` 指定在安全端口使用 RBAC 授权模式，拒绝未通过授权的请求；
- kube-scheduler、kube-controller-manager 一般和 kube-apiserver 部署在同一台机器上，它们使用**非安全端口**和 kube-apiserver通信;
- kubelet、kube-proxy、kubectl 部署在其它 Node 节点上，如果通过**安全端口**访问 kube-apiserver，则必须先通过 TLS 证书认证，再通过 RBAC 授权；
- kube-proxy、kubectl 通过在使用的证书里指定相关的 User、Group 来达到通过 RBAC 授权的目的；
- 如果使用了 kubelet TLS Boostrap 机制，则不能再指定 `--kubelet-certificate-authority`、`--kubelet-client-certificate` 和 `--kubelet-client-key` 选项，否则后续 kube-apiserver 校验 kubelet 证书时出现 ”x509: certificate signed by unknown authority“ 错误；
- `--admission-control` 值必须包含 `ServiceAccount`；
- `--bind-address` 不能为 `127.0.0.1`；
- `runtime-config`配置为`rbac.authorization.k8s.io/v1beta1`，表示运行时的apiVersion；
- `--service-cluster-ip-range` 指定 Service Cluster IP 地址段，该地址段不能路由可达；
- 缺省情况下 kubernetes 对象保存在 etcd `/registry` 路径下，可以通过 `--etcd-prefix` 参数进行调整；
- 如果需要开通http的无认证的接口，则可以增加以下两个参数：`--insecure-port=8080 --insecure-bind-address=127.0.0.1`。注意，生产上不要绑定到非127.0.0.1的地址上

**启动kube-apiserver**

```
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver
```

## 配置和启动 kube-controller-manager

**创建 kube-controller-manager的serivce配置文件**

文件路径`/usr/lib/systemd/system/kube-controller-manager.service`

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/controller-manager
ExecStart=/usr/local/bin/kube-controller-manager \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_MASTER \
        $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

配置文件`/etc/kubernetes/controller-manager`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
###
# The following values are used to configure the kubernetes controller-manager

# defaults from config and apiserver should be adequate

# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --service-cluster-ip-range=10.254.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem --root-ca-file=/etc/kubernetes/ssl/ca.pem --leader-elect=true"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- `--service-cluster-ip-range` 参数指定 Cluster 中 Service 的CIDR范围，该网络在各 Node 间必须路由不可达，必须和 kube-apiserver 中的参数一致；
- `--cluster-signing-*` 指定的证书和私钥文件用来签名为 TLS BootStrap 创建的证书和私钥；
- `--root-ca-file` 用来对 kube-apiserver 证书进行校验，**指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件**；
- `--address` 值必须为 `127.0.0.1`，kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器；

### 启动 kube-controller-manager

```
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
systemctl status kube-controller-manager
```

我们启动每个组件后可以通过执行命令`kubectl get componentstatuses`，来查看各个组件的状态;

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$  kubectl get componentstatuses
NAME                 STATUS      MESSAGE                                                                                        ERROR
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: getsockopt: connection refused   
controller-manager   Healthy     ok                                                                                             
etcd-2               Healthy     {"health": "true"}                                                                             
etcd-0               Healthy     {"health": "true"}                                                                             
etcd-1               Healthy     {"health": "true"}                                 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 配置和启动 kube-scheduler

**创建 kube-scheduler的serivce配置文件**

文件路径`/usr/lib/systemd/system/kube-scheduler.service`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Kubernetes Scheduler Plugin
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/scheduler
ExecStart=/usr/local/bin/kube-scheduler \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBE_MASTER \
            $KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

配置文件`/etc/kubernetes/scheduler`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
###
# kubernetes scheduler config

# default config should be adequate

# Add your own!
KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- `--address` 值必须为 `127.0.0.1`，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器；

### 启动 kube-scheduler

```
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
systemctl status kube-scheduler
```

## 验证 master 节点功能

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$  kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-1               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
etcd-0               Healthy   {"health": "true"}   
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 6、安装flannel网络插件

所有的node节点都需要安装网络插件才能让所有的Pod加入到同一个局域网中，本文是安装flannel网络插件的参考文档。

建议直接使用yum安装flanneld，除非对版本有特殊需求，默认安装的是0.7.1版本的flannel。

```
yum install -y flannel
```

service配置文件`/usr/lib/systemd/system/flanneld.service`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/flanneld
EnvironmentFile=-/etc/sysconfig/docker-network
ExecStart=/usr/bin/flanneld-start \
  -etcd-endpoints=${FLANNEL_ETCD_ENDPOINTS} \
  -etcd-prefix=${FLANNEL_ETCD_PREFIX} \
  $FLANNEL_OPTIONS
ExecStartPost=/usr/libexec/flannel/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`/etc/sysconfig/flanneld`配置文件：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# Flanneld configuration options  
#
# # etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD_ENDPOINTS="https://172.16.138.171:2379,https://172.16.138.172:2379,https://172.16.138.173:2379"
#
# # etcd config key.  This is the configuration key that flannel queries
# # For address range assignment
FLANNEL_ETCD_PREFIX="/kube-centos/network"
#
# # Any additional options that you want to pass
FLANNEL_OPTIONS="-etcd-cafile=/etc/kubernetes/ssl/ca.pem -etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem -etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果是多网卡（例如vagrant环境），则需要在FLANNEL_OPTIONS中增加指定的外网出口的网卡，例如-iface=eth2

**在etcd中创建网络配置**

执行下面的命令为docker分配IP地址段。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
etcdctl --endpoints=https://172.16.138.171:2379,https://172.16.138.172:2379,https://172.16.138.173:2379 \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  mkdir /kube-centos/network

etcdctl --endpoints=https://172.16.138.171:2379,https://172.16.138.171:2379,https://172.16.138.171:2379 \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  mk /kube-centos/network/config '{"Network":"172.30.0.0/16","SubnetLen":24,"Backend":{"Type":"vxlan"}}'
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果你要使用`host-gw`模式，可以直接将vxlan改成`host-gw`即可。

**启动flannel**

```
systemctl daemon-reload
systemctl enable flanneld
systemctl start flanneld
systemctl status flanneld
```

现在查询etcd中的内容可以看到：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$  etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  ls /kube-centos/network/subnets

/kube-centos/network/subnets/172.30.71.0-24
/kube-centos/network/subnets/172.30.16.0-24
/kube-centos/network/subnets/172.30.58.0-24

$  etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  get /kube-centos/network/config

{"Network":"172.30.0.0/16","SubnetLen":24,"Backend":{"Type":"vxlan"}}

$ etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  get /kube-centos/network/subnets/172.30.14.0-24

{"PublicIP":"172.16.138.171","BackendType":"vxlan","BackendData":{"VtepMAC":"7e:0e:49:74:de:b3"}}

$  etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  get /kube-centos/network/subnets/172.30.16.0-24

{"PublicIP":"172.16.138.172","BackendType":"vxlan","BackendData":{"VtepMAC":"5a:ab:55:02:7f:96"}}

$ etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  get /kube-centos/network/subnets/172.30.58.0-24

{"PublicIP":"172.16.138.173","BackendType":"vxlan","BackendData":{"VtepMAC":"3a:37:7d:55:b7:77"}}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

如果可以查看到以上内容证明flannel已经安装完成，下一步是在node节点上安装和配置docker、kubelet、kube-proxy

## 7、部署node节点

Kubernetes node节点包含如下组件：

- Flanneld：参考上一节
- Docker1.17.03：docker的安装很简单，这里也不说了，但是需要注意docker的配置。
- kubelet：直接用二进制文件安装
- kube-proxy:直接用二进制文件安装

**注意**：每台 node 上都需要安装 flannel，master 节点上可以不安装。

**步骤简介**

1. 确认在上一步中我们安装配置的网络插件flannel已启动且运行正常
2. 安装配置docker后启动
3. 安装配置kubelet、kube-proxy后启动
4. 验证

## 目录和文件

我们再检查一下三个节点上，经过前几步操作我们已经创建了如下的证书和配置文件。

```
$ ls /etc/kubernetes/ssl
admin-key.pem  admin.pem  ca-key.pem  ca.pem  kube-proxy-key.pem  kube-proxy.pem  kubernetes-key.pem  kubernetes.pem
$ ls /etc/kubernetes/
apiserver  bootstrap.kubeconfig  config  controller-manager  kubelet  kube-proxy.kubeconfig  proxy  scheduler  ssl  token.csv
```

## 配置Docker

**yum方式安装的flannel**

修改docker的配置文件`/usr/lib/systemd/system/docker.service`，增加一条环境变量配置：

```
EnvironmentFile=-/run/flannel/docker
```

`/run/flannel/docker`文件是flannel启动后自动生成的，其中包含了docker启动时需要的参数。

## 启动docker

重启了docker后还要重启kubelet，这时又遇到问题，kubelet启动失败。报错：

```
Mar 31 16:44:41 k8s-master kubelet[81047]: error: failed to run Kubelet: failed to create kubelet: misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"
```

这是kubelet与docker的**cgroup driver**不一致导致的，kubelet启动的时候有个`—cgroup-driver`参数可以指定为"cgroupfs"或者“systemd”。

```
--cgroup-driver string                                    Driver that the kubelet uses to manipulate cgroups on the host.  Possible values: 'cgroupfs', 'systemd' (default "cgroupfs")
```

配置docker的service配置文件`/usr/lib/systemd/system/docker.service`，设置`ExecStart`中的`--exec-opt native.cgroupdriver=systemd`。

## 安装和配置kubelet

kubelet 启动时向 kube-apiserver 发送 TLS bootstrapping 请求，需要先将 bootstrap token 文件中的 kubelet-bootstrap 用户赋予 system:node-bootstrapper cluster 角色(role)， 然后 kubelet 才能有权限创建认证请求(certificate signing requests)：

```
cd /etc/kubernetes
kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --user=kubelet-bootstrap
```

- `--user=kubelet-bootstrap` 是在 `/etc/kubernetes/token.csv` 文件中指定的用户名，同时也写入了 `/etc/kubernetes/bootstrap.kubeconfig` 文件；

### 下载最新的kubelet和kube-proxy二进制文件

注意请下载对应的Kubernetes版本的安装包。

 

```
wget https://dl.k8s.io/v1.7.16/kubernetes-server-linux-amd64.tar.gz
tar -xzvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes
tar -xzvf  kubernetes-src.tar.gz
cp -r ./server/bin/{kube-proxy,kubelet} /usr/local/bin/
```

### 创建kubelet的service配置文件

文件位置`/usr/lib/systemd/system/kubelet.service`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/local/bin/kubelet \
            $KUBE_LOGTOSTDERR \
            $KUBE_LOG_LEVEL \
            $KUBELET_API_SERVER \
            $KUBELET_ADDRESS \
            $KUBELET_PORT \
            $KUBELET_HOSTNAME \
            $KUBE_ALLOW_PRIV \
            $KUBELET_POD_INFRA_CONTAINER \
            $KUBELET_ARGS
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

kubelet的配置文件`/etc/kubernetes/kubelet`。其中的IP地址更改为你的每台node节点的IP地址。

**注意：**在启动kubelet之前，需要先手动创建`/var/lib/kubelet`目录。

下面是kubelet的配置文件`/etc/kubernetes/kubelet`:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
###
## kubernetes kubelet (minion) config
#
## The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=172.16.138.171"
#
## The port for the info server to serve on
#KUBELET_PORT="--port=10250"
#
## You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=172.16.138.171"
#
## location of the api-server
## COMMENT THIS ON KUBERNETES 1.8+
KUBELET_API_SERVER="--api-servers=http://172.16.138.171:8080"
#
## pod infrastructure container
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=harbor.suixingpay.com/kube/pause-amd64:3.0"
#
## Add your own!
KUBELET_ARGS="--cgroup-driver=systemd --cluster-dns=10.254.0.2 --experimental-bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig --require-kubeconfig --cert
-dir=/etc/kubernetes/ssl --cluster-domain=cluster.local --hairpin-mode promiscuous-bridge --serialize-image-pulls=false"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 如果使用systemd方式启动，则需要额外增加两个参数`--runtime-cgroups=/systemd/system.slice --kubelet-cgroups=/systemd/system.slice`
- `--experimental-bootstrap-kubeconfig` 在1.9版本已经变成了`--bootstrap-kubeconfig`
- `--address` 不能设置为 `127.0.0.1`，否则后续 Pods 访问 kubelet 的 API 接口时会失败，因为 Pods 访问的 `127.0.0.1` 指向自己而不是 kubelet；
- 如果设置了 `--hostname-override` 选项，则 `kube-proxy` 也需要设置该选项，否则会出现找不到 Node 的情况；
- `"--cgroup-driver` 配置成 `systemd`，不要使用`cgroup`，否则在 CentOS 系统中 kubelet 将启动失败（保持docker和kubelet中的cgroup driver配置一致即可，不一定非使用`systemd`）。
- `--experimental-bootstrap-kubeconfig` 指向 bootstrap kubeconfig 文件，kubelet 使用该文件中的用户名和 token 向 kube-apiserver 发送 TLS Bootstrapping 请求；
- 管理员通过了 CSR 请求后，kubelet 自动在 `--cert-dir` 目录创建证书和私钥文件(`kubelet-client.crt` 和 `kubelet-client.key`)，然后写入 `--kubeconfig` 文件；
- 建议在 `--kubeconfig` 配置文件中指定 `kube-apiserver` 地址，如果未指定 `--api-servers` 选项，则必须指定 `--require-kubeconfig` 选项后才从配置文件中读取 kube-apiserver 的地址，否则 kubelet 启动后将找不到 kube-apiserver (日志中提示未找到 API Server），`kubectl get nodes` 不会返回对应的 Node 信息;
- `--cluster-dns` 指定 kubedns 的 Service IP(可以先分配，后续创建 kubedns 服务时指定该 IP)，`--cluster-domain` 指定域名后缀，这两个参数同时指定后才会生效；
- `--cluster-domain` 指定 pod 启动时 `/etc/resolve.conf` 文件中的 `search domain` ，起初我们将其配置成了 `cluster.local.`，这样在解析 service 的 DNS 名称时是正常的，可是在解析 headless service 中的 FQDN pod name 的时候却错误，因此我们将其修改为 `cluster.local`，去掉最后面的 ”点号“ 就可以解决该问题。
- `--kubeconfig=/etc/kubernetes/kubelet.kubeconfig`中指定的`kubelet.kubeconfig`文件在第一次启动kubelet之前并不存在，请看下文，当通过CSR请求后会自动生成`kubelet.kubeconfig`文件，如果你的节点上已经生成了`~/.kube/config`文件，你可以将该文件拷贝到该路径下，并重命名为`kubelet.kubeconfig`，所有node节点可以共用同一个kubelet.kubeconfig文件，这样新添加的节点就不需要再创建CSR请求就能自动添加到kubernetes集群中。同样，在任意能够访问到kubernetes集群的主机上使用`kubectl --kubeconfig`命令操作集群时，只要使用`~/.kube/config`文件就可以通过权限认证，因为这里面已经有认证信息并认为你是admin用户，对集群拥有所有权限。
- `KUBELET_POD_INFRA_CONTAINER` 是基础镜像容器，这里我用的是私有镜像仓库地址，**大家部署的时候需要修改为自己的镜像**。

### 启动kublet

```
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
```

### 通过 kublet 的 TLS 证书请求

kubelet 首次启动时向 kube-apiserver 发送证书签名请求，必须通过后 kubernetes 系统才会将该 Node 加入到集群。

查看未授权的 CSR 请求

```
$ kubectl get csr
NAME                                                   AGE       REQUESTOR           CONDITION
node-csr-0bi8ZxaLgRc4fUV1sGSsG6II84MMlEg-4ttACLGq3AE   21s       kubelet-bootstrap   Pending
$ kubectl get nodes
No resources found.
```

通过 CSR 请求

```
$ kubectl certificate approve node-csr-0bi8ZxaLgRc4fUV1sGSsG6II84MMlEg-4ttACLGq3AE
certificatesigningrequest "node-csr-0bi8ZxaLgRc4fUV1sGSsG6II84MMlEg-4ttACLGq3AE" approved
$ kubectl get nodes
NAME             STATUS    AGE       VERSION
172.16.138.171   Ready     6s        v1.7.16
```

自动生成了 kubelet kubeconfig 文件和公私钥

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ ls -l /etc/kubernetes/kubelet.kubeconfig
-rw------- 1 root root 2284 Apr  7 02:07 /etc/kubernetes/kubelet.kubeconfig
$ ls -l /etc/kubernetes/ssl/kubelet*
-rw-r--r-- 1 root root 1046 Apr  7 02:07 /etc/kubernetes/ssl/kubelet-client.crt
-rw------- 1 root root  227 Apr  7 02:04 /etc/kubernetes/ssl/kubelet-client.key
-rw-r--r-- 1 root root 1103 Apr  7 02:07 /etc/kubernetes/ssl/kubelet.crt
-rw------- 1 root root 1675 Apr  7 02:07 /etc/kubernetes/ssl/kubelet.key
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

假如你更新kubernetes的证书，只要没有更新`token.csv`，当重启kubelet后，该node就会自动加入到kuberentes集群中，而不会重新发送`certificaterequest`，也不需要在master节点上执行`kubectl certificate approve`操作。前提是不要删除node节点上的`/etc/kubernetes/ssl/kubelet*`和`/etc/kubernetes/kubelet.kubeconfig`文件。否则kubelet启动时会提示找不到证书而失败。

**注意：**如果启动kubelet的时候见到证书相关的报错，有个trick可以解决这个问题，可以将master节点上的`~/.kube/config`文件（该文件在[安装kubectl命令行工具](https://jimmysong.io/kubernetes-handbook/practice/kubectl-installation.html)这一步中将会自动生成）拷贝到node节点的`/etc/kubernetes/kubelet.kubeconfig`位置，这样就不需要通过CSR，当kubelet启动后就会自动加入的集群中。

## 配置 kube-proxy

**安装conntrack**

```
yum install -y conntrack-tools
```

**创建 kube-proxy 的service配置文件**

文件路径`/usr/lib/systemd/system/kube-proxy.service`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/proxy
ExecStart=/usr/local/bin/kube-proxy \
        $KUBE_LOGTOSTDERR \
        $KUBE_LOG_LEVEL \
        $KUBE_MASTER \
        $KUBE_PROXY_ARGS
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

kube-proxy配置文件`/etc/kubernetes/proxy`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
###
# kubernetes proxy config

# default config should be adequate

# Add your own!
KUBE_PROXY_ARGS="--bind-address=172.16.138.171 --hostname-override=172.16.138.171 --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig --cluster-cidr=10.254.0.0/16"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

- `--hostname-override` 参数值必须与 kubelet 的值一致，否则 kube-proxy 启动后会找不到该 Node，从而不会创建任何 iptables 规则；
- kube-proxy 根据 `--cluster-cidr` 判断集群内部和外部流量，指定 `--cluster-cidr` 或 `--masquerade-all` 选项后 kube-proxy 才会对访问 Service IP 的请求做 SNAT；
- `--kubeconfig` 指定的配置文件嵌入了 kube-apiserver 的地址、用户名、证书、秘钥等请求和认证信息；
- 预定义的 RoleBinding `cluster-admin` 将User `system:kube-proxy` 与 Role `system:node-proxier` 绑定，该 Role 授予了调用 `kube-apiserver` Proxy 相关 API 的权限；

### 启动 kube-proxy

```
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy
```

### 验证

我们创建一个nginx的service试一下集群是否可用。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$  kubectl run nginx --replicas=2 --labels="run=load-balancer-example" --image=index.tenxcloud.com/xjimmy/nginx:1.9.4  --port=80
deployment "nginx" created
$  kubectl expose deployment nginx --type=NodePort --name=example-service
service "example-service" exposed
$  kubectl describe svc example-service
Name:                   example-service
Namespace:              default
Labels:                 run=load-balancer-example
Annotations:            <none>
Selector:               run=load-balancer-example
Type:                   NodePort
IP:                     10.254.173.196
Port:                   <unset> 80/TCP
NodePort:               <unset> 31498/TCP
Endpoints:              172.17.0.2:80,172.17.0.3:80
Session Affinity:       None
Events:                 <none>
$  curl  10.254.173.196:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 8、安装kubedns插件

官方的yaml文件目录：`kubernetes/cluster/addons/dns`。

该插件直接使用kubernetes部署，官方的配置文件中包含以下镜像：

```
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1
gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1
gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1
```

我clone了上述镜像，上传到我的私有镜像仓库：

```
harbor.suixingpay.com/kube1.6/k8s-dns-kube-dns-amd64:1.14.1
harbor.suixingpay.com/kube1.6/k8s-dns-sidecar-amd64:1.14.1
harbor.suixingpay.com/kube1.6/k8s-dns-dnsmasq-nanny-amd64:1.14.1
```

以下yaml配置文件中使用的是私有镜像仓库中的镜像。

```
kubedns-cm.yaml  
kubedns-sa.yaml  
kubedns-controller.yaml  
kubedns-svc.yaml
```

###  系统预定义的 RoleBinding

预定义的 RoleBinding `system:kube-dns` 将 kube-system 命名空间的 `kube-dns` ServiceAccount 与 `system:kube-dns` Role 绑定， 该 Role 具有访问 kube-apiserver DNS 相关 API 的权限；

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ kubectl get clusterrolebindings system:kube-dns -o yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: 2018-05-10T02:17:04Z
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-dns
  resourceVersion: "91"
  selfLink: /apis/rbac.authorization.k8s.io/v1beta1/clusterrolebindings/system%3Akube-dns
  uid: 3b753e98-53f8-11e8-9a54-00505693535c
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-dns
subjects:
- kind: ServiceAccount
  name: kube-dns
  namespace: kube-system
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

`kubedns-controller.yaml` 中定义的 Pods 时使用了 `kubedns-sa.yaml` 文件定义的 `kube-dns` ServiceAccount，所以具有访问 kube-apiserver DNS 相关 API 的权限。

### 配置 kube-dns ServiceAccount

无需配置

### 配置 `kube-dns` 服务

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# __MACHINE_GENERATED_WARNING__

apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
    kubernetes.io/name: "KubeDNS"
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 10.254.0.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- spec.clusterIP = 10.254.0.2，即明确指定了 kube-dns Service IP，这个 IP 需要和 kubelet 的 `--cluster-dns` 参数值一致；

### 配置kube-dns Deployment

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Should keep target in cluster/addons/dns-horizontal-autoscaler/dns-horizontal-autoscaler.yaml
# in sync with this file.

# __MACHINE_GENERATED_WARNING__

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  # replicas: not specified here:
  # 1. In order to make Addon Manager do not reconcile this replicas parameter.
  # 2. Default is 1.
  # 3. Will be tuned in real time if DNS horizontal auto-scaling is turned on.
  strategy:
    rollingUpdate:
      maxSurge: 10%
      maxUnavailable: 0
  selector:
    matchLabels:
      k8s-app: kube-dns
  template:
    metadata:
      labels:
        k8s-app: kube-dns
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
      volumes:
      - name: kube-dns-config
        configMap:
          name: kube-dns
          optional: true
      containers:
      - name: kubedns
        image: harbor.suixingpay.com/kube1.6/k8s-dns-kube-dns-amd64:1.14.1
        resources:
          # TODO: Set memory limits when we've profiled the container for large
          # clusters, then set request = limit to keep this container in
          # guaranteed class. Currently, this container falls into the
          # "burstable" category so the kubelet doesn't backoff from restarting it.
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        livenessProbe:
          httpGet:
            path: /healthcheck/kubedns
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8081
            scheme: HTTP
          # we poll on pod startup for the Kubernetes master service and
          # only setup the /readiness HTTP server once that's available.
          initialDelaySeconds: 3
          timeoutSeconds: 5
        args:
        - --domain=cluster.local.
        - --dns-port=10053
        - --config-dir=/kube-dns-config
        - --v=2
        #__PILLAR__FEDERATIONS__DOMAIN__MAP__
        env:
        - name: PROMETHEUS_PORT
          value: "10055"
        ports:
        - containerPort: 10053
          name: dns-local
          protocol: UDP
        - containerPort: 10053
          name: dns-tcp-local
          protocol: TCP
        - containerPort: 10055
          name: metrics
          protocol: TCP
        volumeMounts:
        - name: kube-dns-config
          mountPath: /kube-dns-config
      - name: dnsmasq
        image: harbor.suixingpay.com/kube1.6/k8s-dns-dnsmasq-nanny-amd64:1.14.1
        livenessProbe:
          httpGet:
            path: /healthcheck/dnsmasq
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        args:
        - -v=2
        - -logtostderr
        - -configDir=/etc/k8s/dns/dnsmasq-nanny
        - -restartDnsmasq=true
        - --
        - -k
        - --cache-size=1000
        - --log-facility=-
        - --server=/cluster.local./127.0.0.1#10053
        - --server=/in-addr.arpa/127.0.0.1#10053
        - --server=/ip6.arpa/127.0.0.1#10053
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        # see: https://github.com/kubernetes/kubernetes/issues/29055 for details
        resources:
          requests:
            cpu: 150m
            memory: 20Mi
        volumeMounts:
        - name: kube-dns-config
          mountPath: /etc/k8s/dns/dnsmasq-nanny
      - name: sidecar
        image: harbor.suixingpay.com/kube1.6/k8s-dns-sidecar-amd64:1.14.1
        livenessProbe:
          httpGet:
            path: /metrics
            port: 10054
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        args:
        - --v=2
        - --logtostderr
        - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local.,5,A
        - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local.,5,A
        ports:
        - containerPort: 10054
          name: metrics
          protocol: TCP
        resources:
          requests:
            memory: 20Mi
            cpu: 10m
      dnsPolicy: Default  # Don't use cluster DNS.
      serviceAccountName: kube-dns
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 使用系统已经做了 RoleBinding 的 `kube-dns` ServiceAccount，该账户具有访问 kube-apiserver DNS 相关 API 的权限；

## 执行所有定义文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ pwd
/root/kubedns
$ ls *.yaml
kubedns-cm.yaml  kubedns-controller.yaml  kubedns-sa.yaml  kubedns-svc.yaml
$ kubectl create -f .configmap "kube-dns" createddeployment "kube-dns" createdserviceaccount "kube-dns" createdservice "kube-dns" create
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

## 检查 kubedns 功能

新建一个 Deployment

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ cat  my-nginx.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-nginx
spec:
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: index.tenxcloud.com/xjimmy/nginx:1.9.4
        ports:
        - containerPort: 80
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

Export 该 Deployment, 生成 `my-nginx` 服务

```
$ kubectl expose deploy my-nginx
$ kubectl get services --all-namespaces |grep my-nginx
default       my-nginx          10.254.89.137    <none>        80/TCP          5s
```

创建另一个 Pod，查看 `/etc/resolv.conf` 是否包含 `kubelet` 配置的 `--cluster-dns` 和 `--cluster-domain`，是否能够将服务 `my-nginx` 解析到 Cluster IP `10.254.89.137`。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 $  kubectl get pods --all-namespaces
NAMESPACE     NAME                        READY     STATUS    RESTARTS   AGEdefault       my-nginx-3466650801-bngns   1/1       Running   0          1hdefault       my-nginx-3466650801-q8gmv   1/1       Running   0          1hdefault       nginx-608366207-621q4       1/1       Running   0          22hdefault       nginx-608366207-84z2w       1/1       Running   0          22hkube-system   kube-dns-1041264494-l5lkl   3/3       Running   0          1h
$  kubectl get services --all-namespacesNAMESPACE     NAME              CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGEdefault       example-service   10.254.173.196   <nodes>       80:31498/TCP    20hdefault       kubernetes        10.254.0.1       <none>        443/TCP         1ddefault       my-nginx          10.254.89.137    <none>        80/TCP          3mkube-system   kube-dns          10.254.0.2       <none>        53/UDP,53/TCP   6m
$   kubectl exec my-nginx-3466650801-bngns -i -t -- /bin/bash 
root@my-nginx-3466650801-bngns:~# cat /etc/resolv.conf 
nameserver 10.254.0.2
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
root@my-nginx-3466650801-bngns:~#  ping my-nginx
PING my-nginx.default.svc.cluster.local (10.254.89.137): 56 data bytes
^C--- my-nginx.default.svc.cluster.local ping statistics ---
4 packets transmitted, 0 packets received, 100% packet loss
root@my-nginx-3466650801-bngns:~# ping kubernetes
PING kubernetes.default.svc.cluster.local (10.254.0.1): 56 data bytes
^C--- kubernetes.default.svc.cluster.local ping statistics ---
2 packets transmitted, 0 packets received, 100% packet loss
root@my-nginx-3466650801-bngns:~# ping example-service
PING example-service.default.svc.cluster.local (10.254.173.196): 56 data bytes
^C--- example-service.default.svc.cluster.local ping statistics ---
1 packets transmitted, 0 packets received, 100% packet loss
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

从结果来看，service名称可以正常解析。

**注意**：直接ping ClusterIP是ping不通的，ClusterIP是根据**IPtables**路由到服务的endpoint上，只有结合ClusterIP加端口才能访问到对应的服务。

## 9、安装dashboard插件

官方文件目录：https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/dashboard

我们使用的文件如下：

```
$ ls *.yaml
dashboard-controller.yaml  dashboard-service.yaml dashboard-rbac.yaml
```

由于 `kube-apiserver` 启用了 `RBAC` 授权，而官方源码目录的 `dashboard-controller.yaml` 没有定义授权的 ServiceAccount，所以后续访问 API server 的 API 时会被拒绝，web中提示：

```
orbidden (403)

User "system:serviceaccount:kube-system:default" cannot list jobs.batch in the namespace "default". (get jobs.batch)
```

增加了一个`dashboard-rbac.yaml`文件，定义一个名为 dashboard 的 ServiceAccount，然后将它和 Cluster Role view 绑定，如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard
  namespace: kube-system

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: dashboard
subjects:
  - kind: ServiceAccount
    name: dashboard
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

然后使用`kubectl apply -f dashboard-rbac.yaml`创建。

### 配置dashboard-service

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard
  ports:
  - port: 80
    targetPort: 9090
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 配置dashboard-controller

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  labels:
    k8s-app: kubernetes-dashboard
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      serviceAccountName: dashboard
      containers:
      - name: kubernetes-dashboard
        image: harbor.suixingpay.com/kube1.6/kubernetes-dashboard-amd64:v1.6.0
        resources:
          limits:
            cpu: 100m
            memory: 50Mi
          requests:
            cpu: 100m
            memory: 50Mi
        ports:
        - containerPort: 9090
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      tolerations:
      - key: "CriticalAddonsOnly"
        operator: "Exists"
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 执行所有定义文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ pwd
/root/kubedashboard
$ ls *.yaml
dashboard-controller.yaml  dashboard-service.yaml
$ kubectl create -f  .
service "kubernetes-dashboard" created
deployment "kubernetes-dashboard" created
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

###  检查执行结果

查看分配的 NodePort

```
$ kubectl get services kubernetes-dashboard -n kube-system
NAME                   CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes-dashboard   10.254.166.88   <nodes>       80:31304/TCP   14s
```

- NodePort 31304映射到 dashboard pod 80端口；

检查 controller

```
$ kubectl get deployment kubernetes-dashboard  -n kube-system
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-dashboard   1         1         1            1           20s
$ kubectl get pods  -n kube-system | grep dashboard
kubernetes-dashboard-2428500324-34tbz   1/1       Running   0          25s
```

### 访问dashboard

有以下三种方式：

- kubernetes-dashboard 服务暴露了 NodePort，可以使用 `http://NodeIP:nodePort` 地址访问 dashboard
- 通过 API server 访问 dashboard（https 6443端口和http 8080端口方式）
- 通过 kubectl proxy 访问 dashboard

### 通过 kubectl proxy 访问 dashboard

启动代理

```
$  kubectl proxy --address='172.16.138.171' --port=8086 --accept-hosts='^*$' 
Starting to serve on 172.16.138.171:8086
```

- 需要指定 `--accept-hosts` 选项，否则浏览器访问 dashboard 页面时提示 “Unauthorized”；

浏览器访问 URL：http://172.16.138.171:8086/ui 自动跳转到：http://172.16.138.171:8086/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default

### 通过 API server 访问dashboard

获取集群服务地址列表

```
$  kubectl cluster-info
Kubernetes master is running at https://172.16.138.171:6443
KubeDNS is running at https://172.16.138.171:6443/api/v1/namespaces/kube-system/services/kube-dns/proxy
kubernetes-dashboard is running at https://172.16.138.171:6443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

浏览器访问 https://172.16.138.171:6443/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard（浏览器会提示证书验证，因为通过加密通道，以改方式访问的话，需要提前导入证书到你的计算机中）。

如果你不想使用**https**的话，可以直接访问insecure port 8080端口：[http://172.16.138.171:8080/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard](http://172.20.0.113:8080/api/v1/proxy/namespaces/kube-system/services/kubernetes-dashboard)

 

## 10、安装heapster插件

### 准备YAML文件

```
wget https://github.com/kubernetes/heapster/archive/v1.3.0.zip
unzip v1.3.0.zip
mv v1.3.0.zip heapster-1.3.0
```

文件目录： `heapster-1.3.0/deploy/kube-config/influxdb`

```
$ cd heapster-1.3.0/deploy/kube-config/influxdb
$ ls *.yaml
grafana-deployment.yaml  grafana-service.yaml  heapster-deployment.yaml  heapster-service.yaml  influxdb-deployment.yaml  influxdb-service.yaml heapster-rbac.yaml
```

我们自己创建了heapster的rbac配置`heapster-rbac.yaml`。

### 配置 grafana-deployment 

grafana-deployment.yaml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: harbor.suixingpay.com/kube1.6/heapster-grafana-amd64:v4.0.2
        ports:
          - containerPort: 3000
            protocol: TCP
        volumeMounts:
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GRAFANA_PORT
          value: "3000"
          # The following env variables are required to make Grafana accessible via
          # the kubernetes api-server proxy. On production clusters, we recommend
          # removing these env variables, setup auth for grafana, and expose the grafana
          # service using a LoadBalancer or a public IP.
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          # If you're only using the API Server proxy, set this value instead:
          # value: /api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/
          value: /
      volumes:
      - name: grafana-storage
        emptyDir: {}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 如果后续使用 kube-apiserver 或者 kubectl proxy 访问 grafana dashboard，则必须将 `GF_SERVER_ROOT_URL` 设置为 `/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/`，否则后续访问grafana时访问时提示找不到`http://172.16.138.171:8086/api/v1/proxy/namespaces/kube-system/services/monitoring-grafana/api/dashboards/home` 页面；

### 配置 heapster-deployment

heapster-deployment.yaml

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      containers:
      - name: heapster
        image: harbor.suixingpay.com/kube1.6/heapster-amd64:v1.3.0-beta.1
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes:https://kubernetes.default
        - --sink=influxdb:http://monitoring-influxdb:8086
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 配置 influxdb-deployment

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$ # 导出镜像中的 influxdb 配置文件
$ docker run --rm --entrypoint 'cat'  -ti lvanneo/heapster-influxdb-amd64:v1.1.1 /etc/config.toml >config.toml.orig
$ cp config.toml.orig config.toml
$ # 修改：启用 admin 接口
$ vim config.toml
$ diff config.toml.orig config.toml
35c35
<   enabled = false
---
>   enabled = true
$ # 将修改后的配置写入到 ConfigMap 对象中
$ kubectl create configmap influxdb-config --from-file=config.toml  -n kube-system
configmap "influxdb-config" created
$ # 将 ConfigMap 中的配置文件挂载到 Pod 中，达到覆盖原始配置的目的
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: harbor.suixingpay.com/kube1.6/heapster-influxdb-amd64:v1.1.1
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
        - mountPath: /etc/config.toml
          name: influxdb-config
      volumes:
      - name: influxdb-storage
        emptyDir: {}
      - name: influxdb-config
        configMap:
          name: influxdb-config
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 配置 monitoring-influxdb Service

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 8086
    targetPort: 8086
    name: http
  - port: 8083
    targetPort: 8083
    name: admin
  selector:
    k8s-app: influxdb
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

- 定义端口类型为 NodePort，额外增加了 admin 端口映射，用于后续浏览器访问 influxdb 的 admin UI 界面；

### 配置  heapster-rbac

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
$  vim heapster-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system

---

kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: heapster
subjects:
  - kind: ServiceAccount
    name: heapster
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 执行所有定义文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```bash
$ pwd
/root/heapster-1.3.0/deploy/kube-config/influxdb$ ls *.yaml
grafana-service.yaml      heapster-rbac.yaml     influxdb-cm.yaml          influxdb-service.yaml
grafana-deployment.yaml  heapster-deployment.yaml  heapster-service.yaml  influxdb-deployment.yaml
$ kubectl create -f  .
deployment "monitoring-grafana" created
service "monitoring-grafana" created
deployment "heapster" created
serviceaccount "heapster" created
clusterrolebinding "heapster" created
service "heapster" created
configmap "influxdb-config" created
deployment "monitoring-influxdb" created
service "monitoring-influxdb" created
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

###  检查执行结果

检查 Deployment

```
$ kubectl get deployments -n kube-system | grep -E 'heapster|monitoring'
heapster               1         1         1            1           2m
monitoring-grafana     1         1         1            1           2m
monitoring-influxdb    1         1         1            1           2m
```

检查 Pods

```
$ kubectl get pods -n kube-system | grep -E 'heapster|monitoring'
heapster-110704576-gpg8v                1/1       Running   0          2m
monitoring-grafana-2861879979-9z89f     1/1       Running   0          2m
monitoring-influxdb-1411048194-lzrpc    1/1       Running   0          2m
```