# 第三方logrus日志库使用

日志是程序中必不可少的一个环节，由于Go语言内置的日志库功能比较简洁，我们在实际开发中通常会选择使用第三方的日志库来进行开发。本文介绍了`logrus`这个日志库的基本使用。

## logrus介绍

Logrus是Go（golang）的结构化logger，与标准库logger完全API兼容。

它有以下特点：

- 完全兼容标准日志库，拥有七种日志级别：`Trace`, `Debug`, `Info`, `Warning`, `Error`, `Fatal`and `Panic`。
- 可扩展的Hook机制，允许使用者通过Hook的方式将日志分发到任意地方，如本地文件系统，logstash，elasticsearch或者mq等，或者通过Hook定义日志内容和格式等
- 可选的日志输出格式，内置了两种日志格式JSONFormater和TextFormatter，还可以自定义日志格式
- Field机制，通过Filed机制进行结构化的日志记录
- 线程安全

**安装**：`go get github.com/sirupsen/logrus`

## 基本示例

使用Logrus最简单的方法是简单的包级导出日志程序:

```go
import (
  log "github.com/sirupsen/logrus"
)

func main() {
  log.WithFields(log.Fields{
    "animal": "dog",
  }).Info("一条舔狗出现了。")
}
```

## 进阶示例

对于更高级的用法，例如在同一应用程序记录到多个位置，你还可以创建logrus Logger的实例：

```go
import (
  "os"
  "github.com/sirupsen/logrus"
)

var log = logrus.New() // 创建一个新的logger实例。可以创建任意多个。

func main() {
  log.Out = os.Stdout   // 设置日志输出为os.Stdout

  // 可以设置像文件等任意`io.Writer`类型作为日志输出
  // file, err := os.OpenFile("logrus.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
  // if err == nil {
  //  log.Out = file
  // } else {
  //  log.Info("Failed to log to file, using default stderr")
  // }

  log.WithFields(logrus.Fields{
    "animal": "dog",
    "size":   10,
  }).Info("一群舔狗出现了。")
}
```

## 日志级别

Logrus有七个日志级别：`Trace`, `Debug`, `Info`, `Warning`, `Error`, `Fatal`and `Panic`。

```go
log.Trace("Something very low level.")
log.Debug("Useful debugging information.")
log.Info("Something noteworthy happened!")
log.Warn("You should probably take a look at this.")
log.Error("Something failed but I'm not quitting.")
log.Fatal("Bye.") // 记完日志后会调用os.Exit(1) 
log.Panic("I'm bailing.") // 记完日志后会调用 panic() 
```

### 设置日志级别

你可以在Logger上设置日志记录级别，然后它只会记录具有该级别或以上级别任何内容的条目：

```go
log.SetLevel(log.InfoLevel) // 会记录info及以上级别 (warn, error, fatal, panic)
```

如果你的程序支持debug或环境变量模式，设置`log.Level = logrus.DebugLevel`会很有帮助。

## 字段

Logrus鼓励通过日志字段进行谨慎的结构化日志记录，而不是冗长的、不可解析的错误消息。

例如，区别于使用`log.Fatalf("Failed to send event %s to topic %s with key %d")`，你应该使用如下方式记录更容易发现的内容:

```go
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

`WithFields`的调用是可选的。

## 默认字段

通常，将一些字段始终附加到应用程序的全部或部分的日志语句中会很有帮助。例如，你可能希望始终在请求的上下文中记录`request_id`和`user_ip`。

区别于在每一行日志中写上`log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})`，你可以向下面的示例代码一样创建一个`logrus.Entry`去传递这些字段。

```go
requestLogger := log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})
requestLogger.Info("something happened on that request") # will log request_id and user_ip
requestLogger.Warn("something not great happened")
```

## 日志条目

除了使用`WithField`或`WithFields`添加的字段外，一些字段会自动添加到所有日志记录事中:

- time：记录日志时的时间戳
- msg：记录的日志信息
- level：记录的日志级别

## Hooks

你可以添加日志级别的钩子（Hook）。例如，向异常跟踪服务发送`Error`、`Fatal`和`Panic`、信息到StatsD或同时将日志发送到多个位置，例如syslog。

Logrus配有内置钩子。在`init`中添加这些内置钩子或你自定义的钩子：

```go
import (
  log "github.com/sirupsen/logrus"
  "gopkg.in/gemnasium/logrus-airbrake-hook.v2" // the package is named "airbrake"
  logrus_syslog "github.com/sirupsen/logrus/hooks/syslog"
  "log/syslog"
)

func init() {
  // Use the Airbrake hook to report errors that have Error severity or above to
  // an exception tracker. You can create custom hooks, see the Hooks section.
  log.AddHook(airbrake.NewHook(123, "xyz", "production"))
  hook, err := logrus_syslog.NewSyslogHook("udp", "localhost:514", syslog.LOG_INFO, "")
  if err != nil {
    log.Error("Unable to connect to local syslog daemon")
  } else {
    log.AddHook(hook)
  }
}
```

意：Syslog钩子还支持连接到本地syslog（例如. “/dev/log” or “/var/run/syslog” or “/var/run/log”)。有关详细信息，请查看[syslog hook README](https://github.com/sirupsen/logrus/blob/master/hooks/syslog/README.md)。

## 格式化

logrus内置以下两种日志格式化程序：

```
logrus.TextFormatter` `logrus.JSONFormatter
```

还支持一些第三方的格式化程序，详见项目首页。

## 记录函数名

如果你希望将调用的函数名添加为字段，请通过以下方式设置：

```go
log.SetReportCaller(true)
```

这会将调用者添加为”method”，如下所示：

```json
{"animal":"penguin","level":"fatal","method":"github.com/sirupsen/arcticcreatures.migrate","msg":"a penguin swims by",
"time":"2014-03-10 19:57:38.562543129 -0400 EDT"}
```

**注意：**，开启这个模式会增加性能开销。

## 线程安全

默认的logger在并发写的时候是被mutex保护的，比如当同时调用hook和写log时mutex就会被请求，有另外一种情况，文件是以appending mode打开的， 此时的并发操作就是安全的，可以用`logger.SetNoLock()`来关闭它。

## gin框架使用logrus

```go
var log = logrus.New()

func init() {
	log.Formatter = &logrus.JSONFormatter{} // Log as JSON instead of the default ASCII formatter.
	// Output to stdout instead of the default stderr
	// Can be any io.Writer, see below for File example
	f, _ := os.Create("./gin.log")
	log.Out = f
	gin.SetMode(gin.ReleaseMode)
	gin.DefaultWriter = log.Out
	
	log.Level = logrus.InfoLevel // Only log the warning severity or above.
}

func main() {
	r := gin.Default() // 创建一个默认的路由引擎
	// 当客户端以GET方法请求/hello路径时，会执行后面的匿名函数
	r.GET("/hello", func(c *gin.Context) { // GET：请求方式；/hello：请求的路径
		log.WithFields(logrus.Fields{
			"animal": "walrus",
			"size":   10,
		}).Warn("A group of walrus emerges from the ocean")
		c.JSON(200, gin.H{ // c.JSON：返回JSON格式的数据
			"message": "Hello world!",
		})
	})
	
	r.Run() // 启动HTTP服务，默认在0.0.0.0:8080启动服务
}
```