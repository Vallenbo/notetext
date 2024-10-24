# Zap日志库

本文先介绍了Go语言原生的日志库的使用，然后详细介绍了非常流行的Uber开源的zap日志库，同时介绍了如何搭配Lumberjack实现日志的切割和归档。

## 介绍

在许多Go语言项目中，我们需要一个好的日志记录器能够提供下面这些功能：

- 能够将事件记录到文件中，而不是应用程序控制台。
- 可以设置任何`io.Writer`作为日志记录输出并向其发送要写入的日志
- 日志切割-能够根据文件大小、时间或间隔等来切割日志文件。
- 支持不同的日志级别。例如INFO，DEBUG，ERROR等。
- 能够打印基本信息，如调用文件/函数名和行号，日志时间等。

缺点：

- Zap本身不支持切割归档日志文件

## 默认的Go Logger

在介绍Uber-go的zap包之前，让我们先看看Go语言提供的基本日志功能。Go语言提供的默认日志包是https://golang.org/pkg/log/。

实现一个Go语言中的日志记录器非常简单——创建一个新的日志文件，然后设置它为日志的输出位置。

### Go Logger的优势和劣势

**优势**

它最大的优点是使用非常简单。我们可以设置任何`io.Writer`作为日志记录输出并向其发送要写入的日志。

**劣势**

- 仅限基本的日志级别
  - 只有一个`Print`选项。不支持`INFO`/`DEBUG`等多个级别。
- 对于错误日志，它有`Fatal`和`Panic`
  - Fatal日志通过调用`os.Exit(1)`来结束程序
  - Panic日志在写入日志消息之后抛出一个panic
  - 但是它缺少一个ERROR日志级别，这个级别可以在不抛出panic或退出程序的情况下记录错误
- 缺乏日志格式化的能力——例如记录调用者的函数名和行号，格式化日期和时间格式。等等。
- 不提供日志切割的能力。

## Uber-go Zap

[Zap](https://github.com/uber-go/zap)是非常快的、结构化的，分日志级别的Go日志库。

### 为什么选择Uber-go zap

- 它同时提供了结构化日志记录和printf风格的日志记录
- 它非常的快

根据Uber-go Zap的文档，它的性能比类似的结构化日志包更好——也比标准库更快。 以下是Zap发布的基准测试信息

记录一条消息和10个字段:

|     Package     |    Time     | Time % to zap | Objects Allocated |
| :-------------: | :---------: | :-----------: | :---------------: |
|      ⚡️ zap      |  862 ns/op  |      +0%      |    5 allocs/op    |
| ⚡️ zap (sugared) | 1250 ns/op  |     +45%      |   11 allocs/op    |
|     zerolog     | 4021 ns/op  |     +366%     |   76 allocs/op    |
|     go-kit      | 4542 ns/op  |     +427%     |   105 allocs/op   |
|    apex/log     | 26785 ns/op |    +3007%     |   115 allocs/op   |
|     logrus      | 29501 ns/op |    +3322%     |   125 allocs/op   |
|      log15      | 29906 ns/op |    +3369%     |   122 allocs/op   |

记录一个静态字符串，没有任何上下文或printf风格的模板：

|     Package      |    Time    | Time % to zap | Objects Allocated |
| :--------------: | :--------: | :-----------: | :---------------: |
|      ⚡️ zap       | 118 ns/op  |      +0%      |    0 allocs/op    |
| ⚡️ zap (sugared)  | 191 ns/op  |     +62%      |    2 allocs/op    |
|     zerolog      |  93 ns/op  |     -21%      |    0 allocs/op    |
|      go-kit      | 280 ns/op  |     +137%     |   11 allocs/op    |
| standard library | 499 ns/op  |     +323%     |    2 allocs/op    |
|     apex/log     | 1990 ns/op |    +1586%     |   10 allocs/op    |
|      logrus      | 3129 ns/op |    +2552%     |   24 allocs/op    |
|      log15       | 3887 ns/op |    +3194%     |   23 allocs/op    |

### 安装

运行下面的命令安装zap

```bash
go get -u go.uber.org/zap
```

## Zap Logger日志记录器

Zap提供了两种类型的日志记录器—`Sugared Logger`和`Logger`。

在性能很好但不是很关键的上下文中，使用`SugaredLogger`。它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。

在每一微秒和每一次内存分配都很重要的上下文中，使用`Logger`。它甚至比`SugaredLogger`更快，内存分配次数也更少，但它只支持强类型的结构化日志记录。

**基本使用**

```go
func main() {
	// 获取Logger对象
	// zap.NewExample()
	// zap.NewDevelopment()
	logger, err := zap.NewProduction() // 生产环境的对象
	if err != nil {
		panic(err)
	}
	// 记录日志数据
	var uid int64 = 18967553
	isLogin := true
	name := "杨俊"
	data := []int{1, 2}

	// 默认是输出JSON格式，日志会输出到 标准输出（终端）
	logger.Info(
		"日志信息",
		zap.Int64("uid", uid),
		zap.Bool("isLogin", isLogin),
		zap.String("name", name),
		// zap.Any("data", data), //any任意类型
		zap.Ints("data", data),
	)
}

输出：
{"level":"info","ts":1712626153.978257,"caller":"日志库使用/zap使用.go:22","msg":"日志信息","uid":18967553,"isLogin":true,"name":"杨俊","data":[1,2]}
```

### Logger

- 通过调用`zap.NewProduction()` / `zap.NewDevelopment()` / `zap.Example()`创建一个Logger对象。

​	上面的每一个函数都将创建一个logger。唯一的区别在于它将记录的信息不同。例如production logger默认记录调用函数信息、日期和时间等。

​	通过Logger调用Info/Error等。

- 默认情况下日志都会打印到应用程序的console界面。

```go
func main() {
	logger, _ = zap.NewProduction()
	defer logger.Sync()
	simpleHttpGet("www.google.com")
	simpleHttpGet("http://www.google.com")
}

func simpleHttpGet(url string) {
	resp, err := http.Get(url)
	if err != nil {
		logger.Error(
			"Error fetching url..",
			zap.String("url", url),
			zap.Error(err))
	} else {
		logger.Info("Success..",
			zap.String("statusCode", resp.Status),
			zap.String("url", url))
		resp.Body.Close()
	}
}
```

在上面的代码中，我们首先创建了一个Logger，然后使用Info/ Error等Logger方法记录消息。

日志记录器方法的语法是这样的：

```go
func (log *Logger) MethodXXX(msg string, fields ...Field) 
```

其中`MethodXXX`是一个可变参数函数，可以是Info / Error/ Debug / Panic等。每个方法都接受一个消息字符串和任意数量的`zapcore.Field`场参数。

每个`zapcore.Field`其实就是一组键值对参数。

我们执行上面的代码会得到如下输出结果：

```bash
{"level":"error","ts":1572159218.912792,"caller":"zap_demo/temp.go:25","msg":"Error fetching url..","url":"www.sogo.com","error":"Get www.sogo.com: unsupported protocol scheme \"\"","stacktrace":"main.simpleHttpGet\n\t/Users/q1mi/zap_demo/temp.go:25\nmain.main\n\t/Users/q1mi/zap_demo/temp.go:14\nruntime.main\n\t/usr/local/go/src/runtime/proc.go:203"}
{"level":"info","ts":1572159219.1227388,"caller":"zap_demo/temp.go:30","msg":"Success..","statusCode":"200 OK","url":"http://www.sogo.com"}
```

### Sugared Logger

简化版日志对象生成器，现在让我们使用Sugared Logger来实现相同的功能。

- 大部分的实现基本都相同。
- 惟一的区别是，我们通过调用主logger的`. Sugar()`方法来获取一个`SugaredLogger`。
- 然后使用`SugaredLogger`以`printf`格式记录语句

下面是修改过后使用`SugaredLogger`代替`Logger`的代码：

```go
func main() {
	logger, _ := zap.NewProduction()
	sugarLogger = logger.Sugar()
	defer sugarLogger.Sync()
    
	simpleHttpGet("www.google.com")
	simpleHttpGet("http://www.google.com")
}

func simpleHttpGet(url string) {
	sugarLogger.Debugf("Trying to hit GET request for %s", url) // Debugf拼接
	resp, err := http.Get(url)
	if err != nil {
		sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
	} else {
		sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
		resp.Body.Close()
	}
}
```

当你执行上面的代码会得到如下输出：

```bash
{"level":"error","ts":1572159149.923002,"caller":"logic/temp2.go:27","msg":"Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme \"\"","stacktrace":"main.simpleHttpGet\n\t/Users/q1mi/zap_demo/logic/temp2.go:27\nmain.main\n\t/Users/q1mi/zap_demo/logic/temp2.go:14\nruntime.main\n\t/usr/local/go/src/runtime/proc.go:203"}
{"level":"info","ts":1572159150.192585,"caller":"logic/temp2.go:29","msg":"Success! statusCode = 200 OK for URL http://www.sogo.com"}
```

你应该注意到的了，到目前为止这两个logger都打印输出JSON结构格式。

在本博客的后面部分，我们将更详细地讨论SugaredLogger，并了解如何进一步配置它。

## 定制logger

### 将日志写入文件而不是终端

我们要做的第一个更改是把日志写入文件，而不是打印到应用程序控制台。

- 我们将使用`zap.New(…)`方法来手动传递所有配置，而不是使用像`zap.NewProduction()`这样的预置方法来创建logger。

```go
func New(core zapcore.Core, options ...Option) *Logger
```

`zapcore.Core`需要三个配置——`Encoder`，`WriteSyncer`，`LogLevel`。

1.**Encoder**:编码器(日志格式选择，如json)。我们将使用开箱即用的`NewJSONEncoder()`，并使用预先设置的`ProductionEncoderConfig()`。

```go
zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
```

2.**WriterSyncer** ：指定日志将写入位置（文件、网络...）。我们使用`zapcore.AddSync()`函数并且将打开的文件句柄传进去。

```go
file, _ := os.Create("./test.log")
writeSyncer := zapcore.AddSync(file)
```

3.**Log Level**：哪种级别的日志将被写入。

我们将修改上述部分中的Logger代码，并重写`InitLogger()`方法。其余方法—`main()` /`SimpleHttpGet()`保持不变。

```go
func getEncoder() zapcore.Encoder {
	return zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
}

func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test.log")
	return zapcore.AddSync(file)
}

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

	logger := zap.New(core)
	sugarLogger = logger.Sugar()
}
```

当使用这些修改过的logger配置调用上述部分的`main()`函数时，以下输出将打印在文件——`test.log`中。

```bash
{"level":"debug","ts":1572160754.994731,"msg":"Trying to hit GET request for www.sogo.com"}
{"level":"error","ts":1572160754.994982,"msg":"Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme \"\""}
{"level":"debug","ts":1572160754.994996,"msg":"Trying to hit GET request for http://www.sogo.com"}
{"level":"info","ts":1572160757.3755069,"msg":"Success! statusCode = 200 OK for URL http://www.sogo.com"}
```

### 将JSON Encoder更改为普通的Log Encoder

现在，我们希望将编码器从JSON Encoder更改为普通Encoder。为此，我们需要将`NewJSONEncoder()`更改为`NewConsoleEncoder()`。

```go
return zapcore.NewConsoleEncoder(zap.NewProductionEncoderConfig())
```

当使用这些修改过的logger配置调用上述部分的`main()`函数时，以下输出将打印在文件——`test.log`中。

```bash
1.572161051846623e+09	debug	Trying to hit GET request for www.sogo.com
1.572161051846828e+09	error	Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme ""
1.5721610518468401e+09	debug	Trying to hit GET request for http://www.sogo.com
1.572161052068744e+09	info	Success! statusCode = 200 OK for URL http://www.sogo.com
```

### 更改时间编码并添加调用者详细信息

鉴于我们对配置所做的更改，有下面两个问题：

- 时间是以非人类可读的方式展示，例如1.572161051846623e+09
- 调用方函数的详细信息没有显示在日志中

我们要做的第一件事是覆盖默认的`ProductionConfig()`，并进行以下更改:

- 修改时间编码器
- 在日志文件中使用大写字母记录日志级别

```go
func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder  //时间格式码
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder //大写 日志级别
	return zapcore.NewConsoleEncoder(encoderConfig)
}
```

接下来，我们将修改zap logger代码，添加将调用函数信息记录到日志中的功能。为此，我们将在`zap.New(..)`函数中添加一个`Option`。

```go
logger := zap.New(core, zap.AddCaller()) // AddCaller()显示所有调用信息
```

当使用这些修改过的logger配置调用上述部分的`main()`函数时，以下输出将打印在文件——`test.log`中。

```bash
2019-10-27T15:33:29.855+0800	DEBUG	logic/temp2.go:47	Trying to hit GET request for www.sogo.com
2019-10-27T15:33:29.855+0800	ERROR	logic/temp2.go:50	Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme ""
2019-10-27T15:33:29.856+0800	DEBUG	logic/temp2.go:47	Trying to hit GET request for http://www.sogo.com
2019-10-27T15:33:30.125+0800	INFO	logic/temp2.go:52	Success! statusCode = 200 OK for URL http://www.sogo.com
```

### AddCallerSkip

当我们不是直接使用初始化好的logger实例记录日志，而是将其包装成一个函数等，此时日录日志的函数调用链会增加（即会调用包的文件代码位置），想要获得准确的调用信息就需要通过`AddCallerSkip`函数来跳过。

```go
logger := zap.New(core, zap.AddCaller(), zap.AddCallerSkip(1))
```

### 将日志输出到多个位置

我们可以将日志同时输出到文件和终端。

```go
func getLogWriter() zapcore.WriteSyncer {
	file, _ := os.Create("./test.log")
	ws := io.MultiWriter(file, os.Stdout) // 利用io.MultiWriter支持文件和终端两个输出目标
	return zapcore.AddSync(ws)
}
```

### 将err日志单独输出到文件

有时候我们除了将全量日志输出到`xx.log`文件中之外，还希望将`ERROR`级别的日志单独输出到一个名为`xx.err.log`的日志文件中。我们可以通过以下方式实现。

```go
func InitLogger() {
	encoder := getEncoder()
	
	logF, _ := os.Create("./test.log") // test.log记录全量日志
	c1 := zapcore.NewCore(encoder, zapcore.AddSync(logF), zapcore.DebugLevel)
	
	errF, _ := os.Create("./test.err.log") // test.err.log记录ERROR级别的日志
	c2 := zapcore.NewCore(encoder, zapcore.AddSync(errF), zap.ErrorLevel)
	
	core := zapcore.NewTee(c1, c2) // 使用NewTee将c1和c2合并到core
	logger = zap.New(core, zap.AddCaller())
}
```

# Lumberjack进行日志切割归档

`lumberjack`是一个 Go 语言的日志轮转库，主要用于管理日志文件的大小、备份数量和保留时间等。

官方的说法是为了添加日志切割归档功能，我们将使用第三方库[Lumberjack](https://github.com/natefinch/lumberjack)来实现。

```go
go get gopkg.in/natefinch/lumberjack.v2
```

**主要特点：**

**日志轮转**：可以根据设定的规则自动对日志文件进行轮转。例如，当日志文件达到一定大小、经过一定时间或者满足其他条件时，会创建一个新的日志文件，并将旧的日志文件进行归档备份。

**灵活配置：**

- `Filename`：指定日志文件的路径和名称。
- `MaxSize`：设置单个日志文件的最大大小（单位通常是兆字节），当达到这个大小后会进行日志轮转。
- `MaxBackups`：指定保留的日志备份文件数量。当超过这个数量时，最旧的备份文件会被删除。
- `MaxAge`：设置日志文件的最长保留时间（单位通常是天），超过这个时间的日志文件会被删除。
- `Compress`：可以设置是否对旧的日志文件进行压缩存储，以节省磁盘空间。

使用`lumberjack`库可以有效地管理日志文件，避免日志文件无限制增长，确保系统在长时间运行过程中不会因为日志文件过大而占用过多磁盘空间或影响

## zap logger中加入Lumberjack

要在zap中加入Lumberjack支持，我们需要修改`WriteSyncer`代码。我们将按照下面的代码修改`getLogWriter()`函数：

```go
func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./test.log",  //Filename: 日志文件的位置
		MaxSize:    10,				//MaxSize：在进行切割之前，日志文件的最大大小（以MB为单位）
		MaxBackups: 5,				//MaxBackups：保留旧文件的最大个数
		MaxAge:     30,				//MaxAges：保留旧文件的最大天数
		Compress:   false,			//Compress：是否压缩/归档旧文件
	}
	return zapcore.AddSync(lumberJackLogger)
}
```

## 快速使用

```go
func Test_logrolate(t *testing.T) {
	// 设置日志格式为文本格式（你可以根据需要选择其他格式）
	log.SetFormatter(&log.TextFormatter{})

	// 使用 Lumberjack 进行日志轮转配置
	ljLogger := &lumberjack.Logger{
		Filename:   fmt.Sprintf("logs/%s.log", time.Now().Format("2006-01-02")),
		MaxSize:    10, // megabytes
		MaxBackups: 2,
		MaxAge:     30, // days
	}

	// 将日志输出设置为 Lumberjack 的写入器
	log.SetOutput(ljLogger)
	log.Info("This is a log message.")
	log.Warn("This is a warning message.")
}
```

## 测试所有功能

最终，使用Zap/Lumberjack logger的完整示例代码如下：

```go
var sugarLogger *zap.SugaredLogger

func main() {
	InitLogger()
	defer sugarLogger.Sync()
	simpleHttpGet("www.sogo.com")
	simpleHttpGet("http://www.sogo.com")
}

func InitLogger() {
	writeSyncer := getLogWriter()
	encoder := getEncoder()
	core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

	logger := zap.New(core, zap.AddCaller())
	sugarLogger = logger.Sugar()
}

func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
	return zapcore.NewConsoleEncoder(encoderConfig)
}

func getLogWriter() zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   "./test.log",
		MaxSize:    1,
		MaxBackups: 5,
		MaxAge:     30,
		Compress:   false,
	}
	return zapcore.AddSync(lumberJackLogger)
}

func simpleHttpGet(url string) {
	sugarLogger.Debugf("Trying to hit GET request for %s", url)
	resp, err := http.Get(url)
	if err != nil {
		sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
	} else {
		sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
		resp.Body.Close()
	}
}
```

执行上述代码，下面的内容会输出到文件——test.log中。

```go
2019-10-27T15:50:32.944+0800	DEBUG	logic/temp2.go:48	Trying to hit GET request for www.sogo.com
2019-10-27T15:50:32.944+0800	ERROR	logic/temp2.go:51	Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme ""
2019-10-27T15:50:32.944+0800	DEBUG	logic/temp2.go:48	Trying to hit GET request for http://www.sogo.com
2019-10-27T15:50:33.165+0800	INFO	logic/temp2.go:53	Success! statusCode = 200 OK for URL http://www.sogo.com
```

同时，可以在`main`函数中循环记录日志，测试日志文件是否会自动切割和归档（日志文件每1MB会切割并且在当前目录下最多保存5个备份）。

至此，我们总结了如何将Zap日志程序集成到Go应用程序项目中。



## 其他切割库

想按日期切割可以使用[github.com/lestrrat-go/file-rotatelogs](https://github.com/lestrrat-go/file-rotatelogs)这个库，虽然目前不维护了，但也够用了。

```go
import rotatelogs "github.com/lestrrat-go/file-rotatelogs"

l, _ := rotatelogs.New( // 使用file-rotatelogs按天切割日志
	filename+".%Y%m%d%H%M",
	rotatelogs.WithMaxAge(30*24*time.Hour),    // 最长保存30天
	rotatelogs.WithRotationTime(time.Hour*24), // 24小时切割一次
)
zapcore.AddSync(l)
```



# zap接收gin框架默认日志+归档

我们该如何在日志中记录gin框架本身输出的那些日志呢？

## gin默认日志中间件

首先我们来看一个最简单的gin项目：

```go
func main() {
	r := gin.Default()
	r.GET("/hello", func(c *gin.Context) {
		c.String(200, "hello ")
	})
	r.Run()
}
```

接下来我们看一下`gin.Default()`的源码：

```go
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```

也就是我们在使用`gin.Default()`的同时是用到了gin框架内的两个默认中间件`Logger()`和`Recovery()`。

其中`Logger()`是把gin框架本身的日志输出到标准输出（我们本地开发调试时在终端输出的那些日志就是它的功劳），而`Recovery()`是在程序出现panic的时候恢复现场并写入500响应的。

## gin日志基于zap

我们可以模仿`Logger()`和`Recovery()`的实现，使用我们的日志库来接收gin框架默认输出的日志。

这里以zap为例，我们实现两个中间件如下：

```go
// GinLogger 接收gin框架默认的日志
func GinLogger(logger *zap.Logger) gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()

		cost := time.Since(start)
		logger.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// GinRecovery recover掉项目可能出现的panic
func GinRecovery(logger *zap.Logger, stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					logger.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}

				if stack {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					logger.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}
```

*如果不想自己实现，可以使用github上有别人封装好的https://github.com/gin-contrib/zap。*

这样我们就可以在gin框架中使用我们上面定义好的两个中间件来代替gin框架默认的`Logger()`和`Recovery()`了。

```go
r := gin.New()
r.Use(GinLogger(), GinRecovery())
```

## gin中使用zap日志切割

最后我们再加入我们项目中常用的日志切割，完整版的`logger.go`代码如下：

```go
var lg *zap.Logger

// InitLogger 初始化Logger
func InitLogger(cfg *config.LogConfig) (err error) {
	writeSyncer := getLogWriter(cfg.Filename, cfg.MaxSize, cfg.MaxBackups, cfg.MaxAge)
	encoder := getEncoder()
	var l = new(zapcore.Level)
	err = l.UnmarshalText([]byte(cfg.Level))
	if err != nil {
		return
	}
	core := zapcore.NewCore(encoder, writeSyncer, l)

	lg = zap.New(core, zap.AddCaller())
	zap.ReplaceGlobals(lg) // 替换zap包中全局的logger实例，后续在其他包中只需使用zap.L()调用即可
	return
}

func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	encoderConfig.TimeKey = "time"
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
	encoderConfig.EncodeDuration = zapcore.SecondsDurationEncoder
	encoderConfig.EncodeCaller = zapcore.ShortCallerEncoder
	return zapcore.NewJSONEncoder(encoderConfig)
}

func getLogWriter(filename string, maxSize, maxBackup, maxAge int) zapcore.WriteSyncer {
	lumberJackLogger := &lumberjack.Logger{
		Filename:   filename,
		MaxSize:    maxSize,
		MaxBackups: maxBackup,
		MaxAge:     maxAge,
	}
	return zapcore.AddSync(lumberJackLogger)
}

// GinLogger 接收gin框架默认的日志
func GinLogger() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		c.Next()

		cost := time.Since(start)
		lg.Info(path,
			zap.Int("status", c.Writer.Status()),
			zap.String("method", c.Request.Method),
			zap.String("path", path),
			zap.String("query", query),
			zap.String("ip", c.ClientIP()),
			zap.String("user-agent", c.Request.UserAgent()),
			zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),
			zap.Duration("cost", cost),
		)
	}
}

// GinRecovery recover掉项目可能出现的panic，并使用zap记录相关日志
func GinRecovery(stack bool) gin.HandlerFunc {
	return func(c *gin.Context) {
		defer func() {
			if err := recover(); err != nil {
				// Check for a broken connection, as it is not really a
				// condition that warrants a panic stack trace.
				var brokenPipe bool
				if ne, ok := err.(*net.OpError); ok {
					if se, ok := ne.Err.(*os.SyscallError); ok {
						if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {
							brokenPipe = true
						}
					}
				}

				httpRequest, _ := httputil.DumpRequest(c.Request, false)
				if brokenPipe {
					lg.Error(c.Request.URL.Path,
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
					// If the connection is dead, we can't write a status to it.
					c.Error(err.(error)) // nolint: errcheck
					c.Abort()
					return
				}

				if stack {
					lg.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
						zap.String("stack", string(debug.Stack())),
					)
				} else {
					lg.Error("[Recovery from panic]",
						zap.Any("error", err),
						zap.String("request", string(httpRequest)),
					)
				}
				c.AbortWithStatus(http.StatusInternalServerError)
			}
		}()
		c.Next()
	}
}
```

然后定义日志相关配置：

```go
type LogConfig struct {
	Level string `json:"level"`
	Filename string `json:"filename"`
	MaxSize int `json:"maxsize"`
	MaxAge int `json:"max_age"`
	MaxBackups int `json:"max_backups"`
}
```

在项目中先从配置文件加载配置信息，再调用`logger.InitLogger(config.Conf.LogConfig)`即可完成logger实例的初识化。其中，通过`r.Use(logger.GinLogger(), logger.GinRecovery(true))`注册我们的中间件来使用zap接收gin框架自身的日志，在项目中需要的地方通过使用`zap.L().Xxx()`方法来记录自定义日志信息。

```go
func main() {
	// load config from config.json
	if len(os.Args) < 1 {
		return
	}

	if err := config.Init(os.Args[1]); err != nil {
		panic(err)
	}
	// init logger
	if err := logger.InitLogger(config.Conf.LogConfig); err != nil {
		fmt.Printf("init logger failed, err:%v\n", err)
		return
	}

	gin.SetMode(config.Conf.Mode)

	r := gin.Default()
	r.Use(logger.GinLogger(), logger.GinRecovery(true)) // 注册zap相关中间件
	r.GET("/hello", func(c *gin.Context) {
		var (
			name = "q1mi"	// 假设你有一些数据需要记录到日志中
			age  = 18
		)
		// 记录日志并使用zap.Xxx(key, val)记录相关字段
		zap.L().Debug("this is hello func", zap.String("user", name), zap.Int("age", age))

		c.String(http.StatusOK, "hello liwenzhou.com!")
	})

	addr := fmt.Sprintf(":%v", config.Conf.Port)
	r.Run(addr)
}
```

完整示例代码我已经上传至github：[gin_zap_demo](https://github.com/Q1mi/gin_zap_demo)



# logrus日志库使用

Logrus是Go（golang）的结构化logger，与标准库logger完全API兼容。

它有以下特点：

- 完全兼容标准日志库，拥有七种日志级别：`Trace`, `Debug`, `Info`, `Warning`, `Error`, `Fatal`and `Panic`。
- 可扩展的Hook机制，允许使用者通过Hook的方式将日志分发到任意地方，如本地文件系统，logstash，elasticsearch或者mq等，或者通过Hook定义日志内容和格式等
- 可选的日志输出格式，内置了两种日志格式JSONFormater和TextFormatter，还可以自定义日志格式
- Field机制，通过Filed机制进行结构化的日志记录
- 线程安全

```sh
go get github.com/sirupsen/logrus
```

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

## level日志级别

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

## log.Fields字段

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

## WithFields默认字段

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

## Hooks钩子

Logrus 最令人心动的两个功能，一个是结构化日志，另一个就是 Hooks 了。

Hooks 为 Logrus 提供了极大的灵活性，通过 Hooks 可以实现各种扩展功能。比如可以通过 Hooks 实现：`Error` 以上级别日志发送邮件通知、重要日志告警、日志切割、程序优雅退出等，非常实用。

Logrus 提供了 `Hook` 接口，只要我们实现了这个接口，并将其注册到 Logrus 中，就可以使用 Hooks 的强大能力了。`Hook` 接口定义如下：

```go
type Hook interface {
	Levels() []Level
	Fire(*Entry) error
}
```

>  Levels返回一个日志级别切片，Logrus 记录的日志级别如果存在于切片中，则会触发 Hooks（即调用 Fire 方法）

### 使用示例

```go
type CustomHook struct{}

func (h *CustomHook) Levels() []logrus.Level {
	return logrus.AllLevels
}

func (h *CustomHook) Fire(entry *logrus.Entry) error {
	entry.Data["custom_field"] = "custom_value" // 在日志中添加额外的字段
	if entry.Level >= logrus.ErrorLevel {       // 根据日志级别执行特定操作
		fmt.Println("这是一个错误日志，执行一些特殊操作...")
	}
	return nil
}

func Test_LogrusHook(t *testing.T) {
	Testlog := logrus.New()

	hook := &CustomHook{}	// 添加自定义 Hook
	Testlog.AddHook(hook)

	Testlog.Info("这是一条信息日志")
	Testlog.Error("这是一条错误日志")
}
```

```go
// 输出：
time="2024-10-19T10:31:24+08:00" level=info msg="这是一条信      息日志" custom_field=custom_value
这是一个错误日志，执行一些特殊操作...
time="2024-10-19T10:31:24+08:00" level=error msg="这是一条错     误日志" custom_field=custom_value
--- PASS: Test_LogrusHook (0.01s)
PASS
```

### 通过hook写入db中

通过hook钩子写入如mysql中

```go
type LogEntry struct {
	ID      uint   `gorm:"column:id;primaryKey;autoIncrement" comment:"自增id" json:"id"`
	Key     string `gorm:"column:key;type:varchar(255)" comment:"日志记录匹配 索引关键值" json:"key"`
	Level   string `gorm:"column:level;type:varchar(255)" comment:"日志level字段" json:"level"`
	Message string `gorm:"column:message;type:text" comment:"日志内容" json:"message"`
}

func (LogEntry) TableName() string {
	return "logToDB"
}

func Test_logrusHookDB(*testing.T) {
	// 连接数据库
	db, err := gorm.Open(mysql.Open("root:123456@tcp(192.168.5.5:3306)/test?charset=utf8mb4&parseTime=True&loc=Local"), &gorm.Config{})
	if err != nil {
		logrus.Fatalf("无法连接数据库：%v", err)
	}

	// 创建日志钩子，将日志同时写入数据库
	logrus.AddHook(&DatabaseHook{db: db})
	logrus.WithField("key", "example_key").Info("这是一条信息日志")
	logrus.WithField("key", "another_key").Warn("这是一条警告日志")
}

type DatabaseHook struct {
	db *gorm.DB
}

func (d *DatabaseHook) Levels() []logrus.Level {
	return logrus.AllLevels
}

func (d *DatabaseHook) Fire(entry *logrus.Entry) error {
	fmt.Printf("%+v\n", entry.Data["key"])
	logEntry := LogEntry{
		Key:     entry.Data["key"].(string),
		Level:   entry.Level.String(),
		Message: entry.Message,
	}
	if !d.db.Migrator().HasTable("logToDB") {
		err := d.db.AutoMigrate(&logEntry)
		if err != nil {
			return err
		}
	}

	err := d.db.Create(&logEntry).Error
	if err != nil {
		return err
	}
	return nil
}
```



### 官方使用

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

```go
logrus.TextFormatter logrus.JSONFormatter
```

还支持一些第三方的格式化程序，详见项目首页。

## 记录函数名

如果你希望将调用的函数名添加为字段，请通过以下方式设置：

```go
log.SetReportCaller(true)
```

这会将调用者添加为”method”，如下所示：

```json
{"animal":"penguin","level":"fatal","method":"github.com/sirupsen/arcticcreatures.migrate","msg":"a penguin swims by","time":"2014-03-10 19:57:38.562543129 -0400 EDT"}
```

**注意：**开启这个模式会增加性能开销。

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



# 日志切割

`lumberjack`是一个 Go 语言的日志轮转库，主要用于管理日志文件的大小、备份数量和保留时间等。

```go
"gopkg.in/natefinch/lumberjack.v2"
```



```go

```

# 实时读取log文件内容

在做日志分析的时候，需要实时的获取日志里面的内容找到了`hpcloud/tail`

```go
package main

import (
    "fmt"
    "time"

    "github.com/hpcloud/tail"
)

func main() {
    fileName := "./my.log"
    tailconfig := tail.Config{
        ReOpen:    true,                                 // 重新打开
        Follow:    true,                                 // 是否跟随
        Location:  &tail.SeekInfo{Offset: 0, Whence: 2}, // 从文件的哪个地方开始读
        MustExist: false,                                // 文件不存在不报错
        Poll:      true,
    }
    tails, err := tail.TailFile(fileName, tailconfig)
    if err != nil {
        fmt.Println("tail file failed, err:", err)
        return
    }

    var (
        line *tail.Line
        ok   bool
    )
    for {
        line, ok = <-tails.Lines
        if !ok {
            fmt.Printf("tail file close reopen, filename:%s\n", tails.Filename)
            time.Sleep(time.Second)
            continue
        }
        fmt.Println("line:", line.Text)
    }
}
```

在同级目录下面定义一个my.log文件，在文件里面写入文字敲下回车，并且保存之后，程序会自动的获取并且打印，可以根据业务需要就行修改



# log

Go语言内置的`log`包实现了简单的日志服务。本文介绍了标准库`log`的基本使用。

## 使用Logger

log包定义了Logger类型，该类型提供了一些格式化输出的方法。本包也提供了一个预定义的“标准”logger，可以通过调用函数`Print系列`(Print|Printf|Println）、`Fatal系列`（Fatal|Fatalf|Fatalln）、和`Panic系列`（Panic|Panicf|Panicln）来使用，比自行创建一个logger对象更容易使用。

例如，我们可以像下面的代码一样直接通过`log`包来调用上面提到的方法，默认它们会将日志信息打印到终端界面：

```go
func main() {
	log.Println("这是一条很普通的日志。")
	v := "很普通的"
	log.Printf("这是一条%s日志。\n", v)
	log.Fatalln("这是一条会触发fatal的日志。")
	log.Panicln("这是一条会触发panic的日志。")
}
```

编译并执行上面的代码会得到如下输出：

```bash
2017/06/19 14:04:17 这是一条很普通的日志。
2017/06/19 14:04:17 这是一条很普通的日志。
2017/06/19 14:04:17 这是一条会触发fatal的日志。
```

logger会打印每条日志信息的日期、时间，默认输出到系统的标准错误。Fatal系列函数会在写入日志信息后调用os.Exit(1)。Panic系列函数会在写入日志信息后panic。

## 配置logger

### 标准logger的配置

默认情况下的logger只会提供日志的时间信息，但是很多情况下我们希望得到更多信息，比如记录该日志的文件名和行号等。`log`标准库中为我们提供了定制这些设置的方法。

`log`标准库中的`Flags`函数会返回标准logger的输出配置，而`SetFlags`函数用来设置标准logger的输出配置。

```go
func Flags() int
func SetFlags(flag int)
```

### flag选项

`log`标准库提供了如下的flag选项，它们是一系列定义好的常量。

```go
const (
    // 控制输出日志信息的细节，不能控制输出的顺序和格式。
    // 输出的日志在每一项后会有一个冒号分隔：例如2009/01/23 01:23:23.123123 /a/b/c/d.go:23: message
    Ldate         = 1 << iota     // 日期：2009/01/23
    Ltime                         // 时间：01:23:23
    Lmicroseconds                 // 微秒级别的时间：01:23:23.123123（用于增强Ltime位）
    Llongfile                     // 文件全路径名+行号： /a/b/c/d.go:23
    Lshortfile                    // 文件名+行号：d.go:23（会覆盖掉Llongfile）
    LUTC                          // 使用UTC时间
    LstdFlags     = Ldate | Ltime // 标准logger的初始值
)
```

下面我们在记录日志之前先设置一下标准logger的输出选项如下：

```go
func main() {
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
	log.Println("这是一条很普通的日志。")
}
```

编译执行后得到的输出结果如下：

```go
2017/06/19 14:05:17.494943 .../log_demo/main.go:11: 这是一条很普通的日志。
```

### 配置日志前缀

`log`标准库中还提供了关于日志信息前缀的两个方法：

```go
func Prefix() string
func SetPrefix(prefix string)
```

其中`Prefix`函数用来查看标准logger的输出前缀，`SetPrefix`函数用来设置输出前缀。

```go
func main() {
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
	log.Println("这是一条很普通的日志。")
	log.SetPrefix("[小王子]")
	log.Println("这是一条很普通的日志。")
}
```

上面的代码输出如下：

```bash
[小王子]2017/06/19 14:05:57.940542 .../log_demo/main.go:13: 这是一条很普通的日志。
```

这样我们就能够在代码中为我们的日志信息添加指定的前缀，方便之后对日志信息进行检索和处理。

### 配置日志输出位置

```go
func SetOutput(w io.Writer)
```

`SetOutput`函数用来设置标准logger的输出目的地，默认是标准错误输出。

例如，下面的代码会把日志输出到同目录下的`xx.log`文件中。

```go
func main() {
	logFile, err := os.OpenFile("./xx.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
	if err != nil {
		fmt.Println("open log file failed, err:", err)
		return
	}
	log.SetOutput(logFile)
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
	log.Println("这是一条很普通的日志。")
	log.SetPrefix("[小王子]")
	log.Println("这是一条很普通的日志。")
}
```

如果你要使用标准的logger，我们通常会把上面的配置操作写到`init`函数中。

```go
func init() {
	logFile, err := os.OpenFile("./xx.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
	if err != nil {
		fmt.Println("open log file failed, err:", err)
		return
	}
	log.SetOutput(logFile)
	log.SetFlags(log.Llongfile | log.Lmicroseconds | log.Ldate)
}
```

## 创建logger

`log`标准库中还提供了一个创建新logger对象的构造函数–`New`，支持我们创建自己的logger示例。`New`函数的签名如下：

```go
func New(out io.Writer, prefix string, flag int) *Logger
```

New创建一个Logger对象。其中，参数out设置日志信息写入的目的地。参数prefix会添加到生成的每一条日志前面。参数flag定义日志的属性（时间、文件等等）。

举个例子：

```go
func main() {
	logger := log.New(os.Stdout, "<New>", log.Lshortfile|log.Ldate|log.Ltime)
	logger.Println("这是自定义的logger记录的日志。")
}
```

将上面的代码编译执行之后，得到结果如下：

```bash
<New>2017/06/19 14:06:51 main.go:34: 这是自定义的logger记录的日志。
```

## 总结

Go内置的log库功能有限，例如无法满足记录不同级别日志的情况，我们在实际的项目中根据自己的需要选择使用第三方的日志库，如[logrus](https://github.com/sirupsen/logrus)、[zap](https://github.com/uber-go/zap)等。