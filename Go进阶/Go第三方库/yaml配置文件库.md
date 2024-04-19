#  对 Go 语言的 YAML 支持

## 介绍

yaml 包使 Go 程序能够舒适地编码和解码 YAML 值。它是在[Canonical](https://www.canonical.com/)内部开发的 [Juju](https://juju.ubuntu.com/) 项目的一部分，基于 知名[libyaml](http://pyyaml.org/wiki/LibYAML) C库的纯Go端口，可快速可靠地解析和生成YAML数据。

## 安装和使用

包的导入路径为 *gopkg.in/yaml.v3*。

```go
go get gopkg.in/yaml.v3
```

## 接口文档

如果在浏览器中打开，导入路径本身会指向 API 文档：

- https://gopkg.in/yaml.v3

## 接口稳定性

yaml v3 的包 API 将保持稳定，如 [gopkg.in](https://gopkg.in/) 中所述。

```go
var data = `
a: Easy!
b:
  c: 2
  d: [3, 4]
`
// Note: struct fields must be public in order for unmarshal to
type T struct {// correctly populate the data.
        A string
        B struct {
                RenamedC int   `yaml:"c"`
                D        []int `yaml:",flow"`
        }
}

func main() {
        t := T{}
    
        err := yaml.Unmarshal([]byte(data), &t)
        if err != nil {
                log.Fatalf("error: %v", err)
        }
        fmt.Printf("--- t:\n%v\n\n", t)
    
        d, err := yaml.Marshal(&t)
        if err != nil {
                log.Fatalf("error: %v", err)
        }
        fmt.Printf("--- t dump:\n%s\n\n", string(d))
    
        m := make(map[interface{}]interface{})
    
        err = yaml.Unmarshal([]byte(data), &m)
        if err != nil {
                log.Fatalf("error: %v", err)
        }
        fmt.Printf("--- m:\n%v\n\n", m)
    
        d, err = yaml.Marshal(&m)
        if err != nil {
                log.Fatalf("error: %v", err)
        }
        fmt.Printf("--- m dump:\n%s\n\n", string(d))
}
```

此示例将生成以下输出：

```go
--- t:
{Easy! {2 [3 4]}}

--- t dump:
a: Easy!
b:
  c: 2
  d: [3, 4]


--- m:
map[a:Easy! b:map[c:2 d:[3 4]]]

--- m dump:
a: Easy!
b:
  c: 2
  d:
  - 3
  - 4
```