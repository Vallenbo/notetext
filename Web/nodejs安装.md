一、NodeJS基本介绍

 1、NodeJS是为了开发高性能的服务器而诞生的一种技术

 2、是运行在服务端的 JavaScript，基于V8（谷歌浏览器的版本）进行运行 

 3、使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效

二、学习node的意义

 1、 开发沟通：开发时更容易理解后端实现，降低交流成本

 2、 后端开发：想写些自己感兴趣的项目时，可以自己独立完成，即使没有后端支持，且成本特别低。

 3、 中间层开发：为了进一步的前后端分离，提高性能，使用nodejs做中间层是一个非常好的实践（由于nodejs具有异步io的特点）

# 安装方式

方法一：源码包安装 官网下载 centos下载最新版10.9 

```shell
wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz mkdir /opt/software/ && cd  /opt/software/ tar -xvf node-v10.9.0-linux-x64.tar.xz mv node-v10.9.0-linux-x64 nodejs #建立软连接，变为全局    

①ln -s /opt/software/nodejs/bin/npm /usr/local/bin/     
②ln -s /opt/software/nodejs/bin/node /usr/local/bin/ 查看安装的版本 

[root@localhost]# node -v v10.9.0
[root@localhost]# npm -v  6.2.0
```

方法二：nvm方式安装 curl: 

```shell
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh  
$ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh  #安装完成后，执行下列命令即可安装 Node.js。
$ nvm install stable 查看安装的版本 [root@localhost]# node -v v10.9.0
[root@localhost]# npm -v  6.2.0 
```

 方法三：yum方式

```shell
curl -fsSL https://rpm.nodesource.com/setup_19.x | bash -
yum install -y nodejs

[root@localhost /]# node -v
v10.9.0

[root@localhost /]# npm -v
6.2.0
```





# 配置yarn官方yum存储库

```shell
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo
```





# 安装yarn

```shell
yum install yarn
```





# linux下部署安装

Node.js 安装包及源码下载地址为：https://nodejs.org/en/download/，选择Linux Binaries (x64)

或者：`wget https://nodejs.org/dist/latest-v14.x/node-v14.4.0-linux-x64.tar.gz`

```sh
tar -xzvf node-v14.4.0-linux-x64.tar.gz 进行解压
echo "export PATH=/usr/local/lib/node-v14.4.0-linux-x64/bin:$PATH" >>/etc/profile 设置环境
source /etc/profile //刷新文件
node -v && npm version && npx -v //使用测试安装
```

