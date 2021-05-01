# panic和recover

## panic

- 获取当前调用者所在的 `g` ，也就是 `goroutine`

- 遍历并执行 `g` 中的 `defer` 函数

- 如果 `defer` 函数中有调用 `recover` ，并发现已经发生了 `panic` ，则将 `panic` 标记为 `recovered`

- 在遍历 `defer` 的过程中，如果发现已经被标记为 `recovered` ，则提取出该 `defer` 的 sp 与 pc，保存在 `g` 的两个状态码字段中。

- 调用 `runtime.mcall` 切到 `m->g0` 并跳转到 `recovery` 函数，将前面获取的 `g` 作为参数传给 `recovery` 函数。
  `runtime.mcall` 的代码在 go 源码的 `src/runtime/asm_xxx.s` 中，`xxx` 是平台类型，如 `amd64` 。

  > 这里之所以要切到 `m->g0` ，主要是因为 Go 的 `runtime` 环境是有自己的堆栈和 `goroutine`，而 `recovery` 是在 `runtime` 环境下执行的，所以要先调度到 `m->g0` 来执行 `recovery` 函数。

- `recovery` 函数中，利用 `g` 中的两个状态码回溯栈指针 sp 并恢复程序计数器 pc 到调度器中，并调用 `gogo` 重新调度 `g` ，将 `g` 恢复到调用 `recover` 函数的位置， goroutine 继续执行。



## 关键字介绍

- `panic`：一旦出现，就意味着程序的结束并退出。Go 语言中 `panic` 关键字主要用于主动抛出异常，类似 `java` 等语言中的 `throw` 关键字。
- `recover`：将程序状态从严重的错误中恢复到正常状态。Go 语言中 `recover` 关键字主要用于捕获异常，让程序回到正常状态，类似 `java` 等语言中的 `try ... catch` 。

## 使用

- `panic`

  1. 发生`panic`后，后续代码不会执行
  2. 发生`panic`后，会执行`defer`链表

  我们先创建两个协程，然后在其中一个协程里发生`panic`。看看另一个协程会怎么样。

  ```go
  package main
  
  import "fmt"
  
  func main() {
      // 第一个协程
  	go func() {
         
  		var i int
  		for {
  			i++
  			fmt.Println("协程1")
  			time.Sleep(1*time.Second)
               // 3秒后发生panic
  			if i==3 {
  				panic("异常退出")
  			}
  		}
  
  	}()
  	// 第二个协程
  	go func() {
  		for  {
  			fmt.Println("协程2")
  			time.Sleep(1*time.Second)
  		}
  	}()
      // 让主协程不退出
  	for  {
  		time.Sleep(1*time.Second)
  	}
  }
  ```

  当程序执行5秒后，其中一个协程就会发生`panic("异常退出")`，这时程序就会退出，随之另一个协程也结束了。

  输出：

  ```shell
  协程2
  协程1
  协程2
  协程1
  协程2
  协程1
  协程2
  panic: 异常退出
  
  goroutine 6 [running]:
  main.main.func1()
          /home/zheng/STUDY/GoWork/demo/main.go:28 +0xb9
  created by main.main
          /home/zheng/STUDY/GoWork/demo/main.go:15 +0x35
  
  ```

  **所以`panic`如果不捕获，就会导致程序整体关闭的严重的后果**

- `recovery`

  1. 为了解决`panic`的问题，`go`也提供了`recovery`这个函数，用于捕获异常，保证程序能正常运行。
  2. 只对当前`goroutine`发生的`panic`有效

  2. `recovery`要配合`defer`使用，因为`recovery`要在发生`panic`后执行才有效，但是发生`panic`的后续代码不会执行了，但是会执行`defer`

  我们对上面程序的第一个协程加上`recovery`

  ```go
   // 第一个协程
  go func() {
      // 捕获异常
      defer func() {
          err:=recover()
          fmt.Printf("捕获的：%s\n",err)
      }()
      
      var i int
      for {
          i++
          fmt.Println("协程1")
          time.Sleep(1*time.Second)
          // 3秒后发生panic
          if i==3 {
              panic("异常退出")
          }
      }
  
  }()
  ```

  输出:

  ```go
  协程2
  协程1
  协程1
  协程2
  协程2
  协程1
  协程2
  捕获的：异常退出
  协程2
  协程2
  ...
  ```

  可以看到线程1的`recovery`捕获当前协程的`panic`的异常，并输出异常原因。但是另一个协程2还会继续执行。

  ## 底层逻辑

  在每个`goroutine`也有一个指针指向`_panic`链表表头，然后每增加一个`panic`就会在链表头部加入一个`_panic`结构体。当所有的`defer`执行完后，`_panic`链表就会从尾部开始打印`panic`信息了，也就是说先发生的`panic`先打印信息。![panic2](http://cdn.mjava.top/blog/panic2.jpg)

  ### _panic结构体

  在`go`源码的[runtime/runtime2.go](https://github.com/golang/go/blob/41d8e61a6b9d8f9db912626eb2bbc535e929fefc/src/runtime/runtime2.go#L942)中有`_panic`的结构体信息

  ```go
  type _panic struct {
  	argp      unsafe.Pointer // pointer to arguments of deferred call run during panic; cannot move - known to liblink
  	arg       interface{}    // argument to panic
  	link      *_panic        // link to earlier panic
  	pc        uintptr        // where to return to in runtime if this panic is bypassed
  	sp        unsafe.Pointer // where to return to in runtime if this panic is bypassed
  	recovered bool           // whether this panic is over
  	aborted   bool           // the panic was aborted
  	goexit    bool
  }
  ```

  ![panic3](http://cdn.mjava.top/blog/panic3.jpg)

结构体中的 `pc`、`sp` 和 `goexit` 三个字段都是为了修复 [`runtime.Goexit`](https://draveness.me/golang/tree/runtime.Goexit) 带来的问题引入的。[`runtime.Goexit`](https://draveness.me/golang/tree/runtime.Goexit) 能够只结束调用该函数的 Goroutine 而不影响其他的 Goroutine，但是该函数会被 `defer` 中的 `panic` 和 `recover` 取消，引入这三个字段就是为了保证该函数的一定会生效。

## 嵌套panic

接下来用个嵌套的panic例子来强化理解一下上面讲的

```go
package main

func main() {
	defer func() {
		defer func() {
			panic("双重嵌套：panic")
		}()
		panic("panic1")
	}()
	defer func() {
		panic("panic2")
	}()
	panic("main-panic")
}
```

输出：

```shell
panic: main-panic
        panic: panic2
        panic: panic1
        panic: 双重嵌套：panic

goroutine 1 [running]:
main.main.func1.1()
        /home/zheng/STUDY/GoWork/demo/main.go:6 +0x39
panic(0x466460, 0x48a2b8)
        /usr/local/go/src/runtime/panic.go:965 +0x1b9
main.main.func1()
        /home/zheng/STUDY/GoWork/demo/main.go:8 +0x5b
panic(0x466460, 0x48a2c8)
        /usr/local/go/src/runtime/panic.go:965 +0x1b9
main.main.func2()
        /home/zheng/STUDY/GoWork/demo/main.go:11 +0x39
panic(0x466460, 0x48a298)
        /usr/local/go/src/runtime/panic.go:965 +0x1b9
main.main()
        /home/zheng/STUDY/GoWork/demo/main.go:13 +0x68
```

图解：

![panic4](http://cdn.mjava.top/blog/panic4.jpg)

## recovery

这个函数的目的很简单，就是把`_panic`结构体中的`recovered`字段设为`true`，不过它在执行修改前会判断当前有没有`panic`和当前`panic`有没有被恢复过。可以通过源码[`runtime.gorecover`](https://draveness.me/golang/tree/runtime.gorecover)中体现出来

```go
func gorecover(argp uintptr) interface{} {
	gp := getg()
	p := gp._panic
	if p != nil && !p.recovered && argp == uintptr(p.argp) {
		p.recovered = true
		return p.arg
	}
	return nil
}
```

所以`recovery`要通过`defer`在`panic`之后调用才会生效