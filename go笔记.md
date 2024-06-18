# go笔记

## go语言的函数定义

```go
func Hello(name string) string
```

`Hello`是函数名称，`name`表示参数名称，第一个`string`表示入参类型，第二个`string`表示返回值类型

## 创建模块

当需要创建一个模块时，使用以下命令

```shell
go mod init example.com/hello
```

它会在当前目录下生成一个go.mod文件，当你刚创建好时，它只会有当前模块的信息以及go的版本

## 调用其它模块

接上回，如果你希望在一个模块中调用另一个模块，例如希望在`example.com/hello`模块中调用`example.com/greetings`模块，首先，需要在`example.com/hello`模块的目录下（也就是go.mod目录下），使用以下命令：

```shell
go mod edit -replace example.com/greetings=../greetings
```

这是一个替换提示，然后使用命令：

```shell
go mod tidy
```

输入完后，打开`example.com/hello`的`go.mod`文件，会发现生成了一个类似下面的需求提示：

```go
require example.com/greetings v0.0.0-00010101000000-000000000000
```

这样，就成功地在`example.com/hello`模块调用`example.com/greetings`模块了

## 添加一个测试

go语言为测试函数提供了非常简便的使用方法，观察以下函数`greetings_test.go`：

```go
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

想使用便捷的测试函数，需要遵循以下两点：

1. 首先，一个测试函数必须以`_test.go`结尾
2. 测试函数以指针`testing.T`为入参类型，可以使用这个参数`t`来报告和记录

创建好测试函数后，只需要在对应的目录下，输入

```she
go test
```

即可得到测试结果，可以使用`-v`选项来打印更多的测试信息

## 编译以及下载程序

使用

```shell
go run .
```

可以迅速编译并且运行，但是不会产生可执行的二进制文件，下面介绍两种命令，均可以产生可执行的二进制文件

```shell
go build
go install
```

区别是，`go build`会在当前目录下生成一个可执行二进制文件，而`go install`会把可执行二进制文件下载到系统路径中，可以通过以下命令查看具体会下载到哪个目录下：

```shell
go list -f '{{.Target}}'	//更多的go list指令查看https://pkg.go.dev/cmd/go#hdr-List_packages_or_modules
```

如果你希望设置一个专门存放go可执行文件的路径，使用命令：

```shell
go env -w GOBIN=/path/to/your/bim
```

当这些都配置好后，使用`go install`就可以生成可执行文件在指定路径中了，然后无需指定路径就可以执行这个可执行文件了

## go get

go get指令格式:

```shell
go get [flags] [packages]
```

首先需要明白，`go get`指令会把需要的包下载到哪呢？

1. 如果是`GOPATH`模式，会把包存储在`$GOPATH/src`目录下
2. 如果是`GOMODULE`模式，会把包存储在`$GOPATH/pkg/mod`目录下

go get常用命令：

```shell
go get [packages]	//指示下载包的最新版本
```

```shell
go get -u=patch [packages]
go get -u [packages]@patch	//指示下载包的最新patch version版本
go get -u [packages]@<revision>	//指示下载包的特定revision版本
go get -u [packages]@<branch-or-tag-name>	//指示下载包的特定branch或者tag版本
go get -u ./...		
go get -u all	//指示当前模块的所有依赖包
```

## GOPATH模式以及GOMODULE模式

### GOPATH模式

go 1.11以前采用的是`GOPATH`模式，go项目使用的包统一放在`GOPATH`目录下，`GOPATH`目录下又分了三个不同的目录，分别为`src`，`pkg`，`bin`，这三个目录分别存放go项目不同类型的包

- `src`：存放源代码以及项目下载的以来，当使用`go get`指令下载依赖包时，就存放在这个目录中。
- `pkg`：存放已编译好的包对象（.a文件），当一个包被构建后，结果文件就存放在`pkg`目录下，这样，下次在其他包需要使用就不需要重新编译了。
- `bin`：存放程序可执行的二进制文件，当构建一个可执行的程序时，可执行的二进制文件就会被放在这个文件夹下

`GOPATH`文件树：

```
/home/user/go/         <--- This is your GOPATH
├── bin/
├── pkg/
│   └── linux_amd64/
│       └── github.com/
│           └── someuser/
│               └── somelib.a    <--- Compiled dependency package
└── src/
    ├── github.com/
    │   └── someuser/
    │       └── somelib/         <--- Dependency's source code
    │           └── somelib.go
    └── myapp/                   <--- Your project
        └── main.go
```



### GOMODULE模式

自GO 1.11开始，`GOMODULE`模式出现了，最大的改变在于`go.mod`文件的出现，它使一个go项目的包管理变得更加灵活以及易于管理。

## go.sum文件

`go.sum`文件每行都记录了相关依赖包的哈希值，这个哈希值时使用`SHA-256`算法计算出来的，格式如下：

```go
<module> <version>[/go.mod] <hash>
```

当使用`go get`获取一个依赖包时，会在`go.sum`文件中生成两条记录，如下所示：

```go
github.com/google/uuid v1.1.1 h1:Gkbcsh/GbpXz7lPftLA3P6TYMwjCLYm83jiFQZF/3gY=  
github.com/google/uuid v1.1.1/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=

```

- 第一条记录为该依赖包版本整体（所有文件）的哈希值
- 第二条记录仅表示依赖包版本中go.mod文件的哈希值

依赖包版本中任何一个文件（包括`go.mod`）改动，**都会改变整体哈希值**。

### 与`go.mod`文件的区别

- `go.mod`只需要记录直接以来的依赖包版本，只在依赖包版本不包含`go.mod`文件时候才会记录简介依赖包版本。
- `go.sum`会记录构建用到的所有依赖包版本。

### 校验

**前提**：在构建过程中go命令会下载`go.mod`中的依赖包，下载的依赖包会缓存在本地，以便下次构建。

当拿到某项目的源代码并尝试在本地构建，go命令会从本地缓存中查找所有`go.mod`中记录的依赖包，并计算依赖包的哈希值，然后与`go.sum`中的记录进行比对，即检测本地缓存中使用的依赖包版本是否满足项目`go.sum`文件的期望。

如果校验失败，则证明本地缓存目录中依赖包版本的哈希值和项目中`go.sum`中记录的哈希值不一致，go命令将拒绝构建。

### 校验和数据库

环境变量`GOSUMDB`标识一个`checksum database`，即校验和数据库，实际上是一个web服务器，该服务器提供查询依赖包版本哈希值的服务。

该数据库中记录了很多依赖包版本的哈希值，比如Google官方的`sum.golang.org`则记录了所有的可公开获得的依赖包版本。

除了使用官方的数据库，还可以指定自行搭建的数据库，甚至干脆禁用它（`export GOSUMDB=off`）。

如果系统配置了`GOSUMDB`，在依赖包版本被写入`go.sum`之前会向该数据库查询该依赖包版本的哈希值进行二次校验，校验无误后再写入`go.sum`。

如果系统禁用了`GOSUMDB`，在依赖包版本被写入`go.sum`之前则不会进行二次校验，go命令会相信所有下载到的依赖包，并把其哈希值记录到`go.sum`中。

