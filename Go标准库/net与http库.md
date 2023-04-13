## net/http介绍

Go语言内置的`net/http`包提供了HTTP客户端和服务端的实现。

Go语言内置的`net/http`包十分的优秀，提供了HTTP客户端和服务端的实现。

### HTTP协议

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络传输协议，所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法。

## HTTP客户端

### 基本的HTTP/HTTPS请求

Get、Head、Post和PostForm函数发出HTTP/HTTPS请求。

```go
resp, err := http.Get("http://example.com/")
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

程序在使用完response后必须关闭回复的主体。

```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```

### GET请求示例

使用`net/http`包编写一个简单的发送HTTP请求的Client端，代码如下：

```go
func main() {
	resp, err := http.Get("https://www.liwenzhou.com/")
	if err != nil {
		fmt.Printf("get failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("read from resp.Body failed, err:%v\n", err)
		return
	}
	fmt.Print(string(body))
}
```

将上面的代码保存之后编译成可执行文件，执行之后就能在终端打印`liwenzhou.com`网站首页的内容了，我们的浏览器其实就是一个发送和接收HTTP协议数据的客户端，我们平时通过浏览器访问网页其实就是从网站的服务器接收HTTP数据，然后浏览器会按照HTML、CSS等规则将网页渲染展示出来。

### 带参数的GET请求示例

关于GET请求的参数需要使用Go语言内置的`net/url`这个标准库来处理。

```go
func main() {
	apiUrl := "http://127.0.0.1:9090/get"
	// URL param
	data := url.Values{}
	data.Set("name", "小王子")
	data.Set("age", "18")
	u, err := url.ParseRequestURI(apiUrl)
	if err != nil {
		fmt.Printf("parse url requestUrl failed, err:%v\n", err)
	}
	u.RawQuery = data.Encode() // URL encode
	fmt.Println(u.String())
	resp, err := http.Get(u.String())
	if err != nil {
		fmt.Printf("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("get resp failed, err:%v\n", err)
		return
	}
	fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```go
func getHandler(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()
	data := r.URL.Query()
	fmt.Println(data.Get("name"))
	fmt.Println(data.Get("age"))
	answer := `{"status": "ok"}`
	w.Write([]byte(answer))
}
```

### Post请求示例

上面演示了使用`net/http`包发送`GET`请求的示例，发送`POST`请求的示例代码如下：

```go
func main() {
	url := "http://127.0.0.1:9090/post"
	// 表单数据
	//contentType := "application/x-www-form-urlencoded"
	//data := "name=小王子&age=18"
	// json
	contentType := "application/json"
	data := `{"name":"小王子","age":18}`
	resp, err := http.Post(url, contentType, strings.NewReader(data))
	if err != nil {
		fmt.Printf("post failed, err:%v\n", err)
		return
	}
	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		fmt.Printf("get resp failed, err:%v\n", err)
		return
	}
	fmt.Println(string(b))
}
```

对应的Server端HandlerFunc如下：

```go
func postHandler(w http.ResponseWriter, r *http.Request) {
	defer r.Body.Close()
	// 1. 请求类型是application/x-www-form-urlencoded时解析form数据
	r.ParseForm()
	fmt.Println(r.PostForm) // 打印form数据
	fmt.Println(r.PostForm.Get("name"), r.PostForm.Get("age"))
	// 2. 请求类型是application/json时从r.Body读取数据
	b, err := ioutil.ReadAll(r.Body)
	if err != nil {
		fmt.Printf("read request.Body failed, err:%v\n", err)
		return
	}
	fmt.Println(string(b))
	answer := `{"status": "ok"}`
	w.Write([]byte(answer))
}
```



### 获取请求 URL

Request 结构中的 URL 字段用于表示请求行中包含的 URL，改字段是一个指向url.URL 结构的指针

#### type [URL](https://github.com/golang/go/blob/master/src/net/url/url.go?name=release#230)

```
type URL struct {
    Scheme   string
    Opaque   string    // 编码后的不透明数据
    User     *Userinfo // 用户名和密码信息
    Host     string    // host或host:port
    Path     string
    RawQuery string // 编码后的查询字符串，没有'?'
    Fragment string // 引用的片段（文档位置），没有'#'
}
```

URL类型代表一个解析后的URL（或者说，一个URL参照）。URL基本格式如下：

```
scheme://[userinfo@]host/path[?query][#fragment]
```

scheme后不是冒号加双斜线的URL被解释为如下格式：

```
scheme:opaque[?query][#fragment]
```

注意路径字段是以解码后的格式保存的，如/%47%6f%2f会变成/Go/。这导致我们无法确定Path字段中的斜线是来自原始URL还是解码前的%2f。除非一个客户端必须使用其他程序/函数来解析原始URL或者重构原始URL，这个区别并不重要。此时，HTTP服务端可以查询req.RequestURI，而HTTP客户端可以使用URL{Host: "example.com", Opaque: "//example.com/Go%2f"}代替{Host: "example.com", Path: "/Go/"}。

#### Path 字段

获取请求的 URL

例如：http://localhost:8080/hello?username=admin&password=123456

​	通过 r.URL.Path 只能得到 **/hello**

#### RawQuery 字段

获取请求的 URL 后面?后面的查询字符串

例如：http://localhost:8080/hello?username=admin&password=123456

​	通过 r.URL.RawQuery 得到的是 username=admin&password=123456

### 获取请求头中的信息

通过 Request 结果中的 Header 字段用来获取请求头中的所有信息，Header 字段的类型是 Header 类型，而 Header 类型是一个 map[string][]string，string 类型的 key，string 切片类型的值。

```go
type Header map[string][]string
```

Header代表HTTP头域的键值对。

```
func (h Header) Get(key string) string
```

Get返回键对应的第一个值，如果键不存在会返回""。如要获取该键对应的值切片，请直接用规范格式的键访问map。

```
func (h Header) Set(key, value string)
```

Set添加键值对到h，如键已存在则会用只有新值一个元素的切片取代旧值切片。

```
func (h Header) Add(key, value string)
```

Add添加键值对到h，如键已存在则会将新的值附加到旧值切片后面。

```
func (h Header) Del(key string)
```

Del删除键值对。

```
func (h Header) Write(w io.Writer) error
```

Write以有线格式将头域写入w。

1) 获取请求头中的所有信息

**r.Header**得到的结果如下：、

**map**[User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64) 

AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36] 

Accept:[text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,ima

ge/apng,*/*;q=0.8] Accept-Encoding:[gzip, deflate, br] Accept-Language:[zh

CN,zh;q=0.9,en-US;q=0.8,en;q=0.7] Connection:[keep-alive] Upgrade-Insecure

Requests:[1]]

2) 获取请求头中的某个具体属性的值，如获取 Accept-Encoding 的值

方式一：**r.Header[“Accept-Encoding”]**

i. 得到的是一个字符串切片

ii. 结果[gzip, deflate, br]

方式二：**r.Header.Get(“Accept-Encoding”)**

i. 得到的是字符串形式的值，多个值使用逗号分隔

ii. 结果gzip, deflate, br

### **获取请求体中的信息**

请求和响应的主体都是有 Request 结构中的 Body 字段表示，这个字段的类型是io.ReadCloser 接口，该接口包含了 Reader 接口和 Closer 接口，Reader 接口拥有 Read方法，Closer 接口拥有 Close 方法

```
type ReadCloser interface {
	Reader
	Closer
}
```

ReadCloser是一个接口，它将基本的Read和Close方法组合在一起。

```
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

Reader是包装基本Read方法的接口。

Read将len（p）个字节读入p。它返回字节数 read（0<= n<= len（p））和遇到的任何错误。即使阅读 返回n<len（p），它可以在调用期间使用所有p作为临时空间。 如果某些数据可用，但不是len（p）字节，则按常规读取 返回可用的而不是等待更多。

当读取遇到错误或文件结束条件时 成功阅读n>个字节，则返回 字节读取。它可能从同一个调用返回（非nil）错误 或者从后续调用返回错误（并且n == 0）。 这种一般情况的一个实例是返回 输入流末尾的非零字节数可以 返回err == EOF或err == nil。下一次阅读应该 return 0，EOF.调用方应始终处理之前返回的>n字节 考虑到误差Err.这样做可以正确处理I/O错误 在阅读一些字节之后发生的，并且 允许的EOF行为。

不鼓励Read的实现返回 零字节计数，错误为nil，除非len（p）== 0。 调用者应该把0和nil的返回值视为 什么都没发生特别地，它不指示EOF。

实现不能保留p。

```
type Closer interface {
	Close() error
}
```

Closer是包装基本Close方法的接口。

第一次调用后的Close行为未定义。 具体实现可以记录它们自己的行为。

```go
func handler(w http.ResponseWriter, r *http.Request) {//获取内容的长度
	length := r.ContentLength//创建一个字节切片
	body := make([]byte, length)//读取请求体
	r.Body.Read(body)
	fmt.Fprintln(w, "请求体中的内容是：", string(body))
}
```

> 请求体中的内容是： username=hanzong&password=666666

### 获取请求参数

下面我们就通过 net/http 库中的 Request 结构的字段以及方法获取**请求 URL 后面**的请求参数以及 **form 表单**中提交的请求参数

####  **Form 字段** 

1)  **url.Values** 类型，Form 是解析好的表单数据，包括 URL 字段的 query参数和 POST 或 PUT 的表单数据。

```
type Values map[string][]string
```

Values将字符串键映射到值列表。 它通常用于查询参数和表单值。 与http.Header映射不同，Values映射中的键 区分大小写。

2) Form 字段只有在调用 Request 的 **ParseForm** 方法后才有效。在客户端，会忽略请求中的本字段而使用 Body 替代

```
func (r *Request) ParseForm() error
```

ParseForm解析URL中的查询字符串，并将解析结果更新到r.Form字段。

对于POST或PUT请求，ParseForm还会将body当作表单解析，并将结果既更新到r.PostForm也更新到r.Form。解析结果中，POST或PUT请求主体要优先于URL查询字符串（同名变量，主体的值在查询字符串的值前面）。

如果请求的主体的大小没有被MaxBytesReader函数设定限制，其大小默认限制为开头10MB。

ParseMultipartForm会自动调用ParseForm。重复调用本方法是无意义的。

3) 获取表单中提交的请求参数（username 和 password）

```go
r.ParseForm() //解析表单
fmt.Fprintln(w, "请求参数为：", r.Form)//获取所有请求参数
```

> 请求参数为： map[password:[666666] username:[hanzong]]

#### Value字段

**FormValue 方法**

快速地获取某一个请求参数，该方法调用之前会自动调用 ParseMultipartForm 和 ParseForm 方法对表单进行解析

```go
func (r *Request) FormValue(key string) string
```


窗体值返回查询的命名组件的第一个值。

**PostFormValue 方法**

快速地获取表单中的某一个请求参数，该方法调用之前会自动调用 ParseMultipartForm 和 ParseForm 方法对表单进行解析

```go
func (r *Request) PostFormValue(key string) string
```

PostFormValue 返回 POST、PATCH 或 PUT 请求正文的命名组件的第一个值。

 **MultipartForm 字段** 

为了取得 multipart/form-data 编码的表单数据，我们需要用到 Request 结构的ParseMultipartForm 方法和 MultipartForm 字段，我们通常上传文件时会将 form 表单的enctype 属性值设置为 multipart/form-data

```go
func (r *Request) ParseMultipartForm(maxMemory int64) error //解析表单
fmt.Fprintln(w, r.MultipartForm) //打印表单数据
```

> &{map[username:[hanzong]] map[photo:[0xc042126000]]}

将请求正文解析为多部分/表单数据。



### 客户端响应

1、HTTP 处理程序使用 ResponseWriter 接口来构造 HTTP 响应。

```go
type ResponseWriter interface {
    Header() Header
    Write([]byte) (int, error)
    WriteHeader(statusCode int)
}

func (ResponseWriter) Write([]byte) (int, error) //写入将数据作为 HTTP 回复的一部分写入连接。
```

2、给客户端响应 JSON 格式的数据

```go
	w.Header().Set("Content-Type", "application/json") //设置响应头中内容的类型
	user := User{
		ID:       1,
		Username: "admin",
		Password: "123456",
	}

	json, _ := json.Marshal(user) //将 user 转换为 json 格式
	w.Write(json)
```

响应报文中的内容

> HTTP/1.1 200 OK
>
> **Content-Type: application/json**
>
> Date: Fri, 10 Aug 2018 01:58:02 GMT
>
> Content-Length: 47

3、客户端重定向

```go
w.Header().Set("Location", "https:www.baidu.com") //以下操作必须要在 WriteHeader 之前进行
w.WriteHeader(302)
```

响应报文中的内容

> HTTP/1.1 302 Found
>
> **Location: https:www.baidu.c**om
>
> Date: Fri, 10 Aug 2018 01:45:04 GMT
>
> Content-Length: 0
>
> **Content-Type: text/plain; charset=utf-8**





### 自定义Client

要管理HTTP客户端的头域、重定向策略和其他设置，创建一个Client：

```go
client := &http.Client{
	CheckRedirect: redirectPolicyFunc,
}
resp, err := client.Get("http://example.com")
// ...
req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)
// ...
```

### 自定义Transport

要管理代理、TLS配置、keep-alive、压缩和其他设置，创建一个Transport：

```go
tr := &http.Transport{
	TLSClientConfig:    &tls.Config{RootCAs: pool},
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

Client和Transport类型都可以安全的被多个goroutine同时使用。出于效率考虑，应该一次建立、尽量重用。



### template模板引擎响应

Go 为我们提供了 **text/template** 库和 **html/template** 库这两个模板引擎，模板引擎通过将数据和模板组合在一起生成最终的 HTML，而处理器负责调用模板引擎并将引擎生成的 HTMl 返回给客户端。

Go 的模板都是文本文档（其中 Web 应用的模板通常都是 HTML），它们都嵌入了一些称为**动作**的指令。从模板引擎的角度来说，模板就是嵌入了动作的文本（这些文本通常包含在模板文件里面），而模板引擎则通过分析并执行这些文本来生成出另外一些文本。

#### HelloWorld模板

使用 Go 的 Web 模板引擎需要以下两个步骤:

(1) 对文本格式的模板源进行语法分析，创建一个经过语法分析的模板结构，其中模板源既可以是一个字符串，也可以是模板文件中包含的内容。

(2 )执行经过语法分析的模板，将 ResponseWriter 和模板所需的动态数据传递给模板引擎，被调用的模板引擎会把经过语法分析的模板和传入的数据结合起来，生成出最终的 HTML，并将这些 HTML 传递给 ResponseWriter。

```go
func ParseFiles(filenames ...string) (*Template, error)
```

解析文件创建一个新模板，并从命名文件中解析 模板 定义。

当我们调用 ParseFiles 函数解析模板文件时，Go 会创建一个新的模板，并将给定的模板文件的名字作为新模板的名字，如果该函数中传入了多个文件名，那么也只会返回一个模板，而且以第一个文件的文件名作为模板的名字，至于其他文件对应的模板则会被放到一个 map 中。

```go
t, _ := template.ParseFiles("hello.html")
```

以上代码相当于调用 New 函数创建一个新模板，然后再调用 template 的ParseFiles 方法：

```go
t := template.New("hello.html")
t, _ = t.ParseFiles("hello.html")
t, _ := template.ParseGlob("*.html") //通过该函数可以通过指定一个规则一次性传入多个模板文件
```

#### 解析模板错误

在解析模板时都没有对错误进行处理，Go 提供了一个 Must 函数专门用来处理这个错误。Must 函数可以包裹起一个函数，被包裹的函数会返回一个指向模板的指针和一个错误，如果错误不是 nil，那么 Must 函数将产生一个 panic。

```go
t := template.Must(template.ParseFiles("hello.html"))
```



#### **执行模板**

1) 通过 Execute 方法

```go
func (t *Template) Execute(wr io.Writer, data any) error
```

如果只有一个模板文件，调用这个方法总是可行的；但是如果有多个模板文件，调用这个方法只能得到第一个模板

2) 通过 ExecuteTemplate 方法

```go
t, _ := template.ParseFiles("hello.html", "hello2.html")
t.ExecuteTemplate(w, "hello2.html", "我要在 hello2.html 中显示")
```

变量 t 就是一个包含了两个模板的模板集合，第一个模板的名字是hello.html,第二个模板的名字是 hello2.html,如果直接调用 Execute 方法，则只有模板 hello.html 会被执行，如何想要执行模板 hello2.html，则需要调用 ExecuteTemplate 方法



## 服务端

### 默认的Server

ListenAndServe使用指定的监听地址和处理器启动一个HTTP服务端。处理器参数通常是nil，这表示采用包变量DefaultServeMux作为处理器。

Handle和HandleFunc函数可以向DefaultServeMux添加处理器。

```go
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```

### 默认的Server示例

使用Go语言中的`net/http`包来编写一个简单的接收HTTP请求的Server端示例，`net/http`包是对net包的进一步封装，专门用来处理HTTP协议的数据。具体的代码如下：

```go
// http server

func sayHello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello 沙河！")
}

func main() {
	http.HandleFunc("/", sayHello)
	err := http.ListenAndServe(":9090", nil)
	if err != nil {
		fmt.Printf("http server failed, err:%v\n", err)
		return
	}
}
```

将上面的代码编译之后执行，打开你电脑上的浏览器在地址栏输入`127.0.0.1:9090`回车，此时就能够看到如下页面了。![hello页面](https://www.liwenzhou.com/images/Go/socket/hello.png)

### 自定义Server

要管理服务端的行为，可以创建一个自定义的Server：

```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```