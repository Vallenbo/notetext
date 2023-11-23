# Go语言操作mongoDB

mongoDB是目前比较流行的一个基于分布式文件存储的数据库，它是一个介于关系数据库和非关系数据库(NoSQL)之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

# mongoDB介绍

[mongoDB](https://www.mongodb.com/)是目前比较流行的一个基于分布式文件存储的数据库，它是一个介于关系数据库和非关系数据库(NoSQL)之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。

mongoDB中将一条数据存储为一个文档（document），数据结构由键值（key-value）对组成。 其中文档类似于我们平常编程中用到的JSON对象。 文档中的字段值可以包含其他文档，数组及文档数组。

## mongoDB相关概念

mongoDB中相关概念与我们熟悉的SQL概念对比如下：

| MongoDB术语/概念 |                说明                 | 对比SQL术语/概念 |
| :--------------: | :---------------------------------: | :--------------: |
|     database     |               数据库                |     database     |
|    collection    |                集合                 |      table       |
|     document     |                文档                 |       row        |
|      field       |                字段                 |      column      |
|      index       |                index                |       索引       |
|   primary key    | 主键 MongoDB自动将_id字段设置为主键 |   primary key    |



# Go语言操作mongoDB

我们这里使用的是官方的驱动包，当然你也可以使用第三方的驱动包（如mgo等）。 mongoDB官方版的Go驱动发布的比较晚（2018年12月13号）。

## 安装mongoDB Go驱动包

```bash
go get github.com/mongodb/mongo-go-driver
```

## 通过Go代码连接mongoDB

```go
package main

import (
	"context"
	"fmt"
	"log"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

func main() {
	// 设置客户端连接配置
	clientOptions := options.Client().ApplyURI("mongodb://localhost:27017")

	// 连接到MongoDB
	client, err := mongo.Connect(context.TODO(), clientOptions)
	if err != nil {
		log.Fatal(err)
	}

	// 检查连接
	err = client.Ping(context.TODO(), nil)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("Connected to MongoDB!")
}
```

连接上MongoDB之后，可以通过下面的语句处理我们上面的q1mi数据库中的student数据集了：

```go
// 指定获取要操作的数据集
collection := client.Database("q1mi").Collection("student")
```

处理完任务之后可以通过下面的命令断开与MongoDB的连接：

```go
// 断开连接
err = client.Disconnect(context.TODO())
if err != nil {
	log.Fatal(err)
}
fmt.Println("Connection to MongoDB closed.")
```

## 连接池模式

```go
import (
	"context"
	"time"

	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
)

func ConnectToDB(uri, name string, timeout time.Duration, num uint64) (*mongo.Database, error) {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()
	o := options.Client().ApplyURI(uri)
	o.SetMaxPoolSize(num)
	client, err := mongo.Connect(ctx, o)
	if err != nil {
		return nil, err
	}

	return client.Database(name), nil
}
```

## BSON

MongoDB中的JSON文档存储在名为BSON(二进制编码的JSON)的二进制表示中。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示，使其包含额外的类型，如int、long、date、浮点数和decimal128。这使得应用程序更容易可靠地处理、排序和比较数据。

连接MongoDB的Go驱动程序中有两大类型表示BSON数据：`D`和`Raw`。

类型`D`家族被用来简洁地构建使用本地Go类型的BSON对象。这对于构造传递给MongoDB的命令特别有用。`D`家族包括四类:

- D：一个BSON文档。这种类型应该在顺序重要的情况下使用，比如MongoDB命令。
- M：一张无序的map。它和D是一样的，只是它不保持顺序。
- A：一个BSON数组。
- E：D里面的一个元素。

要使用BSON，需要先导入下面的包：

```go
import "go.mongodb.org/mongo-driver/bson"
```

下面是一个使用D类型构建的过滤器文档的例子，它可以用来查找name字段与’张三’或’李四’匹配的文档:

```go
bson.D{{
	"name",
	bson.D{{
		"$in",
		bson.A{"张三", "李四"},
	}},
}}
```

`Raw`类型家族用于验证字节切片。你还可以使用`Lookup()`从原始类型检索单个元素。如果你不想要将BSON反序列化成另一种类型的开销，那么这是非常有用的。这个教程我们将只使用D类型。

## CRUD

我们现在Go代码中定义一个`Studet`类型如下：

```go
type Student struct {
	Name string
	Age int
}
```

接下来，创建一些`Student`类型的值，准备插入到数据库中：

```go
s1 := Student{"小红", 12}
s2 := Student{"小兰", 10}
s3 := Student{"小黄", 11}
```

### 插入文档

使用`collection.InsertOne()`方法插入一条文档记录：

```go
insertResult, err := collection.InsertOne(context.TODO(), s1)
if err != nil {
	log.Fatal(err)
}

fmt.Println("Inserted a single document: ", insertResult.InsertedID)
```

使用`collection.InsertMany()`方法插入多条文档记录：

```go
students := []interface{}{s2, s3}
insertManyResult, err := collection.InsertMany(context.TODO(), students)
if err != nil {
	log.Fatal(err)
}
fmt.Println("Inserted multiple documents: ", insertManyResult.InsertedIDs)
```

### 更新文档

`updateone()`方法允许你更新单个文档。它需要一个筛选器文档来匹配数据库中的文档，并需要一个更新文档来描述更新操作。你可以使用`bson.D`类型来构建筛选文档和更新文档:

```go
filter := bson.D{{"name", "小兰"}}

update := bson.D{
	{"$inc", bson.D{
		{"age", 1},
	}},
}
```

接下来，就可以通过下面的语句找到小兰，给他增加一岁了：

```go
updateResult, err := collection.UpdateOne(context.TODO(), filter, update)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Matched %v documents and updated %v documents.\n", updateResult.MatchedCount, updateResult.ModifiedCount)
```

### 查找文档

要找到一个文档，你需要一个filter文档，以及一个指向可以将结果解码为其值的指针。要查找单个文档，使用`collection.FindOne()`。这个方法返回一个可以解码为值的结果。

我们使用上面定义过的那个filter来查找姓名为’小兰’的文档。

```go
// 创建一个Student变量用来接收查询的结果
var result Student
err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Found a single document: %+v\n", result)
```

要查找多个文档，请使用`collection.Find()`。此方法返回一个游标。游标提供了一个文档流，你可以通过它一次迭代和解码一个文档。当游标用完之后，应该关闭游标。下面的示例将使用`options`包设置一个限制以便只返回两个文档。

```go
// 查询多个
// 将选项传递给Find()
findOptions := options.Find()
findOptions.SetLimit(2)

// 定义一个切片用来存储查询结果
var results []*Student

// 把bson.D{{}}作为一个filter来匹配所有文档
cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
if err != nil {
	log.Fatal(err)
}

// 查找多个文档返回一个光标
// 遍历游标允许我们一次解码一个文档
for cur.Next(context.TODO()) {
	// 创建一个值，将单个文档解码为该值
	var elem Student
	err := cur.Decode(&elem)
	if err != nil {
		log.Fatal(err)
	}
	results = append(results, &elem)
}

if err := cur.Err(); err != nil {
	log.Fatal(err)
}

// 完成后关闭游标
cur.Close(context.TODO())
fmt.Printf("Found multiple documents (array of pointers): %#v\n", results)
```

### 删除文档

最后，可以使用`collection.DeleteOne()`或`collection.DeleteMany()`删除文档。如果你传递`bson.D{{}}`作为过滤器参数，它将匹配数据集中的所有文档。还可以使用`collection. drop()`删除整个数据集。

```go
// 删除名字是小黄的那个
deleteResult1, err := collection.DeleteOne(context.TODO(), bson.D{{"name","小黄"}})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult1.DeletedCount)
// 删除所有
deleteResult2, err := collection.DeleteMany(context.TODO(), bson.D{{}})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult2.DeletedCount)
```

更多方法请查阅[官方文档](https://godoc.org/go.mongodb.org/mongo-driver)。

# 参考链接

https://docs.mongodb.com/manual/mongo/

https://www.mongodb.com/blog/post/mongodb-go-driver-tutorial