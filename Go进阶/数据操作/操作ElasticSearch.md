# 操作ElasticSearch

## elastic client

我们使用第三方库https://github.com/olivere/elastic 来连接ES并进行操作。

注意下载与你的ES相同版本的client，例如我们这里使用的ES是7.2.1的版本，那么我们下载的client也要与之对应为github.com/olivere/elastic/v7。

使用go.mod来管理依赖：

```
require (
	github.com/olivere/elastic/v7 v7.0.4
)
```

简单示例：

```go
package main

import (
    "context"
    "fmt"

    "github.com/olivere/elastic/v7"
)

// Elasticsearch demo
type Person struct {
    Name    string `json:"name"`
    Age     int    `json:"age"`
    Married bool   `json:"married"`
}

func main() {
    client, err := elastic.NewClient(elastic.SetURL("http://127.0.0.1:9200"))
    if err != nil {
        // Handle error
        panic(err)
    }

    fmt.Println("connect to es success")
    p1 := Person{Name: "lmh", Age: 18, Married: false}
    put1, err := client.Index().
        Index("user").
        BodyJson(p1).
        Do(context.Background())
    if err != nil {
        // Handle error
        panic(err)
    }
    fmt.Printf("Indexed user %s to index %s, type %s\n", put1.Id, put1.Index, put1.Type)
}
```

示例2：

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "reflect"

    "gopkg.in/olivere/elastic.v7" //这里使用的是版本5，最新的是6，有改动
)

var client *elastic.Client
var host = "http://127.0.0.1:9200/"

type Employee struct {
    FirstName string   `json:"first_name"`
    LastName  string   `json:"last_name"`
    Age       int      `json:"age"`
    About     string   `json:"about"`
    Interests []string `json:"interests"`
}

//初始化
func init() {
    errorlog := log.New(os.Stdout, "APP", log.LstdFlags)
    var err error
    client, err = elastic.NewClient(elastic.SetErrorLog(errorlog), elastic.SetURL(host))
    if err != nil {
        panic(err)
    }
    info, code, err := client.Ping(host).Do(context.Background())
    if err != nil {
        panic(err)
    }
    fmt.Printf("Elasticsearch returned with code %d and version %s\n", code, info.Version.Number)

    esversion, err := client.ElasticsearchVersion(host)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Elasticsearch version %s\n", esversion)

}

/*下面是简单的CURD*/
//创建
func create() {
    //使用结构体
    e1 := Employee{"Jane", "Smith", 32, "I like to collect rock albums", []string{"music"}}
    put1, err := client.Index().
        Index("megacorp").
        Type("employee").
        Id("1").
        BodyJson(e1).
        Do(context.Background())
    if err != nil {
        panic(err)
    }
    fmt.Printf("Indexed tweet %s to index s%s, type %s\n", put1.Id, put1.Index, put1.Type)

    //使用字符串
    e2 := `{"first_name":"John","last_name":"Smith","age":25,"about":"I love to go rock climbing","interests":["sports","music"]}`
    put2, err := client.Index().
        Index("megacorp").
        Type("employee").
        Id("2").
        BodyJson(e2).
        Do(context.Background())
    if err != nil {
        panic(err)
    }
    fmt.Printf("Indexed tweet %s to index s%s, type %s\n", put2.Id, put2.Index, put2.Type)

    e3 := `{"first_name":"Douglas","last_name":"Fir","age":35,"about":"I like to build cabinets","interests":["forestry"]}`
    put3, err := client.Index().
        Index("megacorp").
        Type("employee").
        Id("3").
        BodyJson(e3).
        Do(context.Background())
    if err != nil {
        panic(err)
    }
    fmt.Printf("Indexed tweet %s to index s%s, type %s\n", put3.Id, put3.Index, put3.Type)

}

//删除
func delete() {
    res, err := client.Delete().Index("megacorp").
        Type("employee").
        Id("1").
        Do(context.Background())
    if err != nil {
        println(err.Error())
        return
    }
    fmt.Printf("delete result %s\n", res.Result)
}

//修改
func update() {
    res, err := client.Update().
        Index("megacorp").
        Type("employee").
        Id("2").
        Doc(map[string]interface{}{"age": 88}).
        Do(context.Background())
    if err != nil {
        println(err.Error())
    }
    fmt.Printf("update age %s\n", res.Result)
}

//查找
func gets() {
    //通过id查找
    get1, err := client.Get().Index("megacorp").Type("employee").Id("2").Do(context.Background())
    if err != nil {
        panic(err)
    }
    if get1.Found {
        fmt.Printf("Got document %s in version %d from index %s, type %s\n", get1.Id, get1.Version, get1.Index, get1.Type)
    }
}

//搜索
func query() {
    var res *elastic.SearchResult
    var err error
    //取所有
    res, err = client.Search("megacorp").Type("employee").Do(context.Background())
    printEmployee(res, err)

    //字段相等
    q := elastic.NewQueryStringQuery("last_name:Smith")
    res, err = client.Search("megacorp").Type("employee").Query(q).Do(context.Background())
    if err != nil {
        println(err.Error())
    }
    printEmployee(res, err)

    //条件查询
    //年龄大于30岁的
    boolQ := elastic.NewBoolQuery()
    boolQ.Must(elastic.NewMatchQuery("last_name", "smith"))
    boolQ.Filter(elastic.NewRangeQuery("age").Gt(30))
    res, err = client.Search("megacorp").Type("employee").Query(q).Do(context.Background())
    printEmployee(res, err)

    //短语搜索 搜索about字段中有 rock climbing
    matchPhraseQuery := elastic.NewMatchPhraseQuery("about", "rock climbing")
    res, err = client.Search("megacorp").Type("employee").Query(matchPhraseQuery).Do(context.Background())
    printEmployee(res, err)

    //分析 interests
    aggs := elastic.NewTermsAggregation().Field("interests")
    res, err = client.Search("megacorp").Type("employee").Aggregation("all_interests", aggs).Do(context.Background())
    printEmployee(res, err)

}

//简单分页
func list(size, page int) {
    if size < 0 || page < 1 {
        fmt.Printf("param error")
        return
    }
    res, err := client.Search("megacorp").
        Type("employee").
        Size(size).
        From((page - 1) * size).
        Do(context.Background())
    printEmployee(res, err)

}

//打印查询到的Employee
func printEmployee(res *elastic.SearchResult, err error) {
    if err != nil {
        print(err.Error())
        return
    }
    var typ Employee
    for _, item := range res.Each(reflect.TypeOf(typ)) { //从搜索结果中取数据的方法
        t := item.(Employee)
        fmt.Printf("%#v\n", t)
    }
}

func main() {
    create()
    delete()
    update()
    gets()
    query()
    list(1, 3)
}
```

更多使用详见文档：https://godoc.org/github.com/olivere/elastic