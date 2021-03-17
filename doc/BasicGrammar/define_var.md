# 变量
`go`变量分为**局部变量**和**全局变量**

## 单个赋值

```go
    var x = 1

    //or

    var x int
    x = 1

    // or

    x:=1
```

## 多个赋值

```go
    var x,y = 1,2

    //or

    var x,y int
    x=1
    y=2

    //or

    x,y := 1,2

```

## 因式分解法赋值

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