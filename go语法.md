# go语法

## 变量及函数的导出

在GO中，我们约定，如果一个包的变量或者是函数是导出的，则需要大写开头。

## 变量的声明及初始化

1. 在GO中，变量的类型在变量名称之后。

2. 当多个变量同时声明并且类型相同时，可以省略前边的类型声明只在最后一个变量里声明

例如：

```go
x int, y int
```

可写成：

```go
x, y int
```

3. 可以使用`var`声明多个变量，全局变量和局部变量均可，并且变量类型在最后说明即可，但是一个`var`只能声明同一类型的一系列变量，在函数入参及返回值的变量声明中，不需要`var`前置声明
4. 可以使用`var`初始化一系列变量

```go
//带着类型说明
var i,j int = 1, 2
//不带着类型说明，可以同时初始化不同类型的变量
var c, python, java = true, false, "no!"
```

5. 在函数中，可以使用`:=`隐式的初始化变量，这样，省略了类型说明以及`var`，但是在函数外（也就是全局变量），不可以使用`:=`

## 函数的返回值

1. 在GO中，函数可以有多个返回值
2. 在GO中，函数的返回值，可以被声明并当作变量在函数中使用，并且当这个函数的`return`未指定时，返回被定义的函数返回值

```
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}

//结果：
//17 17
```

## GO的基本类型

```go
bool

string

int int8 int16 int32 int64
uint uint8 uint16 uint32 uint64 uintptr //int, uint, uintptr位数取决于电脑的位数（32/64bit）

byte //alias for uint8

rune //alias for int32
	 //represents a Unicode code point

float32 float64

complex64 complex128
```

## 类型转换

`T(v)`将变量`v`转换为`T`类型，在GO中，类型转换需要明确指出来，不存在隐式转换

## 变量类型推断

在GO中，如果变量初始化时未明确指定类型（使用了`:=`或`var =`），那么这个变量的类型会根据变量的值来进行推断，需要注意的是，这个变量的类型会优先被推断为`int`,`float64`,`complex128`,`string`类型（除非超过界限，如在一个32位系统定义超过32位能表示的数字，那么就会被推断为int64）

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
h := "hello"	  // string
```

## 常量的使用

在GO中，常量的定义需要使用`const`修饰，常量可以是`boolean`,`string`以及数字类型，但是不能使用`:=`语法，也不能带`var`，可以在函数内外定义

### 数字常量

数字常量可以存储高精度的值，可以通过初始化的值确定类型

详见链接：https://go.dev/tour/basics/16

## for循环语句

1. 与其他语言的区别，不需要小括号来包住循环条件，一定要大括号包住循环体

2. C语言的`while`语句在GO中使用`for`语句表示，如下

```go
func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

无限执行循环的表示

```go
func main() {
	for {
	}
}
```

## if语句

1. 与`for`语句相同，不需要小括号来包住循环条件，一定要大括号包住循环体
2. 与`for`语句相同，可以初始化**一个**变量，这个变量只在`if`语句中可以使用

```go
	if v := math.Pow(x, n); v < lim {
		return v
	}
```

## switch语句

1. 与C语言中`switch`语句不同的是，可以初始化**一个**变量，使用`;`相隔

```go
func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
```

2. 会在第一个满足的情况下停下来，不需要`break`

3. `switch`没指定对象时会默认为真，也就是会在第一个判断条件停下

## defer语句

1. `defer`语句的执行会在函数执行**return**语句后，但是检查会立马执行

```go
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}

// 运行结果：
// hello
// world
```

2. 因为`defer`语句所对应的函数是使用**栈**去存储的，所以，函数的执行遵循**后进先出**原则

更多相关资料:https://go.dev/blog/defer-panic-and-recover

## 指针

与C语言的指针使用方法差不多，只是不能进行指针偏移等指针运算

声明方法：

```go
var p *int
```

## 结构体

声明方法：

```go
type Vertex struct {
	X int
	Y int
}
```

使用方法：

```go
func main() {
	v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v.X)
}
```

当使用的是结构体指针时，如果需要访问结构体中的元素，一般要使用`*`去取结构体，但是GO简化了这个过程，也就是可以像使用结构体一样访问结构体元素，如下：

```go
func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9
	fmt.Println(v)
}
```

## Range

当需要使用`for`循环遍历`slice`或者`map`时，使用`range`格式

当使用`range`遍历一个`slice`时，会返回两个变量值，第一个代表`index`，第二个代表`slice`中`index`指向的元素`value`

可以使用`_`跳过`index`或者`value`，当你只想要`index`时，可以忽略第二个变量，如下所示

```go
for i := range pow
```

## 函数变量

函数也是变量，可以作为函数的参数以及函数的返回值，如下

```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

/* 打印为
   13
   5
   81
*/
```

## 函数闭包

一个函数是闭包。闭包指的是，一个函数A的变量引用了在这个函数之外的函数B的变量c（注意，A在B中被定义），A可以获取并改变c的值

需要注意的是，每个闭包函数都有**独立的**的闭包变量，如下

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}

/* 结果为
	0 0
    1 -2
    3 -6
    6 -12
    10 -20
    15 -30
    21 -42
    28 -56
    36 -72
    45 -90
*/
```

