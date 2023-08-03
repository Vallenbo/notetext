

| 目录名称    | 目录内相应文件内容                                          |
| ----------- | ----------------------------------------------------------- |
| /boot       | 开机所需文件-----内核、开机菜单以及所需配置文件等           |
| /dev        | 以文件形式存放任何设备与接口                                |
| /etc        | 配置文件                                                    |
| /home       | 用户家目录                                                  |
| /bin        | 存放单用户模式下还可以操作的命令                            |
| /lib        | 开机时用到的函数库，以及/bin 与/sbin 下面的命令要调用的函数 |
| /sbin       | 开机过程中需要的命令                                        |
| /media      | 用于挂载设备文件的目录                                      |
| /opt        | 放置第三方的软件                                            |
| /root       | 系统管理员的家目录                                          |
| /srv        | 些网络服务的数据文件目录                                    |
| /tmp        | 任何人均可使用的“共享”临时目录                              |
| /proc       | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等      |
| /usr/local  | 用户自行安的软件                                            |
| /usr/sbin   | Linux系统开机时不会使用到的软件/命令/脚本                   |
| /usr/share  | 帮助与说明文件，也可放置共享文件                            |
| /var        | 主要存放经常变化的文件，如日志                              |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里        |

/etc/services查看系统所有协议

```sh
[root@desktop22]# echo $LANG
en_US.UTF-8
[root@desktop22]# LANG=zh_CN,UTF-8  #修改系统语言
```

echo export EDITOR=vim >> /etc/profile.d/env.sh解决vimu无颜色问题

开机自启文件	/etc/rc.d/rc.local		chmod _+x rc.local添加执行权限	bc 计算器

hostnamectl  set-hostname computer设置永久主机名		whereis ssh查看服务配置文件夹

elinks文本浏览器			startx 切换图形界面			sudo 临时借用管理员权限

uname -a查看系统内核		cat /proc/version查看系统版本	lscpu查看系统cpu信息

pwd 显示当前所在目录		whoami 谁					which mysql查看内部命令

mv 移动，改名字	-b 若目标文件存在则创建备份文件			-f 强制覆盖且不提醒

tree成树状显示当前目录所有文件

mkdir创建目录	-p多级创建		-v提示是否创建成功	 –m 创建同时设置权限值如755

rm  -r (递归删除)  -f（强制删除且不提醒）文件或目录	rmdir只能删除空目录

touch创建新文件，修改文件的创建时间（覆盖文件时间）		touch /linux{1..3}  建立文件多个文件

cp	-r /etc/passwd	递归拷贝文件或目录	-p拷贝权限，文件夹也是文件

cd ：进入该用户的主目录 ~（root用户为/root,其他用户为/home/用户名）

cd ~student 回到student用户的家目录		cd .. ：返回上一级目录（注意要空格）		cd - ：返回上次所在目录

cd / ：返回根目录 （绝对路径）			cd ./目录1/目录2 ：进入当前目录下的子目录（相对路径）

从当前出发的叫相对路径  cd  aa/bb		从根出发的叫绝对路径	cd /tmp/aa/bb

ls -a :列出文件下所有的文件（包括以“.“开头的隐藏文件）	ll -Z  将文件详细属性全部列出来

date查看系统当前时间 	date -s更改系统时间 2007-08-03		cal 查看系统当前日历

du 显示文件或目录占用磁盘空间		-m以MB为单位显示

ctrl + r 搜索最近使用的命令			crtl +a光标回到命令首行				crtl +e光标回到命令末行

ctrl + w 剪切光标处到行尾的字符		ctrl+d 删除光标所在字符

echo "export TMOUT=10" >> /etc/profile	 && bash 登录超时退出

shutdown 对系统执行关机操作	-t +秒数，推迟多少秒执行		-f，重新启动并不执行fsck命令

-h now 现在将系统关机			-r，关机之后重启

nohup command命令 &	表示不间断的在后台运行

ln  /opt/cc /tmp/aa		硬链接，删除其中一个对另一个没有任何影响，内容同步

ln -s 软链接 会产生一个全新的文件

sar	综合工具，查看系统状况

vmstat查看系统状态、硬件和系统信息等	netstat查看网络状况			iptraf实时网络状况监测

tcpdump	抓取网络数据包，详细分析	tcptrace	数据包分析工具	netperf	网络带宽工具

dstat综合工具，综合了vmstat,iostat, ifstat,netstat等多信息

ifstat：监测网络接口的状态

```
-a监测能监测到的所有网络接口状态		-i指定需要监测的网络接口
-t在每一行开头显示时间戳				-T显示所有监测的网络接口的全部宽带
```

lsof：查看进程打开的文件或文件打开的进程，也可用于查看端口是否为打开的状态

如果存在输出，则说明有进程正在使用该目录，需要先结束这些进程对该目录的占用。例如，输出可能类似如下：

```sh
#lsof | grep /mnt/sdb1_mount #查看正在使用文件的进程
COMMAND     PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash      1234        root  cwd    DIR  253,0     4096   12 /mnt/sdb1_mount
```

则可以使用上述命令中显示的PID来结束该进程对目录的占用。

例如，在本例中可以使用以下命令结束该bash进程：

```
kill -9 1234
```

iostat：监控系统输入/输出设备和CPU使用情况，可能需要用到的安装包 sysstat

```
-d：显示磁盘统计信息。			-x 1：显示详细的磁盘统计信息，包括平均等待时间和队列长度等。
-k：以KB为单位显示统计信息。	   -m：以MB为单位显示统计信息。
-t：显示时间戳。
```

time命令统计执行指定命令所花费的总时间

```
-f格式化时间输出		-a将显示信息追加到文件		-o将显示信息写入文件中
```

```
time iostat 统计iostat命令执行所需总时间
```

nc（netcat）是一个网络工具，它可以在命令行下进行网络连接、端口扫描和数据传输等操作。下面是nc命令的一些常见用法和示例：

1. 连接到主机和端口：使用以下命令连接到指定的主机和端口：

   ```sh
   nc example.com 80 #要连接到主机http://example.com/的端口 80
   ```

2. 监听端口：使用以下命令在指定的端口上监听传入的连接：

   ```sh
   nc -l 8080 #要在本地监听端口 8080
   ```

3. 文件传输：使用以下命令从发送方传输文件到接收方：

   ```
   bashCopy Code# 接收方
   nc -l <port> > output.txt
   
   # 发送方
   nc <receiver_host> <receiver_port> < file_to_send.txt
   ```

   例如，接收方在端口 1234 上接收文件，并保存到 output.txt；发送方使用 `nc receiver.example.com 1234 < file_to_send.txt` 将文件发送给接收方。

4. 端口扫描：使用以下命令扫描指定主机的端口范围：

   ```sh
   nc -zv example.com 1-100 #要扫描http://example.com/的端口从 1 到 100 的范围
   ```

这些只是nc命令的一些基本用法示例，还有更多高级用法可以在 `man nc` 命令中查看。请注意，在实际使用中，请确保你遵守当地的法律法规，并且获得了系统管理员或网络拥有者的授权才能进行网络连接和端口扫描等操作。



# CPU工具

## 内核统计：

1、`uptime` 用于查看系统的负载信息,运行时间

```sh
$ uptime
 00:34:10 up 6:29, 1 user, load average：20.29, 18.90, 18.70
```

## 负载平均值：

1、`top`是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器

```sh
top -p 2822(PID号)	#查询固定PID号
$ top
top - 02:25:43 up  6:37,  2 users,  load average：0.09, 0.04, 0.01
Tasks：227 total,   1 running, 226 sleeping,   0 stopped,   0 zombie
%Cpu(s)： 0.0 us,  3.2 sy,  0.0 ni, 96.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem ：  1963.2 total,    761.0 free,    248.9 used,    953.3 buff/cache
MiB Swap：  1788.0 total,   1788.0 free,      0.0 used.   1556.9 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
   4486 root      20   0   10416   3864   3240 R   6.2   0.2   0:00.01 top
      1 root      20   0  167740  12476   8812 S   0.0   0.6   0:01.10 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.02 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
      5 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 slub_flushwq
      6 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 netns
[...]
```

第1行:系统时间、运行时间、登录终端数、系统负载(三个数值分别为1分钟、5分钟、15 分钟内的平均值，数值越小意味着负载越低)。
第2行:进程总数、运行中的进程数、睡眠中的进程数、停止的进程数、僵死的进程数。
第3行:用户占用资源百分比、系统内核占用资源百分比、改变过优先级的进程资源百分比、空闲的资源百分比等。
第4行:物理内存总量、内存使用量、内存空闲量、作为内核缓存的内存量。
第5行:虚拟内存总量、虛拟内存使用量、虛拟内存空闲量、已被提前加载的内存量。

2、`mpstat`（Multiprocessor Statistics）是实时系统监控工具，可查看多处理器状况。

```sh
apt install sysstat -y
```

其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。
在多CPUs系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。
mpstat最大的特点是：可以查看多核心cpu中每个计算核心的统计数据；而类似工具vmstat只能查看系统整体cpu情况。

```sh
root@l:~# mpstat [-P {|ALL}] [internal [count]]
参数 			解释
-P {|ALL} 	表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
internal 	相邻的两次采样的间隔时间、
count 		采样的次数，count只能和delay一起使用
当没有参数时，mpstat则显示系统启动以后所有信息的平均值。有interval时，第一行的信息自系统启动以来的平均信息。从第二行开始，输出为前一个interval时间段的平均信息。

root@l:~# mpstat -P ALL 1  
Linux 5.19.0-46-generic (l)     08/01/2023      _x86_64_        (2 CPU)
02:27:36 AM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
02:27:37 AM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
02:27:37 AM    0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
02:27:37 AM    1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
[...]
```

## 硬件统计

`perf`：性能分析和跟踪工具，能够进行函数级与指令级的热点查找。

```sh
apt install linux-tools-common linux-tools-generic -y #安装包
```

Perf List：利用perf剖析程序性能时，需要指定当前测试的性能时间。性能事件是指在处理器或操作系统中发生的，可能影响到程序性能的硬件事件或软件事件

```sh
Perf top #实时显示系统/进程的性能统计信息
-e：指定性能事件							   -d 选项用于启用perf工具的统计功能
-a：显示在所有CPU上的性能统计信息				  -C：显示在指定CPU上的性能统计信息
-p：指定进程PID								-t：指定线程TID
-K：隐藏内核统计信息							 -U：隐藏用户空间的统计信息
-s：指定待解析的符号信息

perf record：#运行一个命令并记录其性能数据到perf.data文件中。
perf report：#读取perf.data文件（由perf record创建）并显示性能分析报告，展示函数和符号的耗时信息。
perf stat：#运行一个命令并收集性能计数器的统计信息，例如CPU周期、缓存命中率等。
perf top：#显示当前系统的性能概况，包括使用CPU最多的函数和符号。
perf annotate：#读取perf.data文件（由perf record创建）并显示带注释的代码，显示哪些代码路径上发生了性能事件。
perf diff：#比较两个perf.data文件，并显示它们之间的差异性能分析报告。
perf sched：#跟踪和分析调度器的性能信息，包括任务的运行时间和等待时间等。
perf mem：#对内存访问进行性能分析，包括缓存命中率、页表查找等。
perf ftrace：#使用内核的ftrace功能进行跟踪和分析，例如系统调用、中断和函数调用等。
perf iostat：#显示磁盘I/O的性能指标，如读写速度、延迟等。
perf kmem：#跟踪和分析内核内存的使用情况，包括分配和释放的次数、大小等。
perf script：#读取perf.data文件（由perf record创建）并显示事件跟踪的输出记录。
perf test：#运行性能测试和验证perf工具的功能和稳定性。
perf version：#显示perf工具的版本信息。
```

### 示例

```sh
root@l:~#  perf stat -d gzip file1  #使用perf工具来统计gzip命令对文件file1进行压缩的命令。
 Performance counter stats for 'gzip file1':
              0.67 msec task-clock                #    0.551 CPUs utilized
                 0      context-switches          #    0.000 /sec
                 0      cpu-migrations            #    0.000 /sec
                82      page-faults               #  121.574 K/sec
         1,608,532      cycles                    #    2.385 GHz
            41,723      stalled-cycles-frontend   #    2.59% frontend cycles idle
           115,345      stalled-cycles-backend    #    7.17% backend cycles idle
         1,319,809      instructions              #    0.82  insn per cycle
                                                  #    0.09  stalled cycles per insn
           255,678      branches                  #  379.072 M/sec
     <not counted>      branch-misses                                                 (0.00%)
     <not counted>      L1-dcache-loads                                               (0.00%)
     <not counted>      L1-dcache-load-misses                                         (0.00%)
   <not supported>      LLC-loads
   <not supported>      LLC-load-misses

       0.001223328 seconds time elapsed

       0.001264000 seconds user
       0.000000000 seconds sys

Some events weren't counted. Try disabling the NMI watchdog:
        echo 0 > /proc/sys/kernel/nmi_watchdog
        perf stat ...
        echo 1 > /proc/sys/kernel/nmi_watchdog
```

```sh
root@l:~# perf list # 获取当前处理器和perf工具支持的PMC列表
List of pre-defined events (to be used in -e):
  branch-instructions OR branches                    [Hardware event]
  branch-misses                                      [Hardware event]
  cache-misses                                       [Hardware event]
  cache-references                                   [Hardware event]
  cpu-cycles OR cycles                               [Hardware event]
  instructions                                       [Hardware event]
[...]
```

```sh
root@l:~# perf stat -e mem_load_retired.l3_hit -e mem_load_retired.l3_miss -a -I 1000
#使用perf工具在Linux系统上监测内存访问的性能统计信息。
#-e选项用于指定要监测的事件，mem_load_retired.l3_hit表示L3缓存命中的事件,mem_load_retired.l3_miss表示L3缓存未命中的事件。-a选项表示监测所有的进程，-I 1000表示每1000毫秒输出一次统计结果。
# time counts unit events
 1.001228842 675,693 mem_load_retired.l3_hit 
 1.001228842 868,728 mem_load_retired.l3_miss 
 2.002185329 746,869 mem_load_retired.l3_hit 
 2.002185329 965,421 mem_load_retired.l3_miss 
 3.002952548 1,723,796 mem_load_retired.l3_hit 
[...]
```
### 硬件采样

```sh
root@l:~# perf record -e mem_load_retired.l3_miss -c 50000 -a -g -- sleep 10
[ perf record：Woken up 1 times to write data ]
[ perf record：Captured and wrote 3.355 MB perf.data (342 samples) ]

#使用perf工具在Linux系统上记录内存访问L3缓存未命中事件的性能数据。
#-e mem_load_retired.l3_miss：指定要监测的事件为L3缓存未命中事件。
#-c 50000：设置每个进程记录事件的次数为50000次。
#-a：监测所有的进程。
#-g：收集调用图(Callgraph)数据，以便获取函数调用关系。
#-- sleep 10：运行sleep命令10秒。这里用来模拟进程运行的情况，perf会在sleep命令执行期间记录性能数据。
```

### 定时采样

```sh
[ perf record：Woken up 1 times to write data ]
[ perf record：Captured and wrote 0.438 MB perf.data (651 samples) ]

#使用perf工具在Linux系统上记录性能数据，具体参数含义如下：
#-F 99：设置采样频率为每秒99次。这意味着perf将以每秒99次的频率对CPU进行采样，记录性能数据。
#-a：监测所有的进程。
#-g：收集调用图(Callgraph)数据，以便获取函数调用关系。
#-- sleep 30：运行sleep命令30秒。这里用来模拟进程运行的情况，perf会在sleep命令执行期间记录性能数据。
#通过这个命令，我们可以在一段时间内（这里是30秒）以较高的采样频率记录性能数据，包括函数调用图信息。这可以帮助进行更详细的性能分析和优化，并提供更准确的性能指标。


root@l:~# perf report -n --stdio
#使用perf工具在Linux系统上生成性能报告，并将结果打印到标准输出（stdio）。其中的参数含义如下：
#-n：以不启用交互式模式的方式生成报告，直接打印到标准输出。
#--stdio：将报告结果输出到标准输出。

# To display the perf.data header info, please use --header/--header-only options.
# Total Lost Samples：0
# Samples：651  of event 'cycles'
# Event count (approx.)：3360497157
#
# Children      Self       Samples  Command          Shared Object             Symbol                >
# ........  ........  ............  ...............  ........................  ......................>
#
    11.83%     0.00%             0  swapper          [kernel.kallsyms]         [k] secondary_startup_>
            |
            ---secondary_startup_64_no_verify
               |
               |--6.94%--start_secondary
               |          cpu_startup_entry
               |          |
               |           --6.79%--do_idle
               |                     |
               |                      --5.98%--cpuidle_idle_call
[...]
```

#### 产生CPU火焰图

```sh
root@l:~#git clone https://github.com/brendangregg/FlameGraph
root@l:~## cd FlameGraph
root@l:~## perf record -F 49 -ag -- sleep 30
root@l:~## perf script --header | ./stackcollapse-perf.pl | ./flamegraph.pl > flame1.svg

#将perf工具生成的性能数据进行处理，并生成火焰图。具体流程如下：
#perf script --header：读取perf记录的性能数据并输出到标准输出，同时包含文件头部信息。
#| ./stackcollapse-perf.pl：将perf输出的数据进行堆叠折叠处理，以便生成火焰图所需的格式。
#| ./flamegraph.pl：根据折叠处理后的数据，生成火焰图。
#> flame1.svg：将生成的火焰图保存到名为flame1.svg的SVG文件中。
```

### 周期采样

当perf(1)进行定时采样时，它试图使用基于PMC的硬件CPU周期溢出事件来触发不可屏蔽中断（NMI）以执行采样。然而，在云服务厂商中，许多实例类型没有启用PMCs。这可以使用dmesg(1)命令查看。

```sh
# dmesg | grep PMU
[ 2.827349] Performance Events：unsupported p6 CPU model 85 no PMU driver, 
software events only
```

在这些系统上，perf(1)会回退到基于hrtimer的软件中断。可以使用perf -v参数查看。

```sh
# perf record -F 99 -a -v
Warning:
The cycles event is not supported, trying to fall back to cpu-clock-ticks
[...]
```

软件中断模式通常在大部分性能分析场景是足够的。但是一些内核代码路径没法进行软中断。

### 事件统计与事件跟踪



```sh
root@l:~# perf stat -e sched:sched_process_exec -I 1000
#用于统计调度事件的性能数据。
#-e sched:sched_process_exec：这是perf命令的选项，用于指定要监测的事件。在此示例中，使用#sched_process_exec事件来跟踪进程的执行情况。
#-I 1000：这是perf stat命令的选项，用于指定统计数据的间隔。这里将每隔1000毫秒（1秒）输出一次统计结果。

#           time             counts unit events
     1.002080217                  0      sched:sched_process_exec
     2.008114896                  0      sched:sched_process_exec
     3.010333987                  0      sched:sched_process_exec
[...]
```







## BPF工具

execsnoop：是一个用于跟踪系统中执行的进程和命令的工具。它可以显示每个进程的执行时间、命令参数等信息。常见的用途是用于分析系统中的进程调度和执行情况，例如查找可能导致性能问题的进程或命令。

exitsnoop：用于监测进程的退出事件。它可以显示进程的退出状态码、退出时间等相关信息。常见的用途是用于分析和调试进程的退出原因，例如跟踪特定进程的异常退出情况或检测进程之间的依赖关系。

runqlat：用于监测运行队列的延迟。它提供了关于任务等待处理的时间信息，帮助用户识别系统中可能存在的任务调度延迟问题。常见的用途是用于评估系统的响应性能和任务调度效率。

runqlen：用于监测运行队列的长度。它可以显示运行队列中等待处理的任务数量，帮助用户评估系统的负载情况。常见的用途是用于监测系统的负载水平和任务分配情况。

runqslower：用于监测运行队列中的慢任务。它显示了运行时间超过设定阈值的任务信息，帮助用户发现系统中的性能瓶颈。常见的用途是用于分析和优化系统的任务调度策略。

cpudist：用于检测CPU利用率的分布情况。它提供了关于CPU使用情况的统计数据，帮助用户了解系统中不同CPU核心的负载情况。常见的用途是用于评估系统的CPU性能和识别CPU负载不均衡的问题。

cpufreq：用于跟踪和调整CPU频率。它可以显示当前CPU频率以及频率变化的相关信息，帮助用户优化系统的能耗和性能。常见的用途是用于调整CPU频率以在功耗和性能之间取得平衡。

profile：用于性能分析，可以生成进程级和系统级的性能数据报告。它提供了对CPU使用、内存使用和I/O行为等方面的详细分析，帮助用户识别和解决性能瓶颈问题。常见的用途是用于深入分析应用程序或系统的性能问题，并进行优化。

offcputime：用于监测离线CPU时间。它可以检测到在系统中消耗大量离线CPU时间的进程，并提供相关的统计信息，帮助用户识别并解决这类问题。常见的用途是用于分析系统中离线CPU时间的使用情况，并优化相关的进程或任务。

syscount：用于监测系统调用的次数和延迟。它提供了对系统调用的详细分析，帮助用户评估系统性能和优化应用程序的系统调用过程。常见的用途是用于跟踪系统调用的性能瓶颈和优化系统调用的效率。

argdist and trace：工具用于分析系统调用的参数分布情况，trace工具用于跟踪系统调用的执行过程。它们可以帮助用户深入了解系统调用的使用情况和性能瓶颈。常见的用途是用于分析和优化特定系统调用的参数使用和执行效率。

funccount：用于统计特定函数的调用次数。它可以帮助用户了解特定函数的使用情况和性能影响。常见的用途是用于分析和优化特定函数的调用频率和执行效率。

softirqs：用于监测软中断的使用情况。它可以显示软中断的触发次数和延迟信息，帮助用户评估系统中软中断的性能开销。常见的用途是用于分析和优化软中断的使用和效率。

hardirqs：用于监测硬中断的使用情况。它可以显示硬中断的触发次数和延迟信息，帮助用户评估系统中硬中断的性能开销。常见的用途是用于分析和优化硬中断的使用和效率。

smpcalls：用于监测多处理器系统中的函数调用次数和延迟信息。它可以帮助用户了解多处理器系统中函数调用的分布情况和性能影响。常见的用途是用于分析和优化多处理器系统中函数调用的负载和效率。

llcstat：用于监测最后级缓存（LLC）的使用情况。它提供了关于LLC的命中率、清除率等信息，帮助用户评估系统的缓存性能和优化内存访问模式。常见的用途是用于分析应用程序的缓存行为和检测缓存相关的瓶颈。



# MEM内存

## 内核日志





## 内核统计信息







## 硬件统计和硬件采样





## BPF工具

oomkill ：用于跟踪和记录内核中的 OOM（Out of Memory）杀进程事件。当系统内存不足时，内核会选择终止某些进程以释放内存资源，并通过 oomkill 工具记录这些事件。常见使用方法是查看 oomkill 记录以了解系统中哪些进程被终止。

memleak：用于检测内核模块或驱动程序中的内存泄漏。它通过监视内核堆栈和分配的内存块之间的关系来识别未释放的内存。常见使用方法是在内核模块或驱动程序开发过程中运行 memleak 工具，以帮助发现和修复内存泄漏问题。

mmapsnoop：用于监视和记录进程创建和销毁的内存映射操作。它可以跟踪进程打开、映射和关闭的文件、库和设备，以及相关的内存操作。常见使用方法是在调试和性能优化场景中使用 mmapsnoop 工具，以了解进程的内存映射情况。

brkstack：用于跟踪和记录进程的堆布局和堆栈信息。它可以显示进程的堆起点、堆大小以及堆栈的布局情况。常见使用方法是在调试和分析场景中使用 brkstack 工具，以帮助了解进程的堆栈结构和内存使用情况。

shmsnoop：用于监视和记录共享内存段的创建、连接和断开操作。它可以追踪进程之间的共享内存通信，并提供有关内存段的详细信息。常见使用方法是在调试和性能分析场景中使用 shmsnoop 工具，以了解进程之间的共享内存使用情况。

faults：用于跟踪和记录进程的页面错误（Page Fault）事件。它可以监视进程的内存访问，并记录因页面错误而触发的异常情况。常见使用方法是在调试和优化场景中使用 faults 工具，以了解进程的内存访问模式和性能瓶颈。

ffaults：用于追踪和记录文件系统页面错误（Filesystem Page Fault）事件。它可以监视文件系统的磁盘 IO 操作，并记录因页面错误而导致的异常情况。常见使用方法是在调试和性能分析场景中使用 ffaults 工具，以了解文件系统的磁盘 IO 行为和性能瓶颈。

vmscan：用于监视和记录内核的页面回收（Page Reclaim）操作。它可以显示内核在内存压力情况下如何回收页面，并提供有关内存管理和页面置换的详细信息。常见使用方法是在调试和优化场景中使用 vmscan 工具，以了解内核的页面回收策略和性能影响。

drsnoop：用于监视和记录内核的读写操作。它可以追踪进程访问的文件、设备等，并提供有关读写操作的详细信息。常见使用方法是在调试和性能分析场景中使用 drsnoop 工具，以了解进程的文件和设备访问行为。

swapin：用于监视和记录内核的页面交换（Page Swap In）操作。它可以追踪内核在内存压力情况下将页面从磁盘交换到内存的过程，并提供有关页面交换的详细信息。常见使用方法是在调试和性能分析场景中使用 swapin 工具，以了解内核的页面交换策略和性能影响。

hfaults：用于追踪和记录硬件页面错误（Hardware Page Fault）事件。它可以监视硬件引起的页面错误，并提供有关异常情况的详细信息。常见使用方法是在调试硬件相关问题和系统稳定性分析中使用 hfaults 工具，以帮助诊断硬件故障和异常情况。







# 文本处理工具

cat输出文件的内容		-n显示行数且空行显示行数		-b显示行数且空行不显示行数

-s压缩连续的空行为一行			-A显示所有控制符（用于调试脚本时）

cat >> /s <<EOF	以EOF表示结束，在交互式中输入内容

tee	读取屏幕信息，并再将信息输出到屏幕和文件中,-a或--append追加到文件里		echo "123" | tee -a file1

echo 输出，输入			echo $?输出上次命令反回值	echo `` 输出执行命令结果

echo "redhat" | passwd -- stdin alex 将密码redhat添加到alex账户

tac	/ rev		从下至上 / 从右至左 输出文件内容

less 分页查看

head / tail	显示文件前 / 后几行	-n 指定文本显示的行数		-c 指定文本显示的字符数

tail	参数		-f 实时输出文件内容的变化（tailf也常用于跟踪脚本、日志信息）		-F 跟踪文件名

## cut

列剪贴工具 / paste列合并工具	cut -d " " -f1,4-7 --complement  --c 1-4  a

```
-d delimiter：分隔符，默认的字段分隔符为“Tab”		-c：输出指定范围的字符	
-f  file：提取指定段（可以多个段）的内容		--complement反向选择字符段（除指定字段外的所有）
-b：输出指定范围的字节
```

paste：合并文件的列	 -s 将行换成列输出	-d，--delimiters=<间隔字符>分隔符

tr：标准输入的字符进行替换、压缩和删除	-d删除				-s去重复字符

uniq：去除相邻且重复的行				-c显示重复的次数		-u仅显示不曾重复的行

wc：word count统计行数，如下格式	-l 行数	-w 单词数	-c 字节	-m字符总数	-L文件最长行长度

diff：输出文件间区别		-u以合并的方式来显示文件内容的不同(常用于还原备份)	diff -u s a > s2.brk

sort：排序（默认小到大排序）	-n 数字由大到小排序(-r 降序搭配使用)		sort -n /opt/pp

-t 指定字段的分隔符			-u 去除重复行			-k 指定行数		sort -t ':' -nrk 3,6 /etc/passwd

## seq

squeue 是一个序列的缩写，主要用来输出序列化的东西

```
用法：seq [选项]... 尾数		或：seq [选项]... 首数 尾数		或：seq [选项]... 首数 增量 尾数（默认增量为1）
-s, --separator=字符串    指定分隔字符(默认使用：\n)			seq -s  '#'  1 2 10
-w, --equal-width   在列前添加0 使得宽度相同（自动补位）		seq -w 1 10
-f, --format=格式   指定格式输出，%g为默认格式
```

seq -f ‘%g’1 5	#表示指定“位宽”为三位，数字位数不足部分用空格补位）

seq -f "%03g" 2 9	#表示指定“位宽”为三位，数字位数不足部分用0补位，通过%后添加0替代空格补足空位，等同-w 

seq -f "as%03gaa" 4#% 前面可以指定字符串，同样 g 的后面也可以指定字符串

## grep/egrep/fgrep

强大的文本搜索工具，支持正则表达式搜索文本，显示具有指定字符的行

```
-n显示匹配行及行号		 grep -n root /etc/passwd
-v反向选择匹配到的内容	grep -v aa file
grep -v '^$' /etc/zabbix_server.conf 找出所有非空行
-i不区分大小写						grep -i ROOT /etc/passwd
-c输出匹配到字符的行数				 grep -c root /etc/passwd
-E选择多个字符						grep -E "root|zhunajie"  /etc/passwd
-h查询多文件时不显示文件名				grep -h  root  /etc/passwd /etc/shadow
-l只输出包含匹配字符的文件名				grep  -l root /etc/passwd
-s不显示不存在或无匹配文本的错误信息
^：以某某开头的内容		$：以某某结尾的内容		^$： 空行	.*：匹配所有的字符		^.*：任意多个字符开头
.：表示且只能代表任意一个字符 (当前目录，加载文件) 	\：转移字符，让有着特殊身份的字符，变回原来的字符
*：重复0个或多个前面的一个字符，不代表所有			[abc]：匹配字符集合内任意一个字符[a-z]
[^abc] ：^在中括号表示非，表示不包含a或者b或者c
```

## sed行（Stream EDitor）流编辑器，

sed：以行为单位自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序

### 常用选项：

```
-n：不显示默认输出内容					-i：直接修改(-i.brk先对原文件备份brk，后生成修改文件)
-e：可以指定多个地址定界、选项、命令		 -r：启用扩展的正则表达式,当与其他选项使用时应作为首个选项
-{}：可组合多个命令,以分号分割			-f：使用sed脚本（如定义好的字符串或地址定界）
```

### 地址定界：

```
1、不给地址：对全文进行处理
2、单地址：	如：5：指定的行	$：最后一行		/正则表达式/：能被此模式所匹配到的每一行
3、地址范围：	1，3（1到3行）	或	1，+5（第一行再加5行）或	1，/正则表达式/
/字符串/	或	/正则表达式/		或	/正则表达式（开头）/，/或字符串（结尾）/
4、步进:		1~2（从第1行后每隔2行才进行匹配）奇数行		2~2（从第2行后每隔2行才进行匹配）偶数行
```

### 编辑命令（-i后接）

```
d：删除			p：打印，将匹配的行重复打印
i：前添加 \ a：后添加，a+字符串，原每一行的后一行都会添加指定字符串
c：代替，c+字符串，原每一行都会替换成指定的字符串
w：保存，w+文件名，保存匹配的行到指定文件
！：取反，放地址地址定界的后面		=：显示行号
s///：查找替代，支持其他分割符如s@@@，s###
替代标记：g：所有行替换，不加即替换匹配到的第一个字符串
p：显示替换成功的行	w + 文件名：将成功的行保存至指定文件
 [-n\-i选项]... {选项、地址定界；编辑命令}	 [输入文件]
```

### 输出文本：
sed -n 'p' a.txt输出所有行		sed -n '4p' a.txt 输出第4行		sed -n '4,+10p' a.txt输出第4行及其后的10行内容
sed -n '$=' 输出文件的行数		sed -n '4,7p' a.txt输出4-7行		sed -n '/^bin/p' a.txt 输出以bin开头的行
sed -n 'p;n' a.txt 输出奇数行		sed -n 'n;p’a.txt 输出偶数行	sed -n '10,$' 输出第10行到结尾的所有偶数行

### 删除文本：
sed -i '3,5d' a.txt删除第3-5行	sed -i '$d' a.txt删除最后一行		sed -i '/xml/d' a.txt 删除所有包含xml的行
sed -i '/^$/' a.txt删除重复空行	sed -i '/^$/d' a.txt 删除所有空行	sed -i '/xml/!d' a.txt 删除所有不包含xml的行
sed -i /^install/d' a.txt删除所有以

### 替换文本：
sed 's/xml/XML/' a.txt将每行的第一个xml替换为XML		sed 's/xml/XML/3' a.txt 将每行的第3个xml替换为XML
sed 's/doc/$docs/g' a.txt 	将所有的doc替换为docs $代表查找串
sed -r 's/^(.)(.)(.*)/\2\1\3/' a.txt	将文件中每行的第一个和第二个字符互换
sed -r 's/([a-Z]+)([^a-Z]*)([a-Z]+)(.*)/\3\2\1\4/' a.txt	将文件中每行的第一个和第二个单词互换
sed -r 's/[0-9]//g;s/^( )+//' a.txt	删除所有的数字和行首的空格
sed -r s/[A-Z]/(&)/g a.txt		为每个大写字母添加括号

echo rancher_elasticsearch-conf_v0.5.0| sed 's@_@/@;s@_@:@' 	#两次使用地址定界

ip a | sed -n '9p' | awk -F" " '{print $2}' #打印第9行，空格为界定符打印第二段

ifconfig |sed -r '2!d ; s@.*inet (.*) net.*@\1@'	#删除除第二行的所有行

## awk行列文本分析工具

awk + 选项 +'pattern模式{cation动作}'  ,支持正则表达式、自定义分隔符、自定义变量、类C语言

取行awk 'NR==9' test 取出第九行的内容

取列lsmod | awk -F： '{print $2}'  以：设置分隔符，默认为空格或Tab位打印第二段(列) 	{} 全选

### 内置变量：

NR：已读行的个数					NF：已读列的个数，$NF表示最后一列

FS：等于-F选项					$n：以分隔符为标志，选取段落，$0表示整行

OFS：将分隔符进行替换				FNR：保存当前处理行在原文本内的序号，行号

FILENAME：当前处理的文件名		ENVIRON：表示支持系统环境变量，格式ENVIRON["变量名"]

### 变量运算

ip a | awk 'NR<=9{print $2"@@"$1}' #匹配前九行打印第2段，第1段，并进行反序，中间加入@@字符串

awk -F：-vOFS=@ '{print $1,$2}' /etc/passwd #将第一、二段的分隔符：替换为@

echo "7.7 3.8"|awk '{print ($1-$2)}' #进行减运算并得出结果

awk '{sum+=$7} END {print "Avg=", sum/NR}' #将每一行的第七列累加，最后打印计算平均值	#BEGIN模块、END模块

awk '/^(root|ssh)/' /etc/passwd #打印所有以root或ssh开头的行

awk '$1 > 5 && $2 < 10' test #如果第一个域大于5，并且第二个域小于10，则打印这些行

awk '$1 = 100 {print $1 > "file" }' test #上式表示如果第一个域的值等于100，则把它输出到file中

echo a b c d | awk '{$2=$2" e f g";print}' #将第二段的字符替换为b e f g,再打印出来

awk‘NR%2\==0’test 输出偶数行文本			awk -F：'$1=="sync"’/etc/passwd 用户名为sync的行

awk -F:‘$1==ENVIRON["USER"]’ /etc/passwd

awk '/-rwx/{print}'  test >test1 打印包含“-rwx”字符的行

awk -F：'{if($3>999 && $7 == "/bin/bash" || $7 == "/bin/sh")print}' /etc/passwd 使用if判断列是否符合

## 输入/出重定向

```
1>正确输出						2> 错误输出		cat /opt/cc /home/  >/opt/right		2>/opt/wrong
1>>正确输出只能保存正确的输出		1>>正确且覆盖式重定向	
2>>错误输出只能保存错误的输出		2>>错误重定向	&>只要输出就能保存		&>混合重定向
输出内容到这里</etc/passwd
```

dd：用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。

```
dd if=/dev/zero of=/root/file1 bs=100M count=1 conv=fdatasync
用/dev/zero向file.1里面输入内容	count指定文件个数为1 ，bs=20M大小，fdatasync将数据写入磁盘
```

## find查找文件

find  / -name  "???.conf"任意三个字符（数字，字母，点逗号）

```
-type （d文件夹 f文件 l链接 b块设备 c字符）	find  /  -type  f  -name  "*abc*"
-name匹配文件名							  find  /  -name  "[a-z][a-z][a-z].conf"
-size文件大小（+表示大于）				   find  /  -size +100M 
-user所有者（-group所有组）					find  /  -user zhubajie ! (且)-group zhubajie
-nouser无所有者（-nogroup无所有组）		   find  /  -nouser
-exec …… {}\; 接下一个命令  --exec这个命令只能接在find命令后，也就是find专用的
```

## xargs(命令传递参数命令) 

ls | grep -v ^file$ | xargs  -i  cp  {}  /tmp/		-i 允许用大括号带替前面的内容		{} 代表示管道符号前的内容

ls | grep ^file$ | xargs rm -fr 	查找以file开头且以file结尾的

ls file*  | xargs  -i  mv{}  a{}		在以file开头的文件前加上a这个字符

## tar压缩工具和zip

tar -cjvf backup.tar.bz2 * 文件压缩成backup.tar.bz2名字的文件，最好时进入需要打包文件夹

tar -xzvf test.tar.gz	解压gz文件（在要压缩的目录里）

tar -xvf test.tar 解压tar文件

tar -xf node-v12.18.1-linux-x64.tar.xz 解压xz格式压缩包

```
-z 使用gzip压缩(.gz后缀压缩包)
-j 使用bzip2压缩（压缩的更小,当时间更长）.bz2后缀压缩包（-k 可保留源文件）	-C指定解压地址
-c 创建/压缩	-x 释放/解压缩		-f 指定压缩文件名字		-v 显示提示信息		-tf 查询内容
```

zip /srv/file.zip  * 将当前文件压缩成file.zip文件		-sf查看压缩包里面的内容	-r表示递归压缩子目录下所有文件

zip -d myfile.zip smart.txt #删除压缩文件中smart.txt文件

zip -m myfile.zip ./rpm_info.txt #向压缩文件中myfile.zip中添加rpm_info.txt文件

unzip -od /home/sunny myfile.zip #把myfile.zip文件解压到 /home/sunny/

-o:不提示的情况下覆盖文件	-d:将文件解压缩到指定目录下

## ps 进程信息控制kill

进程是个动态的概念，即正在进行的程序，程序是个实体

内核的功能：进程管理，文件系统，网络功能，内存管理，驱动程序，安全功能

Process：运行中的程序的一个副本，是被载入内存的一个指令集

进程ID（ProcessID，PID）号码被用来标记各个进程

UID、GID、和selinux语境决定对文件系统的存取和访问权限

通常从执行进程的用户来继承

存在生命周期<img src="E:\Project\Textbook\linux云计算\assets\wps3-1682690115363-176.jpg" alt="img" style="zoom：67%;" />

task struct：linux内核储存进程信息的式数据结构格式

task lsit：多个任务的task struct组成的链表5

进程创建：init：第一个进程

### 父子关系

进程：都由其父进程创建，Cow	fork（）【用来生成子进程】，clone（）【克隆】

ps -ef/aux	显示当前进程（ e显示所有进程 f全格式  /  a所有u用户x不同7个终端）

<img src="E:\Project\Textbook\linux云计算\assets\wps4-1682690115363-177.jpg" alt="img" style="zoom:50%;" /> 

kill -9暴力杀死进程		-9（暴力杀死）  -15（逐步杀死）+进程号	18继续 19停止 20暂停

pkill -9 dns	杀掉相关的进程（如杀掉dns服务)				        pkill -9 zhubajie	杀掉用户zhubajie的所有进程

pkill -vu root	(-v反选)杀掉除root用户打开的所有进程			pkill -9 %2		作业号为2的进程

Command &	放到后台运行		jobs查看当前终端后台的进程

fg 2把后台2进程放到前台来运行			fg%1将后台job编号为1的进程调回到前台

bg把后台暂停的进程放到后台运行			bg %2把后台编号为2的进程恢复运行状态

firefox  www.baidu.com  & 打开浏览器放到后台运行





## at一次性任务、周而复始计划任务

echo `Ifconfig ens33` | at now+1 min

-L：查看所有任务 	-c1：查看任务1详细信息		-d1：删除任务1

<img src="E:\Project\Textbook\linux云计算\assets\wps8-1682690115363-181.jpg" alt="img" style="zoom：67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps9-1682690115363-182.jpg" alt="img" style="zoom：67%;" /> 

-l查看当前生效用户的任务列表		-e编辑当前生效用户的任务内容				-r删除当前生效用户的任务内容

-i同上-r，但是在删除之前会询问确认	-u不针对当前生效用户，对指定的用户去操作	-R删掉用户所有周而复始任务

cat /var/log/cron日志文件			/var/spool/cron查看定时任务的文件夹			Crontab内每一行一个任务

watch -n1  ls /srv/ 				watch每一秒监控一次 ls命令srv下输出结果

# 网络管理工具

## ubuntu

```sh
 systemctl start systemd-networkd //开启 systemd-networkd服务
 netplan apply //重启网络服务
 vim 01-network-manager-all.yaml //修改网络配置文件
```



## centos

wget -O -r -p /etc/yum.repos.d/ali.repo http://mirrors.aliyun.com/repo/Centos-7.repo	下载阿里源

```
-O下载并指定文件名保存	-b后台下载	-r递归	-P指定下载目录		-S模拟输出服务器GET响应
```

curl -u admin:12345 http://192.168.2.5:3000	#-u指定admin用户名密码12345和指定端口进行登陆

```
curl -vs -o /dev/null -A "Google Chrome" -e -e www.baidu.com -x 103.219.36.13:8080 http://cacti.rednetcloud.com
```

```
-v输出详细信息			-x指定使用的http代理				-A,--user-agent 用户访问代理软件
-o将输出信息保存到指定文件	-s,silent静默模式，不输出任何信息	-I只显示响应报文头部信息
-O将服务器回应保存成文件，并将 URL 的最后部分当作文件名
-e,--referer告诉服务器我从哪个页面链接过来的
-b,--cookie使用指定cookie文件	-X GET|POST 设置请求方式
```

sz +文件名：将选定的文件发送（send）到本地机器			rz从本地选择文件上传到Linux服务器

iperf3网络质量测试工具（默认用tcp协议测试）

```
-s运行server模式					-D在后台以服务的方式运行
-c + host：连接指定客户端ip地址		-R服务端发送，客户端单方接受
```

vim /etc/NetworkManager/system-connections/ens160.nmconnection rocky系统网卡文件

lsof -i：22	通过端口号查看对应服务（ftp的端口）

ss / netstat -tupln 查看网络状态(查看各类服务)				route 显示和设置linux系统的路由表

route add –host 192.168.168.110 dev eth0				#添加到主机的路由

route add –host 192.168.168.119 gw 192.168.168.1			#添加主机路由网关

route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.1.1		#net添加网段

traceroute -p 80 www.baidu.com #追踪网络数据包的路由途径，-p指定端口

tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

1、tcp、ip、icmp、arp、rarp、tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型

2、-i eth1 ：只抓经过接口eth1的包

3、-t ：不显示时间戳

4、-s 0 ：抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包

5、-c 100 ：只抓取100个数据包

6、dst port ! 22 ：不抓取目标端口是22的数据包

7、src net 192.168.1.0/24 ：数据包的源网络地址为192.168.1.0/24

8、-w ./target.cap ：保存成cap文件，方便用ethereal(即wireshark)分析

### Network、链路聚合

ping -c3 -i0.1 192.168.1.1 		-c 次数		-i间隔	-t不间断

```sh
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"           //修改：dhcp修改为static
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="ef482e1b-4f2f-4d23-ae65-784122f24201"
DEVICE="ens33"
ONBOOT=yes                  //修改：修改为yes, 配置网卡开机自启动
IPADDR="10.193.12.23"       //新增：配置静态IP地址
NETMASK="255.255.255.0"     //新增：配置子网掩码
GATEWAY="10.193.12.254"     //新增：配置网关
DNS1="10.1.26.188"          //新增：配置DNS
```

nmcli	n[etworking]		on\off	#连接设置

nmcli	d[evice]				status查看设备状态 \ show ens33查看网卡设备信息 \set

nmcli 	c[onnection]  	reload重载 \ up激活 + 网卡配置文件 \ down  \ load \ modify 修改 ens160

ipv4.addresses 192.168.127.10/24修改IP地址	+ipv4.addresses 172.16.10.10/16添加第二个IP地址\DNS

ipv4.gateway 192.168.127.2 修改网关			ipv4.dns 8.8.8.8

ipv4.method manual修改为手动配置 			ipv4.method auto修改为DHCP获取

nmcli	g[eneral]				status\hostname\logging(支持服务信息)		#概叙信息



### 创建网络组接口

nmcli con add type team con-name xxx ifname xxx config JSON

type  team 设备类型 ，con-nam 连接名，  ifname 接口名

JSON 指定runner方式，格式： '{"runner"：{"name"："METHOD"}}'

METHOD ：	broadcast, roundrobin,activebackup, loadbalance, lacp

Nmcli con mod team0 ipv4.addresses  192.168.10.25/24 ipv4.method manual

nmcli con add type team-slave (con-name port1) ifname eth1 master team0添加从属接口

Nmcli con add type team-slave (con-name port2) ifname eth2 master team0

Nmcli con up team0

Teamdctl team1 state			查看虚拟链路聚合网卡详细信息

nmcli conn state team0

 

nmtui网卡编辑器				nm-connection-editor网络连接编辑工具

绑定两张网卡（需要重启网卡配置两次）

Team：可以实现服务器数据传输链路的冗余容错，有效消除单点故障隐患。当Team中的一个物理网络连接失效时，其它的可用网络连接会自动接管负载，从而保证数据传输的可持续性

[root@localhost network-scripts]# cp /usr/share/doc/teamd-1.29/example_ifcfgs/1/ifcfg-* .	//复制网卡聚合链路模板

activebackup表示目前是主备模式，roundrobin表示负载均衡

<img src="E:\Project\Textbook\linux云计算\assets\wps11-1682690115363-184.jpg" alt="img" style="zoom:50%;" /> 

bond：主要用于网络吞吐量很大，及对于网络稳定性要求较高的场景。主要是通过将多个物理网卡绑定到一个逻辑网卡上，实现了本地网卡的冗余，带宽扩容以及负载均衡

mode0 (平衡负载模式):平时两块网卡均工作,且自动备援,但需要在与服务器本地网卡相连的交换机设备上进行端口聚合来支持绑定技术

mode1(自动备援模式):平时只有一块网卡工作,在它故障后自动替换为另外的网卡

mode6 (适配器适应性负载均衡模式):平时两块网卡均工作，且自动备援,无须交换机设备提供辅助支持

```
ifcfg-eno1					ifcfg-bond
TYPE=Ethernet				DEVICE=bond	 #和文件名没关系
NAME=eno1					NAME=bond	 #和文件名没关系
DEVICE=eno1					TYPE=Bond
BOOTPROTO=none				IPADDR=192.168.5.249
ONBOOT=yes					NETMASK=255.255.255.0
MASTER=bond0				GATEWAY=192.168.5.1
SLAVE=yes					PEERDNS=yes
ONBOOT=yes
BOOTPROTO=static
BONDING_MASTER=yes
BONDING_OPTS="mode=6 miimon=100 updelay=600000 primary=eno1"	#模式和网卡
```

bond模式起不来步骤：1、service NetworkManager stop 临时和永久关闭NetworkManager		2、reboot重启

IP addr 查看网卡状态	ifdown 关闭网卡	ifup打开网卡		cat /proc/net/bonding/bond0 查看bond是否生效

lspci|grep net 查看所有网卡型号参数		

iftop -i eth2 #支持画面操作（区分大小写）-i指定网卡 -n将主机名显示为IP  -N不将端口号转换为服务 -P显示端口和主机

ethtool #查询或控制网络驱动程序和硬件设置

ethtool -i eth2 #-i显示网卡驱动名称、版本信息		-d网口注册性信息		-S ens33：网口收发包统计

# 软件包管理

## Rpm

/run/media/root/CentOS\ 7\ x86_64/Packages/	文件下默认含有rpmb=包

Rpm -ivh +二进制包	-i 安装（install）-e删除 -h 进度条显示 -v显示	-q查询	-a所有	--force强制安装

-U/--upgrade<套件档> 升级指定的套件档

Rpm -qa	查询所有安装过的包	 -ql	查看所有安装过的路径		-qf  bin/chmod	查看文件由哪个包提供

-qi 查看安装包详细信息			 -e 钥匙 删除不匹配的钥匙		 -K 包名 查看该包需要的钥匙，显示 OK

显示NOT OK (MISSING KEYS：(MD5) PGP#fd431d51) 中文版显示：不正确

gpg -v 钥匙包  		//从系统上所有的钥匙包里面，找到需要的钥匙

rpm包的命名方式：name-version-release.arch.rpm

​      发行号：用于标识rpm包的本身发行号，可还包含所适用的操纵做系统

​      例如：el6：RHEL6

​      arch：主机平台

​      例如：i386、x86_64、amd64、ppc、noarch不区分平台

## Yum仓库

**vim /etc/yum.repo.d/**

```sh
[rhel]
Name=rhel
Baseurl=file:/mnt/sr0  //资源定位路径http ftp nfs:192.168.0.1/media
Gpgcheck=0	//是否开启公钥检测
Enabled=1	//是否开启存库

mount -o loop /root/XianDian-IaaS-v1.4.iso /media/ #挂在镜像文件
```

**Yum命令**

| 命令                      | 作用                         | 命令                     | 作用                 |
| ------------------------- | ---------------------------- | ------------------------ | -------------------- |
| yum repolist all          | 列出所有仓库                 | yum list all             | 列出仓库中所有软件包 |
| yum info软件包名称        | 查看软件包信息               | yum install 软件包名称   | 安装软件包           |
| yum reinstall 软件包名称  | 重新安装软件包               | yum update 软件包名称    | 升级软件包           |
| yum remove 软件包         | 移除软件包                   | yum clean all            | 清除所有仓库缓存     |
| yum grouplist             | 查看系统中已经安装的软件包组 | yum check-update         | 检查可更新的软件包   |
| yum groupinstall 软件包组 | 安装指定的软件包组           | yum groupremove 软件包组 | 移除指定的软件包组   |
| yum groupinfo 软件包组    | 查询指定的软件包组信息       | yum search ntpd          | 搜索安装包ntpd       |
| yum provides              | 查看命令由哪个包提供         |                          |                      |

service yum-updatesd stop	#停止yum自动更新服务		chkconfig yum-updatesd off #禁止yum更新开机启动



## ubuntu使用阿里云源

**切换路径** && **备份源文件** && **新建源文件**

```bash
mv /etc/apt/sources.list /etc/apt/sources.list.bak && vi /etc/apt/sources.list
```

ubuntu 18.04(bionic) 配置如下
```bash
deb https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

```

ubuntu 20.04(focal) 配置如下

```sh
deb https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

# deb https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```



## make编译安装

make：项目管理器，进行源码包编译安装

configure脚本-->Makefie.in模板文件-->Makefile文件

编译环境安装：yum groups install Development\ Tools -y 安装包组

C语言源码编译安装三大步：

1、	./configure（源码包默认自带的安装脚本）		--prefix=/PATH：指定安装路径（默认为/usr/local）	

--with-libevent=/usr/local/libevent/：指定依赖软件地址	--help：查看帮助

 --add-module=/usr/local/src/headers-more-nginx-module-master 添加模块并指定模块地址

（1）通过选项传递参数，指定启用特性、安装路径等；执行时参考用户的指定以及Makefie.in模板文件生成Makefile

（2）会检查依赖到的外部环境，如依赖的软件包

2、	make 根据Makefile文件，构建应用程序		make -j 4	指定CPU运行的数量

3、	make install 复制文件到相对路径

4、	echo "PATH=/usr/local/nginx/sbin:$PATH" >> /etc/bashrc		将启动命令添加至环境变量

# 用户和组的管理

/etc/shadow(用户影子文件) 用户名：密码：最后修改时间：最小时间间隔：最大时间间隔：警告时间：不活动时间：失效时间：标志字段

/etc/passwd（用户账户文件） 账户名称 ：密码 ：用户ID ：群组 ：个人资料：主目录：shell

passwd 存放所有用户信息（含root）			没x代表免密登录

shadow 存放系统所有密码的文件				group 存放所有组的文件

chage命令负责管理用户密码时效问题			chage 改用户密码（也就是shadow类里面的）

```
 -d 0， --lastday 最近日期   指定用户最后修改日期为今天
 -E, --expiredate 过期日期   将帐户过期时间设为“过期日期” 
 -I, --inactive INACITVE	 过期 INACTIVE 天数后，设定密码为失效状态
 -l, --list          		 显示帐户年龄信息
 -m, --mindays 最小天数     将两次改变密码之间相距的最小天数设为“最小天数”
 -M, --maxdays 最大天数     将两次改变密码之间相距的最大天数设为“最大天数”
 -R, --root CHROOT_DIR     chroot 到的目录
 -W, --warndays 警告天数    将过期警告天数设为“警告天数”
```

1.设置用户密码最小天数（-m），最大天数（-M），提醒天数（-W），不活跃天数（-I）

[root@localhost ~]# chage -m0 -M8 -W8 -I5 zhubajie

id -gn zhubajie	查看zhubajie组名			-gn (group name) 组名		-un (user name) 用户名

useradd aaa（建立用户名aaa）

```
usermod 修改用户属性信息 	-d 改变用户主目录		-g 修改用户主组	-G 指定用户所属附加组	-l name:更改账户的名称
-u UID：改变用户的UID为新的值		-s 指定以/sbin/nologin非shell登录	-e设置用户账户使用期限
```

userdel 删除用户 -fr（强制删除用户及其主目录）	passwd aaa（给用户aaa设置密码）

groupadd 添加组

```
groupmod 修改组		-g 修改用户所属的群组	-l 修改用户帐号名称	-u 修改用户ID
-d 修改用户登入时的目录	-s 修改用户登入后所使用的shell（/sbin/logoin非交互式登录)
```

gpasswd  管理组成员	-a向组中添加用户		-d从组中删除用户

groupdel 删除组

## 文件权限

user group other					su 切换用户 用法：su -xxx(用户)

u(所有者)，g(所属组)，o(其他用户)，a(所有用户)		+（代表添加） -（减少） =（权限覆盖）

r（读取权限数字代表为4)，w（写入权限数字代表为2)，x（执行权限数字代表为1）

chmod 更改文件或目录的权限			chmod	a+rwx  a			chmod 777 a	

chown 更换文件所有者/更换文件所属组	chown 	root:bin 	/file

chgrp 变更文件或目录的所属群组 			chgrp	root	-R 	/file

u+s所有人对此文件都有执行权限			g+s强制将此群组里的目录下文件编入到此群组中

o+t这个目录只有root和此目录的拥有者可以删除，其他用户全都不可以

chattr +i/-i	 123	 锁定/取消文件或目录（锁定后无法修改删除移动）	lsattr 查看文件是否被锁定

## ACL访问控制列表

setfacl：具体权限设置ACL		u：用户		g：组	d：默认

```
-m更改文件的访问控制列表		setfacl -m u:zhangy:rw- test修改文件的acl权限，添加一个用户权限
-R递归操作子目录				setfacl -Rm d:zhangsan:rwx /opt/tt 为目录/opt/tt设置ACL规则
-x从文件中访问控制列表移除条目	setfacl -x u:zhangsan /opt/ta 删除用户zhangsan对/opt/ta文件的ACL规则
```

setfacl -x u:zhangsan /opt/ta删除用户zhangsan对/opt/ta文件的ACL规则

```
-b还原成默认acl列表			setfacl -b  test 还原acl列表
-d应用到默认访问控制列表的操作	setfacl -d --set g:zhangsan:rwx /ok 设置ok目录默认ACL
-h显示本帮助信息
```

getfacl  /file查文件权限的内容			-n显示数字的用户/组标识		-R递归显示子目录

umask 权限掩码，设定文件创建时的缺省模式

umask 0644一次性缺省权限		/etc/profile	文件里修改反掩码

用户设置自己永久性的umask值，在自己$HOME目录下的.profile或.bash_profile文件中

## selinux权限设置

配置文件：/etc/sysconfig/seliunx						chcon和setsebool、getsebool都是selinux的一部分

chcon：控制文件的访问属性

```
-R, --recursive：递归处理所有的文件及子目录		-t, --type=类型：设置指定类型的目标安全环境
-u, --user=用户：设置指定用户的目标安全环境		   -r, --role=角色：设置指定角色的目标安全环境
-v, --verbose：为处理的所有文件显示诊断信息		-l, --range=范围：设置指定范围的目标安全环境
```

restorecon：恢复SELinux文件属性即恢复文件的安全上下文		-R/-r：递归处理目录	-v：将过程显示到屏幕上

setsebool：服务功能的开关，修改SElinux策略内各项规则的布尔值	-P:直接将设置值写入配置文件并生效

getsebool：查询SElinux策略内各项规则的布尔值

semanage fcontext：主要用在安全上下文方面 ：用于管理SElinux策略

-l：查询		-a：增加，你可以增加一些目录的默认安全上下文类型设置		-m：修改		-d：删除

<img src="E:\Project\Textbook\linux云计算\assets\wps13-1682690115364-186.jpg" alt="img" style="zoom:67%;" /> 

# 磁盘管理

使用分区空间：1、设备识别	2、设备分区	3、创建文件系统

4、标记文件系统	5、在/etc/fstab中创建条目	 6、挂载新的文件系统

分区目的：优化I/O性能、实习磁盘空间配额限制、提高修复速度、隔离系统与程序、安装多个OS、采用不同文件系统

文件系统：是操作系统明确储存设备或分区上的文件的方法和数据结构，即在储存设备上管理文件的方法

分配文件的大小：linux以块为单位，win上叫簇（4k为一单位）

<img src="E:\Project\Textbook\linux云计算\assets\wps14.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps15-1682690115364-187.jpg" alt="img" style="zoom：67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps16-1682690115364-188.jpg" alt="img" style="zoom:67%;" /> 

磁盘分区：MBR(用fdisk分区) 和 GPT（用gdisk命令，需安装软件）

mkfs.创建文件系统				-t xfs指定文件系统					-b 1024指定块的大小

mount 文件系统挂载命令		umount 卸载命令

-r：readonly，只读挂载			-w：read and write, 读写挂载			-a：自动挂载所有支持自动挂载的设备

-L 'LABEL'：以卷标指定挂载设备	-U 'UUID'：以UUID指定要挂载的设备	-B, --bind：绑定目录到另一个目录上

-o remount +挂载文件夹：重新挂载   ，ro：只读    rw:读写，	user/nouser：是否允许普通用户挂载此设备	

acl：启用此文件系统上的acl功能，sync：同步模式，数据直接写入磁盘 ，async：异步模式，数据先到内存，再转磁盘

diratime/nodiratime：目录的访问时间戳				auto/noauto：是否支持自动挂载

exec/noexec：是否支持将文件系统上应用程序运行为进程	atime/noatime：包含目录和文件

   dev/nodev：是否支持在此文件系统上使用设备文件    suid/nosuid：是否支持在此文件系统上使用特殊权限

fsck  /dev/sda2 修复文件系统(磁盘必须处于非挂载状态)		-p 自动修复错误	-r交互式修护错误

yum install xfs* parted -y #安装软件	#使用parted来对GPT磁盘操作(和fdisk一样)，-s选项不提示用户，？#查看帮助

parted -s /dev/sdb mklabel gpt  #创建新的磁盘标签 (分区表) 

parted -s /dev/sdb mkpart primary 0 -1	 或 parted -s /dev/sdb mkpart primary 0 100% #分配大小为100%

mkfs.xfs -f /dev/sdb

<img src="E:\Project\Textbook\linux云计算\assets\wps17-1682690115364-189.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps18-1682690115364-190.jpg" alt="img" style="zoom：80%;" /> 

Defaults,ro 只读（不能写）		df -Th显示磁盘使用情况		-T显示文件系统的形式	-h以人类可读方式显示

fdisk -l 查看显示硬盘信息		partprobe磁盘探测			lsblk 查看挂载列表		blkid查看块设备id号,文件系统类型

## LVM（Logical Volume Manager）逻辑卷制作

LVM是Linux环境中对磁盘分区进行管理的一种机制，是建立在硬盘和分区之上、文件系统之下的一个逻辑层，可提高磁盘分区管理的灵活性

PE(Physical Extend)　　基本单元：具有唯一编号的PE是能被LVM寻址的最小单元

PE的大小可以指定，默认为4MB。PE的大小一旦确定将不能改变，同一个卷组中的所有的物理卷的PE的大小是一直的

PV(Physical Volume)	物理卷		VG(Volume Group)	卷组			LV(Logical Volume)	逻辑卷

<img src="E:\Project\Textbook\linux云计算\assets\wps19-1682690115364-191.jpg" alt="img" style="zoom：67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps20-1682690115364-192.jpg" alt="img" style="zoom：67%;" /> 

1、制作选择物理卷 fdisk t  8e (修改为linux LVM类型)

2、pvcreate /dev/sda[5-7] 初始化磁盘或分区以供LVM使用

3、vgcreate -s 16M miantuan /dev/sda[5-7] 增加一个miantuan物理卷组			-s 指定PE的大小

4、lvcreate miantuan -L 500M -n baozi		从现有物理卷组miantuan中创建逻辑卷baozi	-L指定大小

\#Lvcreate -l 100%free miantuan -n jiaozi	把miantuan100%给jiaozi  ，		-l 以PE个数为单位也可以是百分比

改回物理卷：umount -->修改注册表fsable-->lvremove用于删除指定LVM逻辑卷组-->pvremove删除指定的物理卷

​		pvs/ vgs /lvs 简要显示物理卷/卷组/逻辑卷 使用率

逻辑卷组管理

扩展：添加新硬盘/dev/sdb-->更改类型-->pvcreate制作成物理卷-->vgextend vg0 /dev/sdb将新硬盘添加至逻辑卷组

改名：vgrename vg0 sss 更改逻辑卷组名字

逻辑卷组迁移：1、vgrename vg0 sss 更改逻辑卷组名字，防止新机器上相同名		2、umount /mnt/lll 卸载逻辑卷挂载点

3、vgchange -an vg0 禁用逻辑卷组						4、vgexport vg0 导出逻辑卷组

新机器上操作：1、vgimport vg0 导入逻辑卷		2、vgchange -ay vg0 激活逻辑卷

逻辑卷管理	注：xfs格式只能扩不能缩

xfs_growfs /mnt/baozi/ 重定义baozi逻辑卷文件系统的大小（只适用xfs）

resize2fs	/dev/vg0/baozi 重定义文件系统的大小（除xfs格式）

拓展：lvextend -Lr +5G /dev/vg0/baozi 添加5G空间到baozi逻辑卷	-r 同步文件系统

缩减：1、umount 卸载		2、fsck /dev/vg0/baozi 对设备整体进行检测

3、resize2fs /dev/vg0/baozi 1G 重定义文件系统的大小（除xfs格式）

4、lvreduce -L 1G /dev/vg0/baozi 缩减成指定大小1G		5、mount -a 刷新

调整逻辑卷的大小：lvresize -L 700M /dev/vg0/baozi

删除VG中的PV：1、pvmove /dev/vg0/ 转移数据（以PE格式），默认随机自动转移到可用其他物理卷

2、lvremove /dev/vg0/baozi	删除逻辑卷

3、vgreduce vg0 /dev/sda3 缩减逻辑卷到最小，即从卷组删除逻辑卷

改名：lvrename /dev/vg0/baozi /dev/vg0/sss

快照：lvcreate -n baozi-snapshot -s -L 1G /dev/vg0/baozi -p r	//-s创建快照，-p r 只读属性 或 挂载使用mount -o ro

//xfs格式原逻辑卷和快照逻辑卷的UUID一样（即不能同时挂载)，或使用-nouuid选项忽略UUID

快照恢复：1、umount取消逻辑卷挂载和快照逻辑卷挂载

2、lvconvert --merge /dev/vg0/baozi-snapshot 使用快照卷还原，同时快照逻辑卷会自动消失

或者此步骤挂载快照逻辑卷

## 制作交换分区

1、制作选择物理卷 fdisk t  82 (修改为linux swap类型，逻辑卷可直接设置)

2、mkswap /dev/sda3给sda3赋予swap类型(贴标签相当于格式化)

3、swapon \ swapoff /dev/sda3 启用 \ 关闭		swapon 显示已使用的虚拟内存量，也可查看交换分区

4、/etc/fsable 永久挂载交换分区（没有挂载点，挂载点为swap）

Free -m 查看内存使用量	-m以MB为单位显示内存使用情况	-h以合适的单位显示内存使用情况，最大为3位数

## 归档备份

dump只能对ext3,ext4进行备份			Xfsdump 	Xfsrestore对xfs进行备份

dump -0u -f /srv/data.dp /mnt/5将5以完全归档的方式全部备份成data.dp

-u显示备份详细信息，比如时间，内容			-f备份成指定文件

dump -1备份	-0更改过的数据		 -2备份（-1）更改过的数据

restore -tf /srv/data.dp		查看归档信息

cd /mnt/5  | Restore -rf /srv/data.dp	会在当前位置释放归档文件

<img src="E:\Project\Textbook\linux云计算\assets\wps21-1682690115364-193.jpg" alt="img" style="zoom:67%;" /> 

cfsdump 也是归档			Xfsrestore	恢复

## Raid 磁盘阵列

提供多个磁盘组合成一个阵列来提供更好的性能、冗余、提高IO能力：磁盘并行读写 、提高耐用性：磁盘冗余来实现

RAID实现的方式：

外接式：通过拓展卡提供适配能力	内接式：主板集成RAID控制器，安装OS在BIOS里配置	软件式：通过OS实现

RAID 0：读写性能提升，两块盘能同时写入、读取、但无任何容错功能，需要至少两块硬盘

RAID 1：着重安全性，两块硬盘执行同步式的操作，存放不追求速度的重要数据，使用率1/n

RAID 2/3：是raid 0的升级版，加入了错误纠正码，但写入变慢

<img src="E:\Project\Textbook\linux云计算\assets\wps22-1682690115364-194.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps23-1682690115364-195.jpg" alt="img" style="zoom:50%;" /> 

RAID 5：最快最安全，每块盘平均存放奇偶校验值，使用率（n-2）/2，且备份盘自动恢复数据

至少需要3块硬盘，允许1块损坏

RAID 1+ 0：先两块硬盘组合成raid 1，再将组合硬盘制作成raid 0格式

JBOD：将多块磁盘空间合并成一个大的逻辑上连续的空间使用

<img src="E:\Project\Textbook\linux云计算\assets\wps24-1682690115365-196.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps25-1682690115365-197.jpg" alt="img" style="zoom:50%;" /> 

mdadm：为软RAID提供管理界面，为空余磁盘添加冗余，结合内核中的md（multi devices）,RAID设备名为/dev/md0等

mdadm[mode] <raid device> [options] <component-devices>

创建模式：-C 创建raid设备		-a {yes|no} 检测设备名称、添加磁盘

-l RAID级别	-n 指定设备数量	-x 指定空闲盘个数			-c 指定数据的chunk大小(默认64k)

管理模式：-r 移除设备			-f 模拟设备损坏			-S 停止RAID磁盘阵列

监控模式：-D 查看详细信息		-v 显示过程				-F 监控				-Q 查看摘要信息

-A 刷新激活raid		-G 增加工作盘			--zero-superblock删除磁盘上残留的raid信息

1、磁盘阵列fdisk  t  fd （修改为Linux RAID类型）

2、mdadm -C  /dev/md0 -a yes -n3 -x1 -l10 /dev/sda{7..10} 

创建raid1+0格式磁盘阵列用3个工作盘、1个备份盘	{}只支持1-9

mdadm -G /dev/md0 -n4 -a /dev/sda5 将sda5加入到md0磁盘阵列

reresize2fs /dev/md0 同步md0上所有磁盘的文件系统

<img src="E:\Project\Textbook\linux云计算\assets\wps26-1682690115365-198.jpg" alt="img" style="zoom：50%;" /> 

2048前面空间是放主引导分区





# systemd架构

新服务，在 /usr/lib/systemd/system 下新建服务脚本，`systemctl daemon-reload` #重新加载

```sh
vim /usr/lib/systemd/system/zdy.service

[Unit]
Description=描述
Environment=环境变量或参数(系统环境变量此时无法使用)
After=network.target

[Service]
Type=forking
EnvironmentFile=所需环境变量文件或参数文件
ExecStart=启动命令(需指定全路径)
ExecStop=停止命令(需指定全路径)
User=以什么用户执行命令

[Install]
WantedBy=multi-user.target
```

示例，配置文件主要放在/usr/lib/systemd/system目录，也可能在/etc/systemd/system目录

```sh
$ systemctl cat sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

## [Unit] 启动顺序与依赖关系

```sh
Description：当前服务的简单描述
Documentation：指定 man 文档位置
 
After：如果 network.target 或 sshd-keygen.service 需要启动，那么 sshd.service 应该在它们之后启动
Before：定义 sshd 应该在哪些服务之前启动
注意：After 和 Before 字段只涉及启动顺序，不涉及依赖关系。
 
Wants：表示 sshd.service 与 sshd-keygen.service 之间存在"弱依赖"关系，即如果"sshd-keygen.service"启动失败或停止运行，不影响 sshd.service 继续执行
Requires：表示"强依赖"关系，即如果该服务启动失败或异常退出，那么sshd.service 也必须退出
注意：Wants 字段与 Requires 字段只涉及依赖关系，与启动顺序无关，默认情况下是同时启动。
```

## [Service] 启动行为

```sh
EnvironmentFile：许多软件都有自己的环境参数文件，该字段指定文件路径
注意：/etc/profile 或者 /etc/profile.d/ 这些文件中配置的环境变量仅对通过 pam 登录的用户生效，而 systemd 是不读这些配置的。
systemd 是所有进程的父进程或祖先进程，它的环境变量会被所有的子进程所继承，如果需要给 systemd 配置默认参数可以在 /etc/systemd/system.conf  和 /etc/systemd/user.conf 中设置。
加载优先级 system.conf 最低，可能会被其他的覆盖。
 

Type：定义启动类型。可设置：simple，exec，forking，oneshot，dbus，notify，idle
simple(设置了 ExecStart= 但未设置 BusName= 时的默认值)：ExecStart 字段启动的进程为该服务的主进程
forking：ExecStart 字段的命令将以 fork() 方式启动，此时父进程将会退出，子进程将成为主进程

 
ExecStart：定义启动进程时执行的命令
上面的例子中，启动 sshd 执行的命令是 /usr/sbin/sshd -D $OPTIONS，其中的变量 $OPTIONS 就来自 EnvironmentFile 字段指定的环境参数文件。类似的，还有如下字段：
ExecReload：重启服务时执行的命令
ExecStop：停止服务时执行的命令
ExecStartPre：启动服务之前执行的命令
ExecStartPost：启动服务之后执行的命令
ExecStopPost：停止服务之后执行的命令


RemainAfterExit：设为yes，表示进程退出以后，服务仍然保持执行


KillMode：定义 Systemd 如何停止服务，可以设置的值如下：
control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉
process：只杀主进程
mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
none：没有进程会被杀掉，只是执行服务的 stop 命令


Restart：定义了退出后，Systemd 的重启方式。可以设置的值如下：
no（默认值）：退出后不会重启
on-success：只有正常退出时（退出状态码为0），才会重启
on-failure：非正常退出时（退出状态码非0），包括被信号终止和超时，才会重启
on-abnormal：只有被信号终止和超时，才会重启
on-abort：只有在收到没有捕捉到的信号终止时，才会重启
on-watchdog：超时退出，才会重启
always：不管是什么退出原因，总是重启


RestartSec：表示 Systemd 重启服务之前，需要等待的秒数
```

## [Install]

```sh
WantedBy：表示该服务所在的 Target(服务组)
```

# Shell编程

shell是一个命令解释器，它在操作系统的最外层，负责直接与用户对话，把用户的输入解释给操作系统，并处理各种各样的操作系统的输出结果，输出屏幕返回给用户，/etc/shells可查看当前所有的shell类型

shell脚本用途：1、有自动化运维常用命令，2、创建简单的应用程序，3、处理文本或文件，4、用于系统管理和故障排除

shel脚本：包含一些命令或声明有一定格式的文本.sh文件

格式要求首行 shebang机制

\#bin/bash	#!/usr/bin/python		#!/usr/bin/perl

检查脚本方式：sh -n不执行脚本，仅检查脚本中的语法问题		-v将执行过的脚本命令打到屏幕上

运行脚本方式：1、bash / sh+脚本			  2、将脚本放入PATH命令对用的环境（echo $PATH查看）目录中

  3、chmod+x执行脚本test.sh		4、./test.sh执行

反斜杠  \  使后面的变量成为单纯的字符串

单引号‘’忽略所有的命令和特殊字符，将其中的内容都作为了字符串来

双引号“”可以使用特殊字符，包括', ", $, \,如果要忽略特殊字符，就可以利用\来转义，忽略特殊字符

反引号  `` 和 $（）都是获取命令输出的结果，而后输出

小括号（）开启子进程，即exit退出后父进程不受影响

常量：即只读变量		readonly -p查看所有常量			常量赋值：readonly name=value

函数：【function】函数名【()】{ 代码段；[return int]}

## 变量

执行顺序:`/etc/profile-->/etc/profile.d/*.shー->~/. bash_profile-->~/.bashrc-->/etc/bashrc`

/etc/profile 或 /etc/bashrc系统级环境变量文件			 ~/.profile或~/.bashrc用户级环境变量文件

【变量引用前加$，命令无法识别变量则加$】

**自定义变量**：自己定义的变量，例如：a='ll' (即将命令输出的结果进行赋值)，echo "$a" (双引号保留原格式)

**环境变量**：Linux系统已定义的变量，例如：$PATH, $HOME 等..., 这类变量我们可以直接使用

env \ export \ declare -x	\printenv 显示当前环境变量

export name=value		添加临时环境变量，变量声明、赋值

alias  qstat='/bin/ps -Ao pid,tt,user,fname,rsz'	将环境变量进行别名，改名字

unset删除指定临时环境变量（bash 刷新环境变量）

永久命令删除，三步		1vim 进去删除	2刷新环境变量	3unalias 删除命令		source/ 	重新加载文件

**参数变量**：$0脚本名，$1第一个参数，$2，$3，${10}...表示执行脚本后传递给脚本文件的第n个参数,花括号{}将一整体概括

**特殊变量**：

$?	执行脚本返回的值，0或非0值			$#   输入的参数个数		$$	查看当前进程PID号

$@  所有参数		$*  所有的参数，全部参数为一个字符串		$RANDOM	产生随机数（0-32767）			 

${#变量名} 表示变量的长度				 ${#变量名[@]} 表示数组的个数

**特殊字符**：

？至少匹配一次字符							*代表0或重复多个前面的字符

；分号，连续运行命令							. 代表一定有一个任意字符

\# 表示注释									.*就代表零个或多个任意长度的字符

| 管道(将前一命令的标准输出传递到下一命令的标准输入)，正则表达式中表示或者		&将命令放到后台执行

() 开启shell子进程，会继承父shell的变量			(()) 算术运算=-*/或比较大小<>

[] 通配符和正则中表示匹配括号中的任意一个字符、条件测试表达式、数组中下标括号

[[]] 双方括号字符串比较测试						{} 通配符扩展 abc{1,2,3}，将分割的多个子shell进行合并

\>或<:将字符串转化为ASCll值然后比较其大小		=~：正则匹配，用来判断其左侧的参数是否符合右边的要求

==：测试是否相等，相等为真，否则为假			！=：测试是否不等，不等为真，否则为假

-n：判断字符是否为空，空为真，不空为假			-z：判断字符是否为不空，不空为真，空则为假

^[0-9]+$：以^开头数字[0-9]且+至少一个$结尾		\<...\> 这组符号在规则表达式中，被定义为"边界"的意思

命令中的与或非：&& 与，前面执行成功才往后执行		||或，前面错误才执行后面		 !非，取判断的相反值

判断中的与或非：&		|	！

$(())或$[]可以进算数运算	``不可以

**判断**：<= 小于等于(需要双括号),如:(("$a" <= "$b"))  		>= 大于等于(需要双括号),如:(("$a" >= "$b"))

大于 -gt (greater than)					小于 -lt (less than)			大于或等于 -ge (greater than or equal)

小于或等于 -le (less than or equal)		不相等 -ne （not equal）	相等 -eq （equal）

**shell脚本内的特殊命令**：

echo $-：查看当前set环境功能		$-代表的是当前Bash的运行选项，这些Bash选项控制着Bash运行时的行为

h，hash进行查看：缓存使用命令的路径及使用次数

i，interactive-comments：交互式的shell环境

m，monitor：打开监控模式，可以通过job control来控制进程的停止、继续、后台或者前台执行

B，braceexpand：{}开启大括号拓展

H，history：开启历史命令

set：显示所有变量及函数，用来定制shell环境			set +h删除hash功能		set --清空所有

**参数**

-u：引用一个没有设置的变量时，显示错误信息，等同set -o nounset

-e：如果一个命令返回值非0退出状态值（失败）就退出，等同set -o errexit

-x：执行命令时，输出该命令和参数

-o：查看当前set命令生效的设置选项

sleep命令：休眠时间，默认为秒1

shift	命令：是shell脚本中使位置参数左移的命令，后可加左移位数，默认为1

break语句：（跳出循环）

在for、while、until等循环语句中，用于跳出当前所在的循环体，执行循环体后的语句

continue语句：（跳出本次循环）

在for、while、until等循环语句中，用于跳出循环体内余下的语句，重新判断条件以便执行下一次循环

let 命令是 BASH 中数字运算工具，用于执行一个或多个表达式（let 进行赋值，数字或字符\字符串）

help let查看系统支持的算数运算

test命令是 Shell 内置命令，用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试	hele test查看帮助

expr命令进行算术运算，格式比较严格		a=`expr 1 \* 4`		a=`expr $a \* 4`

exit 1函数退出当前程序

三目运算符：`a=$([ "$b" == 5 ] && echo "$c" || echo "$d")`

<img src="E:\Project\Textbook\linux云计算\assets\wps27-1682690115365-199.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps28-1682690115365-200.jpg" alt="img" style="zoom:67%;" /> 

## For循环

for的一些用法		例：for i in “file1” “file2” “file3”

| in /boot/*                       | in /etc/*.conf                 | in {1…10}            | in $( ls )                               |
| -------------------------------- | ------------------------------ | -------------------- | ---------------------------------------- |
| in $(< file)	in $(seq 5 -1 1) | in $(seq -w 10) -->等宽的01-10 | in $(cat /srv/grade) | in “$@” -->取所有位置参数，可简写为for i |


```sh
function printInfo (){
	echo -n "Your choice is "
}
```


```sh
for a in {10..15}
do
	printInfo;		#使用函数
done	
```

## If判断语句

[ ]&& ----快捷if				`[ "$?" == 0 ] && echo "success" >>/tmp/install.log`

**单分支if语句**

```sh
if	判断条件; then
	输出结果命令
fi
```

```sh
if ! (ping -c4 -i0.1 192.168.0.22 &>/dev/null) ; then 
	echo "host22 is down" #如果ping22号主机成功的话，会输出host22 is down
fi
```

**双分支if语句**					

```sh
if 判断条件; then
	输出结果命令1	
else
	输出结果命令2
fi
```

```sh
for a in {1..10}
do
	if  ! (ping -c4 -i0.1 192.168.0.$a &>/dev/null) ; then
		echo "host$a is down" &>>/ tmp/down
	else
		echo "host$a is up" &>>/tmp/up
	fi
done
```

**多分支if语句**

```sh
if 	判断条件1; then
	输出结果命令1
elif	判断条件2; then
	输出结果命令2
elif	判断条件3; then
	输出结果命令3
else
	输出结果命令2
fi
```

```sh
if [ "$?" == 0 ] | [ "$?" == 1 ];then	#多个判断条件
	echo "welcome to joinlabs!
elif [ "$1" == "redhat" ] ; then
	echo "this is redhat classroom"
else
	echo "byebye"
fi
```

例18、将zhubajie. sunwukong. tangseng重定向到/tmp/userdb中
创建一个添加用户脚本。在本机上创建一 个脚本,名为/tmp/user.sh, 此脚本能实现为本机系统创建本地用户,
并且这些用户的用户名来自一个包含用户列表的文件(userdb) .同时满足以下要求:
此脚本要求提供一 个参数，此参数就是包含用户列表的文件
如果没有提供参数，此脚本应该给出下面的提示信息Usage:/tmp/userdb, 然后退出并返回相应的值
如果提供一 个不存在的文件名，此脚本应该给出下面的提示信息Input file not found, 然后退出并返回相应的值
此脚本需要为用户设置密码，密码为redhat,且都为非交互式登录shell

```sh
#! /bin/bash
if [ $# -eq 0 ];then
	echo "Usage：/tmp/userdb"
elif [ $1 == "/tmp/userdb" ];then
	for a in、cat /tmp/userdb"
	do
		useradd -s /sbin/nologin $a;
		echo redhat| passwd -- stdin $a;
	done
else
	echo "Input file not found"
fi 
```

## Read语句

read能够和用户进行交互式的读取数据,要保证脚本能够根据用户的反馈数据进行下一步操作

| read                 | 参数                                                    |                      |
| -------------------- | ------------------------------------------------------- | -------------------- |
| -p：(提示语)提示输入 | -n：(限制字符个数)计数输入                              | -t(等待时间)计时输入 |
| -s：(不回显)隐藏输入 | -d：delimiter，读到指定字符即退出，例：-dp，读到q即退出 |                      |

读文件：每次read命令都会读取文件的“一行”内容。当文本没有可读的行时，read命令将以非零状态退出

name是个变量，Read语句一般是和if语句连用的

```sh
while read line
do
	if [[ $line =~ "/sbin/nologin" ]];then
	echo "$line"
	fi
done < /etc/passwd
```

```sh
read -dq -s -p "please input YES or NO：" -t 10 -n3 a
if [[ $a =~ ^[Yy][Ee][Ss]$ ]];then		#模糊匹配
	echo "yes"
fi
```



## While循环语句

```sh
while 条件  //While ：	//无限循环
do
done
```

```sh
count=1							
cat /etc/passwd | while read a
do
 	echo "$count  $a"
	let count++
done
```



## Case语句

<img src="E:\Project\Textbook\linux云计算\assets\wps31-1682690115365-203.jpg" alt="img" style="zoom:67%;" />

```sh
read -n 1 -p "Do you want to cont inue [Y/N]?" answer
case $answer in
	Y)
		echo " fine , continue"
	;;
	N)
		echo " ok, goodbye"
	;;
	*)
		echo "error choice"
esac
```

```sh
case $l in 
	joinlabs)
		echo "welcome to joinlabs !"
	;;
	redhat)
		echo "this is redhat class room'
	;;
	*)
		echo " byebye'
esac
```

 

## select语句

```sh
select variable in list 		select name in ylt yyy lll ttt
do								do
	循环体命令				    	echo $name
done							done
```

select 循环执行后会出现菜单项等待用户选择（不会自动循环所有变量列表），而用户输入的只能是菜单项前面的数字序号，每输入一次对应的序号就会执行一次循环，直到变量后面对应列表取完为止

# 开机破密码

<img src="E:\Project\Textbook\linux云计算\assets\wps34-1682690115365-206.jpg" alt="img" style="zoom：67%;" /> 

Grub2里面有，菜单，核心，默认走第一个核心

Rc.local一定会加载的文件

![img](E:\Project\Textbook\linux云计算\assets\wps35-1682690115365-207.jpg) 

Menu菜单编辑第一块核心按e编辑

删除vmlinuz核心的Rhgh quiet (防止光标出不来)改为init=/bin/bash或者init=/bin/sh

Ctrl+x启动

Mount -o remount,rw / #权限改为可读写

Echo 123456 | passwd --stdin root

\> /.autorelabel #清空SElinux安全策略

Exec /sbin/init #可执行文件

\##chroot /usr/bin/passwd

![img](E:\Project\Textbook\linux云计算\assets\wps36-1682690115365-208.jpg) 

<img src="E:\Project\Textbook\linux云计算\assets\wps37-1682690115366-209.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps38-1682690115366-210.jpg" alt="img" style="zoom:67%;" /> 

systemctl get-default	查看运行级别

ll  /lib/system/system	| grep run	查找级别

![img](E:\Project\Textbook\linux云计算\assets\wps39-1682690115366-211.jpg) 

查看默认级别号

# 生成复杂密码

Linux系统随机生成复杂密码方法：使用 mkpasswd 命令生成随机密码

  首先需要安装 expect 包，安装方式为：yum install expect -y

mkpasswd 使用 mkpasswd 命令来生成随机密码了（区分大小写）：

```
-l  生成密码的长度，默认是 9 位，不同版本默认长度可能是不一样	-d  生成密码中包含数字的位数，默认是 2 位
-c  生成密码中包含小写字母的位数，默认是 2 位			-C  生成密码中包含大写字母的位数，默认是 2 位
-s  生成密码中包含特殊字符的位数，默认是 1 位
```

示例输出：$ mkpasswd -l 20 -d 5 -c 5 -C 5 -s 2		-->	9i'aR:iJSt03t5uU3Drl

使用 date 命令的 MD5SUM 值作为随机密码

可以通过 data 指令获取时间后，计算 md5 值，然后截取其中的一部分当做随机密码

 

操作示例：$ date | md5sum | cut -b 10-20	-->	464dddf2644

openssl工具生成强密码操作示例：$ openssl rand -base64 8		-->	vZfr+eeIxeE=		#生成 8 位随机密码

# vim常用模式

1、命令模式	2、插入模式	3、底行模式	4、可视化模式，命令模式按v进入	5、替换模式，命令模式下按r进入

vim 编辑模式下

：r  !  ls  /etc/sys读取（r）终端(!)下命令ls的输出

R! echo `pwd`		(  ` ` 将命令输出结果（/etc/yum.repo.d/路径）)

2,6 %s/\//9/g	 2到6行的/ 全部换成9（vim里面支持 \转移符号）

**按键**

```
强制退出(不保存) ：普通模式下::q!
保存:	4.1 另存为:	:w 文件名			4.2 保存&退出:	:wq
复制		5.1 复制当行到系统剪贴板:	"+yy 这个是真+			5.2 复制所选至系统剪贴板:	"+y
```

```
8.1 删除所在光标下的#行:	#dd (#自然数)				8.2 删除所在光标上的#行:	#dk
8.3 向下删除至底：dG						  		8.4 向上删除至顶：dgg
撤销以及反撤销	9.1 撤销：普通模式下 u				   9.2 反撤销：普通模式下 Ctrl+r
```

批量选择:

```
选择+复制 普通模式下:	选择：v		复制：y		复制当前行至vim剪贴板:	yy
```

黏贴:	
```
6.1 系统剪贴板:	shift+ctrl+v				vim剪切板粘贴至下一行:	p
普通模式下删除/剪切:	dd
10.3 选中某个方格：Ctrl+v
```

```
屏幕滚动:向下↓ 一页:Ctrl+f ; 向上↑ 一页:Ctrl+b		向下↓ 半页:Ctrl+d ; 向下↓ 半页:Ctrl+u
```

移动光标(上下左右)：

```
向下：G		向上：gg						本行头：0或^ 本行尾：$
迅速移动：ctrl+箭头(跳过空格)或Shift+箭头(跳过符号)
移至顶部：gg 移至底部：G
5gg或者5G：光标跳至第五行
```

在vim中有3中方法可以跳转到指定行（首先按esc进入命令行模式）：

```
1、ngg/nG （跳转到文件第n行，无需回车）
2、:n （跳转到文件第n行，需要回车）
3、vim +n filename （在打开文件后，跳转到文件的第n行）
```

搜索:

13.3 特殊搜索

^	放在字符串前面，匹配行首的字；			$	放在字符串后面，匹配行尾的字；

<	匹配一个字的字头；					>	匹配一个字的字尾；

.	匹配任何单个正文字符；					[str]	匹配 str 中的任何单个字符；

[^str]	匹配任何不在 str 中的单个字符；		[a-b]	匹配 a 到 b 之间的任一字符；

*	匹配前一个字符的 0 次或多次出现；		\	转义后面的字符

13.4 从#1行到#2行,搜索替换x为y

在每一行前加@符号（操作要在在光标前进入编辑模式）

```
Ctrl + v 		Shif + a		@		Esc			esc
```

:#1,#2s/x/y/g (#1 #2 为自然数)

:#1,#2s/x/y/gc (替换前确认confirm)

 

自动对齐 ：shift +v(选中范围)   =（进行对其）

 

代码自动补全：ctrl + p

非编辑模式下查看man手册：shift +k

 

设置set:

```
显示行号:	:set nu				取消行号:	:set nonu			设置缩进:	:set tabstop=#	
自动缩进:	:set autoindent		显示名称:	:set laststatus=2	显示行符:	:set list	
取消行符:	:set nolist
```

多窗口:

15.1 开出新的窗口:	sp				15.2 切换窗口:	ctrl w 上下键

```sh
vim ~/.vimrc		#vim脚本编辑工具
set ignorecase
set cursorline
set autoindent
autocmd BufNewFile *.sh exec ":call SetTitle()"
func SetTitle()
	if expand("%:e") == 'sh'
	call setline(1,"#!/bin/bash")
	call setline(2,"#*******************************")
	call setline(3,"#Author：liulengbo")
	call setline(4,"#weixin：13086119057")
	call setline(5,"#Date：".strftime("%Y-%m-%d"))
	call setline(6,"#Description:	")
	call setline(7,"#*******************************")
	call setline(8,"")
	endif
endfunc
autocmd BufNewFile * normal G
```

 

gcc使用

gcc

-Wall  输出所有警告 