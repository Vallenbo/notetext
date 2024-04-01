# 注释

\#或 Crtl+/：注释，进行解释说明，放哪里都行

多行注释：

```python
"""	# 这是多行注释						
'''	# 这也是多行注释
```

第二行注释			第二行注释

"""					'''

# 变量

```python
变量名 = 值
```

变量：就是存储数据的时候，当前数据所在的内存地址的名字

定义变量：

1、不能由数字开头、不能使用内置关键字	

2、由数字、字母、下划线组成	

3、严格区分大小写

# 关键字

```python
False     None    True   and      as       assert   break     class  
continue  def     del    elif     else     except   finally   for
from      global  if     import   in       is       lambda    nonlocal
not       or      pass   raise    return   try      while     with  	yield
```

 

nonlocal关键字使外函数中的内函数能使用局部变量

变量名 = 值	例：my_name = 'TOM'	#定义字符

命名习惯：

大驼峰：首字母大写法，例：MyName

小驼峰：第二个及以后的单词，首字母大写，例：myName

下划线：例：my_name

# 数据类型

<img src="E:\Project\Textbook\Python\assets\wps4.jpg" alt="img" style="zoom: 50%;" /> 

> 检测数据类型的方法：`type()`

```python
a = 1
print(type(a))  # <class 'int'> -- 整型

b = 1.1
print(type(b))  # <class 'float'> -- 浮点型

c = True
print(type(c))  # <class 'bool'> -- 布尔型

d = '12345'
print(type(d))  # <class 'str'> -- 字符串

e = [10, 20, 30]
print(type(e))  # <class 'list'> -- 列表

f = (10, 20, 30)
print(type(f))  # <class 'tuple'> -- 元组

h = {10, 20, 30}
print(type(h))  # <class 'set'> -- 集合

g = {'name': 'TOM', 'age': 20}
print(type(g))  # <class 'dict'> -- 字典
```



# print输出

格式化字符串除了%s，还可以写为`f'{表达式}'`

```python
age = 18 
name = 'TOM'
weight = 75.5
student_id = 1

# 我的名字是TOM
print('我的名字是%s' % name)

# 我的学号是0001
print('我的学号是%4d' % student_id)

# 我的体重是75.50公斤
print('我的体重是%.2f公斤' % weight)

# 我的名字是TOM，今年18岁了
print('我的名字是%s，今年%d岁了' % (name, age))

# 我的名字是TOM，明年19岁了
print('我的名字是%s，明年%d岁了' % (name, age + 1))

# 我的名字是TOM，明年19岁了
print(f'我的名字是{name}, 明年{age + 1}岁了')
```

> f-格式化字符串是Python3.6中新增的格式化方法，该方法更简单易读。

## 格式化输出数据

| 格式符号 | 转换                   |
| :------- | :--------------------- |
| ==%s==   | 字符串                 |
| ==%d==   | 有符号的十进制整数     |
| ==%f==   | 浮点数                 |
| %c       | 字符                   |
| %u       | 无符号十进制整数       |
| %o       | 八进制整数             |
| %x       | 十六进制整数（小写ox） |
| %X       | 十六进制整数（大写OX） |
| %e       | 科学计数法（小写'e'）  |
| %E       | 科学计数法（大写'E'）  |
| %g       | %f和%e的简写           |
| %G       | %f和%E的简写           |

> 技巧

- %06d，表示输出的整数显示位数，不足以0补全，超出当前位数则原样输出
- %.2f，表示小数点后显示的小数位数。
- #%d即有+正或-负符号的整数

## 转义字符

- \n：换行		

- \t：制表符，一个tab键（4个空格）的距离



# input输入

```python
input("提示信息")
```

## 输入的特点

- 当程序执行到`input`，等待用户输入，输入完成之后才继续向下执行。
- 在Python中，`input`接收用户输入后，一般存储到变量，方便使用。
- 在Python中，`input`会把接收到的任意用户输入的数据都当做字符串处理。

```python
password = input('请输入您的密码：')

print(f'您输入的密码是{password}')
# <class 'str'>
print(type(password))
```



# 运算符

## 1. 算数运算符

| 运算符 |  描述  | 实例                                                  |
| :----: | :----: | ----------------------------------------------------- |
|   +    |   加   | 1 + 1 输出结果为 2                                    |
|   -    |   减   | 1-1 输出结果为 0                                      |
|   *    |   乘   | 2 * 2 输出结果为 4                                    |
|   /    |   除   | 10 / 2 输出结果为 5                                   |
|   //   |  整除  | 9 // 4 输出结果为2                                    |
|   %    |  取余  | 9 % 4 输出结果为 1                                    |
|   **   |  指数  | 2 ** 4 输出结果为 16，即 2 * 2 * 2 * 2                |
|   ()   | 小括号 | 小括号用来提高运算优先级，即 (1 + 2) * 3 输出结果为 9 |

> 注意：

- 混合运算优先级顺序：`()`高于 `**` 高于 `*` `/` `//` `%` 高于 `+` `-`



## 2. 赋值运算符

| 运算符 | 描述 | 实例                                |
| ------ | ---- | ----------------------------------- |
| =      | 赋值 | 将`=`右侧的结果赋值给等号左侧的变量 |

- 多个变量赋值

```python
num1, float1, str1 = 10, 0.5, 'hello world'
print(num1)
print(float1)
print(str1)
```

- 多变量赋相同值

```python
a = b = 10
print(a)
print(b)
```

## 3. 复合赋值运算符 

| 运算符 | 描述           | 实例                       |
| ------ | -------------- | -------------------------- |
| +=     | 加法赋值运算符 | c += a 等价于 c = c + a    |
| -=     | 减法赋值运算符 | c -= a 等价于 c = c- a     |
| *=     | 乘法赋值运算符 | c *= a 等价于 c = c * a    |
| /=     | 除法赋值运算符 | c /= a 等价于 c = c / a    |
| //=    | 整除赋值运算符 | c //= a 等价于 c = c // a  |
| %=     | 取余赋值运算符 | c %= a 等价于 c = c % a    |
| **=    | 幂赋值运算符   | c ** = a 等价于 c = c ** a |



## 4. 比较运算符

比较运算符也叫关系运算符， 通常用来判断。

| 运算符 | 描述                                                         | 实例                                                        |
| ------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| ==     | 判断相等。如果两个操作数的结果相等，则条件结果为真(True)，否则条件结果为假(False) | 如a=3,b=3，则（a == b) 为 True                              |
| !=     | 不等于 。如果两个操作数的结果不相等，则条件为真(True)，否则条件结果为假(False) | 如a=3,b=3，则（a == b) 为 True如a=1,b=3，则(a != b) 为 True |
| >      | 运算符左侧操作数结果是否大于右侧操作数结果，如果大于，则条件为真，否则为假 | 如a=7,b=3，则(a > b) 为 True                                |
| <      | 运算符左侧操作数结果是否小于右侧操作数结果，如果小于，则条件为真，否则为假 | 如a=7,b=3，则(a < b) 为 False                               |
| >=     | 运算符左侧操作数结果是否大于等于右侧操作数结果，如果大于，则条件为真，否则为假 | 如a=7,b=3，则(a < b) 为 False如a=3,b=3，则(a >= b) 为 True  |
| <=     | 运算符左侧操作数结果是否小于等于右侧操作数结果，如果小于，则条件为真，否则为假 |                                                             |



## 5. 逻辑运算符

| 运算符 | 逻辑表达式 | 描述                                                         | 实例                                     |
| ------ | ---------- | ------------------------------------------------------------ | ---------------------------------------- |
| and    | x and y    | 布尔"与"：如果 x 为 False，x and y 返回 False，否则它返回 y 的值。 | True and False， 返回 False。            |
| or     | x or y     | 布尔"或"：如果 x 是 True，它返回 True，否则它返回 y 的值。   | False or True， 返回 True。              |
| not    | not x      | 布尔"非"：如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。 | not True 返回 False, not False 返回 True |

```python
a = 1
b = 2
c = 3
print((a < b) and (b < c))  # True
print((a > b) and (b < c))  # False
print((a > b) or (b < c))   # True
print(not (a > b))          # True
```

# 流程控制语句

continue关键字：退出当前循环，进入下一次循环

break关键字：终止此循环

return关键字：并不是专门用于跳出循环的，return的功能是结束一整个方法，不管这个return处于多少层循环之内

## if语法（条件语句）

基础版：

```py
if 变量 判断式 数据：
	print('为ture时输出的内容')
```

进阶版：

```py
if 变量 判断式 input('请输入内容：')：
	print('为ture时输出的内容')
```

多条件版 ：

```python
if 变量 判断式 input('请输入内容：')：
	print('为ture时输出的内容')
else：
	print('为false时输出的内容')
```

多重判断：

```python
age = input('请输入内容：')

if age变量(age) 判断式 数据：
	print('符合该条件时输出的内容')
elif 变量(age) 判断式 数据：
	print('符合该条件时输出的内容')
else：
	print('为false时输出的内容')
```

- 三目运算符 ：

  ```python
  a = 1 	b = 2	c = a 
  if a > b else b		#if语句成立取a，否则取b值
  ```


## while语法（循环）

基础格式 ：	

```python
while 变量 判断式 数据：
	print('符合条件时输出的内容')	#当输出该行内容时，继续回到while进行判断，直到不符合条件
```

循环判断式：	

```python
while 变量 判断式 数据：
	print('符合条件时输出的内容')
	#break	#break终止循环的情况else下的代码将不会被执行
	#continue #continue退出当前循环且按正常循环走流程结束
else：#else循环：正常的循环及走流程结束
	print('符合条件时输出的内容')
```

嵌套格式 ：	

```python
while 变量 判断式 数据：
	print('符合条件时输出的内容')	
	while 变量 判断式 数据：
		print('符合条件时输出的内容')	
```



## for语法

基本格式：

```python
for 临时变量 in 序列/数据集：				#当然我们也可以使用break和continue语法
```

循环判断正常格式：

```python
for 临时变量 in 序列/数据集：
	print('符合条件时，执行的代码')
    #break	#break终止循环的情况else下的代码将不会被执行
	#continue	#continue退出当前循环且按正常循环走流程结束
else：
	print('循环正常结束之后要执行的代码')
```



# 容器数据类型使用总结

str和list都可以通过下标的方式取出数据、list还能通过列表循环，即for和while取出

dict字典，能按照指定key查找数据、能返回所有key、能返回所有数据、能将字典数据返回成元组数据

**下标**：通过下标快速找到对应的数据（用于标识字符串中某一子串（字符））

```python
语法：str5 = "love"		print(str5[0])	#下标标识，从0开始计识
```

**切片**：指对操作的对象截取其中一部分操作。字符串、列表、元组都支持切片操作

```python
语法：序列[开始位置下标：结束位置下标：步长]
```

\#不包含结束位置下标对应的数据

\#正数下标默认为0，步数默认为1

\#负数下标默认为-1，负步数（表示从右开始）

```python
str6 = '0123456789'
print(str6[-3:-1:1])	输出内容：78

str[2:6]='a' 对指定下标数据进行更新
del str[2:6]对指定下标数据进行删除
```



# str字符串

字符串是 Python 中最常用的数据类型。我们一般使用引号来创建字符串。创建字符串很简单，只要为变量分配一个值即可。

``` python
a = 'hello world'
b = "abcdefg"
print(type(a))
print(type(b))
```

> 注意：控制台显示结果为`<class 'str'>`， 即数据类型为str(字符串)。

\#单双引号和三引号的区别：

```python
'Hello'  # 单引号不能进行分行内容输出
"I’m TOM"	# 单引号不能用于带有引号‘的表示式(如上)字符串(也可用\转义字符解决)
'''Tom''' 或者 """Rose"""	# 三引号可以进行分行内容输出
```

### str取下标

```python
name = "abcdef"

print(name[1])
print(name[0])
print(name[2])

>b
>a
>c
```

### str切片操作

```python
序列[开始位置下标:结束位置下标:步长]

# 注意：
# 1. 不包含结束位置下标对应的数据， 正负整数均可；
# 2. 步长是选取间隔，正负整数均可，默认步长为1。
```

```python
name = "abcdefg"
print(name[2:5:1])  # cde
print(name[2:5])  # cde
print(name[:5])  # abcde
print(name[1:])  # bcdefg
print(name[:])  # abcdefg
print(name[::2])  # aceg
print(name[:-1])  # abcdef, 负1表示倒数第一个数据
print(name[-4:-1])  # def
print(name[::-1])  # gfedcba
```



### str常用操作方法

字符串的常用操作方法有查找、修改和判断三大类。

#### 查找

所谓字符串查找方法即是查找子串在字符串中的位置或出现的次数。

- find()：检测某个子串是否包含在这个字符串中，如果在返回这个子串开始的位置下标，否则则返回-1。

1. 语法

``` python
字符串序列.find(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.find('and'))  # 12
print(mystr.find('and', 15, 30))  # 23
print(mystr.find('ands'))  # -1
```

- index()：检测某个子串是否包含在这个字符串中，如果在返回这个子串开始的位置下标，否则则报异常。

1. 语法

``` python
字符串序列.index(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.index('and'))  # 12
print(mystr.index('and', 15, 30))  # 23
print(mystr.index('ands'))  # 报错
```

- rfind()： 和find()功能相同，但查找方向为==右侧==开始。
- rindex()：和index()功能相同，但查找方向为==右侧==开始。
- count()：返回某个子串在字符串中出现的次数

1. 语法

``` python
字符串序列.count(子串, 开始位置下标, 结束位置下标)
```

> 注意：开始和结束位置下标可以省略，表示在整个字符串序列中查找。

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.count('and'))  # 3
print(mystr.count('ands'))  # 0
print(mystr.count('and', 0, 20))  # 1
```

####  修改

所谓修改字符串，指的就是通过函数的形式修改字符串中的数据。

- replace()：替换

1. 语法

``` python
字符串序列.replace(旧子串, 新子串, 替换次数)
```

> 注意：替换次数如果查出子串出现次数，则替换次数为该子串出现次数。

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.replace('and', 'he')) # 结果：hello world he itcast he itheima he Python
print(mystr.replace('and', 'he', 10)) # 结果：hello world he itcast he itheima he Python
print(mystr) # 结果：hello world and itcast and itheima and Python
```

> 注意：数据按照是否能直接修改分为==可变类型==和==不可变类型==两种。字符串类型的数据修改的时候不能改变原有字符串，属于不能直接修改数据的类型即是不可变类型。



- split()：按照指定字符分割字符串。

1. 语法

``` python
字符串序列.split(分割字符, num)
```

> 注意：num表示的是分割字符出现的次数，即将来返回数据个数为num+1个。

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.split('and')) # 结果：['hello world ', ' itcast ', ' itheima ', ' Python']
print(mystr.split('and', 2)) # 结果：['hello world ', ' itcast ', ' itheima and Python']
print(mystr.split(' ')) # 结果：['hello', 'world', 'and', 'itcast', 'and', 'itheima', 'and', 'Python']
print(mystr.split(' ', 2)) # 结果：['hello', 'world', 'and itcast and itheima and Python']
```

> 注意：如果分割字符是原有字符串中的子串，分割后则丢失该子串。
>

- join()：用一个字符或子串合并字符串，即是将多个字符串合并为一个新的字符串。

1. 语法

``` python
字符或子串.join(多字符串组成的序列)
```

2. 快速体验

``` python
list1 = ['chuan', 'zhi', 'bo', 'ke']
t1 = ('aa', 'b', 'cc', 'ddd')

print('_'.join(list1)) # 结果：chuan_zhi_bo_ke
print('...'.join(t1)) # 结果：aa...b...cc...ddd
```



- capitalize()：将字符串第一个字符转换成大写。

``` python
mystr = "hello world and itcast and itheima and Python"
print(mystr.capitalize()) # 结果：Hello world and itcast and itheima and python
```

> 注意：capitalize()函数转换后，只字符串第一个字符大写，其他的字符全都小写。



- title()：将字符串每个单词首字母转换成大写。

``` python
mystr = "hello world and itcast and itheima and Python"
print(mystr.title()) # 结果：Hello World And Itcast And Itheima And Python
```



- lower()：将字符串中大写转小写。

``` python
mystr = "hello world and itcast and itheima and Python"
print(mystr.lower()) # 结果：hello world and itcast and itheima and python
```



- upper()：将字符串中小写转大写。

``` python
mystr = "hello world and itcast and itheima and Python"
print(mystr.upper()) # 结果：HELLO WORLD AND ITCAST AND ITHEIMA AND PYTHON
```



- lstrip()：删除字符串左侧空白字符。

![image-20190129213453010](./assets/image-20190129213453010.png)



- rstrip()：删除字符串右侧空白字符。

![image-20190129213558850](./assets/image-20190129213558850.png)



- strip()：删除字符串两侧空白字符。

![image-20190129213637584](./assets/image-20190129213637584.png)



- ljust()：返回一个原字符串左对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串。

1. 语法

``` python
字符串序列.ljust(长度, 填充字符)
```

2. 输出效果

![image-20190130141125560](./assets/image-20190130141125560.png)



- rjust()：返回一个原字符串右对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串，语法和ljust()相同。
- center()：返回一个原字符串居中对齐,并使用指定字符(默认空格)填充至对应长度 的新字符串，语法和ljust()相同。

![image-20190130141442074](./assets/image-20190130141442074.png)



#### 判断

所谓判断即是判断真假，返回的结果是布尔型数据类型：True 或 False。

- startswith()：检查字符串是否是以指定子串开头，是则返回 True，否则返回 False。如果设置开始和结束位置下标，则在指定范围内检查。

1. 语法

``` python
字符串序列.startswith(子串, 开始位置下标, 结束位置下标)
```

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python   "

print(mystr.startswith('hello')) # 结果：True
print(mystr.startswith('hello', 5, 20)) # 结果False
```

- endswith()：：检查字符串是否是以指定子串结尾，是则返回 True，否则返回 False。如果设置开始和结束位置下标，则在指定范围内检查。

1. 语法

``` python
字符串序列.endswith(子串, 开始位置下标, 结束位置下标)
```

2. 快速体验

``` python
mystr = "hello world and itcast and itheima and Python"

print(mystr.endswith('Python')) # 结果：True
print(mystr.endswith('python')) # 结果：False
print(mystr.endswith('Python', 2, 20)) # 结果：False
```

- isalpha()：如果字符串至少有一个字符并且所有字符都是字母则返回 True, 否则返回 False。

``` python
mystr1 = 'hello'
mystr2 = 'hello12345'

print(mystr1.isalpha()) # 结果：True
print(mystr2.isalpha()) # 结果：False
```

- isdigit()：如果字符串只包含数字则返回 True 否则返回 False。

``` python
mystr1 = 'aaa12345'
mystr2 = '12345'

print(mystr1.isdigit()) # 结果： False
print(mystr2.isdigit()) # 结果：False
```

- isalnum()：如果字符串至少有一个字符并且所有字符都是字母或数字则返 回 True,否则返回 False。

``` python
mystr1 = 'aaa12345'
mystr2 = '12345-'

print(mystr1.isalnum()) # 结果：True
print(mystr2.isalnum()) # 结果：False
```

- isspace()：如果字符串中只包含空白，则返回 True，否则返回 False。

``` python
mystr1 = '1 2 3 4 5'
mystr2 = '     '

print(mystr1.isspace()) # 结果：False
print(mystr2.isspace()) # 结果：True
```





# list列表（序列）

列表：可以一次性存储多个数据，且可以为不同数据类型。

格式定义：

```python
[数据1, 数据2, 数据3, 数据4......]
```

[x for x in iterable]使用列表推导器、list()或list(iterable)使用类型构造器

## 查找

### 下标

``` python
name_list = ['Tom', 'Lily', 'Rose']

print(name_list[0])  # Tom
print(name_list[1])  # Lily
print(name_list[2])  # Rose
```

### 函数

- index()：返回指定数据所在位置的下标 。

1. 语法

``` python
列表序列.index(数据, 开始位置下标, 结束位置下标)
```

2. 快速体验

``` python
name_list = ['Tom', 'Lily', 'Rose']

print(name_list.index('Lily', 0, 2))  # 1
```

> 注意：如果查找的数据不存在则报错。

- count()：统计指定数据在当前列表中出现的次数。

``` python
name_list = ['Tom', 'Lily', 'Rose']

print(name_list.count('Lily'))  # 1
```

- len()：访问列表长度，即列表中数据的个数。

``` python
name_list = ['Tom', 'Lily', 'Rose']

print(len(name_list))  # 3
```



### 判断是否存在

- in：判断指定数据在某个列表序列，如果在返回True，否则返回False

``` python
name_list = ['Tom', 'Lily', 'Rose']

print('Lily' in name_list) # 结果：True
print('Lilys' in name_list) # 结果：False
```



- not in：判断指定数据不在某个列表序列，如果不在返回True，否则返回False

``` python
name_list = ['Tom', 'Lily', 'Rose']

print('Lily' not in name_list) # 结果：False
print('Lilys' not in name_list) # 结果：True
```

- 体验案例

需求：查找用户输入的名字是否已经存在。

``` python
name_list = ['Tom', 'Lily', 'Rose']

name = input('请输入您要搜索的名字：')

if name in name_list:
    print(f'您输入的名字是{name}, 名字已经存在')
else:
    print(f'您输入的名字是{name}, 名字不存在')
```



## 增加

作用：增加指定数据到列表中。

- append()：列表结尾追加数据。

1. 语法

``` python
列表序列.append(数据)
```

2. 体验

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list.append('xiaoming')

print(name_list) # 结果：['Tom', 'Lily', 'Rose', 'xiaoming']
```

![image-20190130160154636](./assets/image-20190130160154636.png)

> 列表追加数据的时候，直接在原列表里面追加了指定数据，即修改了原列表，故列表为可变类型数据。

3. 注意点

如果append()追加的数据是一个序列，则追加整个序列到列表

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list.append(['xiaoming', 'xiaohong'])

print(name_list) # 结果：['Tom', 'Lily', 'Rose', ['xiaoming', 'xiaohong']]
```



- extend()：列表结尾追加数据，如果数据是一个序列，则将这个序列的数据逐一添加到列表。

1. 语法

```python
列表序列.extend(数据)
```

2. 快速体验

   2.1 单个数据

```python
name_list = ['Tom', 'Lily', 'Rose']
name_list.extend('xiaoming')

print(name_list) # 结果：['Tom', 'Lily', 'Rose', 'x', 'i', 'a', 'o', 'm', 'i', 'n', 'g']
```

​	2.2 序列数据

```python
name_list = ['Tom', 'Lily', 'Rose']
name_list.extend(['xiaoming', 'xiaohong'])

print(name_list) # 结果：['Tom', 'Lily', 'Rose', 'xiaoming', 'xiaohong']
```



- insert()：指定位置新增数据。

1. 语法

``` python
列表序列.insert(位置下标, 数据)
```

2. 快速体验

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list.insert(1, 'xiaoming')

print(name_list) # 结果：['Tom', 'xiaoming', 'Lily', 'Rose']
```



## 删除

- del

1. 语法

``` python
del 目标
```

2. 快速体验

   2.1 删除列表

``` python
name_list = ['Tom', 'Lily', 'Rose']

del name_list # 结果：报错提示：name 'name_list' is not defined
print(name_list)
```

​	2.2 删除指定数据

``` python
name_list = ['Tom', 'Lily', 'Rose']

del name_list[0]
 
print(name_list) # 结果：['Lily', 'Rose']
```



- pop()：删除指定下标的数据(默认为最后一个)，并返回该数据。

1. 语法

``` python
列表序列.pop(下标)
```

2. 快速体验

``` python
name_list = ['Tom', 'Lily', 'Rose']

del_name = name_list.pop(1)

print(del_name) # 结果：Lily
print(name_list) # 结果：['Tom', 'Rose']
```



- remove()：移除列表中某个数据的第一个匹配项。

1. 语法

``` python
列表序列.remove(数据)
```

2. 快速体验

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list.remove('Rose')

print(name_list) # 结果：['Tom', 'Lily']
```



- clear()：清空列表

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list.clear()

print(name_list) # 结果： []
```



## 修改

- 修改指定下标数据

``` python
name_list = ['Tom', 'Lily', 'Rose']
name_list[0] = 'aaa'

print(name_list) # 结果：['aaa', 'Lily', 'Rose']
```



- 逆置：reverse()

``` python
num_list = [1, 5, 2, 3, 6, 8]
num_list.reverse()

print(num_list) # 结果：[8, 6, 3, 2, 5, 1]
```



- 排序：sort()

1. 语法

``` python
列表序列.sort( key=None, reverse=False)
```

> 注意：reverse表示排序规则，**reverse = True** 降序， **reverse = False** 升序（默认）

2. 快速体验

``` python
num_list = [1, 5, 2, 3, 6, 8]
num_list.sort()

print(num_list) # 结果：[1, 2, 3, 5, 6, 8]
```



## 复制

函数：copy()

``` python
name_list = ['Tom', 'Lily', 'Rose']

name_li2 = name_list.copy()

print(name_li2) # 结果：['Tom', 'Lily', 'Rose']
```



## 列表的循环遍历

需求：依次打印列表中的各个数据。

### while

- 代码

``` python
name_list = ['Tom', 'Lily', 'Rose']

i = 0
while i < len(name_list):
    print(name_list[i])
    i += 1
```

- 执行结果

![image-20190130164205143](./assets/image-20190130164205143.png)



### for

- 代码

``` python
name_list = ['Tom', 'Lily', 'Rose']

for i in name_list:
    print(i)
```

- 执行结果

![image-20190130164227739](./assets/image-20190130164227739.png)

## 列表嵌套

所谓列表嵌套指的就是一个列表里面包含了其他的子列表。

应用场景：要存储班级一、二、三三个班级学生姓名，且每个班级的学生姓名在一个列表。

``` python
name_list = [['小明', '小红', '小绿'], ['Tom', 'Lily', 'Rose'], ['张三', '李四', '王五']]
```

> 思考： 如何查找到数据"李四"？

``` python
print(name_list[2]) # 第一步：按下标查找到李四所在的列表
print(name_list[2][1]) # 第二步：从李四所在的列表里面，再按下标找到数据李四
```



## list相关函数

1、cmp(list1, list2)比较两个列表的元素			2、len(list)列表元素个数

3、max(list)返回列表元素最大值				    4、min(list)返回列表元素最小值

5、list(seq)将元组转换为列表

# tuple元组

元组特点：定义元组使用==小括号==，且==逗号==隔开各个数据，数据可相同，数据可以是不同的数据类型且不能修改。

``` python
t1 = (10, 20, 30) # 多个数据元组
t2 = (10,) # 单个数据元组
```

> 注意：如果定义的元组只有一个数据，那么这个数据后面也好添加逗号，否则数据类型为唯一的这个数据的数据类型

``` python
t2 = (10,)
print(type(t2))  # tuple

t3 = (20)
print(type(t3))  # int

t4 = ('hello')
print(type(t4))  # str
```



## tuple的常见操作

元组数据不支持修改，只支持查找，具体如下：

- 按下标查找数据

``` python
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1[0])  # aa
```



- index()：查找某个数据，如果数据存在返回对应的下标，否则报错，语法和列表、字符串的index方法相同。

``` python
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1.index('aa'))  # 0
```



- count()：统计某个数据在当前元组出现的次数。

``` python
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(tuple1.count('bb'))  # 2
```



- len()：统计元组中数据的个数。

``` python
tuple1 = ('aa', 'bb', 'cc', 'bb')
print(len(tuple1))  # 4
```

> 注意：元组内的直接数据如果修改则立即报错

``` python
tuple1 = ('aa', 'bb', 'cc', 'bb')
tuple1[0] = 'aaa'
```

> 但是如果元组里面有列表，修改列表里面的数据则是支持的，故自觉很重要。

``` python
tuple2 = (10, 20, ['aa', 'bb', 'cc'], 50, 30)
print(tuple2[2])  # 访问到列表

tuple2[2][0] = 'aaaaa' # 结果：(10, 20, ['aaaaa', 'bb', 'cc'], 50, 30)
print(tuple2)
```



## tuple常用操作方法

能实现一般队列操作

```python
index()函数：查找数据					格式：元组.index('数据')
count()函数：统计某个数据出现的次数		格式：元组.count('数据')
len()函数：统计元组中数据的个数			格式：len(元组)
yield关键字：返回结果并记住当前返回代码位置，下次调用以上次位置开始
```

# set集合

定义：集合是由不重复元素组成的无序容器，不支持下标		

set()创建空集合，{}是用来创建字典的		格式：s2 = set()

```python
s1 = {10, 20, 30, 40, 50}
print(s1)

s2 = {10, 30, 20, 10, 30, 40, 30, 50}
print(s2)

s3 = set('abcdefg')
print(s3)

s4 = set()
print(type(s4))  # set

s5 = {}
print(type(s5))  # dict
```



## 集合常见操作方法

### 增加数据

- add()

``` python
s1 = {10, 20}
s1.add(100)
s1.add(10)
print(s1)  # {100, 10, 20}
```

> 因为集合有去重功能，所以，当向集合内追加的数据是当前集合已有数据的话，则不进行任何操作。

- update(), 追加的数据是序列。

``` python
s1 = {10, 20}
# s1.update(100)  # 报错
s1.update([100, 200])
s1.update('abc')
print(s1)
```

<img src="./assets/image-20190318121424514.png" alt="image-20190318121424514" style="zoom:50%;" />

### 删除数据

- remove()，删除集合中的指定数据，如果数据不存在则报错。

``` python
s1 = {10, 20}

s1.remove(10)
print(s1)

s1.remove(10)  # 报错
print(s1)
```

- discard()，删除集合中的指定数据，如果数据不存在也不会报错。

``` python
s1 = {10, 20}

s1.discard(10)
print(s1)

s1.discard(10)
print(s1)
```

- pop()，随机删除集合中的某个数据，并返回这个数据。

``` python
s1 = {10, 20, 30, 40, 50}

del_num = s1.pop()
print(del_num)
print(s1)
```



### 查找数据

- in：判断数据在集合序列
- not in：判断数据不在集合序列

``` python
s1 = {10, 20, 30, 40, 50}

print(10 in s1)
print(10 not in s1)
```

### 其他操作

False在集合中表示为0，所以0和false只能存在一个

frozenset()冰冻集合函数：定义不能修改，可以将任何容器类型数据进行转化

```python
frozenset({i*2 for i in range(6)}) # 支持推导式
```



## 常用内置方法

```python
add(elem)将元素 elem 添加到集合中
copy()返回原集合的浅拷贝
update(*others)更新集合，添加来自 others 中的所有元素
len(s)返回集合 s 中的元素数量（即 s 的基数）
discard(elem)如果元素 elem 存在于集合中则将其移除
remove(elem)从集合中移除元素 elem，如果 elem 不存在于集合中则会引发 KeyError
pop()从集合中移除并返回任意一个元素，如果集合为空则会引发 KeyError
clear()从集合中移除所有元素							
```

## 集合的运算

| 运算        | 方法                                                     |                                                              |
| ----------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| 交集符：&   | intersection_update(*others)以交集方式更新引用方法的集合 |                                                              |
| 并集：      | union(*others)以并集方式返回一个新集合                   | difference_update(*others)以差集方式更新引用方法的集合       |
| 差集：-     | difference(*others)以差集方式返回一个新集合              |                                                              |
| 对称差集：^ | symmetric_difference(other)以对称差集方式返回一个新集合  | symmetric_difference_update(other)以对称差集方式更新引用方法的集合 |

示例
```python
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union_set = set1.union(set2)
print(union_set)  # 输出: {1, 2, 3, 4, 5}
```

<img src="E:\Project\Textbook\Python\assets\wps10.jpg" alt="img" style="zoom:50%;" /> 

issuperset(other)检测引用方法集合是否为超集，是则返回Ture

issubset(other)检测引用方法集合是否为other集合的子集，是则返回Ture

isdisjoint(other)检测引用方法集合和other集合是否为交集，不相交则返回Ture

intersection(*others)返回一个新集合，其中包含原集合以及 others 指定的所有集合中共有的元素

# dict字典

字典：里面的数据是以==键值对==形式出现，字典数据和数据顺序没有关系，只需要按照对应的键的名字查找数据即可

## 创建字典的语法

字典特点：

- 符号为==大括号==
- 数据为==键值对==形式出现
- 各个键值对之间用==逗号==隔开

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'} # 有数据字典
dict2 = {} # 空字典
dict3 = dict()
```

> 注意：一般称冒号前面的为键(key)，简称k；冒号后面的为值(value)，简称v。

使用字典推导式

```python
{}, {x: x ** 2 for x in range(10)}
```

使用类型构造器

```python
dict(), dict([('foo', 100), ('bar', 200)]), dict(foo=100, bar=200)
```

## 常用操作：

### 增

写法：==字典序列[key] = 值==

> 注意：如果key存在则修改这个key对应的值；如果key不存在则新增此键值对。

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}

dict1['name'] = 'Rose'
print(dict1) # 结果：{'name': 'Rose', 'age': 20, 'gender': '男'}
dict1['id'] = 110
print(dict1) # {'name': 'Rose', 'age': 20, 'gender': '男', 'id': 110}
```

> 注意：字典为可变类型。



### 删

- del() / del：删除字典或删除字典中指定键值对。

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}

del dict1['gender']

print(dict1) # 结果：{'name': 'Tom', 'age': 20}
```

- clear()：清空字典

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
dict1.clear()

print(dict1)  # {}
```



### 改

写法：==字典序列[key] = 值==

> 注意：如果key存在则修改这个key对应的值 ；如果key不存在则新增此键值对。

### 查

#### key值查找

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1['name'])  # Tom
print(dict1['id'])  # 报错
```

> 如果当前查找的key存在，则返回对应的值；否则则报错。



#### get()

- 语法

``` python
字典序列.get(key, 默认值)
```

> 注意：如果当前查找的key不存在则返回第二个参数(默认值)，如果省略第二个参数，则返回None。

- 快速体验

``` python 
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.get('name'))  # Tom
print(dict1.get('id', 110))  # 110
print(dict1.get('id'))  # None
```

#### keys()

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.keys())  # dict_keys(['name', 'age', 'gender'])
```

#### values()

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.values())  # dict_values(['Tom', 20, '男'])
```

#### items()

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
print(dict1.items())  # dict_items([('name', 'Tom'), ('age', 20), ('gender', '男')])
```

## 字典的循环遍历

### 遍历字典的key

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for key in dict1.keys():
    print(key)
```

![image-20190212103905553](./assets/image-20190212103905553.png)

### 遍历字典的value

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for value in dict1.values():
    print(value)
```

![image-20190212103957777](./assets/image-20190212103957777.png)

### 遍历字典的元素

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for item in dict1.items():
    print(item)
    print(type(item)) // 输出为元组类型

输出：
('name', 'Tom')
<class 'tuple'>
('age', 20)
<class 'tuple'>
('gender', '男')
<class 'tuple'>
```

### 遍历字典的键值对

``` python
dict1 = {'name': 'Tom', 'age': 20, 'gender': '男'}
for key, value in dict1.items():
    print(f'{key} = {value}')
```

![image-20190212104223143](./assets/image-20190212104223143.png)



## 常用内置方法

list(d)返回字典 d 中使用的所有键的列表			len(d)返回字典 d 中的项数

keys()返回由字典键组成的一个新视图			values()返回由字典值组成的一个新视图

iter(d)返回字典中键为元素的迭代器 这是 iter(d.keys()) 的快捷方式

popitem()从字典中移除并返回一个 (键, 值) 对 键值对会按 LIFO后进先出顺序被返回

reversed(d)返回一个逆序获取字典键的迭代器 这是 reversed(d.keys()) 的快捷方式

update([other])使用来自 other 的键/值对更新字典，覆盖原有的键 返回 None

update()接受另一个字典对象，或者一个包含键/值对（以长度为二的元组或其他可迭代对象表示）的可迭代对象 如果给出了关键字参数，则会以其所指定的键/值对更新字典: d.update(red=1, blue=2)

setdefault(key[, default])如果字典存在键 key ，返回它的值如果不存在，插入值为 default 的键 key ，并返回 default  default 默认为 None

clear()移除字典中的所有元素					

copy()返回原字典的浅拷贝



# 公共操作

## 一. 运算符

| 运算符 |      描述      |      支持的容器类型      |
| :----: | :------------: | :----------------------: |
|   +    |      合并      |    字符串、列表、元组    |
|   *    |      复制      |    字符串、列表、元组    |
|   in   |  元素是否存在  | 字符串、列表、元组、字典 |
| not in | 元素是否不存在 | 字符串、列表、元组、字典 |

### 1.1 +

``` python
# 1. 字符串 
str1 = 'aa'
str2 = 'bb'
str3 = str1 + str2
print(str3)  # aabb

# 2. 列表 
list1 = [1, 2]
list2 = [10, 20]
list3 = list1 + list2
print(list3)  # [1, 2, 10, 20]

# 3. 元组 
t1 = (1, 2)
t2 = (10, 20)
t3 = t1 + t2
print(t3)  # (1, 2, 10, 20)
```

### 1.2 *

``` python
# 1. 字符串
print('-' * 10)  # ----------

# 2. 列表
list1 = ['hello']
print(list1 * 4)  # ['hello', 'hello', 'hello', 'hello']

# 3. 元组
t1 = ('world',)
print(t1 * 4)  # ('world', 'world', 'world', 'world')
```

### 1.3 in或not in

``` python
# 1. 字符串
print('a' in 'abcd')  # True
print('a' not in 'abcd')  # False

# 2. 列表
list1 = ['a', 'b', 'c', 'd']
print('a' in list1)  # True
print('a' not in list1)  # False

# 3. 元组
t1 = ('a', 'b', 'c', 'd')
print('aa' in t1)  # False
print('aa' not in t1)  # True
```



## 二. 公共方法

| 函数                    | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| len()                   | 计算容器中元素个数                                           |
| del 或 del()            | 删除                                                         |
| max()                   | 返回容器中元素最大值                                         |
| min()                   | 返回容器中元素最小值                                         |
| range(start, end, step) | 生成从start到end的数字，步长为 step，供for循环使用           |
| enumerate()             | 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。 |

### 2.1 len()

``` python
str1 = 'abcdefg' # 1. 字符串
print(len(str1))  # 7

list1 = [10, 20, 30, 40] # 2. 列表
print(len(list1))  # 4

t1 = (10, 20, 30, 40, 50) # 3. 元组
print(len(t1))  # 5

s1 = {10, 20, 30} # 4. 集合
print(len(s1))  # 3

dict1 = {'name': 'Rose', 'age': 18} # 5. 字典
print(len(dict1))  # 2
```

### 2.2 del()

``` python
str1 = 'abcdefg' # 1. 字符串
del str1
print(str1)

list1 = [10, 20, 30, 40] # 2. 列表
del(list1[0])
print(list1)  # [20, 30, 40]
```

### 2.3 max()

``` python
str1 = 'abcdefg' # 1. 字符串
print(max(str1))  # g

list1 = [10, 20, 30, 40] # 2. 列表
print(max(list1))  # 40
```

### 2.4 min()

``` python
str1 = 'abcdefg' # 1. 字符串
print(min(str1))  # a

list1 = [10, 20, 30, 40] # 2. 列表
print(min(list1))  # 10
```

### 2.5 range()

``` python
for i in range(1, 10, 1):
    print(i) # 1 2 3 4 5 6 7 8 9

for i in range(1, 10, 2):
    print(i) # 1 3 5 7 9

for i in range(10):
    print(i) # 0 1 2 3 4 5 6 7 8 9
```

> 注意：range()生成的序列不包含end数字。

### 2.6 enumerate()

将一个可迭代对象（如列表、元组或字符串）组合为一个索引序列，同时返回索引和对应的值。

- 语法

``` python
enumerate(可遍历对象, start=0)
```

> 注意：start参数用来设置遍历数据的下标的起始值，默认为0。

- 快速体验

``` python
list1 = ['a', 'b', 'c', 'd', 'e']

for i in enumerate(list1):
    print(i)

for index, char in enumerate(list1, start=1):
    print(f'下标是{index}, 对应的字符是{char}')
```

<img src="./assets/image-20190213115919040.png" alt="image-20190213115919040" style="zoom: 33%;" />

## 三. 容器类型转换

### 3.1 tuple()

作用：将某个序列转换成元组

``` python
list1 = [10, 20, 30, 40, 50, 20]
s1 = {100, 200, 300, 400, 500}

print(tuple(list1))
print(tuple(s1))
```



### 3.2 list()

作用：将某个序列转换成列表

``` python
t1 = ('a', 'b', 'c', 'd', 'e')
s1 = {100, 200, 300, 400, 500}

print(list(t1))
print(list(s1))
```



### 3.3 set()

作用：将某个序列转换成集合

``` python
list1 = [10, 20, 30, 40, 50, 20]
t1 = ('a', 'b', 'c', 'd', 'e')

print(set(list1))
print(set(t1))
```

> 注意：

 	1. 集合可以快速完成列表去重
 	2. 集合不支持下标

# 推导式

作用：用一个表达式创建一个有规律的列表或控制一个有规律列表。

列表推导式又叫列表生成式。

## 1.1 快速体验

需求：创建一个0-10的列表。

- while循环实现

``` python
list1 = [] # 1. 准备一个空列表

i = 0
while i < 10:
    list1.append(i) # 2. 书写循环，依次追加数字到空列表list1中
    i += 1

print(list1)
```

- for循环实现

``` python
list1 = []
for i in range(10):
    list1.append(i)

print(list1)
```

- 列表推导式实现

``` python 
list1 = [i for i in range(10)]
print(list1)
```

## 1.2 带if的列表推导式

需求：创建0-10的偶数列表

- 方法一：range()步长实现

``` python
list1 = [i for i in range(0, 10, 2)]
print(list1)
```

- 方法二：if实现

``` python
list1 = [i for i in range(10) if i % 2 == 0]
print(list1)
```

## 1.3 多个for循环实现列表推导式

需求：创建列表如下：

``` html
[(1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
```

- 代码如下：

``` python
list1 = [(i, j) for i in range(1, 3) for j in range(3)]
print(list1)
```



## 字典推导式

思考：如果有如下两个列表：

``` python
list1 = ['name', 'age', 'gender']
list2 = ['Tom', 20, 'man']
```

如何快速合并为一个字典？

答：字典推导式

字典推导式作用：快速合并列表为字典或提取字典中目标数据。

## 2.1 快速体验

1. 创建一个字典：字典key是1-5数字，value是这个数字的2次方。

``` python
dict1 = {i: i**2 for i in range(1, 5)}
print(dict1)  # {1: 1, 2: 4, 3: 9, 4: 16}
```

2. 将两个列表合并为一个字典

``` python 
list1 = ['name', 'age', 'gender']
list2 = ['Tom', 20, 'man']

dict1 = {list1[i]: list2[i] for i in range(len(list1))}
print(dict1)
```

3. 提取字典中目标数据

``` python
counts = {'MBP': 268, 'HP': 125, 'DELL': 201, 'Lenovo': 199, 'acer': 99}

# 需求：提取上述电脑数量大于等于200的字典数据
count1 = {key: value for key, value in counts.items() if value >= 200}
print(count1)  # {'MBP': 268, 'DELL': 201}
```



## 集合推导式

需求：创建一个集合，数据为下方列表的2次方。

``` python
list1 = [1, 1, 2]
```

代码如下：

``` python
list1 = [1, 1, 2]
set1 = {i ** 2 for i in list1}
print(set1)  # {1, 4}
```

> 注意：集合有数据去重功能。

# 函数

函数就是一段具有独立功能的代码快整合到一个整体并命名，在需要的位置调用这个名称即可完成对应的需求

1、函数在开发过程中，可以更高效的实现代码重用

2、如果不调用函数，定义函数内的代码将不会被执行

3、当调用函数时，解释器会回到定义函数的地方执行下方缩进的代码，执行完继续回到调用函数的地方

## 定义函数

``` python
def 函数名(参数1,参数2,*args,...): # *args为不定长参数
    代码1
    代码2
    ......
    return # 返回值
```

**调用函数**

``` python
函数名(参数)
```

> 注意：

 	1. 不同的需求，参数可有可无。
 	2. 在Python中，函数必须==先定义后使用==。

## 定义函数的说明文档

``` python
def 函数名(参数):
    """ 说明文档的位置 """
    代码
    ......
```

- 查看函数的说明文档

```python
help(函数名)
```

快速体验

``` python
def sum_num(a, b):
    """ 求和函数 """
    return a + b


help(sum_num)
```

<img src="./assets/image-20190219112749727.png" alt="image-20190219112749727" style="zoom: 50%;" />

## 不定参数

不定长参数也叫可变参数。用于不确定调用的时候会传递多少个参数(不传参也可以)的场景。此时，可用包裹(packing)位置参数，或者包裹关键字参数，来进行参数传递，会显得非常方便。

- 包裹位置传递

``` python
def user_info(*args):
    print(args)

user_info('TOM') # ('TOM',)
user_info('TOM', 18) # ('TOM', 18)
```

> 注意：传进的所有参数都会被args变量收集，它会根据传进参数的位置合并为一个元组(tuple)，args是元组类型，这就是包裹位置传递。

- 包裹关键字传递

``` python
def user_info(**kwargs):
    print(kwargs)

user_info(name='TOM', age=18, id=110) # {'name': 'TOM', 'age': 18, 'id': 110}
```

> 综上：无论是包裹位置传递还是包裹关键字传递，都是一个组包的过程。



## 引用

### 了解引用

在python中，值是靠引用来传递来的。

**我们可以用`id()`来判断两个变量是否为同一个值的引用。** 我们可以将id值理解为那块内存的地址标识。

``` python
# 1. int类型
a = 1
b = a

print(b)  # 1
print(id(a))  # 140708464157520
print(id(b))  # 140708464157520

a = 2
print(b)  # 1,说明int类型为不可变类型 
print(id(a))  # 140708464157552，此时得到是的数据2的内存地址
print(id(b))  # 140708464157520


# 2. 列表
aa = [10, 20]
bb = aa

print(id(aa))  # 2325297783432
print(id(bb))  # 2325297783432


aa.append(30)
print(bb)  # [10, 20, 30], 列表为可变类型
print(id(aa))  # 2325297783432
print(id(bb))  # 2325297783432
```

### 引用当做实参

代码如下：

``` python
def test1(a):
    print(a)
    print(id(a))

    a += a

    print(a)
    print(id(a))


# int：计算前后id值不同
b = 100
test1(b)

# 列表：计算前后id值相同
c = [11, 22]
test1(c)
```

效果图如下：

<img src="./assets/image-20190220111744493.png" alt="image-20190220111744493" style="zoom:50%;" />

### 可变和不可变类型

所谓可变类型与不可变类型是指：数据能够直接进行修改，如果能直接修改那么就是可变，否则是不可变.

- 可变类型
  - 列表
  - 字典
  - 集合
- 不可变类型
  - 整型
  - 浮点型
  - 字符串
  - 元组



## 常用函数

format()格式化函数

根据对象所在的位置：print('{0} and {1}'.format('spam', 'eggs'))

关键字参数名引用值：print('The story of {0}, {1}, and {other}.'.format('Bill', 'Manfred',other='Georg'))

名称引用变量：table = {'Sjoerd': 4127, 'Jack': 4098, 'Dcab': 8637678}

print('Jack: {Jack:d}; Sjoerd: {Sjoerd:d}; Dcab: {Dcab:d}'.format(**table))Jack: 4098; Sjoerd: 4127; Dcab: 8637678

{}数字格式化{:.2f}保留小数点后两位3.14、	{:.0f}不带小数、			{:>10d}右对齐 (默认, 宽度为10)    13

{:+.2f}带符号保留小数点后两位+3.14、		{:.2%}百分比格式25.00%、	{:x<4d}数字补x (填充右边, 宽度为4) 5xxx



## lambda函数

如果一个函数有一个返回值，并且只有一句话代码，可以使用lambda简化

语法：lambda  参数：表达式(返回值)			#直接打印lambda表达式，输出的是此lambda的内存地址

1、 参数可有可无，参数为1、可变参数*args时，默认输出元组

2、不定长可变参数**args时，默认输出字典

3、表达式能接收任意数量的参数但只能返回一个表达式的值

格式：fn1 = lambda  a，b：a + b		print(fn1(1, 2))		输出内容：3



## 可迭代对象常用处理函数

bin(num)将num十进制转换为二进制

zip([iterable, ...])压缩函数，接收多个可迭代对象，后把每个传送对象中第i个元素组成一个新的迭代器(元组类型的)

sorted(iterable, cmp=None, key=function, reverse=False)排序函数对所有可迭代的对象进行排序操作，返回一个新的列表

参数：iterable=(容器型数据、range序列、迭代器)、reverse排序默认为true、key=自定义或内置函数

map(function, iterable, ...)根据提供的函数对传入对象中每个元素进行处理，返回一个新的迭代器

reduce(function, iterable[, initializer])对传入的可迭代对象中的元素进行function函数运算累积

需要引入 functools 模块导入该函数

filter(function, iterable)过滤函数，function返回Ture或false过滤掉不符合条件的元素，返回由符合条件元素组成的新列表



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

## 数据类型转换函数

| 函数           | 描述                                             | 函数                 | 描述                                                |
| -------------- | ------------------------------------------------ | -------------------- | --------------------------------------------------- |
| int(x [,base]) | 将x转换为一个整数                                | eval(str)            | 用来计算在字符串中的有效Python表达式,并返回一个对象 |
| long(x[,base]) | 将x转换为一个长整数                              | repr(x)              | 将对象 x 转换为表达式字符串                         |
| float(x)       | 将x转换到一个浮点数                              | complex(real[,imag]) | 创建一个复数                                        |
| str(x)         | 将对象 x 转换为字符串                            | frozenset(s)         | 转换为不可变集合                                    |
| tuple(s)       | 将序列 s 转换为一个元组                          | unichr(x)            | 将一个整数转换为Unicode字符                         |
| list(s)        | 将序列 s 转换为一个列表                          | ord(x)               | 将一个字符转换为它的整数值                          |
| set(s)         | 转换为可变集合                                   | hex(x)               | 将一个整数转换为一个十六进制字符串                  |
| dict(d)        | 创建一个字典。d 必须是一个序列 (key,value)元组。 | oct(x)               | 将一个整数转换为一个八进制字符串                    |
| chr(x)         | 将一个整数转换为一个字符                         |                      |                                                     |

## 高阶函数

==把函数作为参数传入==，这样的函数称为高阶函数，高阶函数是函数式编程的体现。函数式编程就是指这种高度抽象的编程范式。

在Python中，`abs()`函数可以完成对数字求绝对值计算。

``` python
abs(-10)  # 10
```

`round()`函数可以完成对数字的四舍五入计算。

``` python
round(1.2)  # 1
round(1.9)  # 2
```

需求：任意两个数字，按照指定要求整理数字后再进行求和计算。

- 方法1

``` python
def add_num(a, b):
    return abs(a) + abs(b)


result = add_num(-1, 2)
print(result)  # 3
```

- 方法2

``` python
def sum_num(a, b, f):
    return f(a) + f(b)


result = sum_num(-1, 2, abs)
print(result)  # 3
```

> 注意：两种方法对比之后，发现，方法2的代码会更加简洁，函数灵活性更高。

函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。



### map()

map(func, lst)，将传入的函数变量func作用到lst变量的每个元素中，并将结果组成新的列表(Python2)/迭代器(Python3)返回。

需求：计算`list1`序列中各个数字的2次方。

``` python
list1 = [1, 2, 3, 4, 5]

def func(x):
    return x ** 2

result = map(func, list1)

print(result)  # <map object at 0x0000013769653198>
print(list(result))  # [1, 4, 9, 16, 25]
```



### reduce()

reduce(func，lst)，其中func必须有两个参数。每次func计算的结果继续和序列的下一个元素做累积计算。

> 注意：reduce()传入的参数func必须接收2个参数。

需求：计算`list1`序列中各个数字的累加和。

``` python
import functools

list1 = [1, 2, 3, 4, 5]


def func(a, b):
    return a + b


result = functools.reduce(func, list1)

print(result)  # 15
```



### filter()

filter(func, lst)函数用于过滤序列, 过滤掉不符合条件的元素, 返回一个 filter 对象。如果要转换为列表, 可以使用 list() 来转换。

``` python
list1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

def func(x):
    return x % 2 == 0


result = filter(func, list1)

print(result)  # <filter object at 0x0000017AF9DC3198>
print(list(result))  # [2, 4, 6, 8, 10]
```



# 面向对象OOP

## 面向对象实现方法

### 定义类

在Python 3中定义一个类非常简单。你只需要使用`class`关键字，后面跟着类的名称，然后是一个冒号，接着是类的定义块。这个块中可以包含类的属性（变量）和方法（函数）。

以下是一个简单的示例，展示了如何在Python 3中定义一个类：

```python
class Dog:
    # 类属性
    species = "Canis familiaris"

    # 初始化方法（构造函数）
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # 方法
    def description(self):
        return f"{self.name} is {self.age} years old"

    def speak(self, sound):
        return f"{self.name} says {sound}"
```

在这个示例中，我们定义了一个名为`Dog`的类。这个类有两个属性`species`和`name`，以及两个方法`description()`和`speak()`。

- `species`是一个类属性，对于所有属于`Dog`类的对象来说都是相同的值，可以通过类来访问，例如`Dog.species`。
- `__init__()`方法是一个特殊的方法，也称为构造函数。当你创建一个新的`Dog`对象时，这个方法会被调用。它用于初始化对象的属性。
- `description()`方法返回一个描述狗的字符串。
- `speak()`方法接受一个声音作为参数，并返回包含狗的名称和声音的字符串。

要创建一个`Dog`对象，你可以这样做：

```python
buddy = Dog("Buddy", 3)
```

这将创建一个名为`buddy`的新的`Dog`对象，其名称为`Buddy`，年龄为3岁。然后你可以调用类的方法来操作这个对象：

```python
print(buddy.description())  # 输出: Buddy is 3 years old
print(buddy.speak("Woof Woof"))  # 输出: Buddy says Woof Woof
print(buddy.species)  # 输出: Canis familiaris
```

这就是在Python 3中定义和使用类的基本方法。

### self

self指的是调用该函数的对象。

``` python
# 1. 定义类
class Washer():
    def wash(self):
        print('我会洗衣服')
        # <__main__.Washer object at 0x0000024BA2B34240>
        print(self)


# 2. 创建对象
haier1 = Washer()
# <__main__.Washer object at 0x0000018B7B224240>
print(haier1)
haier1.wash() # haier1对象调用实例方法


haier2 = Washer()
# <__main__.Washer object at 0x0000022005857EF0>
print(haier2)
```

> 注意：打印对象和self得到的结果是一致的，都是当前对象的内存中存储地址。



## 添加和获取对象属性

属性即是特征，比如：洗衣机的宽度、高度、重量...

对象属性既可以在类外面添加和获取，也能在类里面添加和获取。

### 类外面添加对象属性

- 语法

``` python
对象名.属性名 = 值
```

- 体验

``` python
haier1.width = 500
haier1.height = 800
```



### 类外面获取对象属性

- 语法

``` python
对象名.属性名
```

- 体验

``` python
print(f'haier1洗衣机的宽度是{haier1.width}')
print(f'haier1洗衣机的高度是{haier1.height}')
```



### 类里面获取对象属性

- 语法

``` python
self.属性名
```

- 体验

``` python
# 定义类
class Washer():
    def print_info(self):
        # 类里面获取实例属性
        print(f'haier1洗衣机的宽度是{self.width}')
        print(f'haier1洗衣机的高度是{self.height}')

# 创建对象
haier1 = Washer()

# 添加实例属性
haier1.width = 500
haier1.height = 800

haier1.print_info()
```



## 魔法方法

在Python中，`__xx__()`的函数叫做魔法方法，指的是具有特殊功能的函数。

### `__init__()`

#### 体验`__init__()`

思考：洗衣机的宽度高度是与生俱来的属性，可不可以在生产过程中就赋予这些属性呢？

答：理应如此。

==`__init__()`方法的作用：初始化对象。==

``` python
class Washer():
    # 定义初始化功能的函数
    def __init__(self):
        # 添加实例属性
        self.width = 500
        self.height = 800

    def print_info(self):
        # 类里面调用实例属性
        print(f'洗衣机的宽度是{self.width}, 高度是{self.height}')


haier1 = Washer()
haier1.print_info()
```

> 注意：
>
> - `__init__()`方法，在创建一个对象时默认被调用，不需要手动调用
> - `__init__(self)`中的self参数，不需要开发者传递，python解释器会自动把当前的对象引用传递过去。



#### 带参数的`__init__()`

思考：一个类可以创建多个对象，如何对不同的对象设置不同的初始化属性呢？

答：传参数。

``` python
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def print_info(self):
        print(f'洗衣机的宽度是{self.width}')
        print(f'洗衣机的高度是{self.height}')


haier1 = Washer(10, 20)
haier1.print_info()

haier2 = Washer(30, 40)
haier2.print_info()
```



### `__str__()`

当使用print输出对象的时候，默认打印对象的内存地址。如果类定义了`__str__`方法，那么就会打印从在这个方法中 return 的数据。

``` python
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __str__(self):
        return '这是海尔洗衣机的说明书'


haier1 = Washer(10, 20)
print(haier1) # 这是海尔洗衣机的说明书
```



### `__del__()`

当删除对象时，python解释器也会默认调用`__del__()`方法。

``` python
class Washer():
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def __del__(self):
        print(f'{self}对象已经被删除')


haier1 = Washer(10, 20)
del haier1 # <__main__.Washer object at 0x0000026118223278>对象已经被删除
```

# 面向对象-继承

## 一. 继承的概念

生活中的继承，一般指的是子女继承父辈的财产。

![](./assets/1.png)

- 拓展1：经典类或旧式类

不由任意内置类型派生出的类，称之为经典类。

```python
class 类名:
    代码
    ......
```

- 拓展2：新式类

```python
class 类名(object):
	代码
```



Python面向对象的继承指的是多个类之间的所属关系，即子类默认继承父类的所有属性和方法，具体如下：

``` python
# 父类A
class A(object):
    def __init__(self):
        self.num = 1

    def info_print(self):
        print(self.num)

# 子类B
class B(A):
    pass


result = B()
result.info_print()  # 1
```

> 在Python中，所有类默认继承object类，object类是顶级类或基类；其他子类叫做派生类。



## 二. 单继承

> 故事主线：一个煎饼果子老师傅，在煎饼果子界摸爬滚打多年，研发了一套精湛的摊煎饼果子的技术。师父要把这套技术传授给他的唯一的最得意的徒弟。

分析：徒弟是不是要继承师父的所有技术？

``` python
# 1. 师父类
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')
        
# 2. 徒弟类
class Prentice(Master):
    pass


daqiu = Prentice() # 3. 创建对象daqiu
print(daqiu.kongfu) # 4. 对象访问实例属性
daqiu.make_cake() # 5. 对象调用实例方法
```



## 三. 多继承

> 故事推进：daqiu是个爱学习的好孩子，想学习更多的煎饼果子技术，于是，在百度搜索到黑马程序员，报班学习煎饼果子技术。

所谓多继承意思就是一个类同时继承了多个父类。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


# 创建学校类
class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    pass


daqiu = Prentice()
print(daqiu.kongfu) # [黑马煎饼果子配方]
daqiu.make_cake() # 运用[黑马煎饼果子配方]制作煎饼果子
```

> 注意：当一个类有多个父类的时候，默认使用第一个父类的同名属性和方法。



## 四. 子类重写父类同名方法和属性

> 故事：daqiu掌握了师父和培训的技术后，自己潜心钻研出自己的独门配方的一套全新的煎饼果子技术。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


# 独创配方
class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


daqiu = Prentice()
print(daqiu.kongfu) # [独创煎饼果子配方]
daqiu.make_cake() # 运用[独创煎饼果子配方]制作煎饼果子
print(Prentice.__mro__) # (<class '__main__.Prentice'>, <class '__main__.School'>, <class '__main__.Master'>, <class 'object'>)
```

> 子类和父类具有同名属性和方法，默认使用子类的同名属性和方法。



## 五. 子类调用父类的同名方法和属性

> 故事：很多顾客都希望也能吃到古法和黑马的技术的煎饼果子。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        # 如果是先调用了父类的属性和方法，父类属性会覆盖子类属性，故在调用属性前，先调用自己子类的初始化
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    # 调用父类方法，但是为保证调用到的也是父类的属性，必须在调用方法前调用父类的初始化
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


daqiu = Prentice()
daqiu.make_cake()  # 运用[独创煎饼果子配方]制作煎饼果子
daqiu.make_master_cake()  # 运用[古法煎饼果子配方]制作煎饼果子
daqiu.make_school_cake()  # 运用[黑马煎饼果子配方]制作煎饼果子
daqiu.make_cake()  # 运用[独创煎饼果子配方]制作煎饼果子
```



## 六. 多层继承

> 故事：N年后，daqiu老了，想要把所有技术传承给自己的徒弟。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


# 徒孙类
class Tusun(Prentice):
    pass


xiaoqiu = Tusun()
xiaoqiu.make_cake()
xiaoqiu.make_school_cake()
xiaoqiu.make_master_cake()
```



## 七. super()调用父类方法

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(Master):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')

        # 方法2.1
        # super(School, self).__init__()
        # super(School, self).make_cake()

        # 方法2.2
        super().__init__()
        super().make_cake()


class Prentice(School):
    def __init__(self):
        self.kongfu = '[独创煎饼果子技术]'

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    # 子类调用父类的同名方法和属性：把父类的同名属性和方法再次封装
    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)

    # 一次性调用父类的同名属性和方法
    def make_old_cake(self):
        # 方法一：代码冗余；父类类名如果变化，这里代码需要频繁修改
        # Master.__init__(self)
        # Master.make_cake(self)
        # School.__init__(self)
        # School.make_cake(self)

        # 方法二: super()
        # 方法2.1 super(当前类名, self).函数()
        # super(Prentice, self).__init__()
        # super(Prentice, self).make_cake()

        # 方法2.2 super().函数()
        super().__init__()
        super().make_cake()


daqiu = Prentice()  
daqiu.make_old_cake()  # 运用[黑马煎饼果子配方]制作煎饼果子 # 运用[古法煎饼果子配方]制作煎饼果子
```

> 注意：使用super() 可以自动查找父类。调用顺序遵循 `__mro__` 类属性的顺序。比较适合单继承使用。



## 八. 私有权限

### 8.1 定义私有属性和方法

在Python中，可以为实例属性和方法设置私有权限，即设置某个实例属性或实例方法不继承给子类。

> 故事：daqiu把技术传承给徒弟的同时，不想把自己的钱(2000000个亿)继承给徒弟，这个时候就要为`钱`这个实例属性设置私有权限。

设置私有权限的方法：在`属性名`和`方法名` 前面 加上两个下划线 __。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        # 定义私有属性
        self.__money = 2000000

    # 定义私有方法
    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


# 徒孙类
class Tusun(Prentice):
    pass


daqiu = Prentice()
# 对象不能访问私有属性和私有方法
# print(daqiu.__money)
# daqiu.__info_print()

xiaoqiu = Tusun()
# 子类无法继承父类的私有属性和私有方法
# print(xiaoqiu.__money)  # 无法访问实例属性__money
# xiaoqiu.__info_print()
```

> 注意：私有属性和私有方法只能在类里面访问和修改。

### 8.2 获取和修改私有属性值

在Python中，一般定义函数名`get_xx`用来获取私有属性，定义`set_xx`用来修改私有属性值。

``` python
class Master(object):
    def __init__(self):
        self.kongfu = '[古法煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class School(object):
    def __init__(self):
        self.kongfu = '[黑马煎饼果子配方]'

    def make_cake(self):
        print(f'运用{self.kongfu}制作煎饼果子')


class Prentice(School, Master):
    def __init__(self):
        self.kongfu = '[独创煎饼果子配方]'
        self.__money = 2000000

    # 获取私有属性
    def get_money(self):
        return self.__money

    # 修改私有属性
    def set_money(self):
        self.__money = 500

    def __info_print(self):
        print(self.kongfu)
        print(self.__money)

    def make_cake(self):
        self.__init__()
        print(f'运用{self.kongfu}制作煎饼果子')

    def make_master_cake(self):
        Master.__init__(self)
        Master.make_cake(self)

    def make_school_cake(self):
        School.__init__(self)
        School.make_cake(self)


# 徒孙类
class Tusun(Prentice):
    pass


daqiu = Prentice()
xiaoqiu = Tusun()

print(xiaoqiu.get_money()) # 调用get_money函数获取私有属性money的值
xiaoqiu.set_money() 
print(xiaoqiu.get_money()) # 调用set_money函数修改私有属性money的值
```

 

# 面向对象-三大特性

封装：public公有的（默认）、protected受保护（_属性/方法）、prviate私有的（_ _属性/方法）

<img src="E:\Project\Textbook\Python\assets\wps12.jpg" alt="img" style="zoom: 67%;" />

## 一. 面向对象三大特性

- 继承
  - 子类默认继承父类的所有属性和方法
  - 子类可以重写父类属性和方法
- 封装
  - 将属性和方法书写到类的里面的操作即为封装
  - 封装可以为属性和方法添加私有权限
- 多态
  - 传入不同的对象，产生不同的结果

## 二. 多态

### 2.1 了解多态

多态指的是一类事物有多种形态，（一个抽象类有多个子类，因而多态的概念依赖于继承）。

- 定义：多态是一种使用对象的方式，子类重写父类方法，调用不同子类对象的相同父类方法，可以产生不同的执行结果
- 好处：调用灵活，有了多态，更容易编写出通用的代码，做出通用的编程，以适应需求的不断变化！
- 实现步骤：
  - 定义父类，并提供公共方法
  - 定义子类，并重写父类方法
  - 传递子类对象给调用者，可以看到不同子类执行效果不同

### 2.2 体验多态

``` python
class Dog(object):
    def work(self):  # 父类提供统一的方法，哪怕是空方法
        print('指哪打哪...')


class ArmyDog(Dog):  # 继承Dog类
    def work(self):  # 子类重写父类同名方法
        print('追击敌人...')


class DrugDog(Dog):
    def work(self):
        print('追查毒品...')


class Person(object):
    def work_with_dog(self, dog):  # 传入不同的对象，执行不同的代码，即不同的work函数
        dog.work()


ad = ArmyDog()
dd = DrugDog()

daqiu = Person()
daqiu.work_with_dog(ad)
daqiu.work_with_dog(dd)
```



## 三. 类属性和实例属性

### 3.1 类属性

#### 3.1.1 设置和访问类属性

- 类属性就是 **类对象** 所拥有的属性，它被 **该类的所有实例对象 所共有**。
- 类属性可以使用 **类对象** 或 **实例对象** 访问。

``` python
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()

print(Dog.tooth)  # 10
print(wangcai.tooth)  # 10
print(xiaohei.tooth)  # 10
```

> 类属性的优点
>
> - **记录的某项数据 始终保持一致时**，则定义类属性。
> - **实例属性** 要求 **每个对象** 为其 **单独开辟一份内存空间** 来记录数据，而 **类属性** 为全类所共有 ，**仅占用一份内存**，**更加节省内存空间**。



#### 3.1.2 修改类属性

类属性只能通过类对象修改，不能通过实例对象修改，如果通过实例对象修改类属性，表示的是创建了一个实例属性。

``` python
class Dog(object):
    tooth = 10


wangcai = Dog()
xiaohei = Dog()

# 修改类属性
Dog.tooth = 12
print(Dog.tooth)  # 12
print(wangcai.tooth)  # 12
print(xiaohei.tooth)  # 12

# 不能通过对象修改属性，如果这样操作，实则是创建了一个实例属性
wangcai.tooth = 20
print(Dog.tooth)  # 12
print(wangcai.tooth)  # 20
print(xiaohei.tooth)  # 12
```



### 3.2 实例属性

``` python
class Dog(object):
    def __init__(self):
        self.age = 5

    def info_print(self):
        print(self.age)


wangcai = Dog()
print(wangcai.age)  # 5
# print(Dog.age)  # 报错：实例属性不能通过类访问
wangcai.info_print()  # 5
```



## 四. 类方法和静态方法

### 4.1 类方法

#### 4.1.1 类方法特点

- 需要用装饰器`@classmethod`来标识其为类方法，对于类方法，**第一个参数必须是类对象**，一般以`cls`作为第一个参数。

**使用场景**

- 当方法中 **需要使用类对象** (如访问私有类属性等)时，定义类方法
- 类方法一般和类属性配合使用

``` python
class Dog(object):
    __tooth = 10

    @classmethod
    def get_tooth(cls): # 类方法
        return cls.__tooth
	def info_print(self): # 示例方法
        print(self.age)

wangcai = Dog()
result = wangcai.get_tooth()
print(result)  # 10
```

#### 实例方法和类方法的区别

1. 实例方法：这些方法的第一个参数是实例对象本身（通常被命名为`self`），它们可以访问对象的属性，并且通常用于操作特定的实例。
2. 类方法：这些方法使用`@classmethod`装饰器进行修饰，第一个参数是类对象本身（通常被命名为`cls`），它们可以访问类的属性，并且通常用于执行与整个类相关的操作。

### 4.2 静态方法

#### 4.2.1 静态方法特点

- 需要通过装饰器`@staticmethod`来进行修饰，**静态方法既不需要传递类对象也不需要传递实例对象（形参没有self/cls）**。
- 静态方法 也能够通过 **实例对象** 和 **类对象** 去访问。

**静态方法使用场景**

- 当方法中 **既不需要使用实例对象**(如实例对象，实例属性)，**也不需要使用类对象** (如类属性、类方法、创建实例等)时，定义静态方法
- **取消不需要的参数传递**，有利于 **减少不必要的内存占用和性能消耗**

``` python
class Dog(object):
    @staticmethod
    def info_print():
        print('这是一个狗类，用于创建狗实例....')


wangcai = Dog()

wangcai.info_print() # 静态方法既可以使用对象访问又可以使用类访问
Dog.info_print()
```



## 类的常用方法：

issubclass(A，B)方法：检测A类是否继承自B类，返回Ture或False

isinstance(A，B) 方法：检测A对象是否为B类的实例化对象

hasattr(D，'属性') 方法： 检测D 类/对象是否包含指定名称的属性/方法，返回Ture或False

getattr(D，'成员') 方法：获取D类/对象成员的属性值

setattr(D，'成员'，‘属性值’) 方法：设置D类/对象的成员的属性值

delattr(D，‘属性’) 方法：删除D类/对象的属性

# 异常

## 一. 了解异常

当检测到一个错误时，解释器就无法继续执行了，反而出现了一些错误的提示，这就是所谓的"异常"。

例如：以`r`方式打开一个不存在的文件。

``` python
open('test.txt', 'r')
```

<img src="./assets/image-20190305154200725.png" alt="image-20190305154200725" style="zoom: 50%;" />



## 二. 异常的写法

### 2.1 语法

``` python
try:
    可能发生错误的代码
except:
    如果出现异常执行的代码
```

### 2.2 快速体验

需求：尝试以`r`模式打开文件，如果文件不存在，则以`w`方式打开。

``` python
try:
    f = open('test.txt', 'r')
except:
    f = open('test.txt', 'w')
```

### 2.3 捕获指定异常

#### 2.3.1 语法

``` python
try:
    可能发生错误的代码
except 异常类型:
    如果捕获到该异常类型执行的代码
```

#### 2.3.2 体验

``` python
try:
    print(num)
except NameError:
    print('有错误')
```

> 注意：
>
> 1. 如果尝试执行的代码的异常类型和要捕获的异常类型不一致，则无法捕获异常。
> 2. 一般try下方只放一行尝试执行的代码。

#### 2.3.3 捕获多个指定异常

当捕获多个异常时，可以把要捕获的异常类型的名字，放到except 后，并使用元组的方式进行书写。

``` python
try:
    print(1/0)

except (NameError, ZeroDivisionError):
    print('有错误')
```

#### 2.3.4 捕获异常描述信息

``` python
try:
    print(num)
except (NameError, ZeroDivisionError) as result:
    print(result)
```

#### 2.3.5 捕获所有异常

Exception是所有程序异常类的父类。

``` python
try:
    print(num)
except Exception as result:
    print(result)
```



### 2.4 异常的else

else表示的是如果没有异常要执行的代码。

``` python
try:
    print(1)
except Exception as result:
    print(result)
else:
    print('我是else，是没有异常的时候执行的代码')
```

### 2.5 异常的finally

finally表示的是无论是否异常都要执行的代码，例如关闭文件。

``` python
try:
    f = open('test.txt', 'r')
except Exception as result:
    f = open('test.txt', 'w')
else:
    print('没有异常，真开心')
finally:
    f.close()
```

## 三. 异常的传递

体验异常传递

需求：1、尝试只读方式打开test.txt文件，如果文件存在则读取文件内容，文件不存在则提示用户即可。

​		2、读取内容要求：尝试循环读取内容，读取过程中如果检测到用户意外终止程序，则`except`捕获异常并提示用户。

``` python
import time
try:
    f = open('test.txt')
    try:
        while True:
            content = f.readline()
            if len(content) == 0:
                break
            time.sleep(2)
            print(content)
    except:
        # 如果在读取文件的过程中，产生了异常，那么就会捕获到
        # 比如 按下了 ctrl+c
        print('意外终止了读取数据')
    finally:
        f.close()
        print('关闭文件')
except:
    print("没有这个文件")
```

## 四. 自定义异常

在Python中，抛出自定义异常的语法为` raise 异常类对象`。

需求：密码长度不足，则报异常（用户输入密码，如果输入的长度不足3位，则报错，即抛出自定义异常，并捕获该异常）。

``` python
# 自定义异常类，继承Exception
class ShortInputError(Exception):
    def __init__(self, length, min_len):
        self.length = length
        self.min_len = min_len

    # 设置抛出异常的描述信息
    def __str__(self):
        return f'你输入的长度是{self.length}, 不能少于{self.min_len}个字符'


def main():
    try:
        con = input('请输入密码：')
        if len(con) < 3:
            raise ShortInputError(len(con), 3)
    except Exception as result:
        print(result)
    else:
        print('密码已经输入完成')

main()
```



# 模块

Python 模块(Module)，是一个 Python 文件，以 .py 结尾，包含了 Python 对象定义和Python语句。

模块能定义函数，类和变量，模块里也能包含可执行的代码。

## 1.1. 导入模块

### 1.1.1 导入模块的方式

- import 模块名
- from 模块名 import 功能名
- from 模块名 import *
- import 模块名 as 别名
- from 模块名 import 功能名 as 别名

### 1.1.2 导入方式详解

#### 1.1.2.1 import

- 语法

``` python
# 1. 导入模块
import 模块名
import 模块名1, 模块名2...

# 2. 调用功能
模块名.功能名()
```

- 体验

``` python
import math
print(math.sqrt(9))  # 3.0
```

#### 1.1.2.2 from..import..

- 语法

``` python
from 模块名 import 功能1, 功能2, 功能3...
```

- 体验

``` python
from math import sqrt
print(sqrt(9))
```



#### 1.1.2.3 from .. import *

- 语法

``` python
from 模块名 import *
```

- 体验

``` python
from math import *
print(sqrt(9))
```

#### 1.1.2.4 as定义别名

- 语法

``` python
# 模块定义别名
import 模块名 as 别名

# 功能定义别名
from 模块名 import 功能 as 别名
```

- 体验

``` python
# 模块别名
import time as tt

tt.sleep(2)
print('hello')

# 功能别名
from time import sleep as sl
sl(2)
print('hello')
```



## 1.2. 制作模块

在Python中，每个Python文件都可以作为一个模块，模块的名字就是文件的名字。**也就是说自定义模块名必须要符合标识符命名规则。**

### 1.2.1 定义模块

新建一个Python文件，命名为`my_module1.py`，并定义`testA`函数。

``` python
def testA(a, b):
    print(a + b)
```



### 1.2.2 测试模块

在实际开中，当一个开发人员编写完一个模块后，为了让模块能够在项目中达到想要的效果，这个开发人员会自行在py文件中添加一些测试信息.，例如，在`my_module1.py`文件中添加测试代码。

``` python
def testA(a, b):
    print(a + b)


testA(1, 1)
```

此时，无论是当前文件，还是其他已经导入了该模块的文件，在运行的时候都会自动执行`testA`函数的调用。

解决办法如下：

``` python
def testA(a, b):
    print(a + b)

# 只在当前文件中调用该函数，其他导入的文件内不符合该条件，则不执行testA函数调用
if __name__ == '__main__':
    testA(1, 1)
```



### 1.2.3 调用模块

```python
import my_module1
my_module1.testA(1, 1)
```



### 1.2.4 注意事项

如果使用`from .. import ..`或`from .. import *`导入多个模块的时候，且模块内有同名功能。当调用这个同名功能的时候，调用到的是后面导入的模块的功能。

- 体验

``` python
# 模块1代码
def my_test(a, b):
    print(a + b)

# 模块2代码
def my_test(a, b):
    print(a - b)
   
# 导入模块和调用功能代码
from my_module1 import my_test
from my_module2 import my_test

# my_test函数是模块2中的函数
my_test(1, 1)
```



## 1.3. 模块定位顺序

当导入一个模块，Python解析器对模块位置的搜索顺序是：

1. 当前目录
2. 如果不在当前目录，Python则搜索在shell变量PYTHONPATH下的每个目录。
3. 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/

模块搜索路径存储在system模块的sys.path变量中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。

- 注意
  - 自己的文件名不要和已有模块名重复，否则导致模块功能无法使用
  - `使用from 模块名 import 功能`的时候，如果功能名字重复，调用到的是最后定义或导入的功能。



## 1.4. `__all__`

如果一个模块文件中有`__all__`变量，当使用`from xxx import *`导入时，只能导入这个列表中的元素。

- my_module1模块代码

``` python
__all__ = ['testA']


def testA():
    print('testA')


def testB():
    print('testB')
```

- 导入模块的文件代码

``` python
from my_module1 import *
testA()
testB()
```

<img src="./assets/image-20190305175727272.png" alt="image-20190305175727272" style="zoom:50%;" />



# 包

包将有联系的模块组织在一起，即放到同一个文件夹下，并且在这个文件夹创建一个名字为`__init__.py` 文件，那么这个文件夹就称之为包。

## 2.1 制作包

[New] — [Python Package] — 输入包名 — [OK] — 新建功能模块(有联系的模块)。

注意：新建包后，包内部会自动创建`__init__.py`文件，这个文件控制着包的导入行为。

### 2.1.1 快速体验

1. 新建包`mypackage`
2. 新建包内模块：`my_module1` 和 `my_module2`
3. 模块内代码如下

``` python
# my_module1
print(1)


def info_print1():
    print('my_module1')
```

``` python
# my_module2
print(2)


def info_print2():
    print('my_module2')
```



## 2.2 导入包

### 2.2.1 方法一

``` python
import 包名.模块名

包名.模块名.目标
```

#### 2.2.1.1 体验

``` python
import my_package.my_module1

my_package.my_module1.info_print1()
```

### 2.2.2 方法二

注意：必须在`__init__.py`文件中添加`__all__ = []`，控制允许导入的模块列表。

``` python
from 包名 import *
模块名.目标
```

#### 2.2.2.1 体验

``` python
from my_package import *

my_module1.info_print1()
```

 
