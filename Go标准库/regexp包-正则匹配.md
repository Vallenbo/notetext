# regexp 包

## 单字符：

```
        .              任意字符（标志s==true时还包括换行符）
        [xyz]          字符族
        [^xyz]         反向字符族（反向匹配）
        \d             Perl预定义字符族
        \D             反向Perl预定义字符族
        [:alpha:]      ASCII字符族
        [:^alpha:]     反向ASCII字符族
        \pN            Unicode字符族（单字符名），参见unicode包
        \PN            反向Unicode字符族（单字符名）
        \p{Greek}      Unicode字符族（完整字符名）
        \P{Greek}      反向Unicode字符族（完整字符名）
```

regexp.Compile(`[^a-z]+`) //匹配多个非a-z的字符

regexp.Compile(`[^a-z0-9]+`) //匹配多个非a-z和0-9的字符

## 结合：

```
        xy             匹配x后接着匹配y
        x|y            匹配x或y（优先匹配x）
```

## 重复：

```
        x*             重复>=0次匹配x，越多越好（优先重复匹配x）
        x+             重复>=1次匹配x，越多越好（优先重复匹配x）
        x?             0或1次匹配x，优先1次
        x{n,m}         n到m次匹配x，越多越好（优先重复匹配x）
        x{n,}          重复>=n次匹配x，越多越好（优先重复匹配x）
        x{n}           重复n次匹配x
        x*?            重复>=0次匹配x，越少越好（优先跳出重复）
        x+?            重复>=1次匹配x，越少越好（优先跳出重复）
        x??            0或1次匹配x，优先0次
        x{n,m}?        n到m次匹配x，越少越好（优先跳出重复）
        x{n,}?         重复>=n次匹配x，越少越好（优先跳出重复）
        x{n}?          重复n次匹配x
```

实现的限制：计数格式x{n}等（不包括x*等格式）中n最大值1000。负数或者显式出现的过大的值会导致解析错误，返回ErrInvalidRepeatSize。

## 分组：

```
        (re)           编号的捕获分组
        (?P<name>re)   命名并编号的捕获分组
        (?:re)         不捕获的分组
        (?flags)       设置当前所在分组的标志，不捕获也不匹配
        (?flags:re)    设置re段的标志，不捕获的分组
```

标志的语法为xyz（设置）、-xyz（清楚）、xy-z（设置xy，清楚z），标志如下：

```
        I              大小写敏感（默认关闭）
        m              ^和$在匹配文本开始和结尾之外，还可以匹配行首和行尾（默认开启）
        s              让.可以匹配\n（默认关闭）
        U              非贪婪的：交换x*和x*?、x+和x+?……的含义（默认关闭）
```

## 边界匹配：

```
        ^              匹配文本开始，标志m为真时，还匹配行首
        $              匹配文本结尾，标志m为真时，还匹配行尾
        \A             匹配文本开始
        \b             单词边界（一边字符属于\w，另一边为文首、文尾、行首、行尾或属于\W）
        \B             非单词边界
        \z             匹配文本结尾
```

## 转义序列：

```
        \a             响铃符（\007）
        \f             换纸符（\014）
        \t             水平制表符（\011）
        \n             换行符（\012）
        \r             回车符（\015）
        \v             垂直制表符（\013）
        \123           八进制表示的字符码（最多三个数字）
        \x7F           十六进制表示的字符码（必须两个数字）
        \x{10FFFF}     十六进制表示的字符码
        \*             字面值'*'
        \Q...\E        反斜线后面的字符的字面值
```

## 字符族

（预定义字符族之外，方括号内部）的语法：

```
        x              单个字符
        A-Z            字符范围（方括号内部才可以用）
        \d             Perl字符族
        [:foo:]        ASCII字符族
        \pF            单字符名的Unicode字符族
        \p{Foo}        完整字符名的Unicode字符族
```

## 常用方法

参数说明：

pattern:要查找的正则表达式（想要匹配的内容）
matched:返回是否找到匹配项，布尔类型
re *Regexp:go正则匹配实例对象

1、func Match(pattern string, b []byte)(matchd bool, err error)
匹配检查文本正则表达式是否与字节片匹配

```go
func main() {
	matched, err := regexp.Match(`foo.*`, []byte(`seafood`))
	fmt.Println(matched, err)
	matched, err = regexp.Match(`bar.*`, []byte(`seafood`))
	fmt.Println(matched, err)
	matched, err = regexp.Match(`a(b`, []byte(`seafood`))
	fmt.Println(matched, err)
}
```

2、func MatchString(pattern string, s string) (matched bool, err error)
检查文本正则表达式是否匹配字符串

```go
func main() {
	matched, err := regexp.MatchString("foo.*", "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString("bar.*", "seafood")
	fmt.Println(matched, err)
	matched, err = regexp.MatchString("a(b", "seafood")
	fmt.Println(matched, err)
}
```

但同时我们能看到编辑器有提示，说明直接用 `MatchString` 匹配字符串会影响性能。：

所以我们考虑用 `Compile()` 或者 `MustCompile()`创建一个编译好的正则表达式对象，然后再来进行模式匹配。

3、Compile 函数

`Compile` 函数解析正则表达式，如果成功，则返回可用于匹配文本的 Regexp 对象。编译的正则表达式产生更快的代码。假如正则表达式非法，那么 `Compile()` 方法回返回 error ,而 `MustCompile()` 编译非法正则表达式时不会返回 error ，而是返回 panic 。

先来看 `Compile` 函数：

```go
func main() {
	words := [...]string{"Seven", "even", "Maven", "Amen", "eleven"}
	re, err := regexp.Compile(".even")
	if err != nil {
		log.Fatal(err)
	}

	for _, word := range words {
		found := re.MatchString(word)
		if found {
			fmt.Printf("%s matches\n", word)
		} else {
			fmt.Printf("%s does not match\n", word)
		}
	}
}
```

在代码示例中，我们使用了编译的正则表达式。

```go
re, err := regexp.Compile(".even")
```

即使用 `Compile` 编译正则表达式。然后在返回的正则表达式对象上调用 `MatchString` 函数：

```go
found := re.MatchString(word)
```

运行程序，能看到同样的代码：

```sql
Seven matches
even does not match
Maven does not match
Amen does not match
eleven matches
```

4、func MustCompile(str string) *Regexp

MustCompile 就像编译，但如果表达式不能被解析就会发生混乱。它简化了保存已编译正则表达式的全局变量的安全初始化。

```go
func main() {
	// Compile the expression once, usually at init time.
	// Use raw strings to avoid having to quote the backslashes.
	var validID = regexp.MustCompile(`^[a-z]+\[[0-9]+\]$`)
	fmt.Println(validID.MatchString("adam[23]"))
	fmt.Println(validID.MatchString("eve[7]"))
	fmt.Println(validID.MatchString("Job[48]"))
	fmt.Println(validID.MatchString("snakey"))
}
```
11. func (re *Regexp) MatchString(s string) bool
    MatchString 报告 Regexp 是否匹配字符串 s 。

```go
func main() {
	re := regexp.MustCompile("(gopher){2}")
	fmt.Println(re.MatchString("gopher"))
	fmt.Println(re.MatchString("gophergopher"))
	fmt.Println(re.MatchString("gophergophergopher"))
}
```

12. func (re *Regexp) ReplaceAllString(src, repl string) string
    ReplaceAllString 返回 src 的副本，用替换字符串 repl 替换正则表达式的匹配项。在内部 repl 中，$ 符号被解释为在 Expand 中，因此例如 $ 1 代表第一个子匹配的文本。

```go
func main() {
	re := regexp.MustCompile("a(x*)b")
	fmt.Println(re.ReplaceAllString("-ab-axxb-", "T")) //-T-T-
	fmt.Println(re.ReplaceAllString("-ab-axxb-", "$1")) //--xx-
	fmt.Println(re.ReplaceAllString("-ab-axxb-", "$1W")) //---
	fmt.Println(re.ReplaceAllString("-ab-axxb-", "${1}W")) //-W-xxW
}
```

5、func (re *Regexp) Find(b []byte) []byte
查找返回一个片段，其中包含正则表达式b中最左边匹配的文本。返回值 nil 表示不匹配。

```go
func main() {
	re := regexp.MustCompile(`foo.?`)
	fmt.Printf("%q\n", re.Find([]byte(`seafood fool`)))
}
```

6、func (re *Regexp) FindIndex(b []byte) (loc []int)
FindIndex 返回一个两个元素的整数切片，用于定义正则表达式 b 中最左边匹配的位置。匹配本身在 b [loc0：loc1]。返回值 nil 表示不匹配。

```go
func main() {
	content := []byte(`
	# comment line
	option1: value1
	option2: value2
`)
	// Regex pattern captures "key: value" pair from the content.
	pattern := regexp.MustCompile(`(?m)(?P<key>\w+):\s+(?P<value>\w+)$`)loc := 			pattern.FindIndex(content)
	fmt.Println(loc)
	fmt.Println(string(content[loc[0]:loc[1]]))
}
```

6. func (re *Regexp) FindAll(b []byte, n int) [][]byte
FindAll 是 Find 的所有版本；它返回表达式的所有连续匹配的一部分，如包注释中的 ‘All’ 描述所定义。返回值 nil 表示不匹配。

```go
func main() {
	re := regexp.MustCompile(`foo.?`)
	fmt.Printf("%q\n", re.FindAll([]byte(`seafood fool`), -1))
}
```

7. func (re *Regexp) FindString(s string, n int) []string
FindString 返回一个字符串，其中包含正则表达式的s中最左边匹配的文本。如果不匹配，则返回值为空字符串，但如果正则表达式成功匹配空字符串，则它也将为空。如果有必要区分这些情况，请使用 FindStringIndex 或 FindStringSubmatch 。

```go
func main() {
	re := regexp.MustCompile(`foo.?`)
	fmt.Printf("%q\n", re.FindString("seafood fool"))
	fmt.Printf("%q\n", re.FindString("meat"))
}
```

8. func (re *Regexp) FindStringIndex(s string) (loc []int)
FindStringIndex 返回一个整数的两个元素切片，用于定义正则表达式的 s 中最左边匹配的位置。匹配本身在 s [loc0：loc1] 。返回值 nil 表示不匹配。

```go
func main() {
	re := regexp.MustCompile("ab?")
	fmt.Println(re.FindStringIndex("tablett"))
	fmt.Println(re.FindStringIndex("foo") == nil)
}
```


9. func (re *Regexp) FindAllString(s string, n int) []string
FindAllString 是 FindString 的 'All’版本；它返回表达式的所有连续匹配的一部分，如包注释中的 ‘All’ 描述所定义。返回值 nil 表示不匹配。

```go
func main() {
	re := regexp.MustCompile("a.")
	fmt.Println(re.FindAllString("paranormal", -1))
	fmt.Println(re.FindAllString("paranormal", 2))
	fmt.Println(re.FindAllString("graal", -1))
	fmt.Println(re.FindAllString("none", -1))
}
```

10. func (re *Regexp) FindAllStringSubmatch(s string, n int) [][]string
FindAllStringSubmatch 是 FindStringSubmatch 的 ‘All’ 版本; 它返回表达式的所有连续匹配的一部分，如包注释中的 ‘All’ 描述所定义。返回值 nil 表示不匹配。

```go
func main() {
	re := regexp.MustCompile("a(x*)b")
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-ab-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-axxb-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-ab-axb-", -1))
	fmt.Printf("%q\n", re.FindAllStringSubmatch("-axxb-ab-", -1))
}
```

11. 
