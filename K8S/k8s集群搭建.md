- **minikube**
  只是一个 K8S 集群模拟器，只有一个节点的集群，只为测试用，master 和 worker 都在一起。

- **裸机安装**
  至少需要两台机器（主节点、工作节点个一台），需要自己安装 Kubernetes 组件，配置会稍微麻烦点。
  缺点：配置麻烦，缺少生态支持，例如负载均衡器、云存储。

- **直接用云平台 Kubernetes**
  可视化搭建，只需简单几步就可以创建好一个集群。
  优点：安装简单，生态齐全，负载均衡器、存储等都给你配套好，简单操作就搞定

- **k3s**

  安装简单，脚本自动完成。

  优点：轻量级，配置要求低，安装简单，生态齐全。

# 集群搭建

## 0 环境准备

- 节点数量: 3 台虚拟机 centos7
- 硬件配置: master节点4G / node节点2G RAM，2个CPU或更多的CPU，硬盘至少30G 以上
- 网络要求: 多个节点之间网络互通，每个节点能访问外网

## 1 集群规划

- k8s-node1：10.15.0.5
- k8s-node2：10.15.0.6
- k8s-node3：10.15.0.7

## 2 设置主机名

```shell
$ hostnamectl set-hostname k8s-node1  
$ hostnamectl set-hostname k8s-node2
$ hostnamectl set-hostname k8s-node3
```

## 3 同步 hosts 文件

>  如果 DNS 不支持主机名称解析，还需要在每台机器的 `/etc/hosts` 文件中添加主机名和 IP 的对应关系：

```shell
cat >> /etc/hosts <<EOF
10.15.0.5 k8s-node1
10.15.0.6 k8s-node2
10.15.0.7 k8s-node3
EOF
```

## 4 关闭防火墙

```shell
systemctl stop firewalld && systemctl disable firewalld
```

## 5 关闭 SELINUX

> 注意: ARM 架构请勿执行,执行会出现 ip 无法获取问题!

```shell
setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

## 6 关闭 swap 分区

```shell
swapoff -a && sed -ri 's/.*swap.*/#&/' /etc/fstab
```

## 7 同步时间

```shell
yum install ntpdate -y
ntpdate time.windows.com
```

## 8 安装 containerd

```shell
# 安装 yum-config-manager 相关依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加 containerd yum 源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装 containerd
yum install  -y containerd.io cri-tools  
# 配置 containerd
cat >  /etc/containerd/config.toml <<EOF
disabled_plugins = ["restart"]
[plugins.linux]
shim_debug = true
[plugins.cri.registry.mirrors."docker.io"]
endpoint = ["https://frz7i079.mirror.aliyuncs.com"]
[plugins.cri]
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.2"
EOF
# 启动 containerd 服务 并 开机配置自启动
$ systemctl enable containerd && systemctl start containerd && systemctl status containerd 

# 配置 containerd 配置
$ cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

# 配置 k8s 网络配置
$ cat  > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 加载 overlay br_netfilter 模块
$ modprobe overlay
$ modprobe br_netfilter

# 查看当前配置是否生效
$ sysctl -p /etc/sysctl.d/k8s.conf
```

## 9 添加源

- 查看源

```shell
$ yum repolist
```

- 添加源 x86

```shell
$ cat <<EOF > kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
$ mv kubernetes.repo /etc/yum.repos.d/
```

- 添加源 ARM

```shell
$ cat << EOF > kubernetes.repo 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-aarch64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

$ mv kubernetes.repo /etc/yum.repos.d/
```

## 11 安装 k8s

```shell
# 安装最新版本
$ yum install -y kubelet kubeadm kubectl

# 指定版本安装
# yum install -y kubelet-1.26.0 kubectl-1.26.0 kubeadm-1.26.0

# 启动 kubelet
$ sudo systemctl enable kubelet && sudo systemctl start kubelet && sudo systemctl status kubelet
```

## 12 初始化集群

- **`注意: 初始化 k8s 集群仅仅需要再在 master 节点进行集群初始化!`**

```shell
$ kubeadm init \
--apiserver-advertise-address=192.168.5.8 \
--pod-network-cidr=10.244.0.0/16 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version=v1.30.0 \		# 安装指定（最新）版本
--cri-socket=unix:///var/run/containerd/containerd.sock

# 添加新节点
$ kubeadm token create --print-join-command --ttl=0
$ kubeadm join 10.15.0.21:6443 --token xjm7ts.gu3ojvta6se26q8i --discovery-token-ca-cert-hash sha256:14c8ac5c04ff9dda389e7c6c505728ac1293c6aed5978c3ea9c6953d4a79ed34 
```

## 13 配置集群网络

### flannel插件

```sh
$ curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f kube-flannel.yaml
namespace/kube-flannel created
```



### calico插件

```sh
$ wget --no-check-certificate https://projectcalico.docs.tigera.io/archive/v3.25/manifests/calico.yaml
# 修改 calico.yaml 文件
$ vim calico.yaml
# 在 - name: CLUSTER_TYPE 下方添加如下内容
- name: CLUSTER_TYPE
  value: "k8s,bgp"
  # 下方为新增内容
- name: IP_AUTODETECTION_METHOD
  value: "interface=网卡名称"
# 例如：- name: IP_AUTODETECTION_METHOD
# 例如：  value: "interface=eth0" 可使用通配符，例如：interface="eth.*|en.*"
- name: CALICO_IPV4POOL_CIDR 
	value: "10.244.0.0/16"

$ kubectl apply -f calico.yaml
```



## 14 查看集群状态

```shell
# 查看集群节点状态 全部为 Ready 代表集群搭建成功
$ kubectl get nodes
NAME        STATUS   ROLES           AGE   VERSION
k8s-node1   Ready    control-plane   21h   v1.26.0
k8s-node2   Ready    <none>          21h   v1.26.0
k8s-node3   Ready    <none>          21h   v1.26.0

# 查看集群系统 pod 运行情况,下面所有 pod 状态为 Running 代表集群可用
$ kubectl get pod -A
NAMESPACE      NAME                                READY   STATUS    RESTARTS   AGE
default        nginx                               1/1     Running   0          21h
kube-flannel   kube-flannel-ds-gtq49               1/1     Running   0          21h
kube-flannel   kube-flannel-ds-qpdl6               1/1     Running   0          21h
kube-flannel   kube-flannel-ds-ttxjb               1/1     Running   0          21h
kube-system    coredns-5bbd96d687-p7q2x            1/1     Running   0          21h
kube-system    coredns-5bbd96d687-rzcnz            1/1     Running   0          21h
kube-system    etcd-k8s-node1                      1/1     Running   0          21h
kube-system    kube-apiserver-k8s-node1            1/1     Running   0          21h
kube-system    kube-controller-manager-k8s-node1   1/1     Running   0          21h
kube-system    kube-proxy-mtsbp                    1/1     Running   0          21h
kube-system    kube-proxy-v2jfs                    1/1     Running   0          21h
kube-system    kube-proxy-x6vhn                    1/1     Running   0          21h
kube-system    kube-scheduler-k8s-node1            1/1     Running   0          21h
```

# k8s命令 自动补全功能

```sh
yum install -y bash-completion 
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```



# 测试集群

```sh
[root@master1 ~]# kubectl create deploy nginx --image=nginx
deployment.apps/nginx created
[root@master1 ~]# kubectl expose deploy nginx --port=80 --type=NodePort
service/nginx exposed
[root@master1 ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        4d17h
nginx        NodePort    10.102.88.67   <none>        80:31524/TCP   5s
[root@master1 ~]# curl localhost:31524
```



# 查看集群状态

```sh
kubectl cluster-info
kubectl get nodes
kubectl get pods -A -o wide
```



# 快速切换k8s的context和namespace

```sh
kubectl krew install ctx
kubectl krew install ns
```



# [registry-proxy国外镜像代理](https://github.com/ketches/registry-proxy)

在 Kubernetes 集群中部署 Registry Proxy，自动帮助您使用镜像代理服务拉取新创建的 Pod 中的外网容器镜像（仅限公有镜像）。

[这个镜像代理服务，帮您在 K8S 中愉快地拉取国外镜像 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/663392492)



# 其他nodes节点上执行kubectl和kubadm方法

**方法一：拷贝master节点的/etc/kubernetes/admin.conf 到nodes节点中的同样的目录/etc/kubernetes/ ，然后再配置环境变量**

```
[root@k8s-node1 qq-5201351]# scp k8s-master:/etc/kubernetes/admin.conf /etc/kubernetes/admin.conf
```

然后再配置环境变量：

```
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bash_profile
source ~/.bash_profile
```

**方法二：拷贝master节点的/etc/kubernetes/admin.conf 到nodes节点的$HOME/.kube目录，并且命名为config就可以了**

因为默认是没有$HOME/.kube目录的，先进行创建

```sh
mkdir -p $HOME/.kube
scp k8s-master:/etc/kubernetes/admin.conf $HOME/.kube/config
```



# kubeadm reset的重置

1. **停止kubelet服务**：在执行kubeadm reset之前，需要首先停止kubelet服务。可以使用如下命令：

```sh
systemctl stop kubelet
```

1. **执行kubeadm reset命令**：在停止kubelet服务后，可以执行kubeadm reset命令。这个命令会删除Kubernetes组件的配置文件、证书和Kubernetes系统容器等。执行命令时，系统会询问是否确认执行，需要输入’y’进行确认。命令如下：

```sh
kubeadm reset
```

1. **删除/var/lib/kubelet目录下的数据**：kubeadm reset命令虽然会删除很多数据，但是/var/lib/kubelet目录下的数据并不会被删除。为了完全恢复到初始状态，我们需要手动删除这个目录下的数据。可以使用如下命令：

```shell
rm -rf /var/lib/kubelet
```

1. **重启kubelet服务**：在完成上述步骤后，需要重新启动kubelet服务。可以使用如下命令：

```sh
systemctl start kubelet
```

**重新初始化Kubernetes集群**
