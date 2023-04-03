**Ubuntu22.04 安装Mongodb6.X**
1、Mongodb简介
1.1 什么是MongoDB?
Mongodb是一个跨平台的面向文档的NoSQL数据库。它使用带有可选模式的类似JSON的BSON来存储数据。应用程序可以以JSON格式检索信息。
1.2 MongoDB的优点
可以快速开发web型应用，因为灵活，不用像关系型数据库一样需要建表
MongoDB存储的是文档(document)，文档内存储的是类似json的结构，所谓json就是字符串数组
1.3 MongoDB的数据库分类
数据库(database)：用来存储集合的，而且数据库也有分大小
集合(collection)：集合类似于数组，用于存放文档的
文档(document)：文档是MongoDB数据库中最小的单位
![image-20230329213132191](E:\Project\Textbook\assets\image-20230329213132191.png)



MongoDB关系: 数据库(database) > 集合(collection)> 文档(document)

2、安装mongodb
2.1 导入MongoDB6.0版的公钥

```shell
root@Mongodb:~# curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
```

2.2 更新apt资源库

```shell
root@Mongodb:~# apt update
```

2.3 创建列表文件

```shell
root@Mongodb:~# echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list

root@Mongodb:~# apt update
```

2.4 安装MongoDB的依赖libssl1.1

```shell
root@Mongodb:~# curl -LO http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1-1ubuntu2.1~18.04.21_amd64.deb
root@Mongodb:~# dpkg -i libssl1.1_1.1.1-1ubuntu2.1~18.04.21_amd64.deb
```

2.5 安装mongodb

````shell
root@Mongodb:~# apt-get install -y mongodb-org`
````

#启动MongoDB服务

```shell
root@Mongodb:~# systemctl start mongod
```

#检查MongoDB服务状态

```shell
root@Mongodb:~# systemctl status mongod |grep active
     Active: active (running) since Tue 2023-02-21 16:54:50 CST; 10s ago
```

#设置服务开机自启动

```shell
root@Mongodb:~# systemctl enable mongod
```


