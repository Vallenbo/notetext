

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
| /usr/local  | 用户自行安装的软件                                          |
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

# 网络分析工具

## tcpdump

**tcpdump** 是Linux系统下的一个强大的命令，可以将网络中传送的数据包完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

### tcpdump 命令格式介绍

首先我们对Linux系统下tcpdump命令格式进行一个介绍说明，在下面的使用教程中讲到tcpdump命令示例时，您就可以参考tcpdump命令参数说明进行学习。

**语法：**

```text
tcpdump [ -AdDefIJKlLnNOpqRStuUvxX ] [ -B buffer_size ] [ -c count ]
               [ -C file_size ] [ -G rotate_seconds ] [ -F file ]
               [ -i interface ] [ -j tstamp_type ] [ -m module ] [ -M secret ]
               [ -Q|-P in|out|inout ]
               [ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
               [ -W filecount ]
               [ -E spi@ipaddr algo:secret,...  ]
               [ -y datalinktype ] [ -z postrotate-command ] [ -Z user ]
               [ expression ]
```

**命令行参数介绍：**

- -A 以ASCII格式打印出所有分组，并将链路层的头最小化。
- -c 在收到指定的数量的分组后，tcpdump就会停止。
- -C 在将一个原始分组写入文件之前，检查文件当前的大小是否超过了参数file_size中指定的大小。如果超过了指定大小，则关闭当前文件，然后在打开一个新的文件。参数 file_size的单位是兆字节（是1,000,000字节，而不是1,048,576字节）。
- -d 将匹配信息包的代码以人们能够理解的汇编格式给出。
- -dd 将匹配信息包的代码以c语言程序段的格式给出。
- -ddd 将匹配信息包的代码以十进制的形式给出。
- -D 打印出系统中所有可以用tcpdump截包的网络接口。
- -e 在输出行打印出数据链路层的头部信息。
- -E 用spi@ipaddr algo:secret解密那些以addr作为地址，并且包含了安全参数索引值spi的IPsec ESP分组。
- -f 将外部的Internet地址以数字的形式打印出来。
- -F 从指定的文件中读取表达式，忽略命令行中给出的表达式。
- -i 指定监听的网络接口。
- -l 使标准输出变为缓冲行形式。
- -L 列出网络接口的已知数据链路。
- -m 从文件module中导入SMI MIB模块定义。该参数可以被使用多次，以导入多个MIB模块。
- -M 如果tcp报文中存在TCP-MD5选项，则需要用secret作为共享的验证码用于验证TCP-MD5选选项摘要（详情可参考RFC 2385）。
- -n 不把网络地址转换成名字。
- -N 不输出主机名中的域名部分。例如，`link.linux265.com` 只输出`link`。
- -t 在输出的每一行不打印时间戳。
- -O 不运行分组分组匹配（packet-matching）代码优化程序。
- -P 不将网络接口设置成混杂模式。
- -q 快速输出。只输出较少的协议信息。
- -r 从指定的文件中读取包(这些包一般通过-w选项产生)。
- -S 将tcp的序列号以绝对值形式输出，而不是相对值。
- -s 从每个分组中读取最开始的snaplen个字节，而不是默认的68个字节。
- -T 将监听到的包直接解释为指定的类型的报文，常见的类型有rpc远程过程调用）和snmp（简单网络管理协议；）。
- -t 不在每一行中输出时间戳。
- -tt 在每一行中输出非格式化的时间戳。
- -ttt 输出本行和前面一行之间的时间差。
- -tttt 在每一行中输出由date处理的默认格式的时间戳。
- -u 输出未解码的NFS句柄。
- -v 输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息。
- -vv 输出详细的报文信息。
- -w 直接将分组写入文件中，而不是不分析并打印出来。
- -x 以16进制数形式显示每一个报文 (去掉链路层报头) . 可以显示较小的完整报文, 否则只显示snaplen个字节.
- -xx 以16进制数形式显示每一个报文（包含链路层包头）。
- -X 以16进制和ASCII码形式显示每个报文（去掉链路层报头）。
- -XX 以16进制和ASCII吗形式显示每个报文（包含链路层报头）。
- -y 设置tcpdump 捕获数据链路层协议类型
- -Z 使tcpdump 放弃自己的超级权限(如果以root用户启动tcpdump, tcpdump将会有超级用户权限), 并把当前tcpdump的用户ID设置为user, 组ID设置为user首要所属组的ID

### tcpdump 命令使用示例

linux系统下执行tcpdump命令需要`root`账号或者具备`sudo`权限的账号，否则执行tcpdump命令说，系统会提示`tcpdump: no suitable device found`。

在下面的例子中，`-i eth0` 参数表示只抓取 eth0 接口数据包，不加`-i eth0` 是表示抓取所有的接口包括 `lo`。

01、抓取所有网络包，并在`terminal`中显示抓取的结果，将包以十六进制的形式显示。

```text
tcpdump
```

02、抓取所有的网络包，并存到 `result.cap` 文件中。

```text
tcpdump -w result.cap
```

03、抓取所有的经过`eth0`网卡的网络包，并存到`result.cap` 文件中。

```text
tcpdump -i eth0 -w result.cap
```

04、抓取源地址是`192.168.1.100`的包，并将结果保存到 `result.cap` 文件中。

```text
tcpdump src host 192.168.1.100 -w result.cap
```

05、抓取地址包含是`192.168.1.100`的包，并将结果保存到 `result.cap` 文件中。

```text
tcpdump host 192.168.1.100 -w result.cap
```

06、抓取目的地址包含是`192.168.1.100`的包，并将结果保存到 `result.cap` 文件中。

```text
tcpdump dest host 192.168.1.100 -w result.cap
```

07、抓取主机地址为 `192.168.1.100` 的数据包

```text
tcpdump -i eth0 -vnn host 192.168.1.100
```

08、抓取包含`192.168.1.0/24`网段的数据包

```text
tcpdump -i eth0 -vnn net 192.168.1.0/24
```

09、抓取网卡`eth0`上所有包含端口`22`的数据包

```text
tcpdump -i eth0 -vnn port 22
```

10、抓取指定协议格式的数据包，协议格式可以是「udp,icmp,arp,ip」中的任何一种,例如以下命令：

```text
tcpdump udp  -i eth0 -vnn
```

11、抓取经过 eth0 网卡的源 ip 是 192.168.1.100 数据包，`src`参数表示源。

```text
tcpdump -i eth0 -vnn src host 192.168.1.100
```

12、抓取经过 eth0 网卡目的 ip 是 192.168.1.100 数据包，`dest `参数表示目的。

```text
tcpdump -i eth0 -vnn dest host 192.168.1.100
```

13、抓取源端口是`22`的数据包

```text
tcpdump -i eth0 -vnn src port 22
```

14、抓取源`ip`是 `192.168.1.100` 且目的`ip`端口是`22`的数据包

```text
tcpdump -i eth0 -vnn src host 192.168.1.100 and dest port 22
```

15、抓取源`ip``192.168.1.100``22`

```text
tcpdump -i eth0 -vnn src host 192.168.1.100 or port 22
```

16、抓取源`ip``192.168.1.100``22`

```text
tcpdump -i eth0 -vnn src host 192.168.1.100 and not port 22
```

17、抓取源`ip`是`192.168.1.100`且目的端口是`22`，或源`ip`是`192.168.1.102`且目的端口是`80`的数据包。

```text
tcpdump -i eth0 -vnn ( src host 192.168.1.100 and dest port 22 ) or ( src host 192.168.1.102 and dest port 80 )
```

18、把抓取的数据包记录存到`/tmp/result`文件中，当抓取`100`个数据包后就退出程序。

```text
tcpdump –i eth0 -vnn -w /tmp/result -c 100
```

19、从`/tmp/result`记录中读取`tcp`协议的数据包

```text
tcpdump -i eth0  tcp  -vnn -r /tmp/result
```

20、想要截获所有`192.168.1.100`的主机收到的和发出的所有的数据包：

```text
tcpdump host 192.168.1.100
```

21、如果想要获取主机`192.168.1.100`除了和主机`192.168.1.101`之外所有主机通信的ip包，使用命令：

```text
tcpdump ip host 192.168.1.100 and ! 192.168.1.101
```

22、如果想要获取主机 `192.168.1.100` 接收或发出的 `telnet` 包，使用如下命令：

```text
tcpdump tcp port 23 host 192.168.1.100
```



## tcptrace数据包分析工具





## netperf网络带宽工具







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

iostat：监控系统输入/输出设备和CPU使用情况，可能需要用到的安装包 apt install sysstat -y

```
-d：显示磁盘统计信息。			-x 1：显示详细的拓展（磁盘）统计信息，包括平均等待时间和队列长度等。
-k：以KB为单位显示统计信息。	   -m：以MB为单位显示统计信息。
-t：显示时间戳。
```

time命令统计执行指定命令所花费的总时间

```
-f格式化时间输出		-a将显示信息追加到文件		-o将显示信息写入文件中
```

```sh
$time ls #统计ls命令执行所需总时间

real	0m0.002s #真实时间
user	0m0.001s #用户态时间
sys	0m0.000s	 #内核态时间
```

# nc

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
   # 接收方
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

# 检查远程用户登陆状态、时间、ip地址等信息

通过who命令查看在线远程登录的用户

```
[root@ftp local]# who
root     pts/1        2011-11-09 13:49 (192.168.101.122)
root     pts/2        2011-11-09 13:49 (192.168.101.122)
[root@ftp local]# arp 192.168.101.122Address                  HWtype  HWaddress           Flags Mask            Iface
localhost.168.192.in-ad  ether   00:22:71:dc:2e:0b   C                     eth0
```

能看到MAC地址，有了MAC地址就知道是谁了



# /proc目录下的操作系统信息





# hping3







# strace

系统调用：程序在执行期间,调用了由内核提供的函数时,称为系统调用。
库调用：由库提供的函数,称为库调用,是运行在内核之外的。使用Itrace命令跟踪库调用。程序库可以在不同程序之间共享。

```sh
$ yum install strace			#CentOS/EulerOS系统
$ apt-get install strace –y		#Ubuntu系统
```

利用strace命令跟踪系统调用,例如stace Is ,跟踪Is命令所执行的系统调用。

选项: 
-c + 命令，统计系统调用的次数,花费的时间和错误的系统调用数量。
-p指定某个存在的进程。
-f 跟踪由fork调用产生的子进程。
-e加上表达式，只关注具体指定的系统调用,例如:



# mpstat

mpstat是 Multiprocessor Statistics的缩写，是实时系统监控工具。其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。在多CPU系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。

mpstat [-P {cpu|ALL}] [internal [count]]

其中，各参数含义如下：

-P {cpu l ALL}  表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
 internal    相邻的两次采样的间隔时间
 count   采样的次数，count只能和delay一起使用
 使用mpstat命令
 1.直接使用mpstat命令：当mpstat不带参数时，输出为从系统启动以来的平均值。

2.使用mpstat -P ALL 5 2命令

```sh
mpstat -P ALL 5 2
```

表示每5秒产生一个报告，总共产生2个。

上图表示每5秒产生了2个关于处理器的统计数据报告，一共产生2个interval 的信息，然后再给出这2个interval的平均信息。默认时，输出是按照CPU 号排序。第一个行给出了从系统引导以来的所有活跃数据。接下来每行对应一个处理器的活跃状态。



## 输出参数含义

当没有参数时，mpstat则显示系统启动以后所有信息的平均值。有interval时，第一行的信息自系统启动以来的平均信息。从第二行开始，输出为前一个interval时间段的平均信息。

输出各参数含义：

```csharp
参数  释义  从/proc/stat获得数据
CPU 处理器ID   
%usr    在internal时间段里，用户态的CPU时间（%），不包含 nice值为负进程    usr/total*100
%nice   在internal时间段里，nice值为负进程的CPU时间（%）    nice/total*100
%sys    在internal时间段里，核心时间（%）   system/total*100
%iowait 在internal时间段里，硬盘IO等待时间（%）   iowait/total*100
%irq    在internal时间段里，硬中断时间（%）  irq/total*100
%soft   在internal时间段里，软中断时间（%）  softirq/total*100
%steal  显示虚拟机管理器在服务另一个虚拟处理器时虚拟CPU处在非自愿等待下花费时间的百分比   steal/total*100
%guest  显示运行虚拟处理器时CPU花费时间的百分比   guest/total*100
%gnice      gnice/total*100
%idle   在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间（%）    idle/total*100
```

CPU总的工作时间：

```undefined
total_cur = user + system + nice + idle + iowait + irq + softirq
total_pre = pre_user + pre_system + pre_nice + pre_idle + pre_iowait + pre_irq + pre_softirq
user = user_cur - user_pre
total = total_cur - total_pre
```

其中_ cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。

Note：

1. vmstat和mpstat 命令的差别：mpstat 可以显示每个处理器的统计，而 vmstat 显示所有处理器的统计。因此，编写糟糕的应用程序（不使用多线程体系结构）可能会运行在一个多处理器机器上，而不使用所有处理器。从而导致一个 CPU 过载，而其他 CPU 却很空闲。通过 mpstat 可以轻松诊断这些类型的问题。
2. vmstat中所有关于CPU的总结都适合mpstat。当您看到较低的 %idle 数字时，您知道出现了 CPU 不足的问题。当您看到较高的 %iowait 数字时，您知道在当前负载下 I/O 子系统出现了某些问题。





# top

Linux top 是一个在 Linux 和其他类 Unix 系统上常用的实时系统监控工具。它提供了一个动态的、交互式的实时视图，显示系统的整体性能信息以及正在运行的进程的相关信息。

```sh
top [-] [d delay] [q] [c] [S] [s] [i] [n] [b]
参数说明：
-d <秒数>：指定 top 命令的刷新时间间隔，单位为秒。			-n <次数>：指定 top 命令运行的次数后自动退出。
-p <进程ID>：仅显示指定进程ID的信息。						-u <用户名>：仅显示指定用户名的进程信息。
-H：在进程信息中显示线程详细信息。						   -i：不显示闲置（idle）或无用的进程。
-b：以批处理（batch）模式运行，直接将结果输出到文件。		  -c：显示完整的命令行而不截断。
-S：累计显示进程的 CPU 使用时间。
```



# pidstat 概述

用于监控全部或指定进程的cpu、内存、线程、设备IO等系统资源的占用情况。pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

**pidstat 安装**

pidstat 是sysstat软件套件的一部分，sysstat包含很多监控linux系统状态的工具，它能够从大多数linux发行版的软件源中获得。

```sh
apt-get install sysstat -y
yum install sysstat -y
```

## pidstat 示例

pidstat 的用法：

```css
pidstat [ 选项 ] [ <时间间隔> ] [ <次数> ]
```

如下图：

![img](https:////upload-images.jianshu.io/upload_images/2843224-8b6e338a330cd593.png?imageMogr2/auto-orient/strip|imageView2/2/w/758/format/webp)

常用的参数：

- -u：默认的参数，显示各个进程的cpu使用统计
- -r：显示各个进程的内存使用统计
- -d：显示各个进程的IO使用情况
- -p：指定进程号
- -w：显示每个进程的上下文切换情况
- -t：显示选择任务的线程的统计信息外的额外信息
- -T { TASK | CHILD | ALL }
   这个选项指定了pidstat监控的。TASK表示报告独立的task，CHILD关键字表示报告进程下所有线程统计信息。ALL表示报告独立的task和task下面的所有线程。
   注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。
- -V：版本号
- -h：在一行上显示了所有活动，这样其他程序可以容易解析。
- -I：在SMP环境，表示任务的CPU使用率/内核数量
- -l：显示命令名和所有参数

## 示例一：查看所有进程的  CPU 使用情况（ -u -p ALL）

```undefined
pidstat
pidstat -u -p ALL
```

pidstat 和 pidstat -u -p ALL 是等效的。
pidstat 默认显示了所有进程的cpu使用率。

![img](https:////upload-images.jianshu.io/upload_images/2843224-bee9066f2689e541.png?imageMogr2/auto-orient/strip|imageView2/2/w/803/format/webp)

#### 详细说明

- PID：进程ID
- %usr：进程在用户空间占用cpu的百分比
- %system：进程在内核空间占用cpu的百分比
- %guest：进程在虚拟机占用cpu的百分比
- %CPU：进程占用cpu的百分比
- CPU：处理进程的cpu编号
- Command：当前进程对应的命令

## 示例二:  cpu使用情况统计(-u)

```undefined
pidstat -u
```

使用-u选项，pidstat将显示各活动进程的cpu使用统计，执行”pidstat -u”与单独执行”pidstat”的效果一样。

## 示例三： 内存使用情况统计(-r)

```undefined
pidstat -r
```

使用-r选项，pidstat将显示各活动进程的内存使用统计：

![img](https:////upload-images.jianshu.io/upload_images/2843224-dd469d8d55e58237.png?imageMogr2/auto-orient/strip|imageView2/2/w/807/format/webp)

- PID：进程标识符
- Minflt/s:任务每秒发生的次要错误，不需要从磁盘中加载页
- Majflt/s:任务每秒发生的主要错误，需要从磁盘中加载页
- VSZ：虚拟地址大小，虚拟内存的使用KB
- RSS：常驻集合大小，非交换区五里内存使用KB
- Command：task命令名

## 示例四：显示各个进程的IO使用情况（-d）

```undefined
pidstat -d
```

![img](https:////upload-images.jianshu.io/upload_images/2843224-6ce7d9ed78be38a9.png?imageMogr2/auto-orient/strip|imageView2/2/w/670/format/webp)

报告IO统计显示以下信息：

- PID：进程id
- kB_rd/s：每秒从磁盘读取的KB
- kB_wr/s：每秒写入磁盘KB
- kB_ccwr/s：任务取消的写入磁盘的KB。当任务截断脏的pagecache的时候会发生。
- COMMAND:task的命令名

## 示例五：显示每个进程的上下文切换情况（-w）

```undefined
pidstat -w -p 2831
```

![img](https:////upload-images.jianshu.io/upload_images/2843224-e49e80ecb5d6526a.png?imageMogr2/auto-orient/strip|imageView2/2/w/657/format/webp)

- PID:进程id
- Cswch/s:每秒主动任务上下文切换数量
- Nvcswch/s:每秒被动任务上下文切换数量
- Command:命令名

## 示例六：显示选择任务的线程的统计信息外的额外信息 (-t)

```undefined
pidstat -t -p 2831
```

![img](https:////upload-images.jianshu.io/upload_images/2843224-d77a75f30696eab7.png?imageMogr2/auto-orient/strip|imageView2/2/w/799/format/webp)

- TGID:主线程的表示
- TID:线程id
- %usr：进程在用户空间占用cpu的百分比
- %system：进程在内核空间占用cpu的百分比
- %guest：进程在虚拟机占用cpu的百分比
- %CPU：进程占用cpu的百分比
- CPU：处理进程的cpu编号
- Command：当前进程对应的命令

## 示例七：pidstat -T

```undefined
pidstat -T TASK
pidstat -T CHILD
pidstat -T ALL
```

TASK表示报告独立的task。
CHILD关键字表示报告进程下所有线程统计信息。
ALL表示报告独立的task和task下面的所有线程。

> 注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。

![img](https:////upload-images.jianshu.io/upload_images/2843224-8c8d8247678b6c91.png?imageMogr2/auto-orient/strip|imageView2/2/w/716/format/webp)

- PID:进程id
- Usr-ms:任务和子线程在用户级别使用的毫秒数
- System-ms:任务和子线程在系统级别使用的毫秒数
- Guest-ms:任务和子线程在虚拟机(running a virtual processor)使用的毫秒数
- Command:命令名



# sar工具

[sar](https://link.zhihu.com/?target=http%3A//lovesoo.org/tag/sar)（System Activity Reporter系统活动情况报告）是目前linux上最为全面的系统性能分析工具之一，可以从多方面对系统的活动进行报告，包括：文件的读写情况、系统调用的使用情况、磁盘、I/O、CPU效率、内存使用状况、进程活动及IPC有关的活动等。

```sh
apt install searchandrescue -y 
```

## sar命令常用格式

```sh
sar [options] [-A] [-o file] [ <interval> [ <count> ] ]
```

options 为命令行选项：

​	-o file表示将命令结果以二进制格式存放在文件中，file 是文件名。

​	interval为采样间隔，count为采样次数，默认值是1;

​	t为采样间隔，n为采样次数，默认值是1；

​	sar命令options常用选项如下：

```sh
-A：所有报告的总和									-u：输出CPU使用情况的统计信息
-B：显示换页状态；									-d：输出每一个块设备的活动信息
-v：输出inode、文件和其他内核表的统计信息			  -r：输出内存和交换空间的统计信息
-b：显示I/O和传送速率的统计信息-R：输出内存页面的统计信息
-y：终端设备活动情况
-w：输出系统交换活动信息							  -e：设置显示报告的结束时间
-f：从指定文件提取报告							   -i：设状态信息刷新的间隔时间
-p：报告每个CPU的状态								-q：平均负载分析
```





# sysbench



# sysctl

**sysctl命令** 被用于在内核运行时动态地修改内核的运行参数，可用的内核参数在目录`/proc/sys`中。它包含一些TCP/ip堆栈和虚拟内存系统的高级选项， 这可以让有经验的管理员提高引人注目的系统性能。用sysctl可以读取设置超过五百个系统变量。

```sh
sysctl(选项)(参数)
-n：打印值时不打印关键字；						-N：仅打印名称；
-w：当改变sysctl设置时使用此项；				-p：从配置文件“/etc/sysctl.conf”加载内核参数设置；
-a：打印当前所有可用的内核参数变量和值；		    -A：以表格方式打印当前所有可用的内核参数变量和值。

[root@linuxcool ~]# sysctl dev.cdrom.debug		# 读取一个指定系统内核参数变量的值
dev.cdrom.debug = 0

[root@linuxcool ~]# sysctl -p /etc/sysctl.conf # 生效指定文件内参数
```





# perf

perf 命令(performance 的缩写)，是[Linux](https://link.zhihu.com/?target=https%3A//so.csdn.net/so/search%3Fq%3DLinux%26spm%3D1001.2101.3001.7020)系统提供的性能分析工具集，包含多种子工具，能够监测多种硬件及软件性能指标，包括cpu、内存、io等，这些可监测指标我们称为event。

perf架构图

<img src="./assets/image-20230917100432301.png" alt="image-20230917100432301" style="zoom: 33%;" />

Perf List：查看当前系统支持的性能事件。

利用perf剖析程序性能时，需要指定当前测试的性能时间。性能事件是指在处理器或操作系统中发生的，可能影响到程序性能的硬件事件或软件事件

```sh
-e：指定性能事件							   -d 选项用于启用perf工具的统计功能
-a：收集所有CPU上的性能统计信息				   -C：显示在指定CPU上的性能统计信息
-p：指定进程PID								-t：指定线程TID
-K：隐藏内核统计信息							 -U：隐藏用户空间的统计信息
-s：指定待解析的符号信息
```

```sh
perf top：#显示当前系统的性能概况，包括使用CPU最多的函数和符号。
perf record：#运行一个命令并记录其性能数据到perf.data文件中。 
	-o 指定文件名
perf report：#读取perf.data文件（由perf record创建）并显示性能分析报告，展示函数和符号的耗时信息。
	-i 指定文件名		--stdio 显示到终端
perf stat：#运行一个命令并收集性能计数器的统计信息，例如CPU周期、缓存命中率等。
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

## 示例

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
root@l:~# perf stat -d gzip file1  #使用perf工具来统计gzip命令对文件file1进行压缩的命令。
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

## 硬件采样

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

## 定时采样

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

### 产生CPU火焰图

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

## 周期采样

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

## 事件统计与事件跟踪



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



## 二进制可用安装

[linux-5.4.tar.gz包](http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/v5.x/linux-5.4.122.tar.gz) | 

```sh
tar -xzvf perf-*.tar.gz -C /usr/local/
cd /usr/local/linux-6.5/tools/perf
make WERROR=0 && make install WERROR=0
./perf -v #验证
```

`perf`：性能分析和跟踪工具，能够进行函数级与指令级的热点查找。

```sh
apt install linux-tools-common linux-tools-generic -y #安装包
```





# 文本处理工具

cat输出文件的内容		-n显示行数且空行显示行数		-b显示行数且空行不显示行数

-s压缩连续的空行为一行			-A显示所有控制符（用于调试脚本时）

cat >> /s <<EOF	以EOF表示结束，在交互式中输入内容

tee	读取屏幕信息，并再将信息输出到屏幕和文件中,-a或--append追加到文件里		echo "123" | tee -a file1

echo 输出，输入			echo $?输出上次命令反回值	echo `` 输出执行命令结果

echo "redhat" | passwd -- stdin alex 将密码redhat添加到alex账户

tac	/ rev		从下至上 / 从右至左 输出文件内容

basename

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
-z 使用gzip压缩/解压(.gz后缀压缩包)
-j 使用bzip2压缩/解压（压缩的更小,当时间更长）.bz2后缀压缩包（-k 可保留源文件）	
-C 指定解压地址		-c 创建/压缩	-x 释放/解压缩
-f 指定压缩文件名字		-v 显示提示信息		-tf 查询内容
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

```sh
[root@linuxprobe ~]# at 23:30
at > systemctl restart httpd
at >  #此处请同时按下 Ctrl + D组合键来结束编写计划任务
job 3 at Mon Apr 27 23:30:00 2017
[root@linuxprobe ~]# at -l3 Mon Apr 27 23:30:00 2017 a root
```

crontab 周而复始的计划任务

```
crontab -eu [username]
*  *  *  *  *  /bin/echo"hello"
分 时 日 月  周
例1: 30 21 * *            /bin/echo"hello"
例2: 45 4 1,10,22 * *     (每月的1、10、22日的四点四十五分)
例3: 0.30 18-23 * * *     (18-23时的零分和三十分)
例4: */5 * * * *          (每隔五分钟) 
```

-l查看当前生效用户的任务列表		-e编辑当前生效用户的任务内容				-r删除当前生效用户的任务内容

-i同上-r，但是在删除之前会询问确认	-u不针对当前生效用户，对指定的用户去操作	-R删掉用户所有周而复始任务

cat /var/log/cron日志文件			/var/spool/cron查看定时任务的文件夹			Crontab内每一行一个任务

watch -n1  ls /srv/ 				watch每一秒监控一次 ls命令srv下输出结果

| 参数             | 详解                                                         |
| :--------------- | :----------------------------------------------------------- |
| -n或–interval    | watch缺省每2秒运行一下程序，可以用-n或-interval来指定间隔的时间。 |
| -d或–differences | 用 -d 或 –differences 选项,watch 会高亮显示变化的区域。而-d=cumulative选项会把变动过的地方(不管最近的那次有没有变动)都高亮显示出来。 |
| -t 或 -no-title  | 会关闭watch命令在顶部的时间间隔,命令，当前时间的输出。       |

# 网络管理工具

## ubuntu

```sh
systemctl start systemd-networkd //开启 systemd-networkd服务
netplan apply //重启网络服务
# ens33为要重启的网卡名称
sudo ifdown ens33
sudo ifup ens33

vim 01-network-manager-all.yaml //修改网络配置文件
```

## centos

### ifconfig

启动 / 关闭指定网卡	

```sh
ifconfig ens33 up/down  #启动eth0网卡
```

为网卡配置多个ip地址

```sh
ifconfig ens33:1 192.168.0.2/24 #方式一
ifconfig ens33 192.168.xxx.xxx network 255.255.255.0 broadcast 192.168.xxx.xxx #方式二
```

删除配置的临时ip地址

```sh
ip a del 192.168.0.2/24 dev ens33
```

启用或关闭arp协议

```sh
ifconfig ens33 arp/-arp   #启用 / 关闭
```



### wget/curl

wget -O -r -p /etc/yum.repos.d/ali.repo http://mirrors.aliyun.com/repo/Centos-7.repo	下载阿里源

```
-O下载并指定文件名保存	-b后台下载	-r递归	-P指定下载目录		-S模拟输出服务器GET响应
```

curl -u admin:12345 http://192.168.2.5:3000	#-u指定admin用户名密码12345和指定端口进行登陆

```sh
# -A参数指定客户端的用户代理标头，即User-Agent。curl 的默认用户代理字符串是curl/[version]。
$ curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
# 上面命令将User-Agent改成 Chrome 浏览器。

$ curl -A '' https://google.com

# 上面命令会移除User-Agent标头。也可以通过-H参数直接指定标头，更改User-Agent。
$ curl -H 'User-Agent: php/1.0' https://google.com
```

```sh
# -b参数用来向服务器发送 Cookie。
$ curl -b 'foo=bar' https://google.com
上面命令会生成一个标头Cookie: foo=bar，向服务器发送一个名为foo、值为bar的 Cookie。
$ curl -b 'foo1=bar;foo2=bar2' https://google.com
上面命令发送两个 Cookie。
$ curl -b cookies.txt https://www.google.com
上面命令读取本地文件cookies.txt，里面是服务器设置的 Cookie（参见-c参数），将其发送到服务器。
```

```sh
# -c参数将服务器设置的 Cookie 写入一个文件。
$ curl -c cookies.txt https://www.google.com
上面命令将服务器的 HTTP 回应所设置 Cookie 写入文本文件cookies.txt。

-d
-d参数用于发送 POST 请求的数据体。


$ curl -d'login=emma＆password=123'-X POST https://google.com/login
# 或者
$ curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
使用-d参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法，因此可以省略-X POST。

-d参数可以读取本地文本文件的数据，向服务器发送。


$ curl -d '@data.txt' https://google.com/login
上面命令读取data.txt文件的内容，作为数据体向服务器发送。

--data-urlencode
--data-urlencode参数等同于-d，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码。


$ curl --data-urlencode 'comment=hello world' https://google.com/login
上面代码中，发送的数据hello world之间有一个空格，需要进行 URL 编码。

-e
-e参数用来设置 HTTP 的标头Referer，表示请求的来源。


curl -e 'https://google.com?q=example' https://www.example.com
上面命令将Referer标头设为https://google.com?q=example。

-H参数可以通过直接添加标头Referer，达到同样效果。


curl -H 'Referer: https://google.com?q=example' https://www.example.com
-F
-F参数用来向服务器上传二进制文件。


$ curl -F 'file=@photo.png' https://google.com/profile
上面命令会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。

-F参数可以指定 MIME 类型。


$ curl -F 'file=@photo.png;type=image/png' https://google.com/profile
上面命令指定 MIME 类型为image/png，否则 curl 会把 MIME 类型设为application/octet-stream。

-F参数也可以指定文件名。


$ curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
上面命令中，原始文件名为photo.png，但是服务器接收到的文件名为me.png。

-G
-G参数用来构造 URL 的查询字符串。


$ curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
上面命令会发出一个 GET 请求，实际请求的 URL 为https://google.com/search?q=kitties&count=20。如果省略--G，会发出一个 POST 请求。

如果数据需要 URL 编码，可以结合--data--urlencode参数。


$ curl -G --data-urlencode 'comment=hello world' https://www.example.com
-H
-H参数添加 HTTP 请求的标头。


$ curl -H 'Accept-Language: en-US' https://google.com
上面命令添加 HTTP 标头Accept-Language: en-US。


$ curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
上面命令添加两个 HTTP 标头。


$ curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
上面命令添加 HTTP 请求的标头是Content-Type: application/json，然后用-d参数发送 JSON 数据。

-i
-i参数打印出服务器回应的 HTTP 标头。


$ curl -i https://www.example.com
上面命令收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码。

-I
-I参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。


$ curl -I https://www.example.com
上面命令输出服务器对 HEAD 请求的回应。

--head参数等同于-I。


$ curl --head https://www.example.com
-k
-k参数指定跳过 SSL 检测。


$ curl -k https://www.example.com
上面命令不会检查服务器的 SSL 证书是否正确。

-L
-L参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。


$ curl -L -d 'tweet=hi' https://api.twitter.com/tweet
--limit-rate
--limit-rate用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。


$ curl --limit-rate 200k https://google.com
上面命令将带宽限制在每秒 200K 字节。

-o
-o参数将服务器的回应保存成文件，等同于wget命令。


$ curl -o example.html https://www.example.com
上面命令将www.example.com保存成example.html。

-O
-O参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。


$ curl -O https://www.example.com/foo/bar.html
上面命令将服务器回应保存成文件，文件名为bar.html。

-s
-s参数将不输出错误和进度信息。
$ curl -s https://www.example.com
上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。
如果想让 curl 不产生任何输出，可以使用下面的命令。
$ curl -s -o /dev/null https://google.com

-S
-S参数指定只输出错误信息，通常与-s一起使用。


$ curl -s -o /dev/null https://google.com
上面命令没有任何输出，除非发生错误。

-u
-u参数用来设置服务器认证的用户名和密码。


$ curl -u 'bob:12345' https://google.com/login
上面命令设置用户名为bob，密码为12345，然后将其转为 HTTP 标头Authorization: Basic Ym9iOjEyMzQ1。

curl 能够识别 URL 里面的用户名和密码。


$ curl https://bob:12345@google.com/login
上面命令能够识别 URL 里面的用户名和密码，将其转为上个例子里面的 HTTP 标头。


$ curl -u 'bob' https://google.com/login
上面命令只设置了用户名，执行后，curl 会提示用户输入密码。

-v
-v参数输出通信的整个过程，用于调试。


$ curl -v https://www.example.com
--trace参数也可以用于调试，还会输出原始的二进制数据。


$ curl --trace - https://www.example.com
-x
-x参数指定 HTTP 请求的代理。


$ curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
上面命令指定 HTTP 请求通过myproxy.com:8080的 socks5 代理发出。

如果没有指定代理协议，默认为 HTTP。


$ curl -x james:cats@myproxy.com:8080 https://www.example.com
上面命令中，请求的代理使用 HTTP 协议。

-X
-X参数指定 HTTP 请求的方法。


$ curl -X POST https://www.example.com
上面命令对https://www.example.com发出 POST 请求。
```

```
-v输出详细信息			-x指定使用的http代理				-A,--user-agent 用户访问代理软件
-H参数添加 HTTP 请求的标头										-s参数将不输出错误和进度信息。
-X参数指定 HTTP 请求的方法
-v参数输出通信的整个过程，用于调试						-k参数指定跳过 SSL 检测。
-x参数指定 HTTP 请求的代理									
-I参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。--head参数等同于-I。
-u参数用来设置服务器认证的用户名和密码。					-L参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。
-o将输出信息保存到指定文件	-s,silent静默模式，不输出任何信息	-I只显示响应报文头部信息
-O将服务器回应保存成文件，并将 URL 的最后部分当作文件名
-e,--referer告诉服务器我从哪个页面链接过来的
-b,--cookie使用指定cookie文件	-X GET|POST 设置请求方式
--limit-rate用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。
```

sz +文件名：将选定的文件发送（send）到本地机器			rz从本地选择文件上传到Linux服务器

iperf3网络质量测试工具（默认用tcp协议测试）

```
-s运行server模式					-D在后台以服务的方式运行
-c + host：连接指定客户端ip地址		-R服务端发送，客户端单方接受
```

vim /etc/NetworkManager/system-connections/ens160.nmconnection rocky系统网卡文件

lsof -i：22	通过端口号查看对应服务（ftp的端口）

ss / netstat -tupln 查看网络状态(查看各类服务)				

### route 显示和设置linux系统的路由表

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

ip addr 查看网卡状态	ifdown 关闭网卡	ifup打开网卡		cat /proc/net/bonding/bond0 查看bond是否生效

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

在可视化模式，使用hift + z + z 键：保存退出

：r  !  ls  /etc/sys读取（r）终端(!)下命令ls的输出

R! echo `pwd`		(  `` 将命令输出结果（/etc/yum.repo.d/路径）)

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