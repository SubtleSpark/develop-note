## 1 安装 k8s
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
`registry.aliyuncs.com/google_containers` 是定时同步kubernetes 的镜像到阿里镜像仓库服务的，但只是K8S组件的镜像，阿里云镜像仓库有谷歌和RedHat的镜像，但是不全。
当我们下载 k8s.gcr.io，gcr.io镜像和quay.io镜像，可以把k8s.gcr.io，gcr.io， quay.io 镜像换成阿里云镜像下载

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
echo "192.168.31.31  cluster-endpoint" >> /etc/hosts

#主节点初始化
kubeadm init \
--apiserver-advertise-address=192.168.31.31 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.22.17 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=172.25.0.0/16

```
**需要注意的点：**
- apiserver-advertise-address 是本机ip，且要保证集群中的节点都可以访问这个ip。
- control-plane-endpoint、apiserver-advertise-address 对应 hosts 中的配置
- service-cidr、pod-network-cidr 在内网网段中选择不可以重叠。apiserver-advertise-address 也不能在这两个网段中。*实测在 pod-network-cidr 网段中也可以成功初始化，但是教程说不行，这样可能会有坑。*
- 设置网段注意几个常用网段：
| 网段           | 备注                     |
| -------------- | ------------------------ |
| 10.0.0.0/8     | 内网                     |
| 172.16.0.0/12  | 内网  16二进制 0001,0000 |
| 192.168.0.0/16 | 内网                     |
| 172.17.0.0/16  | 默认 docker0 占用，尽量避免       |

执行成功后会有下面提示：
```bash
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join cluster-endpoint:6443 --token 7v4i30.lhx0egt9js4n32bz \
        --discovery-token-ca-cert-hash sha256:a1ea82cc82f3e7531ef5ae3d8961fb2362c800e70ae2c20f363ec8ecf2d0df5b \
        --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token 7v4i30.lhx0egt9js4n32bz \
        --discovery-token-ca-cert-hash sha256:a1ea82cc82f3e7531ef5ae3d8961fb2362c800e70ae2c20f363ec8ecf2d0df5b 
```

### 1.5 安装 calico 网络插件
[官方安装指南](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico)

- 下载 manifest
``` bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml -O
```
- 修改其中的 CALICO_IPV4POOL_CIDR 属性，和 init 时 pod-network-cidr 设置成一样。
- 使用 kubectl 安装
```bash
kubectl apply -f calico.yaml
```

## 2 存储抽象

### 2.1 环境准备

#### 2.1.1 所有节点

```shell
#所有机器安装
yum install -y nfs-utils
```

#### 2.1.2 主节点

```shell
# nfs主节点配置
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports

mkdir -p /nfs/data
systemctl enable rpcbind --now
systemctl enable nfs-server --now

# 使配置生效
exportfs -r
```

#### 2.1.3 从节点
注意下面出现 ip 的地方是 nfs server 端的 ip
```shell
# 查看有哪些文件夹可以挂载
showmount -e 192.168.31.31

#执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
mkdir -p /nfs/data
mount -t nfs 192.168.31.31:/nfs/data /nfs/data

# 写入一个测试文件
echo "hello nfs server" > /nfs/data/test.txt
```

### 2.2 原生方式数据挂载示例
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-pv-demo
  name: nginx-pv-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pv-demo
  template:
    metadata:
      labels:
        app: nginx-pv-demo
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:      # 定义数据卷的挂载方式
        - name: html
          nfs:      # 除了 nfs，还支持其他的挂载方式
            server: 192.168.31.31 # 这里设置nfs的主机
            path: /nfs/data/nginx-pv
```
server 注意要配置为 nfs 服务端的 ip

**原生数据挂载方式的缺点**
- 挂载的文件夹要自己创建
- pod 删除后，无法自动清理
- 没有限制最大使用的空间

### 2.3 PV&PVC

> *PV：持久卷（**Persistent Volume**），将应用需要持久化的数据保存到指定位置*
> *PVC：持久卷申明（**Persistent Volume Claim**），申明需要使用的持久卷规格*

#### 2.3.1 创建 PV 池（静态供应）
下面的 yaml 创建了两个静态 PV。第一个叫 pv01-10m，最大 10M，`ReadWriteMany`表示可同时被多个节点读写。


```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv01-10m
spec:
  capacity:
    storage: 10M
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/01
    server: 192.168.31.31
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv02-1gi
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/02
    server: 192.168.31.31
```

`---` 代表分隔两个 yaml 配置，等效于将资源配置写到两个文件中分别 apply。
storageClassName 可以自己指定，但要和后面 PVC 的配置相匹配。
```shell
# 查看创建的 PV 池
kubectl get pv -A
```


#### 2.3.2 创建PVC
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: nfs
# 创建了一个叫做 nginx-pvc 的 PVC，需要 200M 使用 nfs（创建PV时使用的 storageClassName）
```

## 3 安装

## 4 调度
为一个节点去除污点
```shell
# $node 为一个节点的 hostname，可以通过 kubectl get nodes -A 查看

kubectl taint nodes $node node-role.kubernetes.io/master:NoSchedule-

```
