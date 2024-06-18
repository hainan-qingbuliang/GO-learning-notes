# Methods

## Methods介绍

GO中没有**类**的概念，但是可以给**类型**定义**方法**，结构如下

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

## methods和function的区别

一个**method**只是一个带着接收者参数的函数，上边的`Abs`使用正常的函数写法如下

```go
type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```

## methods延伸

1. 可以不使用`struct`类型作为`method`的接收者参数，但接收者参数的类型必须在该`package`中定义，就算是内置类型例如`int`也不行
2. 接收者参数可以是指针（但是指针指向的元素不能是指针），并且是指针的话，可以改变指针指向的变量的值，具体如下

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}

// 结果为50
// 如果Scale的接收者参数类型不是指针，结果为5
```

3. 如果`method`的接收者参数是指针，那么，下面这个表达式是合法的

```go
// Scale是一个接收者参数为指针的method
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK

// ScaleFunc的第一个参数为Vertex类型的指针
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```

也就是说，GO解释器会把`v.Scale(5)`解释为`(&v).Scale(5)`，但是这在函数中是行不通的。

同时，反过来也是同理，如下

```go
// Scale是一个接收者参数为Vertex类型的变量的method
var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK

// AbsFunc第一个参数为Vertex类型的变量
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!
```

3. `method`接收者参数选用指针的场景

   1. 当需要修改接收者参数内部变量的情况
   2. 防止在多个`method`之间的多余copy行为

   如下

   ```go
   func (v *Vertex) Scale(f float64) {
   	v.X = v.X * f
   	v.Y = v.Y * f
   }
   
   func (v *Vertex) Abs() float64 {
   	return math.Sqrt(v.X*v.X + v.Y*v.Y)
   }
   
   func main() {
   	v := &Vertex{3, 4}
   	fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
   	v.Scale(5)
   	fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
   }
   ```
