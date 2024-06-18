# Map

## 空map

一个空`map`表示为`nul`，空map没有`key`，并且`key`不能被添加

## make创建一个map

可以使用内置函数`make`创建一个`map`，只需要指定`map`类型，如下

```go
m = make(map[string]Vertex)
```

可以直接使用**key-value**方式初始化一个`map`，如下

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

//更简单的表示方法
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google": {37.42202, -122.08408},
}
```

## 对map进行操作

1. 给map进行赋值

```go
m[key] = elem
```

2. 通过`key`获取`value`

```go
elem = m[key]
```

3. 删除`key`值(m代表map)

```go
delete(m, key)
```

4. 测试`key`在map中是否存在

```go
elem, ok = m[key]
```

如果存在，`ok`为`true`；否则，`ok`为`false`，`elem`为`value`类型对应的**0**值