导入数据库：

```sql
mysql> create database onlinedb;
mysql> use onlinedb;
mysql> source d:/onlinedb sql;
```

 

sql语句中的快捷键：

```sql
\G格式化输出(文本式，竖立显示)
\s查看服务器端信息
\c结束命令输入操作
\q退出当前sq|命令行模式
\h查看帮助
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
```

 

数据库模式：

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

![img](E:\Project\Textbook\linux云计算\assets\wps3-1682771292878-2.jpg) 

 

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

 

 

1、创建数据表

CREATE TABLE<表名> (<表字段名><数据类型> [列级完整性约束条件]

[,<表字段名><数据类型> [列级完整性约束条件]]

[,<表级完整性约束条件>]) ;

 

```sql
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

 

创建函数

```sql
create function xxx()
return varcher（225）	//返回的数形
begin
	return (select  from xxx );
end
```

 

创建储存过程(参数化)	

```sql
delimiter  //------使用作为结束提交符号
create procedure spXXX(id int)
reads sql data
begin
	select  from goods where gdID = id;
end 
```

 

修改函数

```sql
alter function xxx()
return type_数值
begin
	return (select  from xxx );
end
```

 

2、修改表

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

 

3、删除表及约束

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

 

4、查看

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



运算符

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



主键约束

理解：使某个字段不重复且不得为空，确保表内所有数据的唯一性。

```sql
CREATE TABLE user (
  id INT primary KEY,
  name VARCHAR(20)
);
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

`update 表名 set name = 'squirrel' where owner = 'Diane'		修改更新修改数据`

 

## 数据删除

语法：   `DELETE FROM 表名      WHERE CONITION;`

`delect from 表名 where name = 'squirrel'`		//删除数据表中记录where添加条件

`TRUNCATE TABLE tablename`	//删除所有数据，保留表结构，不能撤消还原。

 

# 详解mysql引擎

<img src="E:\Project\Textbook\linux云计算\assets\wps5-1682771292878-4.jpg" alt="img" style="zoom:67%;" /> 

MySQL服务器把数据的存储和提取操作都封装到了一个叫存储引擎的模块里。我们知道表是由一行一行的记录组成的，但这只是一个逻辑上的概念，物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是存储引擎负责的事情。为了实现不同的功能，MySQL 提供了各式各样的存储引擎，不同存储引擎管理的表具体的存储结构可能不同，采用的存取算法也可能不同。

存储引擎以前叫做表处理器，它的功能就是接收上层传下来的指令，然后对表中的数据进行提取或写压操作。

为了管理方便，人们把连接管理、查询缓存、语法解析、查询优化这些并不涉及真实数据存储的功能划分为MySQLserver的功能，把真实存取数据的功能划分为存储引擎的功能。各种不同的存储引擎向上边的MySQL server 层提供统一的调用接口(也就是存储引擎API) ，包含了几十个底层函数，像"读取索引第一条内容"、 "读取索引下一条内容"、"插入记录"等等。所以在MySQL server 完成了查询优化后，只需按照生成的执行计划调用底层存储引擎提供的API,获取到数据后返回给客户端就好了。

<img src="E:\Project\Textbook\linux云计算\assets\wps6-1682771292878-5.jpg" alt="img" style="zoom:67%;" /> 

 

MyISAM和InnoDB表引擎的区别

1)、事务支持

2)、存储结构

3)、表锁差异

4)、表主键

5)、表的具体行数

6)、CURD操作

7)、外键

8)、查询效率

 

 

 

 

 

 

 

 

 