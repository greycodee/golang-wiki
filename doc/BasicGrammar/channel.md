
> 本文为翻译文章
>
> 原文地址：https://golangbot.com/channels/

在本教程中，我们将讨论通道以及Goroutines如何使用通道进行通信。

## 什么是 Channel

可以将通道视为使用 Goroutine 进行通信的管道。与水在管道中从一端流到另一端的方式类似，可以使用 Channel 从一端发送数据并从另一端接收数据。

## 声明 Channel

每个 Channel 都有其自己的类型，类型代表该 Channel 可以传输的数据类型。其他的数据类型不能使用与其 Channel 数据类型不同的 Channel 进行传输数据。

在 `chan T`  中，其中的 T 代表了这个 Channel 的类型。

Channel 的零值是 `nil`,值为 `nil` 的 Channel 不能被使用，所以 Channel 要像 Map 和 Slice 一样使用 `make` 来进行定义。

让我们一些声明 Channel 的代码：

```go
package main

import "fmt"

func main() {  
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

我们声明了一个 Channel a，其值为 `nil`。因此，将执行 if 条件中的语句并使用 `make` 定义 Channel。a 在上面代码中是一个 `int` 类型的 Channel。

该程序输出：

```shell
channel a is nil, going to define it  
Type of a is chan int  
```

通常，短的声明也是定义通道的有效且简洁的方法。

```go
a := make(chan int)  
```

上面代码也定义了一个Channel a。

## Channel 发送数据和接收数据

下面给出了从通道发送和接收数据的语法：

```go
data := <- a // read from channel a  
a <- data // write to channel a  
```

箭头相对于通道的方向指定是发送还是接收数据?

在第一行中，箭头从 a 指向外部，因此我们正在从通道 a 中读取并将值存储到变量数据中。

在第二行中，箭头指向 a，因此我们正在编写通道 a。

## 默认情况下，发送和接收到通道处于阻塞状态

默认情况下，发送和接收到通道处于阻塞状态。这是什么意思？当数据发送到通道时，控件将在 send 语句中被阻塞，直到其他 Goroutine 从该通道读取为止。同样，当从通道读取数据时，将阻止读取，直到某些 Goroutine 将数据写入该通道为止。

通道的此属性可帮助 Goroutine 在不使用其他编程语言中很常见的显式锁或条件变量的情况下有效地进行通信。

## Channel 的示例代码

让我们编写一个程序来了解 Goroutines 如何使用通道进行通信。

实际上，我们将使用此处的通道来重写在学习 [Goroutines](https://golangbot.com/goroutines/) 时编写的程序。

引用下上次的代码：

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

这是在上篇文章中的代码。我们使用 sleep 来让主协程进行睡眠，以至于让 go hello() 协程有执行的时间。如果你不理解，建议看下上篇关于  [Goroutines](https://golangbot.com/goroutines/) 的文章。

我们在上面程序的基础上使用 Channel。

```go
package main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {  
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```

在上面代码中，我们创建了一个名为 done 的 bool 类型的通道，并将其作为参数传给了 go hello()。然后为们在下一行接收 done 通道的数据，程序会在其他 Goroutine 向该通道写入数据前一直被阻塞。因此，这消除了对 `time.Sleep` 的需求，这在原始程序中已经存在，可以防止主 Goroutine 退出。

代码 `<-done` 从完成的通道接收数据，但不使用该数据或将其存储在任何变量中。这是完全合法的。

现在，我们的主 Goroutine 已阻塞，以等待 done 通道上的数据。hello Goroutine 将此通道作为参数接收，打印 Hello world goroutine，然后写入 true 到 done 通道。写入完成后，主 Goroutine 从完成的通道接收数据，将其解除阻塞，然后打印 main function 文本。

程序输出：

```shell
Hello world goroutine  
main function  
```

让我们通过在 hello Goroutine 中引入睡眠来修改此程序，以更好地理解此阻塞概念。

```GO
package main

import (  
    "fmt"
    "time"
)

func hello(done chan bool) {  
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {  
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    <-done
    fmt.Println("Main received data")
}
```

在上面代码中，我们在 hello 协程中设置了睡眠 4 秒。

该程序首先会输出 `Main going to call hello go goroutine`。然后 hello 协程将会开始执行并输出 `hello go routine is going to sleep`。输出之后，hello 协程将会睡眠 4 秒种，在这期间 main 协程会一直被阻塞，直到可以从 done 中读取到数据。4 秒后，`hello go routine awake and going to write to done` 将会被输出，紧接着 `Main received data` 也会被输出。

## Channel 的其他例子🌰

让我们写更多的代码来更好的理解 Channel。该程序将打印数字的单个数字的平方和立方的总和。

例如，如果输入123，则此程序会将输出计算为：

```go
squares = (1 * 1) + (2 * 2) + (3 * 3)
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3)
output = squares + cubes = 50
```

我们将对程序进行结构设计，以便在单独的 Goroutine 中计算平方，在另一个 Goroutine 中计算立方体，最后求和发生在主 Goroutine 中。

```go
package main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```

`calcSquares` 函数会先判断 number 是否等于 0，如果等于 0 就发送 sum 的初始值到 squareop 通道里，如果不为 0 就计算 number 模 10 的 余数到平方和然后发送到 squareop 通道。`calcCubes` 函数也类似，只是它计算的事立方和。

这两个函数分别作为单独的 Goroutine 允许，并传入 channel 作为参数。然后主 Goroutine 从这两个通道中读取数据，分别存入 squares 和 cubes 变量。

程序输出：

```go
Final output 1536  
```

## 死锁

使用通道时要考虑的一个重要因素是死锁。如果 Goroutine 正在通过通道发送数据，则预期其他 Goroutine 应该正在接收数据。如果预期的结果没有发生，则会发生死锁的 panic。

同样的，如果一个 Goroutine 向通道接收数据，则预期其他 Goroutine 应该正在向该通道发送数据，否则也会发送 panic 异常。

```go
package main


func main() {  
    ch := make(chan int)
    ch <- 5
}
```

在上面代码中，创建了一个 ch 的通道，然后用语句 `ch <- 5` 来向该通道写入数据 5 。在这个程序中，没有其他的 Goroutine 从 ch 通道中接收数据。因此这个程序将在运行时发生 panic 异常。

```she
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox046150166/prog.go:6 +0x50
```

## 单向通道

到目前为止，我们讨论的所有通道都是双向通道，即可以在它们上发送和接收数据。也可以创建单向通道，即仅发送或接收数据的通道。

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

在上面代码中，我创建了一个只能发送数据的通道 sendch。语句 `chan<- int` 表示仅发送通道，因为箭头指向 `chan`。我尝试从该通道中读取数据，这是不允许的，并且在程序运行时，编译器会显示：

*./prog.go:12:14: invalid operation: <-sendch (receive from send-only type chan<- int)*

**一切都很好，但是如果无法读取仅发送通道，那么写入的目的是什么！**

**这是使用通道转换的地方。可以将双向通道转换为仅发送或仅接收通道，反之亦然。**

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    chnl := make(chan int)
    go sendData(chnl)
    fmt.Println(<-chnl)
}
```

在上面代码中，创建了一个双向通道 chnl。它作为参数被传入到 sendData Goroutine 中。sendData 函数到参数 `sendch chan<- int` 会把该通道转换为只写入的单向通道。所以 sendch 通道在 Goroutine 中还是双向通道。

## 关闭通道和用于通道上的范围循环

发送者可以关闭该通道，以通知接收者该通道将不再发送任何数据。

接收器可以在从通道接收数据时使用附加变量，以检查通道是否已关闭。

```go
v, ok := <- ch  
```

在上面的语句中，如果该值是通过对通道的成功发送操作接收到的，则 ok 是 true。如果 ok 为 false，则表示我们正在从封闭的通道读取数据。从关闭的通道读取的值将是通道类型的零值。

例如，如果通道是 int 通道，则从封闭通道接收的值将为 0。

```go
package main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {  
    ch := make(chan int)
    go producer(ch)
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
}
```

在上面代码中，producer Goroutine 向 chnl 写入 0-9 的数字，然后关闭该通道。主函数中定义了一个 for 循环，然后用 ok 变量来检查通道是否关闭。如果 ok 为 false ，代表通道是关闭的，因此循环将会被关闭。否则接受数据并打印数据和 ok 的值。

程序输出：

```shell
Received  0 true  
Received  1 true  
Received  2 true  
Received  3 true  
Received  4 true  
Received  5 true  
Received  6 true  
Received  7 true  
Received  8 true  
Received  9 true  
```

for 循环会一直从通道中接收数据直到该通道关闭。

让我们重写上面的的 for 循环代码：

```go
package main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {  
    ch := make(chan int)
    go producer(ch)
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```

for 循环从 ch 通道接收数据，直到关闭为止。 ch 关闭后，循环将自动退出。

该程序输出：

```shell
Received  0  
Received  1  
Received  2  
Received  3  
Received  4  
Received  5  
Received  6  
Received  7  
Received  8  
Received  9  
```

可以使用 for 范围循环来重写[通道另一个示例](https://golangbot.com/channels/#anotherexampleforchannels)部分中的程序，以提高代码的可重用性。

如果仔细看一下程序，您会发现在 calcSquares 函数和 calcCubes 函数中都重复了用于查找数字的单个数字的代码。我们将代码移至其自己的函数并同时调用它。

```go
package main

import (  
    "fmt"
)

func digits(number int, dchnl chan int) {  
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}
func calcSquares(number int, squareop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}
```

现在，上面程序中的 digits 函数包含用于从数字中获取单个数字的逻辑，并且 calcSquares 和 calcCubes 函数同时调用该逻辑。一旦数字中没有更多的数字，通道将关闭。calcSquares 和 calcCubes Goroutines 使用 for 范围循环在其各自的通道上侦听，直到关闭为止。该程序的其余部分是相同的。

该程序将打印:

```shell
Final output 1536  
```






