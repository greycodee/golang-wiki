# io包

## 简介
`io`包提供了`系统io`最基本的封装，可以用来进行文件的读取，写入，复制等功能。不是线程安全的。

## 思维导图概览
由于思维导图比较大，这边就不放图了,提供两种途径观看
- [在线浏览思维导图](https://www.processon.com/view/link/605c48f2e401fd4c0398b4cd#map)
- [下载思维导图（可以用xmind工具打开）](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/mind/io.xmind)

## 功能
上面思维导图中虽然方法，接口一大堆，但是我们正常用的话用下面的几个函数就可以了，这几个函数也是对`io`包里的方法进一步封装。
![](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/xmind_go_io_func1.png)

### 复制

#### func Copy

```go
func Copy(dst Writer, src Reader) (written int64, err error)
```
这个函数可以进行文件夹的复制功能，`dest`参数是目标文件，`src`参数是源文件。可以实现`src`文件复制到`dst`文件的功能。返回值`written`是复制文件的大小

代码示例：
> 将`字符串流`复制到了`系统输出流`里，然后输出了复制的数据大小
```go
package main

import (
	"fmt"
	"io"
	"os"
	"strings"
)

func main() {
	src:=strings.NewReader("这是源文件字符串流")
	l,_:=io.Copy(os.Stdout,src)
	fmt.Printf("\n复制的数据大小：%d",l)
}
```
输出：
```shell
这是源文件字符串流
复制的数据大小：27
```

#### func CopyBuffer
```go
func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error)
```
`CopyBuffer`与`Copy`相同，区别在于`CopyBuffer`使用提供的缓冲区，可以自定义缓冲区大小，而不是分配一个临时缓冲区。**如果`buf`为`nil`，则分配一个临时缓冲区（`func Copy`就是调用了这个函数，然后`buf`传了`nil`）**；如果长度为`0`，则`CopyBuffer`会报异常。

> 代码示例：自定义大小为`100`字节的缓冲区，然后将`字符串流`复制到了`系统输出流`里，然后输出了复制的数据大小
```go
package main

import (
	"fmt"
	"io"
	"os"
	"strings"
)

func main() {
	src:=strings.NewReader("这是源文件字符串流")
	buf:=make([]byte,100)
	l,_:=io.CopyBuffer(os.Stdout,src,buf)
	fmt.Printf("\n复制的数据大小：%d",l)
}
```
输出：
```shell
这是源文件字符串流
复制的数据大小：27
```

#### func CopyN
```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```
这个函数可以自定义复制的流大小`n`，底层实现是先调用`LimitReader`函数，然后将结果作为`src`再复制到`dest上`。

> 代码示例：复制**7个字节**的数据到`系统输出流`，然后输出已复制的数据大小
```go
package main

import (
	"fmt"
	"io"
	"os"
	"strings"
)

func main() {
	src:=strings.NewReader("this is test!")
	l,_:=io.CopyN(os.Stdout,src,7)
	fmt.Printf("\n复制的数据大小：%d",l)
}
```
输出：
```go
this is
复制的数据大小：7
```

### 读取
#### func ReadAll
```go
func ReadAll(r Reader) ([]byte, error)
```
读取`r`的全部字节，然后返回`[]byte`
示例代码：

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	src:=strings.NewReader("this is test!")
	b,_:=io.ReadAll(src)
	fmt.Printf("读取的字符串：%s",b)
}

```
输出：
```shell
读取的字符串：this is test!
```

#### func ReadAtLeast
```go
func ReadAtLeast(r Reader, buf []byte, min int) (n int, err error)
```
- 参数`r`表示要读取的`io流`
- 参数`buf`是自定义的缓冲区，也是数据接收区
- 参数`min`表示最少要读取的字节数

这个函数会根据你传入的`buf`大小来读取多少数据。比如你传入的`buf`的大小为`4`个字节，这个方法就只会从`r`中读取`4`个字节的数据

!> 注意：自定义缓冲区`buf`要大于最小读取字节数`min`。如果自定义缓冲区小于等于最小读取字节数`min`，则会发生异常

代码示例：
```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	src:=strings.NewReader("this is test!")
	buf:=make([]byte,4)
	l,_:=io.ReadAtLeast(src,buf,2)
	fmt.Printf("读取的数据：%s\n",buf)
	fmt.Printf("读取的字节数：%d",l)
}
```
输出：
```shell
读取的数据：this
读取的字节数：4
```

#### func ReadFull
```go
func ReadFull(r Reader, buf []byte) (n int, err error)
```
这个函数功能和`ReadAtLeast`函数一样，而且内部就是调用了这个函数，只是`ReadAtLeast`的`min`值就是`buf`的长度。
所以这个函数的功能也是根据你传入的`buf`大小来读取多少数据。

### 写入
#### func WriteString
```go
func WriteString(w Writer, s string) (n int, err error)
```
- 参数`w`是要写入的目标流
- 参数`s`是要写入的字符串数据
这个函数的功能是将字符串`s`写入到`w`流中，然后返回写入的数据大小`n`

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	l,_:=io.WriteString(os.Stdout,"go test!")
	fmt.Printf("\n写入的数据大小：%d",l)
}

```
输出：
```shell
go test!
写入的数据大小：8
```
### 同步管道流
#### func pipe
```go
func Pipe() (*PipeReader, *PipeWriter)
```
这个函数提供了一个安全读取流的功能，调用这个函数后，会返回一个`写入流`，一个`读取流`。**当未读取`*PipeReader`里的数据时，`*PipeWriter`始终为阻塞状态，而当读取`*PipeReader`里的数据时,`*PipeReader`就变为阻塞状态，期间不能往管道里写入数据**。

也就是说这个是线程安全的方法，在任一时刻，要么只能读取数据，要么只能写入数据。

```go
package main

import (
	"fmt"
	"io"
	"os"
	"time"
)

func main() {
	r, w := io.Pipe()
	go func() {
		io.WriteString(w,"go pip test")
		io.WriteString(w,"\ngo test2")
		// 调用`Close`方法代表数据写入完成，释放写锁
		w.Close()
	}()
	fmt.Println("2秒后开始准备读取数据")
	time.Sleep(2*time.Second)
	io.Copy(os.Stdout, r)
}
```
输出：
```shell
2秒后开始准备读取数据
go pip test
go test2
```