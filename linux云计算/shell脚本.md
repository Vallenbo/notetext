# Shell编程

shell是一个命令解释器，它在操作系统的最外层，负责直接与用户对话，把用户的输入解释给操作系统，并处理各种各样的操作系统的输出结果，输出屏幕返回给用户，/etc/shells可查看当前所有的shell类型

shell脚本用途：

​	1、有自动化运维常用命令，2、创建简单的应用程序，3、处理文本或文件，4、用于系统管理和故障排除

shel脚本：包含一些命令或声明有一定格式的文本.sh文件

格式要求首行 shebang机制

\#bin/bash	#!/usr/bin/python		#!/usr/bin/perl

检查脚本方式：sh -n不执行脚本，仅检查脚本中的语法问题		-v将执行过的脚本命令打到屏幕上

运行脚本方式：

- 1、bash / sh+脚本

- 2、将脚本放入PATH命令对用的环境（echo $PATH查看）目录中

- 3、chmod+x执行脚本test.sh

- 4、./test.sh执行

  

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

**参数变量**：

​	$0脚本名

​	$1第一个参数

​	$2，$3，${10}...表示执行脚本后传递给脚本文件的第n个参数,花括号{}将一整体概括

**特殊变量**：

$?	执行脚本返回的值，0或非0值			$#   输入的参数个数		$$	查看当前进程PID号

$@  所有参数		$*  所有的参数，全部参数为一个字符串		$RANDOM	产生随机数（0-32767）			 

${#变量名} 表示变量的长度				 ${#变量名[@]} 表示数组的个数

## 特殊字符：

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

判断中的与或非：&	|	！

$(())或$[]可以进算数运算	``不可以

## 判断：

<= 小于等于(需要双括号),如:(("$a" <= "$b"))  		>= 大于等于(需要双括号),如:(("$a" >= "$b"))

大于 -gt (greater than)					小于 -lt (less than)			大于或等于 -ge (greater than or equal)

小于或等于 -le (less than or equal)		不相等 -ne （not equal）		相等 -eq （equal）

## shell脚本内的特殊命令：

echo $-：查看当前set环境功能		$-代表的是当前Bash的运行选项，这些Bash选项控制着Bash运行时的行为

h，hash进行查看：缓存使用命令的路径及使用次数

i，interactive-comments：交互式的shell环境

m，monitor：打开监控模式，可以通过job control来控制进程的停止、继续、后台或者前台执行

B，braceexpand：{}开启大括号拓展

H，history：开启历史命令

### set中的内置参数

set：显示所有变量及函数，用来定制shell环境

set +h删除hash功能

set --清空所有

set -o：查看当前set命令生效的设置选项

-o pipefail 只要一个子命令失败，整个管道命令就失败，脚本就会终止执行

`set -v`或者 `set -o verbose` 启用详细模式，将所有执行过的脚本命令打印到标准输出

-u：引用一个没有设置的变量时，显示错误信息，等同set -o nounset

-e：如果sh脚本中命令返回值非0退出状态值（失败）就退出，等同set -o errexit

-x 和 +x：显示脚本执行过程并将脚本内的变量的值暴露出来的一个开关，-x 是开，+x是关





## 常用命令

sleep命令：休眠时间，默认为秒1

shift命令：是shell脚本中使位置参数左移的命令，后可加左移位数，默认为1

break语句：（跳出循环）

let 命令是 BASH 中数字运算工具，用于执行一个或多个表达式（let 进行赋值，数字或字符\字符串）

help let查看系统支持的算数运算

test命令是 Shell 内置命令，用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试	hele test查看帮助

expr命令进行算术运算，格式比较严格		a=`expr 1 \* 4`		a=`expr $a \* 4`

exit 1函数退出当前程序

三目运算符：`a=$([ "$b" == 5 ] && echo "$c" || echo "$d")`

<img src="E:\Project\Textbook\linux云计算\assets\wps27-1682690115365-199.jpg" alt="img" style="zoom:67%;" /><img src="E:\Project\Textbook\linux云计算\assets\wps28-1682690115365-200.jpg" alt="img" style="zoom:67%;" /> 

在for、while、until等循环语句中，用于跳出当前所在的循环体，执行循环体后的语句

continue语句：（跳出本次循环）

在for、while、until等循环语句中，用于跳出循环体内余下的语句，重新判断条件以便执行下一次循环

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