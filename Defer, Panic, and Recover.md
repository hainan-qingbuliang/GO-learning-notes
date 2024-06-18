# Defer, Panic, and Recover

## Defer

### 三大规则

1. 一个`defer`函数的参数在`defer`函数检查时就确定了，如下所示：

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
// 结果是打印0
```

2. 所有`defer`函数使用一个**栈**去存的，遵循**后进先出**原则，如下所示

```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
// 结果是打印3210
```

3. `defer`函数可以读取并给函数的有名返回值赋值，如下所示

```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}

// 这个函数会返回2
```

需要注意的是，`defer`、`return`的执行逻辑应该是，`return`最先执行，`return`负责将结果写入返回值中；接着`defer`开始执行一些收尾工作；最后函数携带当前返回值退出，来看下面一个经典例子

```go
// 无名返回值情况
package main

import (
        "fmt"
)

func main() {
        fmt.Println("return:", a()) // 打印结果为 return: 0
}

func a() int {
        var i int
        defer func() {
                i++
                fmt.Println("defer2:", i) // 打印结果为 defer: 2
        }()
        defer func() {
                i++
                fmt.Println("defer1:", i) // 打印结果为 defer: 1
        }()
        return i
}
```

```go
// 有名返回值情况
package main

import (
        "fmt"
)

func main() {
        fmt.Println("return:", b()) // 打印结果为 return: 2
}

func b() (i int) {
        defer func() {
                i++
                fmt.Println("defer2:", i) // 打印结果为 defer: 2
        }()
        defer func() {
                i++
                fmt.Println("defer1:", i) // 打印结果为 defer: 1
        }()
        return i // 或者直接 return 效果相同
}
```

可以从这两个代码中看出，`a`函数的返回值没有被提前声明，其值来自于其他变量的赋值，而`defer`修改的也是其他变量，并未真正影响到返回值本身；`b`函数的返回值被提前声明，也就意味着`defer`中是可以调用到真实返回值的，因此`defer`在`return`赋值返回值`i`之后，再一次的修改了`i`的值

## panic

`panic`是一个内置函数用于暂停控制流以及开始`panicking`。当一个函数`F`调用了`panic`，`F`函数的执行会暂停，所有在`F`函数中的`defer`函数会接着执行，然后`F`函数返回到调用函数`G`，对于调用者来说，此时，调用`F`函数就像是调用`panic`函数一样。整个进程会持续直到`gorouting`中所有存储在栈中的函数都被调用后（也就是`defer`函数），该进程崩溃。`panic`过程可由运行时错误产生或者直接调用`panic`函数产生



## Recover

`recover`是一个内置函数用于恢复处于`panicking`状态的`goroutine`，`recover`只有在`defer`函数的内部有效，在正常的执行中，调用`recover`函数会返回nil并且没有任何影响，如果当前`goroutine`处于`panicking`状态，调用`recover`会获取到传递给`panic`的值并返回，重新恢复控制流。

来看下面这个程序：

```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}

/*
函数执行结果
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.
*/

// 如果没有在defer函数中使用recover恢复控制流，则会是另外一个结果
/*
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
*/
```



