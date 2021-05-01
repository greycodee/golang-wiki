

> 本文为翻译文章
>
> 原文地址：https://golangbot.com/select/

## 什么是 select？

select 语句用于从多个发送/接收通道操作中进行选择。select 语句将阻塞，直到发送/接收操作之一准备就绪为止。如果准备好多个操作，则随机选择其中之一。语法与 switch 相似，不同之处在于每个 case 语句将是一个通道操作。让我们深入研究一些代码以更好地理解。

## 例子

```go
package main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    time.Sleep(6 * time.Second)
    ch <- "from server1"
}
func server2(ch chan string) {  
    time.Sleep(3 * time.Second)
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

在上面代码中，server1 函数睡眠了 6 秒后，然后将字符串 from server1 写入了 ch 通道。server2 函数里睡眠了 3 秒后将 from server2 字符串写入了 ch 通道。

主函数中开启了两个Goroutine server1和 server2。

当执行到 select 语句后，select 语句将阻塞，直到其中一种情况准备就绪为止。在上面的程序中，server1 Goroutine 在 6 秒后写入 output1 通道，而 server2 在 3 秒后写入 output2 通道。因此，select 语句将阻塞 3 秒钟，并等待 server2 Goroutine 写入 output2 通道。 3 秒钟后，程序将打印：

```shell
from server2
```

然后将终止。

## 实际使用 select

将上面程序中的功能命名为 server1 和 server2 的原因是为了说明 select 的实际用法。

假设我们有一个关键任务应用程序，我们需要尽快将输出返回给用户。此应用程序的数据库已复制并存储在世界各地的不同服务器中。假定功能`server1`和`server2`实际上与 2 台这样的服务器进行通信。每个服务器的响应时间取决于每个服务器的负载和网络延迟。我们将请求发送到两个服务器，然后使用该`select`语句在相应的通道上等待响应。由选择选择首先响应的服务器，而忽略其他响应。这样，我们可以将相同的请求发送到多台服务器，并将最快的响应返回给用户:)。

## Default 关键字

当其他情况都不准备就绪时，将执行 select 语句中的 Default。通常用于防止 select 语句被阻塞。

```go
package main

import (  
    "fmt"
    "time"
)

func process(ch chan string) {  
    time.Sleep(10500 * time.Millisecond)
    ch <- "process successful"
}

func main() {  
    ch := make(chan string)
    go process(ch)
    for {
        time.Sleep(1000 * time.Millisecond)
        select {
        case v := <-ch:
            fmt.Println("received value: ", v)
            return
        default:
            fmt.Println("no value received")
        }
    }

}
```

在上面的程序中，process 函数 睡眠10500毫秒（10.5秒），然后将 process successful 写入 ch 通道。主 Goroutine 创建了一个 process Goroutine。

同时调用 process Goroutine 后，主 Goroutine 中将启动一个无限 for 循环。无限循环在每次迭代开始时睡眠 1000 毫秒（1秒），然后执行选择操作。在最初的 10500 毫秒内，select 语句的第一种情况即`case v := <-ch:`未准备就绪，因为 process Goroutine 里的 `ch` 通道仅在 10500 毫秒后才写入数据。因此，该 `default` 将在这段时间内执行，程序将打印 `no value received` 10 次。

10.5 秒后，process Goroutine 写入`process successful`到 `ch` 通道。现在，将执行 select 语句的第一种情况，程序将打印 `received value:  process successful`，然后终止。该程序将输出：

```shell
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
received value:  process successful  
```

## 死锁和 default

```go
package main

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    }
}
```

在上面的程序中，我们创建了一个 ch 通道。我们尝试从 select 语句里面读取 ch 通道。select 语句将永远阻塞，因为没有其他 Goroutine 正在写入此通道，因此将导致死锁。该程序将在运行时发生 panic

```shell
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:  
main.main()  
    /tmp/sandbox627739431/prog.go:6 +0x4d
```

如果存在 default，则不会发生此死锁，因为将在没有其他情况准备好时执行 default。

上面的程序用下面的 default 重写:

```go
package main

import "fmt"

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```

程序输出：

```shell
default case executed
```

同样，即使选择只 `nil` 通道，也会执行default。

```go
package main

import "fmt"

func main() {  
    var ch chan string
    select {
    case v := <-ch:
        fmt.Println("received value", v)
    default:
        fmt.Println("default case executed")

    }
}
```

在上面的程序中，`ch` 通道的值为 `nil`，我们尝试从 select 中读取 `ch` 通道里的数据。如果 `default` 不存在，则该 `select` 将永远受阻并造成死锁。由于我们在选择框内有一个 default，它将被执行并且程序将打印出来：

```shell
default case executed  
```

## 随机选择

当 select 语句中的多个 case 准备就绪时，将随机执行其中之一。

```go
package main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    ch <- "from server1"
}
func server2(ch chan string) {  
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    time.Sleep(1 * time.Second)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

在上面的程序中，调用了 server1 和 server Goroutine。然后，主程序休眠 1 秒钟。当到达 select 语句时。server1 已经写入字符串 `from server1` 到  `output1` 通道，`server2` 已经写入字符串 `from server2` 到 `output2`通道，因此这两种情况下都准备好执行。如果您多次运行此程序，则输出将在随机选择的情况之间 `from server1` 或 `from server2` 取决于选择的情况而有所不同。

## 陷阱 — 空 select

```go
package main

func main() {  
    select {}
}
```

您认为上面程序的输出是什么？

我们知道 select 语句将阻塞直到执行其中一种情况。在这种情况下，select 语句没有任何情况，因此它将永远阻塞，从而导致死锁。该程序将发生 panic：

```shell
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:  
main.main()  
    /tmp/sandbox246983342/prog.go:4 +0x25
```


