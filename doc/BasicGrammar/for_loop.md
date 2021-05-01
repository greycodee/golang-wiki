

> 本文为翻译文章
>
> 原文地址：https://golangbot.com/loops/

## 简介

循环一般用于执行重复的语句，在 Go 中，for 是唯一能执行循环的关键字，它没有像其他语言一样提供类似 while 的关键字来执行循环。

## for 循环语法

```go
for initialisation; condition; post {  
}
```

上面的 `initialisation` 语句只能执行一次。语句初始化后，然后会检查 `condition` 条件，如果条件为 `true` 时，就会执行花括号内的代码，然后在执行 `post` 语句。`post` 语句会在每次循环执行成功后被执行。执行post语句后，将重新检查条件。如果为true，则循环将继续执行，否则for循环终止。

Go中的所有三个组件（即初始化，条件和发布）都是可选的。让我们看一个示例，以更好地理解循环。

## 举个栗子🌰

让我们来写一个用 for 循环，打印出数字 1 到 10。

```go
package main

import (  
    "fmt"
)
func main() {  
    for i := 1; i <= 10; i++ {
        fmt.Printf(" %d",i)
    }
}
```

在上面的代码中，`i` 被初始化为 1。`initialisation` 语句会检查 `i<=10`，如果检查结果为 `true` ,就会打印出 `i` 的值，否则本次循环会被终止。`post` 语句会在每次迭代后把 `i` 的值加 1。当 `i` 比 10 大时，循环就会终止。

上面程序会打印出：1 2 3 4 5 6 7 8 9 10

在for循环中声明的变量仅在循环范围内可用，因此，无法在循环体外访问 `i`。

## break 关键字

`break` 语句用于在完成正常执行之前突然终止 for 循环，并将控制移到 for 循环之后的代码行。

让我们编写一个使用 break 打印从1到5的数字的程序。

```go
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i > 5 {
            break //loop is terminated if i > 5
        }
        fmt.Printf("%d ", i)
    }
    fmt.Printf("\nline after for loop")
}
```

在上面代码中，每次迭代都会检查 `i` 的值。当 `i` 大于 5 时，`break` 就会被执行，然后循环终止。然后 for 循环中后的 print 语句被执行。上面代码会输出：

```shell
1 2 3 4 5  
line after for loop  
```

## continue 关键字

`continue` 关键字用来跳过 for 循环的当前迭代。在 `continue` 关键字之后的所有代码都不会被执行。循环将继续进行下一次迭代。

让我们编写一个程序，使用 `continue` 打印从 1 到 10 的所有奇数。

```go
package main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Printf("%d ", i)
    }
}
```

在上面代码中的 `i%2 == 0` 用于检查 `i` 是否能被 2 整除。如果能被整除，则会执行 `continue` 关键字，然后跳过当前迭代，for 循环进入下一次迭代。因此，continue 之后的 print 语句将不会被调用，循环会进行到下一个迭代。

上面的语句会输出：1 3 5 7 9

## 嵌套 for 循环

for 循环也能在循环体里执行其他 for 循环。

让我们通过一段程序来理解嵌套 for 循环。

```shell
*
**
***
****
*****
```

下面的程序使用嵌套的for循环来打印序列。变量 `n` 存储着序列的行数，我们当前的例子是 5。外部 for 循环将 i 从 0 迭代到 4，内部 for 循环将 j 从 0 迭代到 i 的当前值。内循环为每次迭代打印 `*`，外循环在每次迭代结束时打印新行。运行该程序，您会看到序列打印为输出。

```go
package main

import (  
    "fmt"
)

func main() {  
    n := 5
    for i := 0; i < n; i++ {
        for j := 0; j <= i; j++ {
            fmt.Print("*")
        }
        fmt.Println()
    }
}
```

## 标签

标签可用于从内部 for 循环内部中断外部循环。让我们通过一个简单的例子来理解我的意思。

```go
package main

import (  
    "fmt"
)

func main() {  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
        }

    }
}
```

上面的程序它将打印:

```shell
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
i = 1 , j = 2  
i = 1 , j = 3  
i = 2 , j = 1  
i = 2 , j = 2  
i = 2 , j = 3  
```

这没什么特别的:)

如果我想在 i 和 j 相等时停止打印该怎么办。为此，我们需要从外部 for 循环中打破。当 i 和 j 相等时，在内部 for 循环中添加一个 break 只会从内部 for 循环中中断。

```go
package main

import (  
    "fmt"
)

func main() {  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
            if i == j {
                break
            }
        }

    }
}
```

在上面的程序中，当 i 和 j 相等时，我在内部 for 循环内添加了一个 break。这只会从内部 for 循环中断，而外部循环将继续。该程序将打印。

```shell
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
i = 2 , j = 1  
i = 2 , j = 2 
```

这不是预期的输出。当 i 和 j 相等时，即当它们等于 1 时，我们需要停止打印。

这是标签为我们提供帮助的地方。标签可用于从外部循环中断开。让我们使用标签重写上面的程序。

```go
package main

import (  
    "fmt"
)

func main() {  
outer:  
    for i := 0; i < 3; i++ {
        for j := 1; j < 4; j++ {
            fmt.Printf("i = %d , j = %d\n", i, j)
            if i == j {
                break outer
            }
        }

    }
}
```

在上面的程序中，我们添加了标签 outer。 我们通过指定标签来 break 外部的 for 循环。当 i 和 j 相等时，该程序将停止打印。

该程序将输出:

```shell
i = 0 , j = 1  
i = 0 , j = 2  
i = 0 , j = 3  
i = 1 , j = 1  
```

## 更多的栗子🌰

让我们再写一些代码来涵盖 for 循环的所有变体。

下面的程序打印从 0 到 10 的所有偶数。

```go
package main

import (  
    "fmt"
)

func main() {  
    i := 0
    for ;i <= 10; { // 省略 initialisation 和 post 语句
        fmt.Printf("%d ", i)
        i += 2
    }
}
```

众所周知，for循环的所有三个组件（即初始化，条件和发布）都是可选的。在上述程序中，省略了初始化和发布。i 在 for 循环外被初始化为 0。只要 i <= 10，就将执行该循环。i 在 for 循环内以 2 递增。上面的程序输出：0 2 4 6 8 10。

上面程序的 for 循环中的分号也可以省略。可以将这种格式视为 while 循环的替代方法。上面的程序可以改写为：

```go
package main

import (  
    "fmt"
)

func main() {  
    i := 0
    for i <= 10 { //semicolons are ommitted and only condition is present
        fmt.Printf("%d ", i)
        i += 2
    }
}
```

可以在 for 循环中声明多个变量并对其进行操作。让我们编写一个使用多个变量声明打印以下序列的程序。

```shell
10 * 1 = 10  
11 * 2 = 22  
12 * 3 = 36  
13 * 4 = 52  
14 * 5 = 70  
15 * 6 = 90  
16 * 7 = 112  
17 * 8 = 136  
18 * 9 = 162  
19 * 10 = 190  
```

```go
package main

import (  
    "fmt"
)

func main() {  
    for no, i := 10, 1; i <= 10 && no <= 19; i, no = i+1, no+1 { //multiple initialisation and increment
        fmt.Printf("%d * %d = %d\n", no, i, no*i)
    }

}
```

在上面的程序中，no 和 i 分别声明和初始化为 10 和 1。在每次迭代结束时，它们将增加 1。布尔运算符 `&&` 用于确保 i 小于或等于 10 且 no 小于或等于 19 的条件。

## 无限循环

创建无限循环的语法是：

```go
for {  
}
```

以下程序将连续打印 Hello World，而不会终止。

```go
package main

import "fmt"

func main() {  
    for {
        fmt.Println("Hello World")
    }
}
```

还有更多的构造范围可用于数组操作的循环中。当我们了解数组时，我们将进行介绍。