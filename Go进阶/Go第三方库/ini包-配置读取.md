[go-ini/ini 官方文档](https://ini.unknwon.io/docs/intro/getting_started)	ini包提供了Go语言中INI文件的读写功能。

## 产品特点

- 从多个数据源（文件、`[]byte`、`io.Reader`和`io.ReadCloser`）加载并覆盖。
- 使用递归值读取。
- 以父子分段阅读。
- 使用自动递增密钥名称读取。
- 读取多行值。
- 阅读大量的帮助方法。
- 读取值并将其转换为Go类型。
- 读取和写入节和键的注释。
- 轻松操作节、键和注释。
- 在解析和保存时，保持节和键的顺序。

## 安装

围棋的最低要求是1.13。

```sh
$ go get gopkg.in/ini.v1
```

请添加`-u`标志以便将来更新。

## 开始使用

我们将通过一个非常简单的例子来了解如何使用。

首先，我们需要在任意目录创建两个文件（`my.ini` 和 `main.go`），在这里我们选择 `/tmp/ini` 目录。

```sh
$ mkdir -p /tmp/ini
$ cd /tmp/ini
$ touch my.ini main.go
$ tree .
.
├── main.go
└── my.ini

0 directories, 2 files
```

现在，我们编辑 `my.ini` 文件并输入以下内容（_部分内容来自 Grafana_）。

```ini
# possible values : production, development
app_mode = development

[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
data = /home/git/grafana

[server]
# Protocol (http or https)
protocol = http

# The http port  to use
http_port = 9999

# Redirect to correct domain if host header does not match domain
# Prevents DNS rebinding attacks
enforce_domain = true
```

很好，接下来我们需要编写 `main.go` 文件来操作刚才创建的配置文件。

```go
func main() {
    cfg, err := ini.Load("my.ini")
    if err != nil {
        fmt.Printf("Fail to read file: %v", err)
        os.Exit(1)
    }

    // 典型读取操作，默认分区可以使用空字符串表示
    fmt.Println("App Mode:", cfg.Section("").Key("app_mode").String())
    fmt.Println("Data Path:", cfg.Section("paths").Key("data").String())

    // 我们可以做一些候选值限制的操作
    fmt.Println("Server Protocol:",
        cfg.Section("server").Key("protocol").In("http", []string{"http", "https"}))
    // 如果读取的值不在候选列表内，则会回退使用提供的默认值
    fmt.Println("Email Protocol:",
        cfg.Section("server").Key("protocol").In("smtp", []string{"imap", "smtp"}))

    // 试一试自动类型转换
    fmt.Printf("Port Number: (%[1]T) %[1]d\n", cfg.Section("server").Key("http_port").MustInt(9999))
    fmt.Printf("Enforce Domain: (%[1]T) %[1]v\n", cfg.Section("server").Key("enforce_domain").MustBool(false))

    // 差不多了，修改某个值然后进行保存
    cfg.Section("").Key("app_mode").SetValue("production")
    cfg.SaveTo("my.ini.local")
}
```

运行程序，我们可以看下以下输出：

```sh
$ go run main.go
App Mode: development
Data Path: /home/git/grafana
Server Protocol: http
Email Protocol: smtp
Port Number: (int) 9999
Enforce Domain: (bool) true

$ cat my.ini.local
# possible values : production, development
app_mode = production

[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
data = /home/git/grafana
...
```

完美！这个例子很简单，展示的也只是极其小部分的功能，想要完全掌握还需要多读多看，毕竟学无止境嘛。

