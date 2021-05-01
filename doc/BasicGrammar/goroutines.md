
> 本文为翻译文章
>
> 原文地址：https://golangbot.com/goroutines/

在本教程中，我们将讨论如何使用Goroutines在Go中实现并发。

## 什么是 Goroutines？

Goroutine 是与其他函数或方法同时运行的函数或方法。Goroutines 可以被认为是轻量级的线程。与线程相比，创建 Goroutine 的成本很小。因此，Go 应用程序同时运行数千个 Goroutine 是很常见的。

## Goroutines 相对于线程的优势

- 与线程相比，goroutine 占内存非常小。它们的堆栈大小只有几 kb，并且堆栈可以根据应用程序的需要进行扩展和收缩，而对于线程，则必须指定堆栈大小并固定堆栈大小。
- Goroutines 被多路复用到更少的 OS 线程。在具有数千个 Goroutine 的程序中，可能只有一个线程。如果该线程块中的任何 Goroutine 说正在等待用户输入，则将创建另一个 OS 线程，并将其余的 Goroutines 移至新的 OS 线程。所有这些都由运行时小心处理，作为程序员，我们从这些复杂的细节中抽象出来，并为它们提供了干净的 API 以与并发一起工作。
- Goroutine 使用 channel 进行通信。通过设计 channel，可以防止在使用 Goroutines 访问共享内存时发生争用情况。可以将 channel 视为使用 Goroutine 进行通信的管道。在下一个教程中，我们将详细讨论频道。

## 如何启动一个 Goroutines？

使用关键字 go 前缀函数或方法调用，您将同时运行一个新的 Goroutine。

创建一个 Goroutines：

```go
package main

import (  
    "fmt"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    fmt.Println("main function")
}
```

`go hello()` 将会创建一个新的 Goroutines。现在 `hello()` 函数将与 `main()` 函数同时运行。 main 函数在其自己的 Goroutine 中运行，并称为 `main Goroutine`。

运行该程序，您会感到惊讶！

该程序仅输出文本主 `main function`。我们在开始时创建的 Goroutine 怎么了？我们需要了解 go 协程的两个主要属性，以了解为什么会发生这种情况。

- 当启动新的 Goroutine 时，goroutine 调用会立即返回。与函数不同，控件不等待 Goroutine 完成执行。在 Goroutine 调用之后，控件会立即返回到下一行代码，并且 Goroutine 中的所有返回值都将被忽略。
- 主 Goroutine 应该正在运行，其他任何 Goroutine 才能运行。如果主 Goroutine 终止，则该程序将终止，并且其他 Goroutine 将不会运行。 

我想现在您将能够理解为什么我们的Goroutine没有运行。在第调用  `hello()` 之后。 控件立即返回到下一行代码，而无需等待 hello goroutine 完成并打印 `main function`。然后，由于没有其他代码可以执行，因此主 Goroutine 终止了，因此 hello Goroutine 没有获得运行的机会。

让我们现在解决这个问题。

```go
package main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}

```

在上面程序中，我们调用了 time 包的 Sleep 方法，该方法将休眠正在执行 go 协程的。在这种情况下，主 goroutine 进入睡眠状态 1 秒钟。现在，对 go hello() 的调用在主 Goroutine 终止之前有足够的时间执行。该程序首先打印 `Hello world goroutine`，等待1秒钟，然后打印 `main function`。

在主要 Goroutine 中使用睡眠来等待其他 Goroutine 完成执行的这种方式是我们用来了解 Goroutine 如何工作的一种技巧。channel 可用于阻塞主 Goroutine，直到所有其他 Goroutine 完成执行为止。我们将在下一个教程中讨论。

## 启动多个Goroutine

让我们再编写一个程序来启动多个 Goroutine，以更好地理解 Goroutine。

```go
package main

import (  
    "fmt"
    "time"
)

func numbers() {  
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d ", i)
    }
}
func alphabets() {  
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c ", i)
    }
}
func main() {  
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}
```

上面的程序中启动了两个 Goroutine。 这两个 Goroutine 同时运行。`numbers` Goroutine 最初休眠 250 毫秒，然后打印 1，然后再次休眠并打印 2，并且发生相同的周期，直到打印 5。类似地，`alphabets` Goroutine 从 a 到 e 打印字母，并具有 400 毫秒的睡眠时间。主 Goroutine 启动 `numbers` 和 `alphabets` Goroutine，休眠 3000 毫秒，然后终止。

程序输出：

```shell
1 a 2 3 b 4 c 5 d e main terminated  
```

下图描述了该程序的工作方式。请在新标签页中打开图片以获得更好的可见性：）

![Goroutines-explained](http://cdn.mjava.top/blog/Goroutines-explained.png)

图像的第一部分是蓝色，代表 `numbers` Goroutine，栗色的第二部分代表 `alphabets` Goroutine，绿色的第三部分代表主 Goroutine，黑色的最后一部分将上述三个部分合并在一起，向我们展示了程序的工作方式。每个框顶部的 0 ms，250 ms 之类的字符串表示时间（以毫秒为单位），每个框底部的输出表示为 1、2、3，依此类推。最后一个黑匣子的底部具有值 1 a 2 3 b 4 c 5 d e 是主要终止的，也是程序的输出。该图像不言自明，您将能够了解该程序的工作方式。


