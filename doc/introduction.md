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
