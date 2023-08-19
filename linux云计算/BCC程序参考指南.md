# bcc-Python参考指南

用于搜索 （Ctrl-F） 和参考。有关教程，请从 [tutorial.md](https://github.com/iovisor/bcc/blob/master/docs/tutorial.md) 开始。

本指南不完整。如果感觉缺少某些内容，请检查密件抄送和内核源代码。如果您确认我们缺少某些内容，请发送拉取请求来修复它，并帮助每个人。

- - https://github.com/iovisor/bcc/blob/master/docs/reference_guide.md#2-kernel-version-overriding)

# Python

# 初始化

构造 函数。

## 1. BPF

语法：`BPF({text=BPF_program | src_file=filename} [, usdt_contexts=[USDT_object, ...]] [, cflags=[arg1, ...]] [, debug=int])`

创建一个 BPF 对象。这是定义 BPF 程序并与其输出交互的主要对象。

恰好是其中之一或必须提供（不是两者兼而有之）。`text``src_file`

指定要传递给编译器的其他参数，例如 或 。参数作为数组传递，每个元素都是一个附加参数。请注意，字符串不会在空格上拆分，因此每个参数必须是数组的不同元素，例如 .`cflags``-DMACRO_NAME=value``-I/include/path``["-include", "header.h"]`

这些标志控制调试输出，并且可以一起或：`debug`

- `DEBUG_LLVM_IR = 0x1`编译的 LLVM IR
- `DEBUG_BPF = 0x2`在分支上加载 BPF 字节码和寄存器状态
- `DEBUG_PREPROCESSOR = 0x4`预处理器结果
- `DEBUG_SOURCE = 0x8`嵌入源代码的 ASM 指令
- `DEBUG_BPF_REGISTER_STATE = 0x10`除DEBUG_BPF外，还对所有指令注册状态
- `DEBUG_BTF = 0x20`打印库中的消息。`libbpf`

例子：

```
# define entire BPF program in one line:
BPF(text='int do_trace(void *ctx) { bpf_trace_printk("hit!\\n"); return 0; }');

# define program as a variable:
prog = """
int hello(void *ctx) {
    bpf_trace_printk("Hello, World!\\n");
    return 0;
}
"""
b = BPF(text=prog)

# source a file:
b = BPF(src_file = "vfsreadlat.c")

# include a USDT object:
u = USDT(pid=int(pid))
[...]
b = BPF(text=bpf_text, usdt_contexts=[u])

# add include paths:
u = BPF(text=prog, cflags=["-I/path/to/include"])
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=BPF+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=BPF+path%3Atools+language%3Apython&type=Code)

## 2. USTD

语法：`USDT({pid=pid | path=path})`

创建一个对象来检测用户静态定义的跟踪 （USDT） 探测器。它的主要方法是 。`enable_probe()`

参数：

- pid：附加到此进程 ID。
- 路径：从此二进制路径进行 USDT 探测的工具。

例子：

```
# include a USDT object:
u = USDT(pid=int(pid))
[...]
b = BPF(text=bpf_text, usdt_contexts=[u])
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=USDT+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=USDT+path%3Atools+language%3Apython&type=Code)

# 事件

## 1. attach_kprobe（）

语法：`BPF.attach_kprobe(event="event", fn_name="name")`

使用函数条目的内核动态跟踪来检测内核函数，并附加要在调用内核函数时调用的 C 定义的函数。`event()``name()`

例如：

```
b.attach_kprobe(event="sys_clone", fn_name="do_trace")
```



这将检测内核函数，然后每次调用时都会运行我们的 BPF 定义的函数。`sys_clone()``do_trace()`

您可以多次调用 attach_kprobe（），并将 BPF 函数附加到多个内核函数。 您还可以多次调用 attach_kprobe（） 将多个 BPF 函数附加到同一个内核函数。

请参阅前面的 kprobes 部分，了解如何检测来自 BPF 的参数。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_kprobe+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_kprobe+path%3Atools+language%3Apython&type=Code)

## 2. attach_kretprobe（）

语法：`BPF.attach_kretprobe(event="event", fn_name="name" [, maxactive=int])`

使用函数返回的内核动态跟踪来检测内核函数的返回，并附加要在内核函数返回时调用的 C 定义的函数。`event()``name()`

例如：

```
b.attach_kretprobe(event="vfs_read", fn_name="do_return")
```



这将检测内核函数，然后每次调用时都会运行我们的 BPF 定义的函数。`vfs_read()``do_return()`

您可以多次调用 attach_kretprobe（），并将 BPF 函数附加到多个内核函数返回。 您也可以多次调用 attach_kretprobe（） 以将多个 BPF 函数附加到同一个内核函数返回。

当 kretprobe 安装在内核函数上时，它可以捕获的并行调用数量是有限制的。您可以使用 更改该限制。请参阅 kprobes 文档以了解其默认值。`maxactive`

请参阅前面的 kretprobes 部分，了解如何检测 BPF 的返回值。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_kretprobe+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_kretprobe+path%3Atools+language%3Apython&type=Code)

## 3. attach_tracepoint（）

语法：`BPF.attach_tracepoint(tp="tracepoint", fn_name="name")`

检测 描述的内核跟踪点，命中时运行 BPF 函数。`tracepoint``name()`

这是检测跟踪点的显式方法。前面的跟踪点部分中介绍的语法是一种替代方法，其优点是自动声明包含跟踪点参数的结构。使用 ，跟踪点参数需要在 BPF 程序中声明。`TRACEPOINT_PROBE``args``attach_tracepoint()`

例如：

```
# define BPF program
bpf_text = """
#include <uapi/linux/ptrace.h>

struct urandom_read_args {
    // from /sys/kernel/debug/tracing/events/random/urandom_read/format
    u64 __unused__;
    u32 got_bits;
    u32 pool_left;
    u32 input_left;
};

int printarg(struct urandom_read_args *args) {
    bpf_trace_printk("%d\\n", args->got_bits);
    return 0;
};
"""

# load BPF program
b = BPF(text=bpf_text)
b.attach_tracepoint("random:urandom_read", "printarg")
```



请注意，第一个参数现在是我们定义的结构。`printarg()`

原位示例：[代码](https://github.com/iovisor/bcc/blob/a4159da8c4ea8a05a3c6e402451f530d6e5a8b41/examples/tracing/urandomread-explicit.py#L41)、搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_tracepoint+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_tracepoint+path%3Atools+language%3Apython&type=Code)

## 4. attach_uprobe（）

语法：`BPF.attach_uprobe(name="location", sym="symbol", fn_name="name" [, sym_off=int])``BPF.attach_uprobe(name="location", sym_re="regex", fn_name="name")``BPF.attach_uprobe(name="location", addr=int, fn_name="name")`

使用函数条目的用户级动态跟踪从库或二进制文件中检测用户级函数，并附加要在调用用户级函数时调用的 C 定义函数。如果给定，则该函数附加到符号内的偏移量。`symbol()``location``name()``sym_off`

可以提供实际地址来代替 ，在这种情况下，必须将其设置为默认值。如果文件是非 PIE 可执行文件，则必须是虚拟地址，否则必须是相对于文件加载地址的偏移量。`addr``sym``sym``addr`

可以在 中提供正则表达式，而不是符号名称。然后，uprobe 将附加到与提供的正则表达式匹配的符号。`sym_re`

库可以在不带 lib 前缀的 name 参数中给出，也可以使用完整路径 （/usr/lib/...）。二进制文件只能使用完整路径 （/bin/sh） 给出。

例如：

```
b.attach_uprobe(name="c", sym="strlen", fn_name="count")
```



这将检测来自libc的函数，并在调用时调用我们的BPF函数。请注意，“libc”中的“lib”是不需要指定的。`strlen()``count()`

其他例子：

```
b.attach_uprobe(name="c", sym="getaddrinfo", fn_name="do_entry")
b.attach_uprobe(name="/usr/bin/python", sym="main", fn_name="do_main")
```



您可以多次调用 attach_uprobe（），并将 BPF 函数附加到多个用户级函数。

请参阅前面的 uprobes 部分，了解如何检测来自 BPF 的参数。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_uprobe+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_uprobe+path%3Atools+language%3Apython&type=Code)

## 5. attach_uretprobe（）

语法：`BPF.attach_uretprobe(name="location", sym="symbol", fn_name="name")`

使用函数返回的用户级动态跟踪，从库或二进制文件中检测用户级函数的返回，并附加要在用户级函数返回时调用的 C 定义函数。`symbol()``location``name()`

例如：

```
b.attach_uretprobe(name="c", sym="strlen", fn_name="count")
```



这将检测来自libc的函数，并在返回时调用我们的BPF函数。`strlen()``count()`

其他例子：

```
b.attach_uretprobe(name="c", sym="getaddrinfo", fn_name="do_return")
b.attach_uretprobe(name="/usr/bin/python", sym="main", fn_name="do_main")
```



您可以多次调用 attach_uretprobe（），并将 BPF 函数附加到多个用户级函数。

请参阅前面的尿素探针部分，了解如何检测 BPF 的返回值。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_uretprobe+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_uretprobe+path%3Atools+language%3Apython&type=Code)

## 6. USDT.enable_probe（）

语法：`USDT.enable_probe(probe=probe, fn_name=name)`

将 BPF C 函数附加到 USDT 探针。`name``probe`

例：

```
# enable USDT probe from given PID
u = USDT(pid=int(pid))
u.enable_probe(probe="http__server__request", fn_name="do_trace")
```



要检查您的二进制文件是否有 USDT 探针以及它们是什么，您可以运行并检查 stap 调试部分。`readelf -n binary`

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=enable_probe+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=enable_probe+path%3Atools+language%3Apython&type=Code)

## 7. attach_raw_tracepoint（）

语法：`BPF.attach_raw_tracepoint(tp="tracepoint", fn_name="name")`

检测由 （ 仅，否） 描述的内核原始跟踪点，命中时运行 BPF 函数。`tracepoint``event``category``name()`

这是检测跟踪点的显式方法。前面的原始跟踪点部分中介绍的语法是一种替代方法。`RAW_TRACEPOINT_PROBE`

例如：

```
b.attach_raw_tracepoint("sched_switch", "do_trace")
```



原位示例：[搜索/工具](https://github.com/iovisor/bcc/search?q=attach_raw_tracepoint+path%3Atools+language%3Apython&type=Code)

## 8. attach_raw_socket（）

语法：`BPF.attach_raw_socket(fn, dev)`

将 BPF 函数附加到指定的网络接口。

必须是的类型，bpf_prog类型必须是`fn``BPF.function``BPF_PROG_TYPE_SOCKET_FILTER` (`fn=BPF.load_func(func_name, BPF.SOCKET_FILTER)`)

```
fn.sock`是已创建并绑定到 的非阻塞原始套接字。`dev
```

所有由 处理的网络数据包在经过 bpf_prog 处理后都会复制到 的。尝试使用 rev/recvfrom/recvmsg 接收数据包形式。请注意，如果在 已满后未及时读取 ，则复制的数据包将被丢弃。`dev``recv-q``fn.sock``fn.sock``recv-q``recv-q`

我们可以使用此功能来捕获网络数据包，就像.`tcpdump`

我们可以用来观察.`ss --bpf --packet -p``fn.sock`

例：

```
BPF.attach_raw_socket(bpf_func, ifname)
```



原位示例：[搜索/示例](https://github.com/iovisor/bcc/search?q=attach_raw_socket+path%3Aexamples+language%3Apython&type=Code)

## 9. attach_xdp（）

语法：`BPF.attach_xdp(dev="device", fn=b.load_func("fn_name",BPF.XDP), flags)`

检测 描述的网络驱动程序，然后接收数据包，使用标志运行 BPF 函数。`dev``fn_name()`

下面是可选标志的列表。

```
# from xdp_flags uapi/linux/if_link.h
XDP_FLAGS_UPDATE_IF_NOEXIST = (1 << 0)
XDP_FLAGS_SKB_MODE = (1 << 1)
XDP_FLAGS_DRV_MODE = (1 << 2)
XDP_FLAGS_HW_MODE = (1 << 3)
XDP_FLAGS_REPLACE = (1 << 4)
```



你可以使用这样的标志`BPF.attach_xdp(dev="device", fn=b.load_func("fn_name",BPF.XDP), flags=BPF.XDP_FLAGS_UPDATE_IF_NOEXIST)`

标志的默认值为 0。这意味着如果没有带有 的 xdp 程序，则 fn 将与该设备一起运行。如果有xdp程序随设备一起运行，则旧程序将替换为新的fn程序。`device`

目前，密件抄送不支持XDP_FLAGS_REPLACE标志。以下是其他标志的说明。

### 1. XDP_FLAGS_UPDATE_IF_NOEXIST

如果 XDP 程序已附加到指定的驱动程序，则再次附加 XDP 程序将失败。

### 2. XDP_FLAGS_SKB_MODE

驱动程序不支持 XDP，但内核伪造了它。 XDP 程序可以工作，但没有真正的性能优势，因为数据包无论如何都会传递到内核堆栈，然后模拟 XDP – 这通常由家用计算机、笔记本电脑和虚拟化硬件中使用的通用网络驱动程序支持。

### 3. XDP_FLAGS_DRV_MODE

驱动程序具有 XDP 支持，无需内核堆栈交互即可移交给 XDP – 很少有驱动程序可以支持它，这些驱动程序通常用于企业硬件。

### 4. XDP_FLAGS_HW_MODE

XDP 可以直接在网卡上加载和执行 – 只有少数网卡可以做到这一点。

例如：

```
b.attach_xdp(dev="ens1", fn=b.load_func("do_xdp", BPF.XDP))
```



这将检测网络设备，然后每次接收数据包时都会运行我们的 BPF 定义的函数。`ens1``do_xdp()`

别忘了在最后打电话！`b.remove_xdp("ens1")`

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=attach_xdp+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=attach_xdp+path%3Atools+language%3Apython&type=Code)

## 10. attach_func（）

语法：`BPF.attach_func(fn, attachable_fd, attach_type [, flags])`

将指定类型的 BPF 函数附加到特定的 .如果是 ，则该函数应附加到当前 net 命名空间，并且必须为 0。`attachable_fd``attach_type``BPF_FLOW_DISSECTOR``attachable_fd`

例如：

```
b.attach_func(fn, cgroup_fd, BPFAttachType.CGROUP_SOCK_OPS)
b.attach_func(fn, map_fd, BPFAttachType.SK_MSG_VERDICT)
```



小说当连接到“全局”钩子（xdp，tc，lwt，cgroup）时。如果程序终止后不再需要“BPF 函数”，请确保在程序退出时调用。`detach_func`

原位示例：

[搜索/示例](https://github.com/iovisor/bcc/search?q=attach_func+path%3Aexamples+language%3Apython&type=Code)，

## 11. detach_func（）

语法：`BPF.detach_func(fn, attachable_fd, attach_type)`

分离指定类型的 BPF 函数。

例如：

```
b.detach_func(fn, cgroup_fd, BPFAttachType.CGROUP_SOCK_OPS)
b.detach_func(fn, map_fd, BPFAttachType.SK_MSG_VERDICT)
```



原位示例：

[搜索/示例](https://github.com/iovisor/bcc/search?q=detach_func+path%3Aexamples+language%3Apython&type=Code)，

## 12. detach_kprobe（）

语法：`BPF.detach_kprobe(event="event", fn_name="name")`

分离指定事件的 kprobe 处理程序函数。

例如：

```
b.detach_kprobe(event="__page_cache_alloc", fn_name="trace_func_entry")
```



## 13. detach_kretprobe（）

语法：`BPF.detach_kretprobe(event="event", fn_name="name")`

分离指定事件的 kretprobe 处理程序函数。

例如：

```
b.detach_kretprobe(event="__page_cache_alloc", fn_name="trace_func_return")
```



# 调试输出

## 1. trace_print（）

语法：`BPF.trace_print(fmt="fields")`

此方法持续读取全局共享的 /sys/kernel/debug/tracing/trace_pipe 文件并打印其内容。可以通过 BPF 和 bpf_trace_printk（） 函数写入此文件，但是，该方法具有局限性，包括缺乏并发跟踪支持。首选前面介绍的BPF_PERF_OUTPUT机制。

参数：

- `fmt`：可选，可以包含字段格式字符串。默认为 .`None`

例子：

```
# print trace_pipe output as-is:
b.trace_print()

# print PID and message:
b.trace_print(fmt="{1} {5}")
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=trace_print+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=trace_print+path%3Atools+language%3Apython&type=Code)

## 2. trace_fields（）

语法：`BPF.trace_fields(nonblocking=False)`

此方法从全局共享的 /sys/kernel/debug/tracing/trace_pipe 文件中读取一行，并将其作为字段返回。可以通过 BPF 和 bpf_trace_printk（） 函数写入此文件，但是，该方法具有局限性，包括缺乏并发跟踪支持。首选前面介绍的BPF_PERF_OUTPUT机制。

参数：

- `nonblocking`：可选，默认为 。设置为 时，程序不会阻止等待输入。`False``True`

例子：

```
while 1:
    try:
        (task, pid, cpu, flags, ts, msg) = b.trace_fields()
    except ValueError:
        continue
    [...]
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=trace_fields+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=trace_fields+path%3Atools+language%3Apython&type=Code)

# 输出接口

BPF 程序的正常输出为：

- 每个事件：使用 PERF_EVENT_OUTPUT、open_perf_buffer（） 和 perf_buffer_poll（）。
- 地图摘要：使用地图部分中介绍的 items（） 或 print_log2_hist（）。

## 1. perf_buffer_poll（）

语法：`BPF.perf_buffer_poll(timeout=T)`

这会从所有打开的性能环形缓冲区轮询，调用为每个条目调用open_perf_buffer时提供的回调函数。

超时参数是可选的，以毫秒为单位。在没有投票的情况下，投票将无限期地继续进行。

例：

```
# loop with callback to print_event
b["events"].open_perf_buffer(print_event)
while 1:
    try:
        b.perf_buffer_poll()
    except KeyboardInterrupt:
        exit();
```



原位示例：[代码](https://github.com/iovisor/bcc/blob/v0.9.0/examples/tracing/hello_perf_output.py#L55)、搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=perf_buffer_poll+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=perf_buffer_poll+path%3Atools+language%3Apython&type=Code)

## 2. ring_buffer_poll（）

语法：`BPF.ring_buffer_poll(timeout=T)`

这会从所有打开的 ringbuf 环形缓冲区轮询，调用为每个条目调用 open_ring_buffer 时提供的回调函数。

超时参数是可选的，以毫秒为单位。在没有投票的情况下，投票将持续到 没有更多数据或回调返回负值。

例：

```
# loop with callback to print_event
b["events"].open_ring_buffer(print_event)
while 1:
    try:
        b.ring_buffer_poll(30)
    except KeyboardInterrupt:
        exit();
```



原位示例：[搜索/示例](https://github.com/iovisor/bcc/search?q=ring_buffer_poll+path%3Aexamples+language%3Apython&type=Code)，

## 3. ring_buffer_consume（）

语法：`BPF.ring_buffer_consume()`

这会消耗所有打开的 ringbuf 环形缓冲区，调用为每个条目调用 open_ring_buffer 时提供的回调函数。

与 不同，此方法在尝试使用之前**不会轮询数据**。 这减少了延迟，但代价是更高的 CPU 消耗。如果您不确定要使用哪个， 用。`ring_buffer_poll``ring_buffer_poll`

例：

```
# loop with callback to print_event
b["events"].open_ring_buffer(print_event)
while 1:
    try:
        b.ring_buffer_consume()
    except KeyboardInterrupt:
        exit();
```



原位示例：[搜索/示例](https://github.com/iovisor/bcc/search?q=ring_buffer_consume+path%3Aexamples+language%3Apython&type=Code)，

# 地图接口

映射是 BPF 数据存储，在 bcc 中用于实现表，然后在表顶部实现更高级别的对象，包括哈希和直方图。

## 1. get_table（）

语法：`BPF.get_table(name)`

返回表对象。这不再使用，因为现在可以将表作为 BPF 中的项目读取。例如：。`BPF[name]`

例子：

```
counts = b.get_table("counts")

counts = b["counts"]
```



这些是等效的。

## 2. open_perf_buffer（）

语法：`table.open_perf_buffers(callback, page_cnt=N, lost_cb=None)`

这将对 BPF 中定义为 BPF_PERF_OUTPUT（） 的表进行操作，并将回调 Python 函数关联在 perf 环形缓冲区中可用时调用。这是将每个事件的数据从内核传输到用户空间的推荐机制的一部分。可以通过参数指定 perf 环形缓冲区的大小，该参数必须是两个页数的幂，默认值为 8。如果回调处理数据的速度不够快，则提交的部分数据可能会丢失。 将被调用以记录/监控丢失的计数。如果为默认值，则只会将一行消息打印到 .`callback``page_cnt``lost_cb``lost_cb``None``stderr`

例：

```
# process event
def print_event(cpu, data, size):
    event = ct.cast(data, ct.POINTER(Data)).contents
    [...]

# loop with callback to print_event
b["events"].open_perf_buffer(print_event)
while 1:
    try:
        b.perf_buffer_poll()
    except KeyboardInterrupt:
        exit()
```



请注意，传输的数据结构需要在 BPF 程序中用 C 声明。例如：

```
// define output data structure in C
struct data_t {
    u32 pid;
    u64 ts;
    char comm[TASK_COMM_LEN];
};
BPF_PERF_OUTPUT(events);
[...]
```



在 Python 中，你可以让 bcc 从 C 声明自动生成数据结构（推荐）：

```
def print_event(cpu, data, size):
    event = b["events"].event(data)
[...]
```



或手动定义：

```
# define output data structure in Python
TASK_COMM_LEN = 16    # linux/sched.h
class Data(ct.Structure):
    _fields_ = [("pid", ct.c_ulonglong),
                ("ts", ct.c_ulonglong),
                ("comm", ct.c_char * TASK_COMM_LEN)]

def print_event(cpu, data, size):
    event = ct.cast(data, ct.POINTER(Data)).contents
[...]
```



原位示例：[代码](https://github.com/iovisor/bcc/blob/v0.9.0/examples/tracing/hello_perf_output.py#L52)、搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=open_perf_buffer+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=open_perf_buffer+path%3Atools+language%3Apython&type=Code)

## 3. items()

语法：`table.items()`

返回表中键的数组。这可以与BPF_HASH映射一起使用，以获取和迭代键。

例：

```
# print output
print("%10s %s" % ("COUNT", "STRING"))
counts = b.get_table("counts")
for k, v in sorted(counts.items(), key=lambda counts: counts[1].value):
    print("%10d \"%s\"" % (v.value, k.c.encode('string-escape')))
```



此示例还使用该方法按值排序。`sorted()`

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=items+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=items+path%3Atools+language%3Apython&type=Code)

## 4. values()

语法：`table.values()`

返回表中值的数组。

## 5. clear()

语法：`table.clear()`

清除表：删除所有条目。

例：

```
# print map summary every second:
while True:
    time.sleep(1)
    print("%-8s\n" % time.strftime("%H:%M:%S"), end="")
    dist.print_log2_hist(sym + " return:")
    dist.clear()
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=clear+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=clear+path%3Atools+language%3Apython&type=Code)

## 6. items_lookup_and_delete_batch（）

语法：`table.items_lookup_and_delete_batch()`

返回表中键的数组，其中对 BPF 系统调用进行一次调用。这可以与BPF_HASH映射一起使用，以获取和迭代键。它还会清除表：删除所有条目。 你应该使用table.items_lookup_and_delete_batch（）而不是table.items（），后跟table.clear（）。它需要内核 v5.6。

例：

```
# print call rate per second:
print("%9s-%9s-%8s-%9s" % ("PID", "COMM", "fname", "counter"))
while True:
    for k, v in sorted(b['map'].items_lookup_and_delete_batch(), key=lambda kv: (kv[0]).pid):
        print("%9s-%9s-%8s-%9d" % (k.pid, k.comm, k.fname, v.counter))
    sleep(1)
```



## 7. items_lookup_batch（）

语法：`table.items_lookup_batch()`

返回表中键的数组，其中对 BPF 系统调用进行一次调用。这可以与BPF_HASH映射一起使用，以获取和迭代键。 你应该使用table.items_lookup_batch（）而不是table.items（）。它需要内核 v5.6。

例：

```
# print current value of map:
print("%9s-%9s-%8s-%9s" % ("PID", "COMM", "fname", "counter"))
while True:
    for k, v in sorted(b['map'].items_lookup_batch(), key=lambda kv: (kv[0]).pid):
        print("%9s-%9s-%8s-%9d" % (k.pid, k.comm, k.fname, v.counter))
```



## 8. items_delete_batch（）

语法：`table.items_delete_batch(keys)`

当键为 None 时，它会清除 BPF_HASH 映射的所有条目。它比 table.clear（） 更有效，因为它只生成一个系统调用。您可以通过提供键数组作为参数来删除映射的子集。这些键及其关联的值将被删除。它需要内核 v5.6。

参数：

- 键是可选的，默认情况下为 None。

## 9. items_update_batch（）

语法：`table.items_update_batch(keys, values)`

使用新值更新所有提供的键。两个参数的长度必须相同，并且在映射限制范围内（介于 1 和最大条目数之间）。它需要内核 v5.6。

参数：

- 键是要更新的键的列表
- 值是包含新值的列表。

## 10. print_log2_hist（）

语法：`table.print_log2_hist(val_type="value", section_header="Bucket ptr", section_print_fn=None)`

将表打印为 ASCII 格式的 log2 直方图。该表必须存储为 log2，这可以使用 BPF 函数来完成。`bpf_log2l()`

参数：

- val_type：可选，列标题。
- section_header：如果直方图具有辅助键，则将打印多个表，section_header可以用作每个表的标题描述。
- section_print_fn：如果section_print_fn不是 None，则会传递存储桶值。

例：

```
b = BPF(text="""
BPF_HISTOGRAM(dist);

int kprobe__blk_account_io_done(struct pt_regs *ctx, struct request *req)
{
	dist.increment(bpf_log2l(req->__data_len / 1024));
	return 0;
}
""")
[...]

b["dist"].print_log2_hist("kbytes")
```



输出：

```
     kbytes          : count     distribution
       0 -> 1        : 3        |                                      |
       2 -> 3        : 0        |                                      |
       4 -> 7        : 211      |**********                            |
       8 -> 15       : 0        |                                      |
      16 -> 31       : 0        |                                      |
      32 -> 63       : 0        |                                      |
      64 -> 127      : 1        |                                      |
     128 -> 255      : 800      |**************************************|
```



此输出显示多模态分布，最大模式为 128->255 KB，计数为 800。

这是汇总数据的有效方法，因为汇总是在内核中执行的，并且只有计数列传递到用户空间。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=print_log2_hist+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=print_log2_hist+path%3Atools+language%3Apython&type=Code)

## 11. print_linear_hist（）

语法：`table.print_linear_hist(val_type="value", section_header="Bucket ptr", section_print_fn=None)`

将表打印为 ASCII 格式的线性直方图。这旨在可视化小整数范围，例如，0 到 100。

参数：

- val_type：可选，列标题。
- section_header：如果直方图具有辅助键，则将打印多个表，section_header可以用作每个表的标题描述。
- section_print_fn：如果section_print_fn不是 None，则会传递存储桶值。

例：

```
b = BPF(text="""
BPF_HISTOGRAM(dist);

int kprobe__blk_account_io_done(struct pt_regs *ctx, struct request *req)
{
	dist.increment(req->__data_len / 1024);
	return 0;
}
""")
[...]

b["dist"].print_linear_hist("kbytes")
```



输出：

```
     kbytes        : count     distribution
        0          : 3        |******                                  |
        1          : 0        |                                        |
        2          : 0        |                                        |
        3          : 0        |                                        |
        4          : 19       |****************************************|
        5          : 0        |                                        |
        6          : 0        |                                        |
        7          : 0        |                                        |
        8          : 4        |********                                |
        9          : 0        |                                        |
        10         : 0        |                                        |
        11         : 0        |                                        |
        12         : 0        |                                        |
        13         : 0        |                                        |
        14         : 0        |                                        |
        15         : 0        |                                        |
        16         : 2        |****                                    |
[...]
```



这是汇总数据的有效方法，因为汇总是在内核中执行的，并且只有计数列中的值才会传递到用户空间。

原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=print_linear_hist+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=print_linear_hist+path%3Atools+language%3Apython&type=Code)

## 12. open_ring_buffer（）

语法：`table.open_ring_buffer(callback, ctx=None)`

这将对 BPF 中定义为 BPF_RINGBUF_OUTPUT（） 的表进行操作，并关联在 ringbuf 环形缓冲区中有数据可用时要调用的回调 Python 函数。这是新的 （Linux 5.8+） 推荐机制的一部分，用于将每个事件的数据从内核传输到用户空间。与 perf 缓冲区不同，ringbuf 大小在 BPF 程序中指定，作为宏的一部分。如果回调处理数据的速度不够快，则提交的部分数据可能会丢失。在这种情况下，应更频繁地轮询事件和/或增加环形缓冲区的大小。`callback``BPF_RINGBUF_OUTPUT`

例：

```
# process event
def print_event(ctx, data, size):
    event = ct.cast(data, ct.POINTER(Data)).contents
    [...]

# loop with callback to print_event
b["events"].open_ring_buffer(print_event)
while 1:
    try:
        b.ring_buffer_poll()
    except KeyboardInterrupt:
        exit()
```



请注意，传输的数据结构需要在 BPF 程序中用 C 声明。例如：

```
// define output data structure in C
struct data_t {
    u32 pid;
    u64 ts;
    char comm[TASK_COMM_LEN];
};
BPF_RINGBUF_OUTPUT(events, 8);
[...]
```



在 Python 中，你可以让 bcc 从 C 声明自动生成数据结构（推荐）：

```
def print_event(ctx, data, size):
    event = b["events"].event(data)
[...]
```



或手动定义：

```
# define output data structure in Python
TASK_COMM_LEN = 16    # linux/sched.h
class Data(ct.Structure):
    _fields_ = [("pid", ct.c_ulonglong),
                ("ts", ct.c_ulonglong),
                ("comm", ct.c_char * TASK_COMM_LEN)]

def print_event(ctx, data, size):
    event = ct.cast(data, ct.POINTER(Data)).contents
[...]
```



原位示例：[搜索/示例](https://github.com/iovisor/bcc/search?q=open_ring_buffer+path%3Aexamples+language%3Apython&type=Code)，

## 13. push()

语法：`table.push(leaf, flags=0)`

将元素推送到堆栈表或队列表上。如果操作不成功，则引发异常。 将QueueStack.BPF_EXIST作为标志传递会导致队列或堆栈丢弃最旧的元素（如果该元素已满）。

原位示例：[搜索/测试](https://github.com/iovisor/bcc/search?q=push+path%3Atests+language%3Apython&type=Code)，

## 14. pop()

语法：`leaf = table.pop()`

从堆栈或队列表中弹出元素。与 不同，在返回元素之前将其从表中删除。 如果操作不成功，则引发 KeyError 异常。`peek()``pop()`

原位示例：[搜索/测试](https://github.com/iovisor/bcc/search?q=pop+path%3Atests+language%3Apython&type=Code)，

## 15. eek()

语法：`leaf = table.peek()`

查看堆栈或队列表头部的元素。与 不同，不会从表中删除元素。如果操作不成功，则引发异常。`pop()``peek()`

原位示例：[搜索/测试](https://github.com/iovisor/bcc/search?q=peek+path%3Atests+language%3Apython&type=Code)，

# 助手

密件抄送提供的一些帮助程序方法。请注意，由于我们在 Python 中，我们可以导入任何 Python 库及其方法，包括例如库：argparse、collections、ctypes、datetime、re、socket、struct、subprocess、sys 和 time。

## 1. ksym（）

语法：`BPF.ksym(addr)`

将内核内存地址转换为返回的内核函数名称。

例：

```
print("kernel function: " + b.ksym(addr))
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=ksym+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=ksym+path%3Atools+language%3Apython&type=Code)

## 2. ksymname（）

语法：`BPF.ksymname(name)`

将内核名称转换为地址。这与 ksym 相反。当函数名称未知时返回 -1。

例：

```
print("kernel address: %x" % b.ksymname("vfs_read"))
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=ksymname+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=ksymname+path%3Atools+language%3Apython&type=Code)

## 3. 符号（）

语法：`BPF.sym(addr, pid, show_module=False, show_offset=False)`

将内存地址转换为 pid 的函数名称，返回该名称。小于零的 pid 将访问内核符号缓存。和 参数控制是否应显示符号所在的模块，以及是否应显示从符号开头开始的指令偏移量。这些额外参数默认为 .`show_module``show_offset``False`

例：

```
print("function: " + b.sym(addr, pid))
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=sym+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=sym+path%3Atools+language%3Apython&type=Code)

## 4. num_open_kprobes（）

语法：`BPF.num_open_kprobes()`

返回打开的 k[ret]探测器的数量。可用于在连接和分离探测器时使用event_re的方案。不包括perf_events读者。

例：

```
b.attach_kprobe(event_re=pattern, fn_name="trace_count")
matched = b.num_open_kprobes()
if matched == 0:
    print("0 functions matched by \"%s\". Exiting." % args.pattern)
    exit()
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=num_open_kprobes+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=num_open_kprobes+path%3Atools+language%3Apython&type=Code)

## 5. get_syscall_fnname（）

语法：`BPF.get_syscall_fnname(name : str)`

返回系统调用的相应内核函数名称。此帮助程序函数将尝试不同的前缀，并使用正确的前缀与系统调用名称连接。请注意，返回值在不同版本的 linux 内核中可能会有所不同，有时会导致麻烦。（见[#2590](https://github.com/iovisor/bcc/issues/2590))

例：

```
print("The function name of %s in kernel is %s" % ("clone", b.get_syscall_fnname("clone")))
# sys_clone or __x64_sys_clone or ...
```



原位示例：搜索/[示例、搜索/](https://github.com/iovisor/bcc/search?q=get_syscall_fnname+path%3Aexamples+language%3Apython&type=Code)[工具](https://github.com/iovisor/bcc/search?q=get_syscall_fnname+path%3Atools+language%3Apython&type=Code)

# BPF 错误

请参阅内核源代码中文档/网络/过滤器.txt下的“了解 eBPF 验证程序消息”部分。

# 1. 无效的内存访问

这可能是由于尝试直接读取内存，而不是在 BPF 堆栈上的内存上运行。所有内核内存读取都必须通过 bpf_probe_read_kernel（） 传递，以将内核内存复制到 BPF 堆栈中，在某些简单取消引用的情况下，bcc 重写器可以自动执行此操作。bpf_probe_read_kernel（） 执行所有必需的检查。

例：

```
bpf: Permission denied
0: (bf) r6 = r1
1: (79) r7 = *(u64 *)(r6 +80)
2: (85) call 14
3: (bf) r8 = r0
[...]
23: (69) r1 = *(u16 *)(r7 +16)
R7 invalid mem access 'inv'

Traceback (most recent call last):
  File "./tcpaccept", line 179, in <module>
    b = BPF(text=bpf_text)
  File "/usr/lib/python2.7/dist-packages/bcc/__init__.py", line 172, in __init__
    self._trace_autoload()
  File "/usr/lib/python2.7/dist-packages/bcc/__init__.py", line 612, in _trace_autoload
    fn = self.load_func(func_name, BPF.KPROBE)
  File "/usr/lib/python2.7/dist-packages/bcc/__init__.py", line 212, in load_func
    raise Exception("Failed to load BPF program %s" % func_name)
Exception: Failed to load BPF program kretprobe__inet_csk_accept
```



# 2. 不能从专有程序调用仅 GPL 函数

当从非 GPL BPF 程序调用仅限 GPL 的帮助程序时，会发生此错误。要修复此错误，请不要使用专有 BPF 程序中的仅限 GPL 的帮助程序，或者根据与 GPL 兼容的许可证重新许可 BPF 程序。检查哪些 [BPF 助手](https://github.com/iovisor/bcc/blob/master/docs/kernel-versions.md#helpers)是仅限 GPL 的，以及哪些许可证被认为是与 GPL 兼容的。

示例调用，一个仅限 GPL 的 BPF 助手，来自专有程序 （）：`bpf_get_stackid()``#define BPF_LICENSE Proprietary`

```sh
bpf: Failed to load program: Invalid argument
[...]
8: (85) call bpf_get_stackid#27
cannot call GPL only function from proprietary program
```

# 环境变量

# 1. 内核源目录

eBPF 程序编译需要内核源代码或带有标头的内核标头 编译。如果您的内核源位于密件抄送的非标准位置 找不到，则可以向密件抄送提供该位置的绝对路径 通过设置它。`BCC_KERNEL_SOURCE`

# 2. 内核版本覆盖

默认情况下，密件抄送将 存储在生成的 eBPF 对象中 然后在加载 eBPF 程序时传递给内核。 有时这很不方便，尤其是当内核稍微 更新了诸如 LTS 内核版本。它极不可能轻微 不匹配会导致加载的 eBPF 程序出现任何问题。通过设置为正在运行的内核版本，检查 用于验证内核版本可以绕过。这是程序所必需的 使用kprobes。这需要按以下格式编码：。例如，如果正在运行的内核是 ， 然后可以设置为覆盖内核 版本检查成功。`LINUX_VERSION_CODE``BCC_LINUX_VERSION_CODE``(VERSION * 65536) + (PATCHLEVEL * 256) + SUBLEVEL``4.9.10``export BCC_LINUX_VERSION_CODE=264458`