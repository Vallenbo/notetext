学习一个服务的过程:

1、此服务器的概述:名宁，功能，特点，端口号1
2、安装
3、配置文件的位置
4、服务启动关闭脚本，查看端口
5、此服务的使用方法
6、修改配置文件，实战举例
7、 排错(从下到上，从内到外)

#  远程登录工具

##  ssh远程登陆

<img src="E:\Project\Textbook\linux云计算\assets\wps2.jpg" alt="img" style="zoom:67%;" /> 

redhat7默认安装，服务名sshd，端口号22		主配置文件：/etc/ssh/sshd_config（注：修改端口号要关闭selinux）

| 参数                                | 作用                                | 参数                              | 作用                                |
| ----------------------------------- | ----------------------------------- | --------------------------------- | ----------------------------------- |
| Port 22                             | 默认的sshd服务端口                  | ListenAddress 0.0.0.0             | 设定sshd服务器监听的IP地址          |
| Protocol 2                          | SSH协议的版本号                     | HostKey /etc/ssh/ssh_host_key     | SSH协议版本为1时，DES私钥存放的位置 |
| HostKey /etc/s3h/ssh_host_rsa_ _key | SSH协议版本为2时，RSA私钥存放的位置 | HostKey /etc/ssh/ssh_host_dsa_key | SSH协议版本为2时，DSA私钥存放的位置 |
| ==PermitRootLogin yes==             | 设定是否允许root管理员直接登录      | StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接  |
| MaxAuthTries 6                      | 最大密码尝试次数                    | MaxSessions 10                    | 最大终端数                          |
| PasswordAuthentication yes          | 是否允许密码验证                    | PermitEmptyPasswords no           | 是否允许空密码登录(很不安全)        |

ssh命令格式：ssh  root@10.0.0.11 -p22 远程登录命令	-p指定端口号

ssh 192.168.0.24 -l（登录用户）root  -X(带图形化)

ssh-keygen	在本机中生成钥匙对

 <img src="E:\Project\Textbook\linux云计算\assets\wps4.jpg" alt="img" style="zoom: 67%;" />

Ssh-copy-id  192.168.0.26#SSH服务分发公钥命令，将密钥发送给远程机下次登录不需要密码

-i 	/root/.ssh/id_rsa发送指定密钥文件	-t echo 123 执行命令

~/.ssh/known_hosts默认存放远程主机密码文件

vim /etc/hosts.deny远程登录黑名单【hosts.allow 文件中的规则优先级高】

sshd:	172.16.0.1/24拒绝访问的主机

### scp远程复制命令

scp  -rp /srv/aa.sh 192.168.0.24:/opt/				远程拉取命令-r递归拷贝-p拷贝权限

scp -P22  alex@192.168.0.24:/opt/  /srv/aa.sh		远程推送命令-P(大写)指定端口号22

sftp远程文件传输服务，主要用于windos与linux之间登录进行加密传输，比FTP有更高的安全性

sftp -P22 root@192.168.0.11		远程连接命令

mget下载多个文件			mirror 下载文件夹		mput命令传输多个文件

!pwd 调用终端命令			lls  显示本地文件列表	lpwd 查看本地当前位置

last	或w	查看所有系统的登录规则

##  dropbear远程登陆工具

dropbear作为一款基于ssh协议的轻量级sshd服务器，相比OpenSSH，其更简洁，更小巧，运行起来内存占用比也更小。OpenSSH会开启两个sshd进程服务，而dropbear只开启一个进程，相较于OpenSSH，其对于硬件要求也更低，也更节约系统资源。dropbear实现完整的SSH客户端和服务器版本2协议，不支持SSH版本1协议的向后兼容性，以节省空间和资源，并避免在SSH版本1的固有的安全漏洞。

dropbear主要有以下程序：服务程序：dropbear（类似于Openssh的sshd）

客户程序：dbclinet（累世于Openssh的ssh）		密钥生成程序：dropbearkey

yum在线安装的配置文件：/usr/lib/systemd/system/dropbear.service

```sh
yum groupinstall Development tools -y
yum install zlib-devel -y

./configure  --prefix=/usr/local	#指定安装目录
make PROGRAMS="dropbear dbclient dropbearkey dropbearconvert scp"
make PROGRAMS="dropbear dbclient dropbearkey dropbearconvert scp" install
mkdir /etc/dropbear/	#生成key文件存储目录
dropbearkey -t rsa -f /etc/dropbear/dropbear_rsa_host_key	#-t生成的密钥，-f指定存放地
dropbear -E -p 2222 #-E-将日志规则到stderr而不是syslog，-F前台输出信息
```

##  Rsync 文件拷贝命令

拷贝会查询是否有相同文件进行询问（命令格式和cp一样）

rsync /etc/fstab /tmp # 在本地同步

rsync -r /etc  172.16.10.5:/tmp将本地/etc目录拷贝到远程主机的/tmp下，以保证远程同步

rsync -r -b /123/ 192.168.0.109:/123 已存在的文件就被做一个备份

rsync -avz /mnt/sata/* /var/lib/mysql/data #拷贝mysql数据，与cp命令不同

-v：显示rsync过程中详细信息。可以使用"-vvvv"获取更详细信息

-a --archive  ：归档模式，表示递归传输并保持文件属性。

-z     ：传输时进行压缩提高效率。

-r --recursive：递归到目录中去

-P：显示文件传输的进度信息。(实际上"-P"="--partial --progress"，其中的"--progress"才是显示进度信息的)。

-n --dry-run  ：仅测试传输，而不实际传输。常和"-vvvv"配合使用来查看rsync是如何工作的。

-e      ：指定所要使用的远程shell程序，默认为ssh。

-t --times：保持mtime属性。强烈建议任何时候都加上"-t"，否则目标文件mtime会设置为系统时间，导致下次更新

​     ：检查出mtime不同从而导致增量传输无效。

-D     ：是"--device --specials"选项的组合，即也拷贝设备文件和特殊文件。

-l --links：如果文件是软链接文件，则拷贝软链接本身而非软链接所指向的对象。

--exclude  ：指定排除规则来排除不需要传输的文件。

-b --backup ：对目标上已存在的文件做一个备份，备份的文件名后默认使用"~"做后缀。

--backup-dir：指定备份文件的保存路径。不指定时默认和待备份文件保存在同一目录下。

--port    ：连接daemon时使用的端口号，默认为873端口。

-W --whole-file：rsync将不再使用增量传输，而是全量传输。在网络带宽高于磁盘带宽时，该选项比增量传输更高效。

--existing  ：要求只更新目标端已存在的文件，目标端还不存在的文件不传输。注意，使用相对路径时如果上层目录不存在也不会传输。

--ignore-existing：要求只更新目标端不存在的文件。和"--existing"结合使用有特殊功能，见下文示例。



#  时间同步服务器

date -s "20140225 20:16:00"	#设置时间

##  NTP网络时钟服务（network time protocol ）

三种方式：广播模式（时间一致）	主被动模式		port：123

C/S模式（服务与客户）权威时间		ntpdate -u ntp.api.bz或者配置文件里的权威时间同步地址

包：ntp（服务端包）	ntpdate(客户端包)		服务名ntpd

Systemctl enable ntpd 设置开机自启  Systemctl disable ntpd 设置开机关闭

**主配置文件**vim /etc/ntp.conf

server 127.127.1.0 prefer #设置本机为NTP服务器

restrict 156.0.26.7    #允许客户端156.0.26.7向本机请求时间同步

restrict 156.0.26.0 mask 255.255.255.0 #允许客户端156.0.26.0网段的所有主机向本机请求时间同步

<img src="E:\Project\Textbook\linux云计算\assets\wps5.jpg" alt="img" style="zoom:67%;" /> 

客户端去匹服务器的时间

客户端需要修改/etc/ntp.conf，添加以下内容

server 156.0.26.6  #指名上层NTP服务器　　restrict 156.0.26.6    #放行156.0.26.6

Chronyc sources -v	查看客户端时钟服务状态

System-config-date	图形化时间设置	ntpq -p查看是否能够被匹配

<img src="E:\Project\Textbook\linux云计算\assets\wps6.jpg" alt="img" style="zoom:67%;" /> 

Restrict	策略限制

Notrap	不远程登录服务器					Nopeer	不与同一级别服务器做时间比对

Notrust	不信任（客户端要认证）				Noquery	不能

##  Chrony时间同步服务

包：chrony						端口port:123/UDP

修改主配置文件/etc/chrony.conf		allow 192.168.100.0/24允许100网段主机同步

修改客户机配置文件/etc/chrony.conf	server 192.168.100.101 iburst

chronyc sources：查看当前时间同步服务器地址		chronyc makestep：立即同步时间

# Yum服务器

YUM服务端搭建

```sh
gpgcheck=1  进行公钥检测，要提供对应的钥匙才能安装对应的包
rpm -K 包名    	//查看该包需要的钥匙，显示 NOT OK (MISSING KEYS: (MD5) PGP#fd431d51) 
中文版显示：不正确

gpg -v 钥匙包  		//从系统上所有的钥匙包里面，找到需要的钥匙
rpm --import  钥匙包	//将匹配的钥匙包导入系统
rpm -qa |grep pubkey	//查看系统上已经导入的所有钥匙
rpm -e 钥匙			//删除不匹配的钥匙 
rpm -K 包名			//查看该包需要的钥匙，显示 OK
rpm -ivh 或者 yum install 	//最后安装
```

## 使用FTP服务端搭建yum源

​	安装ftp服务，开机自启，启动服务

​	指定镜像永久挂载路径：/var/ftp/pub/sss

​	解决权限拒绝的sebool值问题

​	客户端仓库文件：baseurl=ftp://192.168.0.22/pub/sss

## 使用httpd服务端搭建yum源

​	安装httpd服务，开机自启，启动服务

​	指定镜像永久挂载路径：/var/www/html/sss

​	客户端仓库文件：baseurl=http://192.168.0.22/sss

​	原本镜像已经挂载到了ftp目录，想要同时使用http共享镜像：

​	做软连接：ln  -s  /var/ftp/pub/dvd  /var/www/html/iso

# VPN虚拟专用网络

PPTP，Point to Point Tunneling Protocol，点对点隧道协议，这是一种支持多协议虚拟专用网络（VPN）技术。远程用户能够通过装有点对点协议的系统安全访问公司网络

设置两块网卡（外网内网）		安装包和服务名：pptpd

主配置文件：/etc/pptpd.conf

```
localip 192.168.127.11
remoteip 192. 168.0.20-238，192.168.0.245
```

localip表示VPN服务器本地IP，可以设置为172.16.1.0/24网段，也可以设置为其他网段；

remoteip表示设置一个地址段供客户机连接使用

用户文件/etc/ppp/chap-secrets 

```
#用户名				服务器名				密码					IP *号表示随机分配
#client				server					secret					IP addresses
 admin				pptpd					123456					*
```

/etc/ppp/options.pptpd文件一般用户设置DNS的文件（ms-dns配置项）

```
#配置DNS
#ms-dns 10.0.0.1
#ms-dns 10.0.0.2
```



# DHCP

C/S模式(客户端与服务端不在同一主机上）

Dynamic Host Configuration Protocol动态主机配置协议，是TCP／IP协议簇中的一种，是一个局域网的网络协议，使用 UDP协议工作。

![img](E:\Project\Textbook\linux云计算\assets\wps10.jpg) 

安装包:dhcp		服务名：dhcpd	端口port：67	主配置文件及模板文件地址：/etc/dhcp/dhcpd.conf

<img src="E:\Project\Textbook\linux云计算\assets\wps11.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps12.jpg" alt="img" style="zoom:67%;" /> 

分配固定IP地址<img src="E:\Project\Textbook\linux云计算\assets\wps13.jpg" alt="img" style="zoom: 67%;" />

#  网络文件共享服务

DAS: 直连式存储(Direct-AttachedStorage)

　　存储设备是通过电缆（通常是SCSI接口电缆）直接挂到服务器总线上。

　　DAS方案中外接式存储设备目前主要是指RAID磁盘阵列、JBOD磁盘簇等

NAS：网络附属存储(Network Attached Storage)

NAS是数据以文件格式存放于存储端，数据格式由存储管理，用户对数据的操作不依赖于具体的应用。

存储呈现给主机是文件夹/卷的形式，主机可以挂接和共享

SAN：存储区域网络(Storage Area Network)

　　SAN连接又分ISCSI（网口）SAS（SAS口）以及FC（光纤口）连接，以硬盘块的方式显示（即通过网络的方式添加硬盘）

　　SAN是数据以数据块格式存放于存储，对存储是不可见的，数据格式由使用该数据的主机或应用进行定义。存储呈现给主机的是LUN的形式，对主机而言就是一个磁盘设备，一般是独占时访问

<img src="E:\Project\Textbook\linux云计算\assets\wps14.png" alt="img" style="zoom: 67%;" /> 

##  Samba服务

基于SMB协议（Session MessageBlock）可实现文件共享和打印机服务共享，实现在线编辑，实现登录SAMBA用户身份认证，可以进行NetBIOS名称解析，外围设备共享，主要用于windos和linux通过网络传输文件

Samba服务器组件：有两个主要的进程smbd和nmbd

smbd进程提供了文件和打印服务

nmbd提供NetBIOS名称解析服务和浏览支持，帮助SMB客户定位服务器，处理所有基于UDP的协议

服务端端口号：139，445		客户端端口号：137，138		安装包：samba	主配置文件：/etc/samba/smb.conf

<img src="E:\Project\Textbook\linux云计算\assets\wps15.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps16.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps17.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps18.jpg" alt="img" style="zoom:67%;" /> 

man smb.conf 查看samba服务帮助文档

[global]	#全局配置下，可附加如下配置

netbios name = MYSERVER	在smb服务器的电脑名称，此设置需要启动nmb服务（即可将登录ip地址换为该名称）

passdb backend = tdbsam：Samba 用户的存储方式，smbpasswd 表示明文存储，tdbsam 表示密文存储

log file = /var/log/samba/log.%I	设置Samba Server日志文件的存储位置以及日志文件名称。在文件名后加个宏%m（主机名），表示对每台访问Samba Server的机器都单独规则一个日志文件

samba配置中的宏定义:

%m客户端主机的 Netbios名			%M客户端主机的FQDN			%H当前用户家目录路径

%U当前用户的用户名				%g当前用户所属组				%h samba服务器的主机名

% L samba服务器的NetBIOS名		%I客户端主机的P				%T当前日期和时间

%S可登录的用户名

log level=2 日志的详细程度，默认为0(不规则日志文件)

deadtime = 0：服务器将自动关闭未连接会话的时间。单位是分钟，0代表Samba Server不自动切断任何连接

config file = /e	tc/samba/conf.d/%U：实现不同smb用户访问相同的共享目录redhat，实现的配置效果不同，如下

<img src="E:\Project\Textbook\linux云计算\assets\wps19.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps20.jpg" alt="img" style="zoom:67%;" /> 

[redhat]	共享名，#smclient访问的共享名

comment：对该共享的描述					path = /home/share/%m：共享目录路径（可以使用宏定义）

browseable：该共享是否可以浏览				public：该共享是否允许匿名访问（默认为no）

writable：共享路径是否可写					write list = bobyuan，+student，@bob EXCEPT 192.168.1.15

允许写入该共享的用户或组（except除某某以外）。writable选项优先级低于该选项

valid / invalid users = alex，+student，@bob：允许 / 禁止访问共享资源的用户或组

max connection：最大连接数					hosts allow：主机访问的主机地址段（deny拒绝）

admin users = bobyuan，jane：该共享的管理者	available：该共享资源是否可用

printable：是否可打印（值为yes时客户端看不到任何东西）

create mask = 0775：客户端上传文件的默认权限	directory mask = 0775：客户端创建目录的默认权限

guest account = nobody：指定匿名登录的账户为nobody

display charset = UTF8：置显示的字符集			guest ok = yes/no：意义同“public”

testparm：检查格式

<img src="E:\Project\Textbook\linux云计算\assets\wps21.jpg" alt="img" style="zoom:67%;" /> 

修改对象（文件）的安全上下文，比如：用户		角色	：		类型	：级别		：	属性

chcon -Rt samba_share_t  /joinlabs/	在创建的文件和目录上设置SELinux标签

samba_enable_home_dirs #共享用户家目录				samba_export_all_rw #共享新建的文件夹

allow_smbd_anon_write #samba允许匿名用户可写 			smbd_disable_trans #允许daemon启动samba

use_samba_home_dirs #允许本机访问远程samba根目录  	allow_rsync_anon_write #允许匿名用户可写

rsync_disable_trans #允许daemon启动rsync

-R， --recursive：递归处理所有的文件及子目录。 				-t， --type=类型：设置指定类型的目标安全环境。

-r， --role=角色：设置指定角色的目标安全环境。				-l， --range=范围：设置指定范围的目标安全环境

-h， --no-dereference：影响符号连接而非引用的文件。		-v， --verbose：为处理的所有文件显示诊断信息。 		-u， --user=用户：设置指定用户的目标安全环境。		--reference=参考文件：使用指定参考文件的安全环境，而非指定值

###  Smb用户设置

client端安装smb-common-tools包

添加smb账户useradd -s  /sbin/nologin user1				#smb的用户必须是系统用户，且最好是/sbin/nologin

smbpasswd：为系统创建用户添加 \ 修改smb账号密码，管理smb账户

-a，add添加\ 修改smb用户密码	-x，delete删除smb用户			smbpasswd -a user1

-d，disable禁用smb用户			-e，enable启用smb用户		smbpasswd -d user1

pdbedit：管理Samba用户数据库	-a 建立smb账户	-x 删除	-L 输出账户列表	-v 列出账户详细信息

用户数据库文件：/var/lib/samba/private/passdb.tdb

smbclient -L 192.168.127.10列出共享资源

smbclient //192.168.127.10/共享名 -U user%123456	user登录samba(客户端安装samba-client)

smbstatus：查看smb服务器被连接状态列表

###  CIFS通用网络文件系统协议

cifs用于UNIX和windows间共享，而NFS用于UNIX和UNIX之间共享。NAS是一个设备，一个功能。而CIFS/NFS是一种协议

需安装cifs-utils包

一次性挂载：mount -o username=user1，password=123  //192.168.0.109/redhat  /media

![img](E:\Project\Textbook\linux云计算\assets\wps22.jpg) 

multiuser多用户挂载机制：管理员只需作一次挂载，其他用户登录只需提权即可，无需重新挂载

multiuser：普通用户可以cifs借权具有写权限		sec：指定认证方式

![img](E:\Project\Textbook\linux云计算\assets\wps23.jpg) 

cifscreds add -u zhubajie 192.168.0.22	当使用系统其他用户时，需cifscreds借权，借别人的账号去具有写权限

cifscreds update -u zhubajie 192.168.0.22	更新cifscreds借权

在windos中使用net use \\192.168.1.102\IPC$ /del 删除系统缓存账户，或者在windos搜索凭证管理器删除对应凭证

###  配置过程中可能遇到的问题

1、客户端登录samba时出现以下提示：

session setup failed: NT_STATUS_LOGON_FAILURE

该错误提示表示用户有误，可能是用户不存在，也可能是密码错误，或者只是在samba用户和系统用户及密码出现错误，总之就是用户和密码的问题

tree connect failed: NT_STATUS_BAD_NETWORK_NAME

该错误表示坏的网络名，表示共享目录不存在，或共享目录权限问题，可用setfacl -m给用户加权限

Connection to 192.168.4.7 failed (Error NT_STATUS_HOST_UNREACHABLE)

防火墙或selinux问题重启

2、客户端连接到samba共享目录时出现以下提示：

smb: \> ls

NT_STATUS_ACCESS_DENIED listing \*

文件权限不足，或者存在selinux限制，调整文件的权限，并打开selinux开关

3、执行setsebool -P 操作启用SElinux开关参数时失败，提示：Killed

内存不足，而且交换空间也不足，添加交换分区（1GB）在重试

4、用put上传文件时提示NT_STATUS_OBJECT_PATH_NOT_FOUND opening remote file \/etc/passwd

　　原因：文件目标远

　　解决：切换到文件所在目录下，再用smbclient命令登录并用put命令上传，不用绝对路径

##  FTP服务（vsery secure FTP Daemon）

<img src="E:\Project\Textbook\linux云计算\assets\wps24.jpg" alt="img" style="zoom: 67%;" /> 

包和服务名：vsftpd		数据端口port：20	只负责数据传输		指令端口port：21	以命令方式负责该服务的数据开启

<img src="E:\Project\Textbook\linux云计算\assets\wps25.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps26.jpg" alt="img" style="zoom:50%;" /> 

服务端被动模式数据端口：227 Entering Passive Mode（172.16.0.1，224，59）	224*256+59

```sh
listen=NO 设置为YES时vsftpd以独立运行方式启动，设置为NO时以xinetd方式启动（xinetd是管理守护进程的，将服务集中管理，可以减少大量服务的资源消耗）
listen_ipv6=YES 以上两个只能一个YES一个NO否则会出错
listen_port=21 设置控制连接的监听端口号，默认为21
listen_address=ip地址 将在绑定到指定IP地址运行，适合多网卡
nopriv_user=nobody 指定运行vsftpd服务的用户身份，默认
download_enable=yes|no 是否允许下载文件
connect_from_port_20=YES/NO 若为YES，则强迫FTP－DATA的数据传送使用port 20，默认YES
ftp_data_port=20 指定主动端的端口，默认20
idle_session_timeout=300 设置多长时间不对FTP服务器进行任何操作，则断开该FTP连接，单位为秒
accept_timeout=60	建立FTP连接超时时间，单位秒
connection_timeout=60 PORT方式下建立FTP数据连接超时时间，单位秒
data_connection_timeout=60 数据传输60s超时时间，单位秒
pasv_enable=YES/NO 是否开启被动模式进行数据传输，有的客户机在防火墙后面，所以建议开启
pasv_min_port=n	pasv_max_port=m	设置被动模式后的数据连接端口范围在n和m之间
max_clients=n 在独立启动时限制服务器的连接数，0表示无限制
max_per_ip=0 同一IP地址的最大连接数，0表示无限制
use_localtime=YES 使用本地时间
```

登录提示信息：

```sh
ftpd_banner=Welcome to blahFTP service 服务ftp的欢迎信息
banner_file=/etc/vsftpd/banner_file.txt  设置服务ftp的欢迎信息文件，优先级大于ftpd_banner选项
```

目录访问提示信息：

```sh
dirmessage_enable=YES 是否显示目录说明文件，需要手工创建.message文件
message_file=.message 此文件放入需要提示的共享文件夹中，默认
```

vim .message 	#以带有颜色的方式显示

^[[31m  Tips ^[[0m	#同时摁ctrl+v+[ 生成^[  符号，再加上[ 符号，31m为红色，32绿色，33黄色，0m表示结束

或者echo重定向也有颜色：echo -e '\E[32m Tips \E[0m' >>/var/ftp/pub/.message

wu-ftp日志：默认启用

```sh
xferlog_enable=YES 开启日志规则上传下载日志，默认启用	xferlog_std_format=YES 使用wu-ftp日志格式，默认启用
xferlog_file=/var/log/xferlog 默认日志文件
```

vsftpd日志：默认不启用

```sh
dual_log_enable=YES 使用vsftpd日志格式，默认不启用		vsftpd_log_file=/var/log/vsftpd.log 可自动生成，默认路径
```

服务器用户根目录（如/var/ftp）不能拥有写权限，如果具有写权限，那所有用户不能登录

解决方法1：在根目录下创建777权限目录	chmod 777 share

2：根目录具有写权限添加：

```sh
allow_writeable_chroot=YES 允许被禁锢在 FTP 根目录的用户有写权限，而且不拒绝用户的登录请求
```

打开ftp服务共享功能，开启 SELinux 域中对 FTP 服务的允许策略

```sh
setsebool -P更改sebool值		getsebool -a查看所有sebool值
ftpd_anon_write=on			ftpd_full_access=on
allow_ftpd_anon_write #允许vsvtp匿名用户写入权限			ftp_home_dir #ftp用户可以访问自己的家目录的话
ftpd_is_daemon #vsftpd以daemon的方式运行				ftpd_disable_trans 1 #关闭SELINUX对ftpd的保护
```



###  匿名开放模式

匿名开放模式：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP 服务器，但是会锁定在默认目录

默认只读权限，匿名用户只能对根目录下/var/ftp某一子目录设置写权限

设置共享出去的目录的权限	setfacl -mR u:ftp:rwx  /var/ftp/pub

```sh
anonymous_enable=YES 允许匿名登陆					anon_mkdir_write_enable=YES 允许匿名用户创建目录
anon_upload_enable=YES 允许匿名用户上传下载，#它的父目录要有可写权限
anon_other_write_enable=YES 允许匿名用户有其他写权限（重命名，删除等）
anon_world_readable_only=YES  允许匿名用户下载a+r的只读文件
anon_root=/var/ftp 匿名用户默认的FTP根目录
no_anon_password=YES 匿名登陆不需要密码
anon_umask=0333 匿名用户上传文件的umask值
anon_max_rate=0 匿名用户的最大传输速率，单位为B/s，0表示不限制
chown_uploads=YES 开启匿名用户账户映射			chown_username=whoever 将匿名用户映射为whoever用户
chown_upload_mode=0644 设置映射用户上传文件的权限
```

###  本地用户模式

本地用户模式：通过服务端系统本地的账户密码进行认证的模式，相较于匿名开放模式更安全。由于密码是明文如果被黑客通过抓包等破解了账户的信息。本模式默认用户能随意切换目录，具有读写权限

```sh
write_enable=YES 登陆用户是否有写权限，全局设置			local_enable=YES 允许本地用户登录
local_root= 指定所有本地用户的FTP根目录，默认为用户各自家目录	local_umask=022 本地用户模式创建文件umask 值
chroot_local_user=YES 将本地用户禁锢在FTP根目录，此时用户不能登录，设置该用户家目录为555即可，或打开以下选项
allow_writeable_chroot=YES 允许被禁锢在 FTP 根目录的用户有写权限，而且不拒绝用户的登录请求
chroot_list_enable=YES 本地用户禁锢列表打开
chroot_list_file=/etc/vsftpd/chroot_list 默认文件
```

1、 当chroot_local_user=YES和chroot_list_enable=YES时，该文件是不禁锢名单，即白名单

2、 当chroot_local_user=NO，该文件是禁锢名单，即黑名单

```sh
userlist_enable=YES  开启用户白名单文件vsftpd.user_list功能
userlist_deny=YES	开启“禁止用户名单”名单文件有 ftpusers 和user_list
ftpusers是黑名单，user_list有白名单和黑名单两种功能
guest_enable=yes 将本地用户账户映射为guest用户	，此时默认的根为/var/ftp，可以通过local_root修改默认根
guest_username=vsftp 将guest用户再映射为vsftp用户
anon_max_rate=0 本地用户的最大传输速率，单位为B/s，0表示不限制
```

更改该用户家目录属性		chcon -R -t public_content_t /var/ftp

<img src="E:\Project\Textbook\linux云计算\assets\wps27.jpg" alt="img" style="zoom:67%;" /> 

![img](E:\Project\Textbook\linux云计算\assets\wps28.jpg) 

###  虚拟用户模式

虚拟用户模式：是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供 FTP 服务程序进行认证使用。即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响

所有虚拟用户会统一映射为一个指定的系统账户：访问共享位置，即映射账户的家目录，同时禁止非认证文件内的所有用户登录

各虚拟用户可被赋予不同的访问权限，通过匿名用户的权限控制参数进行指定

虚拟用户的储存方式：文件编辑文本文件，文件需要被编码为hash格式，

用哈希（hash）算法将原始的明文信息文件转换成数据库文件

1、vim vusers.list编辑虚拟用户表【单数行:用户名，双数行：密码】![img](E:\Project\Textbook\linux云计算\assets\wps29.jpg)

db_load -T -t hash -f  vusers.list  vusers.db用 db_load

-T让非db用户的程序使用创建出来的db文件		-t读取文件的方式 		-f形成文件(能够让非db程序访问)

chmod 	600  vusers.db修改数据库权限			删除原有明文信息文件vuser.list

2、使用pam_userdb.so模块：功能通过db=所对应文件进行验证		系统默认认证文件/etc/pam.d/vsftpd

创建一个新的模块vim /etc/pam.d/vsftpd.vu【第一行密码	第二行账号】

<img src="E:\Project\Textbook\linux云计算\assets\wps30.jpg" alt="img" style="zoom:67%;" /> 

3、打开虚拟用户表，指定认证文件，设置权限文件夹

```
guest_enable=YES虚拟用户模式开启			guest_username=test指定虚拟映射的本地用户名
pam_service_name=vsftpd.vu指定认证文件
\#user_config_dir=/etc/vsftpd/vusers_dir虚拟用户不同的访问权限文件夹
\#allow_writeable_chroot=YES 允许对禁锢的 FTP 根目录执行写入操作，且不拒绝用户的登录请求
```

4、将虚拟用户的映射账号到本地用户

```sh
useradd test -s /sbin/nologin(不能登录服务器但能登录ftp) -d /var/ftproot/
```

添加虚拟用户账号及虚拟用户登陆进来的主目录，在主目录不能写，设置一个子目录，供用户上传数据

```
mkdir /var/ftproot/ftp
chmod 777 /var/ftproot/ftp修改虚拟用户主目录权限
```

5、建立虚拟用户设置不同的访问权限文件（设置按匿名用户操作）	mkdir /etc/vsftpd/vusers_dir

```
vim /etc/vsftpd/vusers_dir/vuser1（用户） 	
anon_mkdir_write_enable=YES 允许匿名用户创建目录
anon_upload_enable=YES 允许匿名用户上传下载，#它的父目录要有可写权限
anon_other_write_enable=YES 允许匿名用户有其他写权限（重命名，删除等）
local_root= /share 指定该用户根目录
```



###  基于msql验证的ftp虚拟用户

创建两台centos主机，一台安装vsftpd，一台安装mariadb数据库

 

##  Lftp客户端工具

<img src="E:\Project\Textbook\linux云计算\assets\wps31.jpg" alt="img" style="zoom:67%;" /> 

##  Tftp简单文件传输协议

TFTP（Trivial File Transfer Protocol）	只能从文件服务器上获得或写入文件，不能列出目录，不进行认证

端口port:69/UDP协议				安装包：tftp.x86_64 tftp-server.x86_64  xinetd.x86_64 

vim 配置文件/etc/xinetd.d/tftp

service tftp

```
{	socket_type	= dgram
​    protocol		= udp
​    wait			= yes
​    user			= root
​    server		= /usr/sbin/in.tftpd
​    server_args	= -s /var/lib/tftpboot	#服务器根目录，共享文件夹
​    disable		= no					#禁用服务
​    per_source	= 11
​    cps			= 100 2
​    flags		= IPv4				}
```



##  NFS网络文件系统（Network File System）

NFS是基于内核的文件系统，可以将远程的计算机磁盘挂载到本地，读写文件像访问本地磁盘一样操作，可以共享彼此的文件。由于端口多，主要用于局域网使用

原理：NFS本身的服务并没有提供数据传递的协议，而是通过使用RPC（远程过程调用 Remote Procedure Call）来实现。当NFS启动后，会随机的使用一些端口，NFS就会向RPC注册中心提交这些端口。RPC就会规则下这些端口，RPC会开启111端口。通过client端和sever端端口的连接来进行数据的传输。因此在启动nfs之前，首先要确保rpc服务启动

<img src="E:\Project\Textbook\linux云计算\assets\wps32.jpg" alt="img" style="zoom:67%;" /> 

守护进程，连带进程rpc-bind、nfs-server		端口prot：111		包：nfs-utils（包含服务端与客户端相关工具）

编辑主配置文件：/etc/exports				#可有多个文件夹、网段

/public		192.168.0.0/24(rw，sync)	#默认只读ro，默认客户机所有用户映射成服务端nfsnobody用户

\#当开启映射条件后，默认拒绝匿名用户

\#sync 同步模式，数据直接写入磁盘						async 异步模式，数据先到内存，再转磁盘

\#hide 隐藏文件系统									noaccess 阻止访问这个目录及其子目录

\#wdelay 为合并多次更新而延迟写入磁盘					no_wdelay 尽可能快地把数据写入磁盘

\#all_squash 将所有本地和远程账户映射到匿名用户			no_root_squash 将远程根用户映射成本地根用户

\#root_squash 将根用户及所属组都映射为匿名用户或用户组（nfsnobody默认）

\#anonuid=1 为匿名账户指定用户ID	（客户端系统根据对应ID填入用户）	anongid=1 为匿名账户指定组ID

\#subtree_check  验证每个被请求的文件都在导出的目录树中	no_subtree_check 只验证涉及被导出的文件系统的文件请求	

exportfs：管理NFS共享文件系统列表			-au临时关闭所有共享

-a：打开取消所有目录共享	-r：重载配置文件	-u：取消一个或多个目录的共享		-v：输出共享的目录详细信息

### 使用

showmount -e 192.168.0.22查看服务器22的共享文件夹

mount -t nfs 192.168.0.109:/www  /public	指定挂载类型，nfs挂载文件umount命令)

192.168.0.109：/www		/www		nfs		defaults，_netdev	0	0		#_netdev无网络不挂载

win10挂载NFS服务：程序-->启用windos功能-->NFS服务-->此电脑-->映射驱动器中添加nfs地址，和要共享的文件夹

nfs_export_all_ro #将本机的NFS共享设置成只读		nfs_export_all_rw #将本机的NFS共享设置成可读可写

use_nfs_home_dirs #将远程NFS的家目录共享到本机

##  iscsi网络存储服务(类似samba可跨系统)

SCSI是一种用于计算机和硬盘、光驱等设备之间系统级接口的通用标准，具有系统资源占用率低、转速高、传输速度快等优点

SCSI（Small Computer System Interface）是一种I/O技术，规范了一种并行的I/O总线和相关的协议，SCSI的数据传输是以块的方式进行的

SCSI控制器称为Target，访问的客户端应用称为Initiator。窄SCSI总线最多允许8个、宽SCSI总线最多允许16个不同的SCSI设备和它进行连接，每个SCSI设备都必须有自己唯一的SCSI ID（设备的地址）

LUN（Logical Unit Number，逻辑单元号）是为了使用和描述更多设备及对象而引进的一个方法，每个SCSI ID上最多有32个LUN，一个LUN对应一个逻辑设备

广泛应用于小型机上，正在成为PC 服务器的标准接口，实现高速数据传输（可达320MB/s），常见的SCSI设备：硬盘、磁盘阵列、打印机、光盘刻录机等

工作过程：

1、Initiator发出请求后，会在本地的操作系统会生成了相应的SCSI命令和数据I/O请求，然后这些命令和请求被封装加密成IP信息包，通过以太网（TCP/IP）传输到Targer

2、当Targer接收到信息包时，将进行解密和解析，将SCSI命令和I/O请求分开。SCSI命令被发送到SCSI控制器，再传送到SCSI存储设备

3、设备执行SCSI命令后的响应，经过Target封装成iSCSI响应PDU，再通过已连接的TCP/IP网络传送给Initiator

4、Initiator会从iSCSI响应PDU里解析出SCSI响应并传送给操作系统，操作系统再响应给应用程序

<img src="E:\Project\Textbook\linux云计算\assets\wps33.jpg" alt="img" style="zoom:80%;" />包与服务:target*	工作在tcp3260端口

targetcli命令：用于管理iSCSI服务端存储资源的专用配置命令，能够提供交互式配置功能

<img src="E:\Project\Textbook\linux云计算\assets\wps34.jpg" alt="img" style="zoom:67%;" /> 

/backstores/block是iSCSI服务端配置共享设备的位置

1、 创建创建块(即给要发布的逻辑卷起一个名字)

将 /dev/sda3分区加入网络存储，并重名为redhat，这样用户就不知道是哪块硬盘来提供共享存储资源

<img src="E:\Project\Textbook\linux云计算\assets\wps35.jpg" alt="img" style="zoom:67%;" /> 

2、创建 iqn 名字(即创建ISCSI对象)

<img src="E:\Project\Textbook\linux云计算\assets\wps36.jpg" alt="img" style="zoom:67%;" /> 

自动创建iSCSI target配置共享资源名称。由系统自动生成的，用于描述共享资源的唯一字符串

<img src="E:\Project\Textbook\linux云计算\assets\wps37.jpg" alt="img" style="zoom:67%;" /> 

3、设置访问控制列表（acls），acls参数目录用于存放能够访问iSCSI服务端共享存储资源的客户端名称

iSCSI协议是通过客户端名称进行验证的，即用户在访问存储共享资源时不需要输入密码，只要iSCSI客户端的名称与服务端中设置的访问控制列表中某一名称条目一致即可

![img](E:\Project\Textbook\linux云计算\assets\wps38.jpg) 

<img src="E:\Project\Textbook\linux云计算\assets\wps39.jpg" alt="img" style="zoom:67%;" /> 

4、设置luns共享名关联，即将ISCSI共享对象与主机名绑定

<img src="E:\Project\Textbook\linux云计算\assets\wps40.jpg" alt="img" style="zoom:67%;" /> 

5、设置iSCSI服务端的监听IP地址和端口号，系统自动开启服务器对应网卡的3260端口将向外提供iSCSI共享存储资源服务

![img](E:\Project\Textbook\linux云计算\assets\wps41.jpg)删除原有端口号

<img src="E:\Project\Textbook\linux云计算\assets\wps42.jpg" alt="img" style="zoom:67%;" /> 

/iscsi/iqn.20...hel:rhel/tpg1> get attribute 

authentication=0		//认证关闭

<img src="E:\Project\Textbook\linux云计算\assets\wps43.jpg" alt="img" style="zoom:67%;" />//写保护模块关闭(让客户可写)

<img src="E:\Project\Textbook\linux云计算\assets\wps44.jpg" alt="img" style="zoom:67%;" />//生成节点列表开启

使用Exit命令退出后生成文件，主配置vim /etc/target/saveconfig.json

###  Linux客户端

安装iSCSI客户端服务程序 yum install iscsi-initiator-utils【centos 7默认已安装】

### 1、启动客户端iscsid

service iscsid start	&& systemctl daemon-reload(重新加载)

iSCSI 客户端访问 “先发现，再登录，最后挂载并使用” 。iscsiadm 是用于管理、查询、插入、更新或删除 iSCSI 数据库配置文件的命令行工具，用户需要先使用这个工具扫描发现远程 iSCSI 服务端，然后查看找到的服务端上有哪些可用的共享存储资源

-m {discovery|node|session|iface} （--mode discoverydb）

\#{扫描发现某服务器target输出的可用的存储资源 | 管理跟某target的关联关系 | 会话管理 | 接口管理}

-d {0-8} (--discover) #发现设备，查看名称，打印调试信息，有0到8这9个等级

-t st （--type sendtargets）#这里可以使用的类型为sendtargets(可简写为st)、slp、fw和 isns，此选项仅用于discovery模式，且目前仅支持st、fw和isns；其中st表示允许每个iSCSItarget发送一个可用target列表给initiator

-p 192.168.10.10（--portal）#指定target服务的IP和端口

-o	#指定针对discoverydb数据库的操作，其仅能为new、delete、update、show和nonpersistent其中之一

-I	#指定执行操作的iSCSI接口，这些接口定义在/var/lib/iscsi/ifaces中

-l	#登录节点										-u	#登出节点(服务器)

[root@linuxprobe ~]# iscsiadm -m discovery -t st -p 192.168.10.10

[root@linuxprobe ~]# vim /etc/iscsi/initiatorname.iscsi #写入acls需要的认证信息

InitiatorName=iqn.2003-01.com.redhat:rhel

### 2、 登录 iSCSI 服务端

-T  iqn.2003-01.com.redhat:rhel #指定要使用的存储资源		 -l，--login参数进行登录验证，--logout注销登录

[root@linuxprobe ~]# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe. x8664:sn.d497c356ad80 -p 192.168.127.1 -l

由于设备名重启后会改变，先格式化iSCSI硬盘得到UUID号， 再进性挂载

（如果忘记加_netdev字段重启后，客户端一直在找，无法正常启动，须umount再修改fstab文件）

![img](E:\Project\Textbook\linux云计算\assets\wps45.jpg) 

如果我们不再需要使用iSCSI共享设备资源了，可以用iscsiadm命令的-u参数将其设备卸载

[root@linuxprobe ~]# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.d497c356ad80 -u

在linux 环境中登录挂载需要进行CHAP验证的 ISCSI Target

1、检查是否可以发现

iscsiadm --mode discovery --type sendtargets --portal 192.168.1.55

2、登入需验证码的节点，在登陆前需执行

iscsiadm -m node -T iqn.1994-05.com.redhat:wsfnk -p 192.168.1.55 -o update --name node.session.auth.authmethod --value=CHAP  # 先开启验证

iscsiadm -m node -T iqn.1994-05.com.redhat:wsfnk -p 192.168.1.55 -o update --name node.session.auth.username --value=yonghu # 添加用户

iscsiadm –m node –T iqn.1994-05.com.redhat:wsfnk -p 192.168.1.55 -o update --name node.session.auth.password --value=yonghu-password # 添加密码

3、测试挂载设备

iscsiadm -d2 -m node -T iqn.1994-05.com.redhat:wsfnk -p 192.168.1.55 --login   #挂载

###  Windos客户端

控制面板—小图标-管理工具--iSCSI 发起程序—目标—快速连接

计算机管理—磁盘存储—找到iSCSI共享—格式化

##  网络自动挂载 Autofs服务	

<img src="E:\Project\Textbook\linux云计算\assets\wps46.jpg" alt="img" style="zoom:67%;" /> 

安装包：autofs包	相对路径法

<img src="E:\Project\Textbook\linux云计算\assets\wps47.jpg" alt="img" style="zoom:67%;" /> 

当用户到达挂载文件夹内，通过特定的触发器访问到物理或网络资源设备

触发器：是一个虚拟文件夹，用来触发对应的挂载路径

选项	-fstype= nfs（iso9660，samba）

soft软资源，除硬盘是硬资源，其他全是软资源		Intr可中断地挂载 	--timeout超时时间5秒（主配置文件）

<img src="E:\Project\Textbook\linux云计算\assets\wps48.jpg" alt="img" style="zoom:67%;" /> 

vim /etc/sudoers		sudo命令文件

<img src="E:\Project\Textbook\linux云计算\assets\wps49.jpg" alt="img" style="zoom: 67%;" /> 

zhubajie用户能够在任何地方使用useradd和userdel命令

sudo -l	查看用户所有特权

NOPASSWD	不需要密码就能使用

%wheel行	这个组在任何地方都不需要命令

<img src="E:\Project\Textbook\linux云计算\assets\wps50.jpg" alt="img" style="zoom:67%;" /> 

将useradd和userdel的命令赋予给TEST

##  网络自动挂载 Autofs服务	

<img src="E:\Project\Textbook\linux云计算\assets\wps51.jpg" alt="img" style="zoom:67%;" /> 

安装包：autofs包	相对路径法

<img src="E:\Project\Textbook\linux云计算\assets\wps52.jpg" alt="img" style="zoom:67%;" /> 

当用户到达挂载文件夹内，通过特定的触发器访问到物理或网络资源设备

触发器：是一个虚拟文件夹，用来触发对应的挂载路径

选项	-fstype= nfs（iso9660，samba）

soft软资源，除硬盘是硬资源，其他全是软资源		Intr可中断地挂载 	--timeout超时时间5秒（主配置文件）

<img src="E:\Project\Textbook\linux云计算\assets\wps53.jpg" alt="img" style="zoom:67%;" /> 

vim /etc/sudoers		sudo命令文件

<img src="E:\Project\Textbook\linux云计算\assets\wps54.jpg" alt="img" style="zoom:67%;" /> 

zhubajie用户能够在任何地方使用useradd和userdel命令

sudo -l	查看用户所有特权

NOPASSWD	不需要密码就能使用

%wheel行	这个组在任何地方都不需要命令

<img src="E:\Project\Textbook\linux云计算\assets\wps55.jpg" alt="img" style="zoom:67%;" /> 

将useradd和userdel的命令赋予给TEST

 

#  DNS服务器（域名解析系统）

<img src="E:\Project\Textbook\linux云计算\assets\wps56.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps57.jpg" alt="img" style="zoom:67%;" /> 

 <img src="E:\Project\Textbook\linux云计算\assets\wps58.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps59.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps60.jpg" alt="img" style="zoom:67%;" />

named_write_master_zones #允许修改dns的主zone文件		named_disable_trans #允许daemon启动named

安装包：bind	服务名named主配置文件/etc/named.conf

```
options {    //options段用于定义全局设置
listen-on port 53 { 192.168.127.10; };    //定义bind的监听IP地址(IPv4) 
listen-on-v6 port 53{ ::1; };  //定义bind的监听IP地址(IPv6) 
directory "/var/named";     //zone文件的默认路径
dump-file "/var/named/data/cache_dump.db";         //cache的备份
statistics-file "/var/named/data/named_ stats.txt";↓    //静态文件
memstatistics-file "/var/named/data/named_ mem_ stats.t";  //内存静态文件
allow-query { any; };		//允许谁向此DNS进行查询
#allwo-transfer { ip地址 }；	//主备模式时允许区域传送的主机；白名单
recursion yes;				//允许递归查询
}

logging {				//定义日志
	channel my_file {   //定义channel名称
	file "data/named.run";    //以文件形式存储日志
	severity dynamic;  //存储日志的级别，一共7个级别从高到低分别是：crit，error，warning.notice，info(前面5个属于syslog);debug[level]，dynami(后两个属于Bind8，9独有的级别)
};
  category statistics { my_file; };   //定义bind系统中各子系统的日志
  //将日志发给channel，可以发给多个channel，一个channel只能接受一个category
};
```

```
zone "." IN { //定义Dns的zone，"."代表根区域
	type hint;	//定义zone的类型，根区域的类型就为hint
	file "named.ca";	//指定zone文件，默认已经生成
};
```

named-checkconf /etc/named.conf	查看主配置文件是否有错误

<img src="E:\Project\Textbook\linux云计算\assets\wps62.jpg" alt="img" style="zoom: 80%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps63.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps64.jpg" alt="img" style="zoom:67%;" />解析多个域可以对应同一个网段

allow-update {}：//允许更新区域数据库中的内容

<img src="E:\Project\Textbook\linux云计算\assets\wps65.jpg" alt="img" style="zoom: 80%;" /> <img src="E:\Project\Textbook\linux云计算\assets\wps66.jpg" alt="img" style="zoom: 80%;" />

Dig	 -t axtr redhat.com正向挖掘的redhat.com

##  添加dns从服务器

1，在主服务器著配置文件里面添加一行

<img src="E:\Project\Textbook\linux云计算\assets\wps67.jpg" alt="img" style="zoom: 80%;" />Allow-transfer行		允许0.28主机当做从服务器

2在从服务器修改主配置文件

![img](E:\Project\Textbook\linux云计算\assets\wps68.jpg)<img src="E:\Project\Textbook\linux云计算\assets\wps69.jpg" alt="img" style="zoom:80%;" /> 

File “slaves/joinlabs.zone	要拿的文件						Maters{192.168.0.22} 从0.22主机那拿到

#  Mail电子邮件服务器

mail服务    postfix      sendmail

简单邮件传输协议（Simple Mail Transfer Protocol，SMTP）用于发送和中转发出 的电子邮件，占用服务器的 25/TCP 端口

邮局协议版本 3（Post Office Protocol 3）用于将电子邮件存储到本地主机，占用服务器的 110/TCP 端口

Internet 消息访问协议版本 4（Internet Message Access Protocol 4）：用于在本地主机上访问邮件，占用服务器的 143/TCP 端口

##  postfix邮件服务器

postfix是Wietse Venema在IBM的GPL协议之下开发的MTA（邮件传输代理）软件

Postfix试图更快、更容易管理、更安全，同时还与sendmail保持足够的兼容性，因此它是免费的

postfix的产生是为了替代传统的sendmail。相较于sendmail，postfix在速度，性能和稳定性上都更胜一筹

现在主流邮件服务都在采用postfix. 当需要一个轻量级的的邮件服务器时，postfix也是一种选择<img src="E:\Project\Textbook\linux云计算\assets\wps70.jpg" alt="img" style="zoom: 67%;" />![img](E:\Project\Textbook\linux云计算\assets\wps71.jpg)

redhat默认安装有postfix邮件，包名与服务名：postfix，主配置文件vim /etc/postfix/main.cf	 #默认无需修改配置

myhostname = sample.test.com　 #设置系统的主机名

mydomain = test.com　 #设置域名（我们将让此处设置将成为E-mail地址“@”后面的部分） 

myorigin = $mydomain　 #将发信地址“@”后面的部分设置为域名（非系统主机名） 

inet_interfaces = all　 #接受来自所有网络的请求 

mydestination = $myhostname， localhost.$mydomain， localhost， $mydomain　 #指定发给本地>邮件的域名

mynetworks #设置可转发哪些主机的邮箱

relay_domains #设置可转发网域的邮箱

home_mailbox = Maildir/　 #指定用户邮箱目录

relayhost = [gateway.my.domain] #转发邮件(接到邮件会转发一份)

local_transport=error:local  #表示本地所有用户都拒收任何邮件

/var/spool/mail/alex邮件池				/var/mail/root

发邮件格式：echo I am root | mail -s(主题) hello alex@rhel.redhat.com(不能用ip地址)

建立corntab周而复始任务

```
30 7 * * *	echo good morning | mail -S happy zhubaj ie@desktop22.example.com
```

##  Dovecot服务

Dovecot是一个安全性较好的POP3/IMAP服务器软件，响应速度快而且扩展性好

POP3 / IMAP 是 MUA 从邮件服务器中读取邮件时使用的协议。其中，POP3是从邮件服务器中下载邮件，而IMAP则是将邮件留在服务器端直接对邮件进行管理、操作。

Dovecot使用PAM方式（ Pluggable Authentication Module，可插拔认证模块）进行身份认证，以便识别并验证系统用户，通过认证的用户才允许从邮箱中收取邮件。对于RPM方式安装的dovecot，会自动建立该PAM文件

##  Sendmail邮件服务

#  HTTP服务

##  Nginx

nginx(engine x) 是一款自由的、开源的、高性能的HTTP服务器和反向代理服务器；同时也是一个IMAP、POP3、SMTP代理服务器；nginx可以作为一个HTTP服务器进行网站的发布处理，另外nginx可以作为反向代理进行负载均衡

正向代理和反向代理：通常情况下我们访问网址，但由于流量巨大，转接先访问代理服务器，由代理接受请求转给主服务器(此时的服务器并不知道具体客户机的请求，将数据统统交给正向代理服务器)，反向恰好相反，客户机发送请求却不知道具体主机

负载均衡：代理服务器接受的请求就是负载量，将请求按照规则发给不同服务器处理称为负载均衡的实现

​	![img](E:\Project\Textbook\linux云计算\assets\wps73.jpg)

```sh
1、yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel g++安装依赖包（可以使用epel源进行安装）
cd  /usr/local/ && wget http://nginx.org/download/nginx-1.21.1.tar.gz下载tar包		//自定义下载版本
tar -xvf nginx-1.13.7.tar.gz && cd nginx-1.13.7	解压成可编译文件，进入可编译文件夹

2、编译配置 ./configure && make && make install--->生成相应的可执行文件、配置、默认站点等文件/usr/local/nginx
useradd nginx -s /sbin/nologin添加nginx用户设置不能登录
echo "/usr/local/sbin/nginx" >>/etc/rc.local	开机启动服务的命令
```

```sh
nginx -s quit //平稳关闭Nginx stop快速关闭，可能不保存相关信息 reload重载配置 reopen重新打开日志文件
nginx -V 显示 //nginx 的版本，编译器版本和配置参数
nginx -t	//将检查配置文件的语法的正确性
nginx -?，-h //打开帮助信息
killall nginx //杀死所有nginx进程
```

**主配置文件**

vim /usr/local/nginx/conf/nginx.conf

\####### 每个指令必须有分号结束。#################

```
user root; #配置用户或者组，默认为nobody nobody
worker_processes 2;  #cpu核数允许生成的进程数
pid /nginx/pid/nginx.pid;  #指定nginx进程运行文件存放地址
error_log log/error.log; #制定日志路径	级别以此为：debug |info |notice |warn |error|crit|alert|emerg

events {
  accept_mutex on;  #设置网路连接序列化，防止惊群现象发生，默认为on
  multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
	#use epoll;    #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
worker_connections  1024;   #允许客户机连接数
}

http {		//主配置区
  include    mime.types;  #文件扩展名与文件类型映射表
  default_type  application/octet-stream; #默认文件类型，默认为text/plain
  #access_log off; #取消服务日志
  access_log log/access.log myFormat;  #combined为日志格式的默认值
  sendfile on;  #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块
  sendfile_max_chunk 100k; 每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
  keepalive_timeout 65; #连接超时时间，默认为75s，可以在http，server，location块
\#more_clear_headers "X-Powered-By:";	清除服务器及php信息，在配置文件http段添加(需要head模块)：
\#more_clear_headers "Server:"
\#add_header Cache-Control max-age=no-cache; #设置强制缓存时间，协商缓存

upstream myserver; {  //负载均衡的服务器名字myserver
	轮询（默认）：nginx默认就是轮询其权重都默认为1，服务器处理请求的顺序：ABAB
	server 192.168.100.138:8080;
	server 192.168.100.138:8081;
	热备(mater挂了转backup)
	server 127.0.0.1:8080;
	server 192.168.100.138:8081 backup;  
	加权轮询：跟据权重的大小分发给不同服务器不同数量的请求，权重越高分配越多，服务器的请求顺序为：ABBABB
 	server 192.168.100.138:8081 weight=1;
	server 192.168.100.138:8080 weight=2;

ip_hash:nginx会让相同的客户端ip请求相同的服务器	//fair根据响应时间分配，最短先分配原则
	server 127.0.0.1:8080;
	server 192.168.100.138:8081;
	ip_hash	/	fair;
● down，表示当前的server暂时不参与负载均衡。
● backup，预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机器的压力最轻。
● max_fails，允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误。
● fail_timeout，在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。

server 127.0.0.1:7878 weight=2 max_fails=2 fail_timeout=2;
server 192.168.10.121:3333 weight=1 max_fails=2 fail_timeout=1;
}

error_page 404 500 502 504 https://www.baidu.com; #错误页
server {	//多个服务时注意文件{}格式
	keepalive_requests 120 #单连接请求上限次数
	listen    80 #监听端口
	server_name  www.aa.com #监听地址    	
	location  /redhat  {  //在网页根目录下的redhat下
	root html;  #安装位置根目录/ngnix下
#proxy_pass  http://192.168.100.138:8080;  #反向代理服务器端口
#proxy_pass  http://myserver;	#负载均衡指向的服务器名字
	index index.html;  #设置默认页		//autoindex  on;自动查找，列出反文件
	deny 127.0.0.1;  #拒绝的ip
	allow 172.18.5.54; #允许的ip	} 

#动静分离（静态网页和动态网页分别存放）
#location = / |~|~*|^~ { ＃ /通用匹配= 为严格匹配，~区分大小写，~*不区分大小写，^~模糊匹配
location ~* \ . ( jpg | png | gif)$ {	#当访问以jpg结尾时直接走代理服务器8081端口，静态访问
	proxy_pass	http://192.168.100.138:8081:
	root     bb;		#nginx根目录下的文件夹， 	/bb	#绝对路径/下的文件夹
	autoindex  on;	#自动查找，列出反文件	//index index.html; 设置默认页
}	
}		}
```



###  高可用主备集群模式

<img src="E:\Project\Textbook\linux云计算\assets\wps74.jpg" alt="img" style="zoom:67%;" /> 

(1)需要两台服务器192168.100.138和192.168.100.137

(2)在两台服务器安装 nginx

(3)在两台服务器yum install keepalived.x86_64		主配置：vim /etc/keepalived.conf

Copy vi keepalived.conf

keepalived.conf:

Copy#检测脚本

```
vrrp_script chk_http_port {
  script "/usr/local/src/check_nginx_pid.sh" #心跳执行的脚本，检测nginx是否启动
  interval 2              #检测脚本执行的间隔，单位是秒
  weight -2               #执行脚本后权重变化
}

vrrp_instance VI_1 { #vrrp 虚拟ip配置
  state MASTER       # 指定keepalived的角色，MASTER为主	//BACKUP为备
  interface ens33     # 需要绑定的物理网卡
  virtual_router_id 66   # 虚拟路由编号，主从要一直
  priority 100       # 优先级，数值越大，获取处理请求的优先级越高//从服务器修改为99
  advert_int 1       # 检查间隔，默认为1s(vrrp组播周期秒数)
}

authentication { #授权访问
	auth_type PASS #设置验证类型和密码，MASTER和BACKUP必须使用相同的密码才能正常通信
	auth_pass 1111	#密码
}

track_script {
	chk_http_port       #（调用检测脚本）
}

virtual_ipaddress {
	192.168.100.100       # 定义虚拟ip(VIP)，可多设，每行一个
  }
}
```

添加检测脚本vim /usr/local/src/check_nginx_pid.sh		chmod 775 check_nginx_pid.sh

Copy#!/bin/bash

\#检测nginx是否启动了

```sh
A=`ps -C nginx --no-header |wc -l`  #nginx不带头
if [ $A -eq 0 ];then   #如果nginx没有启动就启动nginx
   systemctl start nginx         #重启nginx
   if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then   #nginx重启失败，则停掉keepalived服务，进行VIP转移
		killall keepalived
   fi
fi
```

启动两台服务器nginx和service keepalived start启动服务		<--停止服务，测试

###  Rewrite规则

Rewite 规则作用：可以实现对url的重写，以及重定向

应用：URL访问跳转，支持开发设计，如页面跳转，兼容性支持，展示效果等

rewrite  ^(.*)$  /index.html break; 会把所有的请求都重定向到 /index.html（默认页面 ）

常用的正则表达式

![img](E:\Project\Textbook\linux云计算\assets\wps75.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps76.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps77.jpg) 

使用If判断



### nginx 配置vue3前后端项目

nginx.conf文件内容

```conf
user root;
worker_processes auto;
pid /run/nginx.pid;

events {
        worker_connections 768;
        multi_accept on;
}

http {
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        gzip on;
        gzip_disable "msie6";

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;

    # 虚拟主机
    server {
        listen       80; # 监听地址以及端口
        server_name  192.168.0.27; # 外部可以访问的域名
        root   /var/www/html/dist/ ; # 本地项目地址
        index  index.html ; # 入口文件默认类型,可以接收index、index.html、index.htm文件

        #charset utf-8; # 默认字符集
        #access_log  logs/localhost.access.log  access;

        # 路由
        location /ok/ {
					  client_max_body_size 600M; //客户端 上传文件大小限制
            client_body_buffer_size 600M;

            proxy_connect_timeout 120s; //客户端连接时长设置 ，配合文件上传使用
            proxy_read_timeout 120s;

            proxy_pass http://192.168.0.41:80/ ; //需要指定端口号
            root   /var/www/html/dist/ ; # 访问根目录 nginx-1.21.0\html
            index  index.html ; # 入口文件,可以接收index、index.html、index.htm文件
            proxy_set_header   Host             $host;        # 传递域名
            proxy_set_header   X-Real-IP        $remote_addr; # 传递ip
            proxy_set_header   X-Scheme         $scheme;      # 传递协议
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
    }
}
```



#### 注意点（后端接口配置）：

如果你的配置出现404了，这个时候你可以对照检查下面的问题是不是你遇到的。

1. proxy_pass 地址后面要不要加“/”,这个取决于匹配的 /api/ 作不作为你uri的一部分，如果 /api/ 是其中一部分,则不需要带上“/”；
   反之带上。加了“/”相当于是绝对根路径，nginx 不会把location 中匹配的路径 /api/ 带上。

   ##### [举个列子]：

   ```如果你的配置跟上面一样，同时请求a.html页面：
    请求地址原本是这样： http://192.168.1.1/api/a.html;
    如果配置是这样：proxy_pass http://192.168.1.1/;（后端接口地址）
    那么请求接口地址应该变成这样： http://192.168.1.1/a.html 
   ```

2. proxy_pass的地址记得在hosts文件做ip映射，建议直接使用域名对应的ip地址。

3. location 中 ~ （区分大小写）与 ~* （不区分大小写）标识均为正则匹配。
   如果你不确定，请在location后面加上 location ~* /api/ { }这样的配置（目的：不区分“api”三个字母的大小写）。



##  Apache服务

![img](E:\Project\Textbook\linux云计算\assets\wps78.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps79.jpg) 

http协议	超文本传输协议	80		https协议	TLS/SSL协议	443

安装包：httpd包					服务名service httpd start

![img](E:\Project\Textbook\linux云计算\assets\wps80.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps81.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps82.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps83.jpg) 



```
NameVirtualHost *:80
<VirtualHost *:80>
​	ServerName [web1.abc.com](http://web1.abc.com/)
​	DocumentRoot /var/www/web1
</VirtualHost>

<VirtualHost *:80>
​	ServerName [web2.abc.com](http://web2.abc.com/)
​	DocumentRoot /var/www/web2
</VirtualHost>
```

建立子目录：子网页只能搭建在虚拟子目录下，且访问还需指定文件夹

<VirtualHost>虚拟主机				<Directory>为指定目录设置权限

DocumentRoot 网站数据目录		ServerName 网站服务器的域名				ServerAdmin 管理员邮箱

User 运行服务的用户				Group 运行服务的用户组					ServerRoot 服务目录

Listen 监听的IP地址与端口号			DirectoryIndex 默认的索引页页面

ErrorLog	错误日志文件				CustomLog 访问日志文件			Timeout 网页超时时间，默认为300秒

/etc/httpd/conf.d文件夹下的文件会被apache执行，Httpd -t 检查主配置文件

httpd_anon_write #允许httpd匿名用户可写 				httpd_enable_homedirs #允许访问用户的根目录allow_httpd_sys__anon_write #同上						httpd_enable_cgi #httpd被设置允许cgi被执行	httpd_tty_comm #允许httpd控制终端 						httpd_unified #httpd之间相互独立

httpd_builtin_ing #同httpd环境一样运行 					httpd_can_sendmail #允许httpd发送email

httpd_can_network_connect_db #httpd可以连接到数据库(如连接mysql就必须设置)

httpd_can_network_connect #httpd可以连接到网络(如连接redis就必须设置)

httpd_read_user_content #开启用户文件的访问权限(如日志文件就必须设置)

httpd_suexec_disable_trans #禁用suexec过度 

httpd_disable_trans #允许daemon用户启动httpd

httpd_read_user_content 1 #允许用户httpd访问其家目录

vim /etc/httpd/conf.d/apache.conf

<VirtualHost 192.168.2.5:80>

DocumentRoot /aa

ServerName www.aa.com

<Directory "/aa">						#目录访问控制写法

​	<RequireAll>						#请求容器，允许容器要写在拒绝容器里面

require all granted/denied		#允许 / 拒绝声明

require not ip 192.168.2.5		#拒绝的ip地址

require not host redhat.com	#拒绝的域

<RequireAll>				#允许容器

require host desktop22.redhat.com

require ip 192.168.2.6

</RequireAll>

​	</RequireAll>

</Directory>

<Directory "/www">

options indexes					#只对该目录生效

order deny，allow

deny from ilt.example.com			#拒绝该域的访问

\#deny from 192.168.1.103

allow from server23.example.com

​    </Directory>

</VirtualHost>

 

<VirtualHost 192.168.2.5:81>			#基于端口访问，修改指定端口

DocumentRoot /bb

ServerName www.bb.com				#多域名对应同一ip

<Directory "/bb">

</Directory>

</VirtualHost>

Alias  /down  "/var/www/html"			#别名设置，当浏览端访问/down时，转跳至指定文件夹

<VirtualHost 192.168.2.5:80>

DocumentRoot /cc

ServerName www.cc.com				#网页跳转

RewriteEngine on						#重定向引擎打开

RewriteRule ^(.*)$ http://www.aa.com	#重定向规则

</VirtualHost>

 

Semanage fcontext -l 管理工具

![img](E:\Project\Textbook\linux云计算\assets\wps84.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps85.jpg) 

###  创建SSL证书

![img](E:\Project\Textbook\linux云计算\assets\wps86.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps87.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps88.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps89.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps90.jpg) 

 ![img](E:\Project\Textbook\linux云计算\assets\wps91.jpg)

![img](E:\Project\Textbook\linux云计算\assets\wps92.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps93.jpg) 

###  动态网页

 

![img](E:\Project\Textbook\linux云计算\assets\wps94.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps95.jpg) 

查看端口888有无占用

![img](E:\Project\Textbook\linux云计算\assets\wps96.jpg) 

安装mod_wsgi文件，动态网页文件以wsgi结尾，服务不识别该类文件须将其别名指定在（根目录下）

Journalctl -xn查看具体报错信息

![img](E:\Project\Textbook\linux云计算\assets\wps97.jpg) 

给666端口添加apache属性

###  访问用户主页

1、vim /etc/httpd/conf.d/userdir.conf #用户个人主页配置文件

</IfModule>

\#   UserDir disabled #用户家目录关闭

UserDir public_html #允许访问的用户主页目录

</IfModule>

 

<Directory "/home/*/public_html">	#默认的用户访问目录权限设置

  AllowOverride FileInfo AuthConfig Limit Indexes

  Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec

  Require method GET POST OPTIONS

\#     require all granted			#允许所有请求，选择此配置或默认配置

</Directory>

2、用户家目录下创建public_html主页目录，echo "aa" > /home/u1/public_html/index.html（默认访问文件)

对用户家目录及下所有设置可访问权限o+rwx，我们可以在该主页目录下建立多个访问目录及文件，只要符合权限及selinux

测试http：192.168.2.3/~u1/				更改性cp -avx 	/var/www /home/rhel/l	 #复制文件属性

###  用户访问认证

vim /etc/httpd/conf.d/userdir.conf #配置所有用户个人主页配置文件

<Directory "/home/*/public_html">	#默认的用户访问目录权限设置

​    AuthType Basic	#基本的认证类型

​    AuthName "111"	#密码提示语

​    {AuthUserFile | AuthGroupFile} /etc/httpd/conf.d/htpasswd 

\#{用户|组}密码数据库文件（存放访问该站点的账号密码）

 Require {user | groups} u1	#允许登录的 {用户|组}

</Directory>

Htpasswd -c /etc/httpd/htpasswd alex	#创建认证数据库且加入用户alex

-D删除		-m加密（再创建不用-c用加密）

配置个人用户的

![img](E:\Project\Textbook\linux云计算\assets\wps98.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps99.jpg) 

让其他用户对服务器有修改用户的权限

![img](E:\Project\Textbook\linux云计算\assets\wps100.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps101.jpg) 

###  代理服务器

代理服务器：客户端想访问服务端搭建的所有站点，必须要能解析出该站点

代理服务器能看到的站点，你通过代理服务器也许能看到

代理服务器看不到的站点，你通过代理服务器也看不到

包：squid.x86_64		服务名squid	主配置：/

![img](E:\Project\Textbook\linux云计算\assets\wps102.jpg) 

第1行为代理服务器配置文件

![img](E:\Project\Textbook\linux云计算\assets\wps103.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps104.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps105.jpg)可以用localnet代替多网站，端口也可被代替

![img](E:\Project\Textbook\linux云计算\assets\wps106.jpg) 允许test，拒绝全部除safe外

![img](E:\Project\Textbook\linux云计算\assets\wps107.jpg) 

拒绝所有（由于配置从上到下精确读取，所以允许先写在前面）

![img](E:\Project\Textbook\linux云计算\assets\wps108.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps109.jpg) 

防火墙打开代理服务器端口3128，在浏览器内设置代理服务器

![img](E:\Project\Textbook\linux云计算\assets\wps110.jpg) 

##  Tomcat-apache应用服务器

开源的轻量级Web应用服务器，使用非常广泛。server.xml是Tomcat中最重要的配置文件，**server.xml的每一个元素都对应了Tomcat中的一个组件**；通过对xml文件中元素的配置，可以实现对Tomcat中各个组件的控制。

https://tomcat.apache.org/ 官网  Apache拓展服务		默认端口port：8080

/bin/startup.sh开启服务	主配置文件：/conf/ server.xml

1. bin --启动命令目录  2 conf --配置文件目录   3 lib --库文件目录 	4 logs --日志文件目录
2. temp --临时缓存文件		6 webapps/ROOT/index.jsp  --网页家目录   7 work --工作缓存目录

<Server port="8005" shutdown="SHUTDOWN">  关闭指令端口

​	<Connector port="8080" protocol="HTTP/1.1"    http请求访问端口http://localhost:8080

​	redirectPort="8443" />			 https请求访问端口

​	<Connector protocol="AJP/1.3"

​        	port="8009"			AJP协议的处理端口

直接启动访问8080端口

复制多个tomcat服务		cp -r tomcat1 tomcat2

修改端口号为8006		8081	8444	8010启动

 

#  PXE无人值守

![img](E:\Project\Textbook\linux云计算\assets\wps111.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps112.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps113.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps114.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps115.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps116.jpg) ![img](E:\Project\Textbook\linux云计算\assets\wps117.jpg)

##  1安装DHCP服务器，主配置文件/etc/dhcp/dhcpd.conf，让客户机去127.10服务器拿pxelinux文件

##  2安装tftp.x86_64 	 tftp-server.x86_64  xinetd.x86_64服务器(为客户机提供引导及驱动)

![img](E:\Project\Textbook\linux云计算\assets\wps118.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps119.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps120.jpg) 

##  3配置syslinux服务程序

![img](E:\Project\Textbook\linux云计算\assets\wps121.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps122.jpg) 

 ![img](E:\Project\Textbook\linux云计算\assets\wps123.jpg)

客户端启动流程，TFTP服务根目录下要放置的文件

yum install syslinux安装引导加载程序

1）客户端执行pxelinux.0（由syslinux生成）

2）initrd.img  initrd（用来临时的引导硬件到实际内核vmlinuz能够接管并继续引导的状态）

3）menu.c32（安装系统菜单）

4）boot.msg（启动信息，可以修改）

5）vmlinuz（可引导的、压缩的内核文件，vmlinuz是vmlinux的压缩文件是可执行的Linux内核）

[root@rhel pxelinux.cfg]# vim default	编辑开机时选项菜单

![img](E:\Project\Textbook\linux云计算\assets\wps124.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps125.jpg) 

##  4脚本制作工具yum install system-config-kickstart

将服务端yum仓库名改为development		服务重新安装remove， repolist重启

![img](E:\Project\Textbook\linux云计算\assets\wps126.jpg) ![img](E:\Project\Textbook\linux云计算\assets\wps127.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps128.jpg)

若要使用图形桌面环境，建议将GNOME相关的包勾选上

![img](E:\Project\Textbook\linux云计算\assets\wps129.jpg) ![img](E:\Project\Textbook\linux云计算\assets\wps130.jpg) ![img](E:\Project\Textbook\linux云计算\assets\wps131.jpg)

![img](E:\Project\Textbook\linux云计算\assets\wps132.jpg) ![img](E:\Project\Textbook\linux云计算\assets\wps133.jpg)

保存至ftp共享文件夹			Chmod 647 /var/ftp/ ks.cfg

\# Kickstart file automatically generated by anaconda.

\#version=DEVEL

install                      # 命令段  ，安装

url --url=http://192.168.4.150/centos/os/    # 指定网络url安装目录 

lang en_US.UTF-8                 # 默认字体

keyboard us                    # 键盘类型

network --onboot yes --device eth0 --bootproto dhcp  --noipv6  

\# 指定开机自启，网络接口eth0 ，dhcp获取网络地址，ipv6 禁用

 

rootpw  --iscrypted $6$ZOGP2tA0PI/6SI/X$MlC5bJyXfP9TBN5/0vwoc6dqAqIijOQthEbAZUnIXft85Tj9n4sKWB2PfxrsVfkZ2ibqX63apu8ElmdEvBo9o/  

\# root 加密密码，使用grub-crypt 生成的字符串替代

 

reboot     # 配置完毕后，重启内核

firewall --disabled   # 防火墙禁用

authconfig --enableshadow --passalgo=sha512  # 登录身份使用 sha1 的 512bits 加密算法

selinux --disabled   # selinux 功能禁用

timezone Asia/Shanghai # 定义上海时区 

bootloader --location=mbr --driveorder=sda --append="crashkernel=auto rhgb quiet"  

\# 定义bootloader，grub安装mbr ，安装在sda磁盘

 

\# The following is the partition information you requested

\# Note that any partitions you deleted are not expressed

\# here so unless you clear all partitions first， this is

\# not guaranteed to work

clearpart --all     # 清除磁盘分区表

text           # 纯文本格式安装显示

zerombr          # 对磁盘进行初始化

 

part /boot --fstype=ext4 --asprimary --size=2000 # 分区信息 ，定义boot分区 ，格式为ext4 ，大小为2G

part swap --size=4096     # 分区信息 ，定义swap分区 ，大小为4G

part pv.008003 --size=80000 # 分区信息 ，定义lv分区 pv.008003，大小为80G

 

volgroup vg0 --pesize=8192 pv.008003  # 分区信息 ，在lv分区pv.008003定义vg0卷组 ，pe大小为8M

logvol / --fstype=ext4 --name=root --vgname=vg0 --size=15000

logvol /usr --fstype=ext4 --name=/usr --vgname=vg0 --size=30000

logvol /var --fstype=ext4 --name=/var --vgname=vg0 --size=20000

logvol /home --fstype=ext4 --name=/home --vgname=vg0 --size=12000

 

repo --name="CentOS-6.6" --baseurl=http://192.168.4.150/centos/os/ --cost=100 

\# 定义yum仓库 ，类别为bashurl ，名称为CentOS-6.6

 

%packages       # 包组段，安装包组及程序包

@core         

@server-policy       

@workstation-policy

%end

##  5安装vsftpd服务(分享ISO镜像和ks.cfg脚本)			

分享镜像mount /dev/sr0	 /var/ftp/pub/

修改Vim /var/ftp/ks.cfg	内容	添加url --url=ftp://192.168.127.11/pub提供包的位置

设置firewalld防火墙和selinux		setsebool -P ftpd_full_access on

# firewalld防火墙

<img src="E:\Project\Textbook\linux云计算\assets\wps134.jpg" alt="img" style="zoom: 80%;" /> 

1.systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。

2.配置firewalld-cmd

查看帮助： firewall-cmd –h			查看版本： firewall-cmd -V		永久生效：--permanent

允许服务：firewall-cmd --add-service={http/nfs/rpc-bind/mountd/samba/dns/ntp}

添加开启端口：firewall-cmd --zone=public --add-port=80/tcp 

重新载入：firewall-cmd --reload		firewall-cmd –-list-all	列出firewall所有规则

查看所有打开的端口：firewall-cmd --zone=public --list-ports

删除端口	firewall-cmd --zone= public --remove-port=80/tcp

删除服务：firewall-cmd --zone= public --remove-service=http

开启防火墙伪装：firewall-cmd  --add-masquerade --zone=public  //开启后才能转发端口

增加一个新域test：firewall-cmd --new-zone=test

更改网卡的默认区域为public：firewall-cmd –zone=public --change-interface=eth0

查看指定接口所属区域：firewall-cmd --get-zone-of-interface=eth0

查看是否拒绝： firewall-cmd --query-panic

添加转发规则：firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.1.1

（此规则将本机80端口转发到192.168.1.1的8080端口上，配置完--reload才生效）

如果配置完以上规则后仍不生效，检查防火墙是否开启80端口，如果80端口已开启，仍无法转发，可能是由于内核参数文件未配置ip转发功能：net.ipv4.ip_forward = 1 >/etc/sysctl.conf

保存文件后，输入命令sysctl -p生效

 

添加富规则firewall-cmd --add-rich-rule “rule family=ipv4 source address=192.168.0.33 port port=3260 protocol=tcp accrpt(允许)/reject(拒绝)/drop(丢弃)”

![img](E:\Project\Textbook\linux云计算\assets\wps135.jpg) 

# Iptables(只是一条命令)

tables实不是真正的防火墙，我们可以把它理解成一个客户端代理，用户通过 iptables这个代理，将用户的安全设定执行到对应的安全框架中，netfilter是防火墙真正的安全框架( framework)， netfilter位于内核空tables其实是一个命令行位于用户我们用这个工具操作真正的框架。				`包：iptables`

iptables 默认维护着 4 个表和 5 个链，所有的防火墙策略规则都被分别写入这些表与链中。

“四表”是指 iptables 的功能，默认的 iptable s规则表有 filter 表（过滤规则表）、nat 表（地址转换规则表）、mangle（修改数据标记位规则表）、raw（跟踪数据表规则表）：

> filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。
> nat 表：控制数据包中地址转换，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING。
> mangle：修改数据包中的原数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING。
> raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT。

<img src="E:\Project\Textbook\linux云计算\assets\wps136.jpg" alt="img" style="zoom:67%;" /> <img src="E:\Project\Textbook\linux云计算\assets\wps137.jpg" alt="img" style="zoom:67%;" /> <img src="E:\Project\Textbook\linux云计算\assets\wps138.jpg" alt="img" style="zoom:67%;" />

iptables [-t 表名] <-A增加|I插入|D删除|R丢弃> 链名 [设置规则的执行编号] [-i|o 网卡名称] [-p 协议类型] [-s 源ip|源子网] [--sport 源端口号] [-d 目的IP|目标子网] [--dport 目标端口号] [-j 动作]

iptables 命令常用的选项及功能

```sh
-A在规则链的末尾加人新规则				-F清空规则链					-L查看规则链
-I +num，在规则链的头部加新规则			-D +num，删除某一条规则		  -R 替换防火墙规则
-Z	清空防火墙数据表统计信息			-P设置默认策略
```

iptables 命令常用匹配参数及功能

```sh
-p +匹配协议，如TCP、UDP、ICMP					[!]-p +匹配协议,！取反
-s匹配来源地址IP/MASK，					    -d匹配目标地址
-i +网卡名称，匹配从这块网卡流入的数据		      -o +网卡名称，匹配从这块网卡流出的数据
--dport +num，匹配目标端口号					--sport +num，匹配来源端口号
--src-range	匹配源地址范围						--dst-range	匹配目标地址范围
--limit	四配数据表速率							--mac-source 匹配源MAC地址
--stste	匹配状态（INVALID、ESTABLISHED、NEW、RELATED)
--string	匹配应用层字串
-n,--numeric 数字化显示ip和端口					--line-number查看规则行数
```

iptables 命令触发动作及功能

| 触发动作   | 功 能          | 触发动作 | 功 能                        |
| :--------- | :------------- | -------- | ---------------------------- |
| ACCEPT     | 允许数据包通过 | REJECT   | 拒绝数据包通过               |
| DROP       | 丢弃数据包     | LOG      | 将数据包信息规则 syslog 曰志 |
| DNAT       | 目标地址转换   | SNAT     | 源地址转换                   |
| MASQUERADE | 地址欺骗       | REDIRECT | 重定向                       |

**1、查看当前iptables状态**

iptables -nL #默认查看filter表的状态，如果需要查看其他表的状态加上 -t 表名

iptables -nL --line-numbers --verbose #可以查看到规则行数包过滤的流量统计，访问次数

**2、插入规则**

iptables -A INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j  ACCEPT

#第一行追加一条规则将 INPUT设置为只允许指定网段主机访问本机的22端口，拒绝来自其他主机的流量

iptables -I INPUT 1 -i lo -j ACCEPT#在第一条的位置插入一条规则，接受所有来自lo网口的访问

iptables -I INPUT 2 -s 192.168.1.0/24 -j ACCEPT#如果在INPUT中不指明在第几条插入，默认就是在第一条插入

**3、修改规则**

iptables -R INPUT 6 -s 194.168.1.5 -j ACCEPT #在第 6 行规则的 DROP 修改为 ACCEPT

**4、删除规则**

iptables -D INPUT 7 #删除第7条规则

iptables -P INPUT DROP #把 INPUT 规则链的默认策略设置为拒绝

iptables -I INPUT -p imcp -j ACCEPT #针对协议开放

**5、针对端口开放（需要指明协议）**

iptables -I INPUT -p tcp --dport 22 -j ACCEPT

**6、限制ip端口访问**

iptables -I INPUT -s 192.168.1.0/24 -p tcp --dport 22 -j ACCPET

**7、拒绝所有访问**

iptables -A INPUT -j DROP #这个一般放到最后，不然会对前面的规则造成影响。

**8、根据时段限制访问**

iptables -A INPUT -p tcp -m time --timestart 00:00 --timestop 02:00 -j DROP #这里的时间是指UTC时间记得换算

**9、限制单个IP一分钟内建立的连接数**

iptables -A INPUT -p tcp --syn --dport 80 -m connlimit --connlimit-above 25 -j REJECT

**10、从文件里面恢复iptables规则**

iptables-restore < /etc/sysconfig/iptables

**11、对外建立的连接经过INPUT不拦截**

iptables -I INPUT -m conntrack --ctstate RELATED，ESTABLISHED -j ACCEPT

**12、端口转发**（本机8080转发到远程192.168.1.22:80）

iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 8080 -j DNAT --to 192.168.1.22:80

iptables -t nat -A POSTROUTING -j MASQUERADE

```sh
echo 1 > /proc/sys/net/ipv4/ip_forward #需要打开ip转发
```

iptables -t filter FORWARD -d 192.168.1.22/32 -j ACCEPT #转发的FROWARD要允许双方的数据传输

**13、规则保存和备份**

service iptables save #让配置的防火墙策略永久生效

iptables-save > /etc/sysconfig/iptables #保存在默认文件夹中（保存防火墙规则）

iptables-restore < 文件名称 #批量导入Linux防火墙规则

**14、自动获取当前网卡ip地址来做SNAT源端口**

iptables -t nat -A POSTROUTING -j MASQUERADE

# 服务的访问控制列表(TCP Wrappers)

TCP Wrappers 是RHEL 7 系统中默认启用的一款流量监控程序，它能够根据来访主机的地址 与本机的目标服务程序作出允许或拒绝的操作。

Linux 系统中其实有两个层面的防火墙

第一种是前面讲到的基于 TCP/IP 协议的流量过滤工具

而 TCP Wrappers 服务则是能允许或 禁止 Linux 系统提供服务的防火墙

![img](E:\Project\Textbook\linux云计算\assets\wps139.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps140.jpg) 

# KVM虚拟技术

业务需求 举例：公司现有部分linux服务器利用率不高，为充分使用这些服务器，可以部署kvm，在物理机上部署多个业务系统。

例如在运行的nginx服务器上部署kvm，然后再虚拟机上运行tomcat

虚拟化就是把硬件资源从物理方式转变为逻辑方式，打破原有物理结构，使用户灵活管理这些资源，并且允许一台物理机上同时运行多个操作系统，以实现资源利用率最大化和灵活管理的技术

**优点**

1减少服务器数量，降低硬件采购成本		2资源利用率量大化		3降低机房空间，收热:用电消耗的成本

4硬件资源可动态调整，提高企业IT业多贝活性		5高可用性		6在不中断服务的情况下进行物理硬件调整

7降低管理成本							8具备更高效的灾备能力

KVM是开源软件，全称是kernel-based virtual machine（基于内核的虚拟机）

KVM 是基于虚拟化扩展（Intel VT 或者 AMD-V）硬件的开源的 Linux 原生的全虚拟化解决方案

![img](E:\Project\Textbook\linux云计算\assets\wps141.jpg)![img](E:\Project\Textbook\linux云计算\assets\wps142.jpg) 

modprobe kvm加载kvm模块

yum -y install libvirt-daemon-kvm qemu-kvm virt-manager libvirt启动libvirtd服务

查看系统上所有的模块

![img](E:\Project\Textbook\linux云计算\assets\wps143.jpg)

```sh
[root@Init ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
DEFROUTE= "yes"
NAME="ens33"
DEVICE="ens33"
ONBOOT="yes"
NM_CONTROLLED=no  //#是否受Network Mangement控制，这里选择no。
BRIDGE=br0
[root@Init ~]# vim /etc/sysconfig/network-scripts/ifcfg-brO
BOOTPROTO=dhcp
DEVICE=br0
TYPE=Bridge
NM_CONTROLLED=no
```

![img](E:\Project\Textbook\linux云计算\assets\wps146.jpg) 

查看是否支持虚拟化（vmx是intel专用）		brctl	show查看桥接信息

创建fdisk虚拟机所需硬盘（建议50G大小），格式化系统mkfs.xfs，进行挂载到/var/lib/libvirt/images

virt-manager打开可视化界面		Virsh虚拟化命令	--help			snapshot快照信息区

创建一个名为gurobi的1个8核cpu，内存容量为512M的，硬盘容量为1TB

```sh
[root@localhost ~]# virt-install --name=gurobi --memory=512，maxmemory=1024 --vcpus=1，maxvcpus=2 --os-type=linux --os-variant=rhel7 --location=/tmp/CentOS-7-x86_64-DVD-1708.iso --disk path=/kvm_data/gurobi.img，size=10 --bridge=br0 --graphics=none --console=pty，target_type=serial --extra-args=“console=tty0 console=ttyS0”
```

![img](E:\Project\Textbook\linux云计算\assets\wps147.jpg) 

![img](E:\Project\Textbook\linux云计算\assets\wps148.jpg) 

