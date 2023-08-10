## 1 安装
官方文档
https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management

### 1.1 基础环境
```bash
#各个机器设置自己的域名
hostnamectl set-hostname xxxx


# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### 1.2 安装kubelet、kubeadm、kubectl
安装这三个工具时要注意版本，版本偏差太多会造成不可预期的错误。参考官方[支持的版本偏差](https://kubernetes.io/zh-cn/releases/version-skew-policy/)

>  `kubeadm`：用来初始化集群的指令。
>  `kubelet`：在集群中的每个节点上用来启动 Pod 和容器等。
>  `kubectl`：用来与集群通信的命令行工具。

```bash
# 配置阿里镜像
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


sudo yum install -y kubelet-1.22.6 kubeadm-1.22.6 kubectl-1.22.6 --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

### 1.3 使用kubeadm引导集群
#### 1.3.1 下载镜像
`registry.aliyuncs.com/google_containers` 是定时同步kubernetes的镜像到阿里镜像仓库服务的，但只是K8S组件的镜像，阿里云镜像仓库有谷歌和RedHat的镜像，但是不全。
当我们下载k8s.gcr.io，gcr.io镜像和quay.io镜像，可以把k8s.gcr.io，gcr.io， quay.io镜像换成阿里云镜像下载

```bash
# 默认仓库可能连不上，查看阿里镜像
% kubeadm config images list --image-repository registry.aliyuncs.com/google_containers

I0810 22:38:00.233864   13354 version.go:255] remote version is much newer: v1.27.4; falling back to: stable-1.22
registry.aliyuncs.com/google_containers/kube-apiserver:v1.22.17
registry.aliyuncs.com/google_containers/kube-controller-manager:v1.22.17
registry.aliyuncs.com/google_containers/kube-scheduler:v1.22.17
registry.aliyuncs.com/google_containers/kube-proxy:v1.22.17
registry.aliyuncs.com/google_containers/pause:3.5
registry.aliyuncs.com/google_containers/etcd:3.5.0-0
registry.aliyuncs.com/google_containers/coredns:v1.8.4

# 可以直接使用docker下载也可以用下面命令下载
% kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers

```

### 1.4 初始化主节点
```bash
#所有机器添加master域名映射，以下需要修改为自己的
echo "172.31.0.4  cluster-endpoint" >> /etc/hosts

#主节点初始化
kubeadm init \
--apiserver-advertise-address=172.31.0.4 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```

