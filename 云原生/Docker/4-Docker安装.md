# Docker-ce包社区版安装

[docker安装 官方文档](https://docs.docker.com/engine/install/ubuntu/)

## 一、在线安装Docker

1: 安装必要的一些系统工具	

```sh
yum install -y yum-utils
```

2: 添加软件源信息

```sh
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3: 更新并安装Docker-CE

```sh
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

**配置镜像加速器 vim /etc/docker/daemon.json**

```sh
{
"registry-mirrors": ["https://registry.docker-cn.com"，"https://docker.mirrors.ustc.edu.cn"]
}
```

`systemctl daemon-reload && systemctl restart docker`	#重新加载配置文件及服务

## **二、离线二进制安装**

```sh
curl -O https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz
cp docker/* /usr/bin/   # 复制到可执行目录
dockerd &  				# 启动Docker守护程序
docker-api操作，/etc/sysconfig/docker

[root@localhost ~]# systemctl enable docker				#添加/var/run/docker.sock文件
OPTIONS='--selinux-enabled --log-driver=journald -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock' #添加api接口
```

## **三、脚本安装**

#运行docker便捷安装脚本

```sh
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

卸载docker

•  检查安装的docker			`yum list installed | grep docker`

•  如查找出内容需要先进行docker卸载，无内容即可正常安装

•  卸载docker				   `yum -y remove docker 名称`

•  删除镜像或者容器等等		`rm -rf docker路径`



# [在 Ubuntu 上安装 Docker 引擎 |Docker 文档](https://docs.docker.com/engine/install/ubuntu/#prerequisites)

步骤 1：添加Docker官方的GPG密钥

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

步骤 2：添加Docker的稳定版存储库

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

步骤 3：安装Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

步骤 4：验证Docker安装

```bash
sudo docker --version
```

步骤 5：将当前用户添加到docker组（可选）

```bash
sudo usermod -aG docker $USER
```

