# Slice，Array

## Array和Slice

定义方法：

1. array

```go
[3]bool{true, true, false}
```

`[n]T`，`n`作为数组类型的一部分，不能省略

2. slice

```go
[]bool{true, true, false}
```

`[]T`，声明时不需要指定大小

快捷表示方法：

```go
// 以下表达式是等价的
a[0:10]
a[:10]
a[0:]
a[:]
```

## Slice

可以认为，`slice`也是一个结构体，它的组成成员有`length`以及`capacity`，其中`length`表示这个`slice`中包含元素的个数，在赋值的时候确定；`capacity`表示`slice`引用的`array`包含元素的个数，需要注意的是，这个值从`slice`表示`array`的第一个元素开始算起

## 空Slice

例子：

```go
var s []int
```

一个空`slice`有`length`以及`capacity`，并且都为0，但是没有引用的`array`

## 内置函数make

内置`make`函数会创建一个全为0值的`array`，并且返回一个引用这个`array`的`slice`

## Slice元素类型

slice元素的类型不受限制，甚至可以包含slice，如下所示

```go
    board := [][]string{
            []string{"_", "_", "_"},
            []string{"_", "_", "_"},
            []string{"_", "_", "_"},
    }

	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

```

## 内置函数append

函数定义

```go
func append(s []T, vs ...T) []T
```

第一个参数被`append`的`slice`，后面的参数全是需要`append`的元素，需要注意的是类型元素类型必须与`slice`中元素类型相同。

特殊的是，可以`append`一个`string`类型的变量在一个`[]byte`类型的slice中，如下

```go
slice = append([]byte("hello "), "world"...)
```

假设`s`拥有充足的`capacity`可以装下`append`的元素，那就会在原`slice`中添加这些元素；如果`capacity`容量不足，就会重新分配一个`array`以满足容量需求