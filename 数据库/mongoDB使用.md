# ==MongoDB==

官方文档：https://docs.mongodb.com/manual/mongo/

中文社区：https://mongoing.com

官方中文文档：https://docs.mongoing.com

中文文档：https://www.runoob.com/mongodb/mongodb-databases-documents-collections.html

# 简介

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

# 基本操作

### 启动服务

```bash
#fork 后台运行  若要使用账号密码服务 需要在启动时加 --auth
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork
#不在后端运行，可以在控制台上查看运行过程，使用配置文件启动
mongod --config /usr/local/etc/mongod.conf

//关闭方法db.shutdownServer()  之后exit
需要账号密码的登陆方式：
方法一：
mongo 服务器IP地址：mongo 127.0.0.1/admin -u admin -p 123456
方法二：
mongo进入shell
use admin
db.auth('admin','123456') 返回1则验证成功
```

### 查看服务信息

```
ps aux | grep -v grep | grep mongod
```

### 查看数据库

show databses

### 选择数据库

use database[数据库名称]   选择不存在的数据库会隐式创建该数据库

### 删除数据库

先选择到要删除的数据库 db.dropDatabase



### 查看集合

show collections

### 创建集合

db.createCollection('c1')

### 删除集合

db.集合名.drop()

### ID的组成：

0 1 2 3 4 5 6 7 8 9 10 11

0-3:时间戳  4-6：机器mac码  7-8:PID 9-11:计数器

也可以自定义ID，只需要给插入的JSON数据增加_id键即可覆盖（强烈不推荐）



# 增删改查

## C增

db.集合名.insert(JSON数据)

集合存在，则直接插入数据，集合不存在，隐式创建并插入

```bash
use test2
db.c1.insert({username:"xzy",age:18})
db.c1.insertMany({})
留心一：数据库和集合不存在时，都隐式创建
留心二：对象的键同意不加引号方便看，但是查看集合数据时系统会自动加上
留心三：mongodn会给每条数据添加一个全球唯一的ID

插入多条数据：
传递数组，数组中每个元素都是一个JSON类型
db.c1.insert([
  {username:"z3",age:3},
  {username:"z4",age:4},
  {username:"z5",age:5}
])

插入N条数据：
mongodb底层使用JS引擎实现的，所以支持部分js语法，可以使用for循环
for(var i=1;i<=10;i++){
  print(i)
}

插入十条数据：
  
for(var i=1;i<=10;i++){
  db.c1.insert({username:"a"+i,age:i})
}
```



## R查

语法： db.集合名.find(条件，[查询的列])

格式化：db.集合名.find().pretty()

```bash
条件：
	查询所有数据  				{}或者不写
	查询age=6的数据 			 {age:6}
	查询age=6且性别为男   {age:6,sex:'男'}

查询的列
	不写 - 查询全部的列
	{age:1} 只显示age列，可以显示多个想要的列{user:1,age:1.......} 
	{age:0} 除了age列外都显示 可以不显示多个想要的列{user:0,age:0}
	无论怎么写系统自定义_id都会在
```

升级语法：

```bash
db.集合名.find(键：值)  注：值不直接写

							{运算符：值}

db.集合名.find({

	键：{运算符：值}

})


例如：
年龄小于5的
db.c1.find({age:{$lt:5}})

年龄等于3、4、5的
db.c1.find({age:{$in:[3,4,5]}})
```

运算符表：

| 运算符 | 作用     |
| ------ | -------- |
| $gt    | 大于     |
| $gte   | 大于等于 |
| $lt    | 小于     |
| $lte   | 小于等于 |
| $ne    | 不等于   |
| $in    | in       |
| $nin   | Not in   |





## U改

基础语法： db.集合名.update（条件，新数据[,是否新增，是否修改多条]）

```bash
是否新增：指条件匹配不到数据则插入，true是插入，false否不插入默认
是否修改多条：指将匹配成功的数据都修改（true是，false否默认）

for(var i=1;i<=10;i++){
	db.c3.insert({username:"zs"+i,age:i});
}


db.c3.update({username:"zs1"},{username:"zs2"})#这样是替换，将符合条件的行直接换成这个
```



升级语法：

```bash
db.c3.update（{username:"zs2"}，{$set:{username:"zs222"}}）	

给zs10 增加2岁
db.c3.update({username:"zs10"},{$inc:{age:2}})
给zs10 减少2岁
db.c3.update({username:"zs10"},{$inc:{age:-2}})

准备：插入一个数据： db.c4.insert({username:"熊子阳",age:18,who:"男",other:"没钱"})

修改数据，将 熊子阳 改为 Aoi  ，age 改为999 ，who 改为 sex ，other 删除
db.c4.update({username:"熊子阳"},
{$set:{username:"Aoi"}},
{$inc:{age:971}},
{$rename:{who:sex}},
{$unset:{other:true}})

正确写法：
db.c4.update({username:"熊子阳"},{
	$set:{username:"Aoi"},
	$inc:{age:971},
	$rename:{who:"sex"},
	$unset:{other:true}
})

#更新不存在的值,若不存在则不会有操作
> db.c3.update({username:"zs30"},{$set:{age:30}})
#在最后加一个true参数，作用是，如果不存在，则插入该条数据,默认为false则不管
> db.c3.update({username:"zs30"},{$set:{age:30}},true)

#第四个参数如果为true，当匹配到多条条件符合的元素时，都更改，默认为false，只改一条
> db.c3.update({},{$set:{age:20}},false,true)
```



| 运算符  | 作用     |
| ------- | -------- |
| $inc    | 递增     |
| $rename | 重命名列 |
| $set    | 修改列值 |
| $unset  | 删除列   |



## D删

语法：db.集合名.remove(条件[,是否删除一条])

是否删除一条 true是，false否 默认

```js
当存在多条符合条件的行时，只删除一条
db.c3.remove({username:"zs30"},true)
存在多条时，全部删除
db.c3.remove({username:"zs30"},true)
```





# try catch

当一次性插入或者更新n条数据时，mongodb不会因为一条数据的错误而使得整个操作终止并回滚，只会终止接下来的操作，所以可以使用trycatch来进行异常的捕捉处理。测试的时候可以不处理。

例如：

```json
try{
  db.c1.insertMany([
  {"_id":1,name:"xzy"},
  {"_id":2,name:"lhl"},
	{"_id":3,name:"yzh"},
	{"_id":4,name:"lwy"}...
}
  ])
}catch(e){
	print
}
```





# 排序&分页

准备：

```bash
use test
db.c2.insert({_id:1,name:"a",sex:1,age:1})
db.c2.insert({_id:2,name:"b",sex:2,age:2})
db.c2.insert({_id:3,name:"c",sex:3,age:3})
db.c2.insert({_id:4,name:"d",sex:4,age:4})
db.c2.insert({_id:5,name:"e",sex:5,age:5})
```

### 排序

```bash
语法：db.集合名.find().sort(JSON数据)
说明：键-就是要排序的列/字段，值：1升序 -1降序
使用：对年龄进行降序排序
db.c2.find().sort({age:-1})
```

### 分页

```bash
语法：db.集合名.find().skip(数字).limit(数字)
说明：skip里的数字指跳过指定数量（可选），limit限制查询的数量
db.c2.find().sort({age:-1}).skip(1).limit(2)
```



# 聚合查询

语法：

```bash
db.集合名称.aggregate([
	{管道:{表达式}}
	....
])

常用管道：
$group   将集合中的文档分组，用于统计结果
$match   过滤数据，只要输出符合条件的文档
$sort		 聚合数据进一步排序
$skip    跳过指定文档数
$limit   限制集合数据返回文档数
....

常用表达式：
$sum 总和 $sum:1同count表示统计
$avg 平均
$min 最小值
$max 最大值

准备：
db.c3.insert({_id:1,name:"a",sex:1,age:1})
db.c3.insert({_id:2,name:"a",sex:1,age:2})
db.c3.insert({_id:3,name:"b",sex:1,age:3})
db.c3.insert({_id:4,name:"c",sex:2,age:4})
db.c3.insert({_id:5,name:"d",sex:2,age:5})

操作：
男女生的总年龄
#_id 必须加，后跟指定列
#rew 求和 返回结果数
db.c3.aggregate([
	{
		$group:{	
			_id:"$sex", 			
			res:{$sum:"$sex"}   
	}
	}
])

求男女总人数
db.c3.aggregate([
	{
		$group:{	
			_id:"$sex", 			
			res:{$sum:1}   
	}
	}
])

求学生总数和平均年龄
db.c3.aggregate([
	{
		$group:{	
			_id:null, 			
			res:{$sum:1},
      total_avg:{$avg:"$age"}
	}
	}
])

查询男生女生人数，升序排序
db.c3.aggregate([
	{$group:{	_id:"$sex",res:{$sum:1}}},
	{$sort:{res:1}}
])
```



# 优化索引

### 基本操作

创建索引语法： db.集合名.createIndex(待创建索引的列[,额外选项])

参数：

> 待创建索引的列:{键:1,...,键:-1}
>
> 说明：1升序  -1降序 列入{age：1}表示创建age索引并按照升序的方式存储
>
> 额外选项：设置索引的名称或者唯一索引等等

```bash
#创建只对单个列为条件的索引
db.c1.create({name:1})
#创建一个自己取名的索引
db.c1.create({name:1},{name:"xzy"})
#创建条件为多个列的组合索引
db.c1.create({name:1,age:-1},{"hh"})

#创建唯一索引
db.c1.createIndex({name:1},{unique:"name"})
```



删除索引语法：

>全部删除：db.集合名.dropIndexes()
>
>删除指定：db.集合名.dropIndex(索引名)
>
>

查看索引语法：db.集合名.getIndexes()



### 分析索引

语法：db.集合名.find().explain('executionStats')

说明：

COLLSCAN 全表扫描

IXSCAN索引扫描

FETCH根据索引去检索指定document



# 权限机制

**开启验证模式概念：**指用户需要输入账号密码才能登陆使用

操作步骤：

```bash
1、添加超级管理员
2、退出卸载服务
3、重新安装需要输入账号密码的服务（注在原安装命令基础上加上--auth即可）
4、启动服务-〉登陆测试
```

步骤一：添加超级管理员

```sql
use admin
db.createUser({
              "user":"admin",
              "pwd":"123456",
             	"roles":[{
                       role:"root",
                       db:"admin"
                       }]
              })
              
查看管理员
use admin
show collections
db.system.users.find().pretty()
```

步骤二：退出卸载服务

```bash
//关闭方法db.shutdownServer()  
之后exit
```

步骤三：安装需要验证的MongoDB服务

```bash
#使用--auth参数来开启认证服务
mongod --dbpath /usr/local/var/mongodb --logpath /usr/local/var/log/mongodb/mongo.log --auth --fork

#或在config文件中添加上
security:
	#开启授权认证后再用配置文件来启动即可
	authorization:enabled

需要账号密码的登陆方式：
方法一：
mongo 服务器IP地址：mongo 127.0.0.1/admin -u admin -p 123456
方法二：
mongo进入shell
use admin
db.auth('admin','123456') 返回1则验证成功
```

例子：

```bash
for(var i = 1;i<=10;i++){
db.goods.insert({"name":"goodsName"+i,"price":i})
}
添加用户shop1可以读shop数据库
db.createUser({
              "user":"shop1",
              "pwd":"123456",
             	"roles":[{
                       role:"read",
                       db:"shop"
                       }]
              })
添加用户shop2可以读写shop数据库
db.createUser({
              "user":"shop2",
              "pwd":"123456",
             	"roles":[{
                       role:"readWrite",
                       db:"shop"
                       }]
              })
```



# 备份还原

语法：

```bash
在终端中执行，该命令不是mongo命令
导出：mongodump -h -port -u -p -d -o
导出语法说明
-h host 服务器IP地址（一般不写 默认本机）
-port port 端口（不写默认27017）
-u user 用户
-p pwd 密码
-d database 数据库（不写默认导出全部）
-o open 备份到指定目录下

mongodump -u admin -p 123456 -o /Users/didi/xzy文件/mongo
#注意 最新的mongodb版本4.4中，是没有mongodump工具的，需要通过使用brew命令单独下载
#brew install mongodb-database-tools

单独备份一个指定数据库：
mongodump -u shop2 -p 123456 -d shop -o /Users/didi/xzy文件/mongo
#此时好像不能使用admin作为用户来备份，可能是因为这个不是创建在shop中的用户？
```



# Mongoose

官方：http://mongoosejs.com

中文：http://mongosejs.net/

是node中提供操作MongoDB的模块

能过通过Node语法实现MongoDB数据库CURD

从而实现使用node写程序

下载：

```bash
npm i mongoose
或者
yarn add mongoose
```





# Java使用



## 添加依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
    </dependencies>
```

## 配置属性文件（appliction.yml）

```yml
spiring:
	#数据源配置
	data:
		mongodb:
			#主机地址
			host: locahost
			#数据库
			database: test
			#默认端口时27017
			port:	27017
			#也可以使用url连接
			#url:mongodb://localhost:27017/test
```

## 添加数据

```json
db.comment.insert({
  _id:'1',
  content:"我们不应该吧清晨浪费在手机上，哈哈",
  pulishtime :null,
  userid:'1002',
  nickname:"Aoi"
})
```



## 创建分页

在CommentReposity中添加方法

```java
public interface CommentRepository extends MongoRepository<Comment,String>{
  //方法名根据已有字段来设置，Mongo会提示，拼写错误则无法使用
  //第一个参数是查询条件，第二个是分页
  Page<Comment> findByParentid(String parentid,Pageable pageable);
}
```

在Service中添加该方法

```java
public Page<Comment> findCommentListByParentid(String parentid,int page,int size){
  //之所以-1是因为索引从0开始  
  return commentRepository.findByParentid(parentid, PageRequest.of(page-1,size));
    }
```



## 实现点赞

在Service中新增updateThumbup方法

```java
/*
点赞-效率低
@param id
*/
public void updateCommentThumbupToIncrementingOld(String id){
  Comment comment = commentRepository.findById(id).get();
  comment.setLikenum(comment.getLikenum()+1);
  CommentRepository.save(comment);
}
```



以上方法效率不高，只需要将点赞数+1就可以，没必要查出所有字段以后再更新所有字段。



可以使用MongoTemplate类来实现对某列的操作

（1）修改CommentService

```java

//注入MongoTemplate
@Autowired
private MongoTemplate mongoTemplate;

public void updateCommentLikenum(String id){
  //查询对象
  Query query = Query.query(Criteria.where("_id"),is(id));
  //更新对象
  Update update = new Update();
	//局部更新，相当于$set
  //update.set(key,value)
  //递增$inc
  // update.inc("likenum",1)
  update.inc("likenum");
  
  //参数1:查询对象
  //参数2:更新对象
  //参数3:集合的名字或实体类的类型Comment.class
  mongoTemplate.updateFirst(query,update,"comment");
}
```



# ==MongoDB集群和安全==



# 副本集

是一组维护相同数据集的Mongod服务，副本集可以提供冗余和高可用性，是所有生产部署的基础。

同时也是类似于有自动故障恢复功能的主从集群。用多台机器进行同一数据的异步和同步，从而使得多台机器拥有同一数据的多个副本。并且当主库宕机时不需要用户敢于的情况下自动切换其他备份服务器做主库。还可以利用副本服务器做只读服务器，实现读写分离，提高负载。



（1）冗余和数据可用性

复制提供冗余并提高数据可用性。通过在不同数据库服务器上提供多个数据副本，复制可提高一定级别的容错功能，以防止丢失单个数据库服务器。

某些情况下，复制可以提供增加的读取性能，因为客户端可以将读取操作发送到不同的服务上，在不同数据中心维护数据副本可以增加分布式应用程序的数据位置和可以性。还可以用于维护其他副本，灾难恢复，报告或者备份。

（2）复制

副本集时一组维护相同数据集的mongod实例。包含多个数据承载节点和可选的一个仲裁节点。在承载数据的节点中，一个且仅有一个成为被视为主节点，其他节点被视为次要节点。

主节点接收所有写操作，副本集只有一个主要能够确认具有{w:"most"}写入关注的写日；虽然某些情况下，另一个mongod实例可能暂时认为自己也是主要的。主要记录其操作日志中的数据集的所有更改，即oplog。

（3）主从复制和副本集区别

主从集群和副本集最大区别就是副本集没有固定的"主节点"；整个集群会选出一个"主节点"，当其挂掉后，又在剩下的从节点中选中其他节点为"主节点"，副本集总有一个活跃点{主、primary}和一个或多个备份节点{从、secondary}.



副本集的三个角色

副本集有两种数据类型三个角色

两种类型：

- 主节点（primary）类型：数据操作的主要连接点，可读写。
- 次要（辅助、从）节点类型：数据冗余备份节点，可以读或选举。

三种角色：

主要成员（primary）：主要接收所有写操作。就是主节点。

副本成员（Replicate）：从主节点通过复制操作以维护相同的数据集，即数据备份，不可写操作，但可以读操作（但需要配置）。是默认的一种从节点类型。

仲裁者（Arbiter）：不保留任何数据的副本，只具有投票选举作用。当然也可以将仲裁服务器维护为副本集的一部分，即副本成员同时也可以是仲裁者。也是一种从节点类型。



# 搭建副本集

一主一从一仲裁。

## 主节点

建立存放数据和日志的目录

```bash
#---------myrs
#主节点
mkdir -p /Users/didi/xzy/replica_sets/myrs_27017/log
mkdir -p /Users/didi/xzy/replica_sets/myrs_27017/data/db
```

新建或修改配置文件：

```bash
vim /Users/didi/xzy/replica_sets/myrs_27017/mongod.conf
```

myrs_27017:

```yaml
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/replica_sets/myrs_27017/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/replica_sets/myrs_27017/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/replica_sets/myrs_27017/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27017
replication:
  #副本集的名称
  replSetName: myrs
```

启动节点服务：

```bash
mongod -f /Users/didi/xzy/replica_sets/myrs_27017/mongod.conf

mongod --dbpath /Users/didi/xzy/replica_sets/myrs_27017/data/db --logpath /Users/didi/xzy/replica_sets/myrs_27017/log/mongod.log --fork
```



## 从节点

建立存放数据和日志的目录

```shell
#---------myrs
#从节点
mkdir -p /Users/didi/xzy/replica_sets/myrs_27018/log
mkdir -p /Users/didi/xzy/replica_sets/myrs_27018/data/db
```

新建或修改配置文件：

```shell
vim /Users/didi/xzy/replica_sets/myrs_27018/mongod.conf
```

myrs_27018:

```shell
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/replica_sets/myrs_27018/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/replica_sets/myrs_27018/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/replica_sets/myrs_27018/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27018
replication:
  #副本集的名称
  replSetName: myrs
```

启动服务

```shell
mongod -f /Users/didi/xzy/replica_sets/myrs_27018/mongod.conf

mongod --dbpath /Users/didi/xzy/replica_sets/myrs_27018/data/db --logpath /Users/didi/xzy/replica_sets/myrs_27018/log/mongod.log --fork
```



## 仲裁节点

建立存放数据和日志的目录

```shell
#---------myrs
#从节点
mkdir -p /Users/didi/xzy/replica_sets/myrs_27019/log
mkdir -p /Users/didi/xzy/replica_sets/myrs_27019/data/db
```

新建或修改配置文件：

```shell
vim /Users/didi/xzy/replica_sets/myrs_27019/mongod.conf
```

myrs_27019:

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/replica_sets/myrs_27019/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/replica_sets/myrs_27019/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/replica_sets/myrs_27019/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27019
replication:
  #副本集的名称
  replSetName: myrs
```

启动服务

```shell
mongod -f /Users/didi/xzy/replica_sets/myrs_27019/mongod.conf

mongod --dbpath /Users/didi/xzy/replica_sets/myrs_27019/data/db --logpath /Users/didi/xzy/replica_sets/myrs_27019/log/mongod.log --fork
```



# 连接节点

使用客户端命令连接任意一个节点，但这里尽量要连接主节点（27017节点）使之成为主节点：

```shell
mongo --host=localhost --port=27017
```

连入后必须初始化副本才行

```bash
rs.initiate()  #可加参数configuration
```

初始化之后按一下回车从secondary变为primary

之后可以使用

rs.conf()和rs.status()来查看相应的信息



## 添加副本从节点

在主节点添加从节点，将其他成员加入到副本集中

语法：

```shell
rs.add(host,arbiterOnly)
```

| Parameter   | Type               | Description                                                  |
| ----------- | ------------------ | ------------------------------------------------------------ |
| host        | string or document | 要添加到副本集的新成员。指定为字符串或配置文档：1）如果是一个字符串，则需要指定新成员的主机名和可选的端口号；2）如果是一个文档，请指定在members数组中找到的副本集成员配置文档。您必须在成员配置文档中指定主机字段。有关文档配置字段的说明，详见下方文档："主机成员的配置文档" |
| arbiterOnly | boolean            | 可选的。仅在<host>值为字符串时适用。如果为true，则添加的主机是仲裁者。 |

主机成员的配置文档：

```json
{
  _id:<int>,
  host:<string>,
  arbiterOnly:<boolean>,
  buildIndexes:<boolean>,
  hidden:<boolean>,
  priority:<number>,
  tags:<document>,
  slaveDelay:<int>,
  votes:<number>
}
```

示例：

将27018的副本节点加添加到副本集汇总：

```json
rs.add("localhost:27018")
```



## 添加仲裁者节点

```json
rs.add(host,arbiterOnly)
或
rs.addArb(host)

例子：
rs.addArb("localhost:27019")
```



# 副本集读写操作



登陆主节点27017，写入和读取数据：

```json
mongo --host localhost --port 27017
use test
db.comment.insert({"articleid":"100000","content":"今天天气真好，阳光明媚","userid":"1001","nickname":"Aoi","createdatetime":new Date()})

```



登陆从节点：

```jaon
mongo --host localhost --port 27018
#此时进入时会发现无法读取任何数据，要先将当前节点变为从节点
rs.slaveOk()
或
rs.slaveOk(true)

若要取消从节点
rs.slaveOk(false)
```



仲裁者节点

该节点不存放任何数据信息，只用于查看配置信息



# 主节点的选举原则

MongoDB在副本集中，会自动进行主节点的选举，主节点选举的触发条件：

1. 主节点故障
2. 主节点网络不可达（默认心跳信息为10秒）
3. 人工干预（rs.stepDown(600))

一旦触发选举，就要根据一定的规则来选主节点

选举规则是根据票数来决定谁获胜：

- 票数最高，且获得了"大多数"成员的投票支持的节点获胜

"大多数"的定义为：假设复制集内投票成员数量为N，则大多数为N/2+1。例如：3个投票成员，则大多数的值是2.当复制集内存活的数量不足大多数时，整个复制集将无法选举出Primary，复制集将无法提供写服务，处于只读状态。

- 若票数相同，且都获得了"大多数"成员的投票支持的，数据新的节点获胜。

数据的新旧是通过操作日志oplog来对比的。



# SpringDataMongoDB连接副本集

语法：

```xml
mongodb://host1,host2,host3/?connect=replicaSet&slaveOk=true&replicaSet=副本集名字
```

其中：

- slaveOk=true：开启副本节点读的功能，可实现读写分离。
- connect=replicaSet：自动到副本集中选择读写的主机。如果slaveOk是打开的，则实现读写分离。



示例：

连接replica set三台服务器（端口27017，27018，27019），直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器，写入操作应用在主服务器并且分布查询到从服务器。

```yaml
spring: 
	#数据源配置
	data:
		mongodb: 
			#主机地址
			#host: localhost
			#数据库
			#database: test
			#默认端口号是27017
			#port: 27017
			#也可以使用uri连接
			uri: mongodb://localhost:27017,localhost:27018,localhost:27019/test?connect=replicaSet&slaveOk=true&replicaSet=myrs
```



# 分片集群

分片是一种跨多台机器分布数据的方法，MongoDB使用分片来支持具有非常大的数据集和高吞吐量操作的部署。

换句话说：分片就是将数据拆分，将其分散到不同的机器上的过程。

<img src="/Users/didi/Documents/MongoDB.assets/image-20201201135548498.png" alt="image-20201201135548498" style="zoom:50%;" />

## 分片包含的组件

- 分片（存储）：每个分片包含分片数据的子集。每个分片都可以部署为副本集。
- mongos（路由）：mongos充当查询路由器，在客户端应用程序和分片集群之间提供接口。
- config servers（"调度"的配置）：配置服务器存储群集的元数据和配置设置。



## 第一套副本集

准备存放数据和日志的目录

```shell
#--------------------myshardrs01
mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27018/log
mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27018/data/db

mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27118/log
mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27118/data/db

mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27218/log
mkdir -p /Users/didi/xzy/sharded_cluster/myshardrs01_27318/data/db
```



新建或修改配置文件

```shell
vim /Users/didi/xzy/sharded_cluster/myshardrs01_27018/mongod.conf
```

Myshardrs01_27018

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/sharded_cluster/myshardrs01_27018/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27018/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27018/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27018
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
	#分片角色
	clusterRole: shardsvr
```

sharding.clusterRole:

| Value     | Description                                                  |
| --------- | ------------------------------------------------------------ |
| configsvr | Start this instance as a config server.The instance starts on port 27019 by default. |
| shardsvr  | Start this instance as a shard.The instance starts on port 27018 by default. |

注意：

设置sharding.clusterRole需要mongod示例运行复制。要将实例部署为副本集成员，请使用replSetName设置并指定副本集的名称。

第二个服务：

新建或修改配置文件：

```shell
vim /Users/didi/xzy/sharded_cluster/myshardrs01_27118/mongod.conf
```

Myshardrs01_27118

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/sharded_cluster/myshardrs01_27118/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27118/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27118/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27118
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
	#分片角色
	clusterRole: shardsvr
```



第三个服务：

新建或修改配置文件：

```shell
vim /Users/didi/xzy/sharded_cluster/myshardrs01_27218/mongod.conf
```

Myshardrs01_27218

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/sharded_cluster/myshardrs01_27218/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27218/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/sharded_cluster/myshardrs01_27218/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27218
replication:
  #副本集的名称
  replSetName: myshardrs01
sharding:
	#分片角色
	clusterRole: shardsvr
```



## 第二个副本集

创建三个服务，将端口和存储路径以及副本集名称改为myshardrs02即可，其他都相同



## 配置集

同样创建三个服务

新建或修改配置文件：

```shell
vim /Users/didi/xzy/sharded_cluster/myconfigrs_27019/mongod.conf
```

Myconfigrs_27019:

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/sharded_cluster/myconfigrs_27019/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/sharded_cluster/myconfigrs_27019/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/sharded_cluster/myconfigrs_27019/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27019
replication:
  #副本集的名称
  replSetName: myconfigrs
sharding:
	#分片角色
	clusterRole: configsvr
```



## 初始化副本集

与上面连接节点处相同

但是配置集中不需要添加仲裁节点，将两个节点以从节点的方式加入即可。



## 路由集

是mongos的服务，不是mongod的服务

第一步：准备存放日志的目录：

```shell
#-------------------mongos01  路由节点不存放数据，所以不需要存放数据的目录
mkdir /Users/didi/xzy/sharded_cluster/mymongos_27019/log
```

Mymongos_27017:

新建或修改配置文件：

```shell
vim /Users/didi/xzy/sharded_cluster/mymongos_27017/mongos.conf
```

mongos.conf

```bash
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: "/Users/didi/xzy/sharded_cluster/mymongos_27017/log/mongod.log"
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
storage:
  #mongod实例存储其数据的目录，storage.dbPath设置仅适用于mongod。
  dbPath: "/Users/didi/xzy/sharded_cluster/mymongos_27017/data/db"
  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: "/Users/didi/xzy/sharded_cluster/mymongos_27017/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll:true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27017
sharding:
	#指定配置节点副本集
	configDB: myconfigrs/localhost:27019,localhost:27119,localhost:27219
```

启动mongos:

```shell
mongoa -f /Users/didi/xzy/sharded_cluster/mymongos_27017/mongos.conf
```

此时路由还不能找到分片，所要要添加分片到路由中。

使用命令添加分片：

## (1)添加分片：

语法：

```shell
sh.addShard("IP:Port")
```

将第一套副本集添加进来：

```shell
mongos>
sh.addShard("myshardrs01/localhost:27018,localhost:27118,localhost:27218")
#---------------查看分片状况情况
sh.status()
```

将第二套副本集添加进来：

```shell
mongos>
sh.addShard("myshardrs01/localhost:27318,localhost:27418,localhost:27518")
#---------------查看分片状况情况
sh.status()
```



## (2)开启分片功能

sh.enableSharding("库名")、sh.shardCollection("库名.集合名",{"key":1})

```
sh.enableSharding("")
```

## (3)集合分片

对集合分片，必须使用sh.shardCollection()方法指定集合和分片键

语法：

```bash
sh.shardCollection(namespace,key,unique)
```

| Parameter | Type     | Description                                                  |
| --------- | -------- | ------------------------------------------------------------ |
| namespace | string   | 要（分片）共享对目标集合对命名空间，格式：<databse>.<collection> |
| key       | document | 用作分片键对所以规则文档。shard键决定MongoDB如何在shard之间分法文档。除非集合为空，否则索引必须在shard collection命令之前存在。如果集合为空，则MongoDB在对集合进行分片之前创建索引，前提是支持分片键的索引不存在。简单来说：由包含字段和该字段的索引遍历方向的文档组成。 |
| unique    | boolean  | 当值为true，片键字段上会限制为确保是唯一索引，哈希策略片键不支持唯一索引，默认为false |

例如在articled库中的comment集合中，以nickname为键，以哈希策略来分片。

```shell
#-------------------------首先开启article库的分片功能
sh.enableSharding("articledb")

sh.shardCollection("articledb.comment",{"nickname":"hashed"})
```



增加第二个路由节点：

和创建第一个节点的方式相同，开启服务后不需要再添加分配，会由配置服务自动同步。

# 安全认证



常用的内置角色：

- 数据库用户角色：read、readWrite
- 所有数据库用户角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
- 数据库管理角色：dbAdmin、dbOwner、userAdmin
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager
- 备份恢复角色：backup、restore
- 超级用户角色：root
- 内部角色：system

角色说明：

| 角色                 | 权限描述                                                     |
| -------------------- | ------------------------------------------------------------ |
| read                 | 可以读取数据库中任何数据。                                   |
| readWrite            | 可以读写所有数据库中任何数据，包括创建、重命名、删除集合     |
| readAnyDatabase      | 可以读取所有数据库中任何数据（除了数据库config和local之外）  |
| readWriteAnyDatabase | 可以读写所有数据库中任何数据（除了数据库config和local之外）  |
| userAdminAnyDatabase | 可以在指定数据库创建和修改用户（除了数据库config和local之外） |
| dbAdminAnyDatabase   | 可以读取任何数据库以及数据库进行清理、修改、压缩、获取统计信息、执行检查等操作（除了数据库config和local之外）。 |
| dbAdmin              | 可以读取指定数据库以及对数据库进行清理、修改、压缩、获取统计信息、执行检查等操作。 |
| userAdmin            | 可以指定数据库创建和修改用户                                 |
| clusterAdmin         | 可以对整个集群或数据库系统进行管理操作                       |
| backup               | 备份MongoDB数据最小的权限                                    |
| restore              | 从备份文件中还原恢复MongoDB数据（处理system.profile集合）的权限 |
| root                 | 超级账号，超级权限                                           |

# MongoDB安装

我们这里下载和安装社区版，[官网下载地址](https://www.mongodb.com/download-center/community)。 打开上述连接后，选择对应的版本、操作系统平台（常见的平台均支持）和包类型，点击Download按钮下载即可。

这里补充说明下，Windows平台有`ZIP`和`MSI`两种包类型: * ZIP：压缩文件版本 * MSI：可执行文件版本，点击”下一步”安装即可。

macOS平台除了在该网页下载`TGZ`文件外，还可以使用`Homebrew`安装。

更多安装细节可以参考[官方安装教程](https://docs.mongodb.com/manual/administration/install-community/)，里面有`Linux`、`macOS`和`Windows`三大主流平台的安装教程。
