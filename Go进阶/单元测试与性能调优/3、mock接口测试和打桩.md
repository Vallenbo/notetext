## gomock

[gomock](https://github.com/golang/mock)是Go官方提供的测试框架，它在内置的testing包或其他环境中都能够很方便的使用。我们使用它对代码中的那些接口类型进行mock，方便编写单元测试。

### 安装mockgen

> 互联网开源库更新迭代比较快，建议直接查看官方文档：https://github.com/golang/mock

首先需要确保你的`$GOPATH/bin`已经加入到环境变量中。

Go版本号<1.16时：

```bash
GO111MODULE=on go get github.com/golang/mock/mockgen@v1.6.0
```

Go版本>=1.16时：

```bash
go install github.com/golang/mock/mockgen@v1.6.0
```

如果是在你的CI流水线中安装，则需要安装与你的CI环境匹配的合适版本。

### 运行mockgen

`mockgen` 有两种操作模式：源码（source）模式和反射（reflect）模式。

#### 源码模式

源码模式根据源文件mock接口。它是通过使用 `-source` 标志启用。在这个模式下可能有用的其他标志是 `-imports` 和 `-aux_files`。

例如：

```bash
mockgen -source=foo.go [other options]
```

#### 反射模式

反射模式通过构建使用反射来理解接口的程序来mock接口。它是通过传递两个非标志参数来启用的：一个导入路径和一个逗号分隔的符号列表。可以使用 ”.”引用当前路径的包。

例如：

```bash
mockgen database/sql/driver Conn,Driver

# Convenient for `go:generate`.
mockgen . Conn,Driver
```

### flags

`mockgen` 命令用来为给定一个包含要mock的接口的Go源文件，生成mock类源代码。它支持以下标志：

- `-source`：包含要mock的接口的文件。
- `-destination`：生成的源代码写入的文件。如果不设置此项，代码将打印到标准输出。
- `-package`：用于生成的模拟类源代码的包名。如果不设置此项包名默认在原包名前添加`mock_`前缀。
- `-imports`：在生成的源代码中使用的显式导入列表。值为foo=bar/baz形式的逗号分隔的元素列表，其中bar/baz是要导入的包，foo是要在生成的源代码中用于包的标识符。
- `-aux_files`：需要参考以解决的附加文件列表，例如在不同文件中定义的嵌入式接口。指定的值应为foo=bar/baz.go形式的以逗号分隔的元素列表，其中bar/baz.go是源文件，foo是`-source`文件使用的文件的包名。
- `-build_flags`：（仅反射模式）一字不差地传递标志给go build
- `-mock_names`：生成的模拟的自定义名称列表。这被指定为一个逗号分隔的元素列表，形式为`Repository = MockSensorRepository,Endpoint=MockSensorEndpoint`，其中`Repository`是接口名称，`mockSensorrepository`是所需的mock名称(mock工厂方法和mock记录器将以mock命名)。如果其中一个接口没有指定自定义名称，则将使用默认命名约定。
- `-self_package`：生成的代码的完整包导入路径。使用此flag的目的是通过尝试包含自己的包来防止生成代码中的循环导入。如果mock的包被设置为它的一个输入(通常是主输入)，并且输出是stdio，那么mockgen就无法检测到最终的输出包，这种情况就会发生。设置此标志将告诉 mockgen 排除哪个导入
- `-copyright_file`：用于将版权标头添加到生成的源代码中的版权文件
- `-debug_parser`：仅打印解析器结果
- `-exec_only`：（反射模式） 如果设置，则执行此反射程序
- `-prog_only`：（反射模式）只生成反射程序；将其写入标准输出并退出。
- `-write_package_comment`：如果为true，则写入包文档注释 (godoc)。（默认为true）

### 构建mock

这里就以日常开发中经常用到的数据库操作为例，讲解一下如何使用gomock来mock接口的单元测试。

假设有查询MySQL数据库的业务代码如下，其中`DB`是一个自定义的接口类型：

```go
// db.go
// DB 数据接口
type DB interface {
	Get(key string)(int, error)
	Add(key string, value int) error
}

// GetFromDB 根据key从DB查询数据的函数
func GetFromDB(db DB, key string) int {
	if v, err := db.Get(key);err == nil{
		return v
	}
	return -1
}
```

我们现在要为`GetFromDB`函数编写单元测试代码，可是我们又不能在单元测试过程中连接真实的数据库，这个时候就需要mock `DB`这个接口来方便进行单元测试。

使用上面提到的 `mockgen` 工具来为生成相应的mock代码。通过执行下面的命令，我们就能在当前项目下生成一个`mocks`文件夹，里面存放了一个`db_mock.go`文件。

```bash
mockgen -source=db.go -destination=mocks/db_mock.go -package=mocks
```

`db_mock.go`文件中的内容就是mock相关接口的代码了。

我们通常不需要编辑它，只需要在单元测试中按照规定的方式使用它们就可以了。例如，我们编写`TestGetFromDB` 函数如下：

```go
// db_test.go
func TestGetFromDB(t *testing.T) {
	// 创建gomock控制器，用来记录后续的操作信息
	ctrl := gomock.NewController(t)
	// 断言期望的方法都被执行
	// Go1.14+的单测中不再需要手动调用该方法
	defer ctrl.Finish()
	// 调用mockgen生成代码中的NewMockDB方法
	// 这里mocks是我们生成代码时指定的package名称
	m := mocks.NewMockDB(ctrl)
	// 打桩（stub）
	// 当传入Get函数的参数为liwenzhou.com时返回1和nil
	m.
		EXPECT().
		Get(gomock.Eq("liwenzhou.com")). // 参数
		Return(1, nil).                  // 返回值
		Times(1)                         // 调用次数

	// 调用GetFromDB函数时传入上面的mock对象m
	if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
		t.Fatal()
	}
}
```

### 打桩（stub）

软件测试中的打桩是指用一些代码（桩stub）代替目标代码，通常用来屏蔽或补齐业务逻辑中的关键代码方便进行单元测试。

> 屏蔽：不想在单元测试用引入数据库连接等重资源
>
> 补齐：依赖的上下游函数或方法还未实现

上面代码中就用到了打桩，当传入`Get`函数的参数为`liwenzhou.com`时就返回`1, nil`的返回值。

`gomock`支持针对参数、返回值、调用次数、调用顺序等进行打桩操作。

#### 参数

参数相关的用法有：

- gomock.Eq(value)：表示一个等价于value值的参数
- gomock.Not(value)：表示一个非value值的参数
- gomock.Any()：表示任意值的参数
- gomock.Nil()：表示空值的参数
- SetArg(n, value)：设置第n（从0开始）个参数的值，通常用于指针参数或切片

具体示例如下：

```go
m.EXPECT().Get(gomock.Not("q1mi")).Return(10, nil)
m.EXPECT().Get(gomock.Any()).Return(20, nil)
m.EXPECT().Get(gomock.Nil()).Return(-1, nil)
```

这里单独说一下`SetArg`的适用场景，假设你有一个需要mock的接口如下：

```go
type YourInterface {
  SetValue(arg *int)
}
```

此时，打桩的时候就可以使用`SetArg`来修改参数的值。

```go
m.EXPECT().SetValue(gomock.Any()).SetArg(0, 7)  // 将SetValue的第一个参数设置为7
```

#### 返回值

gomock中跟返回值相关的用法有以下几个：

- Return()：返回指定值
- Do(func)：执行操作，忽略返回值
- DoAndReturn(func)：执行并返回指定值

例如：

```go
m.EXPECT().Get(gomock.Any()).Return(20, nil)
m.EXPECT().Get(gomock.Any()).Do(func(key string) {
	t.Logf("input key is %v\n", key)
})
m.EXPECT().Get(gomock.Any()).DoAndReturn(func(key string)(int, error) {
	t.Logf("input key is %v\n", key)
	return 10, nil
})
```

#### 调用次数

使用gomock工具mock的方法都会有期望被调用的次数，默认每个mock方法只允许被调用一次。

```go
m.
	EXPECT().
	Get(gomock.Eq("liwenzhou.com")). // 参数
	Return(1, nil).                  // 返回值
	Times(1)                         // 设置Get方法期望被调用次数为1

// 调用GetFromDB函数时传入上面的mock对象m
if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
	t.Fatal()
}
// 再次调用上方mock的Get方法时不满足调用次数为1的期望
if v := GetFromDB(m, "liwenzhou.com"); v != 1 {
	t.Fatal()
}
```

gomock为我们提供了如下方法设置期望被调用的次数。

- `Times()` 断言 Mock 方法被调用的次数。
- `MaxTimes()` 最大次数。
- `MinTimes()` 最小次数。
- `AnyTimes()` 任意次数（包括 0 次）。

#### 调用顺序

gomock还支持使用`InOrder`方法指定mock方法的调用顺序：

```go
// 指定顺序
gomock.InOrder(
	m.EXPECT().Get("1"),
	m.EXPECT().Get("2"),
	m.EXPECT().Get("3"),
)

// 按顺序调用
GetFromDB(m, "1")
GetFromDB(m, "2")
GetFromDB(m, "3")
```

此外知名的Go测试库[testify](https://github.com/stretchr/testify)目前也提供类似的mock工具—`testify/mock`和`mockery`。

## GoStub

[GoStub](https://github.com/prashantv/gostub)也是一个单元测试中的打桩工具，它支持为全局变量、函数等打桩。

不过我个人感觉它为函数打桩不太方便，我一般在单元测试中只会使用它来为全局变量打桩。

### 安装

```bash
go get github.com/prashantv/gostub
```

### 使用示例

这里使用官方文档中的示例代码演示如何使用gostub为全局变量打桩。

```go
// app.go 
var (
	configFile = "config.json"
	maxNum = 10
)

func GetConfig() ([]byte, error) {
	return ioutil.ReadFile(configFile)
}

func ShowNumber()int{
	// ...
	return maxNum
}
```

上面代码中定义了两个全局变量和两个使用全局变量的函数，我们现在为这两个函数编写单元测试。

```go
// app_test.go

import (
	"github.com/prashantv/gostub"
	"testing"
)

func TestGetConfig(t *testing.T) {
	// 为全局变量configFile打桩，给它赋值一个指定文件
	stubs := gostub.Stub(&configFile, "./test.toml")
	defer stubs.Reset()  // 测试结束后重置
	// 下面是测试的代码
	data, err := GetConfig()
	if err != nil {
		t.Fatal()
	}
	// 返回的data的内容就是上面/tmp/test.config文件的内容
	t.Logf("data:%s\n", data)
}

func TestShowNumber(t *testing.T) {
	stubs := gostub.Stub(&maxNum, 20)
	defer stubs.Reset()
	// 下面是一些测试的代码
	res := ShowNumber()
	if res != 20 {
		t.Fatal()
	}
}
```

执行单元测试，查看结果：

```bash
❯ go test -v
=== RUN   TestGetConfig
    app_test.go:18: data:blog="liwenzhou.com"
--- PASS: TestGetConfig (0.00s)
=== RUN   TestShowNumber
--- PASS: TestShowNumber (0.00s)
PASS
ok      golang-unit-test-demo/gostub_demo       0.012s
```

从上面的示例中我们可以看到，在单元测试中使用`gostub`可以很方便的对全局变量进行打桩，将其mock成我们预期的值从而进行测试。

## 总结

在日常工作开发中为代码编写单元测试时如何处理代码中的接口类型是十分常见的问题，本文介绍了如何使用`gomock`mock相关接口和如何使用`gostub`工具对全局变量进行打桩。

在下一篇中，我们将更进一步，详细介绍如何在编写单元测试时使用更全能的打桩工具——`monkey`。

## monkey

### 介绍

[monkey](https://github.com/bouk/monkey)是一个Go单元测试中十分常用的打桩工具，它在运行时通过汇编语言重写可执行文件，将目标函数或方法的实现跳转到桩实现，其原理类似于热补丁。

monkey库很强大，但是使用时需注意以下事项：

- monkey不支持内联函数，在测试的时候需要通过命令行参数`-gcflags=-l`关闭Go语言的内联优化。
- monkey不是线程安全的，所以不要把它用到并发的单元测试中。

### 安装

```bash
go get bou.ke/monkey
```

### 使用示例

假设你们公司中台提供了一个用户中心的库`varys`，使用这个库可以很方便的根据uid获取用户相关信息。但是当你编写代码的时候这个库还没实现，或者这个库要经过内网请求但你现在没这能力，这个时候要为`MyFunc`编写单元测试，就需要做一些mock工作。

```go
// func.go
func MyFunc(uid int64)string{
	u, err := varys.GetInfoByUID(uid)
	if err != nil {
		return "welcome"
	}
	// 这里是一些逻辑代码...

	return fmt.Sprintf("hello %s\n", u.Name)
}
```

我们使用`monkey`库对`varys.GetInfoByUID`进行打桩。

```go
// func_test.go
func TestMyFunc(t *testing.T) {
	// 对 varys.GetInfoByUID 进行打桩
	// 无论传入的uid是多少，都返回 &varys.UserInfo{Name: "liwenzhou"}, nil
	monkey.Patch(varys.GetInfoByUID, func(int64)(*varys.UserInfo, error) {
		return &varys.UserInfo{Name: "liwenzhou"}, nil
	})

	ret := MyFunc(123)
	if !strings.Contains(ret, "liwenzhou"){
		t.Fatal()
	}
}
```

执行单元测试：

> 注意：这里为防止内联优化添加了`-gcflags=-l`参数。

```bash
go test -run=TestMyFunc -v -gcflags=-l
```

输出：

```bash
=== RUN   TestMyFunc
--- PASS: TestMyFunc (0.00s)
PASS
ok      monkey_demo     0.009s
```

除了对函数进行mock外`monkey`也支持对方法进行mock。

```go
// method.go
type User struct {
	Name string
	Birthday string
}

// CalcAge 计算用户年龄
func (u *User) CalcAge() int {
	t, err := time.Parse("2006-01-02", u.Birthday)
	if err != nil {
		return -1
	}
	return int(time.Now().Sub(t).Hours()/24.0)/365
}

// GetInfo 获取用户相关信息
func (u *User) GetInfo()string{
	age := u.CalcAge()
	if age <= 0 {
		return fmt.Sprintf("%s很神秘，我们还不了解ta。", u.Name)
	}
	return fmt.Sprintf("%s今年%d岁了，ta是我们的朋友。", u.Name, age)
}
```

如果我们为`GetInfo`编写单元测试的时候`CalcAge`方法的功能还未完成，这个时候我们可以使用monkey进行打桩。

```go
// method_test.go
func TestUser_GetInfo(t *testing.T) {
	var u = &User{
		Name:     "q1mi",
		Birthday: "1990-12-20",
	}

	// 为对象方法打桩
	monkey.PatchInstanceMethod(reflect.TypeOf(u), "CalcAge", func(*User)int {
		return 18
	})

	ret := u.GetInfo()  // 内部调用u.CalcAge方法时会返回18
	if !strings.Contains(ret, "朋友"){
		t.Fatal()
	}
}
```

执行单元测试：

```bash
❯ go test -run=User -v
=== RUN   TestUser_GetInfo
--- PASS: TestUser_GetInfo (0.00s)
PASS
ok      monkey_demo     0.012s
```

`monkey`基本上能满足我们在单元测试中打桩的任何需求。

社区中还有一个参考monkey库实现的[gomonkey](https://github.com/agiledragon/gomonkey)库，原理和使用过程基本相似，这里就不再啰嗦了。除此之外社区里还有一些其他打桩工具如[GoStub](https://github.com/prashantv/gostub)（上一篇介绍过为全局变量打桩）等。

熟练使用各种打桩工具能够让我们更快速地编写合格的单元测试，为我们的软件保驾护航。

## 总结

本文通过外部函数依赖及内部方法依赖两个示例，介绍了如何使用`monkey`对依赖的函数和方法进行打桩。

在下一篇中，我们将介绍编写单元测试时常用的工具——`goconvey`。