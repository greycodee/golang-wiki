

## 初始化

```go
// 使用make赋值
map1 = make(map[string]string)

// 字面量赋值
map2 := map[string]string{}
```

## 赋值

```go
// make方式
map1 = make(map[string]string)
map1["go"] = "go语言"

// 字面量
map2 := map[string]string{
    "go" : "go语言",
}
```

## 访问

```go
// 获取map1里key为`go`的值
value := map1["go"]
```

## 遍历

```go
// 使用range关键字
for k,v:=range map1{
    fmt.Printf("key:%s  ==>  value:%s\n",k,v)
}
```



## 使用map存储指针会出现的问题

当map存在指针类型的值时，可能会出现如下情况

首先看看下面的代码输出情况：

```go
package main

import (
	"fmt"
)

func main() {

	m2:=make(map[string]*Student)

	people := []Student{
		{Name: "Zhangsan", Age: 20},
		{Name: "Lilei", Age: 23},
		{Name: "Wangmeimei", Age: 22},
	}
	for _,v:=range people{
		m2[v.Name]=&v
	}

	for k,v:=range m2{
		fmt.Printf("key:%s  ==>  Name:%s\n",k,v.Name)
	}
}

type Student struct {
	Name string
	Age int
}

```

输出：

```shell
key:Zhangsan  ==>  Name:Wangmeimei
key:Lilei  ==>  Name:Wangmeimei
key:Wangmeimei  ==>  Name:Wangmeimei
```

是不是和期望的不一样，原先期望的是把切片里的`*Student`全部拷贝到`m2`集合里，可以现在复制的都是同一个。

## 图解

![range](http://cdn.mjava.top/blog/range.jpg)

这是因为这个语句中`v`是`people`的一个拷贝的值，他会随着遍历而不断变化。而`m2`里存的是`v`的在内存中的地址，所以在遍历完后，`v`地址上的值就是切片的最后一个值，所以我们`m2`集合里存的值就统一变成了切片最后的一个值了。

```go
for _,v:=range people{
    m2[v.Name]=&v
}
```

## 改进

解决上面的方法也比较简单，就是改一下循环语句

```diff
- for _,v:=range people{
-    m2[v.Name]=&v
- }

+ for i,v:=range people{
+ 	m2[v.Name]=&people[i]
+ }
```

这样就是直接存储原切片的地址了。