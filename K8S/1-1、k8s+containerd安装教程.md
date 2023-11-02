==Kubernetes[k8s] 最新版1.27.3安装教程，使用containerd模式==

**背景**
公司使用的是交老的k8s版本（1.16），由于老版本的K8s对于现在很多新特性不支持，所以需要升级到新版本。目前2023年7月11日最新版本的k8s是v1.27.3。通过参考官方文档进行k8s部署工作。其中涉及到操作系统配置、防火墙配置、私有镜像仓库等。

**环境**
操作系统：centos7.9
机器：1个master 和 1个node 节点

**安装**
**设置系统**

```sh
# 所有机器设置hostname
hostnamectl set-hostname  master1
hostnamectl set-hostname  node1
hostnamectl set-hostname  node2
# 所有机器增加内网ip和 master1 对应关系
vi /etc/hosts
如：
192.168.0.4 master1
192.168.0.5 node1
192.168.0.6 node2
```

**关闭防火墙**

```sh
# 注意，如果不关闭防火墙，需要将kubernates所有端口放行
systemctl stop firewalld.service 
systemctl disable firewalld.service
```

# 设置机器同步

可使用ntpdate，如果是各大厂的云服务器，也可以不设置，云服务器已设置好

**关闭交换空间**

```sh
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

**关闭selinux**

```sh
getenforce
cat /etc/selinux/config
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
cat /etc/selinux/config
```

**使用阿里云的Yum库**

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
# 是否开启本仓库
enabled=1
# 是否检查 gpg 签名文件
gpgcheck=0
# 是否检查 gpg 签名文件
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

配置网桥

```sh
#加载模块
modprobe  br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | tee /etc/sysctl.d/k8s.conf
vm.swappiness = 0
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sysctl --system
```

## 安装containerd

> 由于新的Kubernates [1.24.0以上] 建议使用contanerd, 而且kubernates如何使用containerd 不会像使用docker一样，要中间转几层，故其性能很好。大概CPU使用率减少60%，内存使用率能减少12%。

```sh
yum install wget -y
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo 

yum install -y yum-utils device-mapper-persistent-data lvm2
yum install -y containerd.io

systemctl stop containerd.service

containerd config default > /etc/containerd/config.toml
sed -i "s#registry.k8s.io/pause#registry.cn-hangzhou.aliyuncs.com/google_containers/pause#g" /etc/containerd/config.toml

sed -i "s#SystemdCgroup = false#SystemdCgroup = true#g" /etc/containerd/config.toml

systemctl enable --now containerd.service
systemctl status containerd.service
```

安装k8s

```sh
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes --nogpgcheck
systemctl daemon-reload && systemctl restart kubelet && systemctl enable kubelet
```

初始化k8s master节点

```sh
kubeadm init --apiserver-advertise-address=192.168.0.4 \
--image-repository=registry.aliyuncs.com/google_containers \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--cri-socket=unix:///var/run/containerd/containerd.sock

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

## 增加k8s worker节点

> 利用上一步中 kubeadm init后产生的命令在work节点中执行

```sh
kubeadm join 172.16.64.9:6443 --token token.fake  --discovery-token-ca-cert-hash sha256:fake 	
```



# 安装网络插件

安装flannel网络插件

```sh
[root@node2 ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
[root@node2 ~]# kubectl apply -f kube-flannel.yaml
namespace/kube-flannel created
```

安装calico网络插件

```sh
wget --no-check-certificate https://projectcalico.docs.tigera.io/archive/v3.25/manifests/calico.yaml
# 修改 calico.yaml 文件
vim calico.yaml
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

kubectl apply -f calico.yaml
```

## 查看集群状态

```sh
kubectl cluster-info
kubectl get nodes
kubectl get pods -A -o wide
```



## 测试集群

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



## 集群在任意nodes节点上执行kubectl和kubadm命令的方法

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



# containerd命令

```sh
# 查看镜像
ctr image list
或者
crictl images
 
# 拉取镜像
ctr i pull --all-platforms registry.xxxxx/pause:3.2
或者
ctr i pull --user user:passwd --all-platforms registry.xxx.xx/pause:3.2
或者
crictl pull --creds user:passwd registry.xxx.xx/pause:3.2

# 镜像打tag
镜像标记tag
ctr -n k8s.io i tag registry.xxxxx/pause:3.2 k8s.gcr.io/pause:3.2
或者
ctr -n k8s.io i tag --force registry.xxxxx/pause:3.2 k8s.gcr.io/pause:3.2

# 删除镜像tag
ctr -n k8s.io i rm registry.xxxxx/pause:3.2

# 推送镜像
ctr images push --user user:passwd registry.xxxxx/pause:3.2
```





