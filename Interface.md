# Interface

## interface介绍

一个接口就是由一系列方法签名定义的，如下

```go
type Abser interface {
	Abs() float64
}
```

一个类型实现一个接口通过实现这个接口中的所有方法，通过这个方法将接口的定义与其实现解耦，然后接口可以出现在任何包中无需提前设置，如下

```go
type I interface {
	M()
}

type T struct {
	S string
}

// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}
```

在底层，接口值可以被认为是一个值和具体类型的元组

```
(value, type)
```

## 一个底层值为空的接口变量

如果一个类型作为`inteface`中一个`method`接收者参数，那么，如果声明这个类型的一个变量，但是没有初始化，这个变量仍旧可以作为变量赋值给接口变量，只不过接口变量的底层值为`nil`，但是类型不为`nil`

。如下

```go
type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}

func main() {
	var i I

	var t *T
	i = t
	describe(i)
	i.M()

	i = &T{"hello"}
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}

/* 结果为
(<nil>, *main.T)
<nil>
(&{hello}, *main.T)
hello
*/

```

## 空指针接口

一个`nil`接口（也就是未初始化的接口）没有具体值以及类型（具体值和类型都为`nil`），调用这个接口中的方法会报run-time error

## 空方法接口

如果一个`interface`中没有任何`method`，那么这个接口可以被任何类型的变量初始化，因为所有类型都至少实现过**\`0\`**个interface中的方法，这样的技巧常用于处理不知道类型的值，如下

```go
func main() {
	var i interface{}
	describe(i)

	i = 42
	describe(i)

	i = "hello"
	describe(i)
}

func describe(i interface{}) {
	fmt.Printf("(%v, %T)\n", i, i)
}

/* 结果为
(<nil>, <nil>)
(42, int)
(hello, string)
*/
```

## 接口的类型推断

接口的类型推断让我们可以使用接口的底层具体值

```go
t := i.(T)
```

这个表达式判断接口值`i`拥有底层类型为`T`的值，并将这个值赋给变量`t`

在这个表达式中，如果`i`不拥有底层类型为`T`的值，那么会触发`panic`

可以下面这个更聪明的方法防止`panic`

```go
t, ok := i.(T)
```

在这个表达式中，如果接口值`i`拥有底层类型为`T`的值，并将这个值赋给变量`t`，`ok`的值为`true`；反之，`t`被赋给`t`对应类型的0值，`ok`的值为`false`，不会触发`panic`

## 接口的类型switch

可以对接口的类型进行`siwtch`选择，接口类型的`switch`跟其他`switch`的区别是：接口类型的`switch`是对接口底层值的类型进行判断选择执行，如下

```go
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}

```

其中，变量`v`的值为接口值底层类型所对应的值。
