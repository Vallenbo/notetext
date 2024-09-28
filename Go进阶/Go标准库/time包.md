# 时间函数

时间和日期是我们编程中经常会用到的，本文主要介绍了 Go 语言内置的 time 包的基本用法。time 包提供了一些关于时间显示和测量用的函数。time 包中日历的计算采用的是公历，不考虑润秒。

# 时间类型

Go 语言中使用`time.Time`类型表示时间。我们可以通过`time.Now`函数获取当前的时间对象，然后从时间对象中可以获取到年、月、日、时、分、秒等信息。

```go
func timeDemo() {  // timeDemo 时间对象的年月日时分秒
	now := time.Now() // 获取当前时间
	fmt.Printf("current time:%v\n", now)

	year := now.Year()     // 年
	month := now.Month()   // 月
	day := now.Day()       // 日
	hour := now.Hour()     // 小时
	minute := now.Minute() // 分钟
	second := now.Second() // 秒
	fmt.Println(year, month, day, hour, minute, second)
}
```

# Location和time zone

Go 语言中使用 location 来映射具体的时区。时区（Time Zone）是根据世界各国家与地区不同的经度而划分的时间定义，全球共分为24个时区。中国差不多跨5个时区，但为了使用方便只用东八时区的标准时即北京时间为准。

下面的示例代码中使用`beijing`来表示东八区8小时的偏移量，其中`time.FixedZone`和`time.LoadLocation`这两个函数则是用来获取location信息。

```go
func timezoneDemo() { // timezoneDemo 时区示例
	// 中国没有夏令时，使用一个固定的8小时的UTC时差。
	secondsEastOfUTC := int((8 * time.Hour).Seconds()) // 对于很多其他国家需要考虑夏令时。
	// FixedZone 返回始终使用给定区域名称和偏移量(UTC 以东秒)的 Location。
	beijing := time.FixedZone("Beijing Time", secondsEastOfUTC)

	// 如果当前系统有时区数据库，则可以加载一个位置得到对应的时区
	// 例如，加载纽约所在的时区
	newYork, err := time.LoadLocation("America/New_York") // UTC-05:00
	if err != nil {
		fmt.Println("load America/New_York location failed", err)
		return
	}
	fmt.Println()
    
	//shanghai, err := time.LoadLocation("Asia/Shanghai") //UTC+08:00，加载上海所在的时区
	//tokyo, err := time.LoadLocation("Asia/Tokyo") // UTC+09:00 // 加载东京所在的时区

	// 创建时间对象需要指定位置。常用的位置是 time.Local（当地时间） 和 time.UTC（UTC时间）。
	//timeInLocal := time.Date(2009, 1, 1, 20, 0, 0, 0, time.Local)  // 系统本地时间
	timeInUTC := time.Date(2009, 1, 1, 12, 0, 0, 0, time.UTC)
	sameTimeInBeijing := time.Date(2009, 1, 1, 20, 0, 0, 0, beijing)
	sameTimeInNewYork := time.Date(2009, 1, 1, 7, 0, 0, 0, newYork)

	// 北京时间（东八区）比UTC早8小时，所以上面两个时间看似差了8小时，但表示的是同一个时间
	timesAreEqual := timeInUTC.Equal(sameTimeInBeijing)
	fmt.Println(timesAreEqual)

	// 纽约（西五区）比UTC晚5小时，所以上面两个时间看似差了5小时，但表示的是同一个时间
	timesAreEqual = timeInUTC.Equal(sameTimeInNewYork)
	fmt.Println(timesAreEqual)
}
```

在日常编码过程中使用时间对象的时候一定要注意其时区信息，由于`time.LoadLocation`依赖系统的时区数据库，在不太确定程序运行环境的情况下建议先定义时区偏移量然后使用`time.FixedZone`的方式指定时区。

# Unix Time

Unix Time是自1970年1月1日 00:00:00 UTC 至当前时间经过的总秒数。下面的代码片段演示了如何基于时间对象获取到Unix 时间。

```go
func timestampDemo() { 		// timestampDemo 时间戳
	now := time.Now()        // 获取当前时间
	timestamp := now.Unix()  // 秒级时间戳
	milli := now.UnixMilli() // 毫秒时间戳 Go1.17+
	micro := now.UnixMicro() // 微秒时间戳 Go1.17+
	nano := now.UnixNano()   // 纳秒时间戳
	fmt.Println(timestamp, milli, micro, nano)
}
```

time 包还提供了一系列将 int64 类型的时间戳转换为时间对象的方法。

```go
func timestamp2Time() { // timestamp2Time 将时间戳转为时间对象
	secondsEastOfUTC := int((8 * time.Hour).Seconds()) // 获取北京时间所在的东八区时区对象
	beijing := time.FixedZone("Beijing Time", secondsEastOfUTC)

	// 北京时间 2022-02-22 22:22:22.000000022 +0800 CST
	t := time.Date(2022, 02, 22, 22, 22, 22, 22, beijing)

	var (
		sec  = t.Unix()
		msec = t.UnixMilli()
		usec = t.UnixMicro()
	)
	timeObj := time.Unix(sec, 22)  // 将秒级时间戳转为时间对象（第二个参数为不足1秒的纳秒数）
	fmt.Println(timeObj)           // 2022-02-22 22:22:22.000000022 +0800 CST
	timeObj = time.UnixMilli(msec) // 毫秒级时间戳转为时间对象
	fmt.Println(timeObj)           // 2022-02-22 22:22:22 +0800 CST
	timeObj = time.UnixMicro(usec) // 微秒级时间戳转为时间对象
	fmt.Println(timeObj)           // 2022-02-22 22:22:22 +0800 CST
}
```

# 时间间隔

`time.Duration`是`time`包定义的一个类型，它代表两个时间点之间经过的时间，以纳秒为单位。`time.Duration`表示一段时间间隔，可表示的最长时间段大约290年。

time 包中定义的时间间隔类型的常量如下：

```go
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

例如：`time.Duration`表示1纳秒，`time.Second`表示1秒。

# 时间运算操作

## Add

Go语言的时间对象有提供Add方法如下：

```go
func (t Time) Add(d Duration) Time
```

举个例子，求一个小时之后的时间：

```go
func main() {
	now := time.Now()
	later := now.Add(time.Hour) // 当前时间加1小时后的时间
	fmt.Println(later)
}
```

## Sub

求两个时间之间的差值：

```go
func (t Time) Sub(u Time) Duration
```

返回一个时间段t-u。如果结果超出了Duration可以表示的最大值/最小值，将返回最大值/最小值。要获取时间点t-d（d为Duration），可以使用t.Add(-d)。

## Equal

```go
func (t Time) Equal(u Time) bool
```

判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。本方法和用t==u不同，这种方法还会比较地点和时区信息。

## Before

```go
func (t Time) Before(u Time) bool
```

如果t代表的时间点在u之前，返回真；否则返回假。

## After

```go
func (t Time) After(u Time) bool
```

如果t代表的时间点在u之后，返回真；否则返回假。

# 定时器

使用`time.Tick(时间间隔)`来设置定时器，定时器的本质上是一个通道（channel）。

```go
func tickDemo() {
	ticker := time.Tick(time.Second) //定义一个1秒间隔的定时器
	for i := range ticker {
		fmt.Println(i)//每秒都会执行的任务
	}
}
```

定时执行  

```Go
tm := time.NewTimer(3 * time.Second)
<-tm.C //阻塞3秒钟
//do something
tm.Stop()

//或者用：
<-time.After(3 * time.Second) //阻塞3秒钟
```

周期执行  

```Go
tk := time.NewTicker(1 * time.Second)
for i := 0; i < 10; i++ {
    <-tk.C //阻塞1秒钟
    //do something
}
tk.Stop()
```



# 时间格式化

`time.Format`函数能够将一个时间对象格式化输出为指定布局的文本表示形式，需要注意的是 Go 语言中时间格式化的布局不是常见的`Y-m-d H:M:S`，而是使用 `2006-01-02 15:04:05.000`（记忆口诀为2006 1 2 3 4 5）。

其中：

- 2006：年（Y）
- 01：月（m）
- 02：日（d）
- 15：时（H）
- 04：分（M）
- 05：秒（S）

**补充**

- 如果想格式化为12小时格式，需在格式化布局中添加`PM`。
- 小数部分想保留指定位数就写0，如果想省略末尾可能的0就写 9。

```go
func formatDemo() { // formatDemo 时间格式化
	now := time.Now() // 格式化的模板为 2006-01-02 15:04:05

	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan")) // 24小时制
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan")) // 12小时制

	// 小数点后写0，因为有3个0所以格式化输出的结果也保留3位小数
	fmt.Println(now.Format("2006/01/02 15:04:05.000")) // 2022/02/27 00:10:42.960
	// 小数点后写9，会省略末尾可能出现的0
	fmt.Println(now.Format("2006/01/02 15:04:05.999")) // 2022/02/27 00:10:42.96
	fmt.Println(now.Format("15:04:05")) // 只格式化时分秒部分
	fmt.Println(now.Format("2006.01.02")) // 只格式化日期部分
}
```

# 解析字符串格式的时间

对于从文本的时间表示中解析出时间对象，`time`包中提供了`time.Parse`和`time.ParseInLocation`两个函数。

其中`time.Parse`在解析时不需要额外指定时区信息。

```go
func parseDemo() { // parseDemo 指定时区解析时间
	// 在没有时区指示符的情况下，time.Parse 返回UTC时间
	timeObj, err := time.Parse("2006/01/02 15:04:05", "2022/10/05 11:25:20")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(timeObj) // 2022-10-05 11:25:20 +0000 UTC

	// 在有时区指示符的情况下，time.Parse 返回对应时区的时间表示
    // RFC3339     = "2006-01-02T15:04:05Z07:00"
	timeObj, err = time.Parse(time.RFC3339, "2022-10-05T11:25:20+08:00")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(timeObj) // 2022-10-05 11:25:20 +0800 CST
}
```

`time.ParseInLocation`函数需要在解析时额外指定时区信息。

```go
func parseDemo() { // parseDemo 解析时间
	now := time.Now()
	fmt.Println(now)

	loc, err := time.LoadLocation("Asia/Shanghai") 	// 加载时区
	if err != nil {
		fmt.Println(err)
		return
	}
	// 按照指定时区和指定格式解析字符串时间
	timeObj, err := time.ParseInLocation("2006/01/02 15:04:05", "2022/10/05 11:25:20", loc)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(timeObj)
	fmt.Println(timeObj.Sub(now))
}
```



