# 语法简介

## 基本语法
`go`语言是从`package`中构建的，每个`go`项目中必须要有一个`package`名为**main的包**和一个**main函数**

下面就是一个简单的`go`程序

```go

 // 包名
 package main

 // 导入包
 import "fmt"

 // 函数
 func main(){
     fmt.Println("hello go!")
 }

```

## 注释
`go`支持两种注释，**单行注释**和**多行注释**

- 单行注释

```go
    // 注释的内容
    func test(){
        fmt.Println("hello go!")
    }
```

- 多行注释

```go
    /*
        多行注释
        第二行
        第三行
    */
    func test(){
        fmt.Println("hello go!")
    }
```

## 特殊函数init

这个函数会先于`main`函数被调用，所以可以用它来初始化一些值

代码示例：
```go
package main

import "fmt"

func init() {
	fmt.Println("我是init函数")
}

func main() {
	fmt.Println("我是mian函数")
}
```
输出：
```shell
我是init函数
我是mian函数
```
## 变量赋值


### 单个赋值

```go
    var x = 1

    //or

    var x int
    x = 1

    // or

    x:=1
```

### 多个赋值

```go
    var x,y = 1,2

    //or

    var x,y int
    x=1
    y=2

    //or

    x,y := 1,2

```

### 因式分解法赋值

> 用于声明全局变量

```go
    var (
        x = 1
        y = 2
    )

    //or

    var (
        x int
        y int
    )
    x=1
    y=2
```

## 常量
关键字`const`用于赋值常量
```go
// 格式1    显式类型定义
const num int = 9

// 格式2  可以省略类型  隐式类型定义
const num = 9

// 格式3    同类型多重复制
const num, size = 9, 3

// 格式4    多类型多重赋值
const num, isture = 9, true

// 格式5    因式分解赋值
const (
	A = 1
	B = 2
	ISTRUE = true
)
```
### iota
`iota`是特殊常量，在`const`关键字出现时被重置为`0`。`const`中每新增一行常量声明，`iota`计算`+1`

```go
package main

import "fmt"

func main() {
	fmt.Println(A)
	fmt.Println(B)
	fmt.Println(C)
	fmt.Println(D)
}

const (
	A = iota
	B
	C
	D
)
```
输出：
```shell
0
1
2
3
```
