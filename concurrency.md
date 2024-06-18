# Goroutine

`goroutine`，中文名`协程`，是一种被`Go runtime`管理的轻量级线程，格式如下

```go
go f(x,y,z)
```

`f`、`x`、`y` 和 `z` 的**计算**发生在当前 goroutine 中，**f 的执行**发生在新 goroutine 中。

goroutine运行在相同的地址空间中，所以访问共享内存必须是同步的。

# Channel

`channel`是一种类型化的**管道**，可以通过管道操作符`<-`发送或者接收数据，如下，箭头指的方向就是数据流往的方向。

```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```

管道可以通过`make`函数创建，如下

```go
ch := make(chan int)
```

默认情况下，发送和接收被阻塞直到另一端准备好，这样就允许`goroutine`同步而不需要**锁**或者**条件变量**

## Buffered Channel

管道是可以有缓冲区的，可以通过在`make`函数中提供缓冲区长度来初始化一个缓冲区管道，如下

```go
ch := make(chan int, 100)
```

当缓冲区满的时候，发送数据给缓冲区管道的行为被阻塞；当缓冲区空的时候，缓冲区管道接收数据的行为被阻塞

## Range and Close

可以使用`close`函数关闭一个发送管道，表明没有更多的数据将被发送了，接收者可以使用额外的参数接收管道是否关闭的标识，如下

```go
v, ok := <-ch
```

当`ok`的值为**false**的时候，表明没有更多的数据给接收者了并且发送管道已经关闭了

需要注意的是，`channel`不像是file，不许要经常关闭它们，只有当需要告诉接收者将没有数据继续被发送后，才需要关闭这个管道，如`range`循环，如下

```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

当`c`被关闭后，`range`循环退出

## select

`select`表达式让一个`goroutine`等待在多个通信操作，`select`被阻塞直到其中一个例子能运行时，当多个例子都能跑时，它会随机挑选**一个**执行看，如下

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	select {
	case c <- x:
		x, y = y, x+y
	case <-quit:
		fmt.Println("quit")
		return
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```



跟`switch`表达式一样，可以使用`default`来覆盖当没有任何一个例子满足的情况，如下

```go
select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```



# sync.Mutex

GO也提供了类似于互斥锁的两个方法，**Lock**和**Unlock**，它保证了同一时间只有一个`goroutine`可以访问一个指定变量来避免冲突。可以使用`defer`保证**Lock**的变量到最后会**Unlock**