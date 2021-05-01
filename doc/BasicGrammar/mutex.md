

> 本文为翻译文章
>
> 原文地址：https://golangbot.com/mutex/

## 关键部分

讲互斥锁之前，了解并发编程中[关键部分](https://en.wikipedia.org/wiki/Critical_section)的概念很重要。当一个程序并发运行时，修改共享资源的代码部分不应同时被多个 [Goroutine](https://golangbot.com/goroutines/) 访问。修改共享资源的这段代码称为关键部分。例如，假设我们有一段代码将变量 x 递增 1。

```go
x = x + 1  
```

只要单个 Goroutine 访问以上代码，就不会有任何问题。

让我们看看为什么在同时运行多个 Goroutine 时此代码会失败。为了简单起见，我们假设有 2 个 Goroutines 同时运行上述代码行。

在内部，上述代码行将由系统按照以下步骤执行（有更多的技术细节涉及寄存器，加法工作原理等，但是出于本教程的考虑，我们假设这是三个步骤），

1. 得到x的当前值
2. 计算x + 1
3. 将步骤2中的计算值分配给x

当通过一个 Goroutine 来执行这三个步骤时，一切都很好。

让我们讨论两个 Goroutines 同时运行此代码时发生的情况。下图描述了两个 Goroutine 同时访问代码 `x = x + 1` 时可能发生的情况。

![cs5](http://cdn.mjava.top/blog/cs5.png)

我们假设 `x` 的初始值为 `0`。`Goroutine 1` 获得 `x` 的初始值，计算 `x + 1`，然后在将计算的值赋给 `x` 之前，系统上下文切换为 `Goroutine 2`。现在 `Goroutine 2` 得到其初始值 `x` 仍然是 `0`，计算 `x + 1`。此后，系统上下文再次切换到 `Goroutine 1`。现在 `Goroutine 1` 将其计算值 `1` 分配给 `x`，因此 `x` 变为 `1`。然后 `Goroutine 2` 再次开始执行，然后将其计算值再次分配给 `x`，因此 `x` 在两个 `Goroutines` 执行后均为 `1`。

现在让我们看一下可能发生的另一种情况。

![cs-6](http://cdn.mjava.top/blog/cs-6.png)

在上述情况下，`Goroutine 1` 开始执行并完成其所有三个步骤，因此 `x` 的值变为 `1`。然后 `Goroutine 2` 开始执行。现在 `x` 的值为 `1`，当 `Goroutine 2` 完成执行时，`x` 的值为 `2`。

因此，从这两种情况中，您可以看到 x 的最终值为 1 或 2，具体取决于上下文切换发生的方式。程序输出取决于 Goroutines 的执行顺序的这种不良情况称为[竞争条件](https://en.wikipedia.org/wiki/Race_condition)。

在上述情况下，如果在任何时间点仅允许一个 Goroutine 访问代码的关键部分，则可以避免争用条件。通过使用 Mutex，可以做到这一点。

## Mutex

Mutex 用于提供一种锁定机制，以确保在任何时间点只有一个 Goroutine 正在运行代码的关键部分，以防止发生竞争情况。

Mutex 在[同步](https://golang.org/pkg/sync/)包中可用。[Mutex](https://tip.golang.org/pkg/sync/#Mutex) 上定义了两种方法，即 [Lock](https://tip.golang.org/pkg/sync/#Mutex.Lock) 和 [Unlock](https://tip.golang.org/pkg/sync/#Mutex.Unlock) 。在调用之间存在的任何代码，`Lock` 并且 `Unlock` 将仅由一个 Goroutine 执行，从而避免了争用情况。

```go
mutex.Lock()  
x = x + 1  
mutex.Unlock()  
```

在上面的代码中，`x = x + 1` 在任何时间都只能由一个 Goroutine 执行，从而防止出现竞争状况。

**如果一个 Goroutine 已经持有该锁，并且如果有新的 Goroutine 试图获取锁，则新的 Goroutine 将被阻止，直到互斥锁被解锁为止。**

## 有竞争的程序

在本节中，我们将编写一个具有竞态条件的程序，在接下来的部分中，我们将解决竞态条件。

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup) {  
    x = x + 1
    wg.Done()
}
func main() {  
    var w sync.WaitGroup
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

在上面的程序中，`increment` 函数增量的值 `x` 加  `1 `，然后调用 `Done()` ,通知 [WaitGroup](https://golangbot.com/buffered-channels-worker-pools#waitgroup) 其完成。

我们产生了 1000 个 increment Goroutines。这些 Goroutine 中的每一个都同时运行，并且在尝试递增 x 时出现争用条件。因为多个 Goroutine 尝试同时访问 x 的值。

在本地计算机上多次运行此程序，您会发现由于竞争条件，每次的输出都会有所不同。其中一些我所遇到的产出是`final value of x 941`，`final value of x 928`，`final value of x 922`等。

## 使用互斥锁解决竞争条件

在上面的程序中，我们生成了 1000 个 Goroutines。如果每个将 x 的值增加 1，则 x 的最终期望值应为 1000。在本节中，我们将使用互斥锁在上面的程序中修复竞争条件。

```go
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, m *sync.Mutex) {  
    m.Lock()
    x = x + 1
    m.Unlock()
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

[Mutex](https://golang.org/pkg/sync/#Mutex) 是一种 struct 类型，我们创建一个零值 Mutex 类型的变量 m。在上面的程序中，我们更改了 `increment` 函数，使 x。递增的代码 `x = x + 1` 在 `m.Lock()` 和 `m.Unlock()` 之间。现在，此代码没有任何竞争条件，因为在任何时间点都只允许一个 Goroutine 执行这段代码。

程序输出：

```shell
final value of x 1000  
```

传递互斥锁的地址很重要。如果互斥锁是通过值而不是通过地址传递的，则每个 Goroutine 将具有自己的互斥锁副本，并且竞争条件仍然会发生。

## 使用 channel 解决竞争条件

我们也可以使用 channel 通道解决竞争条件。让我们看看这是如何完成的。

```GO
package main  
import (  
    "fmt"
    "sync"
    )
var x  = 0  
func increment(wg *sync.WaitGroup, ch chan bool) {  
    ch <- true
    x = x + 1
    <- ch
    wg.Done()   
}
func main() {  
    var w sync.WaitGroup
    ch := make(chan bool, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

在上面的程序中，我们创建了一个容量为 1 的[缓冲的通道](https://golangbot.com/buffered-channels-worker-pools/)，并将该[通道](https://golangbot.com/buffered-channels-worker-pools/)传递到行 `increment` 的Goroutine 中。此缓冲通道用于确保只有一个 Goroutine 可以访问递增 x 的代码关键部分。这是通过传递 `true `到缓冲通道来完成的。 `x` 递增之前为 8。由于缓冲的通道的容量为 `1`，所有其他尝试写入该通道的 Goroutine 都会被阻塞，直到将 x 递增后从该通道读取该值为止。实际上，这仅允许一个 Goroutine 访问关键部分。

程序输出：

```shell
final value of x 1000  
```

## 互斥锁对比通道

我们已经使用互斥体和通道解决了竞争条件问题。那么我们如何决定什么时候使用什么呢？答案在于您要解决的问题。如果您要解决的问题更适合互斥锁，请继续使用互斥锁。如果需要，请毫不犹豫地使用互斥锁。如果问题似乎更适合通道，请使用:)。

大多数 Go 新手都尝试使用通道解决每种并发问题，因为这是该语言的一个很酷的功能。这是错误的。该语言为我们提供了使用 Mutex 或 Channel 的选项，并且选择两者都没有错。

通常，当 Goroutine 需要相互通信时使用通道，而当只有一个 Goroutine 应该访问代码的关键部分时则使用互斥锁。

在上面我们解决了问题的情况下，我更喜欢使用互斥锁，因为此问题不需要 goroutine 之间的任何通信。因此，互斥锁是很自然的选择。

我的建议是选择解决问题的工具，而不要尝试使问题适合该工具:)


