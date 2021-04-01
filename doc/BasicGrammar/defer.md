## 作用

`defer`在`go`中的作用就是延迟执行，一般常用来关闭一些`io`流等。简单来个例子：

```go
package main

import "fmt"

func main() {
	fmt.Println("1")
	defer func() {
		fmt.Println("2")
	}()
	fmt.Println("3")
}
```

上面的例子输出：

```go
1
3
2
```

**从上面得知`defer`会在程序结束前被执行**，但是如果有多个`defer`呢？这时候`go`是执行的呢？

## 多个defer执行流程

在`go1.12`版本中，`defer`对应的是一个结构体`_defer`。每个`go`的`goroutine`上都有一个指向这个结构体的指针`*_defer`。所有`_defer`通过结构体中的`link`组成一个**链表**。每添加一个`_defer`都会在链表头部上插入，所以就导致了**先入后出**现象。**所有的`_defer`**结构体都在堆内分配内存。

> **频繁的堆分配势必影响性能，所以Go语言会预分配不同规格的deferpool，执行时从空闲_defer中取一个出来用。没有空闲的或者没有大小合适的，再进行堆分配。用完以后，再放回空闲_defer池。这样可以避免频繁的堆分配与回收。**

![defer1](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/defer/defer1.jpg)

通过上面的流程，我们就可以理解多个`defer`的输出流程了

```go
package main

import "fmt"

func main() {

	fmt.Println("1")
	defer func() {
		fmt.Println("2")
	}()
	defer func() {
		fmt.Println("3")
	}()
	defer func() {
		fmt.Println("4")
	}()
	fmt.Println("5")
}
```

输出：

```shell
1
5
4
3
2
```

## defer的优化

上面说到每个`defer`都是由一个个结构组成的，所以`go`会对多个同样的没有捕获参数的`defer`做出优化。具体看图解

![defer4](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/defer/defer4.jpg)



## defer的参数取值

在`defer`注册的时候，编译器会在它自己的个参数后面，开辟一段空间，用于存放defer函数的返回值和参数。**这一段空间会在注册defer时，直接拷贝到_defer结构体的后面。![defer5](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/defer/defer5.jpg)**

所以就可以解释下面这段代码了：

```go
package main

import "fmt"

func main() {
	i:=1
	defer T1(i)
	i++
}
func T1(i int)  {
	fmt.Println(i)
}
```

输出：

```shell
1
```

## 1.13的defer

在1.13版本中，`go`语言改进了`defer`的创建。改为了直接用栈来存储`_defer`结构体。然后把栈上分配的`_defer`结构体注册到defer链表通过这样的方式避免在堆上分配`_defer`结构体。减少了堆和栈之间频繁复制导致的额外内存开销。

但是，值得注意的是，1.13版本中并不是所有`defer`都能够在栈上分配。**循环中的`defer`，无论是显示的`for循环`，还是`goto`形成的隐式循环，都只能使用1.12版本中的处理方式在堆上分配。即使只执行一次的for循环也是一样。**

例如：

```go
//显示循环
for i:=0; i< n; i++{
    defer B(i)
}
......

//隐式循环
again:
    defer B()
    if i<n {
        n++
        goto again
    }
```

所以在1.13版本中也保存了1.12的defer特性。

### 图解

在新版本中`_defer`的结构体新加了一个字段`heap`用来判断结构是在堆上还是在栈上。然后通过编译，把`_defer`结构体分配到栈上，`help`字段设为`false`。其他执行步骤与`1.12`没有太大的差别。

![defer1_13](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/defer/defer1_13.jpg)

除了分配位置的不同，栈上分配和堆上分配执行流程并没有本质的不同，而该方法可以适用于绝大多数的场景，与堆上分配相比，该方法可以将 `defer` 关键字的额外开销降低 30%。
