Docker是一种容器技术，解决软件跨环境迁移的问题

优势; 更快速的交付和部署	更高效的资源利用	更轻松的迁移和扩展	更简单的更新管理

 

Docker是一个开源的应用容器引擎	(c/s模式)构建Paas层的应用

基于Go语言实现

可以让开发者打包应用(及依赖包)到一个可移植的容器中然后发布到其他liunx机器实现虚拟化

容器完全使用虚拟化沙箱机制，相互之间不会有任何接口

Docker引擎 包括支持在桌面系统或云平台安装 Docker，以及为企业提供简单安全弹性的容器集群编排和管理，17.3版本后分为CE社区版和EE企业版

DockerHub 官方提供的云托管服务，可以提供公有或私有的镜像仓库

DockerCloud 官方提供的容器云服务，可以完成容器的部署与管理，可以完整地支持容器化项目，还有CI、 CD功能

<img src="E:\Project\Textbook\linux云计算\assets\wps1-1682691150322-323.jpg" alt="img" style="zoom: 67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps2-1682691150322-324.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps3-1682691150322-325.jpg" alt="img" style="zoom: 67%;" /> 

不了解Linux内核的Cgroups技术，无法知道容器是如何做资源(CPU、 内存等)限制的

不了解Linux的Namespace技术，无法知道容器是如何做主机名、网络、文件等资源隔离的

dockerd负责响应和处理来自Docker客户端的请求然后将客户端的请求转化为Docker的具体操作



Docker是基于Namespace、Cgroups 和联合文件系统实现的

Cgroups不仅可以用于容器资源的的限制，还可以提供容器的资源使用率

Cgroups的工作目录/sys/fs/cgroup下包含了Cgroups的所有内容





<img src="E:\Project\Textbook\linux云计算\assets\wps4-1682691150322-326.jpg" alt="img" style="zoom:50%;" /> 

容器的本质是进程而不是一个完整操作系统



# 安装docker-ce包社区版

## 一、在线安装Docker

\# 1: 安装必要的一些系统工具	`yum install -y yum-utils`

\# 2: 添加软件源信息

```sh
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

\# 3: 更新并安装Docker-CE		

```sh
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

配置镜像加速器`vim  /etc/docker/daemon.json`

```sh
{"registry-mirrors": ["https://ustc-edu-cn.mirror.aliyuncs.com","https://docker.mirrors.ustc.edu.cn"]}
```

`systemctl daemon-reload && systemctl restart docker`	#重新加载配置文件及服务

## **二、离线二进制安装**

```
curl -O https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz 
cp docker/* /usr/bin/ #复制到可执行目录
dockerd &  #启动Docker守护程序
docker-api操作，/etc/sysconfig/docker

[root@localhost ~]# systemctl enable docker	#添加/var/run/docker.sock文件
OPTIONS='--selinux-enabled --log-driver=journald -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock' #添加api接口
```



## **三、脚本安装**

`curl -fsSL https://test.docker.com -o test-docker.sh `#运行docker便捷安装脚本

[Docker官方文档](https://docs.docker.com/engine/install/centos/ )

卸载docker

•  检查安装的docker			`yum list installed | grep docker`

•  如查找出内容需要先进行docker卸载，无内容即可正常安装

•  卸载docker			`	yum -y remove docker 名称`

•  删除镜像或者容器等等		`rm -rf docker路径`

镜像操作

![img](E:\Project\Textbook\linux云计算\assets\wps5-1682691150322-327.jpg) 

<img src="E:\Project\Textbook\linux云计算\assets\wps6-1682691150322-328.jpg" alt="img" style="zoom: 67%;" /> 

docker镜像：是一个只读的Docker容器模板，包含启动容器所需要的所有文件系统结构和内容

 

<img src="E:\Project\Textbook\linux云计算\assets\wps7-1682691150322-329.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps8-1682691150322-330.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps9-1682691150322-331.jpg" alt="img" style="zoom:67%;" /> 

**man docker-images查看镜像**

导出镜像docker save -o zi_redis.tar zi_redis:1.0镜像打包成tar文件（-o指定保存目录）

导入镜像docker load<zi_redis.tar压缩文件还原成镜像文件

注: 数据卷目录不能被制作成自制镜像的一部分

上传镜像docker push test:latest 上传本地的test :latest镜像

获取镜像docker pull redis:3.2拉取相应镜像版本(默认latest最新版)	-a获取仓库中的所有镜像

查看镜像docker images查看本地镜像						-a列出所有（包括临时文件）镜像文件

-f since=centos过滤列出来自centos库的镜像				-q仅输出ID信息

Docker tag  ubunbu:latest  xxx:lates添加xxx标签(类似链接的作用)

docker inspect centos/a1查看镜像或容器信息				-f {{".Architecture"}} nginx指定查看内容

docker history ubuntu:18.04查看镜像创建各层级的具体信息

搜索镜像Docker search Ubuntu 搜索Ubuntu镜像				-f is-official=true nginx过滤出官方的镜像

删除清理镜像docker  rmi  redis:3.2删除镜像(id号也行)		-f强制删除即使有容器依赖它

docker image prune -a删除所有无用镜像不光是临时镜像

-f is-official=true nginx出官方的镜像 						-f自动清理且强制删除镜像不进行提示确认

ll *.tar|awk '{print $NF}'|sed -r 's#(.*)#docker load -i \1#' |bash #批量导入镜像

Dockerfile创建镜像

<img src="E:\Project\Textbook\linux云计算\assets\wps10-1682691150322-332.jpg" alt="img" style="zoom: 67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps11-1682691150323-333.jpg" alt="img" style="zoom: 67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps12-1682691150323-334.jpg" alt="img" style="zoom:67%;" /> <img src="E:\Project\Textbook\linux云计算\assets\wps13-1682691150323-335.jpg" alt="img" style="zoom:67%;" />

dockerfile书写原则https://www.cnblogs.com/ainimore/p/14409165.html

 

创建docker-file文件夹进入编写vim dockerfile1（构建ssh镜像文件）

```dockerfile
FROM 192.168.2.5:5000/centos/latest:latest
MAINTAINER  liu  <2581210093@qq.com>
RUN rm -f /etc/yum.repos.d/*
ADD yum.repo /etc/yum.repos.d/	#本地相对路径文件yum.repo，添加至容器内部目录下/etc/yum.repos.d/
RUN yum repolist
RUN yum install mariadb-server -y
RUN mysql_install_db --user=mysql	#使用mysql用户初始化数据库
ENV LC_ALL en_US.UTF-8			#语言环境设置
ENV MYSQL_USER xiandian MYSQL_PASS xiandian		#环境设置
EXPOSE 3306
CMD mysql_safe	#运行命令
```

`docker build -t centos:v1 . `			`docker build -f /test/Dockerfile .`

\#构建镜像，-f指定dockerfile文件，-t指定镜像标签

`docker run -it --name c5 -p 9999:22 centos_sshd:1.0` #映射端口进入

**## Dockerfile****关键字**	**[区分大小写]**

|FROM   |指定父镜像  |指定dockerfile基于那个image构建 | FROM centos:7

|MAINTAINER|作者信息 |用来标明这个dockerfile谁写的 | MAINTAINER lb <codelnb@qq.com>

|LABEL |标签 |用来标明dockerfile的标签可以使用Label代替Maintainer 最终都是在docker image基本信息中可以查看 |LABEL version="l.0.0-rc3" LABEL author="yeasy@github" date="2020-01-01"

|RUN  |执行命令 |格式为 RUN <command＞（前者默认将在 shell 终端中运行命令，即／bin/sh -c, 命令较长时可以使用＼来换行）或 RUN ["executable ",”paraml”,”param2”]

RUN [“/bin/bash”,“-c”,echo hello”］#指定bash环境

|ENTRYPOINT|入口 |镜像的默认入口命令，入口命令在启动容器时作为根命令执行，所有传入值作为该命令的参数

ENTRYPOINT ["nginx"]	#nginx命令

CMD ["-g","daemon off;"]	#至附加参数

|CMD  |容器启动命令 |指定启动容器时默认执行的命令,如果用户run创建容器时执行命令就会覆盖掉该命令

|COPY |复制文件 |复制本地主机的 为Dockerfile所在目录的相对路径，文件或目录)为容器中的某路径;。目录路径不存在时，会自动创建 |COPY /media /sss

|ADD  |添加文件 |将本地的一个文件或目录拷贝到容器的某个目录里。为Dockerfile所在目录的相对路 径，它也可以是一个URL；如果为tar文件，会自动解压到/;路径下。可以是镜像内的绝对路径，或者相对于工作目录（WORKDIR）的相对路径 | ADD http://nginx.org/download/nginx-1.12.2.tar.gz .

|ENV  |环境变量 |指定环境变量， 在镜像生成过程中会被后续RUN指令使用， 在镜像启动的容器中也会存在. 指令指定的环境变量在运行时可以被覆盖掉， 如docker run --env| ENV APP VERSION=l.0.0 	ENV APP_HOME=/usr/local/app 		ENV PATH /usr/local/mysql/bin:$PATH

|ARG  | 构建参数| 定义创建镜像过程中使用的变量|Docker 内置了一些镜像创建变量。用户可以直接使用而无须声明， 包括（不区分大小写） HTTP PROXY、 HTTPS PROXY、 FTP PROXY、 NO PROXY|ARG VERSION=9.3

|VOLUME | 定义外部可以挂载的数据卷 | 创建一个可以从本地主机或其他容器挂载的挂载点，一般用于存放数据库和需要保持的数据,启动容器的时候使用 -v 绑定 | VOLUME ["/emdia"] 

|EXPOSE | 暴露端口 |定义容器运行的时候监听的端口,启动容器的使用-p来绑定暴露端口 | EXPOSE 22 8080/udp

|WORKDIR | 工作目录 |为后续的 RUN、 CMD、 ENTRYPOINT 指令配置工作目录 |WORKDIR /path/to/workdir

|USER   | 指定执行用户| 指定运行容器时的用户名或UID，后续的RUN也会指定用户.当服务不需要管理员权限时，可以通过该指令指定运行的用户。并且可以在之前创建所需要的用户 |(注：要临时获取管理员权限可以使用gosu，而不推荐sudo)RUN groupadd -r postgres && useradd -r -g postgres postgres| USER root

|HEALTHCHECK| 健康检查 |指定监测当前容器的健康监测的命令基本上没用因为很多时候应用本身有健康监测机制 |

|ONBUILD | 触发器 |当存在ONBUILD关键字的镜像作为基础镜像的时候 当执行FROM完成之后会执行 ONBUILD的命令但是不影响当前镜像用处也不怎么大 |

ONBUILD ADD . / app/src ONBUILD RUN /usr / local/bin/python build --dir / app/src

|STOPSIGNAL |发送信号量到宿主机	|指定所创建镜像启动的容器接收退出的信号值 | STOPSIGNAL signal

|SHELL  |指定执行脚本的shell		|指定其他命令使用shell 时的默认shell 类型. (可以使用转义字符)默认值[“/bin/sh＂,”c” | SHELL [”executable”,”parameters”]

 

使用.dockerignore文件

<img src="E:\Project\Textbook\linux云计算\assets\wps14-1682691150323-336.jpg" alt="img" style="zoom: 67%;" /> 

 

Docker build的具体使用

<img src="E:\Project\Textbook\linux云计算\assets\wps15-1682691150323-337.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps16-1682691150323-338.jpg" alt="img" style="zoom:67%;" /> 

容器操作

**(ctrl +p +q退出不中止)**

创建容器`docker create -it --name=a1 ubuntu:latest`	新建一个容器

启动容器`docker start a1`启动一个已经创建的容器

运行进入容器docker run			-i 以交互模式运行容器			-t为容器分配一个伪终端

-d后台运行容器，并返回容器ID	**--name**为容器指定一个名称		-u执行命令的用户名或ID	

--privileged指定容器为特权容器，特权容器拥有所有的capabilities		--rm容器在终止后会立刻删除

-m设置容器使用内存最大值		-u指定容器的用户				-w指定容器的工作目录

-e, --env=指定环境变量，容器中可以使用该环境变量

--entrypoint=""， 覆盖image的入口点

--restart=no，默认策略，在容器退出时不重启容器

on-failure，在容器非正常退出时（退出状态非0），才会重启容器

on-failure:3，在容器非正常退出时重启容器，最多重启3次

always，在容器退出时总是重启容器 

unless-stopped，容器退出时总是重启容器(不考虑在Docker守护进程启动前就停止的容器

docker run -d --restart=always ubuntu:latest ping www.docker.com

运行一个在后台不断执行的容器，同时带有命令，程序被终止后还能重启继续跑，还能用控制台管理

--cidfile运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法

**--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行

--env-file=[]， 指定环境变量文件，文件格式为每行一个环境变量

 

`docker run -it --name=a1 centos:7.6 /bin/bash`创建且进入容器a1以/bin/bash环境运行（默认退出中断）

`docker exec  -it a1 /bin/bash`进入容器

**过程创建容器过程：**

检查本地是否存在指定的镜像，不存在就从公有仓库下载

利用镜像创建一个容器，并启动该容器

分配一个文件系统给容器，并在只读的镜像层外面挂载一层可读写层

从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去

从网桥的地址池配置一个 IP 地址给容器

进入交互式终端

Exit退出后容器自动终止

查看容器docker ps查看正在运行的容器 		-a查看所有运行过的容器	-q只查看ID号  -n=1最后退出的容器

`docker top test`查看容器内进程					-a输出所有容器统计信息，默认仅在运行中

`docker info`查看docker服务运行详细信息		docker port ubunt查看容器的端口映射

`docker stats ubunt`查看容器占用硬件资源情况	docker logs ubunt查看容器日志信息(历史命令)

`docker logs --tail=5 9b53baf2246a`	#显示日志最后5行

`docker diff ubunt`查看容器内文件的变更信息

`docker inspect container/images -f {{.Size}}`	 #查看容器或镜像的详细信息	-f 过滤"key1=value"

`docker ps -qf status=exited/running/paused`	#查看状态停止容器 ID

`docker ps -f ancestor=nginx`	#根据镜像过滤

`docker stats --no-stream 49a877175264`	#查询 registry 容器的 CPU、内存等统计信息

停止删除容器`docker stop  {c1,c2}`停止多个容器		docker pause test暂停一个运行中的容器

`docker rm $(docker ps -qa)`删除所有容器				-v删除容器挂载的数据卷

－f强行终止并删除一个运行中的容器				-l删除容器的连接，保留容器

docker rm `docker ps -a|grep Exited|awk '{print $1}'`

`docker system prune -a`清理dangling镜像、退出的容器、无用的数据卷和网络、未使用的镜像[自动档]

导入导出容器docker export -o test.tar ubunt导出容器(不管是否运行 -o指定保存目录)

`docker export ubunt >test.tar`

`docker import test.tar liunx:1.0`导入容器

复制文件`docker cp test.sh ubunt:/ `在容器和主机之间复制文件		-a复制文件带有原始的uid/gid 信息

链接容器在两个互联的容器之间创建了一个安全隧道，而且不用映射它们的端口到宿主主机上

在启动db容器的时候没有使用 -p 和 -P 标记，从而避免了暴露数据库端口到外部网络上

`docker run -it --name c5 --link c1:c5  centos  bash`		通过查看hosts文件获取链接信息

`docker run -d -it  --name mysqldb  -P  192.168.2.5:5000/mysql/8.0  /bin/bash`

\#链接

6bdaa58b581e7b14ebd9cec3b0b9bb4688e4b3813df8a36dae271a039d7a4768

`docker run -d -it  --name nginxweb  -P --link mysqldb:db  192.168.2.5:5000/nginx/latest  /bin/bash`

\#链接数据库容器的db

b15a009ebcdca820c6f92de7c104f414fd9fc6def115094078eb500fb282557d

`docker inspect --format {{.HostConfig.Links}} b15a009ebcdc`

[/mysqldb:/nginxweb/db]

修改容器docker update修改容器配置信息

基于运行容器创建镜像docker commit 基于已有容器创建镜像		-a作者信息	-m描述信息

`docker commit -a "liu" -m "nginx"  cc  centos_nginx:1.0`以容器cc为基础创建centos_nginx镜像

基于本地模板导入`cat ubuntu-18.04-x86_64-minimal.tar.gz I docker import - ubuntu:18.04`

私有仓库

Docker官方的Docker hub（https://hub.docker.com）是一个用于管理公共镜像的仓库，我们可以从上面拉取镜像到本地，也可以把我们自己的镜像推送上去。但是，有时候我们的服务器无法访问互联网，或者你不希望将自己的镜像放到公网当中，那么我们就需要搭建自己的私有仓库来存储和管理自己的镜像

![img](E:\Project\Textbook\linux云计算\assets\wps17-1682691150323-339.jpg) 

<img src="E:\Project\Textbook\linux云计算\assets\wps18-1682691150323-340.jpg" alt="img" style="zoom:50%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps19-1682691150323-341.jpg" alt="img" style="zoom:50%;" /> 

 

<img src="E:\Project\Textbook\linux云计算\assets\wps20-1682691150323-342.jpg" alt="img" style="zoom: 67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps21-1682691150323-343.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps22-1682691150323-344.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps23-1682691150324-345.jpg" alt="img" style="zoom:67%;" /> 

**## Docker 私有仓库**	**### 一、私有仓库搭建**

1、拉取私有仓库镜像 	`docker pull registry`

2、启动私有仓库容器 	`docker run -id --name=registry -p 5000:5000 registry`

3、打开浏览器 http://私有仓库服务器ip:5000/v2/_catalog，看到{"repositories":[]} 表示私有仓库搭建成功

4、修改daemon.json  `vim /etc/docker/daemon.json`

在上述文件中添加一个key，保存退出。此步用于让 docker 信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip 	{"insecure-registries":["私有仓库服务器ip:5000"]} 

5、重启docker 服务 	systemctl restart docker		docker start registry

**### 二、将镜像上传至私有仓库**(服务开启状态)

1、标记镜像为私有仓库的镜像   docker tag centos:7 私有仓库服务器IP:5000/centos:7

2、上传标记的镜像   		docker push 私有仓库服务器IP:5000/centos:7

**### 三、 从私有仓库拉取镜像** 	docker pull 私有仓库服务器ip:5000/centos:7

数据卷

<img src="E:\Project\Textbook\linux云计算\assets\wps24-1682691150324-346.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps25-1682691150324-347.jpg" alt="img" style="zoom:50%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps26-1682691150324-348.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps27-1682691150324-349.jpg" alt="img" style="zoom:50%;" /> 

注：1、目录必须是绝对路径	2、容器内被映射目录没有则会创建，有且目录下有文件则会清空	3、可以挂载多个数据卷	

数据卷类型type

volume普通数据卷，默认映射到主机/var/lib/docker/volumes路径下

bind	绑定数据卷，映射到主机指定路径下

tmpfs	临时数据卷，只存在于内存中

-v, --volume=[]，给容器挂载存储卷，挂载到容器的某个目录

--volumes-from=[]，给容器挂载其他容器上的卷，挂载到容器的某个目录

普通数据卷	docker run -it --name=c1 -v /media:/media : ro -v /media:/sss centos:latest  bash

创建运行c1映射主机的/media以只读到容器的/media目录下

docker volume create test建立本地数据卷test

docker volume inspect（查看详细信息）ls(列出已有数据卷）prune（清理未使用数据卷）rm(删除数据卷）

绑定数据卷	src/source主机挂载点		target/destination/dst映射文件夹	ro/readonly指定数据卷只可读

docker run -id  --name c1 --mount type=bind,source=/media,target=/sss, readonly  ubuntu  bash

创建容器c1挂载类型bind将主机目录/media绑定到容器/sss以只读的形式

数据卷容器

<img src="E:\Project\Textbook\linux云计算\assets\wps28-1682691150324-350.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps29-1682691150324-351.jpg" alt="img" style="zoom:50%;" /> 

创建数据卷容器（默认映主机/var/lib/docker/volumes/）

`docker run -it --name=c1  -v :/sss centos:latest /bin/bash`创建c1数据卷容器

`docker run -it --name=c2 --volumes-from c1 centos:latest`容器c2映射/sss目录

使用--volumes-from参数所挂载数据卷的容器并不需要在运行状态

删除容器后，数据卷并不会消失， docker rm -v 可删除容器挂载的数据卷 

备份`docker run -it --name=c3 --volumes-from c1 -v $(pwd):/backup centos tar cvf /backup/backup.tar /sss`

创建c3将c1的数据卷/sss挂载到c3，再将主机当前目录映射到/backup,进入容器后将数据卷/sss打包成tar

恢复`docker run –name c4 --volumes-from c3 -v $(pwd) :/backup c3 tar xvf /backup/backup.tar`

创建c4，挂载c3数据卷到容器，映射主机当前目录到c4，使用tar解压backup.tar备份文件到当前c4目录

网络操作

bridge --net=bridge 指定，此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信

host --net=host 指定，容器不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口

none --net=none 指定，将容器放置在它自己的网络栈中，不进行任何配置。该模式关闭了容器的网络功能，在以下两种情况下是有用的：容器并不需要网络（例如只需要写磁盘卷的批处理任务）

container，创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围

创建docker network create -d bridge lll创建网络定义驱动类型为bridge

`docker network create --subnet=192.168.5.0/24 --ip-range=192.168.5.0/24 --gateway=192.168.5.1 xd_net`

\#创建subnet子网，ip地址范围为5网段，网关为5.1

`[root@server ~]#docker inspect -f '{{.State.Pid}}' ce3271024189`   #查找正在运行容器的pid号

1337

`[root@server ~]#mkdir /var/run/netns` 创建命名空间

`[root@server ~]#ln -s /proc/1337/ns/net  /var/run/netns/rancher-server`  #建立软链接

`[root@server ~]#ip netns list` #查看所有network namespace

rancher-server

`[root@server ~]#ip netns exec rancher-server ip a ` #查看容器内ip

查看docker network ls查看网络

`docker network inspect -f {{.IPAM.Config}} xd_net`	#查看网络内部信息

`docker inspect -f {{.NetworkSettings.Networks.xd_net.IPAddress}}  xd_net `

\#进行筛选，以搜索{{}}开始xd_net网络，下一级用点好隔开

断开`docker network disconnet test-network centos`移除网络之前需要断开所有连接到网络的容器

删除`docker network rm test-network`移除网络

连接`docker network connect test-network c1`将c1容器加入到test-network网络

`docker run -it --name c1 --network=lll centos bash`创建容器指定网络连接方式

--device=[]， 添加主机设备给容器，相当于设备直通

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致

报错：WARNING: IPv4 forwarding is disabled. Networking will not work

容器要想通过宿主机访问到外部网络，则需要宿主机进行辅助转发

开启ipv4转发`echo net.ipv4.ip_forward=1 >>/etc/sysctl.conf`

\# 重启network服务,查看是否修改成功		`systemctl restart network && sysctl -p`

`docker run -d -p 192.168.127.10::80/udp centos bash`

192.168.127.10映射地址随机映射端口到容器80端口udp协议

\#-P随机映射端口		-p指定映射端口		-h指定容器的主机名

`docker run -d -P --name web --link c1:c2 centos bash`建立互联关系

# docker应用部署

## Nginx和Apache

```sh
docker pull httpd #拉取镜像
docker run -it -p 8080:80 httpd /bin/bash #创建容器及映射端口
docker cp daff89e81c3c:/usr/local/apache2/conf/httpd.conf httpd.conf #复制配置文件
\#ServerName www.example.com:80 >> ServerName localhost:80 #修改文件

docker cp httpd.conf daff89e81c3c:/usr/local/apache2/conf/httpd.conf

bin/apachectl start #容器内启动服务
nginx命令启动		#nginx -c /etc/nginx/nginx.conf #指定配置文件启动
```



### nginx负载均衡

```sh
docker run -itdP -v /tmp/w1:/usr/local/tomcat/webapps/ROOT 192.168.2.5:5000/tomcat/latest
docker run -itdP -v /tmp/w2:/usr/local/tomcat/webapps/ROOT 192.168.2.5:5000/tomcat/latest
```

启动两台tomcat为后端服务器，启动nginx为负载均衡服务器

`docker run -it -p 80:80 -v /tmp/nginx:/nginx 192.168.2.5:5000/nginx/latest bash`

```sh
location / {	#/etc/nginx/nginx.conf文件
		proxy_pass http://192.168.2.5;	#指定http代理网址
#     root  /usr/share/nginx/html;
#     index  index.html index.htm;
}

upstream 192.168.2.5 {		#/etc/nginx/conf.d/default.conf文件
  server 192.168.2.5:32768 weight=1;		#需要增加权重，nginx有详细配置
  server 192.168.2.5:32769 weight=2;
}
```

## SSH

```sh
docker run -it --name c1  centos:7 bash创建容器
yum install  openssh-server openssh-clients -y安装所需软件包		ssh-keygen -A生成证书
passwd修改root密码		/usr/sbin/sshd -D & #启动sshd服务

docker commit  c1  sshd:centos #根据容器创建镜像
docker run -it --name c3 -p  5555:22  ssh:centos  bash使用ssh镜像映射端口进入容器
/usr/sbin/sshd -D & #启动sshd服务
```

## FTP

```sh
docker run -itd --name ftp --privileged  centos  /usr/sbin/init
--privileged获取宿主机root特殊权限
systemctl start vsftpd
```



## Mysql

 

## Tomcat

`docker run -id --name=c_tomcat -p 8080:8080 -v $PWD:/usr/local/tomcat/webapps/ROOT tomcat`

```shell
/usr/local/tomcat/webapps/ROOT：**将主机中当前目录挂载到容器的ROOT下
```

创建测试目录 mkdir text 		`echo “ni hao!!!” > index.jsp`

使用外部浏览器访问tomcat	http://192.168.100.132:8080

## Redis

3. 创建容器，设置端口映射		`docker run -id --name=c_redis -p 6379:6379 redis:5.0`
4. 使用外部机器连接redis		`./redis-cli.exe -h 192.168.149.135 -p 6379`

# registry私有仓库搭建

registry 用于保存docker 镜像，包括镜像的层次结构和元数据。

启动容器时，docker daemon会试图从本地获取相关的镜像；本地镜像不存在时，其将从registry中下载该镜像并保存到本地；

拉取镜像时，如果不知道registry仓库地址，默认从Docker Hub搜索拉取镜像

`docker run -itd -p 5000:5000 --restart=always registry`	#保证registry私有仓库自动重启

`docker tag docker.io/registry 192.168.2.7:5000/registry:v1`	#添加标签

`docker push 192.168.2.7:5000/registry:v1`		#上传镜像

docker 默认不允许http 方式推送镜像

```sh
{ "insecure-registries" : ["192.168.60.18:5000"] } >> /etc/docker/daemon.json
echo ADD_REGISTRY='--add-registry 192.168.2.7:5000' >>/etc/sysconfig/docker	#增添私有仓库
echo INSECURE_REGISTRY='--insecure-registry 192.168.2.10:5000' >>/etc/sysconfig/docker	#添加不安全仓库
```

验证：docker info命令输出最后一行有：

registries: master.example.com:5000 (insecure), docker.io (secure)

`curl http://192.168.2.7:5000/v2/_catalog	`	#查看registry私有仓库镜像

docker-compose服务编排

<img src="E:\Project\Textbook\linux云计算\assets\wps30-1682691150324-352.jpg" alt="img" style="zoom:67%;" /> 

![img](E:\Project\Textbook\linux云计算\assets\wps31-1682691150324-353.jpg) 

<img src="E:\Project\Textbook\linux云计算\assets\wps32-1682691150324-354.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps33-1682691150324-355.jpg" alt="img" style="zoom: 33%;" /> 

**## Docker Compose**安装使用

**### 一、安装Docker Compose**		**【需要梯子】**

\# Compose目前已经完全支持Linux、Mac OS和Windows，在我们安装Compose之前，需要先安装Docker。下面我 们以编译好的二进制包方式安装在Linux系统中。 

curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

*你可以也通过执行下面的命令，高速安装 Docker Compose。*v2.4.1版本号可替换

curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

\# 设置文件可执行权限 	chmod +x /usr/local/bin/docker-compose

\# 查看版本信息 		docker-compose -version

**### 二、卸载Docker Compose**

\# 二进制包方式安装的，删除二进制文件即可	rm /usr/local/bin/docker-compose

**### 三、 使用docker compose编排nginx+springboot项目**

1. 创建docker-compose目录		mkdir ~/docker-compose	cd ~/docker-compose
2. 编写 vim docker-compose.yml 文件

version: '3'

services:

 nginx:

 image: nginx	#指定镜像tag或者ID

 	restart: always #重启策略

 ports:	#映射端口号

\- 80:80

  links:

  \- app

 volumes:		#目录映射

  \- ./nginx/conf.d:/etc/nginx/conf.d

 app:				#第二个容器

 image: app

 expose:	#提供container之间的端口访问，不会暴露给主机使用

   \- "8080"	

## yaml文件关键字：

version  指定 compose 文件的版本

services 定义所有的 service 信息

image   指定为镜像名称或镜像 ID(如果镜像在本地不存在，Compose 将会尝试拉取这个镜像)

pid    跟主机系统共享进程命名空间。容器和宿主机系统之间可以通过进程 ID 来相互访问和操作

ports   暴露端口信息

command  覆盖容器启动后默认执行的命令：command: bundle exec thin -p 3000

build：  #指定 Dockerfile 所在文件夹的路径

 context: ./dir #路径

 dockerfile: Dockerfile-alternate #指定编排文件

 args:

  buildno: 1

restart	 指定容器退出后的重启策略为始终重启, 保持服务始终运行, 推荐配置为 always 或者 unless-stopped

depends_on:	#服务依赖容器，将会优先于服务创建并启动依赖，一般是mysql、redis等

expose  	暴露端口，但不映射到宿主机，只被连接的服务访问

extends:		#继承自当前yml文件或者其它文件中定义的服务，可以选择性的覆盖原有配置

 file: common.yml

 service: webapp		#service必须有，file可选。service是需要继承的服务，例如web、database

environment:		#添加环境变量。可以是数组或者字典格式

 \- RACK_ENV=development

 \- SESSION_SECRET

external_links:	#链接搭配docker-compose.yml文件或者Compose之外定义的服务，通常是提供共享或公共服务

 \- redis_1

 \- project_db_1:mysql	#注意：external_links链接的服务与当前服务必须是同一个网络环境

environment    设置环境变量。可使用数组或字典两种格式

depends_on    解决容器的依赖、启动先后的问题(服务不会等待依赖容器完全启动之后才启动)

volumes:			 #数据卷所挂载路径设置, 可以设置为宿主机路径或者数据卷名称

 \- /var/lib/mysql

 \- cache/:/tmp/cache

 \- ~/configs:/etc/configs/:ro

volumes_from:	#挂载数据卷容器，挂载是容器。container:container_name格式仅支持version 2

 \- service_name

 \- service_name:ro

 \- container:container_name:rw

links:			#链接到其他服务中的容器，别名将自动会在容器的/etc/hosts文件里创建相应记录

 \- db

 \- db:mysql

 \- redis

external_links  #链接到docker-compose.yml外部的容器，甚至不是Compose管理的容器

extra_hosts:		#添加主机名映射

 \- "somehost:162.242.195.82"

 \- "otherhost:50.31.209.229"

将会在/etc/hosts创建记录：

162.242.195.82  somehost

50.31.209.229  otherhost

cap_add  让容器拥有内核的某项能力

cap_drop 去掉容器内核的某项能力

cgroup_parent   指定父 cgroup 组，意味着将继承该组的资源限制

container_name  指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式

devices  指定设备映射关系

dns:		#自定义 DNS 服务器。可以是一个值，也可以是一个列表

 \- 8.8.8.8

 \- 9.9.9.9

dns_search    配置 DNS 搜索域。可以是一个值，也可以是一个列表

tmpfs   挂载一个 tmpfs文件系统到容器

env_file 从文件中获取环境变量, 可为单独的文件路径或列表(如有变量名称与environment指令冲突则>以后者为准)

extra_hosts    类似 Docker 中的 --add-host 参数，指定额外的 host 名称映射信息

healthcheck    通过命令检查容器是否健康运行

logging  配置日志选项, 目前支持三种日志驱动类型(json-file、syslog和none)

network  设置网络模式

net: "bridge"

net: "none"

net: "container:[name or id]"

net: "host"

networks 配置容器连接的网络

secrets  存储敏感数据，例如 mysql 服务密码

security_opt   指定容器模板标签（label）机制的默认属性（用户、角色、类型、级别等）

stop_signal    设置另一个信号来停止容器。在默认情况下使用的是 SIGTERM 停止容器

working_dir    指定容器中工作目录

# docker-compose基本操作：

docker-compose来管理harbor。注意必须切换到docker-compose.yml同级目录下运行一下命令

docker-compose stop/start/restart/up #停止/启动/重启/启动harbor

选项：-h 帮助					-d 在后台运行服务容器

-f --file FILE指定Compose模板文件，默认为docker-compose.yml

-p --project-name NAME 指定项目名称，默认使用当前所在目录为项目名

-v，-version 打印版本并退出		--verbose  输出更多调试信息

--log-level LEVEL 定义日志等级(DEBUG, INFO, WARNING, ERROR, CRITICAL)

-no-color 不是有颜色来区分不同的服务的控制输出

-no-deps 不启动服务所链接的容器

--force-recreate 强制重新创建容器，不能与-no-recreate同时使用

–no-recreate 如果容器已经存在，则不重新创建，不能与–force-recreate同时使用

–no-build 不自动构建缺失的服务镜像

–build 在启动容器前构建服务镜像

–abort-on-container-exit 停止所有容器，如果任何一个容器被停止，不能与-d同时使用

-t, –timeout TIMEOUT 停止容器时候的超时（默认为10秒）

–remove-orphans 删除服务中没有在compose文件中定义的容器

子命令：

ps：列出所有运行容器 #docker-compose ps

logs：查看服务日志输出	#docker-compose logs

port：打印绑定的公共端口，下面命令可以输出 eureka 服务 8761 端口所绑定的公共端口

docker-compose port eureka 8761

build：构建或者重新构建服务 #docker-compose build

start：启动指定服务已存在的容器 #docker-compose start eureka

stop：停止已运行的服务的容器 #docker-compose stop eureka

rm：删除指定服务的容器 #docker-compose rm eureka

up：构建、启动容器 #docker-compose up

kill：通过发送 SIGKILL 信号来停止指定服务的容器 #docker-compose kill eureka

pull：下载服务镜像

scale：设置指定服务运气容器的个数，以 service=num 形式指定docker-compose scale user=3 movie=3

run：在一个服务上执行一个命令docker-compose run web bash

## dcoker-compose案例

version: "3.3"

services:

 wordpress:

image: 10.0.0.3/library/wordpress:latest

depends_on:

\- db

ports:

\- 80

restart: always

environment:

WORDPRESS_DB_HOST: db:3306

WORDPRESS_DB_USER: wordpress

WORDPRESS_DB_NAME: wordpress

WORDPRESS_DB_PASSWORD: wordpress

 db:

image: 10.0.0.3/library/mysql:5.6

restart: always

environment:

MYSQL_ROOT_PASSWORD: wordpress#必须初始化

MYSQL_DATABASE: wordpress

MYSQL_USER: wordpress

MYSQL_PASSWORD: wordpress

volumes:

db_data: {} 

# docker hub官方仓库登录

docker login 

docker push 推送

# Harbor（企业级 Registry 服务）

Harbor是由VMware公司开源的企业级的Docker Registry管理项目，主要提供管理UI。Harbor的目标是帮助用户迅速搭建一个企业级的Docker registry服务。以Docker公司开源的registry为基础

**特性**：

基于角色的访问控制 ：用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限

**镜像复制** ： 镜像可以在多个Registry实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景

**图形化用户界面** ： 用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间

**AD/LDAP 支持** ： Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理

审计管理 ： 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理

**国际化** ： 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来

RESTful API ： RESTful API 提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易

**部署简单** ： 提供在线和离线两种安装工具， 也可以安装到vSphere平台(OVA方式)虚拟设备

Harbor的每个组件都是以Docker容器的形式构建的。用于部署Harbor的Docker Compose模板位于 harbor/docker-compose.yml,打开这个模板文件，发现Harbor是由7个容器组成的；

nginx：nginx负责流量转发和安全验证，对外提供的流量都是从nginx中转，所以开放https的443端口，它将流量分发到后端的ui和正在docker镜像存储的docker registry。

harbor-jobservice：harbor-jobservice 是harbor的job管理模块，job在harbor里面主要是为了镜像仓库之前同步使用的;

harbor-ui：harbor-ui是web管理页面，主要是前端的页面和后端CURD的接口;

registry：registry就是docker原生的仓库，负责保存镜像。

harbor-adminserver：harbor-adminserver是harbor系统管理接口，可以修改系统配置以及获取系统信息。

harbor-db：harbor-db是harbor的数据库，这里保存了系统的job以及项目、人员权限管理。由于本harbor的认证也是通过数据，在生产环节大多对接到企业的ldap中；

harbor-log：harbor-log是harbor的日志服务，统一管理harbor的日志。通过inspect可以看出容器统一将日志输出的syslog。

这几个容器通过Docker link的形式连接在一起，这样，在容器之间可以通过容器名字互相访问。对终端用户而言，只需要暴露proxy （即Nginx）的服务端口。

上传所需docker、docker-compose、harbor文件

chmod +x  /usr/local/bin/docker-compose #移动到/usr目录下设置为可执行文件，并执行

docker-compose -v #查看版本号

tar -xvf harbor-offline-installer-v1.5.3.tgz #进行解压

harbor.cfg是Harbor的配置文件

hostname = 192.168.2.7 #修改此为服务器IP，访问用户界面和register服务

 

\# ./prepare #启动harbor

\# ./install.sh--with-clair #进行安装

docker-compose ps #查看harbor启动容器

进入浏览器登陆http://http://192.168.2.7，默认的用户名 admin，密码 Harbor12345

认证安装后会自动创建一个名称为library的项目，且访问级别为公开(即任何人都可以下载该项目中的镜像)

docker重启harbor也需重启

```sh
docker login/logout 127.0.0.1 #命令行登录/退出harbor：本地通过127.0.0.1登录和推送镜像
docker pull nginx #拉取镜像
docker tag nginx 仓库服务器地址/项目名/服务名：版本号
docker tag nginx 127.0.0.1/project/nginx:v1 #镜像打上标签
docker images #查看镜像
docker push 127.0.0.1/project/nginx:v1 #将打上标签的镜像上传
```

<img src="E:\Project\Textbook\linux云计算\assets\wps34-1682691150324-356.jpg" alt="img" style="zoom:50%;" /> 

（上面的操作都是harbor服务器的本地操作，下面做的是从其他客户端上传镜像到harbor ）

登录报错原因:dockerregistry交互默认使用https服务，但是搭建私有镜像默认使用http服务。未在docker启动文件中添加--insecure-registry信任关系！因为你没有添加信任关系的话，docker默认使用的是https协议，所以端口不对(443)，会报连接拒绝这个错误； 或者提示你 "服务器给HTTPS端的是HTTP响应" 这个错误，因为你没添加端口信任，服务器认为这是默认的https访问，返回的却是http数据！

vi /etc/docker/daemon.json #编辑加速器

```sh
{
  "registry-mirrors": ["https://k4i1aeje.mirror.aliyuncs.com","https://docker.mirrors.ustc.edu.cn"],"insecure-registries": ["localhost"]
}
```

harbor支持http和https，但如果使用http的话，在拉取镜像的时候，会抛出仓库不受信任的异常

需要在所有的docker客户端的docker配置文件/etc/docker/daemon.json中添加如下配置:

```sh
{
  "insecure-registries": ["https://*.*.*.*"]
}
```

添加用户，添加项目管理用户

**Harbor****中的用户有****3****种角色：项目管理员（****MDRWS****）、开发人员（****RWS****）和访客（****RS****），当然还有一个最高管理员权限****admin****系统管理员**

<img src="E:\Project\Textbook\linux云计算\assets\wps35-1682691150324-357.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps36-1682691150325-358.jpg" alt="img" style="zoom:67%;" /> 

# rancher容器管理平台

帮助组织在生产环境中轻松快捷的部署和管理容器。Rancher可以轻松地管理各种环境的Kubernetes，满足IT需求并为DevOps团队提供支持

Rancher支持各类集中式身份验证系统来管理Kubernetes集群

Rancher为DevOps工程师提供了一个直观的用户界面来管理他们的服务容器，用户不需要深入了解Kubernetes概念就可以开始使用Rancher

docker run -itd --restart=always -p 8080:8080 192.168.2.5:5000/rancher/server:v1.6.5	#从私有仓库

Gogs（极易搭建的自助 Git 服务，默认8080）、Elasticsearch（基于Lucene的搜索服务器，默认80）

Prometheus （开源的服务监控系统和时间序列数据库）、Grafana（可视化监控信息工具）

 

 

# swarm集群(支持多容器操作的利器)

docker swarm是运维人员便捷操作多个容器的编排工具，利用每个主机的docker engine集合成一个虚拟的docker资源池，采用最典型的主从结构（即通过manages对worker进行操作）中内置了基于DNS的负载均衡和对外部负载均衡机制的集成支持,通过Raft协议管理多个节点

节点：运行docker engine引擎的主机且加入swarm集群中

管理节点负责外部对集群的请求操作，维持集群资源，分发任务，自身也是工作节点

工作节点负责执行具体任务

端口号port 2377/ TCP集群管理通信						7946节点通信TCP/UDP

4789/UDP覆盖型网络Pover1ay驱动

<img src="E:\Project\Textbook\linux云计算\assets\wps37-1682691150325-359.jpg" alt="img" style="zoom:67%;" /> 

创建集群	docker	swarm init在管理节点上创建一个集群

--advertise-addr指定swarem服务监昕的地址和端口	docker swarm init --advertise-addr 192.168.100.139

--autolock自动锁定管理服务的启停操作，对服务进行启动或停止都需要通过口令来解锁

--availability节点的可用性，包括 active 、 pause、 drain 三种，默认为 active

--cert-expiry根证书的过期时长，默认为90天

--data-path-port指定数据流量使用的网络接口或地址

--dispatcher-heartbeat分配组件的心跳时长，默认为 5 秒

--external-ca指定使用外部的证书签名服务地址

--force-new-cluster强制创建新集群

--max-snapshots协议Raft快照保留的个数

--snapshot-interval协议Raft进行快照的间隔（单位为事务个数）,默认为10000个事物

--task-history-limit任务历史的保留个数，默认为5

查看集群信息	docker info

节点操作	docker node list列出集群中的节点信息

docker node promote 命令来提升一个工作节点为管理节点

docker node demote 命令来将一个管理节点降级为工作节点

加入离开集群	docker swarm join

--token xxx:向指定集群中加入工作节点的认证信息（xxx返回的token串是集群唯一id）

docker swarm join --token xxx 192.168.100.138: 2377 (worker2)

swarm update更新一个Swarm 集群

swarm leave离开一个 Swarm 集群					-f意味着强制离开集群

更新集群		docker swarm update

-autolock启动或关闭自动锁定

-cert-expiry duration根证书的过期时长，默认为90天

-dispatcher-heartbeat duration分配组件的心跳时长，默认为5秒

-external-ca external-ca指定使用外部的证书签名服务地址

-max- snapshots uint : Ra玩协议快照保留的个数

-snapshot-interval uint : Raft 协议进行快照的间隔（单位为事务个数），默认为10000个事物

-task-history-limit int任务历史的保留个数， 默认为5

使用服务操作docker service

create	创建应用

-e环境变量列表

-mode string服务模式

-u指定用户信息,UID:GID

-config config:指定暴露给服务的配置

-constraint list:应用实例在集群中被放置时的位置限制

-dns自定义使用的DNS服务器地址

-endpoint-mode string:指定外部访问的模式,包括vip(虚地址自动负载均衡)或dnsr(DNS轮询)

-workdir string:指定容器中的工作目录位置

ls		列出服务的信息

-f只输出符合过滤条件的服务			-q只输出服务的ID信息

ps 		列出服务中包括的任务信息		docker service ps helloworld

-f只输出符合过滤条件的任务

-no-resolve:不将IDs映射为名称

-no- trunc:不截断输出信息

-quiet:只输出服务的ID信息。

rm 		删除服务

inspect 查看应用的详细信息			-f使用Go板指定格式化输出		--pretty仅显示重要信息

rollback 回滚服务的配置

-q不显示执行进度信息					-d执行后返回，不等待服务状态校验完整

logs 	获取服务或任务的日志信息

-details输出所有的细节日志信息

-f持续跟随输出

-no-resolve在输出中不将对象的ID映射为名称

-no-task-ids输出中不包括任务的ID信息

-no trunc不截断输出信息

-raw输出原始格式信息

-since输出自指定时间开始的日志，如2018-01-02T03:04:56或42m

-tail只输出给定行数的最新日志信息

-t打印日志的时间

scale 	对服务副本调整			docker service scale helloworld=3

update 更新服务

<img src="E:\Project\Textbook\linux云计算\assets\wps38-1682691150325-360.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps39-1682691150325-361.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps40-1682691150325-362.jpg" alt="img" style="zoom:67%;" /> 

初始化第一个管理节点 -> 加入额外的管理节点 -> 加入工作节点 -> 完成

操作修改/etc/hosts文件	master		worker1		wordker2

## 1创建

docker swarm		关闭防火墙

在 manager.1机器上创建 docker swarm集群

docker swarm init --advertise-addr 192.168.100.139 

advertise-addr将该Ip地址的机器设置为集群管理节点（如果是单节点,无需该参数)

## 2查看管理节点集群信息

docker node ls	---生成ingress覆盖式网络

向 docker swarm中添加工作节点:在两个工作节点中分别执行如下命令,ip地址是 manager节点的

添加两个vrk节点

`docker swarm join --token xxx 192.168.100.138: 2377(worker1)`

`docker swarm join --token xxx 192.168.100.138: 2377 (worker2)`

--token xxx:向指定集群中加入工作节点的认证信息（xxx认证信息是在创建docker swarm时产生的）

查看管理节点集群信息与之前的区别		docker node ls

## 3在 docker swarma中部署服务

在 Docker Swarm集群中部署服务时,既可以使用 Docker Hub上自带的像来启动服务,也可以使用Dockerfile构建的镜像来启动服务。使用自己通过 Dockerfilet构建的镜像来启动服务那么必须先将镜像推送到 Docker Hub中心仓库,为了方便读者的学习,这里以使用 Docker Hub上自带的a1pine镜像为例来部集群服务

## 4部署服务

`docker service create --replicas 1 --name helloworld alpine ping docer.com`

docker service create指令:用于在 Swarm集群中创違一个基于 alpine镜像的服务

--replicast参数:指定了该服务只有一个副本实例（只有一个节点去执行）

--name参数:指定创建成功后的服务名称为hel1oword

ping docker.com指令:表示服务启动后执行的命令

## 5查看 docker swarm集群中的服务

`docker service ls`查看服务列表:

`docker service inspect`查看部署具体服务的详细信息

服务名称查看服务在集群节点上的分配以及运行情兄: `docker service ps`服务名称

## 6修改副本数量

在 manager1上,更改服务副本的数量(创建的副本会随机分配到不同的节点

`docker service scale helloworld=5`

## 7删除服务(在管理节点)

docker service rm服务名称

## 8访问服务

查看集群环境下的网络列表: docker network ls

在manager1上创建- -overlay为驱动的网络(默认使用的网络连接 ingress)

`docker network create -d=overlay my-network`

在集群管理节点 manager1上部署一个 nginx服务

`docker service create --network my-network --name my-web -p 8080:80 --replicas 2  nginx`

# 容器限制块设备I/O速率

创建容器，对容器进行数据写入速度测试，及限制为通过修改相应的Cgroup文件来限制写磁盘的速度为1024000字节

```sh
[root@46214e213d1a /]# dd if=/dev/zero of=testfile0 bs=8k count=5000 oflag=direct	#dd进行测试
5000+0 records in
5000+0 records out
40960000 bytes (41 MB) copied, 0.731665 s, 56.0 MB/s

[root@46214e213d1a /]# df -h	#容器内查看
Filesystem                                             Size  Used Avail Use% Mounted on
/dev/mapper/docker-253:0-420471-e9a8dc0ebddd4375ed80ae215da42e330ebd21580d1bc90ce46484deb8abfa7a  10G  250M  9.8G  3% /

[root@localhost ~]# ll /dev/mapper/	#查看这个创建时间
lrwxrwxrwx 1 root root    7 Nov 29 04:54 docker-253:0-420471-e9a8dc0ebddd4375ed80ae215da42e330ebd21580d1bc90ce46484deb8abfa7a -> ../dm-5

[root@localhost ~]# ll /dev/dm-5	查看这个设备的编码(253,5)
brw-rw---- 1 root disk 253, 5 Nov 29 04:54 /dev/dm-5

[root@localhost ~]# echo "253:5 1024000" >/sys/fs/cgroup/blkio/system.slice/docker-46214e213d1a1a182e0cee3a1c8e6d13634d7cb11ccf5375cf574b3dae34ac66.scope/blkio.throttle.write_bps_device	#输入数据

[root@46214e213d1a /]# dd if=/dev/zero of=testfile0 bs=8k count=5000 oflag=direct	#再次测试
```



# 容器硬盘热扩容

Docker容器动态扩展的优点：

1）不需要修改docker配置，不需要重启docker服务；

2）可以直接对运行中的容器进行动态扩展（只能增，无法缩）；

Docker容器动态扩展的条件：

1）docker所在宿主机分区的格式必须是ext2、ext3、ext4；

2）docker存储引擎必须是devicemapper

使用dmsetup查看该文件扇区信息.下面命令结果中的第二个数字（即20971520）是设备的大小，表示有多少个 512－bytes 的扇区. 这个值略高于 10GB 的大小。

```sh
[root@localhost ~]# dmsetup table /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3

0 20971520 thin 253:0 13
```

 

计算20G所需扇区数目

```sh
[root@localhost ~]# echo $((20*1024*1024*1024/512))
41943040
```

精简快照目标的一个神奇的特点是它不会限制卷的大小。当创建它的时候，一个精简的卷使用0个块，当开始往块里面写入的时候，它们会从共用的块池中进行分配。

可以写0个块，或者是10亿个块，这个和精简快照目标没关系。文件系统的大小只和Device Mapper表有关系。

只需要装载一个新的表，这个完全和之前的是一样的，但是有更多的扇区。仅此而已。

将新的扇区大小写入，注意只是改变旧表中的第二个数字20971520的数字，其他数字不变！

```sh
[root@localhost ~]# echo 0 41943040 thin 253:0 13 | dmsetup load /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3
```

 

将修改后的容器存储文件激活

```sh
[root@localhost ~]# dmsetup resume /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3
```

 

重新查看文件信息

```sh
[root@localhost ~]# dmsetup table /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3

0 41943040 thin 253:0 13
```

 

更改文件系统大小，使变更生。

```sh
[root@localhost ~]# resize2fs /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3

resize2fs 1.41.12 (17-May-2010)

Filesystem at /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3 is mounted on /var/lib/docker/devicemapper/mnt/813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3; on-line resizing required

old desc_blocks = 1, new_desc_blocks = 2

Performing an on-line resize of /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3 to 5242880 (4k) blocks.

The filesystem on /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3 is now 5242880 blocks long.
```

\-----------------------------------------------------------------------------------------------------------

如果这一步出现下面报错：

resize2fs 1.42.9 (28-Dec-2013)

resize2fs: 设备或资源忙 当尝试打开 /dev/mapper/docker-253:0-268868570-2163383f446357876b301fb3942b706436b5eea111c06a3acba0006ec5272372 时找不到有效的文件系统超级块.

原因是resize2fs仅能支持ext2、ext3、ext4，不支持xfs。将docker服务器的文件系统格式调整为ext4即可。

本文操作机是centos6系统，分区都是ext4格式，故不会出现这个报错

\-----------------------------------------------------------------------------------------------------------

再次登录my-test容器，发现容器大小已经更新为20G！

```sh
[root@localhost ~]# docker exec -ti my-test /bin/bash
[root@813389572d7f /]# df -hT
Filesystem                                            Type  Size  Used Avail Use% Mounted on
/dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3 ext4   20G  708M  18G  4% /
tmpfs                                              tmpfs  32G   0  32G  0% /dev
shm                                               tmpfs  64M   0  64M  0% /dev/shm
/dev/sda3                                            ext4  193G  103G  80G  57% /etc/hosts
```

扩容后可能出现的问题：停止该容器后，无法重新启动-

当容器扩容之后，由于dm认为设备块大小仍然为之前设置的初始大小，所以会发生无法起启动的情况，这时只要重新操作即可。

1)必须要先启动一下，让其生成dm文件才能修改

```sh
[root@localhost ~]# docker start my-test
```

此时会报错，不要理会，执行以下操作即可

```sh
[root@localhost ~]# echo 0 41943040 thin 253:3 725 | dmsetup load /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3

[root@localhost ~]# dmsetup resume /dev/mapper/docker-8:3-4850707-813389572d7f569e7b3705070033b43cf9e42ed9d304e03662c92533838ddec3
```

 

```sh
#!/bin/bash
#This script is dynamic modify docker container disk容器动态扩容脚本
if [ -z $1 ] || [ -z $2 ]; then
  echo "Usage: container_name increase_capacity"
  echo "Example: I want increase 11G to test"
  echo "The command is:  sh `basename $0` test 11"
  exit 1
fi
if [ `docker inspect $1 &>>/dev/null &&  echo 0 || echo 1` -eq 1 ];then
  echo "The container $1 is no exist!"
  exit 1
fi
container_id=`docker inspect -f '{{ .Id }}' $1`
now_disk=`dmsetup table /dev/mapper/docker-*-$container_id|awk '{print $2}'`
disk=$(($2*1024*1024*1024/512))
if [ $disk -lt $now_disk ];then
  echo "I can't shink container $1 from $(($now_disk*512/1024/1024/1024))G to ${2}G!I only modify contanier increase disk!"
  exit 1
fi
dmsetup table /dev/mapper/docker-*-$container_id|sed "s/0 [0-9]* thin/0 $disk thin/"|dmsetup load /dev/mapper/docker-*-$container_id
dmsetup resume /dev/mapper/docker-*-$container_id
resize2fs /dev/mapper/docker-*-$container_id
if [ $? -eq 0 ];then
  echo "dynamic container $1 disk to ${2}G is success!"
else
  echo "dynamic container $1 disk to ${2}G is fail!"
fi
```

容器日志

容器日志几乎可以保存所有的操作记录。rsyslog可以将所有服务器上的Docker日志收集汇总到一台服务器上统一管理

包：rsyslog				端口port：514

```sh
vim /etc/rsyslog.conf			#编辑主配置文件

# Provides UDP syslog reception	#启用UDP端口
$ModLoad imudp
$UDPServerRun 514

# Provides TCP syslog reception	#启用TCP端口
$ModLoad imtcp
$InputTCPServerRun 514
```

 



可视化工具

Docker 	UI只能连接当前服务器，安全度低

`docker run -d -p 9000:9000 —privileged -v /var/run/docker.sock:/var/run/docker.sock	ifd/ui-for-docker`访问端口即可

portainer目前比较流行，

`docker run -d -p 9001:9000 —privileged -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer`访问端口即可

 

Namespace技术

 

Namespace是Linux内核的一-项功能，该功能对内核资源进行分区，以使一-组进程看到一组资源，而另一组进程看到另一-组资源。Namespace 的工作方式通过为一组资源和进程设置相同的Namespace而起作用，但是这些Namespace引用了不同的资源。资源可能存在于多个Namespace中。这些资源可以是进程ID、主机名、用户ID、文件名、与网络访问相关的名称和进程间通信

 

https://blog.csdn.net/butterfly5211314/article/details/122753954

 

Cgroup技术

 

 

 

 

 

 

 

 