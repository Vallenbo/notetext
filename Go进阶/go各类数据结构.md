[常见数据结构及算法 - 《数据结构和算法（Golang实现）》 - 书栈网 · BookStack](https://www.bookstack.cn/read/hunterhug-goa.c/algorithm-README.md)



# 可变长数组

```go
// Array 可变长数组
type Array struct {
    array []int      // 固定大小的数组，用满容量和满大小的切片来代替
    len   int        // 真正长度
    cap   int        // 容量
    lock  sync.Mutex // 为了并发安全使用的锁
}

// Make 新建一个可变长数组
func Make(len, cap int) *Array {
    s := new(Array)
    if len > cap {
        panic("len large than cap")
    }
    // 把切片当数组用
    array := make([]int, cap, cap)
    // 元数据
    s.array = array
    s.cap = cap
    s.len = 0
    return s
}

// Append 增加一个元素
func (a *Array) Append(element int) {
    // 并发锁
    a.lock.Lock()
    defer a.lock.Unlock()
    // 大小等于容量，表示没多余位置了
    if a.len == a.cap {
        // 没容量，数组要扩容，扩容到两倍
        newCap := 2 * a.len
        // 如果之前的容量为0，那么新容量为1
        if a.cap == 0 {
            newCap = 1
        }
        newArray := make([]int, newCap, newCap)
        // 把老数组的数据移动到新数组
        for k, v := range a.array {
            newArray[k] = v
        }
        // 替换数组
        a.array = newArray
        a.cap = newCap
    }
    // 把元素放在数组里
    a.array[a.len] = element
    // 真实长度+1
    a.len = a.len + 1
}

// AppendMany 增加多个元素
func (a *Array) AppendMany(element ...int) {
    for _, v := range element {
        a.Append(v)
    }
}

// Get 获取某个下标的元素
func (a *Array) Get(index int) int {
    // 越界了
    if a.len == 0 || index >= a.len {
        panic("index over len")
    }
    return a.array[index]
}

// 获取真实长度和容量
// Len 返回真实长度
func (a *Array) Len() int {
    return a.len
}
// Cap 返回容量
func (a *Array) Cap() int {
    return a.cap
}

// Print 辅助打印
func Print(array *Array) (result string) {
    result = "["
    for i := 0; i < array.Len(); i++ {
        // 第一个元素
        if i == 0 {
            result = fmt.Sprintf("%s%d", result, array.Get(i))
            continue
        }
        result = fmt.Sprintf("%s %d", result, array.Get(i))
    }
    result = result + "]"
    return
}

func main() {
    // 创建一个容量为3的动态数组
    a := Make(0, 3)
    fmt.Println("cap", a.Cap(), "len", a.Len(), "array:", Print(a))
    // 增加一个元素
    a.Append(10)
    fmt.Println("cap", a.Cap(), "len", a.Len(), "array:", Print(a))
    // 增加一个元素
    a.Append(9)
    fmt.Println("cap", a.Cap(), "len", a.Len(), "array:", Print(a))
    // 增加多个元素
    a.AppendMany(8, 7)
    fmt.Println("cap", a.Cap(), "len", a.Len(), "array:", Print(a))
}
// 输出
cap 3 len 0 array: []
cap 3 len 1 array: [10]
cap 3 len 2 array: [10 9]
cap 6 len 4 array: [10 9 8 7]
```

# 实现map





# 链表

```go
type LinkNode struct {
    Data     int64
    NextNode *LinkNode
}

func init(){
    // 新的节点
    node := new(LinkNode)
    node.Data = 2
    // 新的节点
    node1 := new(LinkNode)
    node1.Data = 3
    node.NextNode = node1 // node1 链接到 node 节点上
    // 新的节点
    node2 := new(LinkNode)
    node2.Data = 4
    node1.NextNode = node2 // node2 链接到 node1 节点上
}
func main() {

    // 按顺序打印数据
    nowNode := node
    for {
        if nowNode != nil {
            // 打印节点值
            fmt.Println(nowNode.Data)
            // 获取下一个节点
            nowNode = nowNode.NextNode
            continue
        }
        // 如果下一个节点为空，表示链表结束了
        break
    }
}
```

# 栈和队列

1. 栈：先进后出，先进队的数据最后才出来。在英文的意思里，`stack` 可以作为一叠的意思，这个排列是垂直的，你将一张纸放在另外一张纸上面，先放的纸肯定是最后才会被拿走，因为上面有一张纸挡住了它。
2. 队列：先进先出，先进队的数据先出来。在英文的意思里，`queue` 和现实世界的排队意思一样，这个排列是水平的，先排先得。

## 实现数组栈 ArrayStack

用切片实现栈。也可以用链表实现

```go
// 数组栈，后进先出
type ArrayStack struct {
    array []string   // 底层切片
    size  int        // 栈的元素数量
    lock  sync.Mutex // 为了并发安全使用的锁
}

// 入栈
func (stack *ArrayStack) Push(v string) {
    stack.lock.Lock()
    defer stack.lock.Unlock()
    // 放入切片中，后进的元素放在数组最后面
    stack.array = append(stack.array, v)
    // 栈中元素数量+1
    stack.size = stack.size + 1
}

// 出栈
func (stack *ArrayStack) Pop() string {
    stack.lock.Lock()
    defer stack.lock.Unlock()
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }
    // 栈顶元素
    v := stack.array[stack.size-1]
    // 切片收缩，但可能占用空间越来越大
    //stack.array = stack.array[0 : stack.size-1]
    // 创建新的数组，空间占用不会越来越大，但可能移动元素次数过多
    newArray := make([]string, stack.size-1, stack.size-1)
    for i := 0; i < stack.size-1; i++ {
        newArray[i] = stack.array[i]
    }
    stack.array = newArray
    // 栈中元素数量-1
    stack.size = stack.size - 1
    return v
}

// 获取栈顶元素
func (stack *ArrayStack) Peek() string {
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }
    // 栈顶元素值
    v := stack.array[stack.size-1]
    return v
}

// 获取栈大小和判定是否为空
// 栈大小
func (stack *ArrayStack) Size() int {
    return stack.size
}
// 栈是否为空
func (stack *ArrayStack) IsEmpty() bool {
    return stack.size == 0
}

func main() {
    arrayStack := new(ArrayStack)
    arrayStack.Push("cat")
    arrayStack.Push("dog")
    arrayStack.Push("hen")
    fmt.Println("size:", arrayStack.Size())
    fmt.Println("pop:", arrayStack.Pop())
    fmt.Println("pop:", arrayStack.Pop())
    fmt.Println("size:", arrayStack.Size())
    arrayStack.Push("drag")
    fmt.Println("pop:", arrayStack.Pop())
}
// 输出
size: 3
pop: hen
pop: dog
size: 1
pop: drag
```

## 链表栈 LinkStack

```go
// 链表栈，后进先出
type LinkStack struct {
    root *LinkNode  // 链表起点
    size int        // 栈的元素数量
    lock sync.Mutex // 为了并发安全使用的锁
}
// 链表节点
type LinkNode struct {
    Next  *LinkNode
    Value string
}

// 入栈
func (stack *LinkStack) Push(v string) {
    stack.lock.Lock()
    defer stack.lock.Unlock()
    // 如果栈顶为空，那么增加节点
    if stack.root == nil {
        stack.root = new(LinkNode)
        stack.root.Value = v
    } else {
        // 否则新元素插入链表的头部
        // 原来的链表
        preNode := stack.root
        // 新节点
        newNode := new(LinkNode)
        newNode.Value = v
        // 原来的链表链接到新元素后面
        newNode.Next = preNode
        // 将新节点放在头部
        stack.root = newNode
    }
    // 栈中元素数量+1
    stack.size = stack.size + 1
}

// 出栈
func (stack *LinkStack) Pop() string {
    stack.lock.Lock()
    defer stack.lock.Unlock()
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }
    // 顶部元素要出栈
    topNode := stack.root
    v := topNode.Value
    // 将顶部元素的后继链接链上
    stack.root = topNode.Next
    // 栈中元素数量-1
    stack.size = stack.size - 1
    return v
}

// 获取栈顶元素
func (stack *LinkStack) Peek() string {
    // 栈中元素已空
    if stack.size == 0 {
        panic("empty")
    }
    // 顶部元素值
    v := stack.root.Value
    return v
}

// 获取栈大小和判定是否为空
// 栈大小
func (stack *LinkStack) Size() int {
    return stack.size
}
// 栈是否为空
func (stack *LinkStack) IsEmpty() bool {
    return stack.size == 0
}

func main() {
    linkStack := new(LinkStack)
    linkStack.Push("cat")
    linkStack.Push("dog")
    linkStack.Push("hen")
    fmt.Println("size:", linkStack.Size())
    fmt.Println("pop:", linkStack.Pop())
    fmt.Println("pop:", linkStack.Pop())
    fmt.Println("size:", linkStack.Size())
    linkStack.Push("drag")
    fmt.Println("pop:", linkStack.Pop())
}
// 输出
size: 3
pop: hen
pop: dog
size: 1
pop: drag
```

## 实现数组队列 ArrayQueue

队列先进先出，和栈操作顺序相反，我们这里只实现入队，和出队操作，其他操作和栈一样。

```go
// 数组队列，先进先出
type ArrayQueue struct {
    array []string   // 底层切片
    size  int        // 队列的元素数量
    lock  sync.Mutex // 为了并发安全使用的锁
}

// 入队
func (queue *ArrayQueue) Add(v string) {
    queue.lock.Lock()
    defer queue.lock.Unlock()
    // 放入切片中，后进的元素放在数组最后面
    queue.array = append(queue.array, v)
    // 队中元素数量+1
    queue.size = queue.size + 1
}

// 出队
func (queue *ArrayQueue) Remove() string {
    queue.lock.Lock()
    defer queue.lock.Unlock()
    // 队中元素已空
    if queue.size == 0 {
        panic("empty")
    }
    // 队列最前面元素
    v := queue.array[0]
    /*    直接原位移动，但缩容后继的空间不会被释放
        for i := 1; i < queue.size; i++ {
            // 从第一位开始进行数据移动
            queue.array[i-1] = queue.array[i]
        }
        // 原数组缩容
        queue.array = queue.array[0 : queue.size-1]
    */
    // 创建新的数组，移动次数过多
    newArray := make([]string, queue.size-1, queue.size-1)
    for i := 1; i < queue.size; i++ {
        // 从老数组的第一位开始进行数据移动
        newArray[i-1] = queue.array[i]
    }
    queue.array = newArray
    // 队中元素数量-1
    queue.size = queue.size - 1
    return v
}
```



## 实现链表队列 LinkQueue

```go
// 链表队列，先进先出
type LinkQueue struct {
    root *LinkNode  // 链表起点
    size int        // 队列的元素数量
    lock sync.Mutex // 为了并发安全使用的锁
}
// 链表节点
type LinkNode struct {
    Next  *LinkNode
    Value string
}

// 入队
func (queue *LinkQueue) Add(v string) {
    queue.lock.Lock()
    defer queue.lock.Unlock()
    // 如果栈顶为空，那么增加节点
    if queue.root == nil {
        queue.root = new(LinkNode)
        queue.root.Value = v
    } else {
        // 否则新元素插入链表的末尾
        // 新节点
        newNode := new(LinkNode)
        newNode.Value = v
        // 一直遍历到链表尾部
        nowNode := queue.root
        for nowNode.Next != nil {
            nowNode = nowNode.Next
        }
        // 新节点放在链表尾部
        nowNode.Next = newNode
    }
    // 队中元素数量+1
    queue.size = queue.size + 1
}

// 出队
func (queue *LinkQueue) Remove() string {
    queue.lock.Lock()
    defer queue.lock.Unlock()
    // 队中元素已空
    if queue.size == 0 {
        panic("empty")
    }
    // 顶部元素要出队
    topNode := queue.root
    v := topNode.Value
    // 将顶部元素的后继链接链上
    queue.root = topNode.Next
    // 队中元素数量-1
    queue.size = queue.size - 1
    return v
}



```







# 树



```go
type TreeNode1 struct {
	Data  string    // 节点用来存放数据
	Left  *TreeNode1 // 左子树
	Right *TreeNode1 // 右字树
}

// a b c d e f
func Test_treeQueue(t *testing.T) {
	root := &TreeNode1{Data: "A"}
	root.Left = &TreeNode1{Data: "B"}
	root.Right = &TreeNode1{Data: "C"}
	root.Left.Left = &TreeNode1{Data: "D"}
	root.Left.Right = &TreeNode1{Data: "E"}
	root.Right.Left = &TreeNode1{Data: "F"}

	if root == nil {
		return
	}

	queue := []*TreeNode1{root}
	var length int
	for len(queue) > 0 { // 先序遍历
		length++
		element := queue[0]
		println(element.Data)
		queue = queue[1:]
		if element.Left != nil {
			queue = append(queue, element.Left)
		}
		if element.Right != nil {
			queue = append(queue, element.Right)
		}
	}
	println(length)
}
```

