# cron （定时）

`robfig/cron`是一个用于 Go 语言的 cron 库。

```go
go get github.com/robfig/cron/v3@v3.0.0
```

它是 Go 语言中较为流行的定时任务处理库。提供了一种方便的方式来在特定的时间间隔或时间点执行任务。

**支持功能**

- 类似于 Unix cron 表达式的语法
  - 可以使用熟悉的 cron 表达式格式来定义任务的执行时间规则。例如，`*/5 * * * * *`表示每 5 秒执行一次任务。
  - 能够设置复杂的时间计划，包括每分钟、每小时、每天、每周、每月等不同时间粒度的任务调度。
- 灵活的任务添加
  - 可以轻松地向 cron 调度器中添加多个任务。通过`AddFunc`方法，可以将一个函数与一个 cron 表达式相关联，当满足时间条件时，该函数就会被执行。
- 启动和停止调度
  - 提供了`Start`方法来启动定时任务调度，开始按照设定的时间规则执行任务。
  - 可以在需要的时候停止任务调度，以便对任务进行调整或结束程序。

# 快速使用

```golang
import (
    "fmt"
    "github.com/robfig/cron/v3"
)

func main() {
    // 创建一个默认的cron对象
    c := cron.New()

    // 添加任务
    c.AddFunc("30 * * * *", func() { fmt.Println("Every hour on the half hour") })
    c.AddFunc("30 3-6,20-23 * * *", func() { fmt.Println(".. in the range 3-6am, 8-11pm") })
    c.AddFunc("@hourly",      func() { fmt.Println("Every hour, starting an hour from now") })
    c.AddFunc("@every 1h30m", func() { fmt.Println("Every hour thirty, starting an hour thirty from now") })
    
    //开始执行任务
    c.Start()
    
    select {}	//阻塞
    c.Stop // 停止
}
```

# 预定义的 Schedules

| 条目                   | 描述                                       | 等效于    |
| :--------------------- | :----------------------------------------- | :-------- |
| @yearly (or @annually) | Run once a year, midnight, Jan. 1st        | 0 0 1 1 * |
| @monthly               | Run once a month, midnight, first of month | 0 0 1 * * |
| @weekly                | Run once a week, midnight between Sat/Sun  | 0 0 * * 0 |
| @daily (or @midnight)  | Run once a day, midnight                   | 0 0 * * * |
| @hourly                | Run once an hour, beginning of hour        | 0 * * * * |

# 间隔执行

固定间隔执行

```bash
@every <duration>
```

举例

> @every 1h30m10s 表示一个小时30分钟10秒后执行，并且之后的每个时间间隔都执行。需要注意的是间隔是以运行时间点递增的，例如一个任务执行需要耗时 3 分钟，任务每五分钟执行一次，则任务运行完后，距离下次执行为 2 分钟。

另外，@every 支持的字符串由 `time.ParseDuration()`（http://golang.org/pkg/time/#ParseDuration） 解析，只要是其支持的格式都可解析。

# 精确到秒的 Cron 表达式

Cron v3 版本的表达式从六个参数调整为五个，取消了对秒的默认支持，需要精确到秒的控制可以使用 `cron.WithSeconds()` 解析器。

```golang
c := cron.New(cron.WithSeconds())
c.AddFunc("*/1 * * * * *", func() {
    fmt.Println("Every 1 Second")
})
c.Start()
```

如果你仅仅需要每隔 N 秒运行一次 Job，可以使用 `@every` 这样的特殊 spec 表达式。

```golang
c := cron.New()
c.AddFunc("@every 10s", func() {
    fmt.Println("Every 10 Seconds")
})
c.Start()
```

# 常用 Cron 标准表达式

| 描述                                 | 表达式           |
| :----------------------------------- | :--------------- |
| 每1分钟执行一次                      | * * * * *        |
| 每15分钟执行一次                     | */15 * * * *     |
| 每一小时执行                         | * */1 * * *      |
| 每两个小时执行                       | 0 */2 * * *      |
| 每小时的第3和第15分钟执行            | 3,15 * * * *     |
| 在上午8点到11点的第3和第15分钟执行   | 3,15 8-11 * * *  |
| 每晚的21:30执行                      | 30 21 * * *      |
| 每天18:00至23:00之间每隔30分钟执行   | 0,30 18-23 * * * |
| 每星期六的晚上11:00pm执行            | 0 23 * * 6       |
| 晚上11点到早上7点之间,每隔一小时执行 | * 23-7/1 * * *   |
| 指定每天的5:30执行                   | 30 5 * * *       |
| 每小时[第一分钟]执行                 | 01 * * * *       |
| 每天[凌晨4:02]执行                   | 02 4 * * *       |

摘自：https://tooltt.com/crontab-parse/

# 时区设置

```golang
// 本地时区早六点
cron.New().AddFunc("0 6 * * ?", ...)

// 上海时区早六点
nyc, _ := time.LoadLocation("Asia/Shanghai")
c := cron.New(cron.WithLocation(nyc))
c.AddFunc("0 6 * * ?", ...)

// 上海时区早六点
cron.New().AddFunc("CRON_TZ=Asia/Shanghai 0 6 * * ?", ...)

// 重庆时区早六点
c := cron.New(cron.WithLocation(nyc))
c.SetLocation("Asia/Tokyo")
c.AddFunc("CRON_TZ=Asia/Chongqing 0 6 * * ?", ...)
```

# Cron 方法

- `AddFunc()` - 支持传入如 `@every` 这样的 spec 表达式和标准表达式及 `func()` 函数，它封装了 `AddJob()`。
- `AddJob()` - 支持传入 spec 和标准表达式及 Job，Job 是一个接口，实现 Run() 方法的类型即是一个 Job，它封装了 `Schedule()`。
- `Schedule()` - 执行标准表达式和 Job，被 `AddJob()` 函数调用，只支持标准表达式（即 */5 \* * ** ）。
- `Entries()` - 获取全部 Entry，一个 Entry 即为一个定时任务条目。
- `Entry()` - 根据 EntryID 获取指定条目。
- `Remove()` - 根据 EntryID 移除指定条目。
- `Location()` - 获取 Local 时区。
- `Start()` - 使用新的 goroutine 启动，不会阻塞当前协程，已运行的调度器重复调用会被忽略。
- `Run()` - 在当前 goroutine 启动，会阻塞当前协程，已运行的调度器重复调用会被忽略。
- `Stop()` - 停止调度器，返回 Context，可根据 Context 来等待任务执行完成。

**Entry 结构体**

```golang
type Entry struct {
    // ID is the cron-assigned ID of this entry, which may be used to look up a
    // snapshot or remove it.
    ID EntryID

    // Schedule on which this job should be run.
    Schedule Schedule

    // Next time the job will run, or the zero time if Cron has not been
    // started or this entry's schedule is unsatisfiable
    Next time.Time

    // Prev is the last time this job was run, or the zero time if never.
    Prev time.Time

    // WrappedJob is the thing to run when the Schedule is activated.
    WrappedJob Job

    // Job is the thing that was submitted to cron.
    // It is kept around so that user code that needs to get at the job later,
    // e.g. via Entries() can do so.
    Job Job
}
```

可以获取任务上次、下次运行时间。

```golang
c = cron.New()
entryId, _ := c.AddFunc("@every 15m", func() {
    fmt.Println("Every 15 Minutes")
})
entry := c.Entry(entryId)
fmt.Println(entry.Next)
```

# Job Wrappers 包装器

JobWrapper 可以对 Job 进行修饰，添加一些行为。

- `func SkipIfStillRunning(logger Logger) JobWrapper` 如果上次任务还正在运行，那么跳过本次任务的运行并记录日记
- `func DelayIfStillRunning(logger Logger) JobWrapper` 如果上次任务还正在运行，那么延迟执行本次任务的运行并记录日记
- `func Recover(logger Logger) JobWrapper` 如果 Job Panic，记录日记

以上函数中 Skip 及 Recover 会记录 Info 级别的日志，而 Delay 会在延迟超过一分钟后记录 Info 日志。

**使用方法**

```golang
// 作用于调度器
cron.New(cron.WithChain(
    cron.SkipIfStillRunning(logger),
))

// 作用于单个 Job
job = cron.NewChain(
    cron.SkipIfStillRunning(logger),
).Then(job)
```

# Logger 日志记录

```golang
type Logger interface {
    // Info logs routine messages about cron's operation.
    Info(msg string, keysAndValues ...interface{})
    // Error logs an error condition.
    Error(err error, msg string, keysAndValues ...interface{})
}
```

Logger 是一个接口，实现了这些接口的日志包都可被用于 Job 日志的记录工具。

> var DefaultLogger Logger = PrintfLogger(log.New(os.Stdout, "cron: ", log.LstdFlags)) var DiscardLogger Logger = PrintfLogger(log.New(ioutil.Discard, "", 0))

如果未指定，DefaultLogger 是 Cron 的默认输出 Logger，也可以使用 DiscardLogger，不输出日志。

**不输出 Job 日志**

```golang
c := cron.New(cron.WithLogger(cron.DiscardLogger))
```

**Cron 使用 logrus 记录日志示例**

```golang
import (
    "github.com/robfig/cron/v3"
    log "github.com/sirupsen/logrus"
)

type LogrusLog struct {
    logger log.Logger
}

func (l *LogrusLog) Info(msg string, keysAndValues ...interface{}) {
    l.logger.WithFields(log.Fields{
        "data": keysAndValues,
    }).Info(msg)

}

func (l *LogrusLog) Error(err error, msg string, keysAndValues ...interface{}) {
    l.logger.WithFields(log.Fields{
        "msg":  msg,
        "data": keysAndValues,
    }).Error(msg)
}

func main() {
    logrusLog := &LogrusLog{}
    c := cron.New(cron.WithLogger(logrusLog))
}
```

# Option 可选项

`cron.New()` 方法支持传入 Option 可选项。

在上文提及过 `cron.WithSeconds()`、`cron.WithChain()` 和 `cron.WithParser()`，Cron 支持的 Option 函数如下

- `func WithChain(wrappers ...JobWrapper) Option` 指定要应用于此 cron 的所有 Job 的 Job Wrapper。
- `func WithLocation(loc *time.Location) Option` 覆盖 cron 实例的时区。
- `func WithLogger(logger Logger) Option` 使用自定义的日志记录器
- `func WithParser(p ScheduleParser) Option` 重写用于解释 Job 计划的解析器。
- `func WithSeconds() Option` 重写用于解释作业计划的解析器，以将秒字段作为第一个字段。

# 退出 Stop() 方法等待任务执行完成示例

```golang
import (
    "fmt"
    "github.com/robfig/cron/v3"
    "time"
)

func main() {
    c := cron.New(cron.WithSeconds())
    c.AddFunc("@every 10s", func() {
        fmt.Println("goroutine task run")

        time.Sleep(2 * time.Second)
        fmt.Println("-> task out")

        time.Sleep(2 * time.Second)
        fmt.Println("-> task out2")
    })
    c.Start()

    fmt.Println("main sleep 12 seconds")
    fmt.Println("after 10 seconds, goroutine task will run")
    time.Sleep(12 * time.Second)

    for {
        ctx := c.Stop()
        select {
        case <-ctx.Done():
            fmt.Println("all task done, stopped")
            return
        default:
            time.Sleep(time.Second)
            fmt.Println("default, wait")
        }
    }
}
```

输出

```bash
➜ go run main.go
main sleep 12 seconds
after 10 seconds, goroutine task will run
goroutine task run
-> task out
default, wait
-> task out2
default, wait
default, wait
all task done, stopped
```

简单解释下这个示例，它模拟了一个调度器退出的场景，当退出调度器的时候有任务正在执行，那么会等到任务执行完成后再退出。