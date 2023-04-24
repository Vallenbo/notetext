os打开一个指定路径的文件，返回一个文件对象，【可对文件对象做（r,w,close操作），或者使用bufio库等进行操作】
`os.Open(`文件地址`) `返回一个文件对象    

```go
func Open(name string) (*File, error){ } // 打开一个文件,支持使用绝对路径
```



## 1、读取：

（1、`fileObj.read()`  传统的文件读取方式 

```go
Func (f *File) Read(b []byte) (n int, err error)
```

（2、bufio缓冲读取文件：是在file的基础上封装了一层API，支持更多的功能

```go
bufio.NewReader(fileObj) //创建缓存区从终端去读取数据，指定读取截止符
```

（3、`os.ReadFile`方法能够读取完整的文件，只需要将文件名作为参数传入

（4、`*File.close()`方法能够关闭文件

```go
func (f *File) Read(b []byte) (n int, err error) //读取文件
```



## 2、写入：

（1、`os.OpenFile()`函数能够以指定模式打开文件，返回一个文件对象

```go
func OpenFile(name string, flag int, perm FileMode) (*File, error) {...}  
//name：要打开的文件名     flag：打开文件的模式
```

FileMode参数为linux下文件权限     perm：文件权限，一个八进制数。r（读）04，w（写）02，x（执行）01
模式有以下几种：

| 模式        | 含义     | 模式        | 含义 |
| ----------- | -------- | ----------- | ---- |
| os.O_WRONLY | 只写     | os.O_RDWR   | 读写 |
| os.O_CREATE | 创建文件 | os.O_TRUNC  | 清空 |
| os.O_RDONLY | 只读     | os.O_APPEND | 追加 |

*File文件对象的函数：

```go
func (f *File) Write(b []byte) (n int, err error) { } //参数为字节切片数据
func (f *File) WriteString(s string) (n int, err error) { } //参数为字符串数据
```

```
func bufio.NewWriter(w io.Writer) *Writer { }             //返回一个可写的文件操作对象fileObj
func (b *Writer) WriteString(s string) (int, error) { } //将数据先写入缓存

fileObj.Flush() //将缓存中的内容写入文件

func WriteFile(name string, data []byte, perm FileMode) error //写入文件将数据写入命名文件，并在必要时创建它。
```



```go
func (f *File) Seek(offset int64, whence int) (ret int64, err error) //Seek设置当前读/写位置
```

​	offset为相对偏移量（1字节为单位）
​	whence决定相对位置：0为相对文件开头，1为相对当前位置，2为相对文件结尾
​	ret返回新的偏移量（相对开头）和可能的错误

```go
func Rename(oldpath, newpath string) error  //Rename修改一个文件的名字，移动一个文件。可能会有一些个操作系统特定的限制
```

```go
os.Stdin //从键盘标准输入，当从键盘输入的内容包含（空格时）可用
os.Stdout //输出到终端
```



## 进程信息

`os.Args`是一个存储命令行参数的字符串切片，它的第一个元素是执行文件的名称

```
func Getenv(key string) string // Getenv检索并返回名为指定key的环境变量值。如果不存在该环境变量会返回空字符串。
func Setenv(key, value string) error // Setenv设置名为key的环境变量。如果出错会返回该错误。
```

