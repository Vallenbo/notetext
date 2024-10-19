# 性能调优pprof

​	在计算机性能调试领域里，profiling 是指对应用程序的画像，画像就是应用程序使用 CPU 和内存的情况。 Go语言是一个对性能特别看重的语言，因此语言中自带了 profiling 的库，这篇文章就要讲解怎么在 golang 中做 profiling。

**性能分析的5个方面:**

`I/O`：IO使用情况
`CPU profile`：报告程序的 CPU 使用情况，按照一定频率去采集应用程序在 CPU 和寄存器上面的数据
`Memory Profile（Heap Profile）`：报告程序的内存使用情况
`Block Profiling`：报告 goroutines 不在运行状态的情况，可以用来分析和查找死锁等性能瓶颈
`Goroutine Profiling`：报告 goroutines 的使用情况，有哪些 goroutine，它们的调用关系是怎样的
`deadlock`：死锁检测，数据竞争分析



web采集数据
通过基准测试采集数据
硬编码采集数

## 采集性能数据

Go语言内置了获取程序的运行数据的工具，包括以下两个标准库：
`runtime/pprof`：采集工具型应用运行数据进行分析
`net/http/pprof`：采集服务型应用运行时数据进行分析

pprof开启后，每隔一段时间（10ms）就会收集下当前的堆栈信息，获取各个函数占用的CPU以及内存资源；最后通过对这些采样数据进行分析，形成一个性能分析报告。
注意，我们只应该在性能测试的时候才在代码中引入pprof。

## 工具型应用

如果你的应用程序是运行一段时间就结束退出类型。那么最好的办法是在应用退出的时候把 profiling 的报告保存到文件中，进行分析。对于这种情况，可以使用runtime/pprof库。 首先在代码中导入runtime/pprof工具：

**CPU性能分析**
开启CPU性能分析：`pprof.StartCPUProfile(w io.Writer)`
停止CPU性能分析：`pprof.StopCPUProfile()`
应用执行结束后，就会生成一个文件，保存了我们的 CPU profiling 数据。得到采样数据之后，使用`go tool pprof`工具进行CPU性能分析。

**内存性能优化**
记录程序的堆栈信息`pprof.WriteHeapProfile(w io.Writer)`
得到采样数据之后，使用`go tool pprof`工具进行内存性能分析。
`go tool pprof`默认是使用`-inuse_space`进行统计，还可以使用`-inuse-objects`查看分配对象的数量。

## 服务型应用

如果你的应用程序是一直运行的，比如 web 应用，那么可以使用net/http/pprof库，它能够在提供 HTTP 服务进行分析。
如果使用了默认的`http.DefaultServeMux`（通常是代码直接使用 `http.ListenAndServe(“0.0.0.0:8000”, nil)`），只需要在你的web server端代码中按如下方式导入net/http/pprof

```go
import _ "net/http/pprof"
```

如果你使用自定义的 Mux，则需要手动注册一些路由规则：

```go
r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```

如果你使用的是gin框架，那么推荐使用`github.com/gin-contrib/pprof`，在代码中通过以下命令注册pprof相关路由。
`pprof.Register(router)`
不管哪种方式，你的 HTTP 服务都会多出`/debug/pprof`，访问它会得到类似下面的内容：
<img src="assets/Pasted image 20230424094737.png" alt="Pasted image 20230424094737" style="zoom:67%;" />

这个路径下还有几个子页面：

> /debug/pprof/profile：访问这个链接会自动进行 CPU profiling，持续 30s，并生成一个文件供下载
> /debug/pprof/heap： Memory Profiling 的路径，访问这个链接会得到一个内存 Profiling 结果的文件
> /debug/pprof/block：block Profiling 的路径
> /debug/pprof/goroutines：运行的 goroutines 列表，以及调用关系

## go tool pprof命令

不管是工具型应用还是服务型应用，我们使用相应的pprof库获取数据之后，下一步的都要对这些数据进行分析，我们可以使用go tool pprof命令行工具。
`go tool pprof`最简单的使用方式为:`go tool pprof [binary] [source]`

> binary 是应用的二进制文件，用来解析各种符号；
> source 表示 profile 数据的来源，可以是本地的文件，也可以是 http 地址。

注意事项： 获取的 Profiling 数据是动态的，要想获得有效的数据，请保证应用处于较大的负载（比如正在生成中运行的服务，或者通过其他工具模拟访问压力）。否则如果应用处于空闲状态，得到的结果可能没有任何意义。

通过flag我们可以在命令行控制是否开启CPU和Mem的性能分析。 将上面的代码保存并编译成runtime_pprof可执行文件，执行时加上-cpu命令行参数如下：

```sh
./runtime_pprof -cpu
```

等待30秒后会在当前目录下生成一个cpu.pprof文件

未完。。。

