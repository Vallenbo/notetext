runtime包提供和go运行时环境的互操作，如控制go程的函数。
func Caller(skip int) (pc uintptr, file string, line int, ok bool)
Caller报告当前go程调用栈所执行的函数的文件和行号信息。
实参skip为上溯的栈帧数。0表示Callers自身的调用栈，>0数字表示Callers所在的调用栈。
返回值为调用pc栈标识符、file文件名、该调用在文件中的行号line。如果无法获得信息，ok会被设为false。

func FuncForPC(pc uintptr) *Func
FuncForPC返回一个表示调用栈标识符pc对应的调用栈的*Func；如果该调用栈标识符没有对应的调用栈，函数会返回nil。每一个调用栈必然是对某个函数的调用。
func (f *Func) Name() string
Name返回该调用栈所调用的函数的名字。

func Base(path string) string
Base函数返回路径的最后一个元素。在提取元素前会求掉末尾的斜杠。如果路径是""，会返回"."；如果路径是只有一个斜杆构成，会返回"/"。