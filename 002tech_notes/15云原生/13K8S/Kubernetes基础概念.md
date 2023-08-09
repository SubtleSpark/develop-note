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


sudo yum install -y kubelet-1.26.1 kubeadm-1.26.1 kubectl-1.26.1 --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```




