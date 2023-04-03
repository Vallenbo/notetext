# **GORM**

![img](E:\Project\Textbook\Golang常用服务手册\assets\wps1.jpg) 

 

gorm是一个使用Go语言编写的ORM框架。能直接使用代码操作数据库，进行表和数据库的创建。

中文官方网站内含十分齐全的中文文档，对开发者友好，支持主流数据库，有了它你甚至不需要再继续向下阅读本文。

------

优点:>提高开发效率

缺点:>牺牲执行性能>牺牲灵活性>弱化SQL能力



> 安装gorm官方库:  go get -u gorm.io/gorm
>
> gorm官方中文文档：https://gorm.io/zh_CN/docs/



**连接数据库**

连接不同的数据库都需要导入对应数据的驱动程序，GORM 官方支持的数据库类型有： MySQL, PostgreSQL, SQlite, SQL Server

> 导入需要的数据库驱动即可：go get -u gorm.io/driver/mysql 

> 连接函数
>
> gorm.Open("mysql", "user:password@(localhost)/dbname?charset=utf8mb4&parseTime=True&loc=Local")



**GORM基本示例**

Docker快速创建MySQL实例



> 在本地的13306端口运行一个名为mysql8019，root用户名密码为root1234的MySQL容器环境:
>
> docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19

> 在另外启动一个MySQL Client连接上面的MySQL环境，密码为上一步指定的密码root1234:
>
> docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p

 

**GORM Model定义**

在使用ORM工具时，通常我们需要在代码中定义模型（Models）与数据库中的数据表进行映射，在GORM中模型（Models）通常是正常定义的结构体、基本的go类型或它们的指针。 同时也支持sql.Scanner及driver.Valuer接口（interfaces）。

 

**gorm.Model**

为了方便模型定义，GORM内置了一个gorm.Model结构体。你可以将它嵌入到你自己的定义的结构体中

```go
type Model struct {		// gorm.Model 定义
 ID uint `gorm:"primary_key"` // 内嵌，默认ID字段作为表的主键
 CreatedAt time.Time
 UpdatedAt time.Time
 DeletedAt *time.Time
}
```

```go
type User struct { //模型定义示例
	gorm.Model   //内嵌模型
	Name         string
	Age          sql.NullInt64
	Birthday     *time.Time
	Email        string  `gorm:"type:varchar(100);unique_index"`
	Role         string  `gorm:"size:255；default:'小王子'"` // 设置字段大小为255，设置默认值
	MemberNumber *string `gorm:"unique;not null"`        // 设置会员号（member number）唯一并且不为空
	Num          int     `gorm:"AUTO_INCREMENT"`         // 设置 num 为自增类型
	Address      string  `gorm:"index:addr"`             // 给address字段创建名为addr的索引
	IgnoreMe     int     `gorm:"-"`                      // 忽略本字段
}
```



**结构体标记（tags）**

使用结构体声明模型时，标记（tags）是可选项。gorm支持以下标记:



支持的结构体标记（Struct tags）

| 结构体标记（Tag）         | 描述                   | 结构体标记（Tag） | 描述                                                     |
| ------------------------- | ---------------------- | ----------------- | -------------------------------------------------------- |
| Column                    | 指定列名               | PRECISION         | 指定列精度                                               |
| Type                      | 指定列数据类型         | NOT NULL          | 将列指定为非 NULL                                        |
| Size                      | 指定列大小, 默认值255  | AUTO_INCREMENT    | 指定列是否为自增类型                                     |
| UNIQUE_INDEX和 INDEX 类似 | 只不过创建的是唯一索引 | INDEX             | 创建具有或不带名称的索引, 如果多个索引同名则创建复合索引 |
| UNIQUE                    | 将列指定为唯一         | PRIMARY_KEY       | 将列指定为主键                                           |
| DEFAULT                   | 指定列默认值           | EMBEDDED          | 将结构设置为嵌入                                         |
| EMBEDDED_PREFIX           | 设置嵌入结构的前缀     | -                 | 忽略此字段                                               |



**关联相关标记（tags）**

| 结构体标记（Tag）                | 描述                               | 结构体标记（Tag）      | 描述                           |
| -------------------------------- | ---------------------------------- | ---------------------- | ------------------------------ |
| MANY2MANY                        | 指定连接表                         | FOREIGNKEY             | 设置外键                       |
| ASSOCIATION_FOREIGNKEY           | 设置关联外键                       | POLYMORPHIC            | 指定多态类型                   |
| POLYMORPHIC_VALUE                | 指定多态值                         | JOINTABLE_FOREIGNKEY   | 指定连接表的外键               |
| ASSOCIATION_JOINTABLE_FOREIGNKEY | 指定连接表的关联外键               | SAVE_ASSOCIATIONS      | 是否自动完成 save 的相关操作   |
| ASSOCIATION_AUTOUPDATE           | 是否自动完成 update 的相关操作     | ASSOCIATION_AUTOCREATE | 是否自动完成 create 的相关操作 |
| ASSOCIATION_SAVE_REFERENCE       | 是否自动完成引用的 save 的相关操作 | PRELOAD                | 是否自动完成预加载的相关操作   |

 

**主键、表名、列名的约定**

主键（Primary Key)：GORM 默认会使用名为ID的字段作为表的主键。 `gorm:"primary_key"`

表名（Table Name）：表名默认就是结构体名称的复数。	type User struct {} // 默认表名是 `users`

列名（Column Name）：列名由字段名称进行下划线分割来生成。	

 

时间戳跟踪

CreatedAt如果模型有 CreatedAt字段，该字段的值将会是初次创建记录的时间。

> db.Create(&user) // `CreatedAt`将会是当前时间
>
> db.Model(&user).Update("CreatedAt", time.Now())	// 可以使用`Update`方法来改变`CreateAt`的值

 

UpdatedAt如果模型有UpdatedAt字段，该字段的值将会是每次更新记录的时间。

> db.Save(&user) // `UpdatedAt`将会是当前时间
>
> db.Model(&user).Update("name", "jinzhu") // `UpdatedAt`将会是当前时间

 

DeletedAt如果模型有DeletedAt字段，调用Delete删除该记录时，将会设置DeletedAt字段为当前时间，而不是直接将记录从数据库中删除。

# CRUD指南

CRUD通常指数据库的增删改查操作，如何使用GORM实现创建、查询、更新和删除操作。

 

***\*创建\****

***\*创建记录\****：Create(&user)

注意：所有字段的零值, 比如0, "",false或者其它零值，都不会保存到数据库内，但会使用他们的默认值。 如果你想避免这种情况，可以考虑使用指针或实现 Scanner/Valuer接口

 

1、使用指针方式实现零值存入数据库

```go
type User struct { // 使用指针
	ID   int64
	Name *string `gorm:"default:'小王子'"`
	Age  int64
}
user := User{Name: new(string), Age: 18))}
db.Create(&user) // 此时数据库中该条记录name字段的值就是''
```



2、使用Scanner/Valuer接口方式实现零值存入数据库

```go
type User struct { // 使用 Scanner/Valuer
	ID int64
	Name sql.NullString `gorm:"default:'小王子'"` // sql.NullString 实现了Scanner/Valuer接口
	Age int64
}
user := User{Name: sql.NullString{"", true}, Age:18}
db.Create(&user) // 此时数据库中该条记录name字段的值就是''
```



 

 

***\*查询\****