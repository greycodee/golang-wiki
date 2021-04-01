# embed包
`embed`是`go 1.16`新引入的包，能够把静态文件和`go文件`一起编译成二进制文件。使我们能够更好的进行`web`的开发。没有这个包之前我们一般用`go-bindata`或`go-bindata-assetfs`先把静态文件编译成`go`文件，然后再用go命令编译成二进制文件，过程比较繁琐。现在官方提供了`embed`这个包后，可以直接通过一行命令(`//go:embed`)来导入静态文件，然后`go`命令编译时会把导入的静态文件也一起编译成二进制文件。



## 导入静态文件方式
`embed`提供了**3种**导入静态文件的方式，分别可以导入为`string`字符串，或`[]byte`字节数组，或`embed.FS`文件系统。
 - 导入为`string`字符串
```go
//go:embed text.txt
var content string
```
- 导入为`[]byte`字节数组
```go
//go:embed text.txt
var content []byte
```
- 导入为`embed.FS`文件系统
```go
//go:embed static
var static embed.FS
```

## embed.FS文件系统
当导入静态文件为`embed.FS`文件系统时，`go`内部就会构建一个类似`unix`的层级文件系统，**FS里的文件都是只读权限**，所以可以安全的在多线程中访问它。`FS`也提供了三个方法供我们使用
- Open(name)
- ReadDir(name)
- ReadFile(name)

### Open
```go
func (f FS) Open(name string) (fs.File, error)
```
- 参数`name`是`FS`文件系统里的路径

这个方法根据提供的`FS`文件系统里的路径,然后返回`File`类型，接下来可以使用`File`提供的一些方法来访问文件

### ReadDir
```go
func (f FS) ReadDir(name string) ([]fs.DirEntry, error)
```
- 参数`name`是`FS`文件系统里的路径
这个方法是**读取一个文件夹**，然后返回这个文件夹下所有的文件。

### ReadFile
```go
func (f FS) ReadFile(name string) ([]byte, error)
```
- 参数`name`是`FS`文件系统里的路径
这个方法是根据提供的文件路径，直接读取文件里的内容，然后返回`[]byte`

## 基本使用示例
首先在项目下建立一个`static`文件夹（当然名字随意命名，没有规定），然后在下面创建几个文件，分别在文件里填入**我是s1/s2/s3文件里的**具体如下目录结构

![](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/go_embed_demo.png)

### 导入为string
```go
package main

import (
	_ "embed"
	"fmt"
)

//go:embed static/s1.txt
var content string
func main() {
	fmt.Printf("静态文件内容：%s\n",content)
}
```
输出：
```shell
静态文件内容：我是s1文件里的
```

### 导入为[]byte
```go
package main

import (
	_ "embed"
	"fmt"
)

//go:embed static/s1.txt
var content []byte
func main() {
	fmt.Printf("静态文件内容：%s\n",string(content))
}
```
输出：
```shell
静态文件内容：我是s1文件里的
```

### 导入为FS文件系统

#### Open方法

```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	// 打开FS文件系统下路径为`static/s1.txt`的文件
	file,err:=content.Open("static/s1.txt")
	defer file.Close()
	if err != nil {
		fmt.Println(err)
	}

    // 读取File的数据
	var c = make([]byte,30)
	l,err:=file.Read(c)

	fmt.Printf("Open的内容：%s\n",c)
	fmt.Printf("Open读取的内容长度：%d\n",l)
}
```
输出：
```shell
Open的内容：我是s1文件里的
Open读取的内容长度：20
```

#### ReadDir方法
```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	// 打开文件
	files,err:=content.ReadDir("static")
	if err != nil {
		fmt.Println(err)
	}
	for _,f:=range files{
		fmt.Printf("文件名：%s\n",f.Name())
		fmt.Print("是否是文件夹:")
		fmt.Println(f.IsDir())
		fileInfo,_:=f.Info()
		fmt.Printf("文件权限：%s\n",fileInfo.Mode())
		fmt.Println("--------------------------------------")
	}
}
```
输出：
```shell
文件名：s1.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
文件名：s2.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
文件名：s3.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
```

#### ReadFile方法
```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	c2,_:=content.ReadFile("static/s1.txt")
	fmt.Printf("ReadFile内容：%s\n",c2)## 基本使用示例
首先在项目下建立一个`static`文件夹（当然名字随意命名，没有规定），然后在下面创建几个文件，分别在文件里填入**我是s1/s2/s3文件里的**具体如下目录结构

![](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/go_embed_demo.png)

### 导入为string
```go
package main

import (
	_ "embed"
	"fmt"
)

//go:embed static/s1.txt
var content string
func main() {
	fmt.Printf("静态文件内容：%s\n",content)
}
```
输出：
```shell
静态文件内容：我是s1文件里的
```

### 导入为[]byte
```go
package main

import (
	_ "embed"
	"fmt"
)

//go:embed static/s1.txt
var content []byte
func main() {
	fmt.Printf("静态文件内容：%s\n",string(content))
}
```
输出：
```shell
静态文件内容：我是s1文件里的
```

### 导入为FS文件系统

#### Open方法

```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	// 打开FS文件系统下路径为`static/s1.txt`的文件
	file,err:=content.Open("static/s1.txt")
	defer file.Close()
	if err != nil {
		fmt.Println(err)
	}

    // 读取File的数据
	var c = make([]byte,30)
	l,err:=file.Read(c)

	fmt.Printf("Open的内容：%s\n",c)
	fmt.Printf("Open读取的内容长度：%d\n",l)
}
```
输出：
```shell
Open的内容：我是s1文件里的
Open读取的内容长度：20
```

#### ReadDir方法
```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	// 打开文件
	files,err:=content.ReadDir("static")
	if err != nil {
		fmt.Println(err)
	}
	for _,f:=range files{
		fmt.Printf("文件名：%s\n",f.Name())
		fmt.Print("是否是文件夹:")
		fmt.Println(f.IsDir())
		fileInfo,_:=f.Info()
		fmt.Printf("文件权限：%s\n",fileInfo.Mode())
		fmt.Println("--------------------------------------")
	}
}
```
输出：
```shell
文件名：s1.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
文件名：s2.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
文件名：s3.txt
是否是文件夹:false
文件权限：-r--r--r--
--------------------------------------
```

#### ReadFile方法
```go
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var content embed.FS
func main() {
	c2,_:=content.ReadFile("static/s1.txt")
	fmt.Printf("ReadFile内容：%s\n",c2)
}
```
输出：
```shell
ReadFile内容：我是s1文件里的
```
}
```
输出：
```shell
ReadFile内容：我是s1文件里的
```