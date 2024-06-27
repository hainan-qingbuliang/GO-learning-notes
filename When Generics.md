# 什么时候适合用泛型

原文章的主要目的是**在你不知道是否要使用泛型时，给一些建议以及规则。**

## 当使用GO语言特有的容器类型时

当我们去定义一个函数，这个函数使用了GO语言特有的几种容器类型：例如**slice**，**map**，**channel**。当函数中使用到这几种容器类型并且函数并没有对容器元素的类型进行限定，就可以考虑使用泛型。

如下。

```go
// MapKeys returns a slice of all the keys in m.
// The keys are not returned in any particular order.
func MapKeys[Key comparable, Val any](m map[Key]Val) []Key {
    s := make([]Key, 0, len(m))
    for k := range m {
        s = append(s, k)
    }
    return s
}
```

这个代码例子中，并没有对`key`以及`val`的类型进行限定或使用，所以这里使用泛型是比较好的想法。

## 通用数据结构体

当我们需要类似于`slice`或者`map`这样的通用数据结构体但不是GO的**内置类型**时（例如**链表**或者**二叉树**），就可以使用。

编程中需要这样的通用数据结构体来做两件事：使用具体的元素类型定义它们或者使用一个接口类型

- 使用类型参数替换数据结构体中的具体类型可以产生一个更加通用的数据机构体，这个结构体可以在/被其他程序使用。来看下面一段代码。

- 可以使用接口类型替换类型参数，这样可以使数据存储更加高效，节省内存空间，同时，这样可以使代码避免**类型推断**，并且在**build time**时就完全经过类型检查。

来看下面一段代码

```go
// Tree is a binary tree.
type Tree[T any] struct {
    cmp  func(T, T) int
    root *node[T]
}

// A node in a Tree.
type node[T any] struct {
    left, right  *node[T]
    val          T
}

// find returns a pointer to the node containing val,
// or, if val is not present, a pointer to where it
// would be placed if added.
func (bt *Tree[T]) find(val T) **node[T] {
    pl := &bt.root
    for *pl != nil {
        switch cmp := bt.cmp(val, (*pl).val); {
        case cmp < 0:
            pl = &(*pl).left
        case cmp > 0:
            pl = &(*pl).right
        default:
            return pl
        }
    }
    return pl
}

// Insert inserts val into bt if not already there,
// and reports whether it was inserted.
func (bt *Tree[T]) Insert(val T) bool {
    pl := bt.find(val)
    if *pl != nil {
        return false
    }
    *pl = &node[T]{val: val}
    return true
}
```

这段代码中，定义了一个二叉树，这个二叉树中的每一个节点的值都是**类型参数T**，当这颗二叉树被具体类型初始化时，这个具体类型的值会被直接存储在节点中。

这是一种合理使用类型参数的情景，因为这里使用了**树**这样的数据结构体，并且代码中并不对依赖树中节点值的类型进行限定。不会被存储成接口类型。

这个**树**结构体并不知道如何去比较类型`T`的值，而是通过传递一个比较函数去完成，因此，元素类型`T`的具体类型并不重要。

## 对于类型参数，最好使用函数而不是方法

上面的示例也说明了一条规则：当你需要一些比较功能时，最好使用函数而不是方法。

我们可以定义上述**二叉树**数据结构体，不使用比较函数，而使用比较方法。那么，对于初始化的具体类型，我们都要拥有一个对应的比较方法，这就会导致类似于`int`这样的内置基础类型，需要定义成类似于`Interger`这样的类型才能去定义对应的比较方法（因为GO中method的接收者参数必须在该`package`中定义，内置类型也不行），这样做是相当麻烦的。而使用了类型参数比较函数是比较好的，当初始化了具体类型后，可以根据需要编写对应的比较函数并且传递进去。

当一颗**树**的元素类型已经存在了比较方法，我们可以很简单的使用类似于`ElementType.Compare`作为比较函数传递进去。

换句话说，GO中将一个方法转换为一个函数要比向一个类型添加一个方法简单得多，所以对于通用数据结构体，最好使用函数和不是方法。

## 基于Interface的泛型

泛型时实现多台的一种方式，多台可以划分为三种

1. `parametric polymorphism` 参数化多态
2. `Ad-hoc polymorphism` 特设多态

3. `Subtype polymorphism` 子类型多态

GO的泛型实现的是参数化多态，可分为**函数的类型参数**、**类型的类型参数**两种。

### 基本原则

GO泛型编程的基本原则是：泛型代码只能使用参数类型**已经实现**的操作。例子：

```go
func Stringify[T any](s []T) (ret []string) {
    for _, v := range s {
        ret = append(ret, v.String()) //编译失败，不能确定v是否有String()方法
    }
    return ret
}
```

```go
type Stringer interface {
    String() string
}

// 限制类型T必须实现Stringer接口
func Stringify[T Stringer](s []T) (ret []string) {
    for _, v := range s {
        ret = append(ret, v.String()) // OK
    }
    return ret
}
```

第二个例子就是典型的基于Interface的泛型

### 空指针遗毒

使用GO的泛型需要注意空指针问题：

1. 方法接收者是类型或类型指针的区别，其中之一就是方法接收者是类型指针才可以修改非指针的成员变量
2. make类型指针的slice默认不初始化成员，全部都是`nil`

来看一个有趣的例子：

需求是完成一个`FromStrings` 泛型函数，可以从字符串转换为各种类型

```go
type Setter interface {
    Set(string)
}

func FromStrings[T Setter](s []string) []T {
    result := make([]T, len(s))
    for i, v := range s {
        result[i].Set(v)
    }
    return result
}
```

但是调用的时候发现无效

```go
type Settable int

func (p *Settable) Set(s string) {
    n, _ := strconv.Atoi(s)
    *p = Settable(n)
}

func main() {
    // 方式①
    // 编译失败: Settable 没有实现Set方法
    nums := FromStrings[Settable]([]string{"1", "2"})

    // 方式②
    // 运行panic: *Settable 实现了Set方法，但使用make创建的是一个指针数组，这个数组中的每一个
    // 元素都是nil，result[i].Set(v)会调用失败
    nums := FromStrings[*Settable]([]string{"1", "2"})
}
```

可以总结一下，`*Settable`实现了接口的限制，也就是`Set`方法，但是`FromStrings`函数需要使用非指针类型的类型参数`Settable`才能正确运行。我们需要改变`FromStrings`的写法，让它可以使用`Settable`作为参数并且能调用其指针的方法。

我们需要将两种类型都作为`FromStrings`的类型参数，解决方案如下

```go
// Setter2 is a type constraint that requires that the type
// implement a Set method that sets the value from a string,
// and also requires that the type be a pointer to its type parameter.
// Setter2是一个类型约束，如果一个类型需要实现这个接口，就需要实现一个接口定义的Set(string)方法
// 并且需要这个类型是指向其类型参数的指针
type Setter2[B any] interface {
	Set(string)
	*B // non-interface type constraint element
}

// FromStrings2 takes a slice of strings and returns a slice of T,
// calling the Set method to set each returned value.
//
// We use two different type parameters so that we can return
// a slice of type T but call methods on *T aka PT.
// The Setter2 constraint ensures that PT is a pointer to T.
// FromStrings2是一个接收元素类型为T的slice作为参数并且返回也为元素类型为T的函数
// 我们使用两个不同的类型参数，这样可以返回一个元素类型为T的slice并且调用*T(PT)的方法
// Setter2限制了PT是一个指向T的指针
func FromStrings2[T any, PT Setter2[T]](s []string) []T {
	result := make([]T, len(s))
	for i, v := range s {
		// The type of &result[i] is *T which is in the type set
		// of Setter2, so we can convert it to PT.
		p := PT(&result[i])
		// PT has a Set method.
		p.Set(v)
	}
	return result
}
```

可以这样调用`FromString2`函数：

```go
func F2() {
	// FromStrings2 takes two type parameters.
	// The second parameter must be a pointer to the first.
	// Settable is as above.
    // FromStrings2使用了两个类型参数，第二个类型参数必须是指向第一个的指针
    // Settable定义如上不变，并且Settale如上所示实现了Set方法
	nums := FromStrings2[Settable, *Settable]([]string{"1", "2"})
	// Now nums is []Settable{1, 2}.
	...
}
```

**约束类型推断**可以帮我们简化写法，如下

```go
func F3() {
	// Here we just pass one type argument.
    // 这样我们只需要传递一个类型参数
	nums := FromStrings2[Settable]([]string{"1", "2"})
	// Now nums is []Settable{1, 2}.
	...
}
```

## 实现一个共同的方法

另一个例子是当不同的类型需要实现相同方法并且这些方法的内部构造都相同时，**类型参数**就可以起大作用。

举一个例子，考虑标准库接口`sort.Interface`，它需要一个类型实现三个方法：`Len`，`Swap`以及`Less`，下面是一个内部包含任意类型切片的`SliceFn`类型实现`sort.Interface`接口的例子

```go
// SliceFn implements sort.Interface for a slice of T.
type SliceFn[T any] struct {
    s    []T
    less func(T, T) bool
}

func (s SliceFn[T]) Len() int {
    return len(s.s)
}
func (s SliceFn[T]) Swap(i, j int) {
    s.s[i], s.s[j] = s.s[j], s.s[i]
}
func (s SliceFn[T]) Less(i, j int) bool {
    return s.less(s.s[i], s.s[j])
}
```

对于任意类型的切片，`Len`以及`Swap`方法保持一致，`Less`方法需要一个比较函数，也就是`SliceFn`结构体定义的函数部分，在定义这个结构体变量时提供。

下面是如何通过传递比较函数并使用`SliceFn`对任意类型的切片进行排序的例子：

```go
// SortFn sorts s in place using a comparison function.
func SortFn[T any](s []T, less func(T, T) bool) {
    sort.Sort(SliceFn[T]{s, less})
}
```

# 什么时候不适合使用泛型

## 不要使用类型参数替换接口类型

Go语言拥有接口类型，接口类型允许一种**泛型编程**。

例如，广泛使用的`io.Reader`提供了从包含信息（如文件）或者产生信息（如随机数生成器）的任意值中读取数据的通用机制。如果你只需要调用某个值的方法，只需要使用接口就好了，没必要使用类型参数，这样会更加易读，有效且高效。

例如，不要做以下的变换

```go
func ReadSome(r io.Reader) ([]byte, error)

func ReadSome[T io.Reader](r T) ([]byte, error)
```

它们的执行是一致的，而不使用类型参数会显得更简单。

**通常情况下使用类型参数不会比使用接口类型运行更快。**

## 不要使用类型参数如果方法的实现有差异

当决定使用类型参数还是接口类型时，考虑方法的内部实现。如果需要使用的类型的方法的内部实现是一致的，那么使用类型参数；反之，如果不一致，则使用接口类型，不同的类型实现不用的方法。

## 在合适的地方使用反射

GO有**运行时反射**机制。反射允许一种**泛型编程**，它允许你编写在任何类型都适用的代码。

如果一些操作必须支持某些类型，这些类型没有对应方法（用不了接口）并且这个操作对于任何类型的实现是不一致的，使用反射。

一个例子是`encoding/json`，我们不想要求编码的每种类型都必须有一个`MarshalJSON`方法，所以不能使用接口类型。同时，对接口类型进行编码与对结构体类型进行编码完全不同，所以也不该使用类型参数。相反，该包使用反射。

## 参考链接：

- https://go.dev/blog/when-generics
- https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md
