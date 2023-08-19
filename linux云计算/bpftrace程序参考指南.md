# bpftrace参考指南

有关参考摘要，请参阅[README.md中](https://github.com/iovisor/bpftrace/blob/master/README.md)有关以下部分的内容 [探测类型](https://github.com/iovisor/bpftrace/blob/master/README.md#probe-types)以及本指南中的[探测](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#probes)、[变量内置](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#1-builtins)和[函数内置](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#1-builtins-1)部分。

这是一项正在进行的工作。如果丢失了什么，请检查bpftrace源代码，看看这些文档是否是 只是过时了。如果你发现了什么，请提交一个问题或拉取请求来更新这些文档。 此外，请尽可能简洁地保持这些文档的简洁性（受到6页awk的启发 摘要来自[v7vol2b.pdf](https://9p.io/7thEdMan/bswv7.html)第106页）。留下更长的例子和 /docs中的其他文件、/tools/*_examples.txt文件或博客文章和其他文章。

# 术语表

| 术语       | 项目名称                                                     |
| ---------- | ------------------------------------------------------------ |
| BPF        | Berkeley数据包过滤器：最初为优化包过滤器（例如tcpdump表达式）的处理而开发的一种内核技术 |
| eBPF       | 增强BPF：一种内核技术，它扩展了BPF，以便它可以在任何事件上执行更通用的程序，例如下面列出的bpftrace程序。它使用BPF沙盒虚拟机环境。还要注意，eBPF通常仅被称为BPF。 |
| 探针       | 软件或硬件中的一种检测点，它生成可以执行bpftrace程序的事件。 |
| 静态跟踪   | 在代码中硬编码检测点。由于这些是固定的，它们可以作为稳定API的一部分提供，并记录。 |
| 动态追踪   | 也称为动态插装，这是一种可以通过实时修改指令文本来插装任何软件事件（例如函数调用和返回）的技术。目标软件通常不需要特殊的功能来支持动态跟踪，除了bpftrace可以读取的符号表。由于这是所有软件文本的工具，因此它不被认为是稳定的API，并且目标函数可能不会在其源代码之外进行记录。 |
| 跟踪点     | 一种用于提供静态跟踪的Linux内核技术。                        |
| Kprobes    | 一种Linux内核技术，用于提供内核函数的动态跟踪。              |
| 上探针     | 一种Linux内核技术，用于提供用户级函数的动态跟踪。            |
| 超声波时差 | 用户静态定义的跟踪：用户级软件的静态跟踪点。一些应用程序支持USDT。 |
| BPF映射    | BPF内存对象，bpftrace使用它来创建许多高级对象。              |
| BTF        | BPF类型格式：编码BPF程序/映射相关调试信息的元数据格式。      |

# 使用情况

命令行用法由bpftrace总结，不带选项：

```sh
# bpftrace
USAGE:
    bpftrace [options] filename
    bpftrace [options] -e 'program'

OPTIONS:
    -B MODE        output buffering mode ('line', 'full', or 'none')
    -d             debug info dry run
    -dd            verbose debug info dry run
    -e 'program'   execute this program
    -h             show this help message
    -I DIR         add the specified DIR to the search path for include files.
    --include FILE adds an implicit #include which is read before the source file is preprocessed.
    -l [search]    list probes
    -p PID         enable USDT probes on PID
    -c 'CMD'       run CMD and enable USDT probes on resulting process
    -q             keep messages quiet
    -v             verbose messages
    -k             emit a warning when a bpf helper returns an error (except read functions)
    -kk            check all bpf helper functions
    --version      bpftrace version

ENVIRONMENT:
    BPFTRACE_STRLEN             [default: 64] bytes on BPF stack per str()
    BPFTRACE_NO_CPP_DEMANGLE    [default: 0] disable C++ symbol demangling
    BPFTRACE_MAP_KEYS_MAX       [default: 4096] max keys in a map
    BPFTRACE_MAX_PROBES         [default: 512] max number of probes bpftrace can attach to
    BPFTRACE_MAX_BPF_PROGS      [default: 512] max number of generated BPF programs
    BPFTRACE_CACHE_USER_SYMBOLS [default: auto] enable user symbol cache
    BPFTRACE_VMLINUX            [default: none] vmlinux path used for kernel symbol resolution
    BPFTRACE_BTF                [default: none] BTF file
    BPFTRACE_STACK_MODE         [default: bpftrace] Output format for ustack and kstack builtins

EXAMPLES:
bpftrace -l '*sleep*'
    list probes containing "sleep"
bpftrace -e 'kprobe:do_nanosleep { printf("PID %d sleeping...\n", pid); }'
    trace processes calling sleep
bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @[comm] = count(); }'
    count syscalls by process name
```



## 1. Hello World

bpftrace程序的最基本示例：

```sh
# bpftrace -e 'BEGIN { printf("Hello, World!\n"); }'
Attaching 1 probe...
Hello, World!
^C
```

这个程序的语法将在[语言](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#language)部分解释。在本节中，我们将 覆盖工具的使用。

一个程序将继续运行，直到Ctrl-C被命中，或者一个`exit()`函数被调用。当一个程序 退出时，将打印所有填充的贴图：这种行为和映射将在后面的章节中解释。

## 2. `-e 'program'`：单衬层

`-e`选项允许指定一个程序，并且是构造一行程序的一种方法：

```sh
# bpftrace -e 'tracepoint:syscalls:sys_enter_nanosleep { printf("%s is sleeping.\n", comm); }'
Attaching 1 probe...
iscsid is sleeping.
irqbalance is sleeping.
iscsid is sleeping.
iscsid is sleeping.
[...]
```

此示例是在进程调用nanosleep系统调用时打印。同样，程序的语法将 在[语言](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#language)部分解释。

## 3. `filename`：程序文件

保存为文件的程序通常称为脚本，可以通过指定其文件名来执行。 我们经常使用`.bt`文件扩展名，即bpftrace的缩写，但扩展名会被忽略。

例如，使用`cat -n`列出sleepers.bt文件（它列举了输出行）：

```sh
# cat -n sleepers.bt
1 tracepoint:syscalls:sys_enter_nanosleep
2 {
3   printf("%s is sleeping.\n", comm);
4 }
```

运行sleepers.bt：

```sh
# bpftrace sleepers.bt
Attaching 1 probe...
iscsid is sleeping.
iscsid is sleeping.
[...]
```

也可以使其可执行以独立运行。首先在顶部添加解释器行（`#!`） 使用安装的bpftrace的路径（/usr/local/bin是默认值）或`env`的路径 （通常只是`/usr/bin/env`），后面跟着`bpftrace`（所以它会在`$PATH`中找到bpftrace）：

```sh
1 #!/usr/local/bin/bpftrace
2
3 tracepoint:syscalls:sys_enter_nanosleep
4 {
5   printf("%s is sleeping.\n", comm);
6 }
```

然后使其可执行：

```sh
# chmod 755 sleepers.bt
# ./sleepers.bt
Attaching 1 probe...
iscsid is sleeping.
iscsid is sleeping.
[...]
```



## 4. `-l`：列出探头

来自tracepoint和kprobe文库的探针可以用`-l`列出。

```sh
# bpftrace -l | more
tracepoint:xfs:xfs_attr_list_sf
tracepoint:xfs:xfs_attr_list_sf_all
tracepoint:xfs:xfs_attr_list_leaf
tracepoint:xfs:xfs_attr_list_leaf_end
[...]
# bpftrace -l | wc -l
46260
```

其他库动态地生成探测，比如uprobe，并且需要特定的方法来确定 可用探头。参见后面的[探头](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#probes)章节。

可以添加搜索词：

```sh
# bpftrace -l '*nanosleep*'
tracepoint:syscalls:sys_enter_clock_nanosleep
tracepoint:syscalls:sys_exit_clock_nanosleep
tracepoint:syscalls:sys_enter_nanosleep
tracepoint:syscalls:sys_exit_nanosleep
kprobe:nanosleep_copyout
kprobe:hrtimer_nanosleep
[...]
```

在列出跟踪点时，`-v`选项将显示它们的参数，以便从args内置中使用。用于 示例：

```sh
# bpftrace -lv tracepoint:syscalls:sys_enter_open
tracepoint:syscalls:sys_enter_open
    int __syscall_nr;
    const char * filename;
    int flags;
    umode_t mode;
```

如果BTF可用，也可以列出struct/union/enum定义。举例来说：

```sh
# bpftrace -lv "struct path"
struct path {
        struct vfsmount *mnt;
        struct dentry *dentry;
};
```



## 5. `-d`：调试输出

`-d`选项生成调试输出，但不运行程序。这对于调试非常有用 bpftrace本身的问题。您还可以使用`-dd`生成更加详细的调试输出，这将 还打印未优化的IR。

**如果您是bpftrace的最终用户，通常不需要 `-d` 或者是 `-v` 选项，您可以 跳到 [使用语言](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#language) 节。**

```sh
# bpftrace -d -e 'tracepoint:syscalls:sys_enter_nanosleep { printf("%s is sleeping.\n", comm); }'
Program
 tracepoint:syscalls:sys_enter_nanosleep
  call: printf
   string: %s is sleeping.\n
   builtin: comm
[...]
```

输出以`Program`开始，然后是程序的抽象语法树（AST）表示。

续：

```sh
[...]
%printf_t = type { i64, [16 x i8] }
[...]
define i64 @"tracepoint:syscalls:sys_enter_nanosleep"(i8*) local_unnamed_addr section "s_tracepoint:syscalls:sys_enter_nanosleep" {
entry:
  %comm = alloca [16 x i8], align 1
  %printf_args = alloca %printf_t, align 8
  %1 = bitcast %printf_t* %printf_args to i8*
  call void @llvm.lifetime.start.p0i8(i64 -1, i8* nonnull %1)
  %2 = getelementptr inbounds [16 x i8], [16 x i8]* %comm, i64 0, i64 0
  %3 = bitcast %printf_t* %printf_args to i8*
  call void @llvm.memset.p0i8.i64(i8* nonnull %3, i8 0, i64 24, i32 8, i1 false)
  call void @llvm.lifetime.start.p0i8(i64 -1, i8* nonnull %2)
  call void @llvm.memset.p0i8.i64(i8* nonnull %2, i8 0, i64 16, i32 1, i1 false)
  %get_comm = call i64 inttoptr (i64 16 to i64 (i8*, i64)*)([16 x i8]* nonnull %comm, i64 16)
  %4 = getelementptr inbounds %printf_t, %printf_t* %printf_args, i64 0, i32 1, i64 0
  call void @llvm.memcpy.p0i8.p0i8.i64(i8* nonnull %4, i8* nonnull %2, i64 16, i32 1, i1 false)
  %pseudo = call i64 @llvm.bpf.pseudo(i64 1, i64 1)
  %get_cpu_id = call i64 inttoptr (i64 8 to i64 ()*)()
  %perf_event_output = call i64 inttoptr (i64 25 to i64 (i8*, i8*, i64, i8*, i64)*)(i8* %0, i64 %pseudo, i64 %get_cpu_id, %printf_t* nonnull %printf_args, i64 24)
  call void @llvm.lifetime.end.p0i8(i64 -1, i8* nonnull %1)
  ret i64 0
[...]
```

本节显示llvm中间表示（IR）程序集，然后将其编译成BPF。

## 6. `-v`：详细输出

`-v`选项在程序运行时打印有关程序的更多信息：

```sh
# bpftrace -v -e 'tracepoint:syscalls:sys_enter_nanosleep { printf("%s is sleeping.\n", comm); }'
Attaching 1 probe...

The verifier log:
0: (bf) r6 = r1
1: (b7) r1 = 0
2: (7b) *(u64 *)(r10 -24) = r1
3: (7b) *(u64 *)(r10 -32) = r1
4: (7b) *(u64 *)(r10 -40) = r1
5: (7b) *(u64 *)(r10 -8) = r1
6: (7b) *(u64 *)(r10 -16) = r1
7: (bf) r1 = r10
8: (07) r1 += -16
9: (b7) r2 = 16
10: (85) call bpf_get_current_comm#16
11: (79) r1 = *(u64 *)(r10 -16)
12: (7b) *(u64 *)(r10 -32) = r1
13: (79) r1 = *(u64 *)(r10 -8)
14: (7b) *(u64 *)(r10 -24) = r1
15: (18) r7 = 0xffff9044e65f1000
17: (85) call bpf_get_smp_processor_id#8
18: (bf) r4 = r10
19: (07) r4 += -40
20: (bf) r1 = r6
21: (bf) r2 = r7
22: (bf) r3 = r0
23: (b7) r5 = 24
24: (85) call bpf_perf_event_output#25
25: (b7) r0 = 0
26: (95) exit
processed 26 insns (limit 131072), stack depth 40

Attaching tracepoint:syscalls:sys_enter_nanosleep
iscsid is sleeping.
iscsid is sleeping.
[...]
```

这包括`The verifier log:`，然后是来自内核内验证器的日志消息。

## 7.预处理器选项

`-I`选项可用于将目录添加到bpftrace用于查找的目录列表中标题。可以多次定义。

```sh
# cat program.bt
#include <foo.h>

BEGIN { @ = FOO }

# bpftrace program.bt
definitions.h:1:10: fatal error: 'foo.h' file not found

# /tmp/include
foo.h

# bpftrace -I /tmp/include program.bt
Attaching 1 probe...
```

默认情况下，`--include`选项可用于包含标题。可以多次定义。标题 按定义的顺序包含，并且在程序中包含在任何其他包含之前 被处决

```sh
# bpftrace --include linux/path.h --include linux/dcache.h \
    -e 'kprobe:vfs_open { printf("open path: %s\n", str(((struct path *)arg0)->dentry->d_name.name)); }'
Attaching 1 probe...
open path: .com.google.Chrome.ASsbu2
open path: .com.google.Chrome.gimc10
open path: .com.google.Chrome.R1234s
```

## 8.其他选项

- `--version`选项打印bpftrace版本：

```sh
# bpftrace --version
bpftrace v0.8-90-g585e-dirty
```

- `--no-warnings`选项禁用警告。

## 9.环境变量

### 9.1 `BPFTRACE_STRLEN`

默认值：64

在BPF堆栈上为str（）返回的字符串分配的字节数。

如果你想用str（）读取更大的字符串，请将其放大。

请注意BPF堆栈很小（512字节），并且在printf（）中再次支付费用（同时 它组成了性能事件输出缓冲器）。所以在实践中，你只能把它增加到大约200字节。

对更大字符串的支持[正在讨论中](https://github.com/iovisor/bpftrace/issues/305)。

### 9.2 `BPFTRACE_NO_CPP_DEMANGLE`

默认值：0

默认情况下，在用户空间堆栈跟踪中启用C++符号分离。

可以通过将此环境变量的值设置为`1`来关闭此功能。

### 9.3 `BPFTRACE_MAP_KEYS_MAX`

默认值：4096

这是映射中可以存储的最大密钥数。增加价值会消耗更多 内存和增加启动时间。在某些情况下，您需要：例如采样 堆栈跟踪、记录每页的时间戳等。

### 9.4 `BPFTRACE_MAX_PROBES`

默认值：512

这是bpftrace可以连接到的最大探测数。增加价值会消耗更多 内存，增加启动时间，并可能导致高性能开销，甚至冻结或崩溃 系统。

### 9.5 `BPFTRACE_CACHE_USER_SYMBOLS`

系统启用ASLR，未给出`-c`选项，默认为0;否则1

默认情况下，bpftrace仅在ASLR（地址空间布局） 随机化）被禁用。这是因为符号地址随着ASLR的每次执行而改变。 但是，禁用缓存可能会导致一些性能。将此env变量设置为1以强制bpftrace 缓存如果只跟踪一个程序执行，这是可以的。

### 9.6 `BPFTRACE_VMLINUX`

默认值：无

这指定了在将kprobe附加到offset时用于内核符号解析的vmlinux路径。 如果没有给出此值，bpftrace将从预定义的位置搜索vmlinux。 有关详细信息，请参见src/attached_probe.cpp：find_vmlinux（）。

### 9.7 `BPFTRACE_BTF`

默认值：无

BTF文件的路径。默认情况下，bpftrace搜索多个位置以查找BTF文件。 详细信息请参见src/btf.cpp。

### 9.8 `BPFTRACE_PERF_RB_PAGES`

默认值：64

每个CPU为perf环形缓冲区分配的页数。该值必须是2的幂。

如果您收到了很多丢弃的事件，那么Bpftrace可能没有处理环形缓冲区中的事件 足够快将该值提升得更高可能会很有用，以便可以排队等候更多的事件。权衡 bpftrace将使用更多内存。

### 9.9 `BPFTRACE_MAX_BPF_PROGS`

默认值：512

这是bpftrace可以生成的BPF程序（函数）的最大数量。 此限制的主要目的是防止bpftrace由于生成大量探测而挂起 需要大量的资源（而且不应该经常发生）。

### 9.10 `BPFTRACE_STR_TRUNC_TRAILER`

默认值：`..`

要添加到已截断字符串的尾部。设置为空字符串可禁用截断尾部。

### 9.11 `BPFTRACE_STACK_MODE`

默认值：bpftrace

ustack和kstack内置程序的输出格式。可用模式/格式：`bpftrace`、`perf`、`raw`。 这可以在呼叫站点覆盖。

## 10. Clang环境变量

bpftrace使用libclang（Clang的C接口）解析头文件。环境变量 可以使用影响叮当的工具链。例如，如果从非默认的 目录中，可以设置`CPATH`或`C_INCLUDE_PATH`环境变量以允许clang定位 文件。有关这些环境变量及其用法的更多信息，请参阅clang文档。

# 使用语言

## 1. `{...}`：动作块

语法：`tracepoint:name`

一个bpftrace程序可以有多个操作块。过滤器是可选的。

范例：

```sh
# bpftrace -e 'kprobe:do_sys_open { printf("opening: %s\n", str(arg1)); }'
Attaching 1 probe...
opening: /proc/cpuinfo
opening: /proc/stat
opening: /proc/diskstats
opening: /proc/stat
opening: /proc/vmstat
[...]
```

这是bpftrace的单行调用。探头为`kprobe:do_sys_open`。当那个探测器“开火”时 (the检测事件发生）将执行该操作，该操作由`print()` 声明。下面的章节介绍了探测器和操作。

## 2. `/.../`：过滤

语法：`tracepoint:name`

过滤器（也称为谓词）可以添加到探测器名称之后。探测器还在发射，但会的 跳过该操作，除非过滤器为真。

示例：

```sh
# bpftrace -e 'kprobe:vfs_read /arg2 < 16/ { printf("small read: %d byte buffer\n", arg2); }'
Attaching 1 probe...
small read: 8 byte buffer
small read: 8 byte buffer
small read: 8 byte buffer
small read: 8 byte buffer
small read: 8 byte buffer
small read: 12 byte buffer
^C
```

```sh
# bpftrace -e 'kprobe:vfs_read /comm == "bash"/ { printf("read by %s\n", comm); }'
Attaching 1 probe...
read by bash
read by bash
read by bash
read by bash
^C
```

## 3. `//`, `/*`：注释

语法

```sh
// single-line comment

/*
 * multi-line comment
 */
```

这些可以在bpftrace脚本中使用来记录代码。

## 4.字面意思

支持整数、char和字符串文字。 整数字面值是带有可选下划线（`_`）的数字序列，如 字段分隔符 也支持科学表示法，但仅用于作为BPF的整数值 不支持浮点。

```sh
# bpftrace -e 'BEGIN { printf("%lu %lu %lu", 1000000, 1e6, 1_000_000)}'
Attaching 1 probe...
1000000 1000000 1000000
```

字符文字用单引号括起来，例如。#1和#2

字符串文字用双引号括起来，例如。`"a string"`

## 5. `->`：C结构体

跟踪点示例：

```sh
# bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args.filename)); }'
Attaching 1 probe...
snmpd /proc/diskstats
snmpd /proc/stat
snmpd /proc/vmstat
[...]
```

这是从`filename`结构体返回`args`成员，对于跟踪点探测器，该结构体包含 跟踪点参数。参见[静态跟踪，内核级 此](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#6-tracepoint-static-tracing-kernel-level-arguments)结构的内容的参数节。

kprobe示例：

```sh
# cat path.bt
#include <linux/path.h>
#include <linux/dcache.h>

kprobe:vfs_open
{
	printf("open path: %s\n", str(((struct path *)arg0)->dentry->d_name.name));
}

# bpftrace path.bt
Attaching 1 probe...
open path: dev
open path: if_inet6
open path: retrans_time_ms
[...]
```

这通过简短的脚本路径. bt使用`vfs_open()`内核函数的动态跟踪。一些内核 需要包含头文件来理解`path`和`dentry`结构。

## 6. `struct`：结构声明

范例：

```c
struct nameidata { // from fs/namei.c:
        struct path     path;
        struct qstr     last;
        // [...]
};
```

您可以在需要时定义自己的结构体。在某些情况下，内核结构不会在内核中声明 头包，并在bpftrace工具中手动声明（或部分结构为：足够到达 成员取消引用）。

## 7. `? :`：三元运算符

示例：

```sh
# bpftrace -e 'tracepoint:syscalls:sys_exit_read { @error[args.ret < 0 ? - args.ret : 0] = count(); }'
Attaching 1 probe...
^C

@error[11]: 24
@error[0]: 78
```

```sh
# bpftrace -e 'BEGIN { pid & 1 ? printf("Odd\n") : printf("Even\n"); exit(); }'
Attaching 1 probe...
Odd
```

## 8. `if () {...} else {...}`：if-else语句

范例：

```sh
# bpftrace -e 'tracepoint:syscalls:sys_enter_read { @reads = count();
    if (args.count > 1024) { @large = count(); } }'
Attaching 1 probe...
^C
@large: 72
@reads: 80
```

## 9. `unroll () {...}`：展开

范例：

```sh
# bpftrace -e 'kprobe:do_nanosleep { $i = 1; unroll(5) { printf("i: %d\n", $i); $i = $i + 1; } }'
Attaching 1 probe...
i: 1
i: 2
i: 3
i: 4
i: 5
^C
```

## 10. `++` 和/或 `--`：增量运算符

`++`和`--`可用于方便地增加或减少映射或变量中的计数器。

请注意，map将被隐式声明，如果尚未初始化，则初始化为 声明或定义。临时变量必须在使用这些之前初始化 操作员。

示例-变量：

```
bpftrace -e 'BEGIN { $x = 0; $x++; $x++; printf("x: %d\n", $x); }'
Attaching 1 probe...
x: 2
^C
```

示例-映射：

```
bpftrace -e 'k:vfs_read { @++ }'
Attaching 1 probe...
^C

@: 12807
```

示例-带键的映射：

```
# bpftrace -e 'k:vfs_read { @[probe]++ }'
Attaching 1 probe...
^C

@[kprobe:vfs_read]: 13369
```

## 11. `[]`：数组

您可以使用数组访问操作符`[]`访问一维常量数组。

范例：

```sh
# bpftrace -e 'struct MyStruct { int y[4]; } uprobe:./testprogs/array_access:test_struct {
    $s = (struct MyStruct *) arg0; @x = $s->y[0]; exit(); }'
Attaching 1 probe...

@x: 1
```

## 12.整数转换

整数内部表示为64位有符号。如果你还需要 表示，您可以强制转换为以下内置类型：

| 型号    | 说明           |
| ------- | -------------- |
| `int32` | 无符号8位整数  |
| `int8`  | 带符号8位整数  |
| `int32` | 无符号16位整数 |
| `int32` | 带符号16位整数 |
| `int32` | 无符号32位整数 |
| `int32` | 带符号32位整数 |
| `int32` | 无符号64位整数 |
| `int32` | 带符号64位整数 |

范例：

```sh
# bpftrace -e 'BEGIN { $x = 1<<16; printf("%d %d\n", (uint16)$x, $x); }'
Attaching 1 probe...
0 65536
^C
```

## 13.循环结构

**实验**

内核：5.3

bpftrace支持C样式while循环：

```sh
# bpftrace -e 'i:ms:100 { $i = 0; while ($i <= 100) { printf("%d ", $i); $i++} exit(); }'
```

使用`continue`和`break`关键字可以使循环短路。

## 14. `return`：提前终止

`return`关键字用于退出当前探测。这与 `exit()`因为它不退出bpftrace。

## 15. `( , )`：元组

支持N元组，其中N是大于1的任何整数。

使用`.`运算符支持索引。元组一旦创建就不可变。

范例：

```sh
# bpftrace -e 'BEGIN { $t = (1, 2, "string"); printf("%d %s\n", $t.1, $t.2); }'
Attaching 1 probe...
2 string
^C
```



# 探针

- `kprobe` -内核函数启动
- `kretprobe` -内核函数返回
- `uprobe` -用户级功能启动
- `uretprobe` -用户级函数返回
- `tracepoint` -内核静态跟踪点
- `usdt` -用户级静态跟踪点
- `profile` -定时采样
- `interval` -定时输出
- `software` -内核软件事件
- `hardware` -处理器级事件

一些探针类型允许通配符匹配多个探针，例如`kprobe:vfs_*`。您也可以指定 使用逗号分隔列表的操作块的多个附加点。

引用的字符串（例如`uprobe:"/usr/lib/c++lib.so":foo`）可用于逃逸 附着点定义中的字符。

## 1. `kprobe`/`kretprobe`：动态跟踪，内核级

语法：

```
kprobe:function_name[+offset]
kretprobe:function_name
```

它们使用kprobes（Linux内核功能）。`kprobe`工具函数的开始 `kretprobe`记录结束（返回）。

示例：

```sh
# bpftrace -e 'kprobe:do_nanosleep { printf("sleep by %d\n", tid); }'
Attaching 1 probe...
sleep by 1396
sleep by 3669
sleep by 1396
sleep by 27662
sleep by 3669
^C
```

也可以在探测函数中指定offset：

```sh
# gdb -q /usr/lib/debug/boot/vmlinux-`uname -r` --ex 'disassemble do_sys_open'
Reading symbols from /usr/lib/debug/boot/vmlinux-5.0.0-32-generic...done.
Dump of assembler code for function do_sys_open:
   0xffffffff812b2ed0 <+0>:     callq  0xffffffff81c01820 <__fentry__>
   0xffffffff812b2ed5 <+5>:     push   %rbp
   0xffffffff812b2ed6 <+6>:     mov    %rsp,%rbp
   0xffffffff812b2ed9 <+9>:     push   %r15
...
# bpftrace -e 'kprobe:do_sys_open+9 { printf("in here\n"); }'
Attaching 1 probe...
in here
...
```



如果地址与指令对齐，则使用vmlinux检查（带有调试符号） 边界和功能。 如果不是，我们不添加它：

```
# bpftrace -e 'kprobe:do_sys_open+1 { printf("in here\n"); }'
Attaching 1 probe...
Could not add kprobe into middle of instruction: /usr/lib/debug/boot/vmlinux-5.0.0-32-generic:do_sys_open+1
```



如果bpftrace是用`ALLOW_UNSAFE_PROBE`选项编译的，可以使用--unsafe选项跳过检查。 在这种情况下，linux内核仍然检查指令对齐。

默认的vmlinux路径可以使用环境变量`BPFTRACE_VMLINUX`覆盖。

现场示例： [（kprobe）搜索/工具](https://github.com/iovisor/bpftrace/search?q=kprobe%3A+path%3Atools&type=Code) [（kretprobe）/工具](https://github.com/iovisor/bpftrace/search?q=kretprobe%3A+path%3Atools&type=Code)

## 2. `kprobe`/`kretprobe`：动态跟踪，内核级参数

语法：

```
kprobe: arg0, arg1, ..., argN
kretprobe: retval
```

参数可以通过这些变量名访问。`arg0`是第一个参数，只能是 `kprobe`访问。`retval`是检测函数的返回值，并且只能是 `kretprobe`访问。

示例：

```sh
# bpftrace -e 'kprobe:do_sys_open { printf("opening: %s\n", str(arg1)); }'
Attaching 1 probe...
opening: /proc/cpuinfo
opening: /proc/stat
opening: /proc/diskstats
opening: /proc/stat
opening: /proc/vmstat
[...]
```

```sh
# bpftrace -e 'kprobe:do_sys_open { printf("open flags: %d\n", arg2); }'
Attaching 1 probe...
open flags: 557056
open flags: 32768
open flags: 32768
open flags: 32768
[...]
```

```sh
# bpftrace -e 'kretprobe:do_sys_open { printf("returned: %d\n", retval); }'
Attaching 1 probe...
returned: 8
returned: 21
returned: -2
returned: 21
[...]
```

下面是一个struct参数的例子：

```sh
# cat path.bt
#include <linux/path.h>
#include <linux/dcache.h>

kprobe:vfs_open
{
	printf("open path: %s\n", str(((struct path *)arg0)->dentry->d_name.name));
}

# bpftrace path.bt
Attaching 1 probe...
open path: dev
open path: if_inet6
open path: retrans_time_ms
[...]
```

这里arg0被转换为（struct path *），因为这是vfs_open（）的第一个参数。结构 支持与bcc相同，并且基于可用的内核头。这意味着许多人，但不是全部， 结构将可用，您可能需要手动定义一些结构。

如果内核有BTF（BPF类型格式）数据，则所有内核结构始终可用，无需定义 他们。举例来说：

```sh
# bpftrace -e 'kprobe:vfs_open { printf("open path: %s\n", \
                                 str(((struct path *)arg0)->dentry->d_name.name)); }'
Attaching 1 probe...
open path: cmdline
open path: interrupts
[...]
```

有关更多详细信息，请参阅[BTF支持](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#btf-support)。

现场示例： [（kprobe）搜索/工具](https://github.com/iovisor/bpftrace/search?q=kprobe%3A+path%3Atools&type=Code) [（kretprobe）/工具](https://github.com/iovisor/bpftrace/search?q=kretprobe%3A+path%3Atools&type=Code)

## 3. `uprobe`/`uretprobe`：动态跟踪，用户级

语法：

```
uprobe:library_name:function_name[+offset]
uprobe:library_name:offset
uretprobe:library_name:function_name
```

它们使用ubrows（Linux内核功能）。`uprobe`仪表用户级的开始 函数的执行，`uretprobe`记录结束（它的返回）。

要列出可用的uprobe，可以使用任何程序列出二进制文件中的文本段符号，例如 `objdump`和`nm`。举例来说：

```
# objdump -tT /bin/bash | grep readline
00000000007003f8 g    DO .bss	0000000000000004  Base        rl_readline_state
0000000000499e00 g    DF .text	00000000000001c5  Base        readline_internal_char
00000000004993d0 g    DF .text	0000000000000126  Base        readline_internal_setup
000000000046d400 g    DF .text	000000000000004b  Base        posix_readline_initialize
000000000049a520 g    DF .text	0000000000000081  Base        readline
[...]
```

它列出了来自/bin/bash的包含“readline”的各种函数。这些可以使用 #1和#2。

示例：

```
# bpftrace -e 'uretprobe:/bin/bash:readline { printf("read a line\n"); }'
Attaching 1 probe...
read a line
read a line
read a line
^C
```

在跟踪时，这捕获了/bin/bash中`readline()`函数的几次执行。这个例子 在下一节中继续。

也可以使用虚拟地址指定upprobe，如：

```
# objdump -tT /bin/bash | grep main
...
000000000002ec00 g    DF .text  0000000000001868  Base        main
...
# bpftrace -e 'uprobe:/bin/bash:0x2ec00 { printf("in here\n"); }'
Attaching 1 probe...
```

并且要在探测函数中指定偏移量：

```
# objdump -d /bin/bash
...
000000000002ec00 <main@@Base>:
   2ec00:       f3 0f 1e fa             endbr64
   2ec04:       41 57                   push   %r15
   2ec06:       41 56                   push   %r14
   2ec08:       41 55                   push   %r13
   ...
# bpftrace -e 'uprobe:/bin/bash:main+4 { printf("in here\n"); }'
Attaching 1 probe...
...
```

检查地址是否与指令边界对齐。 如果不是，我们就不能添加：

```
# bpftrace -e 'uprobe:/bin/bash:main+1 { printf("in here\n"); }'
Attaching 1 probe...
Could not add uprobe into middle of instruction: /bin/bash:main+1
```

如果bpftrace是用`ALLOW_UNSAFE_PROBE`选项编译的，可以使用--unsafe选项跳过检查：

```
# bpftrace -e 'uprobe:/bin/bash:main+1 { printf("in here\n"); } --unsafe'
Attaching 1 probe...
Unsafe uprobe in the middle of the instruction: /bin/bash:main+1
```

使用--unsafe选项，您还可以将uprobe放置在任意地址上。 当二进制文件被剥离时，这可能会派上用场。

```
$ echo 'int main(){return 0;}' | gcc -xc -o bin -
$ nm bin | grep main
...
0000000000001119 T main
...
$ strip bin
# bpftrace --unsafe -e 'uprobe:bin:0x1119 { printf("main called\n"); }'
Attaching 1 probe...
WARNING: could not determine instruction boundary for uprobe:bin:4377 (binary appears stripped). Misaligned probes can lead to tracee crashes!
```

跟踪库时，只需指定库名而不是 一条完整的路然后将使用`/etc/ld.so.cache`自动解析路径。

```
# bpftrace -e 'uprobe:libc:malloc { printf("Allocated %d bytes\n", arg0); }'
Allocated 4 bytes
...
```

当跟踪C++程序时，可以打开自动符号解混淆 使用`:cpp`前缀：

```
# bpftrace -e 'u:src/bpftrace:cpp:"bpftrace::BPFtrace::add_probe" { print("adding probe\n"); }'
Attaching 1 probe...
adding probe
```

现场示例： [（uprobe搜索/工具](https://github.com/iovisor/bpftrace/search?q=uprobe%3A+path%3Atools&type=Code) [（尿道探针）/工具](https://github.com/iovisor/bpftrace/search?q=uretprobe%3A+path%3Atools&type=Code)

## 4. `uprobe`/`uretprobe`：动态跟踪，用户级参数

语法：

```
uprobe: arg0, arg1, ..., argN
uretprobe: retval
```

参数可以通过这些变量名访问。`arg0`是第一个参数，只能是 `uprobe`访问。`retval`是检测函数的返回值，并且只能是 `uretprobe`访问。

示例：

```
# bpftrace -e 'uprobe:/bin/bash:readline { printf("arg0: %d\n", arg0); }'
Attaching 1 probe...
arg0: 19755784
arg0: 19755016
arg0: 19755784
^C
```

/bin/bash中的`arg0`或`readline()`包含什么？我不知道我不知道我得看看bash的来源 代码来找出它的参数。

```
# bpftrace -e 'uprobe:/lib/x86_64-linux-gnu/libc-2.23.so:fopen { printf("fopen: %s\n", str(arg0)); }'
Attaching 1 probe...
fopen: /proc/filesystems
fopen: /usr/share/locale/locale.alias
fopen: /proc/self/mountinfo
^C
```

在本例中，我知道libc`fopen()`的第一个参数是路径名（参见fopen（3）man 页），所以我用uprobe跟踪了它。调整libc的路径以匹配您的系统（可能不是 libc-2.23.so）上提供。要将char * 指针转到字符串，需要进行`str()`调用，如 后一节。

```
# bpftrace -e 'uretprobe:/bin/bash:readline { printf("readline: \"%s\"\n", str(retval)); }'
Attaching 1 probe...
readline: "echo hi"
readline: "ls -l"
readline: "date"
readline: "uname -r"
^C
```

回到bash `readline()`示例：在检查源代码后，我看到返回值为 字符串读。所以我可以使用`uretprobe`和`retval`变量来查看读取的字符串。

------

如果被跟踪的二进制文件具有DWARF可用，则可以通过名称访问`uprobe`参数。

语法：

```
uprobe: args.NAME
```

参数可以作为内置`args`结构的字段访问。

可以使用verbose list选项检索函数的参数列表：

```
# bpftrace -lv 'uprobe:/bin/bash:rl_set_prompt'
uprobe:/bin/bash:rl_set_prompt
    const char* prompt
```

示例（需要安装`/bin/bash`的debuginfo）：

```
# bpftrace -e 'uprobe:/bin/bash:rl_set_prompt { printf("prompt: %s\n", str(args.prompt)); }'
Attaching 1 probe...
prompt: [user@localhost ~]$
^C
```

现场示例： [（uprobe）搜索/工具](https://github.com/iovisor/bpftrace/search?q=uprobe%3A+path%3Atools&type=Code) [（尿道探针）/工具](https://github.com/iovisor/bpftrace/search?q=uretprobe%3A+path%3Atools&type=Code)

## 5. `tracepoint`静态跟踪，内核级

语法：`tracepoint:name`

它们使用跟踪点（一种Linux内核功能）。

```
# bpftrace -e 'tracepoint:block:block_rq_insert { printf("block I/O created by %d\n", tid); }'
Attaching 1 probe...
block I/O created by 28922
block I/O created by 3949
block I/O created by 883
block I/O created by 28941
block I/O created by 28941
block I/O created by 28941
[...]
```

现场示例： [搜索/工具](https://github.com/iovisor/bpftrace/search?q=tracepoint%3A+path%3Atools&type=Code)

## 6. `tracepoint`：静态跟踪、内核级参数

范例：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args.filename)); }'
Attaching 1 probe...
irqbalance /proc/interrupts
irqbalance /proc/stat
snmpd /proc/diskstats
snmpd /proc/stat
snmpd /proc/vmstat
snmpd /proc/net/dev
[...]
```

每个跟踪点的可用成员可以从/sys中的/format文件中列出。举例来说：

```
# cat /sys/kernel/debug/tracing/events/syscalls/sys_enter_open/format
name: sys_enter_openat
ID: 608
format:
	field:unsigned short common_type;	offset:0;	size:2;	signed:0;
	field:unsigned char common_flags;	offset:2;	size:1;	signed:0;
	field:unsigned char common_preempt_count;	offset:3;	size:1;	signed:0;
	field:int common_pid;	offset:4;	size:4;	signed:1;

	field:int __syscall_nr;	offset:8;	size:4;	signed:1;
	field:int dfd;	offset:16;	size:8;	signed:0;
	field:const char * filename;	offset:24;	size:8;	signed:0;
	field:int flags;	offset:32;	size:8;	signed:0;
	field:umode_t mode;	offset:40;	size:8;	signed:0;

print fmt: "dfd: 0x%08lx, filename: 0x%08lx, flags: 0x%08lx, mode: 0x%08lx", ((unsigned long)(REC->dfd)), ((unsigned long)(REC->filename)), ((unsigned long)(REC->flags)), ((unsigned long)(REC->mode))
```

除了`filename`成员，我们还可以打印`flags`、`mode`等。“共同”成员之后 首先列出的成员是跟踪点特有的。

现场示例： [搜索/工具](https://github.com/iovisor/bpftrace/search?q=tracepoint%3A+path%3Atools&type=Code)

## 7. `rawtracepoint`静态跟踪，内核级

`tracepoint`和`rawtracepoint`触发的挂钩点相同。不同之处在于，原始跟踪点速度更快，因为没有进行参数处理（见下文）。

语法：`tracepoint:name`

它们使用跟踪点（一种Linux内核功能）。

```
# bpftrace -e 'rawtracepoint:block_rq_insert { printf("block I/O created by %d\n", tid); }'
Attaching 1 probe...
block I/O created by 189126
[...]
```



## 8. `rawtracepoint`：静态跟踪、内核级参数

`tracepoint`和`rawtracepoint`在功能上几乎相同。唯一的区别是在程序上下文。`rawtracepoint`为跟踪点提供原始参数，而`tracepoint`对原始参数应用进一步处理。额外的处理是在内核内部定义的。

范例：

```
# bpftrace -e 'rawtracepoint:block_rq_insert { printf("%llx %llx\n", arg0, arg1); }'
Attaching 1 probe...
ffff88810977d6f8 ffff8881097e8e80
[...]
```



每个跟踪点的可用参数可以在内核源代码的相对路径include/trace/events/中找到。举例来说：

```
include/trace/events/block.h
DEFINE_EVENT(block_rq, block_rq_insert,
	TP_PROTO(struct request_queue *q, struct request *rq),
	TP_ARGS(q, rq)
);
```



每个args通过`argN`内置程序按其顺序访问。每个arg都是一个64位整数。

## 9. `usdt`：静态跟踪，用户级

语法：

```
usdt:binary_path:probe_name
usdt:binary_path:[probe_namespace]:probe_name
usdt:library_path:probe_name
usdt:library_path:[probe_namespace]:probe_name
```



其中，如果`probe_namespace`在二进制中是唯一的，`probe_name`是可选的。

示例：

```
# bpftrace -e 'usdt:/root/tick:loop { printf("hi\n"); }'
Attaching 1 probe...
hi
hi
hi
hi
hi
^C
```



探测器的命名空间是自动推导的。如果二进制`/root/tick`包含多个探针 名称`loop`（例如`tick:loop`和`tock:loop`），则不会连接探针。 这可以通过手动指定命名空间或使用通配符来解决：

```
# bpftrace -e 'usdt:/root/tick:loop { printf("hi\n"); }'
ERROR: namespace for usdt:/root/tick:loop not specified, matched 2 probes
INFO: please specify a unique namespace or use '*' to attach to all matched probes
No probes to attach

# bpftrace -e 'usdt:/root/tick:tock:loop { printf("hi\n"); }'
Attaching 1 probe...
hi
hi
^C

# bpftrace -e 'usdt:/root/tick:*:loop { printf("hi\n"); }'
Attaching 2 probes...
hi
hi
hi
hi
^C
```



bpftrace还支持USDT信号量。如果您的环境和bpftrace 支持uprobe引用计数，则USDT信号量将自动激活 探针连接时的所有过程（并且`--usdt-file-activation`变为 noop）。您可以通过运行以下命令来检查系统是否支持uprobe refounctions：

```
# bpftrace --info 2>&1 | grep "uprobe refcount"
  bcc bpf_attach_uprobe refcount: yes
  uprobe refcount (depends on Build:bcc bpf_attach_uprobe refcount): yes
```



如果您的系统不支持uprobe引用计数，您可以通过传入`-p $PID`或 `--usdt-file-activation`. `--usdt-file-activation`查看`/proc`以查找 将探测器的二进制文件与可执行权限映射到它们的地址空间，然后尝试 连接你的探测器请注意，文件激活仅发生一次（在附加时间内）。在其他 如果稍后在跟踪会话期间产生了一个带有可执行文件的新进程，则您的 当前跟踪会话不会激活新进程。`--usdt-file-activation` 根据文件路径匹配。这意味着，如果BPFTrace从根主机运行，事情可能无法工作 如果存在来自私有挂载命名空间或绑定挂载目录的进程`execve`d，则与预期相同。 一种解决方法是在适当的名称空间（即容器）内运行bpftrace。

## 10. `usdt`：静态跟踪，用户级参数

示例：

```
# bpftrace -e 'usdt:/root/tick:loop { printf("%s: %d\n", str(arg0), arg1); }'
my string: 1
my string: 2
my string: 3
my string: 4
my string: 5
^C
```



```
# bpftrace -e 'usdt:/root/tick:loop /arg1 > 2/ { printf("%s: %d\n", str(arg0), arg1); }'
my string: 3
my string: 4
my string: 5
my string: 6
^C
```



## 11. `profile`：定时采样事件

语法：

```
profile:hz:rate
profile:s:rate
profile:ms:rate
profile:us:rate
```



它们使用perf_events（Linux内核工具，也被`perf`命令使用）来操作。

示例：

```
# bpftrace -e 'profile:hz:99 { @[tid] = count(); }'
Attaching 1 probe...
^C

@[32586]: 98
@[0]: 579
```



## 12. `interval`：定时输出

语法：

```
interval:ms:rate
interval:s:rate
interval:us:rate
interval:hz:rate
```



这仅在一个CPU上触发，可用于生成每个间隔的输出。

范例：

```
# bpftrace -e 'tracepoint:raw_syscalls:sys_enter { @syscalls = count(); }
    interval:s:1 { print(@syscalls); clear(@syscalls); }'
Attaching 2 probes...
@syscalls: 1263
@syscalls: 731
@syscalls: 891
@syscalls: 1195
@syscalls: 1154
@syscalls: 1635
@syscalls: 1208
[...]
```



这将打印每秒的syscalls速率。

现场示例： [搜索/工具](https://github.com/iovisor/bpftrace/search?q=interval+extension%3Abt+path%3Atools&type=Code)

## 13. `software`：预定义的软件事件

语法：

```
software:event_name:count
software:event_name:
```



这些是Linux内核提供的预定义软件事件，通常通过perf跟踪 效用它们类似于tracepoint，但只有大约12个这样的，它们是 在perf_event_open（2）手册页中进行了记录。事件名称为：

- \#1或#2
- `task-clock`
- \#1或#2
- \#1或#2
- `cpu-migrations`
- `minor-faults`
- `major-faults`
- `alignment-faults`
- `emulation-faults`
- `dummy`
- `bpf-output`

count是探测器的触发器，对于每个count事件都会触发一次。如果计数不是 提供，则使用默认值。

示例：

```
# bpftrace -e 'software:faults:100 { @[comm] = count(); }'
Attaching 1 probe...
^C

@[ls]: 1
@[pager]: 2
@[locale]: 2
@[preconv]: 2
@[sh]: 3
@[tbl]: 3
@[bash]: 4
@[groff]: 5
@[grotty]: 7
@[sleep]: 9
@[nroff]: 12
@[troff]: 18
@[man]: 97
```



通过对每一百个进程名进行采样，可以粗略地统计谁导致了页面错误 缺点

## 14. `hardware`：预定义硬件事件

语法：

```
hardware:event_name:count
hardware:event_name:
```



这些是Linux内核提供的预定义硬件事件，通常由perf跟踪 效用它们是使用性能监视计数器（PMC）实现的：上的硬件资源 处理器。其中大约有十个，它们都在perf_event_open（2）手册页中进行了记录。 事件名称为：

- \#1或#2
- `instructions`
- `cache-references`
- `cache-misses`
- \#1或#2
- `branch-misses`
- `bus-cycles`
- `frontend-stalls`
- `backend-stalls`
- `ref-cycles`

count是探测器的触发器，对于每个count事件都会触发一次。如果计数不是 提供，则使用默认值。

示例：

```
bpftrace -e 'hardware:cache-misses:1000000 { @[pid] = count(); }'
```



每1000000次缓存未命中将触发一次。这通常表示末级缓存（LLC）。

## 15. `BEGIN`/`END`：内置事件

语法：

```
BEGIN
END
```



这些是bpftrace运行时提供的特殊内置事件。`BEGIN`在所有其他之前触发 探针被连接。`END`在所有其他探头分离后触发。

现场示例： [（开始）search /tools](https://github.com/iovisor/bpftrace/search?q=BEGIN+extension%3Abt+path%3Atools&type=Code) [（END）搜索/工具](https://github.com/iovisor/bpftrace/search?q=END+extension%3Abt+path%3Atools&type=Code)

## 16. `watchpoint`/`asyncwatchpoint`：内存监视点

警告：此功能是实验性的，可能会受到界面更改的影响。内存监视点包括 也依赖于体系结构

语法：

```
watchpoint:absolute_address:length:mode
watchpoint:function+argN:length:mode
```



这些是内核提供的内存观察点。每当向（`w`）写入存储器地址时，读取 从（`r`）或执行（`x`），内核可以生成事件。

在第一种形式中，监视绝对地址。如果提供pid（`-p`）或命令（`-c`）， bpftrace将该地址作为用户空间地址并监视适当的进程。如果不是 bpftrace将该地址作为内核空间地址。

在第二种形式中，是`argN`中存在的地址（参见[uprobe 参数](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#4-uprobeuretprobe-dynamic-tracing-user-level-arguments)）输入`function`时为 监测。必须为此表单提供pid或命令。如果同步（`watchpoint`），则a `SIGSTOP`在函数进入时被发送到tracee。tracee将在 watchpoint是附加的。这是为了确保不会错过事件。如果你想避免 `SIGCONT` + `SIGSTOP`使用`SIGCONT`。

请注意，在大多数体系结构上，在监视读或写时，您可能不会监视执行情况。

示例：

```
bpftrace -e 'watchpoint:0x10000000:8:rw { printf("hit!\n"); exit(); }' -c ./testprogs/watchpoint
```



当监视点进程试图读取或写入0x10000000时，它将输出“hit”并退出。

```
# bpftrace -e "watchpoint:0x$(awk '$3 == "jiffies" {print $1}' /proc/kallsyms):8:w {@[kstack] = count();}"
Attaching 1 probe...
^C
......
@[
    do_timer+12
    tick_do_update_jiffies64.part.22+89
    tick_sched_do_timer+103
    tick_sched_timer+39
    __hrtimer_run_queues+256
    hrtimer_interrupt+256
    smp_apic_timer_interrupt+106
    apic_timer_interrupt+15
    cpuidle_enter_state+188
    cpuidle_enter+41
    do_idle+536
    cpu_startup_entry+25
    start_secondary+355
    secondary_startup_64+164
]: 319
```



它显示了更新jifies的内核堆栈。

```
# cat wpfunc.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((noinline))
void increment(__attribute__((unused)) int _, int *i)
{
  (*i)++;
}

int main()
{
  int *i = malloc(sizeof(int));
  while (1)
  {
    increment(0, i);
    (*i)++;
    usleep(1000);
  }
}

# bpftrace -e 'watchpoint:increment+arg1:4:w { printf("hit!\n"); exit() }' -c ./wpfunc
```



bpftrace将输出“hit”并在`arg1`的`increment`指向的内存为 写的

## 17. `kfunc`/`kretfunc`：内核函数跟踪

语法：

```
kfunc[:module]:function
kretfunc[:module]:function
```



这些是通过eBPF蹦床实现的内核函数探测器，它允许 内核代码调用BPF程序，开销几乎为零。

如果没有给定内核模块，则搜索所有加载的模块以查找给定的函数。

示例：

```
# bpftrace -e 'kfunc:x86_pmu_stop { printf("pmu %s stop\n", str(args.event->pmu->name)); }'
# bpftrace -e 'kretfunc:fget { printf("fd %d name %s\n", args.fd, str(retval->f_path.dentry->d_name.name));  }'
# bpftrace -e 'kfunc:kvm:x86_emulate_insn { @ = count(); }'
```



您可以通过列表选项获取可用功能列表：

```
# bpftrace -l
...
kfunc:vmlinux:ksys_ioperm
kfunc:vmlinux:ksys_unshare
kfunc:vmlinux:ksys_setsid
kfunc:vmlinux:ksys_sync_helper
kfunc:vmlinux:ksys_fadvise64_64
kfunc:vmlinux:ksys_readahead
kfunc:vmlinux:ksys_mmap_pgoff
...
```



## 18. `kfunc`/`kretfunc`：内核函数跟踪参数

语法：

```
kfunc[:module]:function      args.NAME  ...
kretfunc[:module]:function   args.NAME ... retval
```



参数可以作为内置`args`结构的字段访问。 返回值可以被`retval`内置引用，参见[1.内置](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#1-builtins)。

可以通过verbose list选项获取函数的可用参数名称：

```
# bpftrace -lv
...
kfunc:fget
    unsigned int fd;
    struct file * retval;
...
```



`fget`函数接受一个参数作为文件描述符 您可以通过`args.fd`探针中的`kfunc:fget`访问它：

```
# bpftrace -e 'kfunc:fget { printf("fd %d\n", args.fd);  }'
Attaching 1 probe...
fd 3
fd 3
...
```



`fget`函数探针的返回值可通过`retval`访问：

```
# bpftrace -e 'kretfunc:fget { printf("fd %d name %s\n", args.fd, str(retval->f_path.dentry->d_name.name));  }'
Attaching 1 probe...
fd 3 name ld.so.cache
fd 3 name libselinux.so.1
fd 3 name libselinux.so.1
...
```



正如你在上面的例子中看到的，也可以访问`kretfunc`探测器上的函数参数。

## 19. `iter`：迭代器跟踪

警告：此功能是实验性的，可能会受到界面更改的影响。

语法：

```
iter:task[:pin]
iter:task_file[:pin]
```



内核：5.4

这些是eBPF迭代器探测器，允许在内核对象上进行迭代。

迭代器探测器不能与任何其他探测器混合，甚至不能与其他迭代器混合。

每个迭代器探测器都提供了一组字段，这些字段可以用 ctx指针。用户可以通过以下方式显示迭代器的可用字段集 -LV选项如下所述。

示例：

```
# bpftrace -e 'iter:task { printf("%s:%d\n", ctx->task->comm, ctx->task->pid); }'
Attaching 1 probe...
systemd:1
kthreadd:2
rcu_gp:3
rcu_par_gp:4
kworker/0:0H:6
mm_percpu_wq:8
...

# bpftrace -e 'iter:task_file { printf("%s:%d %d:%s\n", ctx->task->comm, ctx->task->pid, ctx->fd, path(ctx->file->f_path)); }'
Attaching 1 probe...
systemd:1 1:/dev/null
systemd:1 2:/dev/null
systemd:1 3:/dev/kmsg
...
su:1622 1:/dev/pts/1
su:1622 2:/dev/pts/1
su:1622 3:/var/lib/sss/mc/passwd
...
bpftrace:1892 1:pipe:[35124]
bpftrace:1892 2:/dev/pts/1
bpftrace:1892 3:anon_inode:bpf-map
bpftrace:1892 4:anon_inode:bpf-map
bpftrace:1892 5:anon_inode:bpf_link
bpftrace:1892 6:anon_inode:bpf-prog
bpftrace:1892 7:anon_inode:bpf_iter
```



您可以通过列表选项获得可用函数的列表：

```
# bpftrace -l iter:*
iter:task
iter:task_file

# bpftrace -l iter:* -v
iter:task
    struct task_struct *task;
iter:task_file
    struct task_struct *task;
    int fd;
    struct file *file;
```



可以通过指定可选的探测器'：pin'部分来固定迭代器， 定义pin文件。它可以指定为绝对路径或相对路径 到/sys/fs/bpf。

相对端号文件示例：

```
# bpftrace -e 'iter:task:list { printf("%s:%d\n", ctx->task->comm, ctx->task->pid); }'
Attaching 1 probe...
Program pinned to /sys/fs/bpf/list


# cat /sys/fs/bpf/list
systemd:1
kthreadd:2
rcu_gp:3
rcu_par_gp:4
kworker/0:0H:6
mm_percpu_wq:8
rcu_tasks_kthre:9
...
```



使用绝对pin文件的示例：

```
# bpftrace -e 'iter:task_file:/sys/fs/bpf/files { printf("%s:%d %s\n", ctx->task->comm, ctx->task->pid, path(ctx->file->f_path)); }'
Attaching 1 probe...
Program pinned to /sys/fs/bpf/files

# cat /sys/fs/bpf/files
systemd:1 anon_inode:inotify
systemd:1 anon_inode:[timerfd]
...
systemd-journal:849 /dev/kmsg
systemd-journal:849 anon_inode:[eventpoll]
...
sssd:1146 /var/log/sssd/sssd.log
sssd:1146 anon_inode:[eventpoll]
...
NetworkManager:1155 anon_inode:[eventfd]
NetworkManager:1155 /var/lib/sss/mc/passwd (deleted)
```



# 变量

## 1.内置变量

- `pid` -进程ID（内核tgid）
- `tid` -线程ID（内核pid）
- `uid`用户ID
- `gid` -组ID
- `nsecs` -纳秒时间戳
- `elapsed` -自bpftrace初始化以来的纳秒
- `numaid` - NUMA节点ID
- `cpu` -处理器ID
- `comm` -进程名称
- `kstack` -内核堆栈跟踪
- `ustack` -用户堆栈跟踪
- `arg0`，`arg1`，…`argN`. - 被跟踪函数的参数;假定为64位宽
- `sarg0`，`sarg1`，…`sargN`. - 跟踪函数的参数（对于存储参数的程序 在堆栈上）;假定为64位宽
- `retval` -从跟踪函数返回值
- `args` -包含被跟踪函数的所有参数的结构体。 可用于`tracepoint`、`kfunc`和`uprobe`（带DWARF）探头。使用方式 `args.x`访问参数`x`或`args`获取包含所有参数的记录。
- `func` -被跟踪函数的名称
- `probe` -探头全名
- `curtask` -当前任务结构为u64
- `rand` -作为u32的随机数
- `cgroup` -当前进程的Cgroup ID
- `cpid` - Child pid（u32），仅在`-c command`标志下有效
- `$1`，`$2`，…三号四号- bpftrace程序的位置参数

其中许多在其他章节中讨论（使用搜索）。

## 2. `@`, `$`基本变量

语法：

```sh
@global_name //BPF映射变量
@thread_local_variable_name[tid]
$scratch_name //临时变量
```

bpftrace支持全局每线程变量（通过BPF映射）和临时变量。

示例：

### 2.1.全局

语法：`@name`

例如：`@start`

```sh
# bpftrace -e 'BEGIN { @start = nsecs; }
    kprobe:do_nanosleep /@start != 0/ { printf("at %d ms: sleep\n", (nsecs - @start) / 1000000); }'
Attaching 2 probes...
at 437 ms: sleep
at 647 ms: sleep
at 1098 ms: sleep
at 1438 ms: sleep
at 1648 ms: sleep
^C

@start: 4064438886907216
```

### 2.2.每线程：

这些可以被实现为在线程ID上键控的关联数组。例如，`@start[tid]`：

```
# bpftrace -e 'kprobe:do_nanosleep { @start[tid] = nsecs; }
    kretprobe:do_nanosleep /@start[tid] != 0/ {
        printf("slept for %d ms\n", (nsecs - @start[tid]) / 1000000); delete(@start[tid]); }'
Attaching 2 probes...
slept for 1000 ms
slept for 1000 ms
slept for 1000 ms
slept for 1009 ms
slept for 2002 ms
[...]
```

### 2.3.临时变量：

语法：`$name`

例如：`$delta`

```sh
# bpftrace -e 'kprobe:do_nanosleep { @start[tid] = nsecs; }
    kretprobe:do_nanosleep /@start[tid] != 0/ { $delta = nsecs - @start[tid];
        printf("slept for %d ms\n", $delta / 1000000); delete(@start[tid]); }'
Attaching 2 probes...
slept for 1000 ms
slept for 1000 ms
slept for 1000 ms
```

## 3. `@[]`：BPF映射数组

语法：

```sh
@associative_array_name[key_name] = value
@associative_array_name[key_name, key_name2, ...] = value
```

这些是使用BPF映射实现的。

例如，`@start`：

```sh
# bpftrace -e 'kprobe:do_nanosleep { @start[tid] = nsecs; }
    kretprobe:do_nanosleep /@start[tid] != 0/ {
        printf("slept for %d ms\n", (nsecs - @start[tid]) / 1000000); delete(@start[tid]); }'
Attaching 2 probes...
slept for 1000 ms
slept for 1000 ms
slept for 1000 ms
[...]
```

```
# bpftrace -e 'BEGIN { @[1,2] = 3; printf("%d\n", @[1,2]); clear(@); }'
Attaching 1 probe...
3
^C
```

## 4. `count()`：频率计数

这是由count（）函数提供的：请参见“[计数](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#2-count-count)”部分。

## 5. `hist()`, `lhist()`：直方图

这些是由hist（）和lhist（）函数提供的。参见[Log2直方图](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#8-hist-log2-histogram) 和[线性直方图](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#9-lhist-linear-histogram)部分。

## 6. `nsecs`：时间戳和时间增量

语法：`tracepoint:name`

这些是使用bpf_ktime_get_ns（）实现的。

示例：

```
# bpftrace -e 'BEGIN { @start = nsecs; }
    kprobe:do_nanosleep /@start != 0/ { printf("at %d ms: sleep\n", (nsecs - @start) / 1000000); }'
Attaching 2 probes...
at 437 ms: sleep
at 647 ms: sleep
at 1098 ms: sleep
at 1438 ms: sleep
^C
```



## 7. `kstack`：堆栈跟踪，内核

语法：`tracepoint:name`

这个内置是[`kstack()`](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#15-kstack-stack-traces-kernel)的别名。

示例：

```
# bpftrace -e 'kprobe:ip_output { @[kstack] = count(); }'
Attaching 1 probe...
[...]
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    tcp_release_cb+225
    release_sock+64
    tcp_sendmsg+49
    sock_sendmsg+48
    sock_write_iter+135
    __vfs_write+247
    vfs_write+179
    sys_write+82
    entry_SYSCALL_64_fastpath+30
]: 1708
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    __tcp_push_pending_frames+45
    tcp_sendmsg_locked+2637
    tcp_sendmsg+39
    sock_sendmsg+48
    sock_write_iter+135
    __vfs_write+247
    vfs_write+179
    sys_write+82
    entry_SYSCALL_64_fastpath+30
]: 9048
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    tcp_tasklet_func+348
    tasklet_action+241
    __do_softirq+239
    irq_exit+174
    do_IRQ+74
    ret_from_intr+0
    cpuidle_enter_state+159
    do_idle+389
    cpu_startup_entry+111
    start_secondary+398
    secondary_startup_64+165
]: 11430
```



## 8. `ustack`：堆栈跟踪，用户

语法：`tracepoint:name`

这个内置是[`ustack()`](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#16-ustack-stack-traces-user)的别名。

示例：

```
# bpftrace -e 'kprobe:do_sys_open /comm == "bash"/ { @[ustack] = count(); }'
Attaching 1 probe...
^C

@[
    __open_nocancel+65
    command_word_completion_function+3604
    rl_completion_matches+370
    bash_default_completion+540
    attempt_shell_completion+2092
    gen_completion_matches+82
    rl_complete_internal+288
    rl_complete+145
    _rl_dispatch_subseq+647
    _rl_dispatch+44
    readline_internal_char+479
    readline_internal_charloop+22
    readline_internal+23
    readline+91
    yy_readline_get+152
    yy_readline_get+429
    yy_getc+13
    shell_getc+469
    read_token+251
    yylex+192
    yyparse+777
    parse_command+126
    read_command+207
    reader_loop+391
    main+2409
    __libc_start_main+231
    0x61ce258d4c544155
]: 9
@[
    __open_nocancel+65
    command_word_completion_function+3604
    rl_completion_matches+370
    bash_default_completion+540
    attempt_shell_completion+2092
    gen_completion_matches+82
    rl_complete_internal+288
    rl_complete+89
    _rl_dispatch_subseq+647
    _rl_dispatch+44
    readline_internal_char+479
    readline_internal_charloop+22
    readline_internal+23
    readline+91
    yy_readline_get+152
    yy_readline_get+429
    yy_getc+13
    shell_getc+469
    read_token+251
    yylex+192
    yyparse+777
    parse_command+126
    read_command+207
    reader_loop+391
    main+2409
    __libc_start_main+231
    0x61ce258d4c544155
]: 18
```



请注意，要使这个例子正常工作，bash必须使用帧指针重新编译。

## 9. `$1`，... `$N`, `$#`：位置参数

语法：`$1`，`$2`，...，`$N`、`$#`

这些是bpftrace程序的位置参数，也称为命令行参数。 如果参数是数字（完全是数字），则可以用作数字。如果它是非数字的，则必须 在`str()`调用中用作字符串。如果使用了未提供的参数，则默认为 零表示数值上下文，“”表示字符串上下文。位置参数也可用于探头 参数，并将被视为字符串参数。

如果在`str()`中使用了位置参数，则将其解释为指向实际给定字符串的指针 literal，它允许对它进行指针运算。只有一个常数的加法，小于或等于 所提供字符串的长度是允许的。

`$#`返回提供的位置参数的数量。

这允许编写使用基本参数来更改其行为的脚本。如果你开发了一个 需要更复杂的参数处理的脚本，它可能更适合于bcc，因为bcc 支持Python的argparse和完全自定义的参数处理。

单行代码示例：

```
# bpftrace -e 'BEGIN { printf("I got %d, %s (%d args)\n", $1, str($2), $#); }' 42 "hello"
Attaching 1 probe...
I got 42, hello (2 args)

# bpftrace -e 'BEGIN { printf("%s\n", str($1 + 1)) }' "hello"
Attaching 1 probe...
ello
```

脚本示例bsize.d：

```
#!/usr/local/bin/bpftrace

BEGIN
{
	printf("Tracing block I/O sizes > %d bytes\n", $1);
}

tracepoint:block:block_rq_issue
/args.bytes > $1/
{
	@ = hist(args.bytes);
}
```

当使用65536参数运行时：

```
# ./bsize.bt 65536
Attaching 2 probes...
Tracing block I/O sizes > 65536 bytes
^C

@:
[512K, 1M)             1 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
```

它将参数作为$1传入，并将其用作过滤器。

如果没有参数，$1默认为零：

```
# ./bsize.bt
Attaching 2 probes...
Tracing block I/O sizes > 0 bytes
^C

@:
[4K, 8K)             115 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[8K, 16K)             35 |@@@@@@@@@@@@@@@                                     |
[16K, 32K)             5 |@@                                                  |
[32K, 64K)             3 |@                                                   |
[64K, 128K)            1 |                                                    |
[128K, 256K)           0 |                                                    |
[256K, 512K)           0 |                                                    |
[512K, 1M)             1 |                                                    |
```

# 内置函数

## 1.建筑物

- `printf(char *fmt, ...)` -打印格式
- `time(char *fmt)` -打印格式化时间
- `join(char *arr[] [, char *delim])` -打印数组
- `str(char *s [, int length])` -返回s指向的字符串
- `ksym(void *p)` -解析内核地址
- `usym(void *p)` -解析用户空间地址
- `kaddr(char *name)` -解析内核符号名称
- `uaddr(char *name)` -解析用户级符号名称
- `reg(char *name)` -返回存储在命名寄存器中的值
- `system(char *fmt)` -执行shell命令
- `exit()` -退出bpftrace
- `cgroupid(char *path)` -解析cgroup ID
- `kstack` -内核堆栈跟踪
- `ustack` -用户堆栈跟踪
- `ntop([int af, ]int|char[4|16] addr)` -将IP地址数据转换为文本
- `pton(const string *addr)` -将文本IP地址转换为字节数组
- `cat(char *filename)` -打印文件内容
- `signal(char[] signal | u32 signal)` -向当前任务发送信号
- `strncmp(char *s1, char *s2, int length)` -比较两个字符串的前n个字符
- `strcontains(const char *haystack, const char *needle)` -比较字符串干草堆是否包含字符串针。
- `override(u64 rc)` -覆盖返回值
- `buf(void *d [, int length])` -返回由d指向的数据的十六进制格式字符串
- `sizeof(...)` -返回类型或表达式的大小
- `print(...)` -使用默认格式打印非映射值
- `strftime(char *format, int nsecs)` -返回格式化的时间戳
- `path(struct path *path)` -返回完整路径
- `uptr(void *p)` -注释为用户空间指针
- `kptr(void *p)` -注释为内核空间指针
- `macaddr(char[6] addr)` -转换MAC地址数据
- `bswap(uint[8|16|32|64] n)` -反向字节顺序
- `offsetof(struct, element)` -结构中元素的偏移量
- `nsecs([TimestampMode mode])` -时间戳和时间增量

其中一些是异步的：内核将事件排队，但在一段时间（毫秒）之后 在用户空间中处理。异步操作包括：`printf()`、`time()`、`join()`。`ksym()` 和`usym()`，以及变量`kstack`和`ustack`，同步记录地址，但随后 符号转换异步。

以下各节将讨论其中的一部分。

## 2. `printf()`：印刷

语法：`tracepoint:name`

它的行为类似于C和其他语言中的printf（），具有有限的格式字符集。范例：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s called %s\n", comm, str(args.filename)); }'
Attaching 1 probe...
bash called /bin/ls
bash called /usr/bin/man
man called /apps/nflx-bash-utils/bin/preconv
man called /usr/local/sbin/preconv
man called /usr/local/bin/preconv
man called /usr/sbin/preconv
man called /usr/bin/preconv
man called /apps/nflx-bash-utils/bin/tbl
[...]
```



## 3. `time()`：时间

语法：`tracepoint:name`

这将使用libc `strftime(3)`支持的格式字符串打印当前时间。

```
# bpftrace -e 'kprobe:do_nanosleep { time("%H:%M:%S\n"); }'
07:11:03
07:11:09
^C
```



如果未提供格式字符串，则默认为“%H：%M：%S\n”。

请注意，此内置函数是异步的。打印的时间戳是 哪个用户空间处理了排队事件，而不是 bpf程序调用`time()`。有关更精确的时间戳，请参见 [public void run（）](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#24-strftime-formatted-timestamp).

## 4. `join()`：加入

语法：`tracepoint:name`

这将用一个空格字符连接字符串数组，并打印出来，用分隔符分隔。该 默认分隔符（如果未提供）是空格字符。此当前版本不返回 字符串，所以它不能用作printf（）中的参数。范例：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_execve { join(args.argv); }'
Attaching 1 probe...
ls --color=auto
man ls
preconv -e UTF-8
preconv -e UTF-8
preconv -e UTF-8
preconv -e UTF-8
preconv -e UTF-8
tbl
[...]
```



```
# bpftrace -e 'tracepoint:syscalls:sys_enter_execve { join(args.argv, ","); }'
Attaching 1 probe...
ls,--color=auto
man,ls
preconv,-e,UTF-8
preconv,-e,UTF-8
preconv,-e,UTF-8
preconv,-e,UTF-8
preconv,-e,UTF-8
tbl
[...]
```



## 5. `str()`：字符串

语法：`tracepoint:name`

返回由% s指向的字符串。`length`可以用于限制读取的大小，和/或引入 空终止符。默认情况下，字符串的大小为64字节（可使用env var进行调整[ `BPFTRACE_STRLEN`](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#91-bpftrace_strlen)）。

示例：

我们可以取`args.filename`的`sys_enter_execve`（一个`const char *filename`），并将字符串读到 它所指向的。这个字符串可以作为printf（）的参数提供：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s called %s\n", comm, str(args.filename)); }'
Attaching 1 probe...
bash called /bin/ls
bash called /usr/bin/man
man called /apps/nflx-bash-utils/bin/preconv
man called /usr/local/sbin/preconv
man called /usr/local/bin/preconv
man called /usr/sbin/preconv
man called /usr/bin/preconv
man called /apps/nflx-bash-utils/bin/tbl
[...]
```



我们可以跟踪bash shell中显示的字符串。采用一些长度调整，因为：

- sys_enter_write（）'s 

  ```
  args.buf
  ```

   不指向以null结尾的字符串

  - 我们使用length参数来限制要读取的指向字符串的字节数

- sys_enter_write（）'s 

  ```
  args.buf
  ```

   包含大于64字节的消息

  - 我们增加BPPTRACE_STRLEN以容纳大型消息

```
# BPFTRACE_STRLEN=200 bpftrace -e 'tracepoint:syscalls:sys_enter_write /pid == 23506/
    { printf("<%s>\n", str(args.buf, args.count)); }'
# type pwd into terminal 23506
<p>
<w>
<d>
# press enter in terminal 23506
<
>
</home/anon
>
<anon@anon-VirtualBox:~$ >
```



## 6. `ksym()`：符号分辨率，内核级

语法：`tracepoint:name`

示例：

```
# bpftrace -e 'kprobe:do_nanosleep { printf("%s\n", ksym(reg("ip"))); }'
Attaching 1 probe...
do_nanosleep
do_nanosleep
```



## 7. `usym()`：符号分辨率，用户级

语法：`tracepoint:name`

示例：

```
# bpftrace -e 'uprobe:/bin/bash:readline { printf("%s\n", usym(reg("ip"))); }'
Attaching 1 probe...
readline
readline
readline
^C
```



## 8. `kaddr()`：地址解析，内核级

语法：`tracepoint:name`

示例：

```
# bpftrace -e 'BEGIN { printf("%s\n", str(*kaddr("usbcore_name"))); }'
Attaching 1 probe...
usbcore
^C
```



这是从drivers/usb/core/usb.c打印`usbcore_name`字符串：

```
const char *usbcore_name = "usbcore";
```



## 9. `uaddr()`：地址解析，用户级

语法：

- `u64 *uaddr(symbol)`（默认）
- `u64 *uaddr(symbol)`
- `u32 *uaddr(symbol)`
- `u16 *uaddr(symbol)`
- `u8  *uaddr(symbol)`

支持的探头类型：

- u（ret）探针
- 超声波时差

**不适用于ASLR，参见问题 [#75](https://github.com/iovisor/bpftrace/issues/75)**

`uaddr`函数返回指定符号的地址。此查找 在程序编译期间发生，不能动态使用。

默认返回类型为`u64*`。如果ELF对象大小与已知的 整数大小（1、2、4或8字节），修改返回类型以匹配宽度 （分别为`u8*`、`u16*`、`u32*`或`u64*`）。由于ELF不包含类型信息 类型总是假定为无符号的。

示例：

```
# bpftrace -e 'uprobe:/bin/bash:readline { printf("PS1: %s\n", str(*uaddr("ps1_prompt"))); }'
Attaching 1 probe...
PS1: \[\e[34;1m\]\u@\h:\w>\[\e[0m\]
PS1: \[\e[34;1m\]\u@\h:\w>\[\e[0m\]
^C
```



这是打印/bin/bash中的`ps1_prompt`字符串，只要有一个`readline()` 函数被执行。

## 10. `reg()`：登记册

语法：`tracepoint:name`

示例：

```
# bpftrace -e 'kprobe:tcp_sendmsg { @[ksym(reg("ip"))] = count(); }'
Attaching 1 probe...
^C

@[tcp_sendmsg]: 7
```



有关寄存器名称列表，请参见src/arch/x86_64.cpp。

## 11. `system()`：系统

语法：`tracepoint:name`

这将在shell中运行所提供的命令。举例来说：

```
# bpftrace --unsafe -e 'kprobe:do_nanosleep { system("ps -p %d\n", pid); }'
Attaching 1 probe...
  PID TTY          TIME CMD
 1339 ?        00:00:15 iscsid
  PID TTY          TIME CMD
 1339 ?        00:00:15 iscsid
  PID TTY          TIME CMD
 1518 ?        00:01:07 irqbalance
  PID TTY          TIME CMD
 1339 ?        00:00:15 iscsid
^C
```



这对于在发生检测事件时执行命令或shell脚本非常有用。

请注意，这是一个不安全的函数。要使用它，bpftrace必须使用`--unsafe`运行。

## 12. `exit()`：退出

语法：`tracepoint:name`

这将退出bpftrace，并且可以与间隔探测器结合使用，以记录某个 持续时间。范例：

```
# bpftrace -e 'kprobe:do_sys_open { @opens = count(); } interval:s:1 { exit(); }'
Attaching 2 probes...
@opens: 119
```



## 13. `cgroupid()`：解析cgroup ID

语法：`tracepoint:name`

这将返回特定cgroup的cgroup ID，并且可以与`cgroup`内置过滤器结合使用 属于特定cgroup的任务，例如：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_openat /cgroup == cgroupid("/sys/fs/cgroup/unified/mycg")/
    { printf("%s\n", str(args.filename)); }':
Attaching 1 probe...
/etc/ld.so.cache
/lib64/libc.so.6
/usr/lib/locale/locale-archive
/etc/shadow
^C
```



在其他终端：

```
# echo $$ > /sys/fs/cgroup/unified/mycg/cgroup.procs
# cat /etc/shadow
```



## 14. `ntop()`：将IP地址数据转换为文本

语法：`tracepoint:name`

这将返回IPv4或IPv6地址的字符串表示形式。ntop将推断地址类型（IPv4 或IPv6）。如果给定整数或`addr`，则ntop假定为IPv4，如果 `char[4]`给出，ntop假设IPv6。也可以将地址类型作为第一个 参数

示例：

下面是一个简单的ntop示例，使用ipv4十六进制编码的文字：

```
bpftrace -e 'BEGIN { printf("%s\n", ntop(0x0100007f));}'
127.0.0.1
^C
```



与前面的示例相同，但将地址类型显式传递给ntop：

```
bpftrace -e '#include <linux/socket.h>
BEGIN { printf("%s\n", ntop(AF_INET, 0x0100007f));}'
127.0.0.1
^C
```



此用法的一个不太简单的示例，跟踪tcp状态更改，并打印目标IPv6 联系地址：

```
bpftrace -e 'tracepoint:tcp:tcp_set_state { printf("%s\n", ntop(args.daddr_v6)) }'
Attaching 1 probe...
::ffff:216.58.194.164
::ffff:216.58.194.164
::ffff:216.58.194.164
::ffff:216.58.194.164
::ffff:216.58.194.164
^C
```



并在另一个终端中发起到这个（或任何）地址的连接：

```
curl www.google.com
```



## 15. `kstack()`：堆栈跟踪，内核

语法：`tracepoint:name`

这些都是使用BPF堆栈映射实现的。

示例：

```
# bpftrace -e 'kprobe:ip_output { @[kstack()] = count(); }'
Attaching 1 probe...
[...]
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    tcp_release_cb+225
    release_sock+64
    tcp_sendmsg+49
    sock_sendmsg+48
    sock_write_iter+135
    __vfs_write+247
    vfs_write+179
    sys_write+82
    entry_SYSCALL_64_fastpath+30
]: 1708
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    __tcp_push_pending_frames+45
    tcp_sendmsg_locked+2637
    tcp_sendmsg+39
    sock_sendmsg+48
    sock_write_iter+135
    __vfs_write+247
    vfs_write+179
    sys_write+82
    entry_SYSCALL_64_fastpath+30
]: 9048
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
    tcp_tasklet_func+348
    tasklet_action+241
    __do_softirq+239
    irq_exit+174
    do_IRQ+74
    ret_from_intr+0
    cpuidle_enter_state+159
    do_idle+389
    cpu_startup_entry+111
    start_secondary+398
    secondary_startup_64+165
]: 11430
```

仅从堆栈中采样三帧（限制= 3）：

```
# bpftrace -e 'kprobe:ip_output { @[kstack(3)] = count(); }'
Attaching 1 probe...
[...]
@[
    ip_output+1
    tcp_transmit_skb+1308
    tcp_write_xmit+482
]: 22186
```

您也可以选择不同的输出格式。可用的格式有`bpftrace`、`perf`和`raw`（无符号化）：

```
# bpftrace -e 'kprobe:do_mmap { @[kstack(perf)] = count(); }'
Attaching 1 probe...
[...]
@[
	ffffffffb4019501 do_mmap+1
	ffffffffb401700a sys_mmap_pgoff+266
	ffffffffb3e334eb sys_mmap+27
	ffffffffb3e03ae3 do_syscall_64+115
	ffffffffb4800081 entry_SYSCALL_64_after_hwframe+61

]: 22186
```

```
# bpftrace -e 'kprobe:do_mmap { @[kstack(raw)] = count(); }'
Attaching 1 probe...
[...]
@[
	ffffffffb4019501
	ffffffffb401700a
	ffffffffb3e334eb
	ffffffffb3e03ae3
	ffffffffb4800081

]: 22186
```

也可以使用不同的输出格式并限制帧数：

```
# bpftrace -e 'kprobe:do_mmap { @[kstack(perf, 3)] = count(); }'
Attaching 1 probe...
[...]
@[
	ffffffffb4019501 do_mmap+1
	ffffffffb401700a sys_mmap_pgoff+266
	ffffffffb3e334eb sys_mmap+27

]: 22186
```

## 16. `ustack()`：堆栈跟踪，用户

语法：`tracepoint:name`

这些都是使用BPF堆栈映射实现的。

示例：

```
# bpftrace -e 'kprobe:do_sys_open /comm == "bash"/ { @[ustack()] = count(); }'
Attaching 1 probe...
^C

@[
    __open_nocancel+65
    command_word_completion_function+3604
    rl_completion_matches+370
    bash_default_completion+540
    attempt_shell_completion+2092
    gen_completion_matches+82
    rl_complete_internal+288
    rl_complete+145
    _rl_dispatch_subseq+647
    _rl_dispatch+44
    readline_internal_char+479
    readline_internal_charloop+22
    readline_internal+23
    readline+91
    yy_readline_get+152
    yy_readline_get+429
    yy_getc+13
    shell_getc+469
    read_token+251
    yylex+192
    yyparse+777
    parse_command+126
    read_command+207
    reader_loop+391
    main+2409
    __libc_start_main+231
    0x61ce258d4c544155
]: 9
@[
    __open_nocancel+65
    command_word_completion_function+3604
    rl_completion_matches+370
    bash_default_completion+540
    attempt_shell_completion+2092
    gen_completion_matches+82
    rl_complete_internal+288
    rl_complete+89
    _rl_dispatch_subseq+647
    _rl_dispatch+44
    readline_internal_char+479
    readline_internal_charloop+22
    readline_internal+23
    readline+91
    yy_readline_get+152
    yy_readline_get+429
    yy_getc+13
    shell_getc+469
    read_token+251
    yylex+192
    yyparse+777
    parse_command+126
    read_command+207
    reader_loop+391
    main+2409
    __libc_start_main+231
    0x61ce258d4c544155
]: 18
```

仅从堆栈中采样六帧（限制= 6）：

```
# bpftrace -e 'kprobe:do_sys_open /comm == "bash"/ { @[ustack(6)] = count(); }'
Attaching 1 probe...
^C

@[
    __open_nocancel+65
    command_word_completion_function+3604
    rl_completion_matches+370
    bash_default_completion+540
    attempt_shell_completion+2092
    gen_completion_matches+82
]: 27
```

您也可以选择不同的输出格式。可用的格式有`bpftrace`、`perf`和`raw`（无符号化）：

```
# bpftrace -e 'uprobe:bash:readline { printf("%s\n", ustack(perf)); }'
Attaching 1 probe...

	5649feec4090 readline+0 (/home/mmarchini/bash/bash/bash)
	5649fee2bfa6 yy_readline_get+451 (/home/mmarchini/bash/bash/bash)
	5649fee2bdc6 yy_getc+13 (/home/mmarchini/bash/bash/bash)
	5649fee2cd36 shell_getc+469 (/home/mmarchini/bash/bash/bash)
	5649fee2e527 read_token+251 (/home/mmarchini/bash/bash/bash)
	5649fee2d9e2 yylex+192 (/home/mmarchini/bash/bash/bash)
	5649fee286fd yyparse+777 (/home/mmarchini/bash/bash/bash)
	5649fee27dd6 parse_command+54 (/home/mmarchini/bash/bash/bash)
```

也可以使用不同的输出格式并限制帧数：

```
# bpftrace -e 'uprobe:bash:readline { printf("%s\n", ustack(perf, 3)); }'
Attaching 1 probe...

	5649feec4090 readline+0 (/home/mmarchini/bash/bash/bash)
	5649fee2bfa6 yy_readline_get+451 (/home/mmarchini/bash/bash/bash)
	5649fee2bdc6 yy_getc+13 (/home/mmarchini/bash/bash/bash)
```

```
# bpftrace -e 'uprobe:bash:readline { printf("%s\n", ustack(raw, 3)); }'
Attaching 1 probe...

	5649feec4090
	5649fee2bfa6
	5649fee2bdc6
```

请注意，要使这些示例正常工作，必须使用帧指针重新编译bash。

## 17. `cat()`：打印文件内容

语法：`tracepoint:name`

这将打印文件内容。举例来说：

```
# bpftrace -e 't:syscalls:sys_enter_execve { printf("%s ", str(args.filename)); cat("/proc/loadavg"); }'
Attaching 1 probe...
/usr/libexec/grepconf.sh 3.18 2.90 2.94 2/977 30138
/usr/bin/grep 3.18 2.90 2.94 4/978 30139
/usr/bin/flatpak 3.18 2.90 2.94 2/980 30143
/usr/bin/grep 3.18 2.90 2.94 3/977 30144
/usr/bin/sed 3.18 2.90 2.94 7/978 30146
/usr/bin/tclsh 3.18 2.90 2.94 5/978 30150
/usr/bin/manpath 3.18 2.90 2.94 2/978 30152
/bin/ps 3.18 2.90 2.94 2/979 30155
^C
```

`cat()`内建也支持格式字符串作为参数：

```
./bpftrace -e 'tracepoint:syscalls:sys_enter_sendmsg { printf("%s => ", comm);
    cat("/proc/%d/cmdline", pid); printf("\n") }'
Attaching 1 probe...
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
Gecko_IOThread => /usr/lib64/firefox/firefox
^C
```



## 18. `signal()`：向当前任务发送信号

语法：

- `signal(u32 signal)`
- `signal("SIG")`

内核：5.3

支持的探头类型：

- k（ret）探针
- u（ret）探针
- 超声波时差
- 轮廓

`signal`向当前任务发送指定信号：

```
# bpftrace  -e 'kprobe:__x64_sys_execve /comm == "bash"/ { signal(5); }' --unsafe
$ ls
Trace/breakpoint trap (core dumped)
```



也可以使用名称指定信号，类似于`kill(1)`命令：

```
# bpftrace -e 'k:f { signal("KILL"); }'
# bpftrace -e 'k:f { signal("SIGINT"); }'
```



## 19. `strncmp()`：比较两个字符串的前n个字符

语法：`tracepoint:name`

如果`length`和`s1`中的前`s2`个字符相等，则返回零，否则返回非零。

示例：

```
bpftrace -e 't:syscalls:sys_enter_* /strncmp("mpv", comm, 3) == 0/ { @[comm, probe] = count() }'
Attaching 320 probes...
[...]
@[mpv/vo, tracepoint:syscalls:sys_enter_rt_sigaction]: 238
@[mpv:gdrv0, tracepoint:syscalls:sys_enter_futex]: 680
@[mpv/ao, tracepoint:syscalls:sys_enter_write]: 1022
@[mpv, tracepoint:syscalls:sys_enter_ioctl]: 2677
@[mpv:cs0, tracepoint:syscalls:sys_enter_ioctl]: 2889
@[mpv/vo, tracepoint:syscalls:sys_enter_read]: 2993
@[mpv/demux, tracepoint:syscalls:sys_enter_futex]: 4745
@[mpv, tracepoint:syscalls:sys_enter_write]: 6936
@[mpv/vo, tracepoint:syscalls:sys_enter_futex]: 7662
@[mpv:cs0, tracepoint:syscalls:sys_enter_futex]: 8127
@[mpv/lua script , tracepoint:syscalls:sys_enter_futex]: 10150
@[mpv/vo, tracepoint:syscalls:sys_enter_poll]: 10241
@[mpv/vo, tracepoint:syscalls:sys_enter_recvmsg]: 15018
@[mpv, tracepoint:syscalls:sys_enter_getpid]: 31178
@[mpv, tracepoint:syscalls:sys_enter_futex]: 403868
```



## 20. `strcontains()`：比较弦草堆是否包含弦针。

语法：`tracepoint:name`

如果字符串干草堆包含字符串针，则返回true，否则返回零。

示例：

```
bpftrace -e 't:syscalls:sys_enter_execve /strcontains(str(args.filename),"bin")/ { @[comm, str(args.filename)] = count(); }'  
Attaching 1 probe...

@[sh, /usr/bin/which]: 2
@[sh, /home/liuting.0xffff/.vscode-server/bin/3b889b090b5ad5793f524b5]: 2
@[cpuUsage.sh, /usr/bin/sleep]: 2
@[sh, /usr/bin/ps]: 2
@[cpuUsage.sh, /usr/bin/sed]: 4
@[node, /bin/sh]: 6
@[cpuUsage.sh, /usr/bin/cat]: 12
```



## 21. `override()`：覆盖返回值

语法：`tracepoint:name`

内核：4.16

支持的探头类型：Kprobes

被探测的函数将不会被执行，而是执行一个帮助器 它只会返回`rc`。

```
# bpftrace -e 'k:__x64_sys_getuid /comm == "id"/ { override(2<<21); }' --unsafe -c id
uid=4194304 gid=0(root) euid=0(root) groups=0(root)
```



此特性仅适用于使用`CONFIG_BPF_KPROBE_OVERRIDE`编译的内核 它只适用于标记为`ALLOW_ERROR_INJECTION`的函数。

bpftrace不测试是否允许对探测的 函数，而不是如果将程序加载到内核中失败：

```
ioctl(PERF_EVENT_IOC_SET_BPF): Invalid argument
Error attaching probe: 'kprobe:vfs_read'
```



## 22. `buf()`：缓冲液

语法：`tracepoint:name`

返回由`d`指向的十六进制格式的数据字符串，该字符串可以安全打印。因为 如果不能总是推断缓冲区的长度，则可以将`length`参数提供给 限制读取的字节数。默认情况下，最大字节数为64，但这可以 使用[`BPFTRACE_STRLEN`](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#91-bpftrace_strlen)环境变量进行优化。

值=32和=126的字节使用其ASCII字符打印，其他 字节以十六进制形式打印（例如，`\x00`）。

例如，我们可以取`buff`的`void *`参数（`sys_enter_sendto`），读取 由`len`（`size_t`）指定的字节数，并将字节格式化为十六进制，以便 它们不会破坏终端显示。结果字符串可以作为参数提供给 printf（）使用`%r`格式说明符：

```
# bpftrace -e 'tracepoint:syscalls:sys_enter_sendto
    { printf("Datagram bytes: %r\n", buf(args.buff, args.len)); }' -c 'ping 8.8.8.8 -c1'
Attaching 1 probe...
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
Datagram bytes: \x08\x00+\xb9\x06b\x00\x01Aen^\x00\x00\x00\x00KM\x0c\x00\x00\x00\x00\x00\x10\x11
\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&'()*+,-./01234567
64 bytes from 8.8.8.8: icmp_seq=1 ttl=52 time=19.4 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 19.426/19.426/19.426/0.000 ms
```



## 23. `sizeof()`：类型或表达式的大小

语法：

- `sizeof(TYPE)`
- `sizeof(EXPRESSION)`

返回参数的字节大小。类似于C/C++ `sizeof`运算符。注记 表达式不会被求值。

示例：

```
# bpftrace -e 'struct Foo { int x; char c; } BEGIN { printf("%d\n", sizeof(struct Foo)); }'
Attaching 1 probe...
8

# bpftrace -e 'struct Foo { int x; char c; } BEGIN { printf("%d\n", sizeof(((struct Foo*)0)->c)); }'
Attaching 1 probe...
1

# bpftrace -e 'BEGIN { printf("%d\n", sizeof(1 == 1)); }'
Attaching 1 probe...
8

# bpftrace -e 'BEGIN { printf("%d\n", sizeof(struct task_struct)); }'
Attaching 1 probe...
13120

# bpftrace -e 'BEGIN { $x = 3; printf("%d\n", sizeof($x)); }'
Attaching 1 probe...
8
```



## 24. `print()`：打印值

语法：`tracepoint:name`

`print()`函数可以使用默认格式打印非映射值。

例如，可以打印局部变量和大多数内置变量：

```
# bpftrace -e 'BEGIN { $t = (1, "string"); print(123); print($t); print(comm) }'
Attaching 1 probe...
123
(1, string)
bpftrace
^C
```



请务必注意，打印值与打印贴图不同。 打印映射和打印值都是异步的：内核会将 事件，但一段时间后它在用户空间中处理。对于值，事件 包含memcopy的值，因此在`print()`调用时的值将为 印刷的 但是，对于映射，只有映射的句柄排队，因此 打印的映射可以与`print()`调用处的映射不同。

## 25. `strftime()`：格式化的时间戳

语法：

- `strftime(const char *format, int nsecs)`

这将返回一个格式化的时间戳，该时间戳可以用`printf`打印。格式 字符串必须由`strftime(3)`支持。`nsecs`是自靴子以来的纳秒， 典型地源自[NSECS](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#6-nsecs-timestamps-and-time-deltas)。

打印返回值时，请使用格式说明符“%s”。注意`strftime` 在BPF（内核）中实际上并不返回字符串，格式化发生在 用户空间。

bpftrace还支持以下格式字符串扩展名：

| 说明符  | 项目名称                   |
| ------- | -------------------------- |
| `int32` | 微秒作为十进制数，左侧补零 |

示例：

```
# bpftrace -e 'i:s:1 { printf("%s\n", strftime("%H:%M:%S", nsecs)); }'
Attaching 1 probe...
13:11:22
13:11:23
13:11:24
13:11:25
13:11:26
^C

# bpftrace -e 'i:s:1 { printf("%s\n", strftime("%H:%M:%S:%f", nsecs)); }'
Attaching 1 probe...
15:22:24:104033
^C
```



## 26. `path()`：返回完整路径

语法：

- `path(struct path *path)`

返回参数中结构路径指针引用的完整路径。 有一个允许的内核函数列表，可以使用这个 探针中的辅助工具。

示例：

```
# bpftrace  -e 'kfunc:filp_close { printf("%s\n", path(args.filp->f_path)); }'
Attaching 1 probe...
/proc/sys/net/ipv6/conf/eno2/disable_ipv6
/proc/sys/net/ipv6/conf/eno2/use_tempaddr
socket:[23276]
/proc/sys/net/ipv6/conf/eno2/disable_ipv6
socket:[17655]
/sys/devices/pci0000:00/0000:00:1c.5/0000:04:00.1/net/eno2/type
socket:[38745]
/proc/sys/net/ipv6/conf/eno2/disable_ipv6

# bpftrace  -e 'kretfunc:dentry_open { printf("%s\n", path(retval->f_path)); }'
Attaching 1 probe...
/dev/pts/1 -> /dev/pts/1
```



## 27. `uptr()`：注释用户空间指针

语法：

- `uptr(void *p)`

注释`p`作为属于用户空间地址空间的指针。

bpftrace通常可以推断指针的地址空间。然而，有 推理失败的极端情况。例如，处理 使用用户空间指针（如`const char __user *p`的参数）。在这些 如果你不知道，你需要对指针进行注释。

示例：

```
# bpftrace -e 'kprobe:do_sys_open { printf("%s\n", str(uptr(arg1))) }'
Attaching 1 probe...
.
state
^C
```



## 28. `kptr()`：注释内核空间指针

语法：

- `kptr(void *p)`

注释`p`作为属于内核地址空间的指针。

就像`uptr`一样，通常只有在bpftrace推断出 指针地址空间不正确。

## 29. `macaddr()`：将MAC地址数据转换为文本

语法：`tracepoint:name`

这将返回MAC地址的规范字符串表示形式。

范例：

```
# bpftrace -e 'kprobe:arp_create { printf("SRC %s, DST %s\n", macaddr(sarg0), macaddr(sarg1)); }'
SRC 18:C0:4D:08:2E:BB, DST 74:83:C2:7F:8C:FF
^C
```



## 30. `cgroup_path`：将cgroup id转换为cgroup路径

语法：`tracepoint:name`

将给定的cgroup id转换为每个cgroup层次结构的相应cgroup路径 出现在。由于转换是在用户空间中完成的，因此生成的对象只能用于 印刷。

可选地，可以将字符串文字作为第二个参数传递，以筛选cgroup层次结构以查找 in（解释为通配符表达式）。

范例：

```
# bpftrace -e 'BEGIN { print(cgroup_path(5386)); }'
Attaching 1 probe...
unified:/user.slice/user-1000.slice/session-3.scope
```



## 31. `bswap`：反向字节顺序

语法：`tracepoint:name`

反转整数`n`中字节的顺序。在8位整数的情况下，返回`n`而不被修改。 返回类型是一个与`n`相同宽度的无符号整数。

范例：

```
# bpftrace -e 'BEGIN { $i = (uint32)0x12345678; printf("Reversing byte order of 0x%x ==> 0x%x\n", $i, bswap($i)); }'
Attaching 1 probe...
Reversing byte order of 0x12345678 ==> 0x78563412
```



## 32. `skb_output`：写 `skb` 的数据节插入到PCAP文件中

语法：`tracepoint:name`

将sk_buff `skb`的数据段写入`path`中的PCAP文件，从`offset`到`offset` +`length`。

PCAP文件封装在RAW IP中，因此不包括以太网头。 结构`data`中的`skb`部分在某些内核环境中可能包含以太网头，您可以将`offset`设置为14字节以排除以太网头。

每个数据包的时间戳是通过添加`nsecs`和靴子时间来确定的，精度在不同的内核上有所不同，参见`nsecs`。

此函数在成功时返回0，或在失败时返回负错误。

环境变量`BPFTRACE_PERF_RB_PAGES`应该增加，以便捕获大数据包，否则这些数据包将被丢弃。

范例：

```
# cat dump.bt
kfunc:napi_gro_receive {
$ret = skboutput("receive.pcap", args.skb, args.skb->len, 0);
}

kfunc:dev_queue_xmit {
// setting offset to 14, to exclude ethernet header
$ret = skboutput("output.pcap", args.skb, args.skb->len, 14);
printf("skboutput returns %d\n", $ret);
}

# export BPFTRACE_PERF_RB_PAGES=1024
# bpftrace dump.bt
...

# tcpdump -n -r ./receive.pcap  | head -3
reading from file ./receive.pcap, link-type RAW (Raw IP)
dropped privs to tcpdump
10:23:44.674087 IP 22.128.74.231.63175 > 192.168.0.23.22: Flags [.], ack 3513221061, win 14009, options [nop,nop,TS val 721277750 ecr 3115333619], length 0
10:23:45.823194 IP 100.101.2.146.53 > 192.168.0.23.46619: 17273 0/1/0 (130)
10:23:45.823229 IP 100.101.2.146.53 > 192.168.0.23.46158: 45799 1/0/0 A 100.100.45.106 (60)
```



## 33. `pton()`：将文本IP地址转换为字节数组

语法：`tracepoint:name`

这会将IPv4或IPv6地址的文本表示转换为字节数组。 `pton`根据给定参数中的`.`或`:`推断地址族。 当我们需要选择具有特定IP地址的数据包时，`pton`会派上用场。

示例：

```
# bpftrace -e 'tracepoint:tcp:tcp_retransmit_skb {
  if (args.daddr_v6[0] == pton("::1")[0]) {
    printf("first octet matched\n");
  }
}'
Attaching 1 probe...
first octet matched
^C

# bpftrace -e 'tracepoint:tcp:tcp_retransmit_skb {
  if (args.daddr[0] == pton("127.0.0.1")[0]) {
    printf("first octet matched\n");
  }
}'
Attaching 1 probe...
first octet matched
^C
```



## 34. `strerror`：获取errno代码的错误消息

语法：`tracepoint:name`

将给定的errno代码转换为描述错误的字符串。结果只能用于打印，因为转换是在用户空间中完成的。

范例：

```
# bpftrace -e '#include <errno.h>
BEGIN { print(strerror(EPERM)); }'
Attaching 1 probe...
Operation not permitted
```



## 35. `offsetof`：结构中元素的偏移

语法：

- `offsetof(struct, element)`
- `offsetof(expression, element)`

获取结构中元素的偏移量。

示例：

```
#!/usr/bin/env bpftrace

BEGIN
{
	printf("Offset of flags: %ld\n", offsetof(struct task_struct, flags));
	printf("Offset of comm: %ld\n", offsetof(*curtask, comm));
	exit();
}
```



```
Attaching 1 probe...
Offset of flags: 44
Offset of comm: 3216
```



## 36. `nsecs()`：时间戳和时间增量

语法：`tracepoint:name`

获取指定时钟的纳秒。当前支持三种时间类型，即

- 靴子：nsecs（）或nsecs（boot）用于获取系统启动后的纳秒
- tai：nsecs（tai）用于通过bpf_ktime_get_tai_ns辅助函数获取CLOCK_TAI的纳秒
- sw_tai：nsecs（sw_tai）是为了获得纳秒级的CLOCK_TAI，但它是通过“三重vdso三明治”方法获得的

示例：

```
# cat -n example.bt
#!/usr/bin/env bpftrace
i:s:1 {
        $sw_tai1 = nsecs(sw_tai);
        $tai = nsecs(tai);
        $sw_tai2 = nsecs(sw_tai);

        printf("sw_tai precision: %lldns\n", ($sw_tai1 + $sw_tai2)/2 - $tai);
}
```



运行example.bt：

```
Attaching 1 probe...
sw_tai precision: -98ns
sw_tai precision: -92ns
sw_tai precision: -99ns
sw_tai precision: -99ns
```



# map内置函数

映射是一种特殊的BPF数据类型，可用于存储计数、统计信息和直方图。它们是 也用于上一节讨论的某些变量类型，只要使用`@`： [全局变量](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#21-global)、[每线程变量](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#22-per-thread)和[关联变量 ](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#3--associative-arrays)数组

当bpftrace退出时，将打印所有映射。例如（第1节中介绍了`count()`函数 如下）：

```sh
# bpftrace -e 'kprobe:vfs_read { @[comm] = count(); }'
Attaching 1 probe...
^C

@[systemd]: 6
@[vi]: 7
@[sshd]: 16
@[snmpd]: 321
@[snmp-pass]: 374
```

在Ctrl-C结束程序后打印地图。如果您使用的地图不希望 退出时自动打印，您可以添加一个END块来清除贴图。举例来说：

```
END
{
	clear(@start);
}
```

## 1.建筑物

- `count()` -计算调用此函数的次数
- `sum(int n)` -求和
- `avg(int n)` -平均值
- `min(int n)` -记录观察到的最小值
- `max(int n)` -记录观察到的最大值
- `stats(int n)` -返回此值的计数、平均值和总计
- `hist(int n)` -生成n值的log 2直方图
- `lhist(int n, int min, int max, int step)` -生成n值的线性直方图
- `delete(@x[key])` -删除作为参数传入的map元素
- `print(@x[, top [, div]])` -打印地图，可选地仅打印顶部条目并带有除数
- `print(value)` -打印值
- `clear(@x)` -从贴图中删除所有关键点
- `zero(@x)` -将所有映射值设置为零

其中一些是异步的：内核将事件排队，但在一段时间（毫秒）之后 在用户空间中处理。异步操作包括：`print()`在地图上，`clear()`和`zero()`。

## 2. `count()`：计数

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @reads = count();  }'
Attaching 1 probe...
^C

@reads: 119
```

这表明在跟踪时有119个对vfs_read（）的调用。

下一个示例包含`comm`变量作为键，以便按每个进程分解该值 名字。例如，`@reads[comm]`：

```
# bpftrace -e 'kprobe:vfs_read { @reads[comm] = count(); }'
Attaching 1 probe...
^C

@reads[sleep]: 4
@reads[bash]: 5
@reads[ls]: 7
@reads[snmp-pass]: 8
@reads[snmpd]: 14
@reads[sshd]: 14
```

## 3. `sum()`：总和

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @bytes[comm] = sum(arg2); }'
Attaching 1 probe...
^C

@bytes[bash]: 7
@bytes[sleep]: 4160
@bytes[ls]: 6208
@bytes[snmpd]: 20480
@bytes[snmp-pass]: 65536
@bytes[sshd]: 262144
```

也就是通过vfs_read（）内核函数对请求的字节求和，该函数是两个可能的条目之一 读取系统调用的点数。要查看实际读取的字节：

```
# bpftrace -e 'kretprobe:vfs_read /retval > 0/ { @bytes[comm] = sum(retval); }'
Attaching 1 probe...
^C

@bytes[bash]: 5
@bytes[sshd]: 1135
@bytes[systemd-journal]: 1699
@bytes[sleep]: 2496
@bytes[ls]: 4583
@bytes[snmpd]: 35549
@bytes[snmp-pass]: 55681
```

现在使用一个过滤器来确保返回值在用于sum（）之前是正的。回归 值在错误的情况下可能为负，与其他函数一样。无论何时都要记住这一点 在retval上使用sum（）。

## 4. `avg()`平均值

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @bytes[comm] = avg(arg2); }'
Attaching 1 probe...
^C

@bytes[bash]: 1
@bytes[sleep]: 832
@bytes[ls]: 886
@bytes[snmpd]: 1706
@bytes[snmp-pass]: 8192
@bytes[sshd]: 16384
```

这是对请求的读取大小求平均值。

## 5. `min()`：最小值

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @bytes[comm] = min(arg2); }'
Attaching 1 probe...
^C

@bytes[bash]: 1
@bytes[systemd-journal]: 8
@bytes[snmpd]: 64
@bytes[ls]: 832
@bytes[sleep]: 832
@bytes[snmp-pass]: 8192
@bytes[sshd]: 16384
```

这显示了所见的最小值。

## 6. `max()`：最大值

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @bytes[comm] = max(arg2); }'
Attaching 1 probe...
^C

@bytes[bash]: 1
@bytes[systemd-journal]: 8
@bytes[sleep]: 832
@bytes[ls]: 1024
@bytes[snmpd]: 4096
@bytes[snmp-pass]: 8192
@bytes[sshd]: 16384
```

这显示了所见的最大值。

## 7. `stats()`：统计

语法：`tracepoint:name`

这是使用BPF映射实现的。

例如，`@start`：

```
# bpftrace -e 'kprobe:vfs_read { @bytes[comm] = stats(arg2); }'
Attaching 1 probe...
^C

@bytes[bash]: count 7, average 1, total 7
@bytes[sleep]: count 5, average 832, total 4160
@bytes[ls]: count 7, average 886, total 6208
@bytes[snmpd]: count 18, average 1706, total 30718
@bytes[snmp-pass]: count 12, average 8192, total 98304
@bytes[sshd]: count 15, average 16384, total 245760
```

stats（）函数返回三个统计信息：事件计数、参数值的平均值 和参数值的总和。这类似于使用count（）、avg（）和sum（）。

## 8. `hist()`：Log2直方图

语法：

```
@histogram_name[optional_key] = hist(value)
```

这是使用BPF映射实现的。

示例：

### 8.1. 2的幂：

```
# bpftrace -e 'kretprobe:vfs_read { @bytes = hist(retval); }'
Attaching 1 probe...
^C

@bytes:
(..., 0)             117 |@@@@@@@@@@@@                                        |
[0]                    5 |                                                    |
[1]                  325 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                  |
[2, 4)                 6 |                                                    |
[4, 8)                 3 |                                                    |
[8, 16)              495 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[16, 32)              35 |@@@                                                 |
[32, 64)              25 |@@                                                  |
[64, 128)             21 |@@                                                  |
[128, 256)             1 |                                                    |
[256, 512)             3 |                                                    |
[512, 1K)              2 |                                                    |
[1K, 2K)               1 |                                                    |
[2K, 4K)               2 |                                                    |
```

### 8.2. 2的幂按键：

```
# bpftrace -e 'kretprobe:do_sys_open { @bytes[comm] = hist(retval); }'
Attaching 1 probe... ^C

@bytes[snmp-pass]:
[4, 8)                 6 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

@bytes[ls]:
[2, 4)                 9 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

@bytes[snmpd]:
[1]                    1 |@@@@                                                |
[2, 4)                 0 |                                                    |
[4, 8)                 0 |                                                    |
[8, 16)                4 |@@@@@@@@@@@@@@@@@@                                  |
[16, 32)              11 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

@bytes[irqbalance]:
(..., 0)              15 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@               |
[0]                    0 |                                                    |
[1]                    0 |                                                    |
[2, 4)                 0 |                                                    |
[4, 8)                21 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
```

## 9. `lhist()`：线性直方图

语法：

```
@histogram_name[optional_key] = lhist(value, min, max, step)
```

这是使用BPF映射实现的。`min`必须为非负数。

示例：

```
# bpftrace -e 'kretprobe:vfs_read { @bytes = lhist(retval, 0, 10000, 1000); }'
Attaching 1 probe...
^C

@bytes:
[0, 1000)            480 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[1000, 2000)          49 |@@@@@                                               |
[2000, 3000)          12 |@                                                   |
[3000, 4000)          39 |@@@@                                                |
[4000, 5000)         267 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@                        |
```

## 10. `print()`：打印地图

语法：`tracepoint:name`

`print()`函数将打印地图，类似于bpftrace结束时的自动打印。两个 可以提供可选参数：顶部编号，使得仅打印顶部编号的条目，以及 一个除数，用来划分值。几个例子将解释它们的用法。

作为top的示例，跟踪`vfs`操作并打印top 5：

```
# bpftrace -e 'kprobe:vfs_* { @[func] = count(); } END { print(@, 5); clear(@); }'
Attaching 54 probes...
^C
@[vfs_getattr]: 91
@[vfs_getattr_nosec]: 92
@[vfs_statx_fd]: 135
@[vfs_open]: 188
@[vfs_read]: 405
```

最后的`clear()`用于防止在退出时自动打印地图。

作为除数的一个示例，在vfs_read（）中按进程名称将总时间求和为毫秒：

```
# bpftrace -e 'kprobe:vfs_read { @start[tid] = nsecs; }
    kretprobe:vfs_read /@start[tid]/ {@ms[pid] = sum(nsecs - @start[tid]); delete(@start[tid]); }
    END { print(@ms, 0, 1000000); clear(@ms); clear(@start); }'
```

这一行代码将vfs_read（）持续时间的总和表示为纳秒，然后除以毫秒 打印时。如果没有这种能力，在求和时是否应该尝试除以毫秒（例如， `sum((nsecs - @start[tid]) / 1000000)`），该值通常会被舍入为零，而不是累积为 应该是的

请注意，打印映射与打印值不同。见解释 在`print()`中[：打印值](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#23-print-print-value)。

# 产出

## 1. `printf()`：每事件输出

语法：`tracepoint:name`

每个事件的详细信息可以使用`print()`打印。

示例：

```
# bpftrace -e 'kprobe:do_nanosleep { printf("sleep by %d\n", tid); }'
Attaching 1 probe...
sleep by 3669
sleep by 1396
sleep by 3669
sleep by 1396
[...]
```

## 2. `interval`：间隔输出

语法：`tracepoint:name`

示例：

```
# bpftrace -e 'kprobe:do_sys_open { @opens = @opens + 1; }
    interval:s:1 { printf("opens/sec: %d\n", @opens); @opens = 0; }'
Attaching 2 probes...
opens/sec: 16
opens/sec: 2
opens/sec: 3
opens/sec: 15
opens/sec: 8
opens/sec: 2
^C

@opens: 2
```

## 3. `hist()`, `print()`：直方图打印

声明的直方图在程序终止时自动打印出来。参见第[5节。 声明](https://github.com/iovisor/bpftrace/blob/master/docs/reference_guide.md#5-histograms)的直方图。

示例：

```sh
# bpftrace -e 'kretprobe:vfs_read { @bytes = hist(retval); }'
Attaching 1 probe...
^C

@bytes:
(..., 0)             117 |@@@@@@@@@@@@                                        |
[0]                    5 |                                                    |
[1]                  325 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@                  |
[2, 4)                 6 |                                                    |
[4, 8)                 3 |                                                    |
[8, 16)              495 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
[16, 32)              35 |@@@                                                 |
[32, 64)              25 |@@                                                  |
[64, 128)             21 |@@                                                  |
[128, 256)             1 |                                                    |
[256, 512)             3 |                                                    |
[512, 1K)              2 |                                                    |
[1K, 2K)               1 |                                                    |
[2K, 4K)               2 |                                                    |
```

也可以使用`print()`功能按需打印直方图。例如：

```sh
# bpftrace -e 'kretprobe:vfs_read { @bytes = hist(retval); } interval:s:1 { print(@bytes); clear(@bytes); }'

[...]
```

## 4. `SIGUSR1`：按需输出

在接收到`SIGUSR1`信号时，bpftrace将把所有映射打印到标准输出。

范例：

```sh
# bpftrace -e 'kretprobe:vfs_read { @bytes = hist(retval); }' &
# kill -s USR1 $(pidof bpftrace)
```

# BTF支持

如果内核具有BTF，则内核类型自动可用，并且不需要包含额外的头文件 来利用它们。为了允许用户在脚本中检测这种情况，预处理器宏`BPFTRACE_HAVE_BTF` 如果检测到BTF，则定义。有关其用法的示例，请参见tools/。

使用BTF for vmlinux的要求：

- Linux 4.18+ 

  ```
  CONFIG_DEBUG_INFO_BTF=y
  ```

  - 建筑需要矮人与pahole v1.13+

- bpftrace v0.9.3+，支持BTF（使用libbpf v0.0.4+构建）

对内核模块使用BTF的其他要求：

- Linux 5.11+ 

  ```
  CONFIG_DEBUG_INFO_BTF_MODULES=y
  ```

  - 建筑需要矮人与pahole v1.19+

有关BTF的更多信息，请参阅[内核文档](https://www.kernel.org/doc/html/latest/bpf/btf.html)。

请注意，如果bpftrace程序包含用户定义类型，则BTF类型对该程序不可用 重新定义了一些BTF类型。这里，“用户定义类型”也是通过包含的头引入的类型。 因此，如果在BPFTrace程序中包含内核头，那么它很可能会定义一些 内核类型和BTF对您程序不可用（您必须定义/包含所有必要的 手动类型）。

# 高级工具

bpftrace可以用来创建一些强大的一行程序和一些简单的工具。对于复杂的工具，其中 可能涉及命令行选项、位置参数、参数处理和定制输出， 考虑改用[BCC](https://github.com/iovisor/bcc)。bcc提供Python（和其他）前端， 允许使用所有其他Python库（包括argparse），以及直接控制 内核BPF程序。缺点是bcc编程起来要冗长和费力得多。一起 bpftrace和bcc是免费的。

预期的开发路径是使用bpftrace一行程序进行探索，然后使用ad-hoc脚本 最后，在需要时，使用bcc高级工具。

作为bpftrace与bcc差异的一个示例，bpftrace xfsdist.bt工具也存在于bcc中，作为 xfsdist.py.两者测量相同的功能并产生相同的信息摘要。然而，BCC 版本支持各种参数：

```
# ./xfsdist.py -h
usage: xfsdist.py [-h] [-T] [-m] [-p PID] [interval] [count]

Summarize XFS operation latency

positional arguments:
  interval            output interval, in seconds
  count               number of outputs

optional arguments:
  -h, --help          show this help message and exit
  -T, --notimestamp   don't include timestamp on interval output
  -m, --milliseconds  output in milliseconds
  -p PID, --pid PID   trace this PID only

examples:
    ./xfsdist            # show operation latency as a histogram
    ./xfsdist -p 181     # trace PID 181 only
    ./xfsdist 1 10       # print 1 second summaries, 10 times
    ./xfsdist -m 5       # 5s summaries, milliseconds
```



bcc版本有131行代码。bpftrace版本是22。

# 错误数

## 1.似乎超出了BPF堆栈限制512字节

对许多数据项进行操作的BPF程序可能会达到这个限制。有很多事情你可以尝试 保持在限制范围内：

1. 找到减少程序中使用的数据大小的方法。例如，避免字符串 不必要：使用`pid`而不是`comm`。使用较少的贴图关键点。
2. 将您的程序拆分为多个探测器。
3. 检查Linux中BPF堆栈限制的状态（将来可能会增加，可能作为 可调谐的）。
4. （高级）：运行-d并检查LLVM IR，并寻找优化src/ast/codegen_llvm.cpp的方法。

## 2.未找到内核标头

bpftrace需要某些特性的内核标头，默认情况下在以下位置搜索：

```
/lib/modules/$(uname -r)
```

默认搜索目录可以使用环境变量`BPFTRACE_KERNEL_SOURCE`和 如果是树外Linux内核构建，也是`BPFTRACE_KERNEL_BUILD`。