# Mariadb数据库管理系统

数据库管理系统分	Oracle	MySQL---开发相同---Mariadb

`vim /etc/yum.repos.d/CentOS-MariaDB.repo`

```sh
[mariadb]
name=MariaDB
baseurl=http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

`yum install mariadb-* -y` #安装

MyCLI ：一个支持自动补全和语法高亮的 MySQL/MariaDB 客户端  //`yum install pip`，`pip install mycli`

包：mariadb*			端口号：3306		服务名：mariadb

-u, --user=name         #指定用户名		-p, --password          #指定密码

-h, --host=name         #指定主机名		-P, --port            		#指定端口

```sql
[root@localhost ~]# mysql -uroot -p123456 #登录
mysql> use mysql #使用数据库
mysql> update user set host='%' where user='root'; #使能够远程连接
mysql> flush privileges; #刷新权限
```

/var/log/mariadb/mariadb.log日志文件		/var/lib/mysql/数据库实体文件

```sh
[root@server23 ~]# grep -Ev "^#|^$" /etc/my.cnf	//数据库服务默认主配置文件
```

```conf
[mysqld]
datadir = /data/mysql # 数据库数据文件存放目录
socket  = /tmp/mysql.sock #为MySQL客户端程序和服务器之间的本地通讯指定一个套接字文件
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log	#记录错误日志文件
pid-file=/var/run/mariadb/mariadb.pid	#pid所在的目录
!includedir /etc/my.cnf.d
```

<img src="E:\Project\Textbook\linux云计算\数据库\assets\wps149.jpg" alt="img" style="zoom:67%;" /> <img src="E:\Project\Textbook\linux云计算\assets\wps150.jpg" alt="img" style="zoom:67%;" />

<img src="E:\Project\Textbook\linux云计算\assets\wps151.jpg" alt="img" style="zoom:67%;" /> <img src="E:\Project\Textbook\linux云计算\数据库\assets\wps152.jpg" alt="img" style="zoom:67%;" />

<img src="E:\Project\Textbook\linux云计算\数据库\assets\wps153.jpg" alt="img" style="zoom:67%;" /> 



## GRANT命令用于对用户进行授权：

```sql
grant create ON 数据库.表单名称 TO 用户名@主机名	//对某个特定数据库中的特定表单给予授权
grant select，delete ON 数据库.* TO 用户名@主机名	//对某个特定数据库中的所有表单给予授权
grant 权限 ON *.* TO 用户名@主机名				//对所有数据库及所有表单给予授权
grant 权限1,权限2 ON 数据库.* TO 用户名@主机名	//对某个数据库中的所有表单给予多个授权
grant all privileges on xd_db.* to 'user'@'%' identified by 'redhat' 
//允许本地用户user在%任何主机IP地址远程登陆对xd_db数据库下*.*所有表格有访问权限设置密码为redhat

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;


create user 'xiandian'@'localhost' identified by 'xd_paas'; 创建一个xiandian用户在本地授权密码xd_paas
create user '[用户名称]'@'%' identified by '[用户密码]';	//创建用户
create user Luigi@localhost identified by "redhat";		//增加管理员账户Luigi及密码redhat

SELECT HOST,USER,PASSWORD FROM user WHERE USER="luke";
```

## 查询luck主机名称、账户名称以及经过加密的密码值信息

```sh
show grants for alex@'localhost';	//查看alex所有的权限
mysqladmin -u root password 123456	//设置mysql数据库密码
mysql -uroot -p XXX < /home/renwole.sql	//导入数据库
```

```sh
[root@rhel ~]#mysql_secure_installation	//更改超级用户管理权限
Enter current password for root (enter for none): 当前数据库密码为空，直接按回车键
Set root password? [Y/n]				设置root用户密码
Remove anonymous users? [Y/n] y		删除匿名用户可登录数据库？
Disallow root login remotely? [Y/n]		禁止用户远程登陆
Remove test database and access to it?	删除test测试数据库并访问它？
Reload privilege tables now?			现在重新加载权限表？

[root@rhel ~]# mysql -u root -p123456	现在需指定用户登录
MariaDB [(none)]> flush privileges #使配置生效
```

# 导入数据库

```sql
mysql> create database onlinedb;
mysql> use onlinedb;
mysql> source d:/onlinedb sql;
```

# sql语句中的快捷键

```sql
\G格式化输出(文本式，竖立显示)			\s查看服务器端信息
\c结束命令输入操作						\q退出当前sq|命令行模式				\h查看帮助
```

# 数据库操作

```sql
use xxx	//使用数据库
show databases;								//查看数据库
show create database xxx;					//查看数据库详细信息
show engines								//查看数据库引擎
create database （if not exists ） 表名；	//创建数据库（如果不存在则创建）
alter database xxx;							//修改数据库
alter database xxx character set utf8; 		//修改数据库编码的命令
drop database xxx;							//删除数据库
exit	//推出数据库命令


mysqldump -u root -p linuxprobe > /root/linuxprobeDB.dump	//mysqldump命令用于备份数据库
mysql -u root -p linuxprobe < /root/linuxprobeDB.dump	//mysql命令用于导入数据库
```

# 数据库模式

creat schema <模式名> authorization <用户名>

create schema test authorization U1



drop schema <模式名> <cascade|restrict>

cascade(级联)表示在删除模式的同时把该模式中所有的数据库对象全部删除

restrict(限制)表示如果该模式中已经定义了下属的数据库对象，如表或视图等，则拒绝该删除语句的执行

| 数据查询                                                 | select （查询出数据，也可用于变量赋值）        |
| -------------------------------------------------------- | ---------------------------------------------- |
| 数据定义(表/视图/查询/存储过程/自定义函数/索引/触发器等) | create (创建)、drop(删除)、alter(修改)         |
| 数据操纵                                                 | insert（插入）、update（更新）、delete（删除） |
| 数据控制                                                 | grant（授权）、revoke（回收权限）              |

# 数据表操作

数据操作DML操作：添加数据、修改数据、删除数据

## 数据类型约束

字符串类型：

<img src="E:\Project\Textbook\linux云计算\assets\wps1-1682771292878-1.jpg" alt="img" style="zoom: 67%;" /> 

 

数值类型：

<img src="E:\Project\Textbook\linux云计算\assets\wps2-1682771292878-3.jpg" alt="img" style="zoom:67%;" /> 

 

日期和时间类型：

<img src="E:\Project\Textbook\linux云计算\assets\wps3-1682771292878-2.jpg" alt="img" style="zoom: 80%;" /> 



表的字段约束：

`int(4)、char(5)、varchar(7)`字段类型后面加括号限制宽度

`unsigned` 表示无正负符号(给数值类型使用，数值小于占用存储空间前导0)

`not null`	不能为空在操作数据库时如果输入该字段的数据为NULL，就会报错

`default`	设置默认值

`primary key`主键不能为空且唯一，一般和自动递增使用

`auto_increment`定义列为自增属性，一般用于主键， 数值会自动加1

`unique`	唯一索引(数据不能重复:用户名)可以增加查询速度,但是会降低插入和更新速度



主键：

1、表中每一行都应该有可以唯一 标识自己的一-列， 用于记录两条记录不能重复，任意两行都不具有相同的主键值

2、 应该总是定义主键虽然并不总是都需要主键，但大多数数据库设计人员都应保证他们创建的每个表具有一个主键，以便于以后的数据操纵和管理。



## 表定义

```sql
use xxx;		//使用数据表
create table XXX_back select * from XXX //数据表进行备份
```



### 创建数据表

**主键约束**：使某个字段不重复且不得为空，确保表内所有数据的唯一性。

```sql
CREATE TABLE<表名> (<表字段名><数据类型> [列级完整性约束条件]
[,<表字段名><数据类型> [列级完整性约束条件]]
[,<表级完整性约束条件>]) ;

CREATE TABLE Student(Sno CHAR(9) PRIMARY KEY,	/*列级完整性约束条件，Sno 是主码*/
	Sname CHAR(20) UNIQUE,		/* Sname取唯一值*/
	Ssex CHAR(2),
	Sage SMALLINT,
	Sdept CHAR(20)
)engine=innodb default charset=utf8； //设置引擎和字符集
```

```sql
create table xxx（列名字  类型（20））	
create table 新表 as select  from 旧表	复制数据表结构及数据到新表
create table 新表 as select  from 旧表 where false	复制数据表结构到新表
```

### 创建函数

```sql
create function xxx()
	return varcher（225）	//返回的数形
begin
	return (select  from xxx );
end
```

### 创建储存过程(参数化)	

```sql
delimiter  //------使用作为结束提交符号
	create procedure spXXX(id int)
reads sql data
begin
	select  from goods where gdID = id;
end 
```

### 修改函数

```sql
alter function xxx()
	return type_数值
begin
	return (select  from xxx );
end
```

## 修改表

```sql
alter table 原表名 rename 新表名	重命名数据表；
alter table 数据表 add num int;						//添加字段
alter table 数据表 chang 原表名 新表名  数据类型；	 	//更改字段名
alter table 数据表 modify 字段名 新数据类型 after uSex	；//修改字段排列在uSex字段之后
alter table 数据表 auto_increment =1000；			//修改表的自增值
alter table 数据表 engine = 'myisam';				//修改表引擎
alter table 表名 drop uPwd(字段名)	删除字段；
truncate xxx	清空数据表所有记录；
```

### 删除表及约束

```sql
drop table xxx1,xxx2，		//删除多个数据表
drop index 索引名 on 表名	//删除索引
drop function fnXXX			//删除函数
drop procedure psXXX		//删除储存过程
show databases tables  (注意加s)	//查询展示数据库查询展示数据表
```

```sql
alter table xxx drop primary key	//删除主键
alter table xxx drop index 索引名	/删除索引
```



## 查看表

```sql
show table engine	//查看数据表引擎
show tables;		//查看数据表
describe xxx		//查看数据表结构
show  from xxx;	//查看表中记录内容
show create table xxx	;			//查看表结构和索引
show create function fnXXX; 		//查看函数的定义
show create procedure spXXX;	//查看储存过程的定义
```



## 数据查询

### 一般查询格式：

```sql
select 输出的表达式 

from 表名/视图名 [as] 别名

where 条件的表达式
[group by] 分组名 having 聚集函数表达式
[order by] (asc|descj降序])
```



### 运算符

<img src="E:\Project\Textbook\linux云计算\assets\wps4-1682771292878-6.jpg" alt="img" style="zoom:67%;" /> 

**聚集函数：平均值**：avg()		**最小值**：min()	**最大值**：max()	**总和**：sum()		**计数**：count()

聚集函数只能用于select子句和having子句



### 连接查询格式：

| where  |                    |                                        |
| ------ | ------------------ | -------------------------------------- |
| join   | 内连接             | [inner] join (内连接)，默认 ... on ... |
| 左连接 | left [outer] join  |                                        |
|        | full [outer] join  |                                        |
| 右连接 | right [outer] join |                                        |

### 嵌套查询格式：

```sql
select function fnXXX	//使用函数
call psXXX(1)		//使用储存过程---1代表参数
select  from student where order by s_class desc;		//以s_class降序查询student数据表
```

## 数据添加与修改

语法一：   `INSERT INTO 表名(字段1,字段2,字段3…字段n) VALUES(值1,值2,值3…值n);`

语法二： `  INSERT INTO 表名 VALUES (值1,值2,值3…值n);`

`insert into 表名 values('804', '李诚', '男', '1958-12-02', '副教授', '计算机系')	向数据表中插入数据`

语法三（插入多条记录）：  ` INSERT INTO 表名 VALUES     `

`(值1,值2,值3…值n),     `

`(值1,值2,值3…值n),     `

`(值1,值2,值3…值n);`

语法四：  ` UPDATE 表名 SET     字段1=值1,     字段2=值2,     WHERE CONDITION;`

```sql
update 表名 set name = 'squirrel' where owner = 'Diane'	//修改更新修改数据
```



## 数据删除

语法：   `DELETE FROM 表名      WHERE CONITION;`

```sql
delect from 表名 where name = 'squirrel'	//删除数据表中记录where添加条件
TRUNCATE TABLE tablename;	//删除所有数据，保留表结构，不能撤消还原。
DROP TABLE table_name ; //删除指定数据库
```



# 详解mysql引擎

<img src="E:\Project\Textbook\linux云计算\assets\wps5-1682771292878-4.jpg" alt="img" style="zoom:67%;" /> 

MySQL服务器把数据的存储和提取操作都封装到了一个叫存储引擎的模块里。我们知道表是由一行一行的记录组成的，但这只是一个逻辑上的概念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是存储引擎负责的事情。为了实现不同的功能，MySQL 提供了各式各样的存储引擎，不同存储引擎管理的表具体的存储结构可能不同，采用的存取算法也可能不同。

存储引擎以前叫做表处理器，它的功能就是接收上层传下来的指令，然后对表中的数据进行提取或写压操作。

为了管理方便，人们把连接管理、查询缓存、语法解析、查询优化这些并不涉及真实数据存储的功能划分为MySQLserver的功能，把真实存取数据的功能划分为存储引擎的功能。各种不同的存储引擎向上边的MySQL server 层提供统一的调用接口(也就是存储引擎API) ，包含了几十个底层函数，像"读取索引第一条内容"、 "读取索引下一条内容"、"插入记录"等等。所以在MySQL server 完成了查询优化后，只需按照生成的执行计划调用底层存储引擎提供的API,获取到数据后返回给客户端就好了。

 **MySQL支持非常多种存储引擎**

| 存储引擎 | 描述                               | 存储引擎  | 描述                           |
| -------- | ---------------------------------- | --------- | ------------------------------ |
| ARCHIVE  | 用于数据存档(行被插入后不能再修改) | BLACKHOLE | 丢弃写操作，读操作会返回空内容 |
| CSV      | 在存储数据时，以逗号分隔各个数据项 | FEDERATED | 用来访问远程表                 |
| InnoDB   | 具备外键支持功能的事务存储引擎     | MEMORY    | 置于内存的表                   |
| MERGE    | 用来管理多个MyISAM表构成的表集合   | MyISAM    | 主要的非事务处理存储引擎       |
| NDB      | MySQL集群专用存储引擎              |           |                                |



MyISAM和InnoDB表引擎的区别

1)、事务支持

2)、存储结构

3)、表锁差异

4)、表主键

5)、表的具体行数

6)、CURD操作

7)、外键

8)、查询效率

