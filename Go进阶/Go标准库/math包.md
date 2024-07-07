# 常用标准库 - 数学计算

数学常量  

```Go
math.E	//自然对数的底，2.718281828459045
math.Pi	//圆周率，3.141592653589793
math.Phi	//黄金分割，长/短，1.618033988749895
math.MaxInt	//9223372036854775807
uint64(math.MaxUint)	//得先把MaxUint转成uint64才能输出，18446744073709551615
math.MaxFloat64	//1.7976931348623157e+308
math.SmallestNonzeroFloat64	//最小的非0且正的浮点数，5e-324
```

NaN(Not a Number)  

```Go
f := math.NaN()
math.IsNaN(f)
```

常用函数  

```Go
math.Ceil(1.1)	//向上取整，2
math.Floor(1.9)	//向下取整，1。 math.Floor(-1.9)=-2
math.Trunc(1.9)	//取整数部分，1
math.Modf(2.5)	//返回整数部分和小数部分，2  0.5
math.Abs(-2.6)	//绝对值，2.6
math.Max(4, 8)	//取二者的较大者，8
math.Min(4, 8)	//取二者的较小者，4
math.Mod(6.5, 3.5)	//x-Trunc(x/y)*y结果的正负号和x相同，3
math.Sqrt(9)		//开平方，3
math.Cbrt(9)		//开三次方，2.08008
```

三角函数  

```Go
math.Sin(1)
math.Cos(1)
math.Tan(1)
math.Tanh(1)
```

对数和指数  

```Go
math.Log(5)	//自然对数，1.60943
math.Log1p(4)	//等价于Log(1+p)，确保结果为正数，1.60943
math.Log10(100)	//以10为底数，取对数，2
math.Log2(8)	//以2为底数，取对数，3
math.Pow(3, 2)	//x^y，9
math.Pow10(2)	//10^x，100
math.Exp(2)	//e^x，7.389
```

rand随机数生成器  

```Go
//创建一个Rand
source := rand.NewSource(1) //seed相同的情况下，随机数生成器产生的数列是相同的
rander := rand.New(source)
for i := 0; i < 10; i++ {
    fmt.Printf("%d ", rander.Intn(100))
}
fmt.Println()
source.Seed(1) //必须重置一下Seed
rander2 := rand.New(source)
for i := 0; i < 10; i++ {
    fmt.Printf("%d ", rander2.Intn(100))
}
fmt.Println()

//使用全局Rand
rand.Seed(1)                //如果对两次运行没有一致性要求，可以不设seed
fmt.Println(rand.Int())     //随机生成一个整数
fmt.Println(rand.Float32()) //随机生成一个浮点数
fmt.Println(rand.Intn(100)) //100以内的随机整数，[0,100)
fmt.Println(rand.Perm(100)) //把[0,100)上的整数随机打乱
arr := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
rand.Shuffle(len(arr), func(i, j int) { //随机打乱一个给定的slice
    arr[i], arr[j] = arr[j], arr[i]
})
fmt.Println(arr)
```

