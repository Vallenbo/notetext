- **minikube**
  只是一个 K8S 集群模拟器，只有一个节点的集群，只为测试用，master 和 worker 都在一起。

- **[KubeKey](https://github.com/kubesphere/kubekey)**
  一款用于部署 Kubernetes 集群的开源轻量级工具。它提供了一种灵活、快速和方便的方式来安装 Kubernetes/K3s，包括 Kubernetes/K3s 和 KubeSphere，以及相关的云原生插件。它也是扩展和升级集群的有效工具。
  
- **[kubeasz](https://github.com/easzlab/kubeasz)**
  
  致力于提供快速部署高可用`k8s`集群的工具, 同时也努力成为`k8s`实践、使用的参考书；基于二进制方式部署和利用`ansible-playbook`实现自动化；既提供一键安装脚本, 也可以根据`安装指南`分步执行安装各个组件。
  
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

# 查看集群状态

```sh
kubectl cluster-info
kubectl get nodes
kubectl get pods -A -o wide
```



# [metrics-server](https://github.com/kubernetes-sigs/metrics-server)

**Kubernetes Metrics Server** (Kubernetes指标服务器)，它是一个**可扩展的、高效的容器资源度量源**。Metrics Server 用于监控每个 Node 和 Pod 的负载（用于Kubernetes内置自动扩缩管道）。Metrics Server 从Kubelets 收集资源指标，并通过 Metrics API 在Kubernetes apiserver中公开，供 Horizontal Pod Autoscaler 和 Vertical Pod Autoscaler 使用。Metrics API 也可以通过 kubectl top 访问，使其更容易调试自动扩缩管道。

- 查看 metrics-server（或者其他资源指标 API `metrics.k8s.io` 服务提供者）是否正在运行， 请键入以下命令：

```shell
kubectl get apiservices
```

- 如果资源指标 API 可用，则会输出将包含一个对 `metrics.k8s.io` 的引用

```sh
NAME
v1beta1.metrics.k8s.io
```

- 安装 metrics-server

```yaml
# components.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
  - apiGroups:
      - metrics.k8s.io
    resources:
      - pods
      - nodes
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/metrics
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - pods
      - nodes
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
        - args:
            - --cert-dir=/tmp
            - --secure-port=4443
            - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
            - --kubelet-use-node-status-port
            - --metric-resolution=15s
            - --kubelet-insecure-tls #修改去掉证书验证
          image: dyrnq/metrics-server:v0.6.2 #修改官方无法下载
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /livez
              port: https
              scheme: HTTPS
            periodSeconds: 10
          name: metrics-server
          ports:
            - containerPort: 4443
              name: https
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /readyz
              port: https
              scheme: HTTPS
            initialDelaySeconds: 20
            periodSeconds: 10
          resources:
            requests:
              cpu: 100m
              memory: 200Mi
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
            - mountPath: /tmp
              name: tmp-dir
      hostNetwork: true  #必须指定这个才行
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
        - emptyDir: {}
          name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

```shell
$ kubectl appply -f components.yaml
```



# 快速切换k8s的context和namespace

```sh
kubectl krew install ctx
kubectl krew install ns
```



# [registry-proxy国外镜像代理](https://github.com/ketches/registry-proxy)

在 Kubernetes 集群中部署 Registry Proxy，自动帮助您使用镜像代理服务拉取新创建的 Pod 中的外网容器镜像（仅限公有镜像）。

## 快速安装

执行以下命令安装 registry-proxy（v1.1.0+ 版本）：

```sh
yum install epel-release -y
yum install jq -y

export VERSION=$(curl -s https://api.github.com/repos/ketches/registry-proxy/releases/latest | jq -r .tag_name)
# 代理地址
kubectl apply -f https://ghproxy.ketches.cn/https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/deploy/manifests.yaml

kubectl apply -f https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/deploy/manifests.yaml
```

## 卸载&清理

**卸载 registry-proxy**：

```sh
# uninstall v1.0.0 version for example
export VERSION=v1.0.0
kubectl delete -f https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/deploy/manifests.yaml

# 代理地址
kubectl delete -f https://ghproxy.ketches.cn/https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/deploy/manifests.yaml
```

**清理示例**：

```sh
kubectl delete -f https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/examples/dockerhub-nginx.yaml

# 代理地址
kubectl delete -f https://ghproxy.ketches.cn/https://raw.githubusercontent.com/ketches/registry-proxy/$VERSION/examples/dockerhub-nginx.yaml
```





# 部署dashboard

[helm 部署 dashboard)](https://artifacthub.io/packages/helm/k8s-dashboard/kubernetes-dashboard)

- 部署服务

```powershell
# 推荐使用下面这种方式
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta5/aio/deploy/recommended.yaml
$ vi recommended.yaml
# 修改Service为NodePort类型
......
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort  # 加上type=NodePort变成NodePort类型的服务
......
```

- 查看访问地址，本例为30133端口

```powershell
kubectl create -f recommended.yaml
kubectl -n kubernetes-dashboard get svc
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.105.62.124   <none>        8000/TCP        31m
kubernetes-dashboard        NodePort    10.103.74.46    <none>        443:30133/TCP   31m 
```

- 使用浏览器访问 https://62.234.133.177:30133，其中62.234.133.177为master节点的外网ip地址，chrome目前由于安全限制，测试访问不了，使用firefox可以进行访问。

- 创建ServiceAccount进行访问

```powershell
$ vi admin.conf
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kubernetes-dashboard

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kubernetes-dashboard

$ kubectl create -f admin.conf
$ kubectl -n kubernetes-dashboard get secret |grep admin-token
admin-token-fqdpf                  kubernetes.io/service-account-token   3      7m17s
# 使用该命令拿到token，然后粘贴到
$ kubectl -n kubernetes-dashboard get secret admin-token-fqdpf -o jsonpath={.data.token}|base64 -d
eyJhbGciOiJSUzI1NiIsImtpZCI6Ik1rb2xHWHMwbWFPMjJaRzhleGRqaExnVi1BLVNRc2txaEhETmVpRzlDeDQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1mcWRwZiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjYyNWMxNjJlLTQ1ZG...

```

![image-20240509100406655](./assets/image-20240509100406655.png)

![image-20240509100419606](./assets/image-20240509100419606.png)





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

# 二进制安装高可用K8s集群

本章采用二进制的方式安装Kubernetes集群，该方式较Kubeadm安装方式更为复杂，比如证书部分Kubeadm方式为自动生成，而二进制方式为手动创建，但是二进制方式可以将证书直接改为100年。

另外，二进制方式在升级时也比较简单，在配置没有更新的前提下，可以直接替换二进制包进行更新。

## 1. 基本环境配置

读者需自行准备5台具有2核CPU和4GB以上内存的服务器，系统为CentOS 7.x。

如果没有这么多的机器，可以只安装3台Master或者一台Master、两台Node。集群采用的服务器IP规划如表2.1所示。

表2.1　高可用Kubernetes集群规划

| 主机名           | IP地址                | 说明            |
| :--------------- | :-------------------- | :-------------- |
| k8s-master-01~03 | 10.0.0.201~203        | master节点*3    |
| VIP              | 10.0.0.236            | keeplived虚拟IP |
| k8s-node01~02    | 10.0.0.204~10.0.0.205 | worker节点*2    |

kubernetes一共涉及三个网段，一个是宿主机的网段，也就是上述的10.0.0.x，同时需要Pod和Service的网段，三者的网段不可交叉。

安装配置信息如表2.2所示。

表2.2　安装配置信息

| 配置信息    | 备注           |
| :---------- | :------------- |
| 系统版本    | Centos7.9      |
| Docker版本  | 20.10.x        |
| Pod网段     | 172.16.0.0/12  |
| Service网段 | 192.168.0.0/16 |

注意

> 宿主机网段、K8s Service网段、Pod网段不能重复。

所有节点配置主机名（其他节点名称自行更改）：

```
$ hostnamectl set-hostname k8s-master01 && eval -- bash
$ hostnamectl set-hostname k8s-master02 && eval -- bash
$ hostnamectl set-hostname k8s-master03 && eval -- bash
$ hostnamectl set-hostname k8s-node01 && eval -- bash
$ hostnamectl set-hostname k8s-node02 && eval -- bash
```

所有节点配置hosts，修改/etc/hosts如下：

```
$ cat /etc/hosts
10.0.0.201 k8s-master01
10.0.0.202 k8s-master02
10.0.0.203 k8s-master03
10.0.0.204 k8s-node01
10.0.0.205 k8s-node02
```

所有节点配置Docker和默认yum源：

```
$ curl -o /etc/yum.repos.d/CentOS-Base.repo https;//mirrors.aliyun.com/repo/Centos-7.repo
$ yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

所有节点安装一些常用的工具：

```
$ yum install wget jq psmisc vim net-tools telnet git -y
```

所有节点关闭防火墙、SELinux、DNSmasq：

```
$ systemctl disable --now firewalld
$ systemctl disable --now dnsmasq
$ systemctl disable --now NetworkManager

$ setenforce 0
$ sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
$ sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
```

所有节点关闭Swap分区：

```
$ swapoff -a && sysctl -w vm.swappiness=0
$ sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

所有节点安装ntpdate（如果公司的服务器已经配置了自动同步时间，关于时间的配置可以不用操作）：

```
$ rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
$ yum install ntpdate -y
```

所有节点同步时间：

```
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ echo 'Asia/Shanghai' >/etc/timezone

$ ntpdate time2.aliyun.com

## 加入到crontab
$ */5 * * * * /usr/sbin/ntpdate time2.aliyun.com
```

所有节点配置limit：

```
$ ulimit -SHn 65535

$ vim /etc/security/limits.conf
## 末尾添加如下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

安装过程中，生成配置文件和证书均在Master01上操作，所以Master01节点需要免密钥登录其他节点，之后将文件传送到其他节点。

集群管理也在Master01上操作（也可以是其他单独的节点）。

配置密钥（只在Master01或管理节点操作，以下步骤将管理节点和Master01统称为Master01）：

```
$ ssh-keygen -t rsa
$ for i in k8s-master01 k8s-master02 k8s-master03 k8s-node01 k8s-node02;do ssh-copy-id -i .ssh/id_rsa.pub $i;done
```

Master01下载安装所有的源码文件：

```
$ cd /root/ ; git clone https://github.com/dotbalo/k8s-ha-install.git
```

所有节点升级系统并重启：

```
$ yum update -y  && reboot
```

## 2. 内核配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#2-内核配置)

为了集群的稳定性和兼容性，生产环境的内核最好升级到4.18版本以上，本示例将升级到4.19版本。

Master01下载离线包：

```
$ cd /root
$ wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm
$ wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm
```

将安装包从Master01节点传到其他节点：

```
$ for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02;do scp kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm $i:/root/ ; done
```

所有节点安装内核：

```
$ cd /root && yum localinstall -y kernel-ml*

## 所有节点更改内核启动顺序
$ grub2-set-default  0 && grub2-mkconfig -o /etc/grub2.cfg

$ grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"

# 检查默认内核是不是4.19
$ grubby --default-kernel
/boot/vmlinuz-4.19.12-1.el7.elrepo.x86_64

# 所有节点重启，然后检查内核是不是4.19
$ uname -a
Linux k8s-master02 4.19.12-1.el7.elrepo.x86_64 #1 SMP Fri Dec 21 11:06:36 EST 2018 x86_64 x86_64 x86_64 GNU/Linux
```

所有节点安装ipvsadm和ipset：

```
$ yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

所有节点配置ipvs模块，在内核4.19+版本nf_conntrack_ipv4已经改为nf_conntrack，4.18以下版本使用nf_conntrack_ipv4即可：

```
$ vim /etc/modules-load.d/ipvs.conf
# 加入以下内容
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack # 4.18改为nf_conntrack_ipv4
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

然后执行`systemctl enable --now systemd-modules-load.service`即可。

开启一些K8s集群中必需的内核参数，所有节点配置K8s内核：

```
$ cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF

$ sysctl --system
```

所有节点配置完内核后，重启服务器，保证重启后内核依旧加载：

```
$ reboot
$ lsmod | grep --color=auto -e ip_vs -e nf_conntrack
```

## 3. 基本组件安装[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#3-基本组件安装)

前两节的安装步骤和Kubeadm方式并无区别，接下来对于Kubernetes、Etcd等组件的安装将采用二进制文件的方式进行安装。

本节主要安装的是集群中用到的各种组件，比如Docker-ce、Containerd、Kubernetes组件等。同样，读者可以自行选择Containerd或Docker作为Kubernetes的Runtime。

### 3.1 Containerd作为Runtime[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#31-containerd作为runtime)

所有节点安装docker-ce-20.10：

```
$ yum install docker-ce-20.10.* docker-ce-cli-20.10.* -y
```

由于并不是每个节点都需要Docker引擎，因此无须启动Docker，只需要配置和启动Containerd即可。

首先配置Containerd所需的模块（所有节点）：

```
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

所有节点加载模块：

```
$ modprobe -- overlay
$ modprobe -- br_netfilter
```

所有节点配置Containerd所需的内核：

```
$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

所有节点加载内核：

```
$ sysctl --system
```

所有节点配置Containerd的配置文件：

```
$ mkdir -p /etc/containerd
$ containerd config default | tee /etc/containerd/config.toml
```

所有节点将Containerd的Cgroup改为Systemd：

```
$ vim /etc/containerd/config.toml
# 找到containerd.runtimes.runc.options，添加SystemdCgroup = true
# 所有节点将sandbox_image的Pause镜像改成符合自己版本的地址registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6
```

所有节点启动Containerd，并配置开机自启动：

```
$ systemctl daemon-reload
$ systemctl enable --now containerd
```

所有节点配置crictl客户端连接的Runtime位置：

```
$ cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

### 3.2 Docker作为Runtime[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#32-docker作为runtime)

所有节点安装docker-ce 20.10：

```
$ yum install docker-ce-20.10.* docker-ce-cli-20.10.* -y
```

由于新版Kubelet建议使用systemd，因此把Docker的CgroupDriver也改成systemd：

```
$ mkdir -p /etc/docker  # 如果没有这个目录先创建，然后添加 daemon.json 文件

$ vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors" : [
    "https://ot2k4d59.mirror.aliyuncs.com/"
  ]
}
```

所有节点设置开机自启动Docker：

```
$ systemctl daemon-reload && systemctl enable --now docker
```

### 3.3 K8s及Etcd的安装[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#33-k8s及etcd的安装)

Master01下载Kubernetes安装包（1.22.0需要更改为最新或读者想要安装的指定版本）：

```
$ wget https://dl.k8s.io/v1.22.0/kubernetes-server-linux-amd64.tar.gz
```

Master01下载Etcd安装包：

```
$ wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-amd64.tar.gz
```

解压Kubernetes安装文件至bin目录：

```
$ tar -xf kubernetes-server-linux-amd64.tar.gz  --strip-components=3 -C /usr/local/bin kubernetes/server/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy}
```

解压Etcd安装文件至bin目录：

```
$ tar -zxvf etcd-v3.5.0-linux-amd64.tar.gz --strip-components=1 -C /usr/local/bin etcd-v3.5.0-linux-amd64/etcd{,ctl}
```

解压后即为安装成功，可以查看当前的版本：

```
$ kubelet --version
Kubernetes v1.22.0

$ etcdctl version
etcdctl version: 3.5.0
API version: 3.5
```

将组件发送到其他节点：

```
$ MasterNodes='k8s-master02 k8s-master03'
$ WorkNodes='k8s-node01 k8s-node02'
$ for NODE in $MasterNodes; do echo $NODE; scp /usr/local/bin/kube{let,ctl,-apiserver,-controller-manager,-scheduler,-proxy} $NODE:/usr/local/bin/; scp /usr/local/bin/etcd* $NODE:/usr/local/bin/; done

$ for NODE in $WorkNodes; do     scp /usr/local/bin/kube{let,-proxy} $NODE:/usr/local/bin/ ; done
```

所有节点创建/opt/cni/bin目录：

```
$ mkdir -p /opt/cni/bin
```

Master01节点切换到1.22.x分支（其他版本可以切换到其他分支，不需要更改为具体的小版本）：

```
$ cd k8s-ha-install && git checkout manual-installation-v1.22.x
```

### 3.4 生成证书[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#34-生成证书)

和Kubeadm安装方式不同的是，二进制安装方式需要自己生成所有的证书，包括Etcd、Kubernetes组件证书。

由于证书是和IP地址和域名进行绑定的，因此在生成证书时，需要注意IP地址等相关配置的修改。本示例采用CFSSL工具生成证书，读者也可以使用OpenSSL等工具生成。

由于生成证书只需要在Master01进行，因此只需要在Master01节点生成下载CFSSL工具即可。Master01下载生成证书工具：

```
$ wget "https://pkg.cfssl.org/R1.2/cfssl_linux-amd64" -O /usr/local/bin/cfssl
$ wget "https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
$ chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

#### 3.4.1 Etcd证书[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#341-etcd证书)

首先生成Etcd的证书，由于Etcd是3个节点组成的集群，因此证书需要包含每个节点的IP地址。

所有Master节点创建Etcd证书目录：

```
$ mkdir /etc/etcd/ssl -p
```

所有节点创建Kubernetes相关目录，Kubernetes所用的配置文件会放置在/etc/kubernetes下，证书文件会放置在/etc/kubernetes/pki下：

```
$ mkdir -p /etc/kubernetes/pki
```

接下来在Master01节点生成Etcd证书，生成证书的CSR（证书签名请求文件，配置了一些域名、公司、单位）文件已放置在前文clone的k8s-ha-install：

```
# 这个目录有我们生成证书需要用到的csr文件
$ cd /root/k8s-ha-install/pki

#查看
[root@k8s-master01 pki]# ls
admin-csr.json      ca-config.json  etcd-ca-csr.json  front-proxy-ca-csr.json      kubelet-csr.json     manager-csr.json
apiserver-csr.json  ca-csr.json     etcd-csr.json     front-proxy-client-csr.json  kube-proxy-csr.json  scheduler-csr.json

# 生成etcd CA证书和CA证书的key
$ cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare /etc/etcd/ssl/etcd-ca

2021/07/23 13:58:44 [INFO] generating a new CA key and certificate from CSR
2021/07/23 13:58:44 [INFO] generate received request
2021/07/23 13:58:44 [INFO] received CSR
2021/07/23 13:58:44 [INFO] generating key: rsa-2048
2021/07/23 13:58:44 [INFO] encoded CSR
2021/07/23 13:58:44 [INFO] signed certificate with serial number 65355458767171380149641516060181865353335743374

# 生成tcd的证书，需要注意修改-hostname的IP为自己的IP 
# 颁发证书
$ cfssl gencert \
   -ca=/etc/etcd/ssl/etcd-ca.pem \
   -ca-key=/etc/etcd/ssl/etcd-ca-key.pem \
   -config=ca-config.json \
   -hostname=127.0.0.1,k8s-master01,k8s-master02,k8s-master03,10.0.0.201,10.0.0.202,10.0.0.203 \
   -profile=kubernetes \
   etcd-csr.json | cfssljson -bare /etc/etcd/ssl/etcd
```

将证书复制到其他Master节点：

```
$ MasterNodes='k8s-master02 k8s-master03'
$ WorkNodes='k8s-node01 k8s-node02'

$ for NODE in $MasterNodes; do
     ssh $NODE "mkdir -p /etc/etcd/ssl"
     for FILE in etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pem; do
       scp /etc/etcd/ssl/${FILE} $NODE:/etc/etcd/ssl/${FILE}
     done
 done
```

#### 3.4.2 K8s组件证书[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#342-k8s组件证书)

相较于Etcd的证书，Kubernetes的证书更为复杂，因为Kubernetes要求组件之间双向加密通信，所以生成Kubernetes的组件会涉及每个组件之间通信的证书，同时也会包含一些聚合证书。

首先在Master01节点生成Kubernetes的根证书，之后其他证书都是根据该证书来生成的：

```
$ cd /root/k8s-ha-install/pki
$ cfssl gencert -initca ca-csr.json | cfssljson -bare /etc/kubernetes/pki/ca
```

接下来生成kube-apiserver的证书，需要注意更改如下配置：

- 192.168.0.是Kubernetes Service的网段，如果需要更改Kubernetes Service网段，那就需要把192.168.0.1更改为Service网段的第一个IP。
- 10.0.0.236为高可用集群的VIP地址，需要改成读者自己的VIP地址或者其他负载均衡的地址，如果是单Master的集群，直接改为Master节点IP即可。
- 10.0.0.201-203为Kubernetes Master节点的IP地址，需要更改为读者的节点IP。

```
$ cfssl gencert   -ca=/etc/kubernetes/pki/ca.pem   \
    -ca-key=/etc/kubernetes/pki/ca-key.pem   \
    -config=ca-config.json   \
    -hostname=192.168.0.1,10.0.0.236,127.0.0.1,kubernetes,kubernetes.default,kubernetes.default.svc, \
    kubernetes.default.svc.cluster,kubernetes.default.svc.cluster.local,10.0.0.201,10.0.0.202,10.0.0.203   \
    -profile=kubernetes   apiserver-csr.json | cfssljson -bare /etc/kubernetes/pki/apiserver
```

生成聚合证书，用于扩展的CRD请求验证，比如metrics-server的metrics.k8s.io和apiserver进行通信时就会使用到该证书：

```
$ cfssl gencert   -initca front-proxy-ca-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-ca 

$ cfssl gencert   -ca=/etc/kubernetes/pki/front-proxy-ca.pem   -ca-key=/etc/kubernetes/pki/front-proxy-ca-key.pem   -config=ca-config.json   -profile=kubernetes   front-proxy-client-csr.json | cfssljson -bare /etc/kubernetes/pki/front-proxy-client
```

同样的方式生成controller-manage的证书，使用的还是最初的CA文件：

```
$ cfssl gencert \
   -ca=/etc/kubernetes/pki/ca.pem \
   -ca-key=/etc/kubernetes/pki/ca-key.pem \
   -config=ca-config.json \
   -profile=kubernetes \
   manager-csr.json | cfssljson -bare /etc/kubernetes/pki/controller-manager

# 注意，如果不是高可用集群，10.0.0.236:16443改为master01的地址，6443改为apiserver的端口，默认是6443
$ set-cluster：设置一个集群项
$ kubectl config set-cluster kubernetes \
     --certificate-authority=/etc/kubernetes/pki/ca.pem \
     --embed-certs=true \
     --server=https://10.0.0.236:16443\
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# set-credentials 设置一个用户项
$ kubectl config set-credentials system:kube-controller-manager \
     --client-certificate=/etc/kubernetes/pki/controller-manager.pem \
     --client-key=/etc/kubernetes/pki/controller-manager-key.pem \
     --embed-certs=true \
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 设置一个环境项，一个上下文
$ kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes \
    --user=system:kube-controller-manager \
    --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig

# 使用某个环境当做默认环境
$ kubectl config use-context system:kube-controller-manager@kubernetes \
     --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig
```

同样的逻辑生成scheduler的证书，参数不再解释：

```
$ cfssl gencert \
   -ca=/etc/kubernetes/pki/ca.pem \
   -ca-key=/etc/kubernetes/pki/ca-key.pem \
   -config=ca-config.json \
   -profile=kubernetes \
   scheduler-csr.json | cfssljson -bare /etc/kubernetes/pki/scheduler

# 注意，如果不是高可用集群，10.0.0.236:16443改为master01的地址，6443改为apiserver的端口，默认是6443
$ kubectl config set-cluster kubernetes \
     --certificate-authority=/etc/kubernetes/pki/ca.pem \
     --embed-certs=true \
     --server=https://10.0.0.236:16443 \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

$ kubectl config set-credentials system:kube-scheduler \
     --client-certificate=/etc/kubernetes/pki/scheduler.pem \
     --client-key=/etc/kubernetes/pki/scheduler-key.pem \
     --embed-certs=true \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

$ kubectl config set-context system:kube-scheduler@kubernetes \
     --cluster=kubernetes \
     --user=system:kube-scheduler \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

$ kubectl config use-context system:kube-scheduler@kubernetes \
     --kubeconfig=/etc/kubernetes/scheduler.kubeconfig
```

至此，Kubernetes Master节点的组件证书已经配置完成。接下来需要生成一个Kubectl连接API Server的kubeconfig。

使用Kubeadm安装的Kubernetes集群，Kubectl连接API Server用到的kubeconfig文件为/etc/kubernetes/admin.conf，由Kubeadm自动生成。

使用二进制安装方式需要手动生成，生成步骤与上述方式类似。首先生成管理员用户的证书文件：

```
$ cfssl gencert \
   -ca=/etc/kubernetes/pki/ca.pem \
   -ca-key=/etc/kubernetes/pki/ca-key.pem \
   -config=ca-config.json \
   -profile=kubernetes \
   admin-csr.json | cfssljson -bare /etc/kubernetes/pki/admin
```

接下来生成kubeconfig文件。注意，如果不是高可用集群，把Master的地址改为10.0.0.236:16443，把API Server的端口改为16443，默认是6443：

```
$ kubectl config set-cluster kubernetes     \ 
    --certificate-authority=/etc/kubernetes/pki/ca.pem     \ 
    --embed-certs=true     \ 
    --server=https://10.0.0.236:16443    \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig

$ kubectl config set-credentials kubernetes-admin     \
    --client-certificate=/etc/kubernetes/pki/admin.pem     \
    --client-key=/etc/kubernetes/pki/admin-key.pem     \
    --embed-certs=true     \
    --kubeconfig=/etc/kubernetes/admin.kubeconfig

$ kubectl config set-context kubernetes-admin@kubernetes     \
      --cluster=kubernetes     \
      --user=kubernetes-admin     \
      --kubeconfig=/etc/kubernetes/admin.kubeconfig

$ kubectl config use-context kubernetes-admin@kubernetes     \
        --kubeconfig=/etc/kubernetes/admin.kubeconfig
```

生成用于创建ServiceAccount Secret Token和验证Token的密钥对：

```
$ openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
$ openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

所有的证书签发完成后，需要将证书发送至其他节点：

```
$ for NODE in k8s-master02 k8s-master03; do
  for FILE in $(ls /etc/kubernetes/pki | grep -v etcd); do
    scp /etc/kubernetes/pki/${FILE} $NODE:/etc/kubernetes/pki/${FILE};
    done;
    for FILE in admin.kubeconfig controller-manager.kubeconfig scheduler.kubeconfig; do
    scp /etc/kubernetes/${FILE} $NODE:/etc/kubernetes/${FILE};
  done;
done
```

### 3.5 高可用配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#35-高可用配置)

和Kubeadm安装方式一致，二进制安装方式仍然采用的是KeepAlived和HAProxy实现的高可用，所以需要安装KeepAlived和HAProxy。

KeepAlived和HAProxy的节点可以和Master在同一个节点，也可以在不同的节点。

如果读者是在公有云上搭建高可用集群，可以采用公有云的负载均衡替代KeepAlived和HAProxy。

提示

> 如果读者想要搭建只有一个Master节点的集群，可以不安装高可用组件。

安装步骤请参考本书第1章。

### 3.6 Etcd集群配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#36-etcd集群配置)

Etcd是Kubernetes的数据库，生产环境最少采用3个以上节点实现高可用，并且节点数必须为奇数个。

Etcd的配置大致相同，主要是每个Master节点的Etcd配置的主机名和IP地址。

#### 3.6.1 创建Etcd的配置文件[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#361-创建etcd的配置文件)

这里将Etcd和Kubernetes Master节点部署在同一台机器，如果读者需要将Etcd和Master节点拆分成不同的服务器，只需要更改对应的IP地址即可。

3个节点的Etcd配置大致相同，主要涉及如下参数需要修改（完整文件见本书下载资源）：

**vim /etc/etcd/etcd.config.yml**

**自行更改相关配置**

```
name: 'k8s-master01'
data-dir: /var/lib/etcd
wal-dir: /var/lib/etcd/wal
snapshot-count: 5000
heartbeat-interval: 100
election-timeout: 1000
quota-backend-bytes: 0
listen-peer-urls: 'https://10.103.236.201:2380'
listen-client-urls: 'https://10.103.236.201:2379,http://127.0.0.1:2379'
max-snapshots: 3
max-wals: 5
cors:
initial-advertise-peer-urls: 'https://10.103.236.201:2380'
advertise-client-urls: 'https://10.103.236.201:2379'
discovery:
discovery-fallback: 'proxy'
discovery-proxy:
discovery-srv:
initial-cluster: 'k8s-master01=https://10.103.236.201:2380,k8s-master02=https://10.103.236.202:2380,k8s-master03=https://10.103.236.203:2380'
initial-cluster-token: 'etcd-k8s-cluster'
initial-cluster-state: 'new'
strict-reconfig-check: false
enable-v2: true
enable-pprof: true
proxy: 'off'
proxy-failure-wait: 5000
proxy-refresh-interval: 30000
proxy-dial-timeout: 1000
proxy-write-timeout: 5000
proxy-read-timeout: 0
client-transport-security:
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
  client-cert-auth: true
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  auto-tls: true
peer-transport-security:
  cert-file: '/etc/kubernetes/pki/etcd/etcd.pem'
  key-file: '/etc/kubernetes/pki/etcd/etcd-key.pem'
  peer-client-cert-auth: true
  trusted-ca-file: '/etc/kubernetes/pki/etcd/etcd-ca.pem'
  auto-tls: true
debug: false
log-package-levels:
log-outputs: [default]
force-new-cluster: false
```

#### 3.6.2 创建Service[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#362-创建service)

接下来在每个Master节点创建Etcd Service并启动：

```
$ vim /usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Service
Documentation=https://coreos.com/etcd/docs/latest/
After=network.target

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.config.yml
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
Alias=etcd3.service
```

所有Master节点创建Etcd的证书目录：

```
$ mkdir /etc/kubernetes/pki/etcd
$ ln -s /etc/etcd/ssl/* /etc/kubernetes/pki/etcd/
$ systemctl daemon-reload
$ systemctl enable --now etcd
```

查看Etcd集群状态：

```
$ export ETCDCTL_API=3
$ etcdctl --endpoints="10.0.0.201:2379,10.0.0.202:2379,10.0.0.203:2379" --cacert=/etc/kubernetes/pki/etcd/etcd-ca.pem --cert=/etc/kubernetes/pki/etcd/etcd.pem --key=/etc/kubernetes/pki/etcd/etcd-key.pem  endpoint status --write-out=table

+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|      ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.0.0.203:2379 | d47768538fbe6e4f |  3.4.13 |   20 kB |      true |      false |         3 |          9 |                  9 |        |
| 10.0.0.202:2379 | 84a44ce3d2a87a83 |  3.4.13 |   20 kB |     false |      false |         3 |          9 |                  9 |        |
| 10.0.0.201:2379 | 5a8e242cc814d239 |  3.4.13 |   20 kB |     false |      false |         3 |          9 |                  9 |        |
+--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

### 3.7 K8s Master节点配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#37-k8s-master节点配置)

接下来在Master节点安装APIServer、Controller Manager、Scheduler组件。

首先在所有节点创建相关目录：

```
$ mkdir -p /etc/kubernetes/manifests/ /etc/systemd/system/kubelet.service.d /var/lib/kubelet /var/log/kubernetes
```

#### 3.7.1 APIServer[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#371-apiserver)

首先配置APIServer，每个节点的配置大致相同，注意修改的地方如下（完整文件见本书下载资源）：

**vim /usr/lib/systemd/system/kube-apiserver.service**

**配置自行更改**

```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
      --v=2  \
      --logtostderr=true  \
      --allow-privileged=true  \
      --bind-address=0.0.0.0  \
      --secure-port=6443  \
      --insecure-port=0  \
      --advertise-address=10.103.236.201 \
      --service-cluster-ip-range=192.168.0.0/16  \
      --service-node-port-range=30000-32767  \
      --etcd-servers=https://10.103.236.201:2379,https://10.103.236.202:2379,https://10.103.236.203:2379 \
      --etcd-cafile=/etc/etcd/ssl/etcd-ca.pem  \
      --etcd-certfile=/etc/etcd/ssl/etcd.pem  \
      --etcd-keyfile=/etc/etcd/ssl/etcd-key.pem  \
      --client-ca-file=/etc/kubernetes/pki/ca.pem  \
      --tls-cert-file=/etc/kubernetes/pki/apiserver.pem  \
      --tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem  \
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver.pem  \
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-key.pem  \
      --service-account-key-file=/etc/kubernetes/pki/sa.pub  \
      --service-account-signing-key-file=/etc/kubernetes/pki/sa.key  \
      --service-account-issuer=https://kubernetes.default.svc.cluster.local \
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname  \
      --enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota  \
      --authorization-mode=Node,RBAC  \
      --enable-bootstrap-token-auth=true  \
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem  \
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.pem  \
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client-key.pem  \
      --requestheader-allowed-names=aggregator  \
      --requestheader-group-headers=X-Remote-Group  \
      --requestheader-extra-headers-prefix=X-Remote-Extra-  \
      --requestheader-username-headers=X-Remote-User
      # --token-auth-file=/etc/kubernetes/token.csv

Restart=on-failure
RestartSec=10s
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Service文件创建完成后，所有Master节点启动kube-apiserver：

```
$ systemctl daemon-reload && systemctl enable --now kube-apiserver
```

检测APIServer状态：

```
$ systemctl status kube-apiserver

● kube-apiserver.service - Kubernetes API Server
   Loaded: loaded (/usr/lib/systemd/system/kube-apiserver.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-07-23 16:23:47 CST; 10s ago
     Docs: https://github.com/kubernetes/kubernetes
 Main PID: 10956 (kube-apiserver)
    Tasks: 13
   Memory: 302.4M
   CGroup: /system.slice/kube-apiserver.service
           └─10956 /usr/local/bin/kube-apiserver --v=2 --logtostderr=true --allow-privileged=true --bind-address=0.0.0.0 --secure-port=64...

Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.176533   10956 storage_rbac.go:331] created rolebinding.rbac.aut...system
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.185158   10956 storage_rbac.go:331] created rolebinding.rbac.aut...system
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.190144   10956 healthz.go:257] poststarthook/rbac/bootstrap-role...readyz
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: [-]poststarthook/rbac/bootstrap-roles failed: not finished
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.192942   10956 storage_rbac.go:331] created rolebinding.rbac.aut...system
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.201305   10956 storage_rbac.go:331] created rolebinding.rbac.aut...system
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.209273   10956 storage_rbac.go:331] created rolebinding.rbac.aut...public
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: W0723 16:23:54.322312   10956 lease.go:233] Resetting endpoints for master serv...2.212]
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.323449   10956 controller.go:611] quota admission added evaluato...points
Jul 23 16:23:54 k8s-master01 kube-apiserver[10956]: I0723 16:23:54.334503   10956 controller.go:611] quota admission added evaluato...k8s.io
Hint: Some lines were ellipsized, use -l to show in full.
```

#### 3.7.2 Controller Manager[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#372-controller-manager)

采用同样的流程配置Controller Manager，不同的是Controller Manager的Service都是一样的。

所有Master节点配置kube-controller-manager Service（完整文件见本书下载资源):

**vim /usr/lib/systemd/system/kube-controller-manager.service**

**配置自行更改**

```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
      --v=2 \
      --logtostderr=true \
      --address=127.0.0.1 \
      --root-ca-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \
      --cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \
      --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
      --kubeconfig=/etc/kubernetes/controller-manager.kubeconfig \
      --leader-elect=true \
      --use-service-account-credentials=true \
      --node-monitor-grace-period=40s \
      --node-monitor-period=5s \
      --pod-eviction-timeout=2m0s \
      --controllers=*,bootstrapsigner,tokencleaner \
      --allocate-node-cidrs=true \
      --cluster-cidr=172.16.0.0/12 \
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.pem \
      --node-cidr-mask-size=24

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

所有Master节点启动kube-controller-manager

```
$ systemctl daemon-reload
$ systemctl enable --now kube-controller-manager
```

查看启动状态

```
$ systemctl status kube-controller-manager
```

#### 3.7.3 Scheduler[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#373-scheduler)

和Controller Manager一致，每个Master节点的Service都是一样的。

所有Master节点配置kube-scheduler service（完整文件见本书下载资源）：

**vim /usr/lib/systemd/system/kube-scheduler.service**

```
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
      --v=2 \
      --logtostderr=true \
      --address=127.0.0.1 \
      --leader-elect=true \
      --kubeconfig=/etc/kubernetes/scheduler.kubeconfig

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

启动Scheduler：

```
$ systemctl daemon-reload

$ systemctl enable --now kube-scheduler
```

查看启动状态

```
$ systemctl  status kube-scheduler
```

至此，Kubernetes Master节点组件和Etcd均已经配置完成，读者可以查看系统日志（CentOS为/var/log/messages，Ubuntu为/var/log/syslog）有无报错日志。

### 3.8 TLS Bootstrapping配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#38-tls-bootstrapping配置)

相对于Master节点，工作节点的动态性相对比较灵活。

在维护集群时，经常会增删工作节点，如果像控制节点的组件一样去管理工作节点组件Kubelet的证书，会显得比较麻烦，并且容易出错，所以本节将在集群中配置Bootstrapping，Bootstrapping会自动为工作节点的Kubelet颁发证书，即生成kubelet.kubeconfig文件。

注意

接下来的命令，如果不是高可用集群，把Master的地址改为10.0.0.236:16443，把API Server的端口改为16443，默认是6443

kubectl命令只在Master01节点执行一次。接下来在Master01节点创建bootstrap所用的权限（配置Bootstrap的kubeconfig和2.4节类似）：

```
$ cd /root/k8s-ha-install/bootstrap

$ kubectl config set-cluster kubernetes     \
  --certificate-authority=/etc/kubernetes/pki/ca.pem     \
  --embed-certs=true     \
  --server=https://10.0.0.236:16443     \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

$ kubectl config set-credentials tls-bootstrap-token-user     \
  --token=c8ad9c.2e4d610cf3e7426e \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

$ kubectl config set-context tls-bootstrap-token-user@kubernetes     \
    --cluster=kubernetes     \
    --user=tls-bootstrap-token-user     \
    --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig

$ kubectl config use-context tls-bootstrap-token-user@kubernetes     \
  --kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig
```

bootstrap-kubelet.kubeconfig 是一个 keepalived 用来向 apiserver 申请证书的文件

注意：如果要修改bootstrap.secret.yaml的token-id和token-secret，需要保证 c8ad9c 字符串一致的，并且位数是一样的。

还要保证上个命令的黄色字体：c8ad9c.2e4d610cf3e7426e与你修改的字符串要一致

```
$ cat bootstrap.secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-c8ad9c
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  description: "The default bootstrap token generated by 'kubelet '."
  token-id: c8ad9c
  token-secret: 2e4d610cf3e7426e
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups:  system:bootstrappers:default-node-token,system:bootstrappers:worker,system:bootstrappers:ingress
```

前文提到，admin.kubeconfig是Kubectl操作集群所用的证书，将该文件复制到/root/.kube/config，Kubectl即可默认使用该文件连接集群：

```
$ mkdir -p /root/.kube ; cp /etc/kubernetes/admin.kubeconfig /root/.kube/config
```

接下来创建Bootstrapping所用的权限：

```
$ kubectl create -f bootstrap.secret.yaml
```

### 3.9 Node节点配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#39-node节点配置)

前面已经把所有Master节点组件安装完成，并且配置了Bootstrapping自动为Node节点签发证书，所以Node节点的Kubelet证书（kubelet.kubeconfig）不需要再手动配置，可以采用Bootstrap自动生成。

但是有一些证书还是需要手动发送到Node节点，比如请求kubelet.kubeconfig证书的bootstrap.kubeconfig证书等。Master01节点将Node节点所用的证书复制到各个Node节点（本次示例Master节点也会部署kubelet和kube-proxy）：

```
$ cd /etc/kubernetes/

$ for NODE in k8s-master02 k8s-master03 k8s-node01 k8s-node02; do
     ssh $NODE mkdir -p /etc/kubernetes/pki /etc/etcd/ssl /etc/etcd/ssl
     for FILE in etcd-ca.pem etcd.pem etcd-key.pem; do
       scp /etc/etcd/ssl/$FILE $NODE:/etc/etcd/ssl/
     done
     for FILE in pki/ca.pem pki/ca-key.pem pki/front-proxy-ca.pem bootstrap-kubelet.kubeconfig; do
       scp /etc/kubernetes/$FILE $NODE:/etc/kubernetes/${FILE}
 done
 done
```

#### 3.9.1 Kubelet配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#391-kubelet配置)

所有节点创建相关目录：

```
$ mkdir -p /var/lib/kubelet /var/log/kubernetes /etc/systemd/system/kubelet.service.d /etc/kubernetes/manifests/
```

所有节点配置Kubelet Service：

```
$ vim  /usr/lib/systemd/system/kubelet.service

# 添加以下内容
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/local/bin/kubelet

Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

所有节点配置kubelet service的配置文件

```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock --cgroup-driver=systemd"
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "
ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS
```

如果读者的Runtime为Docker，请使用如下Kubelet的配置：

**vim /etc/systemd/system/kubelet.service.d/10-kubelet.conf**

```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.kubeconfig --kubeconfig=/etc/kubernetes/kubelet.kubeconfig"
Environment="KUBELET_SYSTEM_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_CONFIG_ARGS=--config=/etc/kubernetes/kubelet-conf.yml --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"
Environment="KUBELET_EXTRA_ARGS=--node-labels=node.kubernetes.io/node='' "
ExecStart=
ExecStart=/usr/local/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_SYSTEM_ARGS $KUBELET_EXTRA_ARGS
```

所有节点创建Kubelet的配置文件（注意修改的地方，完整文件见本书下载资源）：

**vim /etc/kubernetes/kubelet-conf.yml**

```
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: systemd
cgroupsPerQOS: true
clusterDNS:
- 192.168.0.10
clusterDomain: cluster.local
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
```

启动所有节点Kubelet：

```
$ systemctl daemon-reload
$ systemctl enable --now kubelet
```

此时系统日志/var/log/messages显示只有如下信息为正常，因为还没有安装CNI插件：

```
Unable to update cni config: no networks found in /etc/cni/net.d
```

注意

如果有很多报错日志，不只是CNI的报错，或者有大量看不懂的报错，说明Kubelet的配置有误，需要检查Kubelet的配置。

查看集群状态：

集群状态NotReady，因为Calico还没安装

```
$ kubectl get node
NAME           STATUS     ROLES    AGE     VERSION
k8s-master01   NotReady   <none>   2m23s   v1.22.0-beta.1
k8s-master02   NotReady   <none>   2m16s   v1.22.0-beta.1
k8s-master03   NotReady   <none>   2m16s   v1.22.0-beta.1
k8s-node01     NotReady   <none>   2m16s   v1.22.0-beta.1
k8s-node02     NotReady   <none>   2m16s   v1.22.0-beta.1
```

#### 3.9.2 kube-proxy配置[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#392-kube-proxy配置)

注意

如果不是高可用集群，把Master的地址改为10.0.0.236:16443，把API Server的端口改为16443，默认是6443

以同样的方式生成kube-proxy的证书。以下操作只在Master01执行：

```
$ cd /root/k8s-ha-install

$ kubectl -n kube-system create serviceaccount kube-proxy

$ kubectl create clusterrolebinding system:kube-proxy   \
    --clusterrole system:node-proxier         \
    --serviceaccount kube-system:kube-proxy


$ SECRET=$(kubectl -n kube-system get sa/kube-proxy \
    --output=jsonpath='{.secrets[0].name}')

$ JWT_TOKEN=$(kubectl -n kube-system get secret/$SECRET \
--output=jsonpath='{.data.token}' | base64 -d)

$ PKI_DIR=/etc/kubernetes/pki
$ K8S_DIR=/etc/kubernetes


$ kubectl config set-cluster kubernetes     \
    --certificate-authority=/etc/kubernetes/pki/ca.pem    \
    --embed-certs=true     \
    --server=https://10.0.0.236:16443    \
    --kubeconfig=${K8S_DIR}/kube-proxy.kubeconfig

$ kubectl config set-credentials kubernetes     \
    --token=${JWT_TOKEN}     \
    --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig


$ kubectl config set-context kubernetes     \
  --cluster=kubernetes     \
  --user=kubernetes     \
  --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig

$ kubectl config use-context kubernetes     --kubeconfig=/etc/kubernetes/kube-proxy.kubeconfig
```

注意

修改--server=[https://10.0.0.236:16443为读者自己的VIP地址和端口](https://10.0.0.236:16443为读者自己的VIP地址和端口/)，如果不是高可用集群，改为Master节点IP:6443即可。

将kubeconfig发送至其他节点：

```
$ for NODE in k8s-master02 k8s-master03; do
     scp ${K8S_DIR}/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
     scp kube-proxy/kube-proxy.conf $NODE:/etc/kubernetes/kube-proxy.conf
     scp kube-proxy/kube-proxy.service $NODE:/usr/lib/systemd/system/kube-proxy.service
 done

$ for NODE in k8s-node01 k8s-node02; do
     scp /etc/kubernetes/kube-proxy.kubeconfig $NODE:/etc/kubernetes/kube-proxy.kubeconfig
     scp kube-proxy/kube-proxy.conf $NODE:/etc/kubernetes/kube-proxy.conf
     scp kube-proxy/kube-proxy.service $NODE:/usr/lib/systemd/system/kube-proxy.service
 done
```

所有节点添加kube-proxy的配置和Service文件：

**vim /usr/lib/systemd/system/kube-proxy.service**

```
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
After=network.target

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/etc/kubernetes/kube-proxy.yaml \
  --v=2

Restart=always
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

所有节点创建kube-proxy的配置文件，如果更改了集群Pod的网段，就需要更改kube-proxy.yaml的clusterCIDR为自己的Pod网段（完整文件见本书下载资源）：

**vim /etc/kubernetes/kube-proxy.yaml**

```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  acceptContentTypes: ""
  burst: 10
  contentType: application/vnd.kubernetes.protobuf
  kubeconfig: /etc/kubernetes/kube-proxy.kubeconfig
  qps: 5
clusterCIDR: 172.16.0.0/12 
configSyncPeriod: 15m0s
conntrack:
  max: null
  maxPerCore: 32768
  min: 131072
  tcpCloseWaitTimeout: 1h0m0s
  tcpEstablishedTimeout: 24h0m0s
enableProfiling: false
healthzBindAddress: 0.0.0.0:10256
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: 14
  minSyncPeriod: 0s
  syncPeriod: 30s
ipvs:
  masqueradeAll: true
  minSyncPeriod: 5s
  scheduler: "rr"
  syncPeriod: 30s
kind: KubeProxyConfiguration
metricsBindAddress: 127.0.0.1:10249
mode: "ipvs"
nodePortAddresses: null
oomScoreAdj: -999
portRange: ""
udpIdleTimeout: 250ms
```

所有节点启动kube-proxy：

```
$ systemctl daemon-reload
$ systemctl enable --now kube-proxy
```

### 3.10 安装Calico[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#310-安装calico)

启动Kubelet时，会有一个CNI的报错，原因是没有安装CNI插件。目前常用的CNI插件有Calico、Flannel等，由于本书包含NetworkPolicy的知识，需要CNI插件支持网络策略，因此和Kubeadm安装方式一样，同样选择Calico作为CNI插件。

进入Calico安装文件所在的目录。以下步骤只在master01执行：

```
$ cd /root/k8s-ha-install/calico/
```

更改Calico的网段，注意需要将172.16.0.0/12网段改为自己的Pod网段：

```
$ sed -i "s#POD_CIDR#172.16.0.0/12#g" calico.yaml
```

安装Calico：

```
$ kubectl apply -f calico.yaml
```

查看容器状态：

```
$ kubectl  get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS      AGE
calico-kube-controllers-cdd5755b9-4fzg9   1/1     Running   0             113s
calico-node-8xg62                         1/1     Running   0             113s
calico-node-dczxz                         1/1     Running   0             113s
calico-node-gn8ws                         1/1     Running   0             113s
calico-node-qmwkd                         1/1     Running   0             113s
calico-node-zfw8n                         1/1     Running   2 (78s ago)   113s
```

如果容器状态异常可以使用kubectl describe 或者logs查看容器的日志

### 3.11 安装CoreDNS[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#311-安装coredns)

在Master01节点进入CoreDNS安装文件所在目录：

```
$ cd /root/k8s-ha-install/CoreDNS
```

如果更改了Service的网段，就需要将CoreDNS的Service IP改成Kubernetes Service网段的第10个IP：

```
$ COREDNS_SERVICE_IP=`kubectl get svc | grep kubernetes | awk '{print $3}'`0
$ sed -i "s#192.168.0.10#${COREDNS_SERVICE_IP}#g" coredns.yaml
```

安装CoreDNS：

```
$ kubectl  create -f coredns.yaml
serviceaccount/coredns created
...
```

### 3.12 安装Metrics Server[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#312-安装metrics-server)

在新版的Kubernetes中，系统资源的采集和HPA功能均使用Metrics-server，可以通过Metrics采集节点和Pod的内存、磁盘、CPU和网络的使用率。

在Master01安装Metrics Server：

```
$ cd /root/k8s-ha-install/metrics-server

$ kubectl  create -f .
serviceaccount/metrics-server created
...
```

等待Metrics Server启动，然后查看状态：

```
$ kubectl  top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master01   263m         13%    1239Mi          66%       
k8s-master02   213m         10%    1065Mi          57%       
k8s-master03   207m         10%    1050Mi          56%       
k8s-node01     89m          4%     514Mi           27%       
k8s-node02     158m         7%     493Mi           26% 
```

查看pod状态

```
$ kubectl  top pod -A
```

### 3.13 安装dashboard[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#313-安装dashboard)

- 安装指定版本dashboard
- 安装最新版dashboard
- 登录dashboard

Dashboard用于展示集群中的各类资源，同时也可以通过Dashboard实时查看Pod的日志和在容器中执行一些命令等。

（1）安装指定版本dashboard

```
$ cd /root/k8s-ha-install/dashboard/

[root@k8s-master01 dashboard]# ls
dashboard-user.yaml  dashboard.yaml

# kubectl create -f .
serviceaccount/admin-user created
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard
created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard
created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard
created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

（2）安装最新版dashboard

官方GitHub地址：https://github.com/kubernetes/dashboard

可以在官方dashboard查看到最新版dashboard

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
```

创建管理员用户（安装最新版本的时候）

```
$ vim admin.yaml
# 添加以下内容
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

执行

```
$ kubectl apply -f admin.yaml -n kube-system
```

（3）登录dashboard 在谷歌浏览器（Chrome）启动文件中加入启动参数，用于解决无法访问Dashboard的问题，因为使用的证书是自签名（属性->快捷方式->目标，粘贴到最后）

```
 --test-type --ignore-certificate-errors
```

更改dashboard的svc为NodePort：

```
$ kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

修改 type: ClusterIP 为 type:NodePort

修改完成之后会暴露一个端口号，查看端口号：

```
[root@k8s-master01 dashboard]# kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.98.129.195   <none>        443:30711/TCP   85s

[root@k8s-master01 dashboard]# kubectl get pod -A -owide | grep dashboard
kubernetes-dashboard   dashboard-metrics-scraper-7b4bbf8954-zvgm8   1/1     Running   0          13m     172.27.14.193    k8s-node02     <none>           <none>
kubernetes-dashboard   kubernetes-dashboard-6c65b776bd-797lx        1/1     Running   0          13m     172.18.195.1     k8s-master03   <none>           <none>
```

根据自己的实例端口号，通过任意安装了kube-proxy的宿主机或者VIP的IP+端口即可访问到dashboard：访问Dashboard：[https://10.218.22.252:30711](https://10.218.22.252:30711/)（请更改为自己的端口），选择登录方式为令牌（即token方式）

查看token值：

```
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
token值

Name:         admin-user-token-9c4tz
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: d1f2e528-0ef8-4c6b-a384-a18fbca6bc54

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1411 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlNCbEdFa1RQZElhbTBRb29aTTNCTUE1dTJ2enBCeGZxMWJwbmpfZHBXdkEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTljNHR6Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkMWYyZTUyOC0wZWY4LTRjNmItYTM4NC1hMThmYmNhNmJjNTQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.KFH5ed0kJEaU1HSpxkitJxqKJGnSNAWogNSGjGn1wEh7R9zKYkAfNLES6Vl3GU9jvxBCEZW415ZFILr96kpgl_88mD-K-AMgQxKLdpghYDx_CnsLtI6e8rLTNkaPS2Uo3sYAy9U280Niop14Yzuar5FQ3AfSbeXGcF_9Jrgyeh5XWPA0h69Au8pUEOkVdpADmuIaFSqfTnmkOSdGqCgFb_QsUqvjo4ifIxKnN6uW8wfR1s4esWkPq569xhCINaUY6g3rnT1jfVTU2XmrURrKOVok0OfSmtXTKCSs2jliEdmx7qEFTrw2KCPnTfORUtTnmdZ2ZnGGx9Fvf_hGaKk1FQ
```

### 3.14 使用Kuboard界面[¶](https://hujianli94.github.io/Kubernetes/1.安装篇/2.二进制安装高可用K8s集群/#314-使用kuboard界面)

Kuboard是国人开发的另一个dashboard界面,

官网地址

https://www.kuboard.cn/install/v3/install-in-k8s.html

详细内容可以看官网，我这里直接使用命令安装

```
$ kubectl apply -f https://addons.kuboard.cn/kuboard/kuboard-v3.yaml
```

在浏览器中打开链接 [http://your-node-ip-address:30080](http://your-node-ip-address:30080/)

输入初始用户名和密码，并登录

```
用户名： admin
密码： Kuboard123
```

## 参考文献

[二进制安装k8s集群V1.23.0](https://blog.csdn.net/weixin_44280843/article/details/126780243)

[二进制高可用安装k8s集群部署](https://blog.csdn.net/qq_41475058/article/details/119025309)

[kubernetes之二进制安装1.25.+版本](https://zhangzhuo.ltd/articles/2022/01/09/1641717241819.html)