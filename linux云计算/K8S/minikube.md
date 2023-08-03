minikube[官网安装教程](https://minikube.sigs.k8s.io/docs/start/)

## 1、安装

linux-x64版本下载安装

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
install minikube-linux-amd64 /usr/local/bin/minikube
```

还需要[安装docker](https://docs.docker.com/engine/install/ubuntu/)

## 2、启动群集

从具有管理员访问权限（但未以 root 身份登录）的终端中，运行：

```shell
minikube start --driver=docker --force
#minikube start
```

## 3、与群集交互

如果您已经安装了 kubectl，您现在可以使用它来访问您闪亮的新集群：

```shell
minikube kubectl -- get pods -A
```

您还可以通过将以下内容添加到 shell 配置中来使您的生活更轻松：

```shell
alias kubectl="minikube kubectl --"
```

最初，某些服务（如存储预配程序）可能尚未处于“正在运行”状态。这是群集启动期间的正常情况，会立即自行解决。为了进一步了解您的集群状态，minikube 捆绑了 Kubernetes 仪表板，让您轻松适应新环境：

```shell
minikube dashboard
```



## 4、部署应用程序

- [服务](https://minikube.sigs.k8s.io/docs/start/#)
- [负载均衡器](https://minikube.sigs.k8s.io/docs/start/#)
- [入口](https://minikube.sigs.k8s.io/docs/start/#)

创建示例部署并在端口 8080 上公开它：

```shell
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

这可能需要一些时间，但运行时很快就会显示您的部署：

```shell
kubectl get services hello-minikube
```

访问此服务的最简单方法是让minikube为您启动Web浏览器：

```shell
minikube service hello-minikube
```

或者，使用 kubectl 转发端口：

```shell
kubectl port-forward service/hello-minikube 7080:8080
```

哒哒！您的申请现已 http://localhost:7080/ 提供。

您应该能够在应用程序输出中看到请求元数据。尝试更改请求的路径并观察更改。同样，您可以执行 POST 请求并观察正文显示在输出中。

## 5、管理您的集群

暂停 Kubernetes 而不影响已部署的应用程序：

```shell
minikube pause
```

取消暂停暂停的实例：

```shell
minikube unpause
```

停止群集：

```shell
minikube stop
```

更改默认内存限制（需要重新启动）：

```shell
minikube config set memory 9001
```

浏览易于安装的 Kubernetes 服务目录：

```shell
minikube addons list
```

创建运行较旧 Kubernetes 版本的第二个集群：

```shell
minikube start -p aged --kubernetes-version=v1.16.1
```

删除所有迷你集群：

```shell
minikube delete --all
```