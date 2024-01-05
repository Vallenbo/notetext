# 一、计算机组成

<img src="E:\Project\Textbook\Python\assets\wps1.jpg" alt="img" style="zoom:50%;" /> 

控制器：计算机的指挥系统，负责控制计算机所有的其他组件如何运行		控制器-->大脑

运算器：逻辑运算和数学运算		运算器-->大脑

控制器+运算器=大脑

存储器 / IO设备：计算机的记忆功能，负责数据存储

输入设备input：如键盘、鼠标

输出设备output：如显示器、打印机

平台=计算机物理硬件+操作系统

软件跨平台性：一款软件是否可以在任意平台上运行，是衡量软件质量的一个重要指标

# 二、操作系统

操作系统是一个协调、管理、控制计算机硬件资源与应用软件资源的一个控制程序

应用程序（暴风影音）：

控制程序（系统接口）：控制计算机硬件的基本运行（把控制硬件的复杂操作封装成简单指令），供上层软件程序使用

# 三、计算机系统三层应用结构

应用程序		--	操作系统		--	计算机硬件

<img src="E:\Project\Textbook\Python\assets\wps2.jpg" alt="img" style="zoom:50%;" /> 

cpu分类与指令集的概念：

简单指令集：即使用一些简单的指令，特点快捷高效

复杂指令集：通过多个指令完成，集成性高能单个完成任务

# 变量注释关键字

\#或 Crtl+/：注释，进行解释说明，放哪里都行

多行注释："""						'''

这是多行注释			这也是多行注释

第二行注释			第二行注释

"""					'''

变量：就是存储数据的时候，当前数据所在的内存地址的名字

定义变量：1、不能由数字开头、不能使用内置关键字	2、由数字、字母、下划线组成	3、严格区分大小写

<img src="E:\Project\Textbook\Python\assets\wps3.jpg" alt="img" style="zoom: 50%;" /> 

nonlocal关键字使外函数中的内函数能使用局部变量

变量名 = 值	例：my_name = 'TOM'	#定义字符

命名习惯：

大驼峰：首字母大写法，例：MyName

小驼峰：第二个及以后的单词，首字母大写，例：myName

下划线：例：my_name

# 数据类型

<img src="E:\Project\Textbook\Python\assets\wps4.jpg" alt="img" style="zoom:33%;" /> 

num1 = 1	整数		a = "hello world" 字符型		b = True	布尔类型

c = [10, 20, 30]列表		d = (10, 20, 30)元组		f = {10, 20, 30}集合（能含纳列表、元组）

f = {'name': 'TOM', 'age': '18'}字典，name和age互为键值对

# 转义字符

\n：换行		\t：制表符，一个tab键（4个空格）的距离

print('hello world')	# print函数默认自带\n输出，相当于print('hello world', end="\n")

print('hello world', end="\t") # 当然默认输出符号也能修改为\t

print('hello world', end="...")		print('hello world', end="你好")

print('\t hello world')

# 格式化输出数据

<img src="E:\Project\Textbook\Python\assets\wps5.jpg" alt="img" style="zoom:50%;" /> 

\#%d即有+正或-负符号的整数

# 运算符的分类

算数运算符（从左至右优先级依次加大）：+ - 	* /		//整除	%取余	**指数	()小括号

赋值运算符：=将右侧的结果赋值给等号左侧

同时多个变量赋值：a = b = 10	或者		num1, f1, c3 = 10, 0.5, 'hello world'

复合赋值运算符：

<img src="E:\Project\Textbook\Python\assets\wps6.jpg" alt="img" style="zoom:50%;" /> 

判断运算符：

<img src="E:\Project\Textbook\Python\assets\wps7.jpg" alt="img" style="zoom: 50%;" /> 

逻辑运算符：print(a > b and b < c)

<img src="E:\Project\Textbook\Python\assets\wps8.jpg" alt="img" style="zoom:50%;" /> 

拓展：数字之间的逻辑运算

and：取最小值为返回值			or：取最大值为返回值		x > y <z		x and b or c

# 流程控制语句

continue关键字：退出当前循环，进入下一次循环

break关键字：终止此循环

return关键字：并不是专门用于跳出循环的，return的功能是结束一整个方法，不管这个return处于多少层循环之内

## if语法（分段匹配）

基础版：

if 变量 判断式 数据：

print('为ture时输出的内容')

进阶版：

if 变量 判断式 input('请输入内容：')：

print('为ture时输出的内容')

多条件版 ：

if 变量 判断式 input('请输入内容：')：

print('为ture时输出的内容')

else：

print('为false时输出的内容')

多重判断：

age = input('请输入内容：')

if age变量(age) 判断式 数据：

print('符合该条件时输出的内容')

elif 变量(age) 判断式 数据：

print('符合该条件时输出的内容')

else：

print('为false时输出的内容')



\#导入模块			import random(随机模块)

\#使用模块生成随机数	random.randint(0, 2)	#int表示整型

## 三目运算符

a = 1 	b = 2	c = a if a > b else b		#if语句成立取a，否则取b值

## while语法（无限循环）

基础格式 ：	while 变量 判断式 数据：

print('符合条件时输出的内容')		#当输出该行内容时，继续回到while进行判断，直到不符合条件

循环判断式：	while 变量 判断式 数据：

print('符合条件时输出的内容')

break					#break终止循环的情况else下的代码将不会被执行

continue					#continue退出当前循环且按正常循环走流程结束

 else：						#else循环：正常的循环及走流程结束

print('符合条件时输出的内容')

嵌套格式 ：	while 变量 判断式 数据：

print('符合条件时输出的内容')	

while 变量 判断式 数据：

print('符合条件时输出的内容')	

## for语法

基本格式：for 临时变量 in 序列：				#当然我们也可以使用break和continue语法

循环判断正常格式：for 临时变量 in 序列：

print('符合条件时，执行的代码')

break					#break终止循环的情况else下的代码将不会被执行

continue					#continue退出当前循环且按正常循环走流程结束

else：

print('循环正常结束之后要执行的代码')

# 容器数据类型使用操作

str和list都可以通过下标的方式取出数据、list还能通过列表循环，即for和while取出

字典，能按照指定key查找数据、能返回所有key、能返回所有数据、能将字典数据返回成元组数据

下标：通过下标快速找到对应的数据（用于标识字符串中某一子串（字符））

语法：str5 = "love"		print(str5[0])	#下标标识，从0开始计识

切片：指对操作的对象截取其中一部分操作。字符串、列表、元组都支持切片操作

语法：序列[开始位置下标：结束位置下标：步长]

\#切片输出的数据，包含开始位置下标对应的数据、包含步长位置对应的数据、不包含结束位置下标对应的数据

\#我们把切片看成是个在一条带有正负方向的轴

\#正数下标默认为0，步数默认为1

\#负数下标默认为-1，负步数（表示从右开始）

str6 = '0123456789'						print(str6[-3:-1:1])	输出内容：78

str[2:6]='a' 对指定下标数据进行更新		del str[2:6]对指定下标数据进行删除

## str字符串

\#单双引号和三引号的区别：‘Hello’		“I’m TOM”		'''Tom'''	“”“Rose”“”

1、单引号不能进行分行内容输出，单引号不能用于带有引号‘的表示式(如上)字符串(也可用\转义字符解决)

2、三引号可以进行分行内容输出，

常用的内置方法

查找函数：

find(str, beg=0, end=len(string))检测 str 是否包含在字符串中，如果指定范围 beg 和 end ，则检查是否包含在指定范围内，如果包含返回开始的索引值，否则返回-1

rfind(str, beg=0,end=len(string))类似于 find()函数，不过是从右边开始查找

index(str, beg=0, end=len(string))跟find()方法一样，只不过如果str不在字符串中会报一个异常

rindex( str, beg=0, end=len(string))类似于 index()，不过是从右边开始

count(str, beg= 0,end=len(string))返回 str 在 string 里面出现的次数，如果 beg 或者 end 指定则返回指定范围内 str 出现的次数

 

split(str="", num=string.count(str))以 str 为分隔符截取字符串，返回一个列表容器

如果 num 有指定值，则仅截取 num+1 个子字符串

join(seq)以指定字符串作为分隔符，将 seq 中所有的元素(字符串表示)合并为一个新的字符串

replace(old, new [, max])把 将字符串中的 old 替换成 new,如果 max 指定，则替换不超过 max 次

strip([chars])在字符串上执行 lstrip()和 rstrip()

rstrip()删除字符串末尾的空格或指定字符、lstrip()截掉字符串左边的空格或指定字符

title()返回"标题化"的字符串,就是说所有单词都是以大写开始，其余字母均为小写(见 istitle())

lower()转换字符串中所有大写字符为小写

upper()转换字符串中的小写字母为大写

 

1、capitalize()将字符串的第一个字符转换为大写

2、center(width, fillchar)返回一个指定的宽度 width 居中的字符串，fillchar 为填充的字符，默认为空格

3、bytes.decode(encoding="utf-8", errors="strict")Python3 中没有 decode 方法，但我们可以使用 bytes 对象的 decode() 方法来解码给定的 bytes 对象，这个 bytes 对象可以由 str.encode() 来编码返回

5、encode(encoding='UTF-8',errors='strict')以 encoding 指定的编码格式编码字符串，如果出错默认报一个ValueError 的异常，除非 errors 指定的是'ignore'或者'replace'

6、endswith(suffix, beg=0, end=len(string))检查字符串是否以 obj 结束，如果beg 或者 end 指定则检查指定的范围内是否以 obj 结束，如果是，返回 True,否则返回 False

7、expandtabs(tabsize=8)把字符串 string 中的 tab 符号转为空格，tab 符号默认的空格数是 8

8、isalnum()如果字符串至少有一个字符并且所有字符都是字母或数字则返 回 True，否则返回 False

9、isalpha()如果字符串至少有一个字符并且所有字符都是字母或中文字则返回 True, 否则返回 False

10、isdigit()如果字符串只包含数字则返回 True 否则返回 False

11、islower()如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False

12、isnumeric()如果字符串中只包含数字字符，则返回 True，否则返回 False

13、isspace()如果字符串中只包含空白，则返回 True，否则返回 False.

14、istitle()如果字符串是标题化的(见 title())则返回 True，否则返回 False

15、isupper()如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是大写，则返回 True，否则返回 False

16、len(string)返回字符串长度

17、ljust(width[, fillchar])返回一个原字符串左对齐,并使用 fillchar 填充至长度 width 的新字符串，fillchar 默认为空格

18、maketrans()创建字符映射的转换表，对于接受两个参数的最简单的调用方式，第一个参数是字符串，表示需要转换的字符，第二个参数也是字符串表示转换的目标

19、max(str)返回字符串 str 中最大的字母

20、min(str)返回字符串 str 中最小的字母

22、rjust(width,[, fillchar])返回一个原字符串右对齐,并使用fillchar(默认空格）填充至长度 width 的新字符串

24、splitlines([keepends])按照行('\r', '\r\n', \n')分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符

25、startswith(substr, beg=0,end=len(string))检查字符串是否是以指定子字符串 substr 开头，是则返回 True，否则返回 False。如果beg 和 end 指定值，则在指定范围内检查

26、ord()将字符串转换为ASCII码

27、swapcase()将字符串中大写转换为小写，小写转换为大写

28、translate(table, deletechars="")根据 str 给出的表(包含 256 个字符)转换 string 的字符, 要过滤掉的字符放到 deletechars 参数中

29、zfill (width)、返回长度为 width 的字符串，原字符串右对齐，前面填充0

30、isdecimal()检查字符串是否只包含十进制字符，如果是返回 true，否则返回 false

## list列表（序列）

格式定义：[]使用一对方括号表示空列表、[a],[a, b, c]用逗号分隔项目

[x for x in iterable]使用列表推导器、list()或list(iterable)使用类型构造器

列表的常用操作：

<img src="E:\Project\Textbook\Python\assets\wps9.jpg" alt="img" style="zoom:50%;" /> 

通过下标进行的操作：切片、查找、更新、删除del list[2]、（不能添加）

常用函数:1、cmp(list1, list2)比较两个列表的元素			2、len(list)列表元素个数

3、max(list)返回列表元素最大值						4、min(list)返回列表元素最小值

5、list(seq)将元组转换为列表

常用内置方法：1、list.append(obj)在列表末尾添加新的对象

2、list.count(obj)统计obj元素在列表中出现的次数

3、list.extend(iterable)在列表末尾一次性追加可迭代对象容器内的元素

4、list.index(obj)从列表中找出某个值第一个匹配项的索引位置

5、list.insert(index, obj)将对象插入列表

6、list.pop([index=-1])移除列表中指定下标的一个元素（默认最后一个元素），并且返回该元素的值

7、list.remove(obj)移除列表中的第一个obj元素匹配项

8、list.reverse()反向列表中元素

9、list.sort(cmp=None, key=None, reverse=False)对原列表进行排序

10、list.copy()对于多维列表的浅拷贝(拷贝了原始元素的引用（内存地址）)

copy.deepcopy()深拷贝(拷贝了原始元素的引用（内存地址）)

## tuple元组

元组内的数据是不能修改的（即只支持查找）

定义：空元组()、变量=tuple()、单个元素时必须加逗号 (10，）或1，2，3、多个数据(10， 20， 30)	

特殊情况：如元组里面套列表的话，列表里的数据能修改		tuple2 = ('aa', 'bb', ['cc', 'bb'])

元组常用操作方法：能实现一般队列操作

按下标查找数据			print(tuple1[1])

index()函数：查找数据					格式：元组.index('数据')

count()函数：统计某个数据出现的次数		格式：元组.count('数据')

len()函数：统计元组中数据的个数			格式：len(元组)

yield关键字：返回结果并记住当前返回代码位置，下次调用以上次位置开始

## dict字典

定义：1、字典不支持下标，以键值对形式出现	2、符号为大括号		3、各个键值对之间用逗号隔开

创建格式：键值对		字典序列={key：数据}

使用花括号内以逗号分隔 键: 值 对的方式: {'jack': 4098, 'sjoerd': 4127} or {4098: 'jack', 4127: 'sjoerd'}

使用字典推导式: {}, {x: x ** 2 for x in range(10)}

使用类型构造器: dict(), dict([('foo', 100), ('bar', 200)]), dict(foo=100, bar=200)

常用操作：

d[key]返回 d 中以 key 为键的项 如映射中不存在 key 则会引发 KeyError	d[key] = value #将 d[key] 设为 value

del d[key]：将 d[key] 从 d 中移除 如果映射中不存在 key 则会引发 KeyError

key in d：只能检测 d 中存在键 key 则返回 True，否则返回 False

常用内置方法：

list(d)返回字典 d 中使用的所有键的列表			len(d)返回字典 d 中的项数

keys()返回由字典键组成的一个新视图			values()返回由字典值组成的一个新视图

items()获取字典项 ((键, 值) 对) 组成的一个新视图

iter(d)返回字典中键为元素的迭代器 这是 iter(d.keys()) 的快捷方式

get(key[, default])如果 key 存在于字典中则返回 key 的值， 如果 default 未给出则默认为 None，

pop(key[, default])如果 key 存在于字典中则将其移除并返回其值，如没有返回为default

popitem()从字典中移除并返回一个 (键, 值) 对 键值对会按 LIFO后进先出顺序被返回

reversed(d)返回一个逆序获取字典键的迭代器 这是 reversed(d.keys()) 的快捷方式

update([other])使用来自 other 的键/值对更新字典，覆盖原有的键 返回 None

update()接受另一个字典对象，或者一个包含键/值对（以长度为二的元组或其他可迭代对象表示）的可迭代对象 如果给出了关键字参数，则会以其所指定的键/值对更新字典: d.update(red=1, blue=2)

setdefault(key[, default])如果字典存在键 key ，返回它的值如果不存在，插入值为 default 的键 key ，并返回 default  default 默认为 None

clear()移除字典中的所有元素					copy()返回原字典的浅拷贝

## set集合

定义：集合是由不重复元素组成的无序容器		set()创建空集合，{}是用来创建字典的		格式：s2 = set()

False在集合中表示为0，所以0和false只能存在一个

frozenset()冰冻集合函数：定义不能修改，可以将任何容器类型数据进行转化

支持推导式frozenset({i*2 for i in range(6)})

常用内置方与集合运算：

add(elem)将元素 elem 添加到集合中					update(*others)更新集合，添加来自 others 中的所有元素

len(s)返回集合 s 中的元素数量（即 s 的基数）

x not in s：检测 x 是否非 s 中的成员

discard(elem)如果元素 elem 存在于集合中则将其移除

remove(elem)从集合中移除元素 elem，如果 elem 不存在于集合中则会引发 KeyError

pop()从集合中移除并返回任意一个元素，如果集合为空则会引发 KeyError

clear()从集合中移除所有元素							copy()返回原集合的浅拷贝

 

交集符：&		intersection_update(*others)以交集方式更新引用方法的集合

并集：|			union(*others)以并集方式返回一个新集合

差集：-			difference(*others)以差集方式返回一个新集合

difference_update(*others)以差集方式更新引用方法的集合

对称差集：^		symmetric_difference(other)以对称差集方式返回一个新集合

symmetric_difference_update(other)以对称差集方式更新引用方法的集合

<img src="E:\Project\Textbook\Python\assets\wps10.jpg" alt="img" style="zoom:50%;" /> 

issuperset(other)检测引用方法集合是否为超集，是则返回Ture

issubset(other)检测引用方法集合是否为other集合的子集，是则返回Ture

isdisjoint(other)检测引用方法集合和other集合是否为交集，不相交则返回Ture

intersection(*others)返回一个新集合，其中包含原集合以及 others 指定的所有集合中共有的元素

## 推导式（简化代码）

range(start, stop[, step])函数参数说明：

l start: 计数从 start 开始。默认是从 0 开始。例如range（5）等价于range（0， 5）;

l stop: 计数到 stop 结束，但不包括 stop。例如：range（0， 5） 是[0, 1, 2, 3, 4]没有5

l step：步长，默认为1。例如：range（0， 5） 等价于 range(0, 5, 1)

列表推导式（列表生成式）：list1 = [ i for i in range(10)]		元素类型转换：list = [int(i) *2 for i in str]

条件判断：[(x,y) for x in list for y in str if x!=y]

字典推导式：格式：dict1 = {key: key**2 for key in range(1, 6)}

集合推导式：格式：list1 = [1, 2, 3]		set1 = {i**2 for i in list1}		输出内容：{1, 9}

\#集合有去重功能

# 函数

函数就是一段具有独立功能的代码快整合到一个整体并命名，在需要的位置调用这个名称即可完成对应的需求

1、#函数在开发过程中，可以更高效的实现代码重用

2、#如果不调用函数，定义函数内的代码将不会被执行

3、#当调用函数时，解释器会回到定义函数的地方执行下方缩进的代码，执行完继续回到调用函数的地方

return()函数：1、负责函数返回值		2、退出当前函数且下方代码（函数体内）不执行

使用步骤：1、定义函数	格式：def 函数名(参数1，参数2，...)：		#不同需求，参数可有可无

代码1

代码2

2、调用函数	格式：函数名(参数)

参数作用：能使函数变得更加灵活，能进行有效运算

实参：写在函数调用的位置

形参：写在函数括号内的位置

返回值作用：返回结果给函数调用的地方			return：返回值1，返回值2，...

函数说明文档：help()函数：用于查找函数说明的		格式：help(len)

\#定义函数的说明文档	语法：def	 函数名(参数)：

"""	说明文档的位置	"""

代码

...

函数嵌套：一个函数内部嵌套调用另一个函数

## 常用函数

print('my_Name')或print("my_name")：输出函数，输出括号的内容

'  '	：忽略所有特殊字符，将其中的内容都作为了字符串输出，转移字符依旧有效

print(my_name)：输出括号my_name定义的内容

%作用：连接变量

d = 1		print('今年我的年龄是%03d岁' % d)		#以%03d按百位数表示，不足以0补全，超出原样输出

b = 'lnb'		print('我的名字是%s' % b)

c = 70.2		print('我的体重是%0.1fkg' % c) 		#%f默认是保留小数点后至6位，可指定保留位数如0.1

print('我的名字是%s岁，今年%d岁了' % (b, a + 1))	#当输出多个不同类型的数据时，以，结尾添加

print('我的名字是%s岁，今年%s岁了' % (b, a))	#%s能格式化输出字符串，即变量输输入什么，%s就输出什么

 

\# format()格式化函数

根据对象所在的位置：print('{0} and {1}'.format('spam', 'eggs'))

关键字参数名引用值：print('The story of {0}, {1}, and {other}.'.format('Bill', 'Manfred',other='Georg'))

名称引用变量：table = {'Sjoerd': 4127, 'Jack': 4098, 'Dcab': 8637678}

print('Jack: {Jack:d}; Sjoerd: {Sjoerd:d}; Dcab: {Dcab:d}'.format(**table))Jack: 4098; Sjoerd: 4127; Dcab: 8637678

{}数字格式化{:.2f}保留小数点后两位3.14、	{:.0f}不带小数、			{:>10d}右对齐 (默认, 宽度为10)    13

{:+.2f}带符号保留小数点后两位+3.14、		{:.2%}百分比格式25.00%、	{:x<4d}数字补x (填充右边, 宽度为4) 5xxx

print(f'我的名字{b},我的学号是{d}')

另一种表达方法：f'{表达式}' 

print(type(num1)) #输出num1的数据类型type

 

input('提示信息\n') # 这个函数和print函数非常类似

\# 先input函数等待接收到用户输入（全部当作字符串%s处理）后，一般先存储到变量

\# 后input函数会将用户输入的数据，当作字符串输出(需要转换格式）

验证用户是否接收到用户输入的数据：password = input('提示信息\n')

print('%s' % password)

lambda函数：

如果一个函数有一个返回值，并且只有一句话代码，可以使用lambda简化

语法：lambda  参数：表达式(返回值)			#直接打印lambda表达式，输出的是此lambda的内存地址

1、 参数可有可无，参数为1、可变参数*args时，默认输出元组	

2、不定长可变参数**args时，默认输出字典

3、表达式能接收任意数量的参数但只能返回一个表达式的值

格式：fn1 = lambda  a，b：a + b		print(fn1(1, 2))		输出内容：3

## 数据类型转换函数

函数				描述

int(x [,base])		将x转换为一个整数					long(x [,base] )			将x转换为一个长整数

float(x)			将x转换到一个浮点数				complex(real [,imag])	创建一个复数

str(x)			将对象 x 转换为字符串				repr(x)				将对象 x 转换为表达式字符串

eval(str)			用来计算在字符串中的有效Python表达式,并返回一个对象

tuple(s)			将序列 s 转换为一个元组				list(s)				将序列 s 转换为一个列表

set(s)			转换为可变集合					frozenset(s)			转换为不可变集合

dict(d)			创建一个字典。d 必须是一个序列 (key,value)元组。

chr(x)			将一个整数转换为一个字符			unichr(x)				将一个整数转换为Unicode字符

ord(x)			将一个字符转换为它的整数值

hex(x)			将一个整数转换为一个十六进制字符串	oct(x)			将一个整数转换为一个八进制字符串

## 可迭代对象常用处理函数

type(num1) #type()函数输出num1的数据类型

bin(num)将num十进制转换为二进制

iter(num).next()迭代器：将num容器类型数据作为对象传入迭代器，next()方法进行迭代

zip([iterable, ...])压缩函数，接收多个可迭代对象，后把每个传送对象中第i个元素组成一个新的迭代器(元组类型的)

sorted(iterable, cmp=None, key=function, reverse=False)排序函数对所有可迭代的对象进行排序操作，返回一个新的列表

参数：iterable=(容器型数据、range序列、迭代器)、reverse排序默认为true、key=自定义或内置函数

map(function, iterable, ...)根据提供的函数对传入对象中每个元素进行处理，返回一个新的迭代器

reduce(function, iterable[, initializer])对传入的可迭代对象中的元素进行function函数运算累积

需要引入 functools 模块导入该函数

filter(function, iterable)过滤函数，function返回Ture或false过滤掉不符合条件的元素，返回由符合条件元素组成的新列表

## 函数参数类型

变量的作用域：变量生效的范围

局部变量：只在某个函数体内部生效的变量		全局变量：在函数体内外部生效的变量	global a=20

函数的参数：传递和定义的参数顺序必须一致

1、 位置参数：根据前后位置来传递参数def user_info(name，age，gender)	user_info('TOM'，20，'男')

2、 关键字参数：函数调用，通过"键=值"形式加以指定。让函数更加清晰、容易使用，同时也清除了参数的顺序需求

格式：def user_info(name，age，gender)		user_info('TOM'，age=20，gender='男')

\#TOM位置参数必须在关键字参数前，关键字参数无顺序要求

3、 缺省参数(默认参数)：定义函数时，为参数提供默认值（传递参数时，没有对该参数进行传递，则使用默认值）

格式1：def user_info(name，age，gender='男')		user_info('TOM'，age=20)

格式2：def user_info(name，age，gender='男')		user_info('TOM'，age=20，gender='女')

4、不定长参数(可变参数)：用于不确定调用时会传递多少个参数(不传递也可以)的场景

包裹位置传递：接收所有不定个数的位置参数用的

格式：def user_info(*args)		user_info('TOM')

\#传递进的所有数据都被args变量收集，根据传进参数的位置合并成一个元组(tuple)

包裹关键字传递：接收关键字参数且不确定个数时

格式：def user_info(**args)		user_info(name='TOM'，age=20，gender='男')

\#无论是包裹位置还是关键字传递，都是一个组包的过程

拆包：将函数的返回数据进行拆分

拆包：元组：格式：def return_num():		return 100,200		n1，n2=return()

字典：dict1 = {'name'：'TOM'，'age'：18}	a，b=dict1

交换：将多个变量的值进行互换

1、 借助第三变量储存数据	c=0	c=a	a=b	b=c

2、 借助多重赋值			a，b=1，2	a，b=b，a

引用：在python中，值是靠引用来传递来的

\#id()函数查看变量的id值(可以理解为在内存的地址标识)，即判断两个变量是否为同一个值引用

格式：print(id(b))

可变和不可变类型：

不可变类型：int整型、float浮点型、str字符串、tuple元组

可变类型：list[]列表、dict{}字典、{}集合

## 递归函数

特点：1、函数内部自己调用自己，2、必须有出口

```python
格式解析：def num_sum(num):
  		 if num == 1:
			return 1
  			return num + num_sum(num-1)	#递归口

print(num_sum(3))
```

<img src="E:\Project\Textbook\Python\assets\wps11.jpg" alt="img" style="zoom: 67%;" /> 

# 文件操作

input输入/output输出	

步骤：1、打开文件方法

open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)

file: 必需，文件路径（相对路径：../1.txt或者C:/users/appdata/1.txt绝对路径）		

mode: 可选，文件打开模式(1、'r'读取（默认）、不存在则报错，存在则打开文件，指针在文件最前面 

2、'w'写入，不存在文件则创建，存在则打开并清空，打开后文件的指针在文件的最前面、

3、'x'排它性创建，如果文件已存在则失败、

4、'a'打开文件用于写入，如果文件存在则指针在末尾追加、5、'b'二进制模式、6、't'文本模式（默认）、

7、'+'打开用于更新（读取与写入）)，'w+'对文件先写后读(w先清空文件所以什么内容都没有)			

encoding: 默认为二进制字符集，一般使用utf-8

buffering: 设置缓冲							errors: 报错级别							

newline: 区分换行符

closefd: 传入的file参数类型					opener: 设置自定义开启器，开启器的返回值必须是一个打开的文件描述符

 

2、读写文件

file.read([size])方法，从当前指针位置开始读，到指定size字节数的地方，如果未给定或为负则读取所有

file.readline([size])严格且只读取当前指针所在的行，包括 "\n" 字符

file.readlines([sizeint])读取所有行并返回一个由每行组成一个元素的列表

file.write(str)只能将字符串类型内容写入文件

file.writelines(iterable)只能向文件写入一个字符串类型元素的容器，换行需要自己加入每行的换行符

3、关闭文件cloese()方法

 

常用方法：

seek([offset,] whence)：指针位置修改函数，offset偏移字节数，

whence：指针的位置，默认为0即从文件开头开始算起，1 代表从当前位置开始算起，2 代表从文件末尾算起

file.tell()返回当前指针位置

file.truncate([size])从当前指针位置截取 size 个字符进行保留其他全部去掉

# 内置模块(需导入)

## 序列化模块

序列化是指可以把python中的数据，以文本或进制的方式进行转换， 并且还能反序列化为原来的数据

将对象转换为可通过网络传输或可以存储到本地磁盘的数据格式（如：XML、JSON或特定格式的字节串）的过程称为序列化

python中各种容器类型数据都是对象

 

pickle二进制序列化模块

dumps(python对象)序列化函数，可以把一个python的任意对象序列化成为一个二进制对象并返回

loads()反序列化函数，可以把一个序列化后的二进制数据反序列化为python对象并返回

dump(obj对象，open的文件对象)文件序列化函数， 把一个数据对象进行序列化并写入到文件中

load()文件反序列化，在一个文件中读取序列化的数据，并且返回一个反序列化对象

json文本序列化模块(JavaScript Object Notation)

可以把一些符合转换的python数据对象，转为j son格式的数据

JSON在js语言中是一个对象的表示方法，和Python中字典的定义规则和语法几乎一样

JSON在互联网中又是轻量级的文本数据交换格式， 数据传输，数据定义的一种数据格式

dumps(python对象)序列化函数

loads()反序列化函数

dump(python对象，open的文件对象): 对数据进行编码

load(open的文件对象): 对数据进行解码，一个文件中读取序列化的数据，并且返回一个反序列化对象

## 数学模块

math.ceil(x)向上取整函数，返回一个大于或者等于 x 的最小整数

math.floor(x)向下取整函数，返回一个小于或等于 x 的最大整数

math.pow(x, y)幂函数函数，x 的 y 次幂，返回浮点型数据

math.sqrt(x)平方根函数，返回 x 的开平方根，返回浮点型数据

math.fabs(x)绝对值函数，返回 x 的绝对值，返回浮点型数据

math.modf(x)浮点数据拆分， 将x 的小数和整数拆分。返回一个浮点型元素组成的元组

math.copysign(x, y)将y的正负赋值给x并返回浮点型元素，如：copysign(1.0, -0.0) 返回 -1.0

math.fsum(iterable)将可迭代对象内的元素进行求和并返回一个浮点数型的结果

math.pi数学常数 π = 3.141592...，精确到可用精度

math.e数学常数 e = 2.718281...，精确到可用精度

## random随机模块

random.random()返回 [0.0, 1.0) 范围内的一个随机浮点数（左闭右开）

random.randrange([start=0,] stop[, step])从指定范围内返回一个随机整数

random.randint(a, b)返回随机整数 N 满足 a <= N <= b

random.uniform(a, b)返回一个随机浮点数 N

random.choice(seq)从非空容器 seq 返回一个随机元素。 如果 seq 为空，则引发 IndexError

random.shuffle(x[, random])将非空容器x内元素随机打乱位置，并返回

## 系统接口模块

os.getcwd()返回当前工作目录

os.chdir('path')改变当前工作目录为path路径

os.listdir(['path])返回path路径下所有的文件或文件夹的名字组成的str类型元素的列表

os.makedirs(path[, mode])递归创建文件夹函数

os.mkdir('path'[, 0777])创建一个权限数字为名为path的文件夹（无法递归创建），默认mode=0777 (八进制)

os.removedirs('path')递归删除空文件夹

os.rmdir('path')删除path指定的空文件夹，如果目录非空，则抛出一个OSError异常

os.remove('path')删除路径为path的文件。如果path 是一个文件夹，将抛出OSError

os.renames(old, new)递归地对目录/文件进行更名

os.rename(src, dst)重命名文件或目录，将src命名为dst

os.system('command')在子外壳程序中执行命令

os.path路径模块，获取文件的属性信息

os.path.abspath(path)将相对路径转换为绝对路径，并返回

os.path.basename(path)返回路径 path 的基本名称（文件名/目录名）

os.path.dirname(path)返回路径 path 的基本名称外的路径

os.path.join(path, *paths)拼接一个或多个路径部分，并返回

os.path.split(path)将path中的路径和文件名拆分，并以str类型元素存储在元组中返回，若无文件则为空

os.path.splitext(path)将path中的路径和文件后缀名拆分，并以str类型元素存储在元组中返回，若无文件则为空

os.path.getsize(path)返回 path路径中文件的大小，以字节为单位，没有文件则报错

os.path.isdir(path)如果 path 是 现有的 目录，则返回 True

os.path.isfile(path)如果 path 是现有的 常规文件/link符号链接，则返回 True

os.path.islink(path)如果 path 指向的 现有 目录条目是一个符号链接，则返回 True

os.path.exists(path)检测 path路径中的文件或文件夹是否存在，返回 True

os.path.samefile(path1, path2)检测两个路径是否指向相同的文件或目录，则返回 True。这由设备号和 inode 号确定

## shutil(shell utility)高阶文件操作模块

shutil.copyfile(src, dst, *, follow_symlinks=True)将名为 src 的文件的内容（不包括元数据）拷贝到名为 dst 的文件，并返回 dst

shutil.copy2('src', 'dst', *, follow_symlinks=True)将文件src拷贝到文件或目录 dst并保留源文件信息（操作时间、权限等），follow_symlinks=True即src为符号链接进行拷贝

shutil.copy('src', 'dst', *, follow_symlinks=True)将文件 src 拷贝到文件或目录 dst

shutil.copytree(src, dst[, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False, dirs_exist_ok=False])将src整个目录树结构及其内容拷贝到名为 dst 的目录并返回目标目录，要求dst目录不存在，

shutil.rmtree(path[, ignore_errors=False, onerror=None])删除一个整个目录树，path必须指向一个目录（不能是一个目录的符号链接）,ignore_errors=True删除失败导致的错误将被忽略,此类错误将通过调用由 onerror 所指定的处理程序来处理

shutil.move(src, dst[, copy_function=copy2])递归地将一个文件或目录 (src) 移至另一位置 (dst) 并返回目标位置，若是文件存在则覆盖，也可用于修改文件或文件夹的名称

## zipfile解压缩模块

和file文件操作类似

zipfile.ZipFile(file, mode='r', compression=ZIP_STORED, allowZip64=True, compresslevel=None, *, strict_timestamps=True)

1.参数file表示文件的路径或类文件对象(file-like object);

2.参数mode指示打开zip文件的模式，默认值为'r'，表示读已经存在的zip文件，也可以为'w'或'a'，w'表示新建一个zip文档或覆盖一个已经存在的zip文档，'a'表示将数据附加到一个现存的zip文档中;

3.参数compression表示在写zip文档时使用的压缩方法，它的值可以是zipfile. ZIP_STORED 或zipfile. ZIP_DEFLATED。如果要操作的zip文件大小超过2G，应该将allowZip64设置为True。

ZipFile.write(filename, arcname=None, compress_type=None, compresslevel=None)

将名为 filename 的文件写入归档，arcname=指定的归档名

ZipFile.extract(member, path=None, pwd=None)

从归档中提取出一个成员放入当前工作目录

 

 

 

# 面向对象OOP

创建对象的时候，并不会把类中的属性和方法复制一份给对象，而是在对象中引用父类的方法

因此在访问对象的属性时，会先去找对象自己的属性，如果没有就去找这个类的属性和方法

对象：对象的属性/方法能进行引用、添加、修改、覆盖、删除（不能对引用的类方法和类属性操作），

类：支持实例化，属性能进行引用、添加、修改、覆盖、删除，类的方法除了不能进行访问可以进行其他操作

\#对类的操作会直接影响引用该类的对象

 

self：类的实例（引用该类的对象）		#不含self的方法不能被对象调用，但能使用类调用

 

属性：对象属性、类属性、私有属性

内置属性：__dict__返回类/对象所有的属性，组成的字典

__name__返回类、函数、方法的名字，组成的字符串

__module__获取类所在的文件名称，如果是当前文件，显示为_ _mian_ _

__class__返回对象或类所属的类  只是返回基类

__bases__获取当前类的父类列表，__base__获取当前类的父类

__doc__返回类/函数的文档字符串，如果没有定义则为None

__mro__获取当前类的继承链

方法：对象方法、类方法(方法中有cls这个形参，使用了@classmethod装饰器)、静态方法(使用@staticmethod修饰)、

私有方法(_ _方法名)、

内置方法： __init__(self,[...])初始化方法，在类实例化对象后，自动触发的一个方法

__del__(self)析构方法()类实例化对象被销毁时自动触发

_ _dict_ _ 方法：能获取当前类/对象的的

__new__(cls,[...)是对象实例化时第一个调用的方法，只取下 cls 参数并把其他参数传给 __init__ 

通过return object.__new__(cls)返回一个对象给init和del方法，否则init和del方法无self参数无法执行

__call__(self)方法，需在类中自己定义，可以将对象当成函数进行调用，用于类的解释说明调用

__length__(self)方法，需在类中自己定义，返回对象的长度

__str__(self)方法,需在类中自己定义，使用print打印对象时，会被调用

__repr__(self)方法,需在类中自己定义，使用print打印对象时，会被调用

__bool__(self)方法,需在类中自己定义，返回一个bool类型的值

成员相关魔术方法：

__getattribute__(self, name)当访问属性时(无论属性是否存在)，触发该方法

作用：可以在获取对象成员时，对数据进行一些处理

如果想要访问对象的成员必须使用object.__getattr__(self, item)来进行访问

__getattr__(self, name)当访问属性不存在时，触发该方法，#优先触发getattribute方法

作用：防止访问不存在的成员时报错，也可以为不存在的成员进行赋值操作

__setattr__(self，key，value)当给对象成员进行添加、修改时，触发该方法

作用：可以限制或管理对象成员中key键和值value的添加和修改操作

__delattr__(self，item)当删除对象成员时，自动触发

作用：可以去限制对象成员的删除，还可以删除不存在成员时防止报错

如果想要访问对象的成员必须使用object.__delattr__(self, item)来进行访问

 

访问成员顺序：1.调用__getattribute__魔术方法		2.调用数据描述符[后面会讲]

3.调用当前对象的成员							4.调用当前类的成员

5.调用非数据描述符[后面会讲]					6.调用父类的成员

7.调用__getattr__魔术方法

 

描述符类：当一个类中，包含(_ _get__.__set__，__del__ )中至少一个魔术方法时，都包含时为数据描述符

作用：定义描述符类对另一个类中的某个成员进行一个详细的管理操作(获取，赋值，删除)

__get__(self, instance, owner)：调用一个属性时触发

__set__(self, instance, value)：每次属性赋值时触发，instance，value值

__del__(self, instance)：删除属性时触发

 

对象销毁情况：1、程序执行完毕，内存中所有的资源都被销毁

2、使用del删除时

3、对象不再被引用时，会自动销毁

 

 

 

## 三大特性

封装：public公有的（默认）、protected受保护（_属性/方法）、prviate私有的（_ _属性/方法）

<img src="E:\Project\Textbook\Python\assets\wps12.jpg" alt="img" style="zoom: 67%;" /> 

\#在python中给成员进行私有化，其实就是改了成员的名字		私有化==》 _类名_ _成员

 

继承：def 子类(父类，母类)

\#私有成员不能继承、#子类可以多继承（区别于java）、#所有类默认继承object类

继承对象属性的获取：实例对象的属性==>类的属性===>继承的父类===>__getattr__()方法

菱形继承：class d(b,c)	继承顺序：d>b父类>c母类>a祖先类>object类(采用广度优先算法)

super()方法.父类方法()：让子类可以调用父类的方法/属性（实际上是super调用mro列表的上一级中的方法/属性）

类.mro()方法：获取类继承关系列表

 

多态：子类实现的父类的方法，由于重写的方法内容不同，最终实现不同的结果		# obj.方法()

 

常用方法：

issubclass(A，B)方法：检测A类是否继承自B类，返回Ture或False

isinstance(A，B) 方法：检测A对象是否为B类的实例化对象

hasattr(D，'属性') 方法： 检测D 类/对象是否包含指定名称的属性/方法，返回Ture或False

getattr(D，'成员') 方法：获取D类/对象成员的属性值

setattr(D，'成员'，‘属性值’) 方法：设置D类/对象的成员的属性值

delattr(D，‘属性’) 方法：删除D类/对象的属性

 

 

 

单例模式：是一种创建型设计模式，它确保一个类有且只有一个特定类型的对象，并提供全局访问点。

1、确保类有且只有一个对象被创建

2、为对象提供一个访问点，使程序可以全局访问该对象

3、控制共享资源的并行访问

4、可以避免消耗过多的内存或CPU资源

 

Mixin混合设计模式：python中的Mixin是通过多继承实现的

Mixin必须是表示一种功能，而不是一个对象。

Mixin的功能必须单一，如果有多个功能，那就多定义Mixin类

Mixin类通常不单独使用，而是混合到其它类中，去增加功能的

Mixin 类不依赖子类的实现，即便子类没有继承这个Mixin,子类也能正常运行，可能就是缺少了一些功能。。

 

抽象类;与java一样，python也有抽象类的概念但是同样需要借助模块实现，**抽象类是一个特殊的类，它的特殊之处在于只能被继承，不能被实例化,需要子类实现抽象方法**

 

装饰器：在不改变原有函数代码，且保持原函数调用方法不变的情况下，给原函数增加新的功能(或者给类增加属性和

方法)

 

 

 

 

 

 

# pip工具使用

curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py  # 下载安装脚本

yum install python3-pip -y	# linux安装pip

 

python get-pip.py   # 运行安装脚本

pip3 list --outdated #pip检查哪些包需要更新

pip3 install 包名(模块) #安装

pip3 install -U pip #升级 pip  或easy_install --upgrade pip命令

pip3 uninstall SomePackage #卸载包

pip3 search SomePackage #搜索包

pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/	#永久安装ali源

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple package --trusted-host mirrors.aliyun.com

\#临时使用清华大学开源软件镜像站



pip3 download flask #下载flask依赖库，到本级路径

pip3 install pocketsphinx-0.1.15-cp39-cp39-win_amd64.whl #python3安装本地whl



# 编译安装

3. 解压源码包

​	root@master ~# tar -xf Python-3.7.0.tar.xz

4. 编译安装
root@master ~# ./configure --prefix=/usr/local/python3.7
root@master ~# make
root@master ~# make install
5. 设置软连接
root@master ~# ln -fs /usr/local/python3.7/bin/python3 /usr/bin/python3
root@master ~# ln -fs /usr/local/python3.7/bin/pip3 /usr/bin/pip3

