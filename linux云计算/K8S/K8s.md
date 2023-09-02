# **第一章**  kubernetes基础

Kubernetes是谷歌以Borg为前身，基于谷歌15年生产环境经验的基础上开源的一个项目，Kubernetes致力于提供跨主机集群的自动部署、扩展、高可用以及运行应用程序容器的平台。

kubectl (Command line tool)





## 1 Master节点：整个集群的控制中枢

Ø Kube-APIServer：集群的控制中枢，各个模块之间信息交互都需要经过Kube-APIServer，同时它也是集群管理、资源配置、整个集群安全机制的入口。

Ø Controller-Manager：集群的状态管理器，保证Pod或其他资源达到期望值，也是需要和APIServer进行通信，在需要的时候创建、更新或删除它所管理的资源。

Ø Scheduler：集群的调度中心，它会根据指定的一系列条件，选择一个或一批最佳的节点，然后部署我们的Pod。

Ø Etcd：键值数据库，报错一些集群的信息，一般生产环境中建议部署三个以上节点（奇数个）。

 

## 2  Node：工作节点

​	Worker、node节点、minion节点

Ø Kubelet：负责监听节点上Pod的状态，同时负责上报节点和节点上面Pod的状态，负责与Master节点通信，并管理节点上面的Pod。

Ø Kube-proxy：负责Pod之间的通信和负载均衡，将指定的流量分发到后端正确的机器上。

​		查看Kube-proxy工作模式：curl 127.0.0.1:10249/proxyMode

​			1、Ipvs：监听Master节点增加和删除service以及endpoint的消息，调用Netlink接口创建相应的IPVS规则。通过IPVS规则，将流量转发至相应的Pod上。

​			2、Iptables：监听Master节点增加和删除service以及endpoint的消息，对于每一个Service，他都会场景一个iptables规则，将service的clusterIP代理到后端对应的Pod。



### 其他组件

Ø Calico：符合CNI标准的网络插件，给每个Pod生成一个唯一的IP地址，并且把每个节点当做一个路由器。Cilium

Ø CoreDNS：用于Kubernetes集群内部Service的解析，可以让Pod把Service名称解析成IP地址，然后通过Service的IP地址进行连接到对应的应用上。

Ø Docker：容器引擎，负责对容器的管理。

## 3 pod

### 1 什么是Pod？

Pod是Kubernetes中最小的单元，它由一组、一个或多个容器组成，每个Pod还包含了一个Pause容器，Pause容器是Pod的父容器，主要负责僵尸进程的回收管理，通过Pause容器可以使同一个Pod里面的多个容器共享存储、网络、PID、IPC等。

### 2 定义一个Pod

pod的yaml文件格式

```yaml
apiVersion: v1 # 必选，API的版本号
kind: Pod       # 必选，类型Pod
metadata:       # 必选，元数据
  name: nginx   # 必选，符合RFC 1035规范的Pod名称
  # namespace: default # 可选，Pod所在的命名空间，不指定默认为default，可以使用-n 指定namespace 
  labels:       # 可选，标签选择器，一般用于过滤和区分Pod
    app: nginx
    role: frontend # 可以写多个
  annotations:  # 可选，注释列表，可以写多个
    app: nginx
spec:   # 必选，用于定义容器的详细信息
#  initContainers: # 初始化容器，在容器启动之前执行的一些初始化操作
#  - command:
#    - sh
#    - -c
#    - echo "I am InitContainer for init some configuration"
#    image: busybox
#    imagePullPolicy: IfNotPresent
#    name: init-container
  containers:   # 必选，容器列表
  - name: nginx # 必选，符合RFC 1035规范的容器名称
    image: nginx:1.15.2    # 必选，容器所用的镜像的地址
    imagePullPolicy: IfNotPresent     # 可选，镜像拉取策略, IfNotPresent: 如果宿主机有这个镜像，那就不需要拉取了. Always: 总是拉取, Never: 不管是否存储都不拉去
    command: # 可选，容器启动执行的命令 ENTRYPOINT, arg --> cmd
    - nginx 
    - -g
    - "daemon off;"
    workingDir: /usr/share/nginx/html       # 可选，容器的工作目录
#    volumeMounts:   # 可选，存储卷配置，可以配置多个
#    - name: webroot # 存储卷名称
#      mountPath: /usr/share/nginx/html # 挂载目录
#      readOnly: true        # 只读
    ports:  # 可选，容器需要暴露的端口号列表
    - name: http    # 端口名称
      containerPort: 80     # 端口号
      protocol: TCP # 端口协议，默认TCP
    env:    # 可选，环境变量配置列表
    - name: TZ      # 变量名
      value: Asia/Shanghai # 变量的值
    - name: LANG
      value: en_US.utf8
#    resources:      # 可选，资源限制和资源请求限制
#      limits:       # 最大限制设置
#        cpu: 1000m
#        memory: 1024Mi
#      requests:     # 启动所需的资源
#        cpu: 100m
#        memory: 512Mi
#    startupProbe: # 可选，检测容器内进程是否完成启动。注意三种检查方式同时只能使用一种。
#      httpGet:      # httpGet检测方式，生产环境建议使用httpGet实现接口级健康检查，健康检查由应用程序提供。
#            path: /api/successStart # 检查路径
#            port: 80
#    readinessProbe: # 可选，健康检查。注意三种检查方式同时只能使用一种。
#      httpGet:      # httpGet检测方式，生产环境建议使用httpGet实现接口级健康检查，健康检查由应用程序提供。
#            path: / # 检查路径
#            port: 80        # 监控端口
#    livenessProbe:  # 可选，健康检查
      #exec:        # 执行容器命令检测方式
            #command: 
            #- cat
            #- /health
    #httpGet:       # httpGet检测方式
    #   path: /_health # 检查路径
    #   port: 8080
    #   httpHeaders: # 检查的请求头
    #   - name: end-user
    #     value: Jason 
#      tcpSocket:    # 端口检测方式
#            port: 80
#      initialDelaySeconds: 60       # 初始化时间
#      timeoutSeconds: 2     # 超时时间
#      periodSeconds: 5      # 检测间隔
#      successThreshold: 1 # 检查成功为2次表示就绪
#      failureThreshold: 2 # 检测失败1次表示未就绪
#    lifecycle:
#      postStart: # 容器创建完成后执行的指令, 可以是exec httpGet TCPSocket
#        exec:
#          command:
#          - sh
#          - -c
#          - 'mkdir /data/ '
#      preStop:
#        httpGet:      
#              path: /
#              port: 80
      #  exec:
      #    command:
      #    - sh
      #    - -c
      #    - sleep 9
  restartPolicy: Always   # 可选，默认为Always，容器故障或者没有启动成功，那就自动该容器，Onfailure: 容器以不为0的状态终止，自动重启该容器, Never:无论何种状态，都不会重启
  #nodeSelector: # 可选，指定Node节点
  #      region: subnet7
#  imagePullSecrets:     # 可选，拉取镜像使用的secret，可以配置多个
#  - name: default-dockercfg-86258
#  hostNetwork: false    # 可选，是否为主机模式，如是，会占用主机端口
#  volumes:      # 共享存储卷列表
#  - name: webroot # 名称，与上述对应
#    emptyDir: {}    # 挂载目录
#        #hostPath:              # 挂载本机目录
#        #  path: /etc/hosts
```

 

### 3 Pod探针

**Readiness 探针（就绪检测）**：
检查应用程序是否准备好为请求提供服务。

如果 Readiness 探针失败，该 Pod 的 IP 地址不会被列入与该 Pod 关联的 Service 的 Endpoints。

使用场景：确保流量只发送到已经准备好处理请求的 Pod。例如，在容器中的数据库或其他依赖服务启动并准备好之前，一个 Web 服务器可能还没准备好接受请求。

**Liveness 探针（活性检测）**：
检查应用程序是否还在运行。

如果 Liveness 探针失败，Kubelet 会杀死容器，然后根据 Pod 的重启策略决定是否重启容器。

使用场景：确保应用程序处于运行状态，如果应用程序挂起或死锁，Liveness 探针可以确保它被重新启动。

**Startup 探针（启动检测）**

检查容器应用是否已经开始。

如果 Startup 探针失败，Kubelet 将杀死容器，并且容器根据其重启策略进行重启。

使用场景：某些应用程序在启动时可能需要较长的初始化时间，这种情况下，Liveness 探针可能会因为时间过长而失败，Startup探针可以用来专门检查启动状态，而不影响 Liveness 探针的其他检查。

### 4 Pod探针的检测方式

HTTPGetAction：通过应用程序暴露的API地址来检查程序是否是正常的，如果状态码为200~400之间，则认为容器健康。生产环境使用

ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康。

TCPSocketAction：通过TCP连接检查容器内的端口是否是通的，如果是通的就认为容器健康。

### 5 探针检查参数配置

initialDelaySeconds: 60    # 初始化时间
timeoutSeconds: 2   # 超时时间
periodSeconds: 5    # 检测间隔
successThreshold: 1 # 检查成功为1次表示就绪
failureThreshold: 2 # 检测失败2次表示未就绪

```yaml
# 探针示例文件
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        readinessProbe: # 就绪探针检测
          httpGet: # Http Get方式
            path: /index.html # 目标路径
            port: 80 # 目标端口
          failureThreshold: 5 # 失败多少次才真的失效
          periodSeconds: 10 # 检测间隔时间
          timeoutSeconds: 10 # 请求超时时间
```

 

Prestop：先去请求eureka接口，把自己的IP地址和端口号，进行下线，eureka从注册表中删除该应用的IP地址。然后容器进行sleep 90；kill `pgrep java`

 

 
