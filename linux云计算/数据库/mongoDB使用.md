# Ubuntu22.04 安装Mongodb6.X
# Mongodb简介
**什么是MongoDB?**

Mongodb是一个跨平台的面向文档的NoSQL数据库。它使用带有可选模式的类似JSON的BSON来存储数据。应用程序可以以JSON格式检索信息。

**MongoDB的优点**

可以快速开发web型应用，因为灵活，不用像关系型数据库一样需要建表

MongoDB存储的是文档(document)，文档内存储的是类似json的结构，所谓json就是字符串数组

**MongoDB的数据库分类**

- 数据库(database)：用来存储集合的，而且数据库也有分大小
- 集合(collection)：集合类似于数组，用于存放文档的
- 文档(document)：文档是MongoDB数据库中最小的单位

<img src="E:\Project\Textbook\assets\image-20230329213132191.png" alt="image-20230329213132191" style="zoom:50%;" />

MongoDB关系: 数据库(database) > 集合(collection)> 文档(document)

# mongoDB安装

我们这里下载和安装社区版，[官网下载地址](https://www.mongodb.com/download-center/community)。 打开上述连接后，选择对应的版本、操作系统平台（常见的平台均支持）和包类型，点击Download按钮下载即可。

这里补充说明下，Windows平台有`ZIP`和`MSI`两种包类型: * ZIP：压缩文件版本 * MSI：可执行文件版本，点击”下一步”安装即可。

macOS平台除了在该网页下载`TGZ`文件外，还可以使用`Homebrew`安装。

更多安装细节可以参考[官方安装教程](https://docs.mongodb.com/manual/administration/install-community/)，里面有`Linux`、`macOS`和`Windows`三大主流平台的安装教程。

## 导入MongoDB6.0版的公钥

```shell
root@Mongodb:~# apt updateroot@Mongodb:~# curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add //-更新apt资源库
root@Mongodb:~# apt update
```

## 创建列表文件

```shell
root@Mongodb:~# echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list

root@Mongodb:~# apt update
```

## 安装MongoDB的依赖libssl1.1

```shell
root@Mongodb:~# curl -LO http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1-1ubuntu2.1~18.04.21_amd64.deb
root@Mongodb:~# dpkg -i libssl1.1_1.1.1-1ubuntu2.1~18.04.21_amd64.deb
```

## 安装mongodb

````shell
root@Mongodb:~# apt-get install -y mongodb-org
root@Mongodb:~# systemctl start mongod //启动MongoDB服务
root@Mongodb:~# systemctl status mongod |grep active  //检查MongoDB服务状态
     Active: active (running) since Tue 2023-02-21 16:54:50 CST; 10s ago
root@Mongodb:~# systemctl enable mongod //设置服务开机自启动
````

# mongoDB基本使用

## 数据库常用命令

`show dbs;`：查看数据库

```bash
> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```

`use q1mi;`：切换到指定数据库，如果不存在该数据库就创建。

```bash
> use q1mi;
switched to db q1mi
```

`db;`：显示当前所在数据库。

```bash
> db;
q1mi
```

`db.dropDatabase()`：删除当前数据库

```bash
> db.dropDatabase();
{ "ok" : 1 }
```

## 数据集常用命令

`db.createCollection(name,options)`：创建数据集

- name：数据集名称
- options：可选参数，指定内存大小和索引。

```bash
> db.createCollection("student");
{ "ok" : 1 }
```

`show collections;`：查看当前数据库中所有集合。

```bash
> show collections;
student
```

`db.student.drop()`：删除指定数据集

```bash
> db.student.drop()
true
```

## 文档常用命令

插入一条文档：

```bash
> db.student.insertOne({name:"小王子",age:18});
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5db149e904b33457f8c02509")
}
```

插入多条文档：

```bash
> db.student.insertMany([
... {name:"张三",age:20},
... {name:"李四",age:25}
... ]);
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5db14c4704b33457f8c0250a"),
		ObjectId("5db14c4704b33457f8c0250b")
	]
}
```

查询所有文档：

```bash
> db.student.find();
{ "_id" : ObjectId("5db149e904b33457f8c02509"), "name" : "小王子", "age" : 18 }
{ "_id" : ObjectId("5db14c4704b33457f8c0250a"), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("5db14c4704b33457f8c0250b"), "name" : "李四", "age" : 25 }
```

查询age>20岁的文档：

```bash
> db.student.find(
... {age:{$gt:20}}
... )
{ "_id" : ObjectId("5db14c4704b33457f8c0250b"), "name" : "李四", "age" : 25 }
```

更新文档：

```bash
> db.student.update(
... {name:"小王子"},
... {name:"老王子",age:98}
... );
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.student.find()
{ "_id" : ObjectId("5db149e904b33457f8c02509"), "name" : "老王子", "age" : 98 }
{ "_id" : ObjectId("5db14c4704b33457f8c0250a"), "name" : "张三", "age" : 20 }
{ "_id" : ObjectId("5db14c4704b33457f8c0250b"), "name" : "李四", "age" : 25 }
```

删除文档：

```bash
> db.student.deleteOne({name:"李四"});
{ "acknowledged" : true, "deletedCount" : 1 }
> db.student.find()
{ "_id" : ObjectId("5db149e904b33457f8c02509"), "name" : "老王子", "age" : 98 }
{ "_id" : ObjectId("5db14c4704b33457f8c0250a"), "name" : "张三", "age" : 20 }
```

命令实在太多，更多命令请参阅[官方文档：shell命令](https://docs.mongodb.com/manual/mongo/)和[官方文档：CRUD操作](https://docs.mongodb.com/manual/crud/)。
