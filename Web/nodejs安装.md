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





## 配置yarn官方yum存储库

```shell
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo
```





## 安装yarn

```shell
yum install yarn
```

