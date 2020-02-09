# CentOS 8 install Kubernetes singe node



```csharp
# repo 
test -d /etc/yum.repos.d && cat > /etc/yum.repos.d/kubernetes.repo <<-"EOF" 
[kubernetes]
name=kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
gpgcheck=1

[docker-ce]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.ustc.edu.cn/docker-ce/linux/centos/7/$basearch/stable
gpgkey=https://mirrors.ustc.edu.cn/docker-ce/linux/centos/gpg
enabled=1
gpgcheck=1
EOF

yum install -y --nobest ipvsadm docker-ce kubernetes-cni kubeadm

systemctl enable docker kubelet


# disable swap
swapoff -a
sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab

# disable selinux
test -f /etc/selinux/config && setenforce 0 && sed -i "s/SELINUX=enforcing/SELINUX=permissive/" /etc/selinux/config

# disable firewalld
systemctl stop firewalld && systemctl disable firewalld

# treak docker
mkdir -p /etc/systemd/system/docker.service.d
cat > /etc/systemd/system/docker.service.d/override.conf <<-"EOF" 
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --registry-mirror=https://docker.mirrors.ustc.edu.cn  -s overlay2 --data-root /app/lib/docker --live-restore --exec-opt native.cgroupdriver=systemd --log-opt max-size=100m
ExecStartPost=/sbin/iptables -P FORWARD ACCEPT
EOF
systemctl daemon-reload
systemctl restart docker


# mod probe

cat > /etc/modules-load.d/kubernetes.conf <<EOF
br_netfilter
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
nf_conntrack_ipv4
EOF

modprobe br_netfilter
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack
modprobe nf_conntrack_ipv4 || true

cat > /etc/sysctl.d/66-k8s.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
EOF
sysctl -f /etc/sysctl.d/66-k8s.conf

cat > /etc/modprobe.d/ip_vs.conf <<-"EOF"
options ip_vs conn_tab_bits=20
EOF


NODEIPV4=$(ip -4 route get 8.8.8.8 | head -1 | awk '{print $7}')

cat > kube.yaml <<-EOF
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: InitConfiguration
# allow master run pods
nodeRegistration:
  taints:
  - effect: PreferNoSchedule
    key: node-role.kubernetes.io/master
  kubeletExtraArgs:
    pod-infra-container-image: gcr.azk8s.cn/google_containers/pause-amd64:3.1
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
#kubernetesVersion: stable  #v1.15.1
controllerManager:
  extraArgs:
    address: 0.0.0.0
scheduler:
  extraArgs:
    address: 0.0.0.0
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
apiServer:
  certSANs:
  - k8s
  - kubernetes
  - kubernetes.default
  - kubernetes.default.svc
  - kubernetes.default.svc.cluster
  - kubernetes.default.svc.cluster.local
  - 10.96.0.1
  - 127.0.0.1
  - $NODEIPV4
imageRepository: "gcr.azk8s.cn/google_containers"
EOF

# test config 
kubeadm init --config kube.yaml --dry-run 


# install 
kubeadm init --config kube.yaml

# kubectl config
mkdir -p /root/.kube
ln -s /etc/kubernetes/admin.conf  /root/.kube/config


# fetch info
kubectl get nodes -o wide
kubectl get pods --all-namespaces -o wide
kubectl get all --all-namespaces -o wide
```