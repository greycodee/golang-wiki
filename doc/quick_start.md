# 快速开始

环境安装完成后，可以开始敲下我们的第一行`go`代码了

!> 官方现在推荐直接使用`moudle模式`，而且后期官方版本会取消`GOPATH模式`,所以本文档都使用`moudle模式`进行示例

1. 创建文件夹
```shell
    mkdir godemo && cd godemo
```

2. 初始化`moudle`
```go
    go mod init godemo
```
> 此时文件夹下会出现一个*go.mod*文件，这个就是用来管理包版本的

3. 创建文件*demo.go*,并编写如下代码
```go
    package main

    import "fmt"

    func main(){
        fmt.Println("hello go!")
    }
```

4. 运行代码
```go
// 在文件夹下执行命令
go run .
```

此时就会看到*hello go！*的文字输出