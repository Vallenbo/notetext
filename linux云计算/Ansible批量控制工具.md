# Ansible批量控制工具

## 特点

1. 一款基于ssh远程通讯新型的自动化运维工具
2. 配置简单、功能强大、扩展性强，实现批量系统配置，程序部署，运行命令
3. 部署简单，只需在主控端部署Ansible环境，被控端无需做任何操作
4. 基于python开发继承多个运维工具，支持API及自定义模块，可通过Python轻松扩展
5. 通过Playbooks来定制强大的配置、状态管理
6. 轻量级，无需在客户端安装agent，更新时，只需在操作机上进行一次更新即可
7. 提供一个功能强大、操作性强的Web管理界面和REST API接口——AWX平台

## 架构图

Ansible：			Ansible核心程序

HostInventory：	记录由Ansible管理的主机信息，包括端口、密码、ip等

Playbooks：		"剧本”YAML格式文件，多个任务定义在一个文件中，定义主机需要调用哪些模块来完成的功能

CoreModules：	核心模块，主要操作是通过调用核心模块来完成管理任务

CustomModules：	自定义模块，完成核心模块无法完成的功能，支持多种语言

ConnectionPlugins：连接插件，Ansible和Host通信使用

## 任务执行模式及过程

**ad-hoc模式(点对点模式)**

　　使用单个模块，支持批量执行单条命令。ad-hoc 命令是一种可以快速输入的命令，而且不需要保存起来的命令。就相当于bash中的一句话shell

**playbook模式(剧本模式)**

　　是Ansible主要管理方式，也是Ansible功能强大的关键所在。playbook通过多个task集合完成一类功能，如Web服务的安装部署、数据库服务器的批量备份等。可以简单地把playbook理解为通过组合多条ad-hoc操作的配置文件

<img src="E:\Project\Textbook\linux云计算\assets\wps8-1682690463420-258.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps9-1682690463420-259.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps10-1682690463421-260.jpg" alt="img" style="zoom:67%;" /> 

主配置文件/etc/ansible/ansible.cfg，常用参数：

```
inventory = /etc/ansible/hosts#ansuble主机清单(内附配置格式)
library=/usr/share/ansible	#指向存放Ansible模块的目录
forks = 5							#并发连接数，默认为5
sudo_user = root			#设置默认执行命令的用户
remote_port = 22			#指定连接被管节点的管理端口，默认为22端口，建议修改，能够更加安全
host_key_checking = False	#检查SSH主机的密钥，值为True/False。False则第一次连接不会提示配置实例
timeout = 60				#设置SSH连接的超时时间，单位为秒
log_path=/var/log/ansible.log#ansible日志的文件（默认不记录日志）
```

## 常用命令及参数

ansible临时命令执行工具，常用于临时命令的执行

基本用法：ansible <host-pattern清单> [-m module模块] [-a args模块参数]

```sh
ansible *web -m command -a 'ls / ' -u root -k	#对所有配置清单内主机基于-k验证
-i “xx” #指定主机清单的路径，默认为/etc/ansible/hosts
-m 	#执行模块的名字，默认使用 command 模块可以不写-m command
-a	#模块参数

-B SECONDS	#后台运行超时时间
-C	#模拟运行环境并进行预运行，可以进行查错测试
-f FORKS 		#并行任务数，默认为5
-o	#压缩输出，尝试将所有结果在一行输出，一般针对收集工具使用

-c CONNECTION 	#连接类型使用
-R SU_USER 	#指定 su 的用户，默认为 root 用户
-U SUDO_USER	#指定 sudo 的用户，默认为 root 用户
-T TIMEOUT	#指定 ssh 默认超时时间，默认为10s，也可在配置文件中修改
--list	 #查看有哪些主机组
-v 	#查看详细信息，同时支持-vvv可查看更详细信息
-k	#ask for SSH password。登录密码，提示输入SSH密码而不是假设基于密钥的验证
--ask-su-pass 	#ask for su password。su切换密码
-K	#ask for sudo password。提示密码使用sudo，sudo表示提权操作
--ask-vault-pass #ask for vault password。假设我们设定了加密的密码，则用该选项进行访问

通配符：
all：表示所有Inventory中的所有主机
*：通配符ansible “*” -m ping
或关系ansible 'webserver:dbserver' -m ping #执行在web组并且在dbserver组中的主机
与关系ansible 'webserver:&dbserver' -m ping
非逻辑ansible 'webserver:!dbserver' -m ping #【注意此处只能使用单引号！】
```



```sh
ansible-playbook	定制自动化的任务集编排工具
		ansible-playbook <filename.yml>  ... [options]
					-C #预执行yaml检测		--syntax-check #检查语法		-v --v显示过程（-vvv更详细 ）
					--list-hosts /tags 列出运行任务的主机 / 标签 		--limit 列出执行的主机
```

```sh
ansible-doc # 模块功能查看工具
		-l	# 全部模块的信息		-s ping	#以playbook简写片段显示指定模块的使用帮助
ansible-galaxy # 下载/上传优秀代码或Roles模块 的官网平台
			ansible-galaxy install geerlingguy.nginx
					list列出		install安装	remove删除
ansible-pull	# 远程执行命令的工具，拉取配置而非推送配置（使用较少，海量机器时使用，对运维的架构能力要求较高）
ansible-vault	# 文件加密工具
		ansible-vault encrypt hello.yml
				encrypt加密		decrypt解密		view查看		edit编辑		create新建
ansible-console # 基于Linux Consoble界面可与用户交互的命令执行工具
		root@test(2)[f:10] $  执行用户@当前操作的主机组（当前组的主机数量）[f:并发数]$	使用：模块+命令
		设置并发数：fock n 例如：fock 10       切换组：cd 主机组 例如：cd webser / 192.168.0.106
		列出当前组的主机列表：list       	 			列出所有内置命令：？或help
```



## 常用模块

```sh
command模块		# 默认模块可以直接在远程主机上执行命令，并将结果返回本主机
		# 它不会通过shell进行处理，比如$HOME和操作如"<"，">"，"|"，";"，"&" 工作也不支持管道符号 |
shell模块 		# 可以在远程主机上调用shell解释器运行命令，支持shell的各种功能，例如管道等
copy 模块			# 用于将文件复制到远程主机，同时支持给定内容生成文件和修改权限等
			ansible all -m copy -a 'src=/s.sh dest=/ backup=yes mode=0664'
					src		# 本地文件。绝对路径，也可以相对路径。路径是一个目录，则递归复制
					dest	# 远程主机的绝对路径，不存在则会创建
					owner/group	# 指出复制时，目标文件的所有者/所属组
					backup=yes|no	# 默认no，当文件发生改变后，在覆盖之前把源文件先进行备份
					mode	# 递归设定目录的权限，默认为系统默认权限
					content	# 用于替换"src"，可以直接指定文件的值
					force=yes|no	#目标主机包含该文件，但内容不同时，yes（默认）强制覆盖，no被控端位置不存在该文件复制

fetch模块		#从被控端获取（复制）文件到本地，拷贝多个文件可使用tar打包
			ansible web -m fetch -a 'src=/data/hello dest=/data' 
					src #在远程拉取的文件，并且必须是单个file，且不是目录		dest：用来存放文件的目录

hostname模块	#修改n被控端主机名		ansible all -m hostname -a 'name=web1'
script模块	#将本机绝对路径的脚本在被控端上运行	ansible all -m script -a '/s.sh'

file模块		#用于设置文件的属性，比如创建文件、创建链接文件、删除文件等
				state模块		#状态，有以下选项：		ansible all -m file -a 'name=/sls state=touch'
				directory：	#如果目录不存在，就递归创建目录
				touch #如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
     		absent #删除目录、文件或取消链接文件
				link/hard #创建软链接 / 硬链接
				force	#需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目>标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
				mode #定义文件/目录的权限
				path #定义文件/目录的路径
				owner/group	#定义文件/目录的属主/属组。后面必须跟上	
				recurse	#递归设置文件的属性，只对目录有效，后面跟上src：被链接的源文件路径，只应用于state=link的情况
				dest		#被链接到的路径，只应用于state=link的情况

cron 模块			#用于管理cron计划任务，使用语法跟crontab语法一致
		ansible all -m cron -a  'minute=*  weekday=1,3,5  job="/usr/bin/echo  $(date)" name=sss'
				day= #日应该运行的工作( 1-31, *, */2, )
				hour= # 小时 ( 0-23, *, */2, )
				minute= #分钟( 0-59, *, */2, )
				month= # 月( 1-12, *, /2, )
				weekday= # 周 ( 0-6 for Sunday-Saturday,, )
				job= # 指明运行的命令是什么
				name= # 定时任务描述
				reboot # 任务在重启时运行，不建议使用，建议使用special_time
				special_time # 特殊的时间范围，参数：reboot（重启时），annually（每年），monthly（每月），weekly（每周），daily（每天），hourly（每小时）
				state # 指定状态，present表示添加定时任务，也是默认设置，absent表示删除定时任务
				user # 以哪个用户的身份执行

yum模块	#用于软件的安装		
		ansible all -m yum -a 'name=tree,vsftpd  diable_gpg_check = yes'
				#安装rpm包，先推送到被控端，指定路径	ansible all -m yum -a 'name=/pptpd.rpm'
				name=				#所安装的包的名称
				state=			#执行操作present-->安装（默认选项），latest-->安装最新的，absent--> 卸载软件
				update_cache	#强制更新yum的缓存
				conf_file		#指定远程yum安装时所依赖的配置文件（安装本地已有的包）
				diable_gpg_check = yes | no : 是否启用包完整性校验功能gpgcheck
				disablerepo	#临时禁止使用yum库 只用于安装或更新时
				enablerepo	#临时使用的yum库只用于安装或更新时

service模块   #用于服务程序的管理		ansible all -m service -a 'name=vsftpd state=started'
				state			#执行状态，started启动服务， stopped停止服务， restarted重启服务， reloaded重载配置
				enabled		#设置开机启动
				name=			#服务名称
				runlevel		#开机启动的级别，一般不用指定
				sleep		#在重启服务的过程中，是否等待如在服务关闭以后等待2秒再启动(定义在剧本中)
				arguments	#命令行提供额外的参数

user模块    #用来管理用户账号
		ansible all -m user -a 'name=user1  shell=/sbin/nologin system=yes home=/home group=root'
				name		# 指定用户名
				shell		# 指定默认shell
				home		# 指定用户家目录
				system=yes|no # 是否为系统账号，这个设置不能更改现有用户
				createhome		# 是否创建家目录
				state=present|absent # 创建账号或者删除账号，present 创建（默认），absent删除
				remove=yesS|no #当state=absent 时，是否删除用户的家目录
				force		# 在使用state=absent时, 行为与userdel –force一致.
				group		# 指定基本组
				groups	# 指定附加组，如果指定为(groups=)表示删除所有组
				move_home=yes | no :如果设置的家目录以及存在，是否将已经存在的家目录进行移动
				non_unique	# 该选项允许改变非唯一的用户ID值
				password		# 指定用户密码
				uid=1200		# 指定用户的uid
				comment		# 用户的描述信息

group模块	#用于添加或删除组	ansible all -m group -a 'name=sss system=yes gid=1300'
				gid=		#设置组的GID号
				name=	#指定组的名称
				state=	#指定组的状态，默认为创建，设置值为absent为删除
				system=	#设置值为yes，表示创建为系统组

setup模块	#用于收集被控端系统信息，是通过调用facts组件来实现的，facts组件是Ansible用于采集被管机器设备信息
		ansible web -m setup -a 'filter="vcpus"' --tree /tmp/facts筛选的信息发送至主控端
```



## Ansible-playbook剧本

playbook 是 ansible 用于配置，部署，和管理被控节点的剧本

```yaml
---				#ansible-playbook格式：-后多空格		注意平级关系
- hosts: web		#主机清单		支持ansible *web通配符系列，支持逻辑运算符：，&，！
 remote_user: root	# 被控端执行用户
 vars: 
   -  pkname: httpd # 可对多个变量进行赋值
 vars_files: 
   - vars.yml		#指定变量存放文件
 tasks:			#任务集合
   - name: install httpd package	#任务名称，一个任务对应一个模块
​     yum：name=httpd			#使用模块,执行命令
	 - name: install package
 yum：name={{ pkname }}	 #定义变量{{ pkname }}	（变量命名规则与c语言类似）
​    - name: 'shutdown redhad flavored systems'
​     cp: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
​     when: ansible_os_family == "Centos"	#when条件语句，当符合centos的条件时执才行该任务模块
​    - name: unstall web packages
​     yum: name={{ item }} state=absent	#固定变量名“item”
​     with_items:						#with_items	循环：迭代元素列表，需要重复执行的任务
​     - httpd		#迭代元素
​     - php
​    - name: add some users				#迭代嵌套子变量
​     user: name={{ item.name }} group={{ item.group }} state=present	#引入迭代列表的元素
​     with_items:
​       - { name: 'u1', group: 'g1' }		#迭代字典
​       - { name: 'u2', group: 'g2' }
 notify: service	# 通知器：可通知多个触发器handlers
 tags: conf				# 一个标签可对应多个任务模块，使用方法如下红色区
 sudo_user: wang	# sudo为wang		#sudo: yes#默认sudo为root
 # 如果命令或脚本的退出码不为零，可以使用如下方式替代
 shell : /usr/bin/yum || /bin/true
 ignore_errors: True	#使用ignore_errors来忽略错误信息
 handlers: 			#触发器：当脚本运行前面的notify则会执行该触发器，可设置多个任务模块
- name: service
  service: name=httpd state=restarted
```

```sh
ansible-playbook -C sss.yml	#预加载脚本，检测
ansible-playbook -t conf httpd.yml  #只运行脚本里所有带有conf标签的任务模块【使用-t 指定标签名字】
ansible-playbook -e 'pkname=tree pkname1=epel' sss.yml	#对sss.yml里面多个kpname变量临时进行赋值
```

变量的优先级：命令行中的-e > playbook中定义的变量 > /etc/ansible/hosts[webservs]全局变量>vars共有变量

<img src="E:\Project\Textbook\linux云计算\assets\wps11-1682690463421-261.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps12-1682690463421-262.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps13-1682690463421-263.jpg" alt="img" style="zoom:67%;" /> 

templates模板用来存放各种服务配置文件的模板，是一个文本文件，嵌套有脚本（使用模板编程语言编写），用jinja2语言，以 j2 结尾，有如上形式:

Jinja2：Jinja2是python的一种模板语言，以Django的模板语言为原本

存储template模块调用的模板文件夹/etc/ansible/roles/templates/

playbook中template模板对于for  if 循环的使用

```playbook
---
- hosts: web	#主机清单		支持ansible *web通配符系列，支持逻辑运算符：，&，！
 remote_user: root	#被控端执行用户
 vars:
  ports:	#定义变量列表，用于实现for的template模板所需
   - web1: 	
​    port: 81
​    name: web1.magedu.com
​    rootdir: /data/website1
   - web2: 	
​    port: 82
​    #name: web2.magedu.com
​    rootdir: /data/website2
 tasks:
  - name: copy conf
   template: src=if.conf.j2 dest=/data/if.conf
vim templates/for1.conf.j2 模板文件：
{% for p in ports %}			#for循环语句，p自定义变量，ports变量列表	#与shell脚本非常类似
server{
​    listen {{ p.port }}		#从p变量内拿取元素port
{% if p.name is defined %}	#如果p.name被定义就执行servername
​    servername {{ p.name }}
{% endif %}				#否则执行
​    documentroot {{ p.rootdir }}
}
{% endfor %}
```



## 角色订制：roles剧本文件

实际上分解playbook而后结构化地组织，比较灵活。roles 能够根据层次型结构自动装载变量文件、tasks以及handlers等

roles剧本分别将变量(vars)、文件(file)、任务(tasks)、模块(modules)及处理器(handlers)放置于单独的目录中，并可以通过include便捷地使用它们的一种机制

<img src="E:\Project\Textbook\linux云计算\assets\wps14-1682690463421-264.jpg" alt="img" style="zoom: 67%;" /> 

角色集合：/etc/ansible/roles/

mysql/，httpd/，nginx/

files/：存放由copy模块或scripts模块等调用的文件 

template/：template模块 查找所需要模板文件的目录

tasks/：定义tasks，roles的基本元素，至少应该包含一个名为main.yml的文件；其他的文件需要>在此文件中通过include进行调用

handlers/：至少应该包含一个名为main.yml的文件；其他的文件需要在此文件中通过include进行调用

vars/：定义变量，至少应该包含一个名为main.yml的文件；其他的文件需要在此文件中通过include进行调用

meta/：定义当前角色的特殊设定及其依赖关系，至少应该包含一个名为main.yml的文件；其他的文件需要在此文件中通过include进行调用

default/：设定默认变量时使用此目录中的main.yml文件

<img src="E:\Project\Textbook\linux云计算\assets\wps15-1682690463421-265.jpg" alt="img" style="zoom:50%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps16-1682690463421-266.jpg" alt="img" style="zoom:67%;" /> 

<img src="E:\Project\Textbook\linux云计算\assets\wps17-1682690463421-267.jpg" alt="img" style="zoom:67%;" /> 

