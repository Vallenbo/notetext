# Prometheus普罗米修斯监控系统

Prometheus是开源系统监视和警报工具包

Prometheus的主要特点是：一个多维数据模型，其中包含通过度量标准名称和键/值对标识的时间序列数据

PromQL，一种灵活的查询语言，可以利用多维数据完成复杂的查询

不依赖分布式存储；单服务器节点是自治的

时间序列收集通过HTTP上的拉模型进行

通过中间网关支持推送时间序列

通过服务发现或静态配置发现目标

多种图形和仪表板支持模式

组件：Prometheus生态系统包含多个组件，其中许多是可选的：

Prometheus主服务器，它会刮取并存储时间序列数据

客户端库，用于检测应用程序代码

一个支持短期工作的推送网关

诸如HAProxy，StatsD，Graphite等服务的专用出口商

一个alertmanager处理警报

各种支持工具

大多数Prometheus组件都是用Go编写的，因此易于构建和部署为静态二进制文件

<img src="E:\Project\Textbook\linux云计算\assets\wps1-1682690463420-251.jpg" alt="img" style="zoom: 50%;" /> 

## 工作流程是：

Prometheus server 定期从配置好的 jobs 或者 exporters 中拉 metrics，或者接收来自Pushgateway 发过来的 metrics，或者从其他的 Prometheus server 中拉 metrics。

Prometheus server 在本地存储收集到的 metrics，并运行已定义好的 alert.rules，记录新的时间序列或者向 Alertmanager 推送警报。

Alertmanager 根据配置文件，对接收到的警报进行处理，发出告警。

在图形界面中，可视化采集数据。

传统的监控方式分为push和pull方式，prometheus支持默认的pull模式获取数据，这也是官方推荐的方式，但如果因为一些网络或防火墙等原因无法直接pull到数据的情况，就要借助Pushgateway让Prometheus转换为push方式获取数据。

总结：Prometheus 属于一站式监控告警平台，依赖少，功能齐全

Prometheus 支持对云或容器的监控，其他系统主要对主机监控

Prometheus 数据查询语句表现力更强大，内置更强大的统计函数

Prometheus 在数据存储扩展性以及持久性上没有 InfluxDB，OpenTSDB，Sensu 好

## prometheus服务端软件安装：

[prometheus下载地址](https://github.com/prometheus/prometheus/releases/download/v2.29.0-rc.1/prometheus-2.29.0-rc.1.linux-amd64.tar.gz)

```sh
tar -xzvf prometheus-2.29.0-rc.1.linux-amd64.tar.gz	#解压
cp promtool prometheus /usr/local/sbin/		#复制文件至启动文件夹
promtool check config prometheus.yml	#检查配置文件是否有问题
prometheus --config.file="prometheus.yml"	#启动服务指定配置文件，默认监听9090
-- storage.tsdb.path #指定数据文件存储的位置		--web.enable-lifecycle #支持热更新
```

docker容器安装启用：

```sh
docker run -d -p 9090:9090 -v /tmp/prometheus.yml:/prometheus/prometheus.yml prom/prometheus
```

global:

 scrape_interval:   15s #多久刷新一次目标。您可以为单个目标覆盖此目标。在这种情况下，全局设置是每15秒刷新一次

 evaluation_interval: 15s #多久评估一次规则。Prometheus使用规则来创建新的时间序列并生成警报

 

rule_files:	#指定加载的任何规则的位置

/# - "first.rules"

/#- "second.rules"

 

scrape_configs: #控制Prometheus监视哪些资源

 \- job_name: prometheus #job任务名称，可定义多个

  static_configs: #静态服务发现

   \- targets: ['localhost:9090'] #定义节点地址：收集信息端口号

 

  #honor_labels： #用于解决拉取数据标签有冲突，当设置为 true, 以拉取数据为准，否则以服务配置为准

  #params：#数据拉取访问时带的请求参数

  #scrape_timeout:  #拉取超时时间

  #metrics_path： #拉取节点的 metric 路径

  #scheme： #拉取数据访问协议

  #sample_limit： #存储的数据标签个数限制，如果超过限制，该数据将被忽略，不入存储；默认值为0，表示没有限制

  #relabel_configs： #拉取数据重置标签配置

  #metric_relabel_configs：metric #重置标签配置

## 服务发现：

  #dns_sd_configs: DNS 服务发现

  #file_sd_configs: 文件服务发现

  #consul_sd_configs: Consul 服务发现

  #serverset_sd_configs: Serverset 服务发现

  #nerve_sd_configs: Nerve 服务发现

  #marathon_sd_configs: Marathon 服务发现

  #kubernetes_sd_configs: Kubernetes 服务发现

  #gce_sd_configs: GCE 服务发现

  #ec2_sd_configs: EC2 服务发现

  #openstack_sd_configs: OpenStack 服务发现

  #azure_sd_configs: Azure 服务发现

  #triton_sd_configs: Triton 服务发现

## node_exporter监控节点：

Prometheus 监控模型: 主动抓取目标的指标接口(HTTP 协议)获取监控指标, 再存储到本地或远端的时序数据库,且对于指标接口有一套固定的格式要求

格式大致如下:

/# HELP http_requests_total The total number of HTTP requests.

/# TYPE http_requests_total counter

http_requests_total{method="post",code="200"} 1027

http_requests_total{method="post",code="400"}   3 

<img src="E:\Project\Textbook\linux云计算\assets\wps2-1682690463420-252.jpg" alt="img" style="zoom:67%;" /> 

[node_exporter下载地址](https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz)

cp /node_exporter/node_exporter /usr/local/sbin/ #命令拷贝至全局

node_exporter -h	#帮助命令，也可查看控制器打开状态

1、配置文本文件收集器：

mkdir -p  /var/lib/node_exporter/textfile_collector	#创建收集目录

echo 'metadata{role="docker_server",datacenter="NJ"}1' |tee metadata.prom

--collector.textfile.directory=""	#指定文本收集器目录

2、启用system收集器：	--collector.systemd（默认关闭）

--collector.systemd.unit-include=".+"

http://localhost:9100/metrics	#可以查看各项指标

## Pushgateway推送工具

Prometheus 采用 pull 模式，将不同数据汇总, 由 Prometheus 统一收集

弊端：将多个节点数据汇总到 pushgateway, 如果 pushgateway 挂了，受影响比多个 target 大

Prometheus 拉取状态 up 只对 pushgateway插件节点,

Pushgateway 可以持久化推送给它的所有监控数据，

因此，即使你的监控已经下线，prometheus 还会拉取到旧的监控数据，需要手动清理 pushgateway 不要的数据。 

下载地址：https://github.com/prometheus/pushgateway/releases/download/v1.4.1/pushgateway-1.4.1.linux-amd64.tar.gz

./pushgateway #启动，默认监听端口9091

JDK部署安装

JDK（Java Development Kit）是面向对象程序设计语言的开发工具包，拥有这个工具包之后我们就可以使用Java语言进行程序设计和开发

mkdir /usr/lib/jdk	#建立jdk解压文件夹

 wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm

#在线下载jdk8

rpm -ivh jdk-8u131-linux-x64.rpm #安装

tar -xzvf jdk-14.0.1_linux-x64_bin.tar.gz -C /usr/java/jdk1.8.0_131	#进行解压

cd /usr/java/jdk1.8.0_131	#此目录包含了所有和Java运行环境相关的东西

JDK需要这样几个环境变量：

JAVA_HOME ：Java的主目录，把压缩包包解压之后得到的jdk-14文件夹所在的位置（并且包含jdk-14自身）

JRE_HOME：JRE的主目录，JRE是运行Java应用程序的最基本软件环境，所以如果你只是希望Java的程序能够运行的的话你完全不需要安装JDK，尽管JDK里面带有JRE。

CLASSPATH：Java提供的标准或公共类库的位置

PATH：这是系统的环境变量，这个东西只是告知系统你的Java开发环境被安装在了什么位置，这个东西使你在任意目录下都可以直接执行Java的开发工具比如javac等，直接进入javac就可以执行而不需要再重新进入入/usr/lib/jvm/jdk-13/bin/javac

vim /etc/profile			#配置环境

```sh
export JAVA_HOME=/usr/lib/jdk/jdk-14.0.1
export JRE_HOME=/${JAVA_HOME}
export CLASSPATH=.:${JAVA_HOME}/libss:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

```
[root@localhost jdk-14.0.1]# java -version	#查看java版本号
[root@localhost jdk-14.0.1]# javac
[user@localhost ~/jsrc]$ vim Hello.java		#Java程序测试

public class Hello {// Hello.java
  public static void main(String args[]){
	System.out.println("Hello");
  }
}
```

openjdk：是开源社区开发的开源实现，yum list | grep openjdk查看jdk版本号

`yum install java-1.8.0 -y` #进行安装

# NoSQL数据库

NoSQL（Not Only SQL），指的是非关系型的数据库。是对不同于传统的关系型数据库的数据库管理系统的统称。NoSQL用于超大规模数据的存储。这些类型的数据存储不需要固定的模式，无需多余操作就可以横向扩展

1. **键值数据库**		**2. 列式数据库**		**3. 文档数据库**		**4. 图形数据库**

**1. CouchDB，**所用语言： Erlang。特点：DB一致性，易于使用

**最佳应用场景：**适用于数据变化较少，执行预定义查询，进行数据统计的应用程序。适用于需要提供数据版本支持的应用程序。

**例如：** CRM、CMS系统。

**2.Redis，**所用语言：C/C++。特点：运行异常快

**最佳应用场景：**适用于数据变化快且数据库大小可遇见（适合内存容量）的应用程序。

**例如：**股票价格、数据分析、实时数据搜集、实时通讯。

**3. MongoDB，**所用语言：C++。特点：保留了SQL一些友好的特性（查询，索引）

**最佳应用场景**：适用于需要动态查询支持；需要使用索引而不是 map/reduce功能；需要对大数据库有性能要求；需要使用 CouchDB但因为数据改变太频繁而占满内存的应用程序。

**例如：**你本打算采用 MySQL或 PostgreSQL，但因为它们本身自带的预定义栏让你望而却步。

**4. Membase，**所用语言： Erlang和C。特点：兼容 Memcache，但同时兼具持久化和支持集群

**最佳应用场景：**适用于需要低延迟数据访问，高并发支持以及高可用性的应用程序

**例如：**低延迟数据访问比如以广告为目标的应用，高并发的 web 应用比如网络游戏（例如 Zynga）

# MongoDB数据库

MongoDB是一个文档数据库，旨在简化开发和扩展。

#主配置文件/etc/mongod.conf		服务名：mongod	Port端口：27017

1、cat >>		/etc/yum.repos.d/mongodb.repo	<<EOF

[mongodb-org-4.2]

name=MongoDB Repository

baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc

EOF

2、yum install -y mongodb-server

mongod -f /etc/mongod.conf	#启动服务器

#确保运行MongoDB的用户有权访问一个或多个目录：chown -R mongod:mongod <directory>

/var/log/mongodb/mongod.log文件中输出来跟踪错误或重要消息的处理状态

MongoDB的基本操作

Mongodb中关键字种类：

db（数据库实例级别）

​     db本身

​       db.connection 数据库下的集合信息

​         db.collection.xxx(

rs（复制集级别）

sh（分片级别）

基本操作：[mongod@MongoDB ~]$ mongo 10.0.0.152/admin	#客户端对数据库进行连接（默认连接本机test数据库）

\> db #显示登陆数据库		> db.version()	#查看数据库版本	> use test	#切换数据库

\> db.getName() #显示当前数据库							> show dbs #查询所有数据库

\> db.getMongo() #查看当前数据库的连接机器地址				> use clsn; #创建数据库

\> use clsn;	> db.stats() #查看clsn数据库当前状态			> db.dropDatabase() #删除数据库

创建集合：> db.createCollection('a')

当插入一个文档的时候，一个集合就会自动创建：> db.a.insert({name:'clsn'});

查看当前数据下的所有集合：> show collections;	或> db.getCollectionNames()

查看合集里的内容：> db.a.find()

重命名集合：> db.a.renameCollection("clsn")

删除合集：> db.a.drop()

插入1w行数据：> for(i=0;i<10000;i++){ db.log.insert({"uid":i,"name":"mongodb","age":6,"date":new Date()}); }

WriteResult({ "nInserted" : 1 })

查询集合中的查询所有记录：> db.log.find()

注：默认每页显示20条记录，当显示不下的的情况下，可以用it迭代命令查询下一页数据。

\> DBQuery.shellBatchSize=50;   # 每页显示50条记录

app> db.log.findOne()       # 查看第1条记录

app> db.log.count()        # 查询总的记录数

app> db.log.find({uid:1000});   # 查询UUID为1000的数据

删除集合中的记录数：>  db.log.distinct("name")    #  查询去掉当前集合中某列的重复数据

\> db.log.remove({})       #  删除集合中所有记录

查看集合存储信息：> db.log.stats()      # 查看数据状态

\> db.log.dataSize()    # 集合中数据的原始大小

\> db.log.totalIndexSize() # 集合中索引数据的原始大小

\> db.log.totalSize()    # 集合中索引+数据压缩存储之后的大小

\> db.log.storageSize()   # 集合中数据压缩存储的大小

pretty()使用：> db.log.find({uid:1000}).pretty()

删除集合中的记录数：>  db.log.distinct("name")    #  查询去掉当前集合中某列的重复数据

\> db.log.remove({})       #  删除集合中所有记录

查看集合存储信息：> db.log.stats()      # 查看数据状态

\> db.log.dataSize()    # 集合中数据的原始大小

\> db.log.totalIndexSize() # 集合中索引数据的原始大小

\> db.log.totalSize()    # 集合中索引+数据压缩存储之后的大小

\> db.log.storageSize()   # 集合中数据压缩存储的大小

# Memcached数据库

Memcached 是一套开源的高性能分布式内存对象缓存系统，它将所有的数据都存储在内存中，因为在内存中会统一维护一张巨大的Hash表，所以支持任意存储类型的数据。很多网站通过使用Memcached提高网站的访问速度，尤其是对于大型的需要频繁访问数据的网站。使用C语言编写

Memcached是典型的C/S架构，因此需要安装Memcached服务端与MemcachedAPI客户端。

```sh
yum install gcc gcc-c++ make -y #yum安装gcc编译环境包
yum install libevent-devel.x86_64 -y #安装依赖包
wget -O memcached-latest.tar.gz http://memcached.org/latest #下载安装包
tar -xzvf memcached-latest.tar.gz  #解压文件包
./configure --prefix=/usr/local/memcached #指定libevent安装路径
make && make test && make install
ln -s /usr/local/memcached/bin/* /usr/local/bin/	#创建软连接，方便使用memcached服务命令
memcached -d -m 32m -p 11211 -u root	#启动 memcached
```

-d：守护进程后台模式、-m：指定缓存大小为32M 、-p：指定默认端口11211 、 -u：指定登陆用户为 root

ss -tupln | grep memcached		#查看服务启动状态（默认端口号11211）

Memcached数据库操作与管理：Memcached协议简单，可直接使用telenet连接Memcached的11211端口操作

[root@localhost memcached-1.6.6]# telnet localhost 11211	#本地登录服务

gitlab部署安装

Git：是一种版本控制系统，是一个命令，是一种工具。

Gitlib：是用于实现Git功能的开发库。

Github：是一个基于Git实现的在线代码托管仓库，包含一个网站界面，向互联网开放。

GitLab：是一个基于Git实现的在线代码仓库托管软件，你可以用gitlab自己搭建一个类似于Github一样的系统，一般用于在企业、学校等内部网络搭建git私服。

GitLab是利用Ruby on Rails一个开源的版本管理系统，实现一个私有的自托管Git项目仓库，可通过Web界面进行访问公开的或者私人项目。

与Github类似，GitLab能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序(Wall)进行交流。

它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

```
gitlab-ctl start  #启动所有gitlab组件；			gitlab-ctl stop  #停止所有gitlab组件； 
gitlab-ctl restart  #重启所有gitlab组件； 			gitlab-ctl status  #查看服务状态； 
gitlab-ctl reconfigure  #修改配置文件之后,重新加载gitlab配置文件并启动所有gitlab组件
vim /etc/gitlab/gitlab.rb  #修改默认的配置文件； 
gitlab-rake gitlab:check SANITIZE=true --trace  #检查gitlab； 
gitlab-ctl tail  #查看日志；						gitlab-ctl --help  #查看gitlab命令的帮助
```

安装使用GitLab需要至少4GB可用内存(RAM + Swap)! 由于操作系统和其他正在运行的应用也会使用内存, 所以安装GitLab前一定要注意当前服务器至少有4GB的可用内存. 少于4GB内存会出现各种诡异的问题, 而且在使用过程中也经常会出现500错误.

1. 安装依赖软件	yum -y install policycoreutils openssh-server openssh-clients postfix

2.设置postfix开机自启并启动，postfix支持gitlab发信功能	systemctl enable postfix && systemctl start postfix

3.下载gitlab安装包，然后安装：[centos 6系统的下载地址](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6)		[centos 7系统的下载地址](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7)

```sh
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-9.5.9-ce.0.el7.x86_64.rpm
rpm -ivh gitlab-ce-9.5.9-ce.0.el7.x86_64.rpm
```

4.修改gitlab配置文件指定服务器ip和自定义端口	echo "xternal_url 'http://localhost'" >> /etc/gitlab/gitlab.rb

5.设置发邮件功能	

```sh
[root@web1134 ~]# vim /etc/gitlab/gitlab.rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "smtp user@163.com"
gitlab_rails['smtp_password'] = "password"
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
```

# 修改gitlab配置的发信人

```
gitlab_rails['gitlab_email_from'] = "smtp user@163.com"
user["git_user_email"] = "smtp user@163.com"
6.GitLab重置并启动	gitlab-ctl reconfigure && gitlab-ctl restart
ok: run: gitlab-git-http-server: (pid 3922) 1s
ok: run: logrotate: (pid 3929) 0s
ok: run: nginx: (pid 3936) 1s
ok: run: postgresql: (pid 3941) 0s
ok: run: redis: (pid 3950) 0s
ok: run: sidekiq: (pid 3955) 0s
ok: run: unicorn: (pid 3961) 1s
提示“ok: run:”表示启动成功。
```

6.访问 GitLab页面http:192.168.3.8，如果没有域名，直接输入服务器ip和指定端口进行访问，设置初始化密码如: 5iveL!fe，默认账户root

安装过程出现的报错处理

（1）登录502报错：一般是权限问题，解决方法：chmod -R 755 /var/log/gitlab

（2）执行gitlab-ctl reconfigure命令出现账户权限报错

n itdb: could not obtain information about current user: Permission denied

Error executing action `run` on resource 'execute[/opt/gitlab/embedded/bin/initdb -D /var/opt/gitlab/postgresql/data -E UTF8]'

根据报错信息大概锁定用户的权限问题,安装gitlab-ce会自动添加用户四个用户:

gitlab-www:x:497:498::/var/opt/gitlab/nginx:/bin/false

git:x:496:497::/var/opt/gitlab:/bin/sh

gitlab-redis:x:495:496::/var/opt/gitlab/redis:/bin/nologin

gitlab-psql:x:494:495::/var/opt/gitlab/postgresql:/bin/sh

文件/etc/passwd的权限是600,给予644权限后,成功解决报错问题

LAMP 架构部署动态网站环境

免费、高效、扩展性强且资源消耗低

使用linux系统架构 > apache提供服务,接受用户连接请求 > 调用libphpx.so模块 > 访问mysqld

<img src="E:\Project\Textbook\linux云计算\assets\wps3-1682690463420-253.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps4-1682690463420-254.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps5-1682690463420-255.jpg" alt="img" style="zoom:50%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps6-1682690463420-256.jpg" alt="img" style="zoom:50%;" /> 

作用:PHP主要负责PHP脚本程序的解析以及实现与 MYSQL数据库的交互工作,我们项目中的注册登录/下单皮付等大多数功能都是基于PHP+ MYSQL进行实现。PHP是一种通用开源脚本

<img src="E:\Project\Textbook\linux云计算\assets\wps7-1682690463420-257.jpg" alt="img" style="zoom:50%;" /> 

作用:MSQL是一个关系型数据库管理系统,由瑞典 MYSQL AB公司开发,目前属于 Oracl旗下产品。其主要作用用于永久的存储数据

安装启动httpd	php	(非服务不启动)	mariadb-server

```sh
mysqladmin -u root password 123456设置mysql数据库密码	mysql -u root -p进入数据库
```

默认数据库文件/var/lib/mysql		测试网站能否解析php文件cat /var/www/html/index.php

```php
<?php
	phpinfo();
?>
```

传递网页文件		修改文件权限chmod html/		安装

个人博客

```sh
安装apache、php、mariadb更新 yum 中 PHP 的软件源
rpm -Uvh https://mirrors.cloud.tencent.com/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install mod_php72w.x86_64 php72w-cli.x86_64 php72w-common.x86_64 php72w-mysqlnd php72w-fpm.x86_64
echo "<?php phpinfo(); ?>" >> /usr/share/nginx/html/index.php检测环境

配置mariadb数据库
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.5/centos8-amd64
module_hotfixes=1    #是解决被告知的dnf错误的方法
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

yum -y install MariaDB-client MariaDB-server安装
yum install -y httpd mariadb-server

启动相应的服务systemctl start httpd mariadb
chown  apache /var/www/html			修改apache账户为不可登录
mysqladmin -u root password 123456	设置mysql数据库密码

传递wordpress至web服务器根目录
create database wordpress；创建一个名为wordpress的数据库

重启服务	systemctl enable httpd  mariadb && systemctl start mariadb httpd
wp-config.php为wordpres站点基础配置文件

vim /etc/php.ini	修改上传值大小
  `upload_max_filesize = 50M` （最大上传文件大小）
  `post_max_size = 50M`（POST数据最大字节长度）`
  `max_execution_time = 300 `（最大执行时间，单位秒）`
```

修改ECS云服务器selinux需重启服务器

# Ansible批量控制工具

## ansible 特点

1. 一款基于ssh远程通讯新型的自动化运维工具
2. 配置简单、功能强大、扩展性强，实现批量系统配置，程序部署，运行命令
3. 部署简单，只需在主控端部署Ansible环境，被控端无需做任何操作
4. 基于python开发继承多个运维工具，支持API及自定义模块，可通过Python轻松扩展
5. 通过Playbooks来定制强大的配置、状态管理
6. 轻量级，无需在客户端安装agent，更新时，只需在操作机上进行一次更新即可
7. 提供一个功能强大、操作性强的Web管理界面和REST API接口——AWX平台

## ansible 架构图

Ansible：				Ansible核心程序

HostInventory：	记录由Ansible管理的主机信息，包括端口、密码、ip等

Playbooks：		"剧本”YAML格式文件，多个任务定义在一个文件中，定义主机需要调用哪些模块来完成的功能

CoreModules：	核心模块，主要操作是通过调用核心模块来完成管理任务

CustomModules：	自定义模块，完成核心模块无法完成的功能，支持多种语言

ConnectionPlugins：连接插件，Ansible和Host通信使用

## ansible 任务执行模式及过程

**ad-hoc模式(点对点模式)**

　　使用单个模块，支持批量执行单条命令。ad-hoc 命令是一种可以快速输入的命令，而且不需要保存起来的命令。就相当于bash中的一句话shell

**playbook模式(剧本模式)**

　　是Ansible主要管理方式，也是Ansible功能强大的关键所在。playbook通过多个task集合完成一类功能，如Web服务的安装部署、数据库服务器的批量备份等。可以简单地把playbook理解为通过组合多条ad-hoc操作的配置文件

<img src="E:\Project\Textbook\linux云计算\assets\wps8-1682690463420-258.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps9-1682690463420-259.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps10-1682690463421-260.jpg" alt="img" style="zoom:67%;" /> 

主配置文件/etc/ansible/ansible.cfg，常用参数：

```
inventory = /etc/ansible/hosts#ansuble主机清单(内附配置格式)
library=/usr/share/ansible	#指向存放Ansible模块的目录
forks = 5					#并发连接数，默认为5
sudo_user = root			#设置默认执行命令的用户
remote_port = 22			#指定连接被管节点的管理端口，默认为22端口，建议修改，能够更加安全
host_key_checking = False	#检查SSH主机的密钥，值为True/False。False则第一次连接不会提示配置实例
timeout = 60				#设置SSH连接的超时时间，单位为秒
log_path=/var/log/ansible.log#ansible日志的文件（默认不记录日志）
```

## ansible 常用命令及参数

ansible临时命令执行工具，常用于临时命令的执行

基本用法：ansible <host-pattern> [-f forks] [-m module_name] [-a args]

```
ansible *web -m command -a 'ls / ' -u root -k	对所有配置清单内主机基于-k验证
all：表示所有Inventory中的所有主机
`*`：通配符ansible “*” -m ping

或关系ansible 'webserver:dbserver' -m ping #执行在web组并且在dbserver组中的主机
与关系ansible 'webserver:&dbserver‘ -m ping
非逻辑ansible 'webserver:!dbserver' -m ping  【注意此处只能使用单引号！】

-m 	#执行模块的名字，默认使用 command 模块可以不写-m command
-a	#模块的参数
-u alex #远程用户，默认为 root 用户
--list	 #查看有哪些主机组
-k	#ask for SSH password。登录密码，提示输入SSH密码而不是假设基于密钥的验证
--ask-su-pass 	#ask for su password。su切换密码
-K	#ask for sudo password。提示密码使用sudo，sudo表示提权操作
--ask-vault-pass #ask for vault password。假设我们设定了加密的密码，则用该选项进行访问
-B SECONDS	#后台运行超时时间
-C	#模拟运行环境并进行预运行，可以进行查错测试
-c CONNECTION 	#连接类型使用
-f FORKS 		#并行任务数，默认为5
-i INVENTORY 	#指定主机清单的路径，默认为/etc/ansible/hosts
-o	#压缩输出，尝试将所有结果在一行输出，一般针对收集工具使用
-S #用 su 命令		-s #用 sudo 命令
-R SU_USER 	#指定 su 的用户，默认为 root 用户
-U SUDO_USER	#指定 sudo 的用户，默认为 root 用户
-T TIMEOUT	#指定 ssh 默认超时时间，默认为10s，也可在配置文件中修改
-v 	#查看详细信息，同时支持-vvv可查看更详细信息
```

ansible-doc		模块功能查看工具

-l	#全部模块的信息		-s ping	#以playbook简写片段显示指定模块的使用帮助

ansible-galaxy　　下载/上传优秀代码或Roles模块 的官网平台	ansible-galaxy install geerlingguy.nginx

list 列出		install安装	remove删除

ansible-playbook	定制自动化的任务集编排工具ansible-playbook <filename.yml>  ... [options]

-C预执行yaml检测		--syntax-check #检查语法		-v --v显示过程（-vvv更详细 ）

--list-hosts /tags 列出运行任务的主机 / 标签 		--limit 列出执行的主机

ansible-pull		远程执行命令的工具，拉取配置而非推送配置（使用较少，海量机器时使用，对运维的架构能力要求较高） 

ansible-vault		文件加密工具	ansible-vault encrypt hello.yml

encrypt加密	decrypt解密	view查看		edit编辑		create新建

ansible-console 	基于Linux Consoble界面可与用户交互的命令执行工具

root@test(2)[f:10] $  执行用户@当前操作的主机组（当前组的主机数量）[f:并发数]$	使用：模块+命令

设置并发数：fock n 例如：fock 10       切换组：cd 主机组 例如：cd webser / 192.168.0.106   

列出当前组的主机列表：list       	 列出所有内置命令：？或help

## ansible 常用模块

command 模块	默认模块可以直接在远程主机上执行命令，并将结果返回本主机

它不会通过shell进行处理，比如$HOME和操作如"<"，">"，"|"，";"，"&" 工作也不支持管道符号 |

shell 模块		可以在远程主机上调用shell解释器运行命令，支持shell的各种功能，例如管道等

copy 模块		用于将文件复制到远程主机，同时支持给定内容生成文件和修改权限等

ansible all -m copy -a 'src=/s.sh dest=/ backup=yes mode=0664'

src			#本地文件。绝对路径，也可以相对路径。路径是一个目录，则递归复制

dest			#远程主机的绝对路径，不存在则会创建

owner / group#指出复制时，目标文件的所有者 / 所属组

backup=yes|no#默认no，当文件发生改变后，在覆盖之前把源文件先进行备份

mode		#递归设定目录的权限，默认为系统默认权限

content		#用于替换"src"，可以直接指定文件的值

force=yes|no	#目标主机包含该文件，但内容不同时，yes（默认）强制覆盖，no被控端位置不存在该文件复制

fetch 模块		从被控端获取（复制）文件到本地，拷贝多个文件可使用tar打包

ansible web -m fetch -a 'src=/data/hello dest=/data' 

src：在远程拉取的文件，并且必须是单个file，且不是目录		dest：用来存放文件的目录

script 模块	将本机绝对路径的脚本在被控端上运行	ansible all -m script -a '/s.sh'

file 模块		用于设置文件的属性，比如创建文件、创建链接文件、删除文件等

state		#状态，有以下选项：		ansible all -m file -a 'name=/sls state=touch'

directory：如果目录不存在，就创建目录

file：即使文件不存在，也不会被创建

touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间

absent：删除目录、文件或取消链接文件

link / hard：创建软链接 / 硬链接

force　　	#需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目>标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no

mode：定义文件/目录的权限

path：定义文件/目录的路径

owner/group		#定义文件/目录的属主/属组。后面必须跟上	

recurse	#递归设置文件的属性，只对目录有效，后面跟上src：被链接的源文件路径，只应用于state=link的情况

dest		#被链接到的路径，只应用于state=link的情况

hostname模块	修改n被控端主机名		ansible all -m hostname -a 'name=web1'

cron 模块		用于管理cron计划任务，使用语法跟crontab语法一致

ansible all -m cron -a  'minute=*  weekday=1,3,5  job="/usr/bin/echo  $(date)" name=sss'

​    day= #日应该运行的工作( 1-31, *, */2, )

​    hour= # 小时 ( 0-23, *, */2, )

​    minute= #分钟( 0-59, *, */2, )

​    month= # 月( 1-12, *, /2, )

​    weekday= # 周 ( 0-6 for Sunday-Saturday,, )

​    job= #指明运行的命令是什么

​    name= #定时任务描述

​    reboot # 任务在重启时运行，不建议使用，建议使用special_time

​    special_time #特殊的时间范围，参数：reboot（重启时），annually（每年），monthly（每月），weekly

（每周），daily（每天），hourly（每小时）

​    state #指定状态，present表示添加定时任务，也是默认设置，absent表示删除定时任务

​    user # 以哪个用户的身份执行

yum 模块	用于软件的安装			ansible all -m yum -a 'name=tree,vsftpd  diable_gpg_check = yes'

安装rpm包，先推送到被控端，指定路径	ansible all -m yum -a 'name=/pptpd.rpm'

​    name=　　	#所安装的包的名称

​    state=　　	#执行操作present-->安装（默认选项），latest-->安装最新的，absent--> 卸载软件

​    update_cache	#强制更新yum的缓存

​    conf_file		#指定远程yum安装时所依赖的配置文件（安装本地已有的包）

​    diable_gpg_check = yes | no : 是否启用包完整性校验功能gpgcheck

​    disablerepo	#临时禁止使用yum库 只用于安装或更新时

​    enablerepo	#临时使用的yum库只用于安装或更新时

service 模块   用于服务程序的管理		ansible all -m service -a 'name=vsftpd state=started'

​    state			#执行状态，started启动服务， stopped停止服务， restarted重启服务， reloaded重载配置

​    enabled		#设置开机启动

​    name=		#服务名称

​    runlevel		#开机启动的级别，一般不用指定

​    sleep		#在重启服务的过程中，是否等待如在服务关闭以后等待2秒再启动(定义在剧本中)

 arguments	#命令行提供额外的参数

user 模块    用来管理用户账号

ansible all -m user -a 'name=user1  shell=/sbin/nologin system=yes home=/home group=root'

​    name		# 指定用户名

​    shell			# 指定默认shell

​    home		# 指定用户家目录

​    system=yes | no ：是否为系统账号，这个设置不能更改现有用户

​    createhome	# 是否创建家目录

 state=present | absent： 创建账号或者删除账号，present 创建（默认），absent删除

 remove=yes | no ：当state=absent 时，是否删除用户的家目录

​    force		# 在使用state=absent时, 行为与userdel –force一致.

​    group		# 指定基本组

​    groups		# 指定附加组，如果指定为(groups=)表示删除所有组

​    move_home=yes | no :如果设置的家目录以及存在，是否将已经存在的家目录进行移动

​    non_unique	# 该选项允许改变非唯一的用户ID值

​    password		# 指定用户密码

​    uid=1200		# 指定用户的uid

​    comment		# 用户的描述信息

group 模块	用于添加或删除组	ansible all -m group -a 'name=sss system=yes gid=1300'

​    gid=		#设置组的GID号

​    name=	#指定组的名称

​    state=	#指定组的状态，默认为创建，设置值为absent为删除

​    system=	#设置值为yes，表示创建为系统组

setup 模块	用于收集被控端系统信息，是通过调用facts组件来实现的，facts组件是Ansible用于采集被管机器设备信息

ansible web -m setup -a 'filter="vcpus"' --tree /tmp/facts筛选的信息发送至主控端

## Ansible-playbook剧本命令

playbook 是 ansible 用于配置，部署，和管理被控节点的剧本

```yaml
---				#ansible-playbook格式：-后多空格		注意平级关系
- hosts: web		#主机清单		支持ansible *web通配符系列，支持逻辑运算符：，&，！
 remote_user: root	#被控端执行用户
 vars: 
   -  pkname: httpd#可对多个变量进行赋值
 vars_files: 
   - vars.yml		#指定变量存放文件
 tasks:			#任务集合
   - name: install httpd package	#任务名称，一个任务对应一个模块
​     yum：name=httpd			#使用模块,执行命令
- name: install package
 yum：name={{ pkname }}	 #定义变量{{ pkname }}	（变量命名规则与c语言类似）
​    - name: 'shutdown redhad flavored systems'
​     cp: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
​     when: ansible_os_family == "Centos"	#when条件语句，当符合centos的条件时执才行该任务模块
​    - name: unstall web packages
​     yum: name={{ item }} state=absent	#固定变量名“item”
​     with_items:						#with_items	循环：迭代元素列表，需要重复执行的任务
​     - httpd		#迭代元素
​     - php
​    - name: add some users				#迭代嵌套子变量
​     user: name={{ item.name }} group={{ item.group }} state=present	#引入迭代列表的元素
​     with_items:
​       - { name: 'u1', group: 'g1' }		#迭代字典
​       - { name: 'u2', group: 'g2' }
 notify: service		 #通知器：可通知多个触发器handlers
 tags: conf			 #一个标签可对应多个任务模块，使用方法如下红色区
 sudo_user: wang		 #sudo为wang		#sudo: yes#默认sudo为root
 #如果命令或脚本的退出码不为零，可以使用如下方式替代
 shell : /usr/bin/yum || /bin/true
 ignore_errors: True	#使用ignore_errors来忽略错误信息
 handlers: 			#触发器：当脚本运行前面的notify则会执行该触发器，可设置多个任务模块
- name: service
  service: name=httpd state=restarted
```

ansible-playbook -C sss.yml	预加载脚本，检测

ansible-playbook -t conf httpd.yml  只运行脚本里所有带有conf标签的任务模块【使用-t 指定标签名字】

ansible-playbook -e 'pkname=tree pkname1=epel' sss.yml	对sss.yml里面多个kpname变量临时进行赋值

变量的优先级：命令行中的-e > playbook中定义的变量 > /etc/ansible/hosts[webservs]全局变量>vars共有变量

<img src="E:\Project\Textbook\linux云计算\assets\wps11-1682690463421-261.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps12-1682690463421-262.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps13-1682690463421-263.jpg" alt="img" style="zoom:67%;" /> 

templates模板用来存放各种服务配置文件的模板，是一个文本文件，嵌套有脚本（使用模板编程语言编写），用jinja2语言，以 j2 结尾，有如上形式:

Jinja2：Jinja2是python的一种模板语言，以Django的模板语言为原本

存储template模块调用的模板文件夹/etc/ansible/roles/templates/

playbook中template模板对于for  if 循环的使用

\---

\- hosts: web	#主机清单		支持ansible *web通配符系列，支持逻辑运算符：，&，！

 remote_user: root	#被控端执行用户

 vars:

  ports:	#定义变量列表，用于实现for的template模板所需

   \- web1: 	

​    port: 81

​    name: web1.magedu.com

​    rootdir: /data/website1

   \- web2: 	

​    port: 82

​    #name: web2.magedu.com

​    rootdir: /data/website2

 tasks:

  \- name: copy conf

   template: src=if.conf.j2 dest=/data/if.conf

vim templates/for1.conf.j2 模板文件：

{% for p in ports %}			#for循环语句，p自定义变量，ports变量列表	#与shell脚本非常类似

server{

​    listen {{ p.port }}		#从p变量内拿取元素port

{% if p.name is defined %}	#如果p.name被定义就执行servername

​    servername {{ p.name }}

{% endif %}				#否则执行

​    documentroot {{ p.rootdir }}

}

{% endfor %}

## 角色订制：roles剧本文件

实际上分解playbook而后结构化地组织，比较灵活。roles 能够根据层次型结构自动装载变量文件、tasks以及handlers等

roles剧本分别将变量(vars)、文件(file)、任务(tasks)、模块(modules)及处理器(handlers)放置于单独的目录中，并可以通过include便捷地使用它们的一种机制

![img](E:\Project\Textbook\linux云计算\assets\wps14-1682690463421-264.jpg) 

角色集合：/etc/ansible/roles/

mysql/，httpd/，nginx/

files/：存放由copy模块或scripts模块等调用的文件 

template/：template模块 查找所需要模板文件的目录

tasks/：定义tasks，roles的基本元素，至少应该包含一个名为main.yml的文件；其他的文件需要>在此文件中通过include进行调用

handlers/：至少应该包含一个名为main.yml的文件；其他的文件需要在此文件中通过include进行调用

vars/：定义变量，至少应该包含一个名为main.yml的文件；其他的文件需要在此文件中通过include进行调用

meta/：定义当前角色的特殊设定及其依赖关系，至少应该包含一个名为main.yml的文件；其他的文

件需要在此文件中通过include进行调用

default/：设定默认变量时使用此目录中的main.yml文件

![img](E:\Project\Textbook\linux云计算\assets\wps15-1682690463421-265.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps16-1682690463421-266.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps17-1682690463421-267.jpg) 

# LVS（Linux Virtual Server） 负载均衡

LVS即Linux虚拟服务器也称负载调度器，是一个开源的负载均衡集群项目，LVS架构从逻辑上可分为调度层、Server集群层和共享存储，位于第四层传输层

Cluster集群分类：为解决某个特定问题将多台计算机组合形成一个单一系统的操作

LB负载均衡集群（Loadbalancingclusters）：负载均衡集群把很多客户集中访问的请求负载压力可能尽可能平均的分摊到计算机集群中处理，户请求负载通常包括应用程度处理负载和网络流量负载。这样的系统非常适合向使用同一组应用程序为大量用户提供服务。每个节点都可以承担一定的访问请求负载压力，并且可以实现访问请求在各节点之间动态分配，以实现负载均衡。

负载均衡运行时，一般通过一个或多个前端负载均衡器将客户访问请求分发到后端一组服务器上，从而达到整个系统的高性能和高可用性。这样计算机集群有时也被称为服务器群。一般高可用性集群和负载均衡集群会使用类似的技术，或同时具有高可用性与负载均衡的特点。

负载均衡集群的作用1）分担访问流量（负载均衡）2）保持业务的连续性（高可用）

HA高可用性集群（High-availabilityclusters）：一般是指当集群中的任意一个节点失效的情况下，节点上的所有任务自动转移到其他正常的节点上，并且此过程不影响整个集群的运行，不影响业务的提供。  类似是集群中运行着两个或两个以上的一样的节点，当某个主节点出现故障的时候，那么其他作为从 节点的节点就会接替主节点上面的任务。从节点可以接管主节点的资源（IP地址，架构身份等），此时用户不会发现提供服务的对象从主节点转移到从节点。  高可用性集群的作用：当一个机器宕机另一台进行接管。比较常用的高可用集群开源软件有：keepalived，heardbeat。

HA:高可用,SPOF	MTBF: 平均无故障时诃	MTTR: 平均恢复前时间

HA=MTBF/(MTBF+MTTR)(0,1):99.5%,99.9%,99.999%衡量可用性的标准

HPC/HP高性能计算集群（High-perfomanceclusters）：高性能计算集群采用将计算任务分配到集群的不同计算节点儿提高计算能力，因而主要应用在科学计算领域。比较流行的HPC采用Linux操作系统和其它一些免费软件来完成并行运算。这一集群配置通常被称为Beowulf集群。这类集群通常运行特定的程序以发挥HPCcluster的并行能力。这类程序一般应用特定的运行库, 比如专为科学计算设计的MPI库。  HPC集群特别适合于在计算中各计算节点之间发生大量数据通讯的计算作业，比如一个节点的中间结果或影响到其它节点计算结果的情况。

常用集群软硬件常用开源集群软件有：lvs，keepalived，haproxy，nginx，apache，heartbeat

常用商业集群硬件有：F5,Netscaler，Radware，A10等

## LVS工作流程

![img](E:\Project\Textbook\linux云计算\assets\wps18-1682690463421-268.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps19-1682690463421-269.jpg) 

1. 当用户向负载均衡调度器（Director Server）发起请求，调度器将请求发往至内核空间
2. prerouting链首先会接收到用户请求，判断目标IP确定是本机IP，将数据包发往INPUT链
3. IPVS是工作在INPUT链上的，当用户请求到达INPUT时，IPVS会将用户请求和自己已定义好的集群服务进行比对，如果用户请求的就是定义的集群服务，那么此时IPVS会强行修改数据包里的目标IP地址及端口，并将新的数据包发往POSTROUTING链
4. prerouting链接收数据包，对比内网服务器IP地址匹配成功则进行发送，将数据包最终发送给后端的服务器

## LVS组成工具

LVS 由2部分程序组成，包括 ipvs 和 ipvsadm

1. ipvs(ip virtual server)：工作于内核空间netfilter的INPUT链的一个框架
2. ipvsadm：对ipvs内核框架编写规则的管理工具，定义谁是集群服务，而谁是后端真实的服务器(Real Server)

Ipvsadm程序包: ipvsadm		服务名: ipvsadm.service

主程序/usr/sbin/ ipvsadm		规则保存工具:/usr/sbin/ipvsadm-save

规则重载工具:/usr/sbin/ /ipvsadm- restore	配置文件:/etc/ sysconfig/ ipsan- config

```
ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]
```

-A	添加一个虚拟服务，使用ip地址、端口号、协议来唯一定义一个虚拟服务

-E	修改一个虚拟服务	-D 删除一个虚拟服务	-C 清空虚拟服务列表

-Lnc	显示虚拟服务列表	(-n 数字格式显示,-c 当前IPVS 的连接输出

扩展信息，--exact：显示精确值如个位数		--sort   对虚拟服务器和真实服务器排序输出)

--rate：输出速率信息		--stats：统计信息	--timeout 显示tcp tcpfin udp的timeout值

-Z	虚拟服务器列表计数器清零（清空当前连接数）

-a	添加一台真实服务器		-e 修改一台真实服务器	-d 删除一台真实服务器

-t	使用TCP服务，该参数后需加主机与端口信息	-u  使用UDP服务，该参数后需加主机与端口信息

-g	设置lvs工作模式为DR直连路由		-i 设置lvs工作模式为TUN隧道		-m 设置lvs工作模式为NAT地址转换模式

-s	指定lvs的scheduler调度算法

-r	设置真实服务器IP与端口

-w	指定真实服务器权重，默认为1

-f	防火墙标记

-R	从标准输入中还原虚拟服务列表		-S 保存虚拟服务规则至标准输出

```
ipvsadm-save -n >> /etc/sysconfig/ipvsadm	保存至开机文件夹
ipvsadm -Sn > /tmp/ipvsadm-config	保存虚拟服务规则备份
ipvsadm -Rn < /tmp/ipvsadm-config	还原配置
systemctl enable ipvsadm	设置开机自启
```



## LVS相关术语

CIP：Client IP，访问客户端的IP地址		VIP：Virtual server IP面对客户的虚拟IP地址

VS / DS：Director Server调度器			DIP：Director Server IP，面对内网服务器通讯的IP地址

RIP：Real Server IP，后端服务器的IP地址	RS：Real Server后端真实的工作服务器

## LVS集群分类

LVS—nat：修改请求报文的目标ip地址，本质是多目标IP的DNAT(全称为Destination Network Address Translation目的地址转换，常用于防火墙中)，通过将请求报文中的目标地址和目标端口修改为某挑出的RS的RP和Port实现转发

要求：(1)RIP和DP处于同一个IP网段（使用路由可以不用同一网段），且使用私网地址；RS的网关要指向DIP

(2)请求报文和响应报文都必须经由 Director转发, Director易于成为系统瓶颈

(3)支持端口映射,可修改请求报文的目标port

(4)VS必须是 Linux系统,RS可以是任意OS系统

缺陷：对Director Server压力会比较大，请求和响应都需经过director server

![img](E:\Project\Textbook\linux云计算\assets\wps20-1682690463421-270.jpg) 

安装 ipvsadm，增添多网卡，配置各IP，RS安装httpd，将网关指向DS内网IP

director服务器上开启路由转发功能: echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf && sysctl -p 是配置生效

配置NAT--VIP：

```
ipvsadm -A -t 192.168.0.10:80 -s rr #添加虚拟服务10/tcp:80(必须指定port)，采用权重rr
ipvsadm -a -t 192.168.0.10:80 -r 192.168.100.10 -m  -w 3
```

#添加服务器10/tcp:80的真实服务器100.10，lvs工作模式为nat集群

```
ipvsadm -a -t 192.168.0.10:80 -r 192.168.100.11 -m
ipvsadm-save	 -n >> /etc/sysconfig/ipvsadm	保存至开机文件夹
systemctl enable ipvsadm && systemctl restart ipvsadm.service	#设置开机自启
service network restart
#ipvsadm -E -t 192.168.0.10:80 -s wrr 	#修改scheduler调度为wrr
#ipvsadm -e -t 192.168.0.10:80 -r 192.168.100.11:81 -m -w 3	#修改权重为3
#ipvsadm -d -t 192.168.0.10:80 -r 192.168.100.11:81	#删除虚拟服务（不指定，默认删除80端口）
```

LVS—dr（Direct Routing）：封装新的MAC地址，直接路由,LVS默认模式应用最广泛通过为请求报文重新封装一个MAC首部进行转发,源MAC是DP所在的接口的MAC,目标MAC是某挑选出的RS的RIP所在接口的MAC地址;源IP/PORT,以及目标IP/PORT均保持不变

1，Director和各RS都配置有VIP

2，确保前端路由器将目标IP为ⅥP的请求报文发往 Director

（1）在前端网关做静态绑定ⅥP和 Director的MAC地址

（2）在RS上使用 arptables工具

```
arptables -A In-d VIP -j DROP
arptables-a OUT -S SVIP -j mangle --mangle-ip-s SRIP
```

（3）在RS上修改内核参数以限制arp通告及应答级别【推荐】

```
/proc/sys/ net/ipv4/conf/a/arp_ ignore #忽略arp的地址回应请求
```

0：默认值,表示可使用本地任意接口上配置的任意地址进行响应

1：仅在请求的目标IP配置在本地主机的接收到请求报文的接口上时,才给予晌应

```
/proc/sys net/ipv4/conf/all/arp_ announce不公布自己的MAC地址
```

0：默认值,把本机所有接口的所有信息向每个接口的网络进行通告

1：尽量避免将接口信息向非直接连接网络进行通告

2：禁止将接口信息向非本网络进行通告

3，RS的RIP可以使用私网地址,也可以是公网地址;RIP与DP在同一IP网络RIP的网关不能指向DIP,以确

保响应报文不会经由 Director

![img](E:\Project\Textbook\linux云计算\assets\wps21-1682690463421-271.jpg) 

4，RS和 Director要在同一个物理网络

5，请求报文要经由 Director,但响应报文不经由 Director,而由RS直接发往Client

6，不支持端口映射(端口不能修败）

7，RS可使用大多数OS系统

缺陷：RS和DS必须在同一机房中

VS和RS上开启路由转发功能: echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf && sysctl -p

VS安装ipvsadm

```
ifconfig ens33:0 192.168.100.200 netmask 255.255.255.255 up为VS端网卡配置上虚拟VIP
route add -host 192.168.100.200 dev ens33:0 设置路由
ipvsadm -A -t 192.168.100.200:80 -s rr
ipvsadm -a -t 192.168.100.200:80 -r 192.168.100.130 -g
ipvsadm -a -t 192.168.100.200:80 -r 192.168.100.131 -g
```

配置IPdirector,RS安装httpd

```
ifconfig ens33:0 192.168.100.200/24 up为RS端网卡配置上虚拟VIP
route add -host 192.168.100.200 dev ens33:0 设置路由
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
```

LVS—tun：在原请求IP报文之外新增加一个IP首部，不修改请求报文的IP首部(源IP为CIP,目标IP为ⅥP),而在原IP报文之外再封装一个IP首部(源IP是DIP,目标IP是RIP),将报文发往挑选出的目标RS;RS直接响应给客户端(源IP是VIP,目标IP是CIP

1，VIP是公网地址，DIP  RIP可以是私网地址

2，RS的网关一般不能指向DIP

3，请求报文要经由 Director,但响应不能经由 Director

4，不支持端口映射

5，RS的OS须支持隧道功能

![img](E:\Project\Textbook\linux云计算\assets\wps22-1682690463421-272.jpg) 

LVS—fullnat：修改请求报文的源和目的IP，通过同时修改请求报文的源IP地址和目标IP地址进行转发

CIP--> DIP		VIP--> RIP		#让RIP以为是LVS服务器请求

1，VIP是公网地址,RIP和DIP是私网地址,且通常不在同-IP网络;因此RIP的网关一般不会指向DIP

2，RS收到的请求报文源地址是DIP,因此,只需响应给DIP;但 Directori还要将其发往 Client

3，请求和响应报文都经由 Director

4，支持端口映射

注：此类型 kernell默认不支持

## PVS scheduler调度算法

根据其调度时是否考虑各RS当前的负载状态两种:静态方法和动态方法

静态方法：仅根据算法本身进行调度

1、 rr: roundrobin,轮询	123123···

2、 wrr: Weighted RR,加权轮询（按比例分配如5：3：2）

3、 sh: Source Hashing（源IP地址哈希）即首次访问记住IP hash，再次访问会指向首次访问RIP服务器

4、 dh: Destination Hashing（目标IP地址哈希）即同一CIP的请求由第一次握手的RS继续发送,典型使用场景是正向代理缓存场景中的负載均衡,如:宽带运营商

动态方法：主要根据每RS当前的负载状及调度算法进行调度 Overhead= value较小的RS将被调度

1、 lc: least connections（最少连接数）适用于长连接应用

Overhead=activeconns*256（活动连接数）+inactiveconns（非活动连接数，只进行三次握手，不进行数据传输）

wlc: Weighted LC,默认调度方法

Overhead（负载）=(activeconns* 256+inactiveconns)/weight（权重）

2、 sed: Shortest Expection Delay,初始连接为1，高权重优先

Overhead=(activeconns+1) *256/weight

3、 nq: Never Queue,第一轮均匀分配,后续SED

4、 lblc: Locality- Based LC,动态的DH算法,使用场景:根据负载状态实现正向代理

5、 lblcr: LBLC with Replication,带复制功能的LBLC,解決LBLC负载不均衡问题,从负载重的复制到负载轻的RS

keepalived高可用集群

脑裂：即出现在主备模式下的，当主服务器与备服务器连接断开时，备服务器认为主服务器宕机，自动充当主服务器

解决脑裂方法：

1、 添加更多的检测手段,比如冗余的心跳线增加一块网卡做健康监测),ping对方等等。尽量减少"裂脑"发生机会。(指标不治本,

2、 设置仲裁机制。两方都不可靠,那就依赖第三方。比如启用共享磁盘锁,ping网关等。(针对不同的手段还需具体分析

3、算法保证,比如采用投票机制( keepalived没有实现)

keepalived是集群管理中基于vrrp协议的一款高可用软件，vrrp协议: Virtual Router Redundancy Protocol虚拟路由器冗余（备份）协议

（1）抢占模式：正常时，master主节点管理虚拟IP向backup节点发送多播心跳消息，宕机后backup转主服务器，master恢复后继续发送多播心跳消息，备服务器释放虚拟IP等资源

模块：core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析

check模块负责健康检查，健康检查方式有三种，tcp_check、http_check、misc_check

vrrp模块是来实现VRRP协议的，keepalived只有一个配置文件keepalived.conf，内配置区域global_defs、static_ipaddress、static_routes、vrrp_script、vrrp_instance和virtual_server

安装包：keepalived		主配置文件：vim /etc/keepalived/keepalived.conf

#由于keepalived是监控端口IP状态，无法监控web服务状态，当master节点无法访问，整个服务会处于假死状态

```
vim /etc/keepalived/keep_nginx.sh		&&		chmod +x  keep_nginx.sh

#!/bin/bash 
counter=$(netstat -tupln |grep nginx |wc -l)       #检查nginx进程是否存在
if [ "$counter"="0" ]; then
	systemctl restart nginx #尝试启动一次 nginx,停止5秒后再次检测
	sleep 5
	counter=$(netstat -tupln |grep nginx |wc -l)
	if [ "$counter"="0" ]; then
		systemctl stop keepalived #如果启动没成功,就杀掉 keepalive触发主备切换
	fi

fi
```

编辑主配置文件：

```
[root@localhost ~]# vim /etc/keepalived/keepalived.conf
#master节点配置，主要是配置故障发生时的通知对象以及机器标识
global_defs {
#   notification_email {
#     r_xl@xl.com   # 设置报警邮件接收地址，需要开启 sendmail 服务
#   }
#   notification_email_from s_xl@xl.com   # 设置邮件的发送地址
#   smtp_server 192.168.2.241	# 设置通知的 SMTP Server 地址 
#   smtp_connect_timeout 30 	# 设置通知的 SMTP Server 的超时时间 
router_id 1				 # VRRP组ID
}

# 自定义 keepalived只能做到对自身问题和网络故障的监控，Script可以>增加其他的监控来判定是否需要切换主备
vrrp_script keep_nginx {	#VRRP实例健康检查脚本
  script "/etc/keepalived/keep_nginx.sh"    # 示例为检查sshd服务是否运行中
  interval 2			# 检查间隔时间
  fall 3				#当失败三次自动降低权重
  weight -4			# 检查失败降低的权重
}

vrrp_instance VI_1 {	# VRRP实例 定义对外提供服务的VIP区域及其相关属性
  state MASTER		# 必须大写，MASTER 为工作状态，BACKUP 是备用状态
  interface ens33		# 节点固有IP(非VIP)的网卡，用来发VRRP包
  virtual_router_id 51	# 虚拟路由id，和备节点保持一致
  mcast_src_ip 10.139.1.10	# 本机IP地址
  priority 100      	# 优先级， MASTER 优先级必须比 BACKUP 高
  advert_int 1			# 心跳通告间隔，单位为秒

	authentication {		# 设置认证
		auth_type PASS	# 认证方式，支持PASS和HA
		auth_pass 1111	# 认证密码为明文，同一vrrp 实例 MASTER 与 BACKUP 使用相同的密码才能正常通信
	}

	virtual_ipaddress { 	# 虚拟IP地址(VIP)，可以有多个地址，每个地址占一行
		192.168.0.200/24 dev ens33
	}

	track_script {     # 自定义健康检查脚本
		keep_nginx	 # 配置上面自定义的vrrp脚本调用名
	}	
}

 
## 设置虚拟服务器
#virtual_server 192.168.12.200 6500 {   # 指定虚拟IP地址和服务端口
#   delay_loop 6	# 服务健康检查周期，6秒
#   lb_algo rr		# 负载均衡调度算法，一般用wrr、rr、wlc
#   lb_kind DR		# 负载均衡转发规则。一般包括DR,NAT,TUN 3种
#   persistence_timeout 5  #会话保持时间。把用户请求请求间隔在未超过保持时间时一>直分发到某个服务节点

#   protocol TCP	# 转发协议 有TCP和UDP两种# 配置真实服务器
#   real_server 192.168.2.222 6500 {   #指定VIP和端口，可设置多个VIP
#	 weight 1    # 权重，数值越大，权重越高

# 健康检查方式 常见有 TCP_CHECK, HTTP_GET, SSL_GET, MISC_CHECK(自定义脚本)
#     TCP_CHECK {        # 通过TcpCheck方式判断RealServer的健康状态
#       connect_timeout 10   # 连接超时时间
#       nb_get_retry 3		 # 重连次数
#       delay_before_retry 3	 # 重连时间间隔
#       connect_port 6500	 # 检测端口
#     }   }

## 配置真实服务器
#   real_server 192.168.2.222 6500 {   #指定IP和端口
#     weight 1    # 权重，数值越大，权重越高

## 健康检查方式 常见有 TCP_CHECK, HTTP_GET, SSL_GET, MISC_CHECK(自定义脚本)
#     TCP_CHECK {        	# 通过TcpCheck判断RealServer的健康状态
#       connect_timeout 10	# 连接超时时间
#       nb_get_retry 3		# 重连次数
#       delay_before_retry 3  # 重连时间间隔
#       connect_port 6500   # 检测端口
#     }    }  }

service keepalived start	启动服务
```

```
vim  ifcfg-ens33:0
DEVICE=ens33:0
IPADDR=192.168.0.200
NETMASK=255.255.255.0
ONBOOT=yes
```



# 监控服务

## ELK日志分析系统

官方文档：https://www.elastic.co/guide/index.html

ELK是Elasticsearch、Logstash、Kibana开源软件的集合，对外是作为一个日志管理系统的开源方案，它可以从任何来源、任何格式进行日志搜索、分析与可视化展示。

基本组成软件：

1、Filebeat：监控日志文件，获取服务器上指定路径的日志文件，并将这些日志转发到Logstash实例以进行处理。Filebeat的设计是为了可靠性和低延迟。Filebeat在主机上占用的资源很少，而Beats input插件将对Logstash实例的资源需求降到最低。

2、Logstash：它是一个服务端的数据处理管道，可以从多个源中提取数据，对其进行转换，然后将其存储到Elasticsearch中。简单来说就是日志的收集、分析、过滤工具。

3、Kibana：它是一个基于web的图形界面，用于搜索、分析和可视化存储在Elasticsearch中的日志数据。

## Elasticsearch弹性搜索

它是一个开源分布式搜索引擎，提供收集、分析、存储数据三大功能。为了保证搜索服务的高可用性，需要一个集群

### 介绍

Elasticsearch（ES）是一个基于Lucene构建的开源、分布式、RESTful接口的全文搜索引擎。Elasticsearch还是一个分布式文档数据库，其中每个字段均可被索引，而且每个字段的数据均可被搜索，ES能够横向扩展至数以百计的服务器存储以及处理PB级的数据。可以在极短的时间内存储、搜索和分析大量的数据。通常作为具有复杂搜索场景情况下的核心发动机。

### Elasticsearch能做什么

1. 当你经营一家网上商店，你可以让你的客户搜索你卖的商品。在这种情况下，你可以使用ElasticSearch来存储你的整个产品目录和库存信息，为客户提供精准搜索，可以为客户推荐相关商品。
2. 当你想收集日志或者交易数据的时候，需要分析和挖掘这些数据，寻找趋势，进行统计，总结，或发现异常。在这种情况下，你可以使用Logstash或者其他工具来进行收集数据，当这引起数据存储到ElasticsSearch中。你可以搜索和汇总这些数据，找到任何你感兴趣的信息。
3. 对于程序员来说，比较有名的案例是GitHub，GitHub的搜索是基于ElasticSearch构建的，在github.com/search页面，你可以搜索项目、用户、issue、pull request，还有代码。共有40~50个索引库，分别用于索引网站需要跟踪的各种数据。虽然只索引项目的主分支（master），但这个数据量依然巨大，包括20亿个索引文档，30TB的索引文件。

### Elasticsearch基本概念

Near Realtime(NRT) 几乎实时

Elasticsearch是一个几乎实时的搜索平台。意思是，从索引一个文档到这个文档可被搜索只需要一点点的延迟，这个时间一般为毫秒级。

**Cluster 集群**

群集是一个或多个节点（服务器）的集合， 这些节点共同保存整个数据，并在所有节点上提供联合索引和搜索功能。一个集群由一个唯一集群ID确定，并指定一个集群名（默认为“elasticsearch”）。该集群名非常重要，因为节点可以通过这个集群名加入群集，一个节点只能是群集的一部分。

确保在不同的环境中不要使用相同的群集名称，否则可能会导致连接错误的群集节点。例如，你可以使用logging-dev、logging-stage、logging-prod分别为开发、阶段产品、生产集群做记录。

**Node节点**

节点是单个服务器实例，它是群集的一部分，可以存储数据，并参与群集的索引和搜索功能。就像一个集群，节点的名称默认为一个随机的通用唯一标识符（UUID），确定在启动时分配给该节点。如果不希望默认，可以定义任何节点名。这个名字对管理很重要，目的是要确定你的网络服务器对应于你的ElasticSearch群集节点。

我们可以通过群集名配置节点以连接特定的群集。默认情况下，每个节点设置加入名为“elasticSearch”的集群。这意味着如果你启动多个节点在网络上，假设他们能发现彼此都会自动形成和加入一个名为“elasticsearch”的集群。

在单个群集中，你可以拥有尽可能多的节点。此外，如果“elasticsearch”在同一个网络中，没有其他节点正在运行，从单个节点的默认情况下会形成一个新的单节点名为”elasticsearch”的集群。

**Index索引**

索引是具有相似特性的文档集合。例如，可以为客户数据提供索引，为产品目录建立另一个索引，以及为订单数据建立另一个索引。索引由名称（必须全部为小写）标识，该名称用于在对其中的文档执行索引、搜索、更新和删除操作时引用索引。在单个群集中，你可以定义尽可能多的索引。

**Type类型**

在索引中，可以定义一个或多个类型。类型是索引的逻辑类别/分区，其语义完全取决于你。一般来说，类型定义为具有公共字段集的文档。例如，假设你运行一个博客平台，并将所有数据存储在一个索引中。在这个索引中，你可以为用户数据定义一种类型，为博客数据定义另一种类型，以及为注释数据定义另一类型。

**Document文档**

文档是可以被索引的信息的基本单位。例如，你可以为单个客户提供一个文档，单个产品提供另一个文档，以及单个订单提供另一个文档。本文件的表示形式为JSON（JavaScript Object Notation）格式，这是一种非常普遍的互联网数据交换格式。

在索引/类型中，你可以存储尽可能多的文档。请注意，尽管文档物理驻留在索引中，文档实际上必须索引或分配到索引中的类型。

**Shards & Replicas分片与副本**

索引可以存储大量的数据，这些数据可能超过单个节点的硬件限制。例如，十亿个文件占用磁盘空间1TB的单指标可能不适合对单个节点的磁盘或可能太慢服务仅从单个节点的搜索请求。

为了解决这一问题，Elasticsearch提供细分你的指标分成多个块称为分片的能力。当你创建一个索引，你可以简单地定义你想要的分片数量。每个分片本身是一个全功能的、独立的“指数”，可以托管在集群中的任何节点。

**Shards分片的重要性主要体现在以下两个特征：**

1. 分片允许你水平拆分或缩放内容的大小
2. 分片允许你分配和并行操作的碎片（可能在多个节点上）从而提高性能/吞吐量 这个机制中的碎片是分布式的以及其文件汇总到搜索请求是完全由ElasticSearch管理，对用户来说是透明的。

在同一个集群网络或云环境上，故障是任何时候都会出现的，拥有一个故障转移机制以防分片和节点因为某些原因离线或消失是非常有用的，并且被强烈推荐。为此，Elasticsearch允许你创建一个或多个拷贝，你的索引分片进入所谓的副本或称作复制品的分片，简称Replicas。

Replicas的重要性主要体现在以下两个特征：

1. 副本为分片或节点失败提供了高可用性。为此，需要注意的是，一个副本的分片不会分配在同一个节点作为原始的或主分片，副本是从主分片那里复制过来的。
2. 副本允许用户扩展你的搜索量或吞吐量，因为搜索可以在所有副本上并行执行。

ES基本概念与关系型数据库的比较

| **ES概念**                                     | **关系型数据库**   |
| ---------------------------------------------- | ------------------ |
| Index（索引）支持全文检索                      | Database（数据库） |
| Type（类型）                                   | Table（表）        |
| Document（文档），不同文档可以有不同的字段集合 | Row（数据行）      |
| Field（字段）                                  | Column（数据列）   |
| Mapping（映射）                                | Schema（模式）     |

 

rpm --import http://packages.elasticsearch.org/GPG-KEY-elasticsearch

cat > /etc/yum.repos.d/elasticsearch.repo <<EOF

[elasticsearch] name=Elasticsearch repository for 8.x packages baseurl=https://artifacts.elastic.co/packages/8.x/yum gpgcheck=1 gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch enabled=0 autorefresh=1 type=rpm-md

EOF

yum install --enablerepo=elasticsearch elasticsearch //安装

 

 

 

二进制安装

https://www.elastic.co/cn/downloads/elasticsearch //elasticsearch下载地址

 

**报错解决方法**：

报错1、elasticsearch不能以root权限来运行，会出现这种错误：Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root。

因为安全问题elasticsearch 不让用root用户直接运行，所以要创建新用户解决办法：

第一步：liunx创建新用户 adduser XXX 然后给创建的用户加密码 passwd XXX 输入两次密码。 

第二步：切换刚才创建的用户 su XXX 然后执行elasticsearch 会显示Permission denied 权限不足。

第三步：root给esuser赋权限， chown -R esuser elasticsearch-8.1.2 kibana-8.1.2安装目录。

第四步：切换su esuser用户， bin/elasticsearch -d  //以守护进程运行

 

报错2、elasticsearch8.0.1之后无法访问9200:Empty reply from server

在文件conf/elasticsearch.yml将 xpack.security.enable: true 改为 false

 

更多报错查询网址：

https://www.linuxprobe.com/elasticsearch-install-tutorial.html

 

 

使用

curl 'http://localhost:9200/?pretty' 查询服务是否启动

 

查看健康状态

curl -X GET 127.0.0.1:9200/_cat/health?v

 

查询当前es集群中所有的indices

curl -X GET 127.0.0.1:9200/_cat/indices?v

 

创建索引

curl -X PUT 127.0.0.1:9200/www

 

删除索引

curl -X DELETE 127.0.0.1:9200/www

 

插入记录

192.168.1.3:9200/student/_doc/  //以_doc为类型

 

 

 

## kibana安装

需先安装ES

https://www.elastic.co/cn/downloads/kibana 下载kibana地址

rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch #下载并安装公共签名密钥

cat > /etc/yum.repos.d/kibana.repo <<EOF

[kibana-8.x] name=Kibana repository for 8.x packages baseurl=https://artifacts.elastic.co/packages/8.x/yum gpgcheck=1 gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch enabled=1 autorefresh=1 type=rpm-md

EOF

yum install kibana -y //安装

service kibana start && systemctl enable kibana

 

 

二进制安装

vi config/kibana.yml  //配置文件

 

server.port: 5601  //开放端口

server.host: "192.168.1.3"  //映射地址

server.publicBaseUrl: "http://172.31.240.57:5601"  # 这里地址改为你访问kibana的地址，不能以 / 结尾

i18n.locale: "zh-CN"  //支持中文

 

bin/kibana --allow-root  &//后台运行启动，以root用户运行

 

 

 

 

 

 

## kafka分布式消息队列

Kafka是一个分布式数据流平台，可以运行在单台服务器上，也可以在多台服务器上部署形成集群。它提供了发布和订阅功能，使用者可以发送数据到Kafka中，也可以从Kafka中读取数据(以便进行后续的处理)。

Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据，具有高性能、持久化、多副本备份、横向扩展等特点。

![img](E:\Project\Textbook\linux云计算\assets\wps23.png) 

 

### 一、Kafka集群的架构

Producer: 即生产者，消息的产生者，是消息的入口。

kafka cluster: kafka集群，-台或多台服务器组成

1、Broker: 指部署了Kafka实例的服务器节点（有唯一的标识）。每个服务器.上有一个或多个kafka的实例，我们姑且认为每个broker对应一台服务器。每个kafka集群内的broker都有-个不重复的编号，如图中的broker-0、broker-1等....

2、Topic: 消息的主题，可以理解为消息的分类，kafka的数据就保存在topic。在每个broker.上都可以创建多个topic。实际应用中通常是-个业务线建-个topic。

3、Partition: Topic的分区，每个topic可以有 多个分区，分区的作用是做负载，提高kafka的吞吐量。同一个topic在不同的分区的数据是不重复的，partition的表现形式就是一个-个的文件夹!

4、Replication:每- -个分区都有多个副本，副本的作用是做备胎。当主分区(Leader) 故障的时候会选择一个备胎(Follower). 上位， 成为Leader。在kafka中默认副本的最大数量是10个，且副本的数量不能大于Broker的数量，follower和leader绝对 是在不同的机器，同- -机器对同一个分区也只可能存放一个副本(包括自己)。

5、Consumer:消费者，即消息的消费方，是消息的出口。

Consumer Group:我们可以将多个消费组组成一个消费者组，在kafka的设计中同一个分区的数据只能被消费者组中的某一个消费者消费。同一个消费者组的消费者可以消费同-个topic的不同分区的数据，这也是为了提高kafka的吞吐量!

 

Leader:分区的主节点

Follower:分区的从节点

 

 

### 二、生产者往kafka发送数据的流程（6步）

![img](E:\Project\Textbook\linux云计算\assets\wps24.png) 

1.生产者从Kafka集群获取分区leader信息

2.生产者将消息发送给leader

3. leader将消息写入本地磁盘
4. follower从leader拉取消息数据
5. follower将消息写入本地磁盘后向leader发送ACK
6. leader收到所有的follower的ACK之后向生产者发送ACK

 

 

### 三、kafka选择分区的模式（3种）

1、指定往那个分区写

2、指定key，kafka根据key做hash然后决定写哪个分区

3、轮询方式

 

### 四、生产者往kafka发送数据的模式（3种）

`0`:把数据发给leader就成功，效率最高、安全性最低。

`1`:把数据发送给leader,等待leader回ACK

`a11` :把数据发给leader,确保follower从leader拉取数据回复ack给leader, leader再回复ACK;安全性最高

 

###最后要注意的是，如果往不存在的topic写数据，kafka会 自动创建topic, partition和replication的数量

默认配置都是1。

 

 

### 五、分区存储文件的原理

Topic和数据日志

topic是同一类别的消息记录(record) 的集合。在Kafka中，一个主题通常有多个订阅者。对于每个主题，Kafka集群维护 了一个分区数据日志文件结构如下:

![img](E:\Project\Textbook\linux云计算\assets\wps25.png) 

 

每个partition都是一个有序并 且不可变的消息记录集合。当新的数据写入时，就被追加到partition的末尾。在每个partition中，每条消息都会被分配- -个顺序的唯一标识， 这个标识被称为offset，即偏移量。注意，Kafka只保证在同一个partition内部消息是有序的，在不同partition之间， 并不能保证消息有序。

 

Kafka可以配置一个保留期限，用来标识日志会在Kafka集群内保留多长时间。Kafka集群会保留在保留期限内所有被发布的消息，不管这些消息是否被消费过。比如保留期限设置为两天，那么数据被发布到Kafka集群的两天以内，所有的这些数据都可以被消费。当超过两天，这些数据将会被清空，以便为后续的数据腾出空间。由于Kafka会将数据进行持久化存储(即写入到硬盘上)，所以保留的数据大小可以设置为一个比较大的值。

 

**Partition****结构**

Partition在服务器上的表现形式就是一个F 个的文件夹， 每个partition的文件夹 下面会有多组segment文件，每组segment文件又包含. index文件、.1og文件、 . timeindex文件三个文件，其中.1og文件就是实际存储message的地方，而. index和. time index文件为索引文件，用于检索消息。

 

### 六、为什么kafka快?

虽然是写入物理磁盘，但是每条记录都是通过index索引能快速定位

 

### 七、消费者组消费数据的原理

**消费数据**

多个消费者实例可以组成一个消费者组，并用一个标签来标识这个消费者组。一个消费者组中的不同消费者实例可以运行在不同的进程甚至不同的服务器上。如果所有的消费者实例都在同一个消费者组中，那么消息记录会被很好的均衡的发送到每个消费者实例。

如果所有的消费者实例都在不同的消费者组，那么每一条消息记录会被广播到每一个消费者实例。

![img](E:\Project\Textbook\linux云计算\assets\wps26.png) 

 

每个消费者实例可以消费多个分区,但是每个分区最多只能被消费者组中的一个实例消费。

 

 

 

一、需要jdk1.8.0_201环境

tar -zxf jdk1.8.0_201.tar# 解压文件

vim /etc/profile# 配置环境变量

 

### 在 profile 文件最后加上

export JAVA_HOME=/usr/local/java/jdk1.8.0_201

export PATH=$JAVA_HOME/bin:$PATH

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tool.jar

 

source /etc/profile# 使配置生效

### 最后输入 java 命令测试

 

二、安装使用 zookeeper (kafka 依赖 zookeeper，kafka内置zookeeper)

作用：查询kafka节点及进行连接

tar ‐zxvf apache‐zookeeper‐3.5.8‐bin.tar.gz# 解压文件

cp conf/zoo_sample.cfg conf/zoo.cfg# 复制一份配置文件, 方便修改

bin/zkServer.sh start# 启动

#使用内置kafka内置zk

bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

ss -pln | grep 2181 #查看进程端口状态

bin/zkCli.sh # 连接控制台

 

三、安装 kafka

wget **https://dlcdn.apache.org/kafka/3.1.0/kafka_2.12-3.1.0.tgz**

mv kafka-3.1.0-src.tgz /usr/local/

tar -xvf kafka-3.1.0-src.tgz

vim config/server.config# 修改配置文件 

 

broker.id=0# broker.id属性在kafka集群中必须要是唯一

listeners=PLAINTEXT://192.xxx.xx.xx:9092# kafka部署的机器ip和提供服务的端口号,切勿设0.0.0.0可能报错

log.dir=/usr/local/data/kafka‐logs# kafka的消息存储文件

zookeeper.connect=192.xxx.xx.xx:2181# kafka 连接 zookeeper 的地址

 

### 启动服务 , 运行的日志打印在 logs 目录里的server.log 里

1：bin/kafka‐server‐start.sh ‐daemon config/server.properties 

2：bin/kafka‐server‐start.sh config/server.properties &  # 后台启动，不会打印日志到控制台

 

四、使用

bin/kafka-console-producer.sh --bootstrap-server=192.168.1.3:9092 --topic=mysql_log

//生产者生产数据

bin/kafka-console-consumer.sh --bootstrap-server=192.168.1.3:9092 --topic=mysql_log --from-beginning

消费者读取kafka数据

--bootstrapver指定连接服务器		 --from-beginning开始读取

### 启动成功后,可以进入zookeeper 查看kafka节点

bin/zk.Cli.sh

ls /

 

### 停止kafka

bin/kafka‐server‐stop.sh

 

## zookeeper服务注册发现

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby- 个开源的实

现，它是集群的管理者，监视着集群中各个节点的状态根据节点提交的反馈进行下一步合理操作。最

终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。

 

 

 

 

## Cacti服务器监控

C/S模式，采集监控数据		B/S模式，管理监测平台

Cacti是一套基于PHP,MySQL,SNMP及RRDTool开发的网络流量监测图形分析工具。它通过snmpget来获取数据，使用 RRDtool绘画图形，而且你完全可以不需要了解RRDtool复杂的参数。它提供了非常强大的数据和用户管理功能，可以指定每一个用户能查看树状结构、host以及任何一张图，还可以与LDAP结合进行用户验证，同时也能自己增加模板，功能非常强大完善。Cacti 的发展是基于让 RRDTool 使用者更方便使用该软件，除了基本的 Snmp 流量跟系统资讯监控外，Cacti 也可外挂 Scripts 及加上 Templates 来作出各式各样的监控图。

cacti是用php语言实现的一个软件，它的主要功能是用snmp服务获取数据，然后用rrdtool储存和更新数据，当用户需要查看数据的时候用rrdtool生成图表呈现给用户。因此，snmp和rrdtool是cacti的关键。Snmp关系着数据的收集，rrdtool关系着数据存储和图表的生成。

Mysql配合PHP程序存储一些变量数据并对变量数据进行调用，如：主机名、主机ip、snmp团体名、端口号、模板信息等变量

snmp抓到数据不是存储在mysql中，而是存在rrdtool生成的rrd文件中（在cacti根目录的rra文件夹下）。rrdtool对数据的更新和存储就是对rrd文件的处理，rrd文件是大小固定的档案文件（Round Robin Archive），它能够存储的数据笔数在创建时就已经定义。关于RRDTool的知识请参阅RRDTool教学。

snmp(Simple Network Management Protocal, 简单网络管理协议)在架构体系的监控子系统中将扮演重要角色。大体上，其基本原理是，在每一个被监控的主机或节点上 (如交换机)都运行了一个 agent，用来收集这个节点的所有相关的信息，同时监听 snmp 的 port，也就是 UDP 161，并从这个端口接收来自监控主机的指令(查询和设置)。

如果安装 net-snmp，被监控主机需要安装 net-snmp(包含了 snmpd 这个 agent)，而监控端需要安装 net-snmp-utils，若接受被监控端通过trap-communicate发来的信息的话，则需要安装net-snmp，并启用trap服务。如果自行编译，需要 beecrypt(libbeecrypt)和 elf(libraryelf)的库。

RRDtool是指Round Robin Database 工具（环状数据库）。Round robin是一种处理定量数据、以及当前元素指针的技术。想象一个周边标有点的圆环－－这些点就是时间存储的位置。从圆心画一条到圆周的某个点的箭头－－这就是指针。就像我们在一个圆环上一样，没有起点和终点，你可以一直往下走下去。过来一段时间，所有可用的位置都会被用过，该循环过程会自动重用原来的位置。这样，数据集不会增大，并且不需要维护。RRDtool处理RRD数据库。它用向RRD数据库存储数据、从RRD数据库中提取数据。

工作原理：snmp关系着数据的收集，rrdtool关系数据存储和图表的生成，snmp抓取的数据不是存储在数据库中，而是存储在rrdtool生成的rrd文件中

yum install epel-release.noarch -y

yum -y install httpd mariadb* php php-mysql zlib freetype libjpeg fontconfig libxml2 gd php-gd rrdtool net-snmp net-snmp-utils

yum install cacti -y

systemctl start httpd snmpd mariadb

cat> /var/www/html/index.php <<EOF

<?php

phpinfo();

?>

EOF

 

## Nagios监控系统

Nagios是一款开源的电脑系统和网络监视工具，能有效监控Windows、Linux和Unix的主机状态，交换机路由器等网络设备，打印机等。在系统或服务状态异常发出邮件或短信报警第一时间通知网站运维人员，在状态恢复后发出正常的邮件或者短信通知。

主要功能：  

网络服务监控（SMTP，POP3，HTTP，NNTP，ICMP，SNMP，FTP，SSH）

主机资源监控（CPU load，disk usage，system logs），也包括Windows主机（使用NSCLient+plugin）

可以指定自己编写的Plugin通过网络收集数据来监控任何情况（温度，警告。。。）

可以通过配置Nagios远程执行插件，远程执行脚本

远程监控支持ssh或ssl加通道方式进行监控

简单的plugin设计允许用户很容易的开发自己需要的检查服务，支持多开发语言（shell script，c++，Perl，Ruby，python，PHP，c#等）

包含很多图形化数据plugins（Nagiosgraph，Nagiosgrapher，PNP4Nagios等）

可并行服务检查

能够定义网络主机的层次，允许逐级检查，就是从父主机开始向下检查

当服务或主机出现问题时发出通告，可以通过email，pager，sms或任意用户自定义的plugin进行通知

能够自定义事件处理机制重新激活出问题的服务或主机

自动日志循环

支持冗余监控

包括web界面可以查看当前网络状态，通知，问题历史，日志文件等。

Centos7安装步骤（RHEL不同）：

setenforce 0 #关闭selinux

yum install -y gcc glibc glibc-common wget unzip httpd php gd gd-devel perl postfix #安装依赖软件库

wget https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.5.tar.gz #下载软件包

tar -xzvf nagioscore.tar.gz #进行解压

./configure  --prefix=/usr/local/nagios  --with-command-group=nagios && make all #指定安装路径和指定组

make install-groups-users #创建nagios用户和组

usermod -a -G nagios apache #将apache用户添加到所述的nagios组

make install #安装二进制文件，CGI和HTML文件

make install-daemoninit #安装守护程序文件

systemctl enable httpd.service

make install-commandmode #安装并配置外部命令文件

make install-config #安装* SAMPLE *配置文件，因为Nagios需要一些配置文件才能启动

make install-webconf #安装Apache Web 服务器配置文件

firewall-cmd --zone=public --add-port=http --permanent #允许本地防火墙通过http

htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin #创建一个Apache用户帐户才能登录Nagios

systemctl start httpd nagios #启动web服务和nagios服务

http://10.25.5.143/nagios #将您的Web浏览器指向Nagios Core 服务器的IP地址或FQDN

安装nagios-plugins插件，安装依赖包：

yum install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils epel-release 

yum install -y perl-Net-SNMP

下载源：wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.2.1.tar.gz 

解压：tar -zxvf nagios -plugins.tar.gz

编译安装：cd /nagios-plugins-release-2.2.1/ 

./tools/setup 

./configure --prefix=/usr/local/nagios-plugins  --with-command-group=nagios

make && make install && systemctl restart nagios.service

 

https://github.com/NagiosEnterprises/nrpe/releases/download/nrpe-4.0.2/nrpe-4.0.2.tar.gz

## Zabbix集中监控系统

一种网络监视、管理系统，基于 Server-Client 架构。可用于监视各种网络服务、服务器和网络机器等状态

使用各种 Database-end 如 MySQL, PostgreSQL, SQLite, Oracle 或 IBM DB2 储存资料。Server 端基于 C语言、Web 管理端 frontend 则是基于 PHP 所制作的

Zabbix 也可以经由 SNMP、TCP、ICMP利用 IPMI、SSH、telnet 对目标进行监视。包含 XMPP 等各种 Item 警示功能

![img](E:\Project\Textbook\linux云计算\assets\wps27-1682690463422-273.jpg) 

zabbix监控范畴：硬件监控 ：Zabbix IPMI Interface				系统监控 ：Zabbix Agent Interface

Java 监控：ZabbixJMX Interface				网络设备监抟：Zabbix SNMP Interface

应用服务监控：Zabbix Agent UserParameter		MySQL 数据库监控：percona-monitoring-pldlgins

URL监控：Zabbix Web监控

#!/bin/bash

#服务端安装脚本，关闭防火墙，关闭selinux

systemctl stop firewalld && setenforce 0

#配置yum源

rpm -ivh http://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm && yum repolist

```
if [ -e /etc/yum.repos.d/zabbix.repo ];then
​	echo "已存在"
​	yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent mariadb mariadb-server
​	else
​	echo "不存在"
​	exit
fi

systemctl start mariadb

if [ $? -eq 0 ];then
​	echo "service is started"
​	else
​	echo "service not started"
fi
```

#数据库的操作

mysql -e 'create database zabbix character set utf8 collate utf8_bin;'

#授权

mysql -e 'grant all privileges on zabbix.* to zabbix@localhost identified by "zabbix";'

#导入初始数据库

zcat `find / -name zabbix-server-mysql-*`/create.sql.gz | mysql -uzabbix -pzabbix zabbix

#修改配置文件

sed -i 's/# DBPassword=/DBPassword=zabbix/' /etc/zabbix/zabbix_server.conf

#编辑php文件

sed -i 's#;date.timezone =#date.timezone = Asia/Shanghai#' /etc/php.ini

#启动服务

systemctl start httpd zabbix-agent zabbix-server

#解决中文乱码，\cp强制覆盖且不提示

yum -y install wqy-microhei-fonts

\cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf

#输出信息

echo "浏览器访问 http://`hostname -I|awk '{print $1}'`/zabbix"

echo "登陆界面(区分大小写) 账号Admin密码zabbix"

 

#!/bin/bash

#zabbix客户端快速安装脚本，安装zabbix源

rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm

yum clean all && yum -y install zabbix-agent

#修改Master为节点地址，ServerActive为被动接收监控

sed -i.brk "s/Server=127.0.0.1/Server=192.168.3.5/g" /etc/zabbix/zabbix_agentd.conf

sed -i "s/ServerActive=127.0.0.1/ServerActive=192.168.3.5/g" /etc/zabbix/zabbix_agentd.conf

#修改该node机的主机名，在添加被监控时使用的

sed -i "s/Hostname=Zabbix server/Hostname=zabbix node1/g" /etc/zabbix/zabbix_agentd.conf 

#开机自启服务

systemctl start  zabbix-agent.service && systemctl enable  zabbix-agent.service && service firewalld stop

echo 本机IP:`hostname -I|awk '{print $1}'`

 

操作：添加主机（手动）：点击配置—点击主机—点击创建主机—根据提示填写主机名称，选择群组，填写agent代理程序的接口(被监控主机ip)—点击添加

监控图形：只能进行单一的监控

监控聚合图形：同时监控多个主机

监控分类：

创建模板：配置 >> 模板 >> 创建模板

创建应用集：应用集类似(目录/文件夹)，把需要监控的一类划分到同一应用集

创建监控项：key+参数组成，如：net.tcp.service[http]（一个zabbix服务端监控客户端的脚本）

![img](E:\Project\Textbook\linux云计算\assets\wps28-1682690463422-274.jpg) 

用户自定义监控项格式：UserParameter=<key键值>,<shell command命令>		#key名字要唯一

范例：echo UserParameter=login-user,who|wc -l >>/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf

测试：zabbix_get -s 172.16.1.21 -p 10050 -k "login-user"

创建触发器：当监控项获取到的值达到一定条件时就触发报警

自定义名称，该名称是报警时显示的名称			表达式，通过监控项表示		自定义严重性

![img](E:\Project\Textbook\linux云计算\assets\wps29-1682690463422-275.jpg) 

使用zabbix自带模板：如Template OS Linux (Template App Zabbix Agent)提供CPU、内存、磁盘、网卡等常规监控

![img](E:\Project\Textbook\linux云计算\assets\wps30-1682690463422-276.jpg) 

创建动作：即触发报警后执行的动作，有持续时间，消息内容

创建报警媒介：即定义发送报警信息的方式，如mail，管理>>报警媒介类型

![img](E:\Project\Textbook\linux云计算\assets\wps31-1682690463422-277.jpg) 

Web控制台(rhel8对浏览器有要求)

用于管理和监视本地系统以及位于网路环境中的linux服务器，是交互式界面，通过浏览器与操作系统交互

作用：监控基本系统功能,例如硬件信息,时间配置,性能配置等		检查系统日志文件

管理网络接口和配置防火墙		管理虚拟机		管理用户帐户	监视和配置系统服务	管理软件包	

配置Selinux·更新软件			访问终端

服务名和包:cockpit			端口号port:9090

systemctl list-unit-files列出系统所有的服务

web控制台登录账号认证文件/etc/pam.d/cockpit【默认允许系统上所有用户登录】

buffer与cache

buffer：缓冲也叫写缓冲,一般用于写操作,可以将数据先写入内存在写λ磁盘, buffer一般用于写缓冲,用于解决不同介质的速度不一致的缓冲,先将数据临时写入到里自己最近的地方,以提高写入谏度,CPU会把数据先写到内存的磁盘缓冲区,然后就认为数据已经写入完成看,然后由内核在后续的时间在写入磁盘,所以服务器突然断电会丢失内存中的部分数据。

cache：缓存也叫读缓存,一般用于读操作,CPU读文件从内存读,如果内存没有就先从硬盘诪到内存再读到CPU,将需要频繁读取的数据放在里自己最近的缓存区域,下次读取的时候即可快速读取