# Go教程-缓冲通道和工作池

> 本文为翻译文章
>
> 原文地址：https://golangbot.com/buffered-channels-worker-pools/

## 什么是缓冲通道？

在我们前面章节提到过的 channel 基本上都是无缓冲通道。正如我们在通道教程中详细讨论的那样，从无缓冲通道发送和接收数据都处于阻塞状态。

我们可以创建一个有缓冲区的通道。发送数据的时候只有在缓冲区满的时候才会阻塞。同样的，接收数据的时候只有在缓冲区为空的时候才阻塞。

可以通过向 make 函数传递一个附加的容量参数来创建缓冲的通道，该参数指定缓冲区的大小。

```GO
ch := make(chan type, capacity)  
```

上面的语法中的容量应大于 0，以使通道具有缓冲区。无缓冲区通道的容量为 0 ，因此为们前面的文章中创建通道时省略了这个参数。

## 例子

```go
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```

上面代码中，为们创建了一个容量为 2 的缓冲通道。由于通道的容量为 2，因此可以在不阻塞的情况下将 2 个字符串写入通道。我们写入了 `naveen` 和 `paul` 两个字符串到通道，并且通道不会阻塞。然后从通道读取它们。

程序输出：

```SHELL
naveen  
paul
```

## 另外的例子

让我们来看下另外的一个例子，这个例子是在一个 Goroutine 中向缓冲通道写入数据，同时在主 Goroutine 中读取该通道数据。这个例子能够帮助我们更好的理解缓冲通道。

```go
package main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}

```

在上面代码中，一个容量为 2 的缓冲通道被创建，并在主 Goroutine 中传递给 write Goroutine。然后主 Goroutine 睡眠 2 秒钟。在这期间，write Goroutine 也在同时运行着。write Goroutine 用 for 循环向 ch 通道写入了数字 0-4。由于通道的容量只有 2 大小，因此 write Goroutine 只能向通道写入 2 个数据，然后在通道至少被读取一个数据之前将会被阻塞。

程序输出：

```SHELL
successfully wrote 0 to ch  
successfully wrote 1 to ch 
```

在打印两行数据之后，write Goroutine 在 ch 通道被数据数据之前将会一直被阻塞。由于主 Goroutine 在开始从通道读取之前会休眠 2 秒钟，因此该程序将在接下来的 2 秒钟内不打印任何内容。在休眠 2 秒中后，主 Goroutine 开始用 `for range` 来取ch 通道里的数据，打印出读取的数据后再休眠 2 秒钟，一直循环直到 ch 通道关闭为止。因此，该程序将在2秒后打印以下行：

```shell
read value 0 from ch  
successfully wrote 2 to ch  
```

一直循环，直到所有值都写入通道，并且在write Goroutine 中将其关闭为止。最终的输出将是：

```SHELL
successfully wrote 0 to ch  
successfully wrote 1 to ch  
read value 0 from ch  
successfully wrote 2 to ch  
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch
```

## 死锁

```GO
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

在上面的程序中，我们将 3 个字符串写入容量为 2 的缓冲通道中。当第三个写入时，由于通道已超出容量，写入被阻止。现在，必须从通道读取一些 Goroutine 才能使写入继续进行，但是在这种情况下，没有从该通道读取并发例程。因此，将出现**死锁**，并且程序将在运行时发生 panic。

```SHELL
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox091448810/prog.go:11 +0x8d
```

## 长度与容量

缓冲通道的容量是通道可以容纳的值的数量。这是我们在使用 make 函数创建缓冲通道时指定的值。

缓冲通道的长度是当前在其中排队的元素数。

一个程序可以使事情变得清晰😀

```GO
package main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```

在上面代码中，我们创建了一个 容量为 3 的通道，它能容纳 3 个字符串。然后我们在接下来的代码中向通道写入 2 个字符串。现在通道里有 2 个字符串在排队，因此通道的长度为 2。然后我们从通道里读取了一个数据。现在通道里只有一个字符串在排队，因此通道的长度为 1。

程序输出：

```go
capacity is 3  
length is 2  
read value naveen  
new length is 1
```

## WaitGroup

WaitGroup 用于等待 Goroutine 的集合完成执行。在所有 Goroutine 执行完成前，才会解除阻塞。假设我们从主 Goroutine 派生了 3 个并发执行的 Goroutine。主Goroutine必须等待其他3个Goroutine完成后才能终止。这可以使用 WaitGroup 来完成。

代码示例：

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}

```

[WaitGroup](https://golang.org/pkg/sync/#WaitGroup) 是一种结构类型，我们在创建了 WaitGroup 类型的零值变量。WaitGroup 的工作方式是使用计数器。当我们在 WaitGroup 上调用 Add 并将其传递为 int 时，WaitGroup 的计数器将增加传递给 Add 的值。减少计数器的方法是通过在 WaitGroup 上调用 `Done()` 方法。`Wait()` 方法阻止 Goroutine 在其中被调用，直到计数器变为零为止。

在上面的程序中，我们调用 wg.Add(1)。在 for 循环内循环 3 次。因此，计数器现在变为 3。for 循环还会生成 3 个进程 Goroutine，然后调用 wg.Wait()。 使主 Goroutine 等待直到计数器变为零。通过 Goroutine 进程中对 wg.Done 的调用来减少计数器。一旦所有 3 个生成的 Goroutine 都完成了执行，即 wg.Done() 被调用了 3 次，计数器将变为零，并且主 Goroutine 将被解除阻塞。

**在主 Goroutine 循环体中传递 `wg` 的指针很重要。如果未传递指针，则每个 Goroutine 将具有其自己的 WaitGroup 副本，并且在完成执行时不会通知主 Goroutine。**

程序输出：

```SHELL
started Goroutine  2  
started Goroutine  0  
started Goroutine  1  
Goroutine 0 ended  
Goroutine 2 ended  
Goroutine 1 ended  
All go routines finished executing  
```

> 您的输出可能与我的输出不同，因为 Goroutines 的执行顺序可能会有所不同:)。

## 工作池

缓冲通道的重要用途之一是工作池的实现。通常，工作池是等待任务分配给它们的线程的集合。一旦完成分配的任务，他们就可以再次用于下一个任务。我们将使用缓冲通道实现工作池。我们的工作池将执行查找输入数字的位数之和的任务。例如，如果传递了 234，则输出将为 9（2 + 3 + 4）。工作池的输入将是伪随机整数列表。

以下是我们工作池的核心功能：

- 创建 Goroutine 池，该 Goroutine 池在输入缓冲通道上侦听以等待分配作业
- 将作业添加到输入缓冲通道
- 作业完成后将结果写入输出缓冲通道
- 从输出缓冲通道读取和打印结果

我们将逐步编写此程序，以使其更易于理解。

第一步将是创建代表工作和结果的结构。

```GO
type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}
```

每个 Job 结构都有一个 ID 和一个 randomno，必须为其计算各个数字的总和。

Result 结构体具有一个 job 字段，该字段是在 sumofdigits 字段中保存结果（单个数字的总和）的作业。

下一步是创建用于接收作业和写入输出的缓冲通道。

```GO
var jobs = make(chan Job, 10)  
var results = make(chan Result, 10) 
```

Worker Goroutines 在作业缓冲通道上侦听新任务。任务完成后，结果将写入结果缓冲通道。

下面的 digits 函数会执行实际工作，查找整数的各个数字之和，然后将其返回。我们将为此功能增加 2 秒钟的睡眠时间，以模拟该功能需要花费一些时间来计算结果的事实。

```go
func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
```

接下来，我们将编写一个创建工作程序 Goroutine 的函数。

```go
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
```

上面的函数创建一个工作程序，该工作程序从作业通道中读取，使用当前作业和 digits 函数的返回值创建一个 Result 结构，然后将结果写入 results 缓冲通道。此函数将 WaitGroup wg 作为参数，当所有作业完成时，它将在其上调用 Done() 方法。

createWorkerPool 函数将创建工作程序 Goroutine 池。

```go
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
```

上面的函数将要创建工作池的大小作为参数。在创建 Goroutine 来递增 WaitGroup 计数器之前，它将调用 wg.Add(1)。然后，它通过将 WaitGroup wg 的指针传递给 worker 函数来创建 worker Goroutines。创建所需的工作程序 Goroutines 之后，它将通过调用 wg.Wait() 等待所有 Goroutines 完成执行。在所有 Goroutine 完成执行后，由于所有 Goroutine 完成了执行，因此它关闭了结果通道，并且没有其他人将进一步写入结果通道。

现在我们已经准备好工作池，让我们继续编写可以将作业分配给工作人员的函数。

```go
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
```

上面的分配函数将要创建的作业数作为输入参数，生成最大值为 998 的伪随机数，使用随机数和 for 循环计数器 i 作为 id创建 Job 结构，然后将其写入作业 渠道。写入所有作业后，它将关闭作业通道。

下一步将是创建读取结果通道并打印输出的函数。

```go
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
```

result 函数读取 results 通道并打印作业 ID，输入随机数和随机数的位数之和。result 函数还将 done 通道作为参数，一旦它打印了所有结果，就会写入该通道。

我们已经准备好一切。让我们继续完成从 main 函数调用所有这些函数的最后一步。

```go
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

我们首先在 mian 函数中存储程序的执行开始时间，然后在最后一行中计算 endTime 和 startTime 之间的时间差，并显示该程序花费的总时间。 这是必需的，因为我们将通过更改 Goroutine 的数量来进行一些基准测试。

将 noOfJobs 设置为 100，然后调用 allocate 将作业添加到作业通道。

然后创建完工通道并将其传递给结果 Goroutine，以便它可以开始打印输出并在所有内容都打印完后通知。

最终，通过调用 createWorkerPool 函数创建了一个包含 10 个辅助 Goroutine 的池，然后 main 在 done 通道上等待所有结果被打印出来。

这是完整程序供您参考。我也导入了必要的软件包。

```go
package main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

请在本地计算机上运行此程序，以更准确地计算总时间。

程序输出：

```shell
Job id 1, input random no 636, sum of digits 15  
Job id 0, input random no 878, sum of digits 23  
Job id 9, input random no 150, sum of digits 6  
...
total time taken  20.01081009 seconds  

```

将与 100 个作业相对应地打印总共 100 行，然后最后在最后一行中打印程序运行所花费的总时间。您的输出将与我的输出不同，因为 Goroutines 可以以任何顺序运行，并且总时间也将根据硬件而有所不同。就我而言，该程序大约需要 20 秒钟才能完成。

现在让我们将 main 函数中的 noOfWorkers 增加到 20。我们使工作池里的数增加了一倍。由于 work Goroutines 增加了（准确地说是增加了一倍），因此完成程序所需的总时间应该减少（准确地说减少了一半）。在我的情况下，它变成了 10.004364685 秒，程序被打印出来了，

```shell
...
total time taken  10.004364685 seconds  
```

现在我们可以理解，随着工人 Goroutine 数量的增加，完成工作所需的总时间减少了。我将其作为练习，让您在主要函数中使用 noOfJobs 和 noOfWorkers 来获得不同的值并分析结果。