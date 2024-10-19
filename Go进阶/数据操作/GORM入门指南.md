# GORM

<img src="E:\Project\Textbook\Golang常用服务手册\assets\wps1.jpg" alt="img" style="zoom:50%;" /> 

[gorm官方中文文档](https://gorm.io/zh_CN/docs/)

gorm是一个使用Go语言编写的ORM框架。能直接使用代码操作数据库，进行表和数据库的创建。

中文官方网站内含十分齐全的中文文档，对开发者友好，支持主流数据库。

> 优点:>提高开发效率>通过对象进行表操作>规范代码工整
>
> 缺点:>牺牲执行性能>牺牲灵活性>弱化SQL能力
>

**安装gorm官方库**: ` go get -u gorm.io/gorm`
导入需要的**数据库驱动**：`go get -u gorm.io/driver/mysql `<---不同数据库版本



#  连接MySQL

```go
// 连接数据库时需要传递参数
// 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
dsn := "root:123456@tcp(192.168.4.6:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
```

**注意：**想要正确的处理 `time.Time` ，您需要带上 `parseTime` 参数， ([更多参数](https://github.com/go-sql-driver/mysql#parameters)) 要支持完整的 UTF-8 编码，需要将 `charset=utf8` 更改为 `charset=utf8mb4` 查看 [此文章](https://mathiasbynens.be/notes/mysql-utf8mb4) 获取详情

MySQL 驱动程序提供了 [一些高级配置](https://github.com/go-gorm/mysql) 可以在初始化过程中使用，例如：

```go
db, err := gorm.Open(mysql.New(mysql.Config{
  DSN: "gorm:gorm@tcp(127.0.0.1:3306)/gorm?charset=utf8&parseTime=True&loc=Local", // DSN data source name
  DefaultStringSize: 256, // string 类型字段的默认长度
  DisableDatetimePrecision: true, // 禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
  DontSupportRenameIndex: true, // 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和MariaDB 不支持重命名索引
  DontSupportRenameColumn: true, // 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
  SkipInitializeWithVersion: false, // 根据当前 MySQL 版本自动配置
}), &gorm.Config{})
```

## 自定义驱动

GORM 允许通过 `DriverName` 选项自定义 MySQL 驱动，例如：

```go
import (
  _ "example.com/my_mysql_driver"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

db, err := gorm.Open(mysql.New(mysql.Config{
  DriverName: "my_mysql_driver",
  DSN: "gorm:gorm@tcp(localhost:9910)/gorm?charset=utf8&parseTime=True&loc=Local", // data source name, 详情参考：https://github.com/go-sql-driver/mysql#dsn-data-source-name
}), &gorm.Config{})
```

## 现有的数据库连接

GORM 允许通过一个现有的数据库连接来初始化 `*gorm.DB`

```go
import (
    "database/sql"
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
)

sqlDB, err := sql.Open("mysql", "mydb_dsn")
gormDB, err := gorm.Open(mysql.New(mysql.Config{
    Conn: sqlDB,
}), &gorm.Config{})
```

# 连接PostgreSQL

基本代码同上，注意引入对应`postgres`驱动并正确指定`gorm.Open()`参数。

```go
dsn := "host=localhost user=gorm password=gorm dbname=gorm port=9920 sslmode=disable TimeZone=Asia/Shanghai"
db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
```

# 连接Sqlite3

基本代码同上，注意引入对应`sqlite`驱动并正确指定`gorm.Open()`参数。

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{})
```

# 连接SQL Server

基本代码同上，注意引入对应`mssql`驱动并正确指定`gorm.Open()`参数。

```go
dsn := "sqlserver://gorm:LoremIpsum86@localhost:9930?database=gorm"
db, err := gorm.Open(sqlserver.Open(dsn), &gorm.Config{})
```

# GORM操作MySQL基本示例

使用GORM连接上面的`db1`进行创建、查询、更新、删除操作。

```go
type Product struct {
	gorm.Model
	Code  string
	Price uint
}

func main(){
	dsn := "root:123456@tcp(192.168.4.5:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Println("connection mysql err :", err)
	}
}
```



# GORM Model定义

在使用ORM工具时，通常我们需要在代码中定义模型（Models）与数据库中的数据表进行映射，在GORM中模型（Models）通常是正常定义的结构体、基本的go类型或它们的指针。 同时也支持`sql.Scanner`及`driver.Valuer`接口（interfaces）。

## gorm.Model定义模型方式一

为了方便模型定义，GORM内置了一个`gorm.Model`结构体。`gorm.Model`是一个包含了`ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`四个字段的Golang结构体。

[gorm tag字段标签说明 ](https://gorm.io/zh_CN/docs/models.html#字段标签)

```go
type Model struct { // gorm.Model 定义
	ID        uint `gorm:"primarykey"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt DeletedAt `gorm:"index"`
}
```

你可以将它嵌入到你自己的模型中：

```go
type User struct {
  gorm.Model // 将 `ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`字段注入到`User`模型中
  Name string
}
```
模型定义示例

```go
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64 //零值类型
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"` //不可重复字段
  Role         string  `gorm:"size:255"` // 设置字段大小为255
  MemberNumber *string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
  Num          int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型
  Address      string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
  IgnoreMe     int     `gorm:"-"` // 忽略本字段
}
```

## gorm.Model定义模型方式二

匹配User表

```go
type User struct { //2、 不使用gorm.Model，自行定义模型
  ID   int
  Name string
}

func (User) TableName() string {
	return "User"
}
```



## 结构体标记（tags）

使用结构体声明模型时，标记（tags）是可选项。gorm支持以下标记:

### 支持的结构体标记（Struct tags）

| 结构体标记（Tag） | 描述                                                     |
| :---------------- | -------------------------------------------------------- |
| Column            | 指定列名                                                 |
| Type              | 指定列数据类型                                           |
| Size              | 指定列大小, 默认值255                                    |
| PRIMARY_KEY       | 将列指定为主键                                           |
| UNIQUE            | 将列指定为唯一                                           |
| DEFAULT           | 指定列默认值                                             |
| PRECISION         | 指定列精度                                               |
| NOT NULL          | 将列指定为非 NULL                                        |
| AUTO_INCREMENT    | 指定列是否为自增类型                                     |
| INDEX             | 创建具有或不带名称的索引, 如果多个索引同名则创建复合索引 |
| UNIQUE_INDEX      | 和 `INDEX` 类似，只不过创建的是唯一索引                  |
| EMBEDDED          | 将结构设置为嵌入                                         |
| EMBEDDED_PREFIX   | 设置嵌入结构的前缀                                       |
| -                 | 忽略此字段                                               |

### 关联相关标记（tags）

| 结构体标记（Tag）                | 描述                               |
| :------------------------------- | :--------------------------------- |
| MANY2MANY                        | 指定连接表                         |
| FOREIGNKEY                       | 设置外键                           |
| ASSOCIATION_FOREIGNKEY           | 设置关联外键                       |
| POLYMORPHIC                      | 指定多态类型                       |
| POLYMORPHIC_VALUE                | 指定多态值                         |
| JOINTABLE_FOREIGNKEY             | 指定连接表的外键                   |
| ASSOCIATION_JOINTABLE_FOREIGNKEY | 指定连接表的关联外键               |
| SAVE_ASSOCIATIONS                | 是否自动完成 save 的相关操作       |
| ASSOCIATION_AUTOUPDATE           | 是否自动完成 update 的相关操作     |
| ASSOCIATION_AUTOCREATE           | 是否自动完成 create 的相关操作     |
| ASSOCIATION_SAVE_REFERENCE       | 是否自动完成引用的 save 的相关操作 |
| PRELOAD                          | 是否自动完成预加载的相关操作       |

# 主键、表名、列名的约定

## 主键（Primary Key）

GORM 默认会使用名为ID的字段作为表的主键。

```go
type User struct {
  ID   string // 名为`ID`的字段会默认作为表的主键
  Name string
}
type Animal struct { // 使用`AnimalID`作为主键
  AnimalID int64 `gorm:"primary_key"`
  Name     string
  Age      int64
}
```

## 表名（Table Name）

表名默认就是结构体名称的复数，例如：

```go
type User struct {} // 默认表名是 `users`
```

## 列名（Column Name）

列名由字段名称进行下划线分割来生成

```go
type User struct {
  ID        uint      // column name is `id`
  Name      string    // column name is `name`
  Birthday  time.Time // column name is `birthday`
  CreatedAt time.Time // column name is `created_at`
}
```

```go
type Animal struct { //可以使用结构体tag指定列名
  AnimalId    int64     `gorm:"column:beast_id"`         // set column name to `beast_id`
  Birthday    time.Time `gorm:"column:day_of_the_beast"` // set column name to `day_of_the_beast`
  Age         int64     `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast`
}
```

## 时间戳跟踪

# 创建

Gorm 的默认约定

- 在 Gorm 中，如果没有显式指定表名，它会根据结构体的名称进行转换来确定表名。对于`Product`结构体，Gorm 会将其名称转换为复数形式作为默认的表名。
- 转换规则通常是在结构体名称后添加`s`作为复数形式（这是一种简单的默认规则，实际情况可能因语言习惯等因素有所不同）。所以在这里，Gorm 会尝试将数据插入到名为`products`的表中。
- 例如，如果结构体名称是`User`，Gorm 默认会将数据插入到`users`表中。

自定义表名：如果想要自定义表名，可以通过在结构体定义中使用`gorm:"table:your_table_name"`标签来指定

```go
type Product struct {
    Code  string `gorm:"column:code"` // column用于指定结构体中的字段对应的数据库表中的列名
    Price int    `gorm:"column:price;type:int"` //多个属性用；分割
    gorm.Model `gorm:"table:product_table"`	// 指定表名
}
```

## 创建数据表

```go
db.AutoMigrate(&Product{}) // 迁移(创建) 表Product
```

注意，在修改数据库的默认字符集后，如果已经创建了 `teachers` 表，需要使用 SQL 修改表的字符集，如下所示：

```sql
ALTER TABLE teachers CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 创建记录

```go
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

result := db.Create(&user) // Create添加数据,通过数据的指针来创建

user.ID             // 返回插入数据的主键
result.Error        // 返回 error
result.RowsAffected // 返回插入记录的条数
```

## 用指定的字段创建记录

通过Select指定字段，创建记录并更新给出的字段。

```go
db.Select("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
```

创建一个记录且一同忽略传递给略去的字段值。

```go
db.Omit("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`birthday`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
```

## 批量插入

要有效地插入大量记录，请将一个 `slice` 传递给 `Create` 方法。 GORM 将生成单独一条SQL语句来插入所有数据，并回填主键的值，钩子方法也会被调用。

```go
var users = []User{{Name: "jinzhu1"}, {Name: "jinzhu2"}, {Name: "jinzhu3"}}
db.Create(&users)

for _, user := range users {
  user.ID // 1,2,3
}
```

使用 `CreateInBatches` 分批创建时，你可以指定每批的数量，例如：

```go
var users = []User{{name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

db.CreateInBatches(users, 100) // 数量为 100
```

[Upsert](https://gorm.io/zh_CN/docs/create.html#upsert) 和 [Create With Associations](https://gorm.io/zh_CN/docs/create.html#create_with_associations) 也支持批量插入

> **注意** 使用`CreateBatchSize` 选项初始化 GORM 时，所有的创建& 关联 `INSERT` 都将遵循该选项

```go
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  CreateBatchSize: 1000,
})

db := db.Session(&gorm.Session{CreateBatchSize: 1000})

users = [5000]User{{Name: "jinzhu", Pets: []Pet{pet1, pet2, pet3}}...}

db.Create(&users)// INSERT INTO users xxx (5 batches)// INSERT INTO pets xxx (15 batches)
```

# 查询

## 检索单个对象

GORM 提供了 `First`、`Take`、`Last` 方法，以便从数据库中检索单个对象。当查询数据库时它添加了 `LIMIT 1` 条件，且没有找到记录时，它会返回 `ErrRecordNotFound` 错误

```go
db.First(&user) // SELECT * FROM users ORDER BY id LIMIT 1; // 获取第一条记录（主键升序）
db.First(&product, "code = ?", "D42") // 查找 code 字段值为 D42 的记录

db.Take(&user) // SELECT * FROM users LIMIT 1; // 获取一条记录，没有指定排序字段

db.Last(&user) // SELECT * FROM users ORDER BY id DESC LIMIT 1; // 获取最后一条记录（主键降序）

result := db.First(&user)
result.RowsAffected // 返回找到的记录数
result.Error        // returns error or nil

errors.Is(result.Error, gorm.ErrRecordNotFound) // 检查 ErrRecordNotFound 错误
```

> 如果你想避免`ErrRecordNotFound`错误，你可以使用`Find`，比如`db.Limit(1).Find(&user)`，`Find`方法可以接受struct和slice的数据。

> 对单个对象使用`Find`而不带limit，`db.Find(&user)`将会查询整个表并且只返回第一个对象，这是性能不高并且不确定的。

和方法将按主键排序查找第一条和最后一条记录。仅当指向目标结构的指针作为参数传递给方法或使用 指定模型时，它们才有效。此外，如果没有为相关模型定义主键，则模型将按第一个字段排序。例如：`First` `Last` `db.Model()`

```go
var user User
var users []User

// works because destination struct is passed in
db.First(&user)// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

// works because model is specified using `db.Model()`
result := map[string]interface{}{}
db.Model(&User{}).First(&result)// SELECT * FROM `users` ORDER BY `users`.`id` LIMIT 1

result := map[string]interface{}{} 
db.Table("users").First(&result) // doesn't work

result := map[string]interface{}{}
db.Table("users").Take(&result) // works with Take

// no primary key defined, results will be ordered by first field (i.e., `Code`)
type Language struct {
  Code string
  Name string
}
db.First(&Language{})// SELECT * FROM `languages` ORDER BY `languages`.`code` LIMIT 1
```

### 根据主键检索

如果主键是数字，则可以使用内[联条件](https://gorm.io/zh_CN/docs/query.html#inline_conditions)使用主键检索对象。使用字符串时，需要格外小心以避免 SQL 注入;有关详细信息，请查看[“安全性](https://gorm.io/zh_CN/docs/security.html)”部分。

```go
db.First(&user, 10) // SELECT * FROM users WHERE id = 10;
db.First(&user, "10") // SELECT * FROM users WHERE id = 10;
db.Find(&users, []int{1,2,3}) // SELECT * FROM users WHERE id IN (1,2,3);
```

如果主键是字符串（例如，像 uuid），则查询将按如下方式编写：

```go
db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
// SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";
```

当目标对象有一个主键值时，将使用主键构建查询条件，例如：

```go
var user = User{ID: 10}
db.First(&user)// SELECT * FROM users WHERE id = 10;

var result User
db.Model(User{ID: 10}).First(&result)// SELECT * FROM users WHERE id = 10;
```

## 检索全部对象

```go
// Get all records
result := db.Find(&users)// SELECT * FROM users;

result.RowsAffected // returns found records count, equals `len(users)`
result.Error        // returns error
```

## 条件

### String 条件

```go
// 获取第一个匹配的记录
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// 获取所有匹配的记录
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// 时间
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';

db.Where("age>?", 30).First(t).First(&Teacher{}) // 链式查询，将筛选的第一个结果赋值给t，.First(&Teacher{})是继续向下查询，将符合条件的第一条记录读取并赋值给 Teacher{} 的结构体
```

> 如果已设置对象的主键，则条件查询不会涵盖主键的值，而是将其用作“and”条件。例如：
>
> ```go
> var user = User{ID: 10}
> db.Where("id = ?", 20).First(&user)
> // SELECT * FROM users WHERE id = 10 and id = 20 ORDER BY id ASC LIMIT 1
> ```
>
> 此查询将给出错误。因此，在要使用变量（例如从数据库中获取新值）之前，将主键属性设置为 nil。`record not found``id``user`

### Struct & Map 条件

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// Slice of primary keys
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```

> **注意**使用 struct 查询时，GORM 将仅使用非零字段进行查询，这意味着如果您的字段的值为 、 或其他[零值](https://tour.golang.org/basics/12)，则不会用于构建查询条件，例如：`0``''``false`

```go
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
```

要在查询条件中包含零值，您可以使用映射，该映射将所有键值作为查询条件包含在内，例如：

```go
db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
```

有关更多详细信息，请参阅[指定结构搜索字段](https://gorm.io/zh_CN/docs/query.html#specify_search_fields)。

### 指定结构体查询字段

使用 struct 进行搜索时，可以通过将相关字段名称或 dbname 传递给 来指定要在查询条件中使用的结构中的哪些特定值，例如：`Where()`

```go
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;
```

### 内联条件

查询条件可以内联到方法中，类似于 。`First``Find``Where`

```go
// Get by primary key if it were a non-integer type
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;
```

### Not 条件

构建非条件，工作类似于`Where`

```go
db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
```

### Or 条件

```go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);
```

对于更复杂的 SQL 查询。另请参阅[高级查询中的组条件](https://gorm.io/zh_CN/docs/advanced_query.html#group_conditions)。

## 选择特定字段

`Select`允许您指定要从数据库中检索的字段。否则，GORM 将默认选择所有字段。

```go
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```

另请查看[智能选择字段](https://gorm.io/zh_CN/docs/advanced_query.html#smart_select)

## 排序

指定从数据库中检索记录时的顺序

```go
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// Multiple orders
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)
```

# 更新

## 保存所有字段

`Save` 会保存所有的字段，即使字段是零值

```go
user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user) // UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
```

`Save`是一个组合函数。如果保存值不包含主键，它将 ，否则它将执行（包含所有字段）。`Create Update`

```go
db.Save(&User{Name: "jinzhu", Age: 100}) // INSERT INTO `users` (`name`,`age`,`birthday`,`update_at`) VALUES ("jinzhu",100,"0000-00-00 00:00:00","0000-00-00 00:00:00")

db.Save(&User{ID: 1, Name: "jinzhu", Age: 100})
// UPDATE `users` SET `name`="jinzhu",`age`=100,`birthday`="0000-00-00 00:00:00",`update_at`="0000-00-00 00:00:00" WHERE `id` = 1
```

> **注意**不要与 一起使用，这是一种**未定义的行为**。`Save``Model`

## 更新单个列

当使用 更新单个列时，它需要有任何条件，否则会引发错误，请查看[阻止全局更新](https://gorm.io/zh_CN/docs/update.html#block_global_updates)以获取详细信息。当使用该方法并且其值具有主值时，主键将用于构建条件，例如：`Update``ErrMissingWhereClause``Model`

```go
// Update with conditions
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User's ID is `111`:
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// Update with conditions and model value
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

## 更新多列

```go
Updates`支持使用 或 更新，使用 OR 更新时默认只会更新非零字段`struct``map[string]interface{}``struct
// Update attributes with `struct`, will only update non-zero fields
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// Update attributes with `map`
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

> **注意**使用 struct 更新时，GORM 将仅更新非零字段。您可能希望用于更新属性或用于指定要更新的字段`map``Select`

## 更新选定字段

如果要在更新时更新所选字段或忽略某些字段，可以使用 ，`Select``Omit`

```go
// Select with Map
// User's ID is `111`:
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// Select with Struct (select zero value fields)
db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;

// Select all fields (select all fields include zero value fields)
db.Model(&user).Select("*").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})

// Select all fields but omit Role (select all fields include zero value fields)
db.Model(&user).Select("*").Omit("Role").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})
```



# 删除

## 删除一条记录

删除一条记录时，删除对象需要指定主键，否则会触发 [批量 Delete](https://gorm.io/zh_CN/docs/delete.html#batch_delete)，例如：

```go
// Email 的 ID 是 `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// 带额外条件的删除
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```

## 根据主键删除

GORM 允许通过主键(可以是复合主键)和内联条件来删除对象，它可以使用数字（如以下例子。也可以使用字符串——译者注）。查看 [查询-内联条件（Query Inline Conditions）](https://gorm.io/zh_CN/docs/query.html#inline_conditions) 了解详情。

```go
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```



# session

GORM 提供了 `Session` 方法，这是一个 [`New Session Method`](https://gorm.io/zh_CN/docs/method_chaining.html)，它允许创建带配置的新建会话模式：

```go
// Session 配置
type Session struct {
  DryRun                   bool
  PrepareStmt              bool
  NewDB                    bool
  Initialized              bool
  SkipHooks                bool
  SkipDefaultTransaction   bool
  DisableNestedTransaction bool
  AllowGlobalUpdate        bool
  FullSaveAssociations     bool
  QueryFields              bool
  Context                  context.Context
  Logger                   logger.Interface
  NowFunc                  func() time.Time
  CreateBatchSize          int
}
```

## DryRun

生成 `SQL` 但不执行。 它可以用于准备或测试生成的 SQL，例如：

```go
// 新建会话模式
stmt := db.Session(&Session{DryRun: true}).First(&user, 1).Statement
stmt.SQL.String() //=> SELECT * FROM `users` WHERE `id` = $1 ORDER BY `id`
stmt.Vars         //=> []interface{}{1}

// 全局 DryRun 模式
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{DryRun: true})

// 不同的数据库生成不同的 SQL
stmt := db.Find(&user, 1).Statement
stmt.SQL.String() //=> SELECT * FROM `users` WHERE `id` = $1 // PostgreSQL
stmt.SQL.String() //=> SELECT * FROM `users` WHERE `id` = ?  // MySQL
stmt.Vars         //=> []interface{}{1}
```

你可以使用下面的代码生成最终的 SQL：

```go
// 注意：SQL 并不总是能安全地执行，GORM 仅将其用于日志，它可能导致会 SQL 注入
db.Dialector.Explain(stmt.SQL.String(), stmt.Vars...)
// SELECT * FROM `users` WHERE `id` = 1
```

## 预编译

`PreparedStmt` 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率，例如：

```go
// 全局模式，所有 DB 操作都会创建并缓存预编译语句
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  PrepareStmt: true,
})

tx := db.Session(&Session{PrepareStmt: true}) // 会话模式
tx.First(&user, 1)
tx.Find(&users)
tx.Model(&user).Update("Age", 18)

stmtManger, ok := tx.ConnPool.(*PreparedStmtDB) // returns prepared statements manager


stmtManger.Close() // 关闭 *当前会话* 的预编译模式

// 为 *当前会话* 预编译 SQL
stmtManger.PreparedSQL // => []string{}

// 为当前数据库连接池的（所有会话）开启预编译模式
stmtManger.Stmts // map[string]*sql.Stmt

for sql, stmt := range stmtManger.Stmts {
  sql  // 预编译 SQL
  stmt // 预编译模式
  stmt.Close() // 关闭预编译模式
}
```

## NewDB

通过 `NewDB` 选项创建一个不带之前条件的新 DB，例如：

```go
tx := db.Where("name = ?", "jinzhu").Session(&gorm.Session{NewDB: true})

tx.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1

tx.First(&user, "id = ?", 10)
// SELECT * FROM users WHERE id = 10 ORDER BY id

// 不带 `NewDB` 选项
tx2 := db.Where("name = ?", "jinzhu").Session(&gorm.Session{})
tx2.First(&user)
// SELECT * FROM users WHERE name = "jinzhu" ORDER BY id
```

## 初始化

创建一个新的初始化数据库，它不再是方法Method Chain/Goroutine安全的，请参阅[Method Chain](https://gorm.io/zh_CN/docs/method_chaining.html)

```go
tx := db.Session(&gorm.Session{Initialized: true})
```

## 跳过钩子

如果您想跳过 `钩子` 方法，您可以使用 `SkipHooks` 会话模式，例如：

```go
DB.Session(&gorm.Session{SkipHooks: true}).Create(&user)

DB.Session(&gorm.Session{SkipHooks: true}).Create(&users)

DB.Session(&gorm.Session{SkipHooks: true}).CreateInBatches(users, 100)

DB.Session(&gorm.Session{SkipHooks: true}).Find(&user)

DB.Session(&gorm.Session{SkipHooks: true}).Delete(&user)

DB.Session(&gorm.Session{SkipHooks: true}).Model(User{}).Where("age > ?", 18).Updates(&user)
```

## 禁用嵌套事务

在一个 DB 事务中使用 `Transaction` 方法，GORM 会使用 `SavePoint(savedPointName)`，`RollbackTo(savedPointName)` 为你提供嵌套事务支持。 你可以通过 `DisableNestedTransaction` 选项关闭它，例如：

```go
db.Session(&gorm.Session{
  DisableNestedTransaction: true,
}).CreateInBatches(&users, 100)
```

## AllowGlobalUpdate

GORM 默认不允许进行全局 update/delete，该操作会返回 `ErrMissingWhereClause` 错误。 您可以通过将一个选项设置为 true 来启用它，例如：

```go
db.Session(&gorm.Session{
  AllowGlobalUpdate: true,
}).Model(&User{}).Update("name", "jinzhu")
// UPDATE users SET `name` = "jinzhu"
```

## FullSaveAssociations

在创建、更新记录时，GORM 会通过 [Upsert](https://gorm.io/zh_CN/docs/create.html#upsert) 自动保存关联及其引用记录。 如果您想要更新关联的数据，您应该使用 `FullSaveAssociations` 模式，例如：

```go
db.Session(&gorm.Session{FullSaveAssociations: true}).Updates(&user)
// ...
// INSERT INTO "addresses" (address1) VALUES ("Billing Address - Address 1"), ("Shipping Address - Address 1") ON DUPLICATE KEY SET address1=VALUES(address1);
// INSERT INTO "users" (name,billing_address_id,shipping_address_id) VALUES ("jinzhu", 1, 2);
// INSERT INTO "emails" (user_id,email) VALUES (111, "jinzhu@example.com"), (111, "jinzhu-2@example.com") ON DUPLICATE KEY SET email=VALUES(email);
// ...
```

## Context

通过 `Context` 选项，您可以传入 `Context` 来追踪 SQL 操作，例如：

```go
timeoutCtx, _ := context.WithTimeout(context.Background(), time.Second)
tx := db.Session(&Session{Context: timeoutCtx})

tx.First(&user) // 带有 context timeoutCtx 的查询操作
tx.Model(&user).Update("role", "admin") // 带有 context timeoutCtx 的更新操作
```

GORM 也提供了简写形式的方法 `WithContext`，其实现如下：

```go
func (db *DB) WithContext(ctx context.Context) *DB {
  return db.Session(&Session{Context: ctx})
}
```

## 自定义 Logger

Gorm 允许使用 `Logger` 选项自定义内建 Logger，例如：

```go
newLogger := logger.New(log.New(os.Stdout, "\r\n", log.LstdFlags),
              logger.Config{
                SlowThreshold: time.Second,
                LogLevel:      logger.Silent,
                Colorful:      false,
              })
db.Session(&Session{Logger: newLogger})

db.Session(&Session{Logger: logger.Default.LogMode(logger.Silent)})
```

查看 [Logger](https://gorm.io/zh_CN/docs/logger.html) 获取更多信息.

## NowFunc

`NowFunc` 允许改变 GORM 获取当前时间的实现，例如：

```go
db.Session(&Session{
  NowFunc: func() time.Time {
    return time.Now().Local()
  },
})
```

## 调试

`Debug` 只是将会话的 `Logger` 修改为调试模式的简写形式，其实现如下：

```go
func (db *DB) Debug() (tx *DB) {
  return db.Session(&Session{
    Logger:         db.Logger.LogMode(logger.Info),
  })
}
```

## 查询字段

声明查询字段

```go
db.Session(&gorm.Session{QueryFields: true}).Find(&user)
// SELECT `users`.`name`, `users`.`age`, ... FROM `users` // 有该选项
// SELECT * FROM `users` // 没有该选项
```

## CreateBatchSize

默认批量大小

```go
users = [5000]User{{Name: "jinzhu", Pets: []Pet{pet1, pet2, pet3}}...}

db.Session(&gorm.Session{CreateBatchSize: 1000}).Create(&users)
// INSERT INTO users xxx (需 5 次)
// INSERT INTO pets xxx (需 15 次)
```



# 钩子

# 事务

# 数据库插入自定义数据类型

GORM 提供了少量接口，使用户能够为 GORM 定义支持的数据类型，这里以 [json](https://github.com/go-gorm/datatypes/blob/master/json.go) 为例

```go
type Teacher struct {
	Name     string  `gorm:"type:varchar(20);not null"`
	Email    string  `gorm:"type:varchar(20);not null"`
	Salary   float64 `gorm:"type:float"` // 设置为浮点数
	Age      uint8   `gorm:"type:int,check:age>30"`
	birthday int64   `gorm:"serializer:unixtime;type:time"`
	Roles    Roles   `gorm:"type:json"` // 修改为 json 类型
}
```

go run 时，报错（`sql: converting argument $1 type: unsupported type []string, a slice of string`）

分析原因：mysql中不支持`[]string` 类型的数据，不可以直接在结构体中定义`[]string`类型。

## 解决方案

基于`gorm` 库的特性，可以通过自定义`JSON`类型将数据传入，因此解决步骤一共分为两步：

- step 1：将`[]string`类型编码为`JSON`类型数据；
- step 2：自定义`JSON`类型数据，使得`gorm`可以支持，可以参考[GORM-自定义数据类型](https://link.juejin.cn?target=https%3A%2F%2Fgorm.io%2Fzh_CN%2Fdocs%2Fdata_types.html)

**step 1**： 首先将结构体中数据类型定义改为一个新的自定义类型

```go
type Roles []string

/**
 * @description: todos 数据表
 */
type Teacher struct {
	Name     string  `gorm:"type:varchar(20);not null"`
	Email    string  `gorm:"type:varchar(20);not null"`
	Salary   float64 `gorm:"type:float"` // 设置为浮点数
	Age      uint8   `gorm:"type:int,check:age>30"`
	birthday int64   `gorm:"serializer:unixtime;type:time"`
	Roles    Roles   `gorm:"type:json"` // 修改为 json 类型
}
```

**step 2**:自定义一个`JSON`类型，可以参考[GORM-自定义数据类型](https://link.juejin.cn?target=https%3A%2F%2Fgorm.io%2Fzh_CN%2Fdocs%2Fdata_types.html)

我们 step 1把Tag自定义为`type Tag []string`,但是这种类型在gorm中还不可以直接识别，所以需要进一步自定义数据类型。

```go
func (t *Tag) Scan(value interface{}) error {
	bytesValue, _ := value.([]byte)
	return json.Unmarshal(bytesValue, t)
}

func (t Tag) Value() (driver.Value, error) {
	return json.Marshal(t)
}
```

- `Scan()`存入数据库前转为 string
- `Value()`读出数据前转为 json 说白了就是，我们不是直接的把`[]string`类型的数据直接存入MySQL数据库，而是自定义成一个json类型，存储前，我们把它转为`string`,取出时，我们把转变为我们需要的`json`。（一共有3个步骤）就是这么一个简单的原理啦！

[gorm官方文档中的自定义数据类型](https://link.juejin.cn?target=https%3A%2F%2Fgorm.io%2Fzh_CN%2Fdocs%2Fdata_types.html)

GORM 提供了少量接口，让开发者能够自定义GORM 支持的数据类型

**实现自定义数据类型**由以下两个函数实现

- `Scan()`
- `Value()`

自定义的数据类型必须实现 [Scanner](https://link.juejin.cn?target=https%3A%2F%2Fpkg.go.dev%2Fdatabase%2Fsql%23Scanner) 和 [Valuer](https://link.juejin.cn?target=https%3A%2F%2Fpkg.go.dev%2Fdatabase%2Fsql%2Fdriver%23Valuer) 接口，以便让 GORM 知道如何将该类型接收、保存到数据库

```go
type Roles []string
type Teacher struct {
	Name     string  `gorm:"type:varchar(20);not null"`
	Email    string  `gorm:"type:varchar(20);not null"`
	Salary   float64 `gorm:"type:float"` // 设置为浮点数
	Age      uint8   `gorm:"type:int,check:age>30"`
	birthday int64   `gorm:"serializer:unixtime;type:time"`
	Roles    Roles   `gorm:"type:json"` // 修改为 json 类型
}

func (t *Roles) Scan(value interface{}) error {
	bytesValue, _ := value.([]byte)
	return json.Unmarshal(bytesValue, t)
}

func (t Roles) Value() (driver.Value, error) {
	return json.Marshal(t)
}
```



## 数据库json类型解析成go相应类型





# Docker快速创建MySQL实例

很多同学如果不会安装MySQL或者懒得安装MySQL，可以使用一下命令快速运行一个MySQL8.0.19实例，当然前提是你要有docker环境…

在本地的`13306`端口运行一个名为`mysql8019`，root用户名密码为`root1234`的MySQL容器环境:

```bash
docker run --name mysql8019 -p 13306:3306 -e MYSQL_ROOT_PASSWORD=root1234 -d mysql:8.0.19
```

在另外启动一个`MySQL Client`连接上面的MySQL环境，密码为上一步指定的密码`root1234`:

```bash
docker run -it --network host --rm mysql mysql -h127.0.0.1 -P13306 --default-character-set=utf8mb4 -uroot -p
```

在使用GORM前手动创建数据库`db1`：

```mysql
CREATE DATABASE db1;
```

