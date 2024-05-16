# [Rook](https://rook.io/docs/rook/latest-release/Getting-Started/intro/)

Rook 是一个开源**的云原生存储编排器**。为 Ceph 存储提供平台、框架和支持，以便与云原生环境原生集成。

[Ceph](https://ceph.com/) 是一个分布式存储系统，提供文件、块和对象存储，并部署在大规模生产集群中。

Rook 并不是存储系统，在存储和k8s之前搭建了一个桥梁，使存储系统的搭建或者维护变得特别简单，Rook 自动部署和管理 Ceph，以提供自我管理、自我扩展和自我修复的存储服务。Rook Operator 通过构建 Kubernetes 资源来部署、配置、配置、扩展、升级和监控 Ceph。并且Rook支持CSI，可以利用CSI做一些PVC的快照、扩容、克隆等操作。

# 9.云原生存储 Rook

云环境 kubernetes 集群要使用后端存储有很多选择，比如 oss ， nas ， 云盘 等。但是有时候我们可能会出于其他各种原因需要自建存储服务器来为 kubernetes 提供存储卷，一般我们都会选择 ceph 存储，但如果使用物理机或者 ecs 去部署，需要花费大量人力不说，维护起来也相当割裂，幸好有 rook ceph 这种方案，很好地与云原生环境集成，可以让 ceph 直接跑在 kubernetes 集群上，这样一来便可大大方便维护还是管理。

# 9.2 前置条件

1. 生产环境需要至少 3 台以上节点，用于作为 osd 节点存储数据使用。
2. 每个 osd 节点，至少拥有一块裸盘，用于部署 rook ceph 时初始化用。
3. 我们使用的 rook ceph 版本较高，需要 kubernetes 版本 1.22 或更高。

| 名称            | 信息                                                         |
| :-------------- | :----------------------------------------------------------- |
| 节点配置        | 16c64g 200G SSD *1 500G SSD *1(一个系统盘一个 ceph 元数据) 节点数 3 |
| Kubernetes 集群 | v1.26.0                                                      |
| helm            | v3.8.0                                                       |
| Rook            | v1.13.1                                                      |
| Ceph            | v18.2.1                                                      |
| Mon 组件        | 3 个                                                         |
| Mgr 组件        | 2 个                                                         |

# 9.3 部署

## K8S 集群准备

Rook 1.13 支持 Kubernetes v1.23 或更高版本。本文使用的 Kubernetes 集群如下:

```sh
$ kubectl get node
NAME                       STATUS   ROLES           AGE   VERSION
gitee-kubernetes-master1   Ready    control-plane   20d   v1.26.3
gitee-kubernetes-master2   Ready    control-plane   20d   v1.26.3
gitee-kubernetes-master3   Ready    control-plane   20d   v1.26.3
gitee-kubernetes-node01    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node02    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node03    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node04    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node05    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node06    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node07    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node08    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node09    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node10    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node11    Ready    <none>          20d   v1.26.3
gitee-kubernetes-node12    Ready    <none>          20d   v1.26.3
```

关于容器运行时的配置，这里的 Kubernetes 集群的容器运行时是 Containerd。

注意如果 Containerd 的 systemd 配置 containerd.service 中如果有`LimitNOFILE=infinity`的配置，后边在使用 rook 启动 Ceph 集群时，Ceph 的 Mon 组件会有问题，会出现`ms_dispatch`进程的 cpu 一直是 100%，Rook 社区有两个 ISSUES [ISSUE 11253](https://github.com/rook/rook/issues/11253)和[ISSUE 10110](https://github.com/rook/rook/issues/10110)讨论了这个问题，需要将`LimitNOFILE`设置一个合适的值，我这里设置的是`1048576`。

How to fix?

```sh
$ cat >/etc/systemd/system/containerd.service.d/LimitNOFILE.conf<<EOF
[Service]
LimitNOFILE=1048576
EOF
$ systemctl daemon-reload
$ systemctl restart containerd
```

## 本地存储准备(LVM 逻辑卷)

要配置 Ceph 存储集群，至少需要以下其中一种本地存储：

- 原始设备(Raw devices)（无分区或格式化文件系统）
- 原始分区(Raw partitions)（无格式化文件系统）
- LVM 逻辑卷（无格式化文件系统）
- 以块模式(block mode)在存储类(storage class)中提供的持久卷。

我们将使用的本地存储是`LVM逻辑卷`。

gitee-kubernetes-node09~gitee-kubernetes-node11 三台主机上都有一个未分区的本地磁盘将用作 Ceph 集群的本地存储。

```
# gitee-kubernetes-node09
root@gitee-kubernetes-node09:/home/ubuntu# fdisk -l | grep /dev/vdb
Disk /dev/vdb: 500 GiB, 536870912000 bytes, 1048576000 sectors

# gitee-kubernetes-node10
root@gitee-kubernetes-node10:/home/ubuntu# fdisk -l | grep /dev/vdb
Disk /dev/vdb: 500 GiB, 536870912000 bytes, 1048576000 sectors

# # gitee-kubernetes-node11
root@gitee-kubernetes-node11:/home/ubuntu# fdisk -l | grep /dev/vdb
Disk /dev/vdb: 500 GiB, 536870912000 bytes, 1048576000 sectors
```

> 注意：在执行以下步骤之前，请确保磁盘上没有重要数据，因为这些步骤将擦除磁盘上的所有数据。

```sh
# 首先创建物理卷
pvcreate /dev/vdb
# 接下来创建名称为ceph的卷组
vgcreate ceph /dev/vdb
# 最后从卷组ceph创建逻辑卷， 使用lvcreate命令来创建逻辑卷。
# 使用卷组ceph的所有可用空间创建了名称为osd的逻辑卷
lvcreate -n osd -l 100%FREE ceph

# 如果想指定逻辑卷的具体大小可以使用下面的命令:
# lvcreate -n osd -L 400G ceph

# 删除lvm
#yes | lvremove /dev/ceph/osd
#vgremove ceph
#pvremove /dev/vdb
```

因为 Ceph 集群所需要的本地存储是未格式化为文件系统的 LVM 逻辑卷，所以注意一定不要使用`mkfs.ext4`或`mkfs.xfs`格式化逻辑卷。

最后查看一下逻辑卷:

```
$ lvdisplay
  --- Logical volume ---
  LV Path                /dev/ceph/osd
  LV Name                osd
  VG Name                ceph
  LV UUID                cVsDhP-hUtV-7Eoe-XID1-Uvdt-TZ3z-7IEAXG
  LV Write Access        read/write
  LV Creation host, time gitee-kubernetes-node09, 2024-01-02 10:37:19 +0000
  LV Status              available
  # open                 0
  LV Size                <500.00 GiB
  Current LE             127999
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

## 配置时间同步[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#配置时间同步)

注意设置各个服务器节点的时间同步，这个是否重要，否则如果各个服务器节点时间不同步时，rook ceph operator 在操作时，ceph mon 组件的可能会无法正常工作。

关于时间同步推荐使用 chronyd。

## 安装 helm[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#安装-helm)

参考 5.2 安装 helm

## 部署 Rook Operator[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#部署-rook-operator)

我们将 Ceph 的 mon,osd,mrg 等调度到 gitee-kubernetes-node09, gitee-kubernetes-node10, gitee-kubernetes-node11 上。

下面给这 3 个节点打上`role=ceph`的 label，并加上`GiteeFrontendOnly=yes:NoSchedule`的 taint。

```
# kubectl label node gitee-kubernetes-node09 role=ceph
# 注意：ceph和前端服务器部署在一起。
kubectl label node gitee-kubernetes-node09 role=ceph
kubectl label node gitee-kubernetes-node10 role=ceph
kubectl label node gitee-kubernetes-node11 role=ceph

# kubectl taint nodes node4 dedicated=ceph:NoSchedule
kubectl taint nodes gitee-kubernetes-node09 GiteeFrontendOnly=yes:NoSchedule
kubectl taint nodes gitee-kubernetes-node10 GiteeFrontendOnly=yes:NoSchedule
kubectl taint nodes gitee-kubernetes-node11 GiteeFrontendOnly=yes:NoSchedule
```

我们将通过使用 Rook Helm Chart 来部署 rook ceph operator。

Rook 目前将 Ceph Operator 的构建版本发布到发布（release）和主要（master）通道。发布通道是 Rook 的最新稳定版。

```
$ helm repo add rook-release https://charts.rook.io/release
$ helm repo update
#$ helm install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph -f values.yaml
```

或者从https://charts.rook.io/release/rook-ceph-v1.13.1.tgz下载rook-ceph的helm chart，再使用下面的命令安装:

```
$ helm pull rook-release/rook-ceph --version v1.13.1 --untar
# $ helm pull rook-release/rook-ceph --version v1.13.1
$ helm install --create-namespace --namespace rook-ceph rook-ceph rook-ceph-v1.13.1.tgz -f values.yaml
```

关于 values.yaml 中配置的内容，可以根据文档https://github.com/rook/rook/blob/master/deploy/charts/rook-ceph-cluster/values.yaml中的内容按需定制。

以下是当前我所定制的内容，主要配置了在部署 Rook Operator 和 Ceph 时使用私有的镜像仓库地址，以及调度相关的配置。

```
values-prod.yaml
image:
  # -- Image
  repository: rook/ceph
  # -- Image tag
  # @default -- `v1.13.1`
  tag: v1.13.1
  # -- Image pull policy
  pullPolicy: IfNotPresent

tolerations:
  - key: "GiteeFrontendOnly"
    operator: "Equal"
    value: "yes"
    effect: "NoSchedule"

provisionerNodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: role
            operator: In
            values:
              - ceph
provisionerTolerations:
  - key: "GiteeFrontendOnly"
    operator: "Equal"
    value: "yes"
    effect: "NoSchedule"
pluginNodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: role
            operator: In
            values:
              - ceph
pluginTolerations:
  - key: "GiteeFrontendOnly"
    operator: "Equal"
    value: "yes"
    effect: "NoSchedule"

discover:
  tolerations:
    - key: "GiteeFrontendOnly"
      operator: "Equal"
      value: "yes"
      effect: "NoSchedule"
  nodeAffinity:
    nodeSelectorTerms:
      - matchExpressions:
          - key: role
            operator: In
            values:
              - ceph

admissionController:
  tolerations:
    - key: "GiteeFrontendOnly"
      operator: "Equal"
      value: "yes"
      effect: "NoSchedule"
    - key: "node-role.kubernetes.io/master"
      operator: "Exists"
      effect: "PreferNoSchedule"
  nodeAffinity:
    nodeSelectorTerms:
      - matchExpressions:
          - key: role
            operator: In
            values:
              - ceph
```

部署

```
$ kubectl create ns rook-ceph
$ cd rook-ceph
$ helm install -n rook-ceph rook-ceph -f values-prod.yaml .
```

部署完成后，确认 rook-ceph-operator 的正常启动:

```
$ kubectl get pods -n rook-ceph -l "app=rook-ceph-operator"
NAME                                  READY   STATUS    RESTARTS   AGE
rook-ceph-operator-5bb7fbf69c-hn785   1/1     Running   0          64s
```

## 创建 Ceph 集群

# 参考文档

Rook 文档侧重于在各种环境中启动 Rook。在创建 Ceph 集群时，可以考虑在以下的示例集群清单基础上做定制：

- [cluster.yaml](https://github.com/rook/rook/blob/release-1.12/deploy/examples/cluster.yaml)：用于在裸机上运行的生产集群的集群设置。需要至少三个工作节点。
- [cluster-on-pvc.yaml](https://github.com/rook/rook/blob/release-1.12/deploy/examples/cluster-on-pvc.yaml)：用于在动态云环境中运行的生产集群的集群设置。
- [cluster-test.yaml](https://github.com/rook/rook/blob/release-1.12/deploy/examples/cluster-test.yaml)：用于测试环境（如 minikube）的集群设置。

有关更多详细信息，可参考[Ceph 的示例配置](https://rook.io/docs/rook/v1.12/Getting-Started/example-configurations/)。

Rook Ceph Operator 已经部署完成并正常运行，我们可以创建 Ceph 集群了，这里选择的是`cluster.yaml`。

# 创建集群

```sh
$ git clone --single-branch --branch v1.13.1 https://github.com/rook/rook.git
```

基于[cluster-prod.yaml](https://github.com/rook/rook/blob/release-1.12/deploy/examples/cluster.yaml)定制我们自己的 cluster.yaml，以下是只包含 cluster.yaml 的修改内容:

```
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: quay.io/ceph/ceph:v18.2.1
  mon:
    count: 3
    allowMultiplePerNode: false
  mgr:
    count: 2
    allowMultiplePerNode: false
    modules:
      - name: pg_autoscaler
        enabled: true
  dashboard:
    enabled: true
    ssl: false
  storage:
    useAllNodes: true
    useAllDevices: true
    devices:
      - name: /dev/ceph/osd
    config:
      osdsPerDevice: "1"
    onlyApplyOSDPlacement: false

  placement:
    all:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: role
                  operator: In
                  values:
                    - ceph
      podAffinity:
      podAntiAffinity:
      topologySpreadConstraints:
      tolerations:
        - key: "GiteeFrontendOnly"
          operator: "Equal"
          value: "yes"
          effect: "NoSchedule"
        - key: "node-role.kubernetes.io/master"
          operator: "Exists"
          effect: "PreferNoSchedule"
```

因为 Ceph 集群的本地存储将使用前面创建的未格式化文件系统的 LVM 逻辑卷，我们创建的逻辑卷的名称为`osd`，所属的卷组为`ceph`，逻辑卷的路径为`/dev/ceph/osd`。

Rook 是从 1.9 开始支持使用 LVM 逻辑卷作为本地存储的，具体实现的代码是这个 PR https://github.com/rook/rook/pull/7967。 文档[CephCluster CRD](https://rook.io/docs/rook/v1.13/CRDs/Cluster/ceph-cluster-crd/)中对如何使用逻辑卷作存储的说明不是特别详细。从 PR 的实现代码上看，当前是要在 CephCluster 的资源定义中的`spec.storage.devices[].name`配置为`/dev/disk/by-id/dm-name-<vgName>-<lvName>`。

所以我们的 cluster.yaml 配置的`spec.storage`部分， `devices[].name`的值是`/dev/disk/by-id/dm-name-ceph-osd`。

创建 Ceph 集群:

```
$ kubectl create -f cluster-prod.yaml
cephcluster.ceph.rook.io/rook-ceph created
```

通过查看`rook-ceph`命名空间中的 pod，验证集群是否正在运行。

```
$ kubectl get pod
NAME                                                              READY   STATUS      RESTARTS       AGE
csi-cephfsplugin-8l9t7                                            2/2     Running     1 (112s ago)   2m26s
csi-cephfsplugin-provisioner-76bfc9dcd9-42zjb                     5/5     Running     0              15m
csi-cephfsplugin-provisioner-76bfc9dcd9-jfpxt                     5/5     Running     0              15m
csi-cephfsplugin-vvjb9                                            2/2     Running     0              15m
csi-rbdplugin-provisioner-56f7bf6d4d-kr6vq                        5/5     Running     0              15m
csi-rbdplugin-provisioner-56f7bf6d4d-wjgcn                        5/5     Running     0              15m
csi-rbdplugin-r5qsw                                               2/2     Running     1 (112s ago)   2m26s
csi-rbdplugin-slgs2                                               2/2     Running     0              15m
rook-ceph-crashcollector-gitee-kubernetes-node09-748dd9c59wpldm   1/1     Running     0              19h
rook-ceph-crashcollector-gitee-kubernetes-node10-765b7b857xxbds   1/1     Running     0              19h
rook-ceph-crashcollector-gitee-kubernetes-node11-6fbb8b47bmrdzs   1/1     Running     0              19h
rook-ceph-exporter-gitee-kubernetes-node09-5d5db7d54f-4hhlj       1/1     Running     0              19h
rook-ceph-exporter-gitee-kubernetes-node10-75c4755979-jsqsd       1/1     Running     0              19h
rook-ceph-exporter-gitee-kubernetes-node11-5f6744dd9b-m7ktj       1/1     Running     0              19h
rook-ceph-mgr-a-587ff77854-t96j8                                  2/2     Running     0              19h
rook-ceph-mgr-b-578f895c7b-l2hqd                                  2/2     Running     0              19h
rook-ceph-mon-a-84589c6b44-6rlv7                                  1/1     Running     0              19h
rook-ceph-mon-b-664b5fc9cf-9wv82                                  1/1     Running     0              19h
rook-ceph-mon-c-648796588d-x9zkf                                  1/1     Running     0              19h
rook-ceph-operator-5bb7fbf69c-5r2cq                               1/1     Running     0              20h
rook-ceph-osd-0-ddff98578-v2vzr                                   1/1     Running     0              19h
rook-ceph-osd-1-85b7486f99-9x5k6                                  1/1     Running     0              19h
rook-ceph-osd-2-55dbd68f47-9lh95                                  1/1     Running     0              19h
rook-ceph-osd-prepare-gitee-kubernetes-node09-l29rj               0/1     Completed   0              95s
rook-ceph-osd-prepare-gitee-kubernetes-node10-js8tx               0/1     Completed   0              98s
rook-ceph-osd-prepare-gitee-kubernetes-node11-p92dn               0/1     Completed   0              91s
rook-ceph-tools-5996f89559-b4tch                                  1/1     Running     0              18h
```

osd pod 的数量取决于集群中的节点数量和配置的设备数量。对于上述默认的 cluster.yaml，将为每个节点上找到的每个可用设备创建一个 OSD。

创建过程如果遇到问题可以查看`rook-ceph-operator`的 Pod 的日志。

如需重新运行`rook-ceph-osd-prepare-<nodename>` Job，扫描可用本地存储添加 OSD，可以执行以下命令:

```
# 删除旧的job
$ kubectl get job -n rook-ceph | awk '{system("kubectl delete job "$1" -n rook-ceph")}'
# 重启operator
$ kubectl rollout restart deploy rook-ceph-operator  -n rook-ceph
```

每个节点上的 osd 能否添加成功，要注意查看`rook-ceph-osd-prepare-<nodename>` Job 对应 Pod 的日志。

## 验证集群状态[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#验证集群状态)

为了验证集群处于健康状态， 需要[Rook 工具箱](https://rook.io/docs/rook/v1.13/Troubleshooting/ceph-toolbox/)。

```
$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/release-1.13/deploy/examples/toolbox.yaml

$ kubectl get po -n rook-ceph | grep rook-ceph-tools
rook-ceph-tools-68b98695bb-gh76t                  1/1     Running     0             23s
```

> 注: 当前 release-1.13/deploy/examples/toolbox.yaml 中的 Ceph 镜像还是 v17.2.6，可以手动修改成 v18.2.1 还需加上污点容忍和亲和性

```
      nodeSelector:
        role: ceph
      tolerations:
        - key: "node.kubernetes.io/unreachable"
          operator: "Exists"
          effect: "NoExecute"
          tolerationSeconds: 5
        - key: "GiteeFrontendOnly"
          operator: "Equal"
          value: "yes"
          effect: "NoSchedule"
```

连接到工具箱，并运行`ceph status` 命令:

```
$ kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
# 或者如下
$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') -- bash
```

以下是健康状态的验证要点：

- 所有的 monitor (mon) 节点应该处于 quorum（一致性）状态。
- 一个管理器 (mgr) 节点应该处于活动状态。
- 至少有三个 OSD 节点应该处于上线并可用状态。

如果健康状态不是 `HEALTH_OK`，则应该调查警告或错误的原因。

```
bash-4.4$ ceph status
  cluster:
    id:     80acf00a-a7e1-40fe-b6c2-30f546e519bb
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 71m)
    mgr: a(active, since 68m), standbys: b
    osd: 3 osds: 3 up (since 70m), 3 in (since 70m)

  data:
    pools:   1 pools, 1 pgs
    objects: 2 objects, 449 KiB
    usage:   80 MiB used, 1.5 TiB / 1.5 TiB avail
    pgs:     1 active+clean

bash-4.4$ ceph osd status
ID  HOST                      USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE
 0  gitee-kubernetes-node10  26.7M   499G      0        0       0        0   exists,up
 1  gitee-kubernetes-node09  26.7M   499G      0        0       0        0   exists,up
 2  gitee-kubernetes-node11  26.7M   499G      0        0       0        0   exists,up

bash-4.4$ ceph df
--- RAW STORAGE ---
CLASS     SIZE    AVAIL    USED  RAW USED  %RAW USED
hdd    1.5 TiB  1.5 TiB  80 MiB    80 MiB          0
TOTAL  1.5 TiB  1.5 TiB  80 MiB    80 MiB          0

--- POOLS ---
POOL  ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
.mgr   1    1  449 KiB        2  1.3 MiB      0    475 GiB


bash-4.4$ ceph osd tree
ID  CLASS  WEIGHT   TYPE NAME                         STATUS  REWEIGHT  PRI-AFF
-1         1.46489  root default
-5         0.48830      host gitee-kubernetes-node09
 1    hdd  0.48830          osd.1                         up   1.00000  1.00000
-3         0.48830      host gitee-kubernetes-node10
 0    hdd  0.48830          osd.0                         up   1.00000  1.00000
-7         0.48830      host gitee-kubernetes-node11
 2    hdd  0.48830          osd.2                         up   1.00000  1.00000

bash-4.4$ rados df
POOL_NAME       USED  OBJECTS  CLONES  COPIES  MISSING_ON_PRIMARY  UNFOUND  DEGRADED  RD_OPS       RD  WR_OPS       WR  USED COMPR  UNDER COMPR
.mgr         1.3 MiB        2       0       6                   0        0         0     192  288 KiB     133  1.3 MiB         0 B          0 B
replicapool   12 KiB        1       0       3                   0        0         0       0      0 B       0      0 B         0 B          0 B

total_objects    3
total_used       81 MiB
total_avail      1.5 TiB
total_space      1.5 TiB
```

从输出可以看出集群状态一切正常，集群中部署了 3 个 mon, 2 个 mgr, 3 个 osd。

查看一下当前集群中的存储池:

```
bash-4.4$ ceph osd lspools
1 .mgr
```

可以看到当前集群中只有一个名称为`.mgr`的存储池。这表示在这个 Ceph 集群中只创建了默认的管理池（mgr pool），这是一个特殊的池，用于存储管理和监控相关的数据。

工具箱相关查询命令

```
ceph status
ceph osd status
ceph df
rados df
ceph osd lspools
```

## Ceph Dashboard[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#ceph-dashboard)

通过使用 Ceph Dashboard 可以查看集群的状态。使用 Rook 部署的 Ceph 集群已经默认启用了 Ceph Dashboard。

![img](./assets/ceph-v18-dashboard.png)

`rook-ceph-mgr-dashboard`是其在 Kubernetes 集群中的 Service:

```sh
$ kubectl get svc rook-ceph-mgr-dashboard -n rook-ceph
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
rook-ceph-mgr-dashboard   ClusterIP   10.100.133.87   <none>        7000/TCP   73m
```

可通过 Ingress 或创建一个 NodePort 的 Service [dashboard-external-http](https://github.com/rook/rook/blob/master/deploy/examples/dashboard-external-http.yaml)将其暴露的 Kubernetes 集群外部。

```sh
$ kubectl get svc  -n rook-ceph
NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
rook-ceph-exporter                      ClusterIP   10.100.135.90    <none>        9926/TCP            75m
rook-ceph-mgr                           ClusterIP   10.100.15.109    <none>        9283/TCP            75m
rook-ceph-mgr-dashboard                 ClusterIP   10.100.133.87    <none>        7000/TCP            75m
rook-ceph-mgr-dashboard-external-http   NodePort    10.100.213.184   <none>        7000:32396/TCP      33s
rook-ceph-mon-a                         ClusterIP   10.100.29.79     <none>        6789/TCP,3300/TCP   76m
rook-ceph-mon-b                         ClusterIP   10.100.73.21     <none>        6789/TCP,3300/TCP   75m
rook-ceph-mon-c                         ClusterIP   10.100.159.142   <none>        6789/TCP,3300/TCP   75m
```

Ceph Dashboard admin 用户的命名可以通过下面的命令查看:

```sh
$ kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```

# 使用存储-服务持久化

Ceph 提供三种类型的存储接口: 块存储（Block）、共享文件系统（Shared Filesystem）、对象存储（Object）。

下面演示对于使用 Rook 部署和管理的 Ceph 集群，如何使用这三种存储。

通过 Rook 使用 Ceph 提供的三种存储类型以及它们的用途如下:

- 块存储（Block）适用于为单个 Pod 提供读写一致性（RWO）的存储
- CephFS 共享文件系统（Shared Filesystem）适用于多个 Pod 之间共享读写（RWX）的存储
- 对象存储（Object）提供了一个可通过内部或外部的 Kubernetes 集群的 S3 端点访问的存储



## 9.4.1 块存储（Block）

块存储是一种数据存储技术，它以块为单位将数据写入磁盘或其他存储介质中。块存储通常被用于服务器、数据中心和云计算等环境中，以存储大规模数据集。

块存储系统通常由一个或多个块存储设备组成，这些设备可以是磁盘、固态硬盘 (SSD) 或其他存储介质。块存储设备通常具有高速读写能力和高可靠性。每个块存储设备都有一个控制器，负责管理块设备的读写操作，并将数据块复制到多个设备上以提高容错性和数据冗余性。

块存储的主要优点是高速读写能力、高可靠性和数据冗余性。块存储设备可以快速地读取和写入数据，并且可以自动处理数据的复制、备份和恢复等操作。此外，块存储设备还具有高度的容错性和可靠性，因为它们可以将数据复制到多个设备上，并在其中一个设备失败时自动切换到其他设备上。

块存储系统通常被用于存储大规模数据集，如数据库、文件系统、云计算平台等。它们也被用于存储关键数据，如服务器操作系统、应用程序和数据仓库等。

- 1：阿里云：EBS

- 2：腾讯云：CBS
- 3：Ceph：RBD
- ......

相信大家也都用过云盘，那么云盘可以做什么，当然我们的RBD也就可以做什么，比如快照备份，增量备份，内核驱动等，都是可以支持到的

**块存储允许单个 Pod 挂载存储。**

本指南介绍了如何使用 Rook 启用的持久卷，在 Kubernetes 上创建一个简单的多层 Web 应用程序。

### 9.4.1.1 RBD 存储供给

在 Rook 可以提供存储之前，需要创建 StorageClass 和 CephBlockPool CR。这将使 Kubernetes 在提供持久卷时与 Rook 进行交互操作。

> 这个示例需要每个节点至少有 1 个 OSD，并且每个 OSD 需要位于 3 个不同的节点上。每个 OSD 必须位于不同的节点上，因为 `failureDomain` 被设置为 `host`，并且 `replicated.size` 被设置为 3。

直接使用

`rook/deploy/examples/csi/rbd/storageclass.yaml`文件创建 CephBlockPool 存储池和 StorageClass 动态存储卷。

或者创建下面的`storageclass.yaml`文件:

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  # clusterID is the namespace where the rook cluster is running
  clusterID: rook-ceph
  # Ceph pool into which the RBD image shall be created
  pool: replicapool

  imageFormat: "2"
  imageFeatures: layering

  csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
  csi.storage.k8s.io/fstype: ext4

reclaimPolicy: Delete
allowVolumeExpansion: true
```

这个 storageclass.yaml 文件中包含了 StorageClass `rook-ceph-block`和 CephBlockPool `replicapool`的定义。

如果你在一个名为"rook-ceph"以外的命名空间中部署了 Rook Operator，请将该文件中的`provisioner`中的前缀更改为与你使用的命名空间匹配。例如，如果 Rook Operator 在命名空间"my-namespace"中运行，则 provisioner 的值应为`my-namespace.rbd.csi.ceph.com`。

接下来创建这个 StorageClass 和 CephBlockPool:

```sh
$ kubectl apply -f storageclass.yaml
cephblockpool.ceph.rook.io/replicapool created
storageclass.storage.k8s.io/rook-ceph-block created
```

> 根据 Kubernetes 的规定，在使用"Retain"回收策略时，任何由 PersistentVolume 支持的 Ceph RBD 镜像将在 PersistentVolume 被删除后继续存在。这些 Ceph RBD 镜像需要使用`rbd rm`命令手动清理。

上面在创建名称为`replicapool`的 CephBlockPool 资源时，会自动在 Ceph 集群中创建名称为`replicapool`的存储池。

这个操作是由 Ceph Operator 完成的，如果查看发现没有创建这个存储池，可以通过查看 Ceph Operator 的日志进行问题定位。

```sh
$ kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
bash-4.4$ ceph osd lspools
1 .mgr
2 replicapool
```

上面命令的输出说明已经创建了存储池`replicapool`。

将 Ceph 设置为默认存储卷

```sh
[root@k8s-master1 ~]# kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

修改完成后再查看 StorageClass 状态（**有个 default 标识**）

```sh
$ kubectl get sc
NAME                        PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block (default)   rook-ceph.rbd.csi.ceph.com   Delete          Immediate           true                   2m5s
```

### 9.4.1.2 使用存储

```yaml
rook/deploy/examples/csi/rbd/pvc.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rook-ceph-block
```

通过运行以下命令来查看 Kubernetes 的 PVC(卷声明)：

```sh
$ kubectl get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
rbd-pvc   Bound    pvc-f6fa10da-9e2e-4ca1-aec2-0a1b9b457f86   1Gi        RWO            rook-ceph-block   8m55s
```

查看一下创建的持久卷:

```sh
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS      REASON   AGE
pvc-f6fa10da-9e2e-4ca1-aec2-0a1b9b457f86   1Gi        RWO            Delete           Bound    rook-ceph/rbd-pvc   rook-ceph-block            11m
```

查看其中一个持久化卷的具体信息:

```
$ kubectl describe pv pvc-f6fa10da-9e2e-4ca1-aec2-0a1b9b457f86
Name:            pvc-f6fa10da-9e2e-4ca1-aec2-0a1b9b457f86
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: rook-ceph.rbd.csi.ceph.com
                 volume.kubernetes.io/provisioner-deletion-secret-name: rook-csi-rbd-provisioner
                 volume.kubernetes.io/provisioner-deletion-secret-namespace: rook-ceph
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    rook-ceph-block
Status:          Bound
Claim:           rook-ceph/rbd-pvc
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        1Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            rook-ceph.rbd.csi.ceph.com
    FSType:            ext4
    VolumeHandle:      0001-0009-rook-ceph-0000000000000003-487f0510-6c2c-4a39-8198-3c411598a777
    ReadOnly:          false
    VolumeAttributes:      clusterID=rook-ceph
                           imageFeatures=layering
                           imageFormat=2
                           imageName=csi-vol-487f0510-6c2c-4a39-8198-3c411598a777
                           journalPool=replicapool
                           pool=replicapool
                           storage.kubernetes.io/csiProvisionerIdentity=1704334037395-4473-rook-ceph.rbd.csi.ceph.com
Events:                <none>
```

查看一下存储池 replicapool 中的 rbd 镜像:

```
$ kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
bash-4.4$ rbd ls -p replicapool
csi-vol-487f0510-6c2c-4a39-8198-3c411598a777
```

我们创建了一个 Pod 示例应用程序，使用经由 Rook 提供的块存储

```
rook/deploy/examples/csi/rbd/pod.yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: csirbd-demo-pod
spec:
  containers:
    - name: web-server
      image: nginx
      volumeMounts:
        - name: mypvc
          mountPath: /var/lib/www/html
  volumes:
    - name: mypvc
      persistentVolumeClaim:
        claimName: rbd-pvc
        readOnly: false
$ kubectl apply -f pod.yaml
pod/csirbd-demo-pod created

$ kubectl get pod csirbd-demo-pod
NAME              READY   STATUS    RESTARTS   AGE
csirbd-demo-pod   1/1     Running   0          2m1s
```

或者直接创建一个短暂的 pod

```
$ kubectl apply -f pod-ephemeral.yaml
pod/csi-rbd-demo-ephemeral-pod created

$ cat pod-ephemeral.yaml
kind: Pod
apiVersion: v1
metadata:
  name: csi-rbd-demo-ephemeral-pod
spec:
  containers:
    - name: web-server
      image: docker.io/library/nginx:latest
      volumeMounts:
        - mountPath: "/myspace"
          name: mypvc
  volumes:
    - name: mypvc
      ephemeral:
        volumeClaimTemplate:
          spec:
            accessModes: ["ReadWriteOnce"]
            storageClassName: "rook-ceph-block"
            resources:
              requests:
                storage: 1Gi
$ kubectl get pod csi-rbd-demo-ephemeral-pod
NAME                         READY   STATUS    RESTARTS   AGE
csi-rbd-demo-ephemeral-pod   1/1     Running   0          14s
```

查看一下存储池 replicapool 中的 rbd 镜像:

```
$ rbd ls -p replicapool
csi-vol-487f0510-6c2c-4a39-8198-3c411598a777
csi-vol-75a8fa65-83b7-469e-baa2-5856fc96721e
```

### 9.4.1.3 PVC 快照-块存储快照[¶](https://hujianli94.github.io/Kubernetes/4.高级篇/12.云原生存储Rook/#9413-pvc-快照-块存储快照)

snapshotclass 的作用：

和 StorageClass 一样，提供一种管理 volume 供给的方式。VolumeSnapshotClass 也提供一种管理卷快照的方式。

Kubernetes 1.19 以上版本需要单独安装 Snapshot 控制器，才能完成 PVC 的快照功能，所以在此提前安装一下，如果是 1.19 以下的版本，则不需要单独安装。

安装方式如下：

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

参考：[I get an error when deploying an existing snapshotclass.yaml. · Issue #6819 · rook/rook (github.com)](https://github.com/rook/rook/issues/6819)

[Ceph Docs (rook.github.io)](https://rook.github.io/docs/rook/v1.5/ceph-csi-snapshot.html)

文件位置 `rook/deploy/examples/csi/rbd/snapshotclass.yaml`

创建：

```sh
$ kubectl create -f snapshotclass.yaml
volumesnapshotclass.snapshot.storage.k8s.io/csi-rbdplugin-snapclass created
```

内容：实际上就是通过 parameters 指定提供 snapshot 资源的信息。通过 snapshot.storage.k8s.io/v1 这个 api 接口进行管理。

```yaml
cat csi/rbd/snapshotclass.yaml
---
# 1.17 <= K8s <= v1.19
# apiVersion: snapshot.storage.k8s.io/v1beta1
# K8s >= v1.20
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-rbdplugin-snapclass
driver: rook-ceph.rbd.csi.ceph.com # driver:namespace:operator
parameters:
  # Specify a string that identifies your cluster. Ceph CSI supports any
  # unique string. When Ceph CSI is deployed by Rook use the Rook namespace,
  # for example "rook-ceph".
  clusterID: rook-ceph # namespace:cluster
  csi.storage.k8s.io/snapshotter-secret-name: rook-csi-rbd-provisioner
  csi.storage.k8s.io/snapshotter-secret-namespace: rook-ceph # namespace:cluster
deletionPolicy: Delete
```

#### 制作快照

创建快照之前，我们在给需要创建快照的卷里面新增一些内容，以便后面来进行数据验证。

在 web-0-pvc 的 pod 里面增加一个 nginx 的主页数据，后面用于测试。

创建测试容器和 pvc

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: web
  selector:
    app: nginx
  type: ClusterIP

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-0-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: rook-ceph-block

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: web
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumes:
        - name: www
          persistentVolumeClaim:
            claimName: web-0-pvc
```

```sh
$ k apply -f deployment-pvc-restore.yaml
service/nginx created
persistentvolumeclaim/web-0-pvc created

$  k get pvc web-0-pvc
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
web-0-pvc   Bound    pvc-1a93a247-e78c-4d6e-838f-929a520d1343   1Gi        RWO            rook-ceph-block   73s

$ k exec -it pod/web-54c59df478-znqpp -n rook-ceph -- bash
root@web-54c59df478-znqpp:/# cd /usr/share/nginx/html/
root@web-54c59df478-znqpp:/usr/share/nginx/html# echo "this is pvc snapshot rbd test!" > index.html
root@web-54c59df478-znqpp:/usr/share/nginx/html# ls -l
total 20
-rw-r--r-- 1 root root    31 Jan  4 06:50 index.html
drwx------ 2 root root 16384 Jan  4 06:35 lost+found
root@web-54c59df478-znqpp:/usr/share/nginx/html# exit
```

修改了`index.html`文件，接下来，我们进行快照制作

snapsho 定义了类型为 VolumeSnapshot，来源是已经存在的一个 PVC，同时注意：PVC 是有 namespace 区分的，所以必须在同一个 namespace 中，不然无法 Bound。

```yaml
snapshot.yaml
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: rbd-pvc-snapshot
spec:
  volumeSnapshotClassName: csi-rbdplugin-snapclass
  source:
    persistentVolumeClaimName: web-0-pvc
```

```sh
$ k get pvc -n rook-ceph
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
web-0-pvc          Bound    pvc-1a93a247-e78c-4d6e-838f-929a520d1343   1Gi        RWO            rook-ceph-block   118s

$ k apply -f snapshot.yaml
volumesnapshot.snapshot.storage.k8s.io/rbd-pvc-snapshot created

# 查看快照，READYTOUSE必须为true才算成功
$ kubectl get VolumeSnapshot -n rook-ceph
NAME               READYTOUSE   SOURCEPVC   SOURCESNAPSHOTCONTENT   RESTORESIZE   SNAPSHOTCLASS             SNAPSHOTCONTENT                                    CREATIONTIME   AGE
rbd-pvc-snapshot   true         web-0-pvc                           1Gi           csi-rbdplugin-snapclass   snapcontent-7cecab7b-297f-4899-8046-4813f08b58b6   3s             5s
```

#### 指定快照创建 PVC

```yaml
pvc-restore.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc-restore
spec:
  storageClassName: rook-ceph-block
  dataSource:
    name: rbd-pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

> 注意：指定快照创建 PVC 时，需要满足如下条件：
>
> 1、在同一个 namespace；
>
> 2、存储大小必须小于等于快照的大小。不然无法 Bound。

```sh
$ kubectl create -f pvc-restore.yaml -n rook-ceph
persistentvolumeclaim/rbd-pvc-restore created

$ kubectl get pvc
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
rbd-pvc-restore    Bound    pvc-84e3a64e-3740-406e-ae30-38f57312cbc9   1Gi        RWO            rook-ceph-block   14s
web-0-pvc          Bound    pvc-1a93a247-e78c-4d6e-838f-929a520d1343   1Gi        RWO            rook-ceph-block   9m41s
```

#### 数据验证

```yaml
restore-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-restore
  namespace: rook-ceph
  labels:
    app: wordpress
    tier: frontend
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - image: nginx
          name: nginx
          volumeMounts:
            - name: wordpress-persistent-storage-restore
              mountPath: /var/www/html
      volumes:
        - name: wordpress-persistent-storage-restore
          persistentVolumeClaim:
            claimName: rbd-pvc-restore
```

通过创建一个 nginx，把刚刚通过快照创建好的 pvc 进行挂载到 pod 里面。

```sh
$ kubectl create -f restore-nginx.yaml -n rook-ceph
$  k get pod
web-54c59df478-znqpp                                              1/1     Running     0               20m
wordpress-restore-69bb6c76b8-zmb87                                1/1     Running     0               21s
$ k exec pod/web-54c59df478-znqpp -- cat /usr/share/nginx/html/index.html
this is pvc snapshot rbd test!

$ k exec pod/wordpress-restore-69bb6c76b8-zmb87 -- cat /var/www/html/index.html
this is pvc snapshot rbd test!
```

从上面可以看出，数据是 OK 的。

web-0 的挂载点：/usr/share/nginx/html/

wordproess-restore 的挂载点：/var/www/html/

因为快照的数据也是直接挂载的 PVC，所以数据直接落到了 PVC 的挂载点。

### 9.4.1.4 PVC 动态扩容

扩容需求

- 文件共享类型的 PVC 扩容需要 k8s 1.15+
- 块存储类型的 PVC 扩容需要 k8s 1.16+

PVC 扩容需要开启`ExpandCSIVolumes`，新版本的 k8s 已经默认打开了这个功能，可以查看自己的 k8s 版本是否已经默认打开了该功能：

```sh
$ kube-apiserver -h|grep ExpandCSIVolumes
```

如果 default 为 true 就不需要打开此功能，如果 default 为 false，需要开启该功能。

- storageclass 也必须支持动态扩容 `allowVolumeExpansion: true`

```sh
$ kubectl get sc rook-cephfs -o yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
```

#### 扩容操作

扩容前，PVC 是 1GB 的容量：

```sh
$ kubectl get pvc -n rook-ceph
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
web-0-pvc          Bound    pvc-1a93a247-e78c-4d6e-838f-929a520d1343   1Gi        RWO            rook-ceph-block   28m

$ root@gitee-sre2:/home/ubuntu# k exec -it pod/web-54c59df478-znqpp -n rook-ceph -- bash
root@web-54c59df478-znqpp:/# df -Th
Filesystem     Type     Size  Used Avail Use% Mounted on
overlay        overlay  194G   15G  180G   8% /
tmpfs          tmpfs     64M     0   64M   0% /dev
tmpfs          tmpfs     32G     0   32G   0% /sys/fs/cgroup
/dev/vda1      ext4     194G   15G  180G   8% /etc/hosts
shm            tmpfs     64M     0   64M   0% /dev/shm
/dev/rbd0      ext4     974M   28K  958M   1% /usr/share/nginx/html
tmpfs          tmpfs     63G   12K   63G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs          tmpfs     32G     0   32G   0% /proc/acpi
tmpfs          tmpfs     32G     0   32G   0% /proc/scsi
tmpfs          tmpfs     32G     0   32G   0% /sys/firmware
```

扩容后：

```sh
$ kubectl edit pvc/web-0-pvc -n rook-ceph
persistentvolumeclaim/web-0-pvc edited

$ kubectl get pv,pvc -n rook-ceph
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS      REASON   AGE
persistentvolume/pvc-0375f153-0f61-4e0e-93fd-c0caafc9f22c   2Gi        RWX            Delete           Bound    rook-ceph/busybox-data-pvc   rook-cephfs                83m
persistentvolume/pvc-1a93a247-e78c-4d6e-838f-929a520d1343   2Gi        RWO            Delete           Bound    rook-ceph/web-0-pvc          rook-ceph-block            33m

NAME                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistentvolumeclaim/busybox-data-pvc   Bound    pvc-0375f153-0f61-4e0e-93fd-c0caafc9f22c   2Gi        RWX            rook-cephfs       83m
persistentvolumeclaim/web-0-pvc          Bound    pvc-1a93a247-e78c-4d6e-838f-929a520d1343   2Gi        RWO            rook-ceph-block   33m

$ k exec -it pod/web-54c59df478-znqpp -n rook-ceph -- bash
root@web-54c59df478-znqpp:/# df -Th
Filesystem     Type     Size  Used Avail Use% Mounted on
overlay        overlay  194G   15G  180G   8% /
tmpfs          tmpfs     64M     0   64M   0% /dev
tmpfs          tmpfs     32G     0   32G   0% /sys/fs/cgroup
/dev/vda1      ext4     194G   15G  180G   8% /etc/hosts
shm            tmpfs     64M     0   64M   0% /dev/shm
/dev/rbd0      ext4     2.0G   28K  2.0G   1% /usr/share/nginx/html
tmpfs          tmpfs     63G   12K   63G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs          tmpfs     32G     0   32G   0% /proc/acpi
tmpfs          tmpfs     32G     0   32G   0% /proc/scsi
tmpfs          tmpfs     32G     0   32G   0% /sys/firmware
root@web-54c59df478-znqpp:/#
```

### 9.4.1.5 PVC 克隆

PVC 克隆和 PVC 的 snapshot 类似：

创建测试环境

```sh
$ k apply -f deployment-pvc-restore.yaml
service/nginx created
persistentvolumeclaim/web-0-pvc created
deployment.apps/web created

$ k get pv,pvc
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS      REASON   AGE
persistentvolume/pvc-0375f153-0f61-4e0e-93fd-c0caafc9f22c   2Gi        RWX            Delete           Bound    rook-ceph/busybox-data-pvc   rook-cephfs                87m
persistentvolume/pvc-7568c98a-0225-4db0-9e88-d5ea27925440   1Gi        RWO            Delete           Bound    rook-ceph/web-0-pvc          rook-ceph-block            5s

NAME                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
persistentvolumeclaim/busybox-data-pvc   Bound    pvc-0375f153-0f61-4e0e-93fd-c0caafc9f22c   2Gi        RWX            rook-cephfs       87m
persistentvolumeclaim/web-0-pvc          Bound    pvc-7568c98a-0225-4db0-9e88-d5ea27925440   1Gi        RWO            rook-ceph-block   5s
pvc-clone.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: rbd-pvc-clone
spec:
  storageClassName: rook-ceph-block
  dataSource:
    name: web-0-pvc
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
$ k apply -f pvc-clone.yaml
persistentvolumeclaim/rbd-pvc-clone created

$ k get pvc
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
busybox-data-pvc   Bound    pvc-0375f153-0f61-4e0e-93fd-c0caafc9f22c   2Gi        RWX            rook-cephfs       89m
rbd-pvc-clone      Bound    pvc-d3466f64-639b-4720-b094-064121b9ed6b   1Gi        RWO            rook-ceph-block   2s
web-0-pvc          Bound    pvc-7568c98a-0225-4db0-9e88-d5ea27925440   1Gi        RWO            rook-ceph-block   115s
```

**注意事项：**

**1、克隆只能在相同 namespace 下进行；**

**2、克隆后的 pvc 大小必须小于等于克隆前的**

## 9.4.2 CephFS 共享文件系统

**CephFS 文件系统存储（也称为共享文件系统）可以从多个 Pod 中以读/写权限挂载。这对于可以使用共享文件系统进行集群化的应用程序可能非常有用。**

Ceph 从自 Pacific 版本(Ceph 16)开始，支持多个文件系统。

1：RBD的特点：
RBD块存储只能用于单个 VM 或者单个 pods 使用，无法提供给多个虚拟机或者多个 pods ”同时“使用，如果虚拟机或 pods 有共同访问存储的需求需要使用 CephFS 实现。

2：NAS 网络附加存储：多个客户端同时访问
    1：EFS
    2：NAS
    3：CFS

3：CephFS 特点：
    1：POSIX-compliant semantics （符合 POSIX 的语法）
    2：Separates metadata from data （metadata和data 分离，数据放入data，元数据放入metadata）
    3：Dynamic rebalancing （动态从分布，自愈）
    4：Subdirectory snapshots （子目录筷子）
    5：Configurable striping （可配置切片）
    6：Kernel driver support （内核级别挂载）
    7：FUSE support （用户空间级别挂载）
    8：NFS/CIFS deployable （NFS/CIFS方式共享出去提供使用）
    9：Use with Hadoop (replace HDFS) （支持Hadoop 的 HDFS）



### 9.4.2.1 创建 CEPHFILESYSTEM

通过在`CephFilesystem` CRD 中指定元数据池(metadata pool)、数据池(data pools)和元数据服务(metadata server)的所需设置来创建文件系统。

在这里，我们创建的是 3 个副本的元数据池和 3 个副本的单个数据池。有关更多选项，请参阅[创建共享文件系统的文档](https://rook.io/docs/rook/v1.11/CRDs/Shared-Filesystem/ceph-filesystem-crd/)。

创建下面的 filesystem.yaml 文件:

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3
  dataPools:
    - name: replicated
      replicated:
        size: 3
  preserveFilesystemOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```

Rook Ceph Operator 将创建启动服务所需的所有池和其他资源。这可能需要一些时间来完成。

```sh
$ kubectl create -f filesystem.yaml
```

要确认文件系统已配置完成，请等待 mds Pod 启动：

```sh
$ kubectl -n rook-ceph get pod -l app=rook-ceph-mds
NAME                                   READY   STATUS    RESTARTS   AGE
rook-ceph-mds-myfs-a-9cc6945cf-tvv9x   1/1     Running   0          44s
rook-ceph-mds-myfs-b-b988647d4-spmh5   1/1     Running   0          42s
```

要查看文件系统的详细状态，进入 Rook toolbox 中，使用`ceph status`查看， 确认输出中包含 MDS 服务的状态。在这个示例中，有一个处于活动状态的 MDS 实例，并且还有一个处于备用 MDS 实例，以防发生故障切换。

```sh
bash-4.4$ ceph -s
  cluster:
    id:     80acf00a-a7e1-40fe-b6c2-30f546e519bb
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum a,b,c (age 20h)
    mgr: a(active, since 20h), standbys: b
    mds: 1/1 daemons up, 1 hot standby
    osd: 3 osds: 3 up (since 20h), 3 in (since 20h)

  data:
    volumes: 1/1 healthy
    pools:   4 pools, 81 pgs
    objects: 39 objects, 3.7 MiB
    usage:   149 MiB used, 1.5 TiB / 1.5 TiB avail
    pgs:     81 active+clean

  io:
    client:   852 B/s rd, 1 op/s rd, 0 op/s wr
```

使用`ceph osd lspools`查看，确认创建了`myfs-metadata`和`myfs-replicated`的存储池。

```sh
bash-4.4$ ceph osd lspools
1 .mgr
3 replicapool
4 myfs-metadata
5 myfs-replicated
```

### 9.4.2.2 CEPHFS 存储供给

在 Rook 开始提供 CephFS 存储之前，需要基于文件系统创建一个 StorageClass。这对于 Kubernetes 与 CSI 驱动程序进行交互以创建持久卷是必需的。

将以下存储类定义保存为 storageclass.yaml 文件：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-cephfs
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.cephfs.csi.ceph.com # driver:namespace:operator
parameters:
  # clusterID is the namespace where the rook cluster is running
  # If you change this namespace, also change the namespace below where the secret namespaces are defined
  clusterID: rook-ceph # namespace:cluster

  # CephFS filesystem name into which the volume shall be created
  fsName: myfs

  # Ceph pool into which the volume shall be created
  # Required for provisionVolume: "true"
  pool: myfs-replicated

  # The secrets contain Ceph admin credentials. These are generated automatically by the operator
  # in the same namespace as the cluster.
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/controller-expand-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/controller-expand-secret-namespace: rook-ceph # namespace:cluster
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph # namespace:cluster

  # (optional) The driver can use either ceph-fuse (fuse) or ceph kernel client (kernel)
  # If omitted, default volume mounter will be used - this is determined by probing for ceph-fuse
  # or by setting the default mounter explicitly via --volumemounter command-line argument.
  # mounter: kernel
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  # uncomment the following line for debugging
  #- debug
```

如果你将 Rook Operator 部署在了一个与"rook-ceph"不同的命名空间中，请修改 provisioner 中的前缀以匹配你使用的命名空间。

例如，如果 Rook Operator 运行在"rook-op"命名空间中，那么 provisioner 的值应该是`rook-op.rbd.csi.ceph.com`。

创建存储类:

```sh
$ kubectl create -f storageclass.yaml
storageclass.storage.k8s.io/rook-cephfs created

$ kubectl get sc rook-cephfs
NAME          PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-cephfs   rook-ceph.cephfs.csi.ceph.com   Delete          Immediate           true                   4m39s
```

### 9.4.2.3 关于配额

CephFS CSI 驱动程序使用配额来强制执行所请求的 PVC 大小。只有较新的 Linux 内核支持 CephFS 配额，至少是 4.17 版本的内核。

### 9.4.2.4 使用存储: 多个 POD 挂载

创建下面的 busybox.yaml:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: busybox-data-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
  storageClassName: rook-cephfs

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
        - name: busybox-container
          image: busybox:stable-glibc
          command: ["sleep", "3600"]
          volumeMounts:
            - name: data-volume
              mountPath: /data
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: busybox-data-pvc
```

创建这个 busybox 的 deployment 和 pvc:

```sh
$ kubectl apply -f busybox.yaml
persistentvolumeclaim/busybox-data-pvc created
deployment.apps/busybox created
```

查看创建的 PVC 和自动供给的 PV:

```sh
$ kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
busybox-data-pvc                   Bound    pvc-38fe93c8-c55b-4e18-b955-15a09dc45943   2Gi        RWX            rook-cephfs       21s

$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                        STORAGECLASS      REASON   AGE
pvc-38fe93c8-c55b-4e18-b955-15a09dc45943   2Gi        RWX            Delete           Bound    rook-ceph/busybox-data-pvc                   rook-cephfs                25s
```

查看 PV 的的具体信息:

```sh
$ kubectl describe pv pvc-38fe93c8-c55b-4e18-b955-15a09dc45943
Name:            pvc-38fe93c8-c55b-4e18-b955-15a09dc45943
Labels:          <none>
Annotations:     pv.kubernetes.io/provisioned-by: rook-ceph.cephfs.csi.ceph.com
                 volume.kubernetes.io/provisioner-deletion-secret-name: rook-csi-cephfs-provisioner
                 volume.kubernetes.io/provisioner-deletion-secret-namespace: rook-ceph
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    rook-cephfs
Status:          Bound
Claim:           rook-ceph/busybox-data-pvc
Reclaim Policy:  Delete
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        2Gi
Node Affinity:   <none>
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            rook-ceph.cephfs.csi.ceph.com
    FSType:
    VolumeHandle:      0001-0009-rook-ceph-0000000000000001-4f7108ef-5d3e-4763-b429-f2437c2db4c2
    ReadOnly:          false
    VolumeAttributes:      clusterID=rook-ceph
                           fsName=myfs
                           pool=myfs-replicated
                           storage.kubernetes.io/csiProvisionerIdentity=1704338799133-1100-rook-ceph.cephfs.csi.ceph.com
                           subvolumeName=csi-vol-4f7108ef-5d3e-4763-b429-f2437c2db4c2
                           subvolumePath=/volumes/csi/csi-vol-4f7108ef-5d3e-4763-b429-f2437c2db4c2/81a009f7-8e8c-471a-ae4c-4caf1b898e6d
Events:                <none>
```

可以看到持久化卷挂载是 CephFS 实例 myfs 中的`/volumes/csi/csi-vol-4f7108ef-5d3e-4763-b429-f2437c2db4c2/81a009f7-8e8c-471a-ae4c-4caf1b898e6d`。

## 9.4.3 对象存储

参考文章：[使用 Rook 自动部署和管理 Ceph 集群 | 青蛙小白 (frognew.com)](https://blog.frognew.com/2023/06/rook-quick-start.html)



# rook-ceph 卸载、重新部署

```sh
#First: Uninstall the chart.
$ helm uninstall -n rook-ceph rook-ceph
$ helm -n rook-ceph uninstall rook-ceph-cluster


#Second: Delete the remains secrets and configmaps:
$ k -n rook-ceph get cm,secrets
NOTE: If they still remain, use k -n rook-ceph edit cm/NAME and delete the finalizer from it, save, and quit.


#Third: Delete the Rook Resources itself:
$ k -n rook-ceph delete CephObjectStore ceph-objectstore
$ k -n rook-ceph delete CephFilesystem ceph-filesystem
$ k -n rook-ceph delete CephBlockPool ceph-blockpool
$ k -n rook-ceph delete CephBlockPool rook-ceph

#Fourth: Delete the Rook CRDs:
kubectl delete -f rook/deploy/examples/crds.yaml
NOTE: If they still remain, use the below command:
# 或者使用如下
for CRD in $(kubectl get crd -n rook-ceph | awk '/ceph.rook.io/ {print $1}'); do
    kubectl get -n rook-ceph "$CRD" -o name | \
    xargs -I {} kubectl patch -n rook-ceph {} --type merge -p '{"metadata":{"finalizers": []}}'
done

# 如果出现删不掉使用如下
kubectl patch crd/NAME -p '{"metadata":{"finalizers":[]}}' --type=merge

# Finally: Delete /var/lib/rook on every worker node.
sudo rm -rf /var/lib/rook
```

# 测试数据清理

如果 Rook 要继续使用，可以只清理创建的 deploy、pod、pvc 即可。之后可以直接投入使用。

## 数据删除步骤

数据清理步骤：

1. 首先清理挂载了 PVC 和 Pod，可能需要清理单独创建的 Pod 和 Deployment 或者是其他的高级资源
2. 之后清理 PVC，清理掉所有通过 ceph StorageClass 创建的 PVC 后，最好检查下 PV 是否被清理
3. 之后清理快照：`kubectl delete volumesnapshot XXXXXXXX`
4. 之后清理创建的 Pool，包括块存储和文件存储

```sh
$ kubectl delete -n rook-ceph cephblockpool replicapool
$ kubectl delete -n rook-ceph cephfilesystem myfs
```

- 清理 StorageClass：

```sh
$ kubectl delete sc rook-ceph-block rook-cephfs
```

- 清理 Ceph 集群：

```sh
kubectl -n rook-ceph delete cephcluster rook-ceph
```

- 删除 Rook 资源：

```sh
$ kubectl delete -f operator.yaml
$ kubectl delete -f common.yaml
$ kubectl delete -f crds.yaml
```

1.如果卡住需要参考 Rook 的[troubleshooting](https://rook.io/docs/rook/v1.6/ceph-teardown.html#troubleshooting)

```sh
for CRD in $(kubectl get crd -n rook-ceph | awk '/ceph.rook.io/ {print $1}');
do
    kubectl get -n rook-ceph "$CRD" -o name |   xargs -I {} kubectl patch {} --type merge -p '{"metadata":{"finalizers": [null]}}' -n rook-ceph;
done
```

2.清理数据目录和磁盘

参考链接：https://rook.io/docs/rook/v1.6/ceph-teardown.html#delete-the-data-on-hosts

参考链接：https://rook.io/docs/rook/v1.6/ceph-teardown.html

```sh
kubectl -n rook-ceph patch configmap rook-ceph-mon-endpoints --type merge -p '{"metadata":{"finalizers": [null]}}'
kubectl -n rook-ceph patch secrets rook-ceph-mon --type merge -p '{"metadata":{"finalizers": [null]}}'
```

## 删除 ceph pool 存储池

```sh
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
bash-4.4$ ceph fs status
myfs - 0 clients
====
RANK  STATE   MDS  ACTIVITY  DNS  INOS  DIRS  CAPS
 0    failed
      POOL         TYPE     USED  AVAIL
 myfs-metadata   metadata  1632k   474G
myfs-replicated    data       0    474G
bash-4.4$ ceph fs set myfs down true
myfs marked down.

bash-4.4$ ceph fs status
myfs - 0 clients
====
RANK  STATE   MDS  ACTIVITY  DNS  INOS  DIRS  CAPS
 0    failed
      POOL         TYPE     USED  AVAIL
 myfs-metadata   metadata  1632k   474G
myfs-replicated    data       0    474G

bash-4.4$ ceph fs rm myfs --yes-i-really-mean-it
bash-4.4$ ceph fs status
bash-4.4$ ceph osd pool ls
.mgr
replicapool
myfs-metadata
myfs-replicated

bash-4.4$ ceph osd pool delete myfs-metadata myfs-metadata --yes-i-really-really-mean-it
pool 'myfs-metadata' removed

bash-4.4$ ceph osd pool delete myfs-replicated myfs-replicated --yes-i-really-really-mean-it
pool 'myfs-replicated' removed

bash-4.4$ ceph fs status
bash-4.4$ ceph -s
```

参考：[4.12. 使用命令行界面删除 Ceph 文件系统 Red Hat Ceph Storage 4 | Red Hat Customer Portal](https://access.redhat.com/documentation/zh-cn/red_hat_ceph_storage/4/html/file_system_guide/removing-a-ceph-file-system-using-the-command-line-interface_fs#doc-wrapper)

## 删除 rbd 硬盘

```sh
bash-4.4$ rbd ls -p replicapool
csi-vol-5e4e1cd5-d889-4705-9575-c65f0e0b4d01

bash-4.4$ rbd rm replicapool/csi-vol-5e4e1cd5-d889-4705-9575-c65f0e0b4d01
Removing image: 100% complete...done.
```

## ceph 集群报"daemons have recently crashed"问题处理

1.产生该问题的原因是数据在均衡或者回滚等操作的时候，导致其某个守护进程崩溃了，且没有及时归档，所以集群产生告警。

2.解决办法

```sh
$ ceph crash ls
$ ceph crash archive <id>
OR
$ ceph crash archive-allh
```

３.查看集群状态

```
$ ceph -s
```

