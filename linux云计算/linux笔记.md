<img src="E:\Project\Textbook\linux云计算\assets\wps1-1682690115363-174.jpg" alt="img" style="zoom: 67%;" /> 

/etc/services查看系统所有协议

<img src="E:\Project\Textbook\linux云计算\assets\wps2-1682690115363-175.jpg" alt="img" style="zoom:50%;" />修改语言

echo export EDITOR=vim >> /etc/profile.d/env.sh解决vimu无颜色问题

开机自启文件	/etc/rc.d/rc.local		chmod _+x rc.local添加执行权限	bc 计算器

hostnamectl  set-hostname computer设置永久主机名		whereis ssh查看服务配置文件夹

elinks文本浏览器			startx 切换图形界面			sudo 临时借用管理员权限

uname -a查看系统内核		cat /proc/version查看系统版本	lscpu查看系统cpu信息

pwd 显示当前所在目录		whoami 谁					which mysql查看内部命令

mv 移动，改名字	-b 若目标文件存在则创建备份文件			-f 强制覆盖且不提醒

uptime 用于查看系统的负载信息,运行时间					tree成树状显示当前目录所有文件

mkdir创建目录	-p多级创建		-v提示是否创建成功	 –m 创建同时设置权限值如755

rm  -r (递归删除)  -f（强制删除且不提醒）文件或目录	rmdir只能删除空目录

touch创建新文件，修改文件的创建时间（覆盖文件时间）		touch /linux{1..3}  建立文件多个文件

cp	-r /etc/passwd	递归拷贝文件或目录	-p拷贝权限，文件夹也是文件

cd ：进入该用户的主目录 ~（root用户为/root,其他用户为/home/用户名）

cd ~student 回到student用户的家目录

cd .. ：返回上一级目录（注意要空格）		cd - ：返回上次所在目录

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

top		查看进程活动状态以及一些系统状况	vmstat	查看系统状态、硬件和系统信息等

iostat		查看CPU负载，硬盘状况		sar	综合工具，查看系统状况

mpstat	查看多处理器状况	netstat	查看网络状况	iptraf		实时网络状况监测

tcpdump	抓取网络数据包，详细分析	tcptrace	数据包分析工具	netperf	网络带宽工具

dstat		综合工具，综合了vmstat,iostat, ifstat,netstat等多信息



lsof | grep /mnt/sdb1_mount //查看正在使用文件的进程

如果存在输出，则说明有进程正在使用该目录，需要先结束这些进程对该目录的占用。例如，输出可能类似如下：

```
COMMAND     PID        USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash      1234        root  cwd    DIR  253,0     4096   12 /mnt/sdb1_mount
```

则可以使用上述命令中显示的PID来结束该进程对目录的占用。例如，在本例中可以使用以下命令结束该bash进程：

```
kill -9 1234
```





# 文本处理工具

cat输出文件的内容		-n显示行数且空行显示行数		-b显示行数且空行不显示行数

-s压缩连续的空行为一行			-A显示所有控制符（用于调试脚本时）

cat >> /s <<EOF	以EOF表示结束，在交互式中输入内容

tee	读取屏幕信息，并再将信息输出到屏幕和文件中,-a或--append追加到文件里		echo "123" | tee -a file1

echo 输出，输入			echo $?输出上次命令反回值	echo `` 输出执行命令结果

echo "redhat" | passwd -- stdin alex 将密码redhat添加到alex账户

tac	/ rev		从下至上 / 从右至左 输出文件内容

less			分页查看

head / tail	显示文件前 / 后几行	-n 指定文本显示的行数		-c 指定文本显示的字符数

tail	参数		-f 实时输出文件内容的变化（tailf也常用于跟踪脚本、日志信息）		-F 跟踪文件名

## cut

列剪贴工具 / paste列合并工具	cut -d " " -f1,4-7 --complement  --c 1-4  a

-d delimiter：分隔符，默认的字段分隔符为“Tab”			-c：输出指定范围的字符	

-f  file：提取指定段（可以多个段）的内容					--complement反向选择字符段（除指定字段外的所有）

-b：输出指定范围的字节

paste：合并文件的列	 -s 将行换成列输出	-d，--delimiters=<间隔字符>分隔符

tr：标准输入的字符进行替换、压缩和删除	-d删除				-s去重复字符

uniq：去除相邻且重复的行				-c显示重复的次数		-u仅显示不曾重复的行

wc：word count统计行数，如下格式	-l 行数	-w 单词数	-c 字节	-m字符总数	-L文件最长行长度

diff：输出文件间区别		-u以合并的方式来显示文件内容的不同(常用于还原备份)	diff -u s a > s2.brk

sort：排序（默认小到大排序）	-n 数字由大到小排序(-r 降序搭配使用)		sort -n /opt/pp

-t 指定字段的分隔符			-u 去除重复行			-k 指定行数		sort -t ':' -nrk 3,6 /etc/passwd

## seq

squeue 是一个序列的缩写，主要用来输出序列化的东西

**用法**：seq [选项]... 尾数		或：seq [选项]... 首数 尾数		或：seq [选项]... 首数 增量 尾数（默认增量为1）

-s, --separator=字符串    指定分隔字符(默认使用：\n)			seq -s  '#'  1 2 10

-w, --equal-width   在列前添加0 使得宽度相同（自动补位）		seq -w 1 10

-f, --format=格式   指定格式输出，%g为默认格式

seq -f ‘%g’1 5	#表示指定“位宽”为三位，数字位数不足部分用空格补位）

seq -f "%03g" 2 9	#表示指定“位宽”为三位，数字位数不足部分用0补位，通过%后添加0替代空格补足空位，等同-w 

seq -f "as%03gaa" 4#% 前面可以指定字符串，同样 g 的后面也可以指定字符串

## grep/egrep/fgrep

强大的文本搜索工具，支持正则表达式搜索文本，显示具有指定字符的行

-n显示匹配行及行号					grep -n root /etc/passwd

-v反向选择匹配到的内容	grep -v aa file		grep -v '^$' /etc/zabbix_server.conf		找出所有非空行

-i不区分大小写						grep -i ROOT /etc/passwd

-c输出匹配到字符的行数					grep -c root /etc/passwd

-E选择多个字符						grep -E "root|zhunajie"  /etc/passwd

-h查询多文件时不显示文件名				grep -h  root  /etc/passwd /etc/shadow

-l只输出包含匹配字符的文件名			grep  -l root /etc/passwd

-s不显示不存在或无匹配文本的错误信息

^：以某某开头的内容		$：以某某结尾的内容		^$： 空行	.*：匹配所有的字符		^.*：任意多个字符开头

.：表示且只能代表任意一个字符 (当前目录，加载文件) 	\：转移字符，让有着特殊身份的字符，变回原来的字符

*：重复0个或多个前面的一个字符，不代表所有			[abc]：匹配字符集合内任意一个字符[a-z]

[^abc] ：^在中括号表示非，表示不包含a或者b或者c

## sed行（Stream EDitor）流编辑器，

sed：以行为单位自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序

### 常用选项：

-n：不显示默认输出内容					-i：直接修改(-i.brk先对原文件备份brk，后生成修改文件)

-e：可以指定多个地址定界、选项、命令	-r：启用扩展的正则表达式,当与其他选项使用时应作为首个选项

-{}：可组合多个命令,以分号分割			-f：使用sed脚本（如定义好的字符串或地址定界）

### 地址定界：

1、不给地址：对全文进行处理

2、单地址：	如：5：指定的行	$：最后一行		/正则表达式/：能被此模式所匹配到的每一行

3、地址范围：	1，3（1到3行）	或	1，+5（第一行再加5行）或	1，/正则表达式/

/字符串/	或	/正则表达式/		或	/正则表达式（开头）/，/或字符串（结尾）/

4、步进:		1~2（从第1行后每隔2行才进行匹配）奇数行		2~2（从第2行后每隔2行才进行匹配）偶数行

### 编辑命令（-i后接）

d：删除			p：打印，将匹配的行重复打印

i：前添加 \ a：后添加，a+字符串，原每一行的后一行都会添加指定字符串

c：代替，c+字符串，原每一行都会替换成指定的字符串

w：保存，w+文件名，保存匹配的行到指定文件

！：取反，放地址地址定界的后面		=：显示行号

s///：查找替代，支持其他分割符如s@@@，s###

替代标记：g：所有行替换，不加即替换匹配到的第一个字符串

p：显示替换成功的行	w + 文件名：将成功的行保存至指定文件

 [-n\-i选项]... {选项、地址定界；编辑命令}	 [输入文件]

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

取列lsmod | awk -F:  '{print $2}'  以：设置分隔符，默认为空格或Tab位打印第二段(列) 	{} 全选

### 内置变量：

NR：已读行的个数					NF：已读列的个数，$NF表示最后一列

FS：等于-F选项					$n：以分隔符为标志，选取段落，$0表示整行

OFS：将分隔符进行替换				FNR：保存当前处理行在原文本内的序号，行号

FILENAME：当前处理的文件名		ENVIRON：表示支持系统环境变量，格式ENVIRON["变量名"]

### 变量运算

ip a | awk 'NR<=9{print $2"@@"$1}' #匹配前九行打印第2段，第1段，并进行反序，中间加入@@字符串

awk -F: -vOFS=@ '{print $1,$2}' /etc/passwd #将第一、二段的分隔符：替换为@

echo "7.7 3.8"|awk '{print ($1-$2)}' #进行减运算并得出结果

awk '{sum+=$7} END {print "Avg=", sum/NR}' #将每一行的第七列累加，最后打印计算平均值	#BEGIN模块、END模块

awk '/^(root|ssh)/' /etc/passwd #打印所有以root或ssh开头的行

awk '$1 > 5 && $2 < 10' test #如果第一个域大于5，并且第二个域小于10，则打印这些行

awk '$1 = 100 {print $1 > "file" }' test #上式表示如果第一个域的值等于100，则把它输出到file中

echo a b c d | awk '{$2=$2" e f g";print}' #将第二段的字符替换为b e f g,再打印出来

awk‘NR%2\==0’test 输出偶数行文本			awk -F：'$1=="sync"’/etc/passwd 用户名为sync的行

awk -F:‘$1==ENVIRON["USER"]’ /etc/passwd

awk '/-rwx/{print}'  test >test1 打印包含“-rwx”字符的行

awk -F: '{if($3>999 && $7 == "/bin/bash" || $7 == "/bin/sh")print}' /etc/passwd 使用if判断列是否符合

## 输入/出重定向

1>正确输出						2> 错误输出		cat /opt/cc /home/  >/opt/right		2>/opt/wrong

1>>正确输出只能保存正确的输出		1>>正确且覆盖式重定向	

2>>错误输出只能保存错误的输出		2>>错误重定向	&>只要输出就能保存				&>混合重定向

输出内容到这里</etc/passwd

dd if=/dev/zero of=/root/file1 bs=100M count=1

用/dev/zero向file.1里面输入内容	count指定文件个数为1 ，bs=20M大小

## find查找文件

-type （d文件夹 f文件 l链接 b块设备 c字符）		find  /  -type  f  -name  "*abc*"

-name匹配文件名								find  /  -name  "[a-z][a-z][a-z].conf"

find  / -name  "???.conf"任意三个字符（数字，字母，点逗号）

-size文件大小（+表示大于）						find  /  -size +100M 

-user所有者（-group所有组）					find  /  -user zhubajie ! (且)-group zhubajie

-nouser无所有者（-nogroup无所有组）			find  /  -nouser

-exec …… {}\; 接下一个命令  --exec这个命令只能接在find命令后，也就是find专用的

## xargs(命令传递参数命令) 

ls | grep -v ^file$ | xargs  -i  cp  {}  /tmp/		-i 允许用大括号带替前面的内容		{} 代表示管道符号前的内容

ls | grep ^file$ | xargs rm -fr 	查找以file开头且以file结尾的

ls file*  | xargs  -i  mv{}  a{}		在以file开头的文件前加上a这个字符

## tar压缩工具和zip

tar -cjvf backup.tar.bz2 * 文件压缩成backup.tar.bz2名字的文件，最好时进入需要打包文件夹

tar -xzvf test.tar.gz	解压gz文件（在要压缩的目录里）

tar -xf node-v12.18.1-linux-x64.tar.xz 解压xz格式压缩包

-z 使用gzip压缩	.gz后缀压缩包

-j 使用bzip2压缩（压缩的更小,当时间更长）.bz2后缀压缩包（-k 可保留源文件）	-C指定解压地址

-c 创建/压缩		-x 释放/解压缩		-f 指定压缩文件名字		-v 显示提示信息		-tf 查询内容

zip /srv/file.zip  * 将当前文件压缩成file.zip文件		-sf查看压缩包里面的内容	-r表示递归压缩子目录下所有文件

zip -d myfile.zip smart.txt #删除压缩文件中smart.txt文件

zip -m myfile.zip ./rpm_info.txt #向压缩文件中myfile.zip中添加rpm_info.txt文件

unzip -od /home/sunny myfile.zip #把myfile.zip文件解压到 /home/sunny/

-o:不提示的情况下覆盖文件	-d:将文件解压缩到指定目录下

## top ps 进程信息控制kill

进程是个动态的概念，即正在进行的程序，程序是个实体

内核的功能：进程管理，文件系统，网络功能，内存管理，驱动程序，安全功能

Process：运行中的程序的一个副本，是被载入内存的一个指令集

进程ID（ProcessID，PID）号码被用来标记各个进程

UID、GID、和selinux语境决定对文件系统的存取和访问权限

通常从执行进程的用户来继承

存在生命周期<img src="E:\Project\Textbook\linux云计算\assets\wps3-1682690115363-176.jpg" alt="img" style="zoom: 67%;" />

task struct：linux内核储存进程信息的式数据结构格式

task lsit：多个任务的task struct组成的链表5

进程创建：init：第一个进程

### 父子关系

进程：都由其父进程创建，Cow	fork（）【用来生成子进程】，clone（）【克隆】

ps -ef/aux	显示当前进程（ e显示所有进程 f全格式  /  a所有u用户x不同7个终端）

<img src="E:\Project\Textbook\linux云计算\assets\wps4-1682690115363-177.jpg" alt="img" style="zoom:50%;" /> 

kill -9暴力杀死进程		-9（暴力杀死）  -15（逐步杀死）+进程号	18继续 19停止 20暂停

pkill -9 dns	杀掉相关的进程（如杀掉dns服务)				pkill -9 zhubajie	杀掉用户zhubajie的所有进程

pkill -vu root	(-v反选)杀掉除root用户打开的所有进程			pkill -9 %2		作业号为2的进程

Command &	放到后台运行		jobs查看当前终端后台的进程

fg 2把后台2进程放到前台来运行			fg	%1将后台job编号为1的进程调回到前台

bg把后台暂停的进程放到后台运行			bg 	%2把后台编号为2的进程恢复运行状态

firefox  www.baidu.com  & 打开浏览器放到后台运行

top是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器

top -p  2822(PID号)	#查询固定PID号

<img src="E:\Project\Textbook\linux云计算\assets\wps5-1682690115363-178.jpg" alt="img" style="zoom:50%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps6-1682690115363-179.jpg" alt="img" style="zoom:50%;" /> 

![img](E:\Project\Textbook\linux云计算\assets\wps7-1682690115363-180.jpg) 

## at一次性任务、周而复始计划任务

echo `Ifconfig ens33` | at now+1 min

-L：查看所有任务 	-c1：查看任务1详细信息		-d1：删除任务1

<img src="E:\Project\Textbook\linux云计算\assets\wps8-1682690115363-181.jpg" alt="img" style="zoom: 67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps9-1682690115363-182.jpg" alt="img" style="zoom: 67%;" /> 

-l查看当前生效用户的任务列表		-e编辑当前生效用户的任务内容				-r删除当前生效用户的任务内容

-i同上-r，但是在删除之前会询问确认	-u不针对当前生效用户，对指定的用户去操作	-R删掉用户所有周而复始任务

cat /var/log/cron日志文件			/var/spool/cron查看定时任务的文件夹			Crontab内每一行一个任务

watch -n1  ls /srv/ 				watch每一秒监控一次 ls命令srv下输出结果

# 网络管理工具

wget -O -r -p /etc/yum.repos.d/ali.repo http://mirrors.aliyun.com/repo/Centos-7.repo	下载阿里源

 -O下载并指定文件名保存	-b后台下载	-r递归	-P指定下载目录		-S模拟输出服务器GET响应

curl -u admin:12345 http://192.168.2.5:3000	#-u指定admin用户名密码12345和指定端口进行登陆

curl -vs -o /dev/null -A "Google Chrome" -e -e www.baidu.com -x 103.219.36.13:8080 http://cacti.rednetcloud.com

-v输出详细信息				-x指定使用的http代理				-A,--user-agent 用户访问代理软件

-o将输出信息保存到指定文件		-s,silent静默模式，不输出任何信息	-I只显示响应报文头部信息

-O将服务器回应保存成文件，并将 URL 的最后部分当作文件名			-e,--referer告诉服务器我从哪个页面链接过来的

-b,--cookie使用指定cookie文件	-X GET|POST 设置请求方式

sz +文件名：将选定的文件发送（send）到本地机器			rz从本地选择文件上传到Linux服务器

iperf3网络质量测试工具（默认用tcp协议测试）		-s运行server模式		-D在后台以服务的方式运行

-c + host：连接指定客户端ip地址					-R服务端发送，客户端单方接受

lsof -i：22	通过端口号查看对应服务（ftp的端口）

ss / netstat -tupln 查看网络状态(查看各类服务)				route 显示和设置linux系统的路由表

route add –host 192.168.168.110 dev eth0				#添加到主机的路由

route add –host 192.168.168.119 gw 192.168.168.1			#添加主机路由网关

route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.1.1		#net添加网段

traceroute -p 80 www.baidu.com #追踪网络数据包的路由途径，-p指定端口

tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

1、tcp、ip、icmp、arp、rarp、tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型

2、-i eth1 : 只抓经过接口eth1的包

3、-t : 不显示时间戳

4、-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包

5、-c 100 : 只抓取100个数据包

6、dst port ! 22 : 不抓取目标端口是22的数据包

7、src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24

8、-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析

## Network、链路聚合

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

nmcli	n[etworking]			on\off	#连接设置

nmcli	d[evice]				status查看设备状态 \ show ens33查看网卡设备信息 \set

nmcli 	c[onnection]  		reload重载 \ up激活 + 网卡配置文件 \ down  \ load \ modify 修改 ens160

ipv4.addresses 192.168.127.10/24修改IP地址	+ipv4.addresses 172.16.10.10/16添加第二个IP地址\DNS

ipv4.gateway 192.168.127.2 修改网关			ipv4.dns 8.8.8.8

ipv4.method manual修改为手动配置 			ipv4.method auto修改为DHCP获取

nmcli	g[eneral]				status\hostname\logging(支持服务信息)		#概叙信息

 

## 创建网络组接口

nmcli con add type team con-name xxx ifname xxx config JSON

type  team 设备类型 ，con-nam 连接名，  ifname 接口名

JSON 指定runner方式，格式： '{"runner": {"name": "METHOD"}}'

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

ifcfg-eno1					ifcfg-bond

TYPE=Ethernet				DEVICE=bond				#和文件名没关系

NAME=eno1					NAME=bond				#和文件名没关系

DEVICE=eno1					TYPE=Bond

BOOTPROTO=none			IPADDR=192.168.5.249

ONBOOT=yes				NETMASK=255.255.255.0

MASTER=bond0				GATEWAY=192.168.5.1

SLAVE=yes					PEERDNS=yes

ONBOOT=yes

BOOTPROTO=static

BONDING_MASTER=yes

BONDING_OPTS="mode=6 miimon=100 updelay=600000 primary=eno1"	#模式和网卡

bond模式起不来步骤：1、service NetworkManager stop 临时和永久关闭NetworkManager		2、reboot重启

IP addr 查看网卡状态	ifdown 关闭网卡	ifup打开网卡		cat /proc/net/bonding/bond0 查看bond是否生效

lspci|grep net 查看所有网卡型号参数		

iftop -i eth2 #支持画面操作（区分大小写）-i指定网卡 -n将主机名显示为IP  -N不将端口号转换为服务 -P显示端口和主机

ethtool #查询或控制网络驱动程序和硬件设置

ethtool -i eth2 #-i显示网卡驱动名称、版本信息		-d网口注册性信息		-S ens33：网口收发包统计

# 软件包管理

### Rpm

/run/media/root/CentOS\ 7\ x86_64/Packages/	文件下默认含有rpmb=包

Rpm -ivh +二进制包	-i 安装（install）-e删除 -h 进度条显示 -v显示	-q查询	-a所有	--force强制安装

-U/--upgrade<套件档> 升级指定的套件档

Rpm -qa	查询所有安装过的包	 -ql	查看所有安装过的路径		-qf  bin/chmod	查看文件由哪个包提供

-qi 查看安装包详细信息			 -e 钥匙 删除不匹配的钥匙		 -K 包名 查看该包需要的钥匙，显示 OK

显示NOT OK (MISSING KEYS: (MD5) PGP#fd431d51) 中文版显示：不正确

gpg -v 钥匙包  		//从系统上所有的钥匙包里面，找到需要的钥匙

rpm包的命名方式：name-version-release.arch.rpm

​      发行号：用于标识rpm包的本身发行号，可还包含所适用的操纵做系统

​      例如：el6：RHEL6

​      arch：主机平台

​      例如：i386、x86_64、amd64、ppc、noarch不区分平台

### Yum仓库

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



### ubuntu使用阿里云源

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



### make编译安装

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

 -d 0， --lastday 最近日期   指定用户最后修改日期为今天

 -E, --expiredate 过期日期   将帐户过期时间设为“过期日期” 

 -I, --inactive INACITVE	 过期 INACTIVE 天数后，设定密码为失效状态

 -l, --list          		 显示帐户年龄信息

 -m, --mindays 最小天数     将两次改变密码之间相距的最小天数设为“最小天数”

 -M, --maxdays 最大天数     将两次改变密码之间相距的最大天数设为“最大天数”

 -R, --root CHROOT_DIR     chroot 到的目录

 -W, --warndays 警告天数    将过期警告天数设为“警告天数”

1.设置用户密码最小天数（-m），最大天数（-M），提醒天数（-W），不活跃天数（-I）

[root@localhost ~]# chage -m0 -M8 -W8 -I5 zhubajie

id -gn zhubajie	查看zhubajie组名			-gn (group name) 组名		-un (user name) 用户名

useradd aaa（建立用户名aaa）

usermod 修改用户属性信息 	-d 改变用户主目录		-g 修改用户主组	-G 指定用户所属附加组	-l name:更改账户的名称

-u UID：改变用户的UID为新的值		-s 指定以/sbin/nologin非shell登录	-e设置用户账户使用期限

userdel 删除用户 -fr（强制删除用户及其主目录）	passwd aaa（给用户aaa设置密码）

groupadd 添加组

groupmod 修改组		-g 修改用户所属的群组	-l 修改用户帐号名称	-u 修改用户ID

-d 修改用户登入时的目录	-s 修改用户登入后所使用的shell（/sbin/logoin非交互式登录)

gpasswd  管理组成员	-a向组中添加用户		-d从组中删除用户

groupdel 删除组

### 文件权限

user group other					su 切换用户 用法：su -xxx(用户)

u(所有者)，g(所属组)，o(其他用户)，a(所有用户)		+（代表添加） -（减少） =（权限覆盖）

r（读取权限数字代表为4)，w（写入权限数字代表为2)，x（执行权限数字代表为1）

chmod 更改文件或目录的权限			chmod	a+rwx  a			chmod 777 a	

chown 更换文件所有者/更换文件所属组	chown 	root:bin 	/file

chgrp 变更文件或目录的所属群组 			chgrp	root	-R 	/file

u+s所有人对此文件都有执行权限			g+s强制将此群组里的目录下文件编入到此群组中

o+t这个目录只有root和此目录的拥有者可以删除，其他用户全都不可以

chattr +i/-i	 123	 锁定/取消文件或目录（锁定后无法修改删除移动）	lsattr 查看文件是否被锁定

### ACL访问控制列表

setfacl：具体权限设置ACL		u：用户		g：组	d：默认

-m更改文件的访问控制列表		setfacl -m u:zhangy:rw- test修改文件的acl权限，添加一个用户权限

-R递归操作子目录				setfacl -Rm d:zhangsan:rwx /opt/tt 为目录/opt/tt设置ACL规则

-x从文件中访问控制列表移除条目	setfacl -x u:zhangsan /opt/ta 删除用户zhangsan对/opt/ta文件的ACL规则

setfacl -x u:zhangsan /opt/ta删除用户zhangsan对/opt/ta文件的ACL规则

-b还原成默认acl列表			setfacl -b  test 还原acl列表

-d应用到默认访问控制列表的操作	setfacl -d --set g:zhangsan:rwx /ok 设置ok目录默认ACL

-h显示本帮助信息

getfacl  /file查文件权限的内容			-n显示数字的用户/组标识		-R递归显示子目录

umask 权限掩码，设定文件创建时的缺省模式

umask 0644一次性缺省权限		/etc/profile	文件里修改反掩码

用户设置自己永久性的umask值，在自己$HOME目录下的.profile或.bash_profile文件中

### selinux权限设置

配置文件：/etc/sysconfig/seliunx						chcon和setsebool、getsebool都是selinux的一部分

chcon：控制文件的访问属性

-R, --recursive：递归处理所有的文件及子目录			-t, --type=类型：设置指定类型的目标安全环境

-u, --user=用户：设置指定用户的目标安全环境			-r, --role=角色：设置指定角色的目标安全环境

-v, --verbose：为处理的所有文件显示诊断信息			-l, --range=范围：设置指定范围的目标安全环境

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

<img src="E:\Project\Textbook\linux云计算\assets\wps14.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps15-1682690115364-187.jpg" alt="img" style="zoom: 67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps16-1682690115364-188.jpg" alt="img" style="zoom:67%;" /> 

磁盘分区：MBR(用fdisk分区) 和 GPT（用gdisk命令，需安装软件）

mkfs.创建文件系统				-t xfs指定文件系统					-b 1024指定块的大小

mount 文件系统挂载命令		umount 卸载命令

-r: readonly，只读挂载			-w: read and write, 读写挂载			-a：自动挂载所有支持自动挂载的设备

-L 'LABEL': 以卷标指定挂载设备	-U 'UUID': 以UUID指定要挂载的设备	-B, --bind: 绑定目录到另一个目录上

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

<img src="E:\Project\Textbook\linux云计算\assets\wps18-1682690115364-190.jpg" alt="img" style="zoom: 80%;" /> 

Defaults,ro 只读（不能写）		df -Th显示磁盘使用情况		-T显示文件系统的形式	-h以人类可读方式显示

fdisk -l 查看显示硬盘信息		partprobe磁盘探测			lsblk 查看挂载列表		blkid查看块设备id号,文件系统类型

## LVM（Logical Volume Manager）逻辑卷制作

LVM是Linux环境中对磁盘分区进行管理的一种机制，是建立在硬盘和分区之上、文件系统之下的一个逻辑层，可提高磁盘分区管理的灵活性

PE(Physical Extend)　　基本单元：具有唯一编号的PE是能被LVM寻址的最小单元

PE的大小可以指定，默认为4MB。PE的大小一旦确定将不能改变，同一个卷组中的所有的物理卷的PE的大小是一直的

PV(Physical Volume)	物理卷		VG(Volume Group)	卷组			LV(Logical Volume)	逻辑卷

<img src="E:\Project\Textbook\linux云计算\assets\wps19-1682690115364-191.jpg" alt="img" style="zoom: 67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps20-1682690115364-192.jpg" alt="img" style="zoom: 67%;" /> 

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

<img src="E:\Project\Textbook\linux云计算\assets\wps26-1682690115365-198.jpg" alt="img" style="zoom: 50%;" /> 

2048前面空间是放主引导分区

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
	echo "Usage: /tmp/userdb"
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
while 条件  //While : 	//无限循环
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

<img src="E:\Project\Textbook\linux云计算\assets\wps34-1682690115365-206.jpg" alt="img" style="zoom: 67%;" /> 

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

-l  生成密码的长度，默认是 9 位，不同版本默认长度可能是不一样	-d  生成密码中包含数字的位数，默认是 2 位

-c  生成密码中包含小写字母的位数，默认是 2 位					-C  生成密码中包含大写字母的位数，默认是 2 位

-s  生成密码中包含特殊字符的位数，默认是 1 位

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

强制退出(不保存) : 普通模式下::q!

保存:	4.1 另存为:	:w 文件名			4.2 保存&退出:	:wq

复制		5.1 复制当行到系统剪贴板:	"+yy 这个是真+			5.2 复制所选至系统剪贴板:	"+y

5.3 选择+复制 普通模式下:	选择: v		复制: y				5.4 复制当前行至vim剪贴板:	yy

黏贴:		6.1 系统剪贴板:	shift+ctrl+v					6.2 vim剪切板粘贴至下一行:	p

普通模式下删除/剪切:	dd

8.1 删除所在光标下的#行:	#dd (#自然数)				8.2 删除所在光标上的#行:	#dk

8.3 向下删除至底: dG								8.4 向上删除至顶: dgg

撤销以及反撤销	9.1 撤销: 普通模式下 u				9.2 反撤销: 普通模式下 Ctrl+r

批量选择:10.1 向下: G		10.2 向上: gg				10.3 选中某个方格: Ctrl+v

屏幕滚动:11.1 向下↓ 一页:Ctrl+f ; 向上↑ 一页:Ctrl+b		11.2 向下↓ 半页:Ctrl+d ; 向下↓ 半页:Ctrl+u

移动光标(上下左右)：

1.3 本行头:0或^ 本行尾:$	1.4 迅速移动: ctrl+箭头(跳过空格)或Shift+箭头(跳过符号)	1.5 移至顶部：gg 移至底部：G

在vim中有3中方法可以跳转到指定行（首先按esc进入命令行模式）：

1、ngg/nG （跳转到文件第n行，无需回车）

2、:n （跳转到文件第n行，需要回车）

3、vim +n filename （在打开文件后，跳转到文件的第n行）

搜索:

13.3 特殊搜索

^	放在字符串前面，匹配行首的字；			$	放在字符串后面，匹配行尾的字；

<	匹配一个字的字头；					>	匹配一个字的字尾；

.	匹配任何单个正文字符；					[str]	匹配 str 中的任何单个字符；

[^str]	匹配任何不在 str 中的单个字符；		[a-b]	匹配 a 到 b 之间的任一字符；

*	匹配前一个字符的 0 次或多次出现；		\	转义后面的字符

13.4 从#1行到#2行,搜索替换x为y

在每一行前加@符号（操作要在在光标前进入编辑模式）

Ctrl + v 		Shif + a		@		Esc			esc

:#1,#2s/x/y/g (#1 #2 为自然数)

:#1,#2s/x/y/gc (替换前确认confirm)

 

自动对齐 ：shift +v(选中范围)   =（进行对其）

 

代码自动补全: ctrl + p

非编辑模式下查看man手册：shift +k

 

设置set:

14.1 显示行号:	:set nu				14.2 取消行号:	:set nonu				14.3 设置缩进:	:set tabstop=#		

14.4 自动缩进:	:set autoindent		14.5 显示名称:	:set laststatus=2		14.6 显示行符:	:set list	

14.7 取消行符:	:set nolist

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
	call setline(3,"#Author:		liulengbo")
	call setline(4,"#weixin:		13086119057")
	call setline(5,"#Date:		".strftime("%Y-%m-%d"))
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