**文件是什么？**

计算机中的文件是存储在外部介质（通常是磁盘）上的数据集合，文件分为文本文件和二进制文件。

# 打开和关闭文件

os打开一个指定路径的文件，返回一个文件对象，【可对文件对象做（r,w,close操作），或者使用bufio库等进行操作】
`os.Open`(文件地址)返回一个文件对象

```go
func Open(name string) (*File, error){ } // 打开一个文件,支持使用绝对路径
```

`*File.close()`方法能够关闭文件

```go
func main() {
	file, err := os.Open("./main.go") // 只读方式打开当前目录下的main.go文件
	if err != nil {
		fmt.Println("open file failed!, err:", err)
		return
	}
	defer file.Close() //为了防止文件忘记关闭，我们通常使用defer注册文件关闭语句。
}
```

# 一、读取文件：

`path` 包里包含一个子包叫 `filepath`，这个子包提供了跨平台的函数，用于处理文件名和路径。例如 `Base()` 函数用于获得路径中的最后一个元素（不包含后面的分隔符）：

```go
import "path/filepath"
filename := filepath.Base(path)
```

## file.read按字节读取

Read方法定义如下：

```go
Func (f *File) Read(b []byte) (n int, err error) //传统的文件读取方式 
```
它接收一个字节切片，返回读取的字节数和可能的具体错误，读到文件末尾时会返回0和io.EOF。


```go
func main() {
	file, err := os.Open("./main.go") // 只读方式打开当前目录下的main.go文件
	if err != nil {
		fmt.Println("open file failed!, err:", err)
		return
	}
	defer file.Close()
	
	var content []byte
	var tmp = make([]byte, 128)
	for { //使用for循环读取文件中的所有数据。
		n, err := file.Read(tmp) // 使用Read方法读取数据
		if err == io.EOF {
			fmt.Println("文件读完了")
			break
		}
		if err != nil {
			fmt.Println("read file failed, err:", err)
			return
		}
		content = append(content, tmp[:n]...)
	}
	fmt.Println(string(content))
}
```

## fmt.Fscanln按列读取文件中的数据

如果数据是按列排列并用空格分隔的，你可以使用 `fmt` 包提供的以 `FScan...` 开头的一系列函数来读取他们。请看以下程序，我们将 3 列的数据分别读入变量 `v1`、`v2` 和 `v3` 内，然后分别把他们添加到切片的尾部。

```go
func main() {
    file, err := os.Open("products2.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close()

    var col1, col2, col3 []string
    var v1, v2, v3 string
    for {
        _, err := fmt.Fscanln(file, &v1, &v2, &v3)
        if err != nil { // scans until newline
            break
        }
        col1 = append(col1, v1)
        col2 = append(col2, v2)
        col3 = append(col3, v3)
    }

    fmt.Println(col1) //[ABC FUNC GO]
    fmt.Println(col2) //[40 56 45]
    fmt.Println(col3) //[150 280 356]
}
```



## bufio.NewReader带缓冲读取文件

在file的基础上封装了一层API，支持更多的功能

```go
func NewReader(rd io.Reader) *Reader //创建缓存区从终端去读取数据，指定读取截止符
```

```go
func main() {
	file, err := os.Open("./xx.txt")
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()
	reader := bufio.NewReader(file)
	for {
		line, err := reader.ReadString('\n') //注意是字符
		if err == io.EOF {
			if len(line) != 0 {
				fmt.Println(line)
			}
			fmt.Println("文件读完了")
			break
		}
		if err != nil {
			fmt.Println("read file failed, err:", err)
			return
		}
		fmt.Print(line)
	}
}
```

## os.ReadFile读取整个文件

能够读取完整的文件，只需要将文件名作为参数传入

```sh
func ReadFile(name string) ([]byte, error)
```

```go
func main() {
	content, err := os.ReadFile("./main.go")
	if err != nil {
		fmt.Println("read file failed, err:", err)
		return
	}
	fmt.Println(string(content))
}
```

## 读取用户的输入

### Scan和Sscanf函数

从键盘和标准输入 `os.Stdin` 读取输入，最简单的办法是使用 `fmt` 包提供的 `Scan...` 和 `Sscan...` 开头的函数。

```go
var (
   firstName, lastName, s string
   i int
   f float32
   input = "56.12 / 5212 / Go"
   format = "%f / %d / %s"
)

func main() {
   fmt.Println("Please enter your full name: ")
   fmt.Scanln(&firstName, &lastName)
   // fmt.Scanf("%s %s", &firstName, &lastName)
   fmt.Printf("Hi %s %s!\n", firstName, lastName) // Hi Chris Naegels
   fmt.Sscanf(input, format, &f, &i, &s)
   fmt.Println("From the string we read: ", f, i, s) // 输出结果: From the string we read: 56.12 5212 Go
}
```

`Scanln()` 扫描来自标准输入的文本，将空格分隔的值依次存放到后续的参数内，直到碰到换行。`Scanf()` 与其类似，除了 `Scanf()` 的第一个参数用作格式字符串，用来决定如何读取。`Sscan...` 和以 `Sscan...` 开头的函数则是从字符串读取，除此之外，与 `Scanf()` 相同。如果这些函数读取到的结果与您预想的不同，您可以检查成功读入数据的个数和返回的错误。

### bufio缓冲读取器

也可以使用 `bufio` 包提供的缓冲读取器 (buffered reader) 来读取数据，

```go
func main() {
    inputReader = bufio.NewReader(os.Stdin)
    fmt.Println("Please enter some input: ")
    input, err = inputReader.ReadString('\n')  // version 1
    if err == nil {
        fmt.Printf("The input was: %s\n", input)
    }
    
    switch input {  // version 2 
    case "Philip\r\n":  fallthrough
    case "Ivo\r\n":     fallthrough
    case "Chris\r\n":   fmt.Printf("Welcome %s\n", input)
    default: fmt.Printf("You are not welcome here! Goodbye!\n")
    }

    switch input { // version 3
    case "Philip\r\n", "Ivo\r\n":   fmt.Printf("Welcome %s\n", input)
    default: fmt.Printf("You are not welcome here! Goodbye!\n")
    }
}
```



# 二、写入文件：

## os.OpenFile打开文件的方式

`os.OpenFile()`能够以指定模式打开文件，返回一个文件对象

```go
func OpenFile(name string, flag int, perm FileMode) (*File, error) {...}  
//name：要打开的文件名     flag：打开文件的模式
```

FileMode参数为linux下文件权限
perm：文件权限（一般为0666），一个八进制数。r（读）04，w（写）02，x（执行）01
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

## Write和WriteString字节数组写入文件

```go
func main() {
	file, err := os.OpenFile("xx.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()

	str := "hello 沙河"
	file.Write([]byte(str))       //写入字节切片数据
	file.WriteString("hello 小王子") //直接写入字符串数据
}
```

## bufio.NewWriter缓冲区


```go
func bufio.NewWriter(w io.Writer) *Writer { }           //返回一个可写的文件操作对象fileObj
func (b *Writer) WriteString(s string) (int, error) { } //将数据先写入缓存

fileObj.Flush() //将缓存中的内容写入文件

func WriteFile(name string, data []byte, perm FileMode) error //写入文件将数据写入命名文件，并在必要时创建它。
```

```go
func main() {
	file, err := os.OpenFile("xx.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()
	writer := bufio.NewWriter(file)
	for i := 0; i < 10; i++ {
		writer.WriteString("hello沙河\n") //将数据先写入缓存
	}
	writer.Flush() //将缓存中的内容写入文件
}
```

## ioutil.WriteFile

```go
func main() {
	str := "hello 沙河"
	err := ioutil.WriteFile("./xx.txt", []byte(str), 0666)
	if err != nil {
		fmt.Println("write file failed, err:", err)
		return
	}
}
```

## io.Copy文件拷贝

如何拷贝一个文件到另一个文件？最简单的方式就是使用 `io` 包：

```go
func main() {
	CopyFile("target.txt", "source.txt")
	fmt.Println("Copy done!")
}

func CopyFile(dstName, srcName string) (written int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		return
	}
	defer src.Close()

	dst, err := os.Create(dstName)
	if err != nil {
		return
	}
	defer dst.Close()

	return io.Copy(dst, src)
}
```

## Seek()定位


```go
func (f *File) Seek(offset int64, whence int) (ret int64, err error) //Seek设置当前读/写位置
```

- offset为相对偏移量（1字节为单位）
- whence决定相对位置：0为相对文件开头，1为相对当前位置，2为相对文件结尾
- ret返回新的偏移量（相对开头）和可能的错误

```go
func Rename(oldpath, newpath string) error  //Rename修改一个文件的名字，移动一个文件。可能会有一些个操作系统特定的限制
```

```go
os.Stdin //从键盘标准输入，当从键盘输入的内容包含（空格时）可用
os.Stdout //输出到终端
```

# compress 包：读取压缩文件

`compress` 包提供了读取压缩文件的功能，支持的压缩文件格式为：bzip2、flate、gzip、lzw 和 zlib。

下面的程序展示了如何读取一个 gzip 文件。

```go
func main() {
	fName := "MyFile.gz"
	fi, err := os.Open(fName)
	if err != nil {
		fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", os.Args[0], fName, err)
		os.Exit(1)
	}
	defer fi.Close()

	fz, err := gzip.NewReader(fi)
  var r *bufio.Reader
	if err != nil {
		r = bufio.NewReader(fi)
	} else {
		r = bufio.NewReader(fz)
	}

	for {
		line, err := r.ReadString('\n')
		if err != nil {
			fmt.Println("Done reading file")
			os.Exit(0)
		}
		fmt.Println(line)
	}
}
```

# JSON 数据格式

数据结构要在网络中传输或保存到文件，就必须对其编码和解码；目前存在很多编码格式： JSON（JavaScript Object Notation），XML，gob，Google 缓冲协议等等。通常 JSON 被用于 web 后端和浏览器之间的通讯，但是在其它场景也同样的有用。

结构可能包含二进制数据，如果将其作为文本打印，那么可读性是很差的。另外结构内部可能包含匿名字段，而不清楚数据的用意。

通过把数据转换成纯文本，使用命名的字段来标注，让其具有可读性。这样的数据格式可以通过网络传输，而且是与平台无关的，任何类型的应用都能够读取和输出，不与操作系统和编程语言的类型相关。

下面是一些术语说明：

- 数据结构 --> 指定格式 = **序列化** 或 **编码**（传输之前）
- 指定格式 --> 数据结构 = **反序列化** 或 **解码**（传输之后）

序列化是在内存中把数据转换成指定格式（数据 -> 字符串），反之亦然（字符串 -> 数据）。

编码也是一样的，只是输出一个数据流（实现了 `io.Writer` 接口）；解码是从一个数据流（实现了 `io.Reader`）输出到一个数据结构。

## Json示例

这是一个简短的 JSON 片段：

```json
{
    "Person": {
        "FirstName": "Laura",
        "LastName": "Lynn"
    }
}
```

尽管 XML 被广泛的应用，但是 JSON 更加简洁、轻量（占用更少的内存、磁盘及网络带宽）和更好的可读性，这也使它越来越受欢迎。

Go 语言的 `json` 包可以让你在程序中方便的读取和写入 JSON 数据。

```go
type Address struct {
	Type    string
	City    string
	Country string
}

type VCard struct {
	FirstName string
	LastName  string
	Addresses []*Address
	Remark    string
}

func main() {
	pa := &Address{"private", "Aartselaar", "Belgium"}
	wa := &Address{"work", "Boom", "Belgium"}
	vc := VCard{"Jan", "Kersschot", []*Address{pa, wa}, "none"}
	// fmt.Printf("%v: \n", vc) // {Jan Kersschot [0x126d2b80 0x126d2be0] none}:
	// JSON format:
	js, _ := json.Marshal(vc)
	fmt.Printf("JSON format: %s", js)
	// using an encoder:
	file, _ := os.OpenFile("vcard.json", os.O_CREATE|os.O_WRONLY, 0666)
	defer file.Close()
	enc := json.NewEncoder(file)
	err := enc.Encode(vc)
	if err != nil {
		log.Println("Error in encoding json")
	}
}
```

`json.Marshal()` 的函数签名是 `func Marshal(v interface{}) ([]byte, error)`，下面是数据编码后的 JSON 文本（实际上是一个 `[]byte`）：

```json
{
    "FirstName": "Jan",
    "LastName": "Kersschot",
    "Addresses": [{
        "Type": "private",
        "City": "Aartselaar",
        "Country": "Belgium"
    }, {
        "Type": "work",
        "City": "Boom",
        "Country": "Belgium"
    }],
    "Remark": "none"
}
```

出于安全考虑，在 web 应用中最好使用 `json.MarshalforHTML()` 函数，其对数据执行 HTML 转码，所以文本可以被安全地嵌在 HTML `<script>` 标签中。

`json.NewEncoder()` 的函数签名是 `func NewEncoder(w io.Writer) *Encoder`，返回的 `Encoder` 类型的指针可调用方法 `Encode(v interface{})`，将数据对象 `v` 的 json 编码写入 `io.Writer` `w` 中。

JSON 与 Go 类型对应如下：

- `bool` 对应 JSON 的 boolean
- `float64` 对应 JSON 的 number
- `string` 对应 JSON 的 string
- `nil` 对应 JSON 的 null

不是所有的数据都可以编码为 JSON 类型，只有验证通过的数据结构才能被编码：

- JSON 对象只支持字符串类型的 key；要编码一个 Go `map` 类型，`map` 必须是 `map[string]T`（`T` 是 `json` 包中支持的任何类型）
- Channel，复杂类型和函数类型不能被编码
- 不支持循环数据结构；它将引起序列化进入一个无限循环
- 指针可以被编码，实际上是对指针指向的值进行编码（或者指针是 `nil`）



## JSON编码

&#8195;&#8195;json是go标准库里自带的序列化工具，使用了反射，效率比较低。
&#8195;&#8195;easyjson只针对预先定义好的json结构体对输入的json字符串进行纯字符串的截取，并将对应的json字段赋值给结构体。easyjson -all xxx.go 生成go文件中定义的结构体对应的解析，xxx.go所在的package不能是main。  

```Go
func easyjson.Marshal(v easyjson.Marshaler) ([]byte, error)
func easyjson.Unmarshal(data []byte, v easyjson.Unmarshaler) error
```

&#8195;&#8195;sonic是字节跳动开源的json序列化工具包，号称性能强过easyjson、jsoniter，使用起来非常方便。  

```Go
import "github.com/bytedance/sonic"

// Marshal
output, err := sonic.Marshal(&data) 
// Unmarshal
err := sonic.Unmarshal(input, &data) 
```

&#8195;&#8195;base64经常在http环境下用来传输较长的信息。任意byte数组都可以采用base64编码转为字符串，并且可以反解回byte数组。编码和解码的方法是公开、确定的， base64不属于加密算法。

```Go
func (*base64.Encoding).EncodeToString(src []byte) string
func (*base64.Encoding).DecodeString(s string) ([]byte, error)
```

&#8195;&#8195;compress包下实现了zlib、bzip、gip、lzw等压缩算法。  

```Go
writer := zlib.NewWriter(fout)//压缩
writer.Write(bytes)
reader, err := zlib.NewReader(fin) //解压
io.Copy(os.Stdout, reader)   //把reader流里的内容拷贝给标准输出流，即文件解压后的内容打印到控制台
```

## 反序列化：

`json.Unmarshal()` 的函数签名是 `func Unmarshal(data []byte, v interface{}) error` 把 JSON 解码为数据结构。

示例 12.16 中对 `vc` 编码后的数据为 `js` ，对其解码时，我们首先创建结构 `VCard` 用来保存解码的数据：`var v VCard` 并调用 `json.Unmarshal(js, &v)`，解析 `[]byte` 中的 JSON 数据并将结果存入指针 `&v` 指向的值。

虽然反射能够让 JSON 字段去尝试匹配目标结构字段；但是只有真正匹配上的字段才会填充数据。字段没有匹配不会报错，而是直接忽略掉。

## 解码任意的数据：

json 包使用 `map[string]interface{}` 和 `[]interface{}` 储存任意的 JSON 对象和数组；其可以被反序列化为任何的 JSON blob 存储到接口值中。

来看这个 JSON 数据，被存储在变量 `b` 中：

```go
b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)
```

不用理解这个数据的结构，我们可以直接使用 `Unmarshal()` 把这个数据编码并保存在接口值中：

```go
var f interface{}
err := json.Unmarshal(b, &f)
```

f 指向的值是一个 `map`，key 是一个字符串，value 是自身存储作为空接口类型的值：

```go
map[string]interface{} {
	"Name": "Wednesday",
	"Age":  6,
	"Parents": []interface{} {
		"Gomez",
		"Morticia",
	},
}
```

要访问这个数据，我们可以使用类型断言

```go
m := f.(map[string]interface{})
```

我们可以通过 for range 语法和 type switch 来访问其实际类型：

```go
for k, v := range m {
	switch vv := v.(type) {
	case string:
		fmt.Println(k, "is string", vv)
	case int:
		fmt.Println(k, "is int", vv)

	case []interface{}:
		fmt.Println(k, "is an array:")
		for i, u := range vv {
			fmt.Println(i, u)
		}
	default:
		fmt.Println(k, "is of a type I don’t know how to handle")
	}
}
```

通过这种方式，你可以处理未知的 JSON 数据，同时可以确保类型安全。

## 解码数据到结构

如果我们事先知道 JSON 数据，我们可以定义一个适当的结构并对 JSON 数据反序列化。下面的例子中，我们将定义：

```go
type FamilyMember struct {
	Name    string
	Age     int
	Parents []string
}
```

并对其反序列化：

```go
var m FamilyMember
err := json.Unmarshal(b, &m)
```

程序实际上是分配了一个新的切片。这是一个典型的反序列化引用类型（指针、切片和 `map`）的例子。

## 编码和解码流

`json` 包提供 `Decoder` 和 `Encoder` 类型来支持常用 JSON 数据流读写。`NewDecoder()` 和 `NewEncoder()` 函数分别封装了 `io.Reader` 和 `io.Writer` 接口。

```go
func NewDecoder(r io.Reader) *Decoder
func NewEncoder(w io.Writer) *Encoder
```

要想把 JSON 直接写入文件，可以使用 `json.NewEncoder` 初始化文件（或者任何实现 `io.Writer` 的类型），并调用 `Encode()`；反过来与其对应的是使用 `json.NewDecoder` 和 `Decode()` 函数：

```go
func NewDecoder(r io.Reader) *Decoder
func (dec *Decoder) Decode(v interface{}) error
```

来看下接口是如何对实现进行抽象的：数据结构可以是任何类型，只要其实现了某种接口，目标或源数据要能够被编码就必须实现 `io.Writer` 或 `io.Reader` 接口。由于 Go 语言中到处都实现了 Reader 和 Writer，因此 `Encoder` 和 `Decoder` 可被应用的场景非常广泛，例如读取或写入 HTTP 连接、websockets 或文件。

# XML 数据格式

```xml
<Person>
    <FirstName>Laura</FirstName>
    <LastName>Lynn</LastName>
</Person>
```

如同 `json` 包一样，也有 `xml.Marshal()` 和 `xml.Unmarshal()` 从 XML 中编码和解码数据；但这个更通用，可以从文件中读取和写入（或者任何实现了 `io.Reader` 和 `io.Writer` 接口的类型）

和 JSON 的方式一样，XML 数据可以序列化为结构，或者从结构反序列化为 XML 数据；

`encoding`/`xml` 包实现了一个简单的 XML 解析器（SAX），用来解析 XML 数据内容。下面的例子说明如何使用解析器：

```go
import (
	"encoding/xml"
	"fmt"
	"strings"
)

var t, token xml.Token
var err error

func main() {
	input := "<Person><FirstName>Laura</FirstName><LastName>Lynn</LastName></Person>"
	inputReader := strings.NewReader(input)
	p := xml.NewDecoder(inputReader)

	for t, err = p.Token(); err == nil; t, err = p.Token() {
		switch token := t.(type) {
		case xml.StartElement:
			name := token.Name.Local
			fmt.Printf("Token name: %s\n", name)
			for _, attr := range token.Attr {
				attrName := attr.Name.Local
				attrValue := attr.Value
				fmt.Printf("An attribute is: %s %s\n", attrName, attrValue)
				// ...
			}
		case xml.EndElement:
			fmt.Println("End of token")
		case xml.CharData:
			content := string([]byte(token))
			fmt.Printf("This is the content: %v\n", content)
			// ...
		default:
			// ...
		}
	}
}
```

输出：

```
Token name: Person
Token name: FirstName
This is the content: Laura
End of token
Token name: LastName
This is the content: Lynn
End of token
End of token
```

包中定义了若干 XML 标签类型：StartElement，Chardata（这是从开始标签到结束标签之间的实际文本），EndElement，Comment，Directive 或 ProcInst。

包中同样定义了一个结构解析器：`NewParser()` 方法持有一个 `io.Reader`（这里具体类型是 `strings.NewReader`）并生成一个解析器类型的对象。还有一个 `Token()` 方法返回输入流里的下一个 XML token。在输入流的结尾处，会返回 (`nil`,`io.EOF`)

XML 文本被循环处理直到 `Token()` 返回一个错误，因为已经到达文件尾部，再没有内容可供处理了。通过一个 type-switch 可以根据一些 XML 标签进一步处理。Chardata 中的内容只是一个 `[]byte`，通过字符串转换让其变得可读性强一些。

# Gob 传输数据

## 什么是Gob

**Gob的定义：** Gob是Go自己的以二进制形式序列化和反序列化程序数据的格式，这种数据格式简称之为**Gob** (Go binary)。

它类似于Java语言当中的`Serialization` 。你可以在`encoding` 包中找到它。

## Gob可以做什么

Gob 通常用于远程方法调用（RPC）参数和结果的传输，以及应用程序和机器之间的数据传输。

那么，它与我们之前普遍用到的JSON有什么不同呢？

Gob因为是 Go自己的以二进制形式序列化和反序列化程序数据的格式，因此呢只能用于纯Go环境当中，并不适用于异构的环境。例如，它可以用于两个Go程序之间的通信。

## Gob的特点

1. Gob 文件或流是完全自描述的：里面包含的所有类型都有一个对应的描述，并且总是可以用 Go 解码，而不需要了解文件的内容。
2. 只有**可导出**的字段会被编码，零值会被忽略。
3. 在解码结构体的时候，只有**同时匹配名称和可兼容类型**的字段才会被解码。
4. 当源数据类型增加新字段后，Gob 解码客户端仍然可以以这种方式正常工作：解码客户端会继续识别以前存在的字段。

## 使用Gob传输数据

和 JSON 的使用方式一样，Gob 使用通用的 `io.Writer` 接口，通过 `NewEncoder()` 函数创建 `Encoder` 对象并调用 `Encode()`；相反的过程使用通用的 `io.Reader` 接口，通过 `NewDecoder()` 函数创建 `Decoder` 对象并调用 `Decode()`。

```go
type Immortal struct { // 修仙者
	Name   string
	Age    int
	Gender string
}

type SimpleImmortal struct {
	Name   string
	Age    int
}

var buf  bytes.Buffer

func main() {
	var hanli = Immortal{
		Name:   "韩立",
		Age:    18,
		Gender: "男性",
	}

	fmt.Println("发送数据: ",hanli)
	sendMsg(&hanli)
	fmt.Println("buf中的数据：",buf)
	var i SimpleImmortal
	msg, _ := receiveMsg(i)

	fmt.Println("接收到数据：",msg)
}

func sendMsg(immortal *Immortal) error {
	enc :=gob.NewEncoder(&buf)
	return enc.Encode(immortal)
}

func receiveMsg(immortal SimpleImmortal) (SimpleImmortal,error) {
	dec := gob.NewDecoder(&buf)

	return immortal,dec.Decode(&immortal)

}
```

输出：

```go
发送数据:  {韩立 18 男性}
buf中的数据： {[50 255 129 3 1 1 8 73 109 109 111 114 116 97 108 1 255 130 0 1 3 1 4 78 97 109 101 1 12 0 1 3 65 103 101 1 4 0 1 6 71 101 110 100 101 114 1 12 0 0 0 21 255 130 1 6 233 159 169 231 171 139 1 36 1 6 231 148 183 230 128 167 0] 0 0}
接收到数据： {韩立 18}
```



# Go 中的密码学

通过网络传输的数据必须加密，以防止被 hacker（黑客）读取或篡改，并且保证发出的数据和收到的数据检验和一致。
鉴于 Go 母公司的业务，我们毫不惊讶地看到 Go 的标准库为该领域提供了超过 30 个的包：

- `hash` 包：实现了 `adler32`、`crc32`、`crc64` 和 `fnv` 校验；
- `crypto` 包：实现了其它的 hash 算法，比如 `md4`、`md5`、`sha1` 等。以及完整地实现了 `aes`、`blowfish`、`rc4`、`rsa`、`xtea` 等加密算法。

下面的示例用 `sha1` 和 `md5` 计算并输出了一些校验值。

```go
func main() {
    hasher := sha1.New()
    io.WriteString(hasher, "test")
    b := []byte{}
    fmt.Printf("Result: %x\n", hasher.Sum(b))
    fmt.Printf("Result: %d\n", hasher.Sum(b))
    //
    hasher.Reset()
    data := []byte("We shall overcome!")
    n, err := hasher.Write(data)
    if n!=len(data) || err!=nil {
        log.Printf("Hash write error: %v / %v", n, err)
    }
    checksum := hasher.Sum(b)
    fmt.Printf("Result: %x\n", checksum)
}Copy
```

输出：

```php
Result: a94a8fe5ccb19ba61c4c0873d391e987982fbbd3
Result: [169 74 143 229 204 177 155 166 28 76 8 115 211 145 233 135 152 47 187 211]
Result: e2222bfc59850bbb00a722e764a555603bb59b2a
```

通过调用 `sha1.New()` 创建了一个新的 `hash.Hash` 对象，用来计算 SHA1 校验值。`Hash` 类型实际上是一个接口，它实现了 `io.Writer` 接口：

```go
type Hash interface {
    // Write (via the embedded io.Writer interface) adds more data to the running hash.
    // It never returns an error.
    io.Writer

    // Sum appends the current hash to b and returns the resulting slice.
    // It does not change the underlying hash state.
    Sum(b []byte) []byte

    // Reset resets the Hash to its initial state.
    Reset()

    // Size returns the number of bytes Sum will return.
    Size() int

    // BlockSize returns the hash's underlying block size.
    // The Write method must be able to accept any amount
    // of data, but it may operate more efficiently if all writes
    // are a multiple of the block size.
    BlockSize() int
}
```

通过 io.WriteString 或 hasher.Write 将给定的 [] byte 附加到当前的 `hash.Hash` 对象中。
