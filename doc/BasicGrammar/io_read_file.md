>【译文】
>
> 原文地址：https://golangbot.com/read-files/

## 将整个文件读入内存

最基本的文件操作之一是将整个文件读取到内存中。这是借助[ioutil](https://golang.org/pkg/io/ioutil/)包的[ReadFile](https://golang.org/pkg/io/ioutil/#ReadFile)函数完成的。

让我们阅读一个文件并打印其内容。

我通过运行`filehandling`在`Documents`目录中创建了一个文件夹`mkdir ~/Documents/filehandling`。

`filehandling`通过从`filehandling`目录运行以下命令来创建一个名为Go的模块。

```go
go mod init filehandling  
```

我有一个文本文件`test.txt`，可以从我们的Go程序中读取`filehandling.go`。`test.txt`包含以下字符串

```go
Hello World. Welcome to file handling in Go.  
```

这是我的目录结构。

```shell
├── Documents
│   └── filehandling
│       ├── filehandling.go
|       ├── go.mod
│       └── test.txt
```

让我们立即开始编写代码。创建一个`filehandling.go`具有以下内容的文件。

```go
package main

import (  
    "fmt"
    "io/ioutil"
)

func main() {  
    data, err := ioutil.ReadFile("test.txt")
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

请在您的本地环境中运行此程序，因为无法在playground上读取文件。

行号 上面程序的第9行读取文件并返回存储在中的字节`data`。在14行我们转换`data`为a`string`并显示文件的内容。

请从存在**test.txt**的位置运行该程序。

如果**test.txt**位于**〜/ Documents / filehandling**，则使用以下步骤运行该程序，

```shell
cd ~/Documents/filehandling/  
go install  
filehandling
```

如果您不知道如何运行Go程序，请访问https://golangbot.com/hello-world-gomod/了解更多信息。如果您想了解有关程序包和Go模块的更多信息，请访问https://golangbot.com/go-packages/

该程序将打印：

```shell
Contents of file: Hello World. Welcome to file handling in Go.  
```

例如，如果从任何其他位置运行该程序，请尝试从以下位置运行该程序 `~/Documents/`

```shell
cd ~/Documents/  
filehandling
```

它将显示以下错误。

```shell
File reading error open test.txt: no such file or directory  
```

原因是Go是一种编译语言。`go install`做的是它会创建从源代码的二进制。二进制文件与源代码无关，可以在任何位置运行。由于`test.txt`找不到运行二进制文件的位置，因此程序会抱怨找不到指定的文件。

有三种方法可以解决此问题，

1. 使用绝对文件路径
2. 将文件路径作为命令行标志传递
3. 将文本文件与二进制文件捆绑在一起

### 1.使用绝对文件路径

解决此问题的最简单方法是传递绝对文件路径。我已经修改了程序，并将路径更改为第1行中的绝对路径。请将此路径更改为您的的绝对位置`test.txt`。

```go
package main

import (  
    "fmt"
    "io/ioutil"
)

func main() {  
    data, err := ioutil.ReadFile("/home/naveen/Documents/filehandling/test.txt")
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

现在，该程序可以从任何位置运行，它将打印的内容`test.txt`。

例如，即使我从主目录运行它，它也能正常工作

```shell
cd ~/Documents/filehandling  
go install  
cd ~  
filehandling  
```

该程序将打印内容 `test.txt`

这似乎是一种简单的方法，但存在一个陷阱，即文件应位于程序中指定的路径中，否则此方法将失败。

### 2.将文件路径作为命令行标志传递

解决此问题的另一种方法是将文件路径作为命令行参数传递。使用[flag](https://golang.org/pkg/flag/)包，我们可以从命令行获取文件路径作为输入参数，然后读取其内容。

首先让我们了解一下该`flag`程序包是如何工作的。该`flag`软件包具有[String](https://golang.org/pkg/flag/#String) [函数](https://golangbot.com/functions/)。此函数接受3个参数。第一个是标志的名称，第二个是默认值，第三个是标志的简短描述。

让我们编写一个小程序来从命令行读取文件名。内容替换`filehandling.go`为以下，

```go
package main  
import (  
    "flag"
    "fmt"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    fmt.Println("value of fpath is", *fptr)
}
```

上面程序的第8行，使用该函数创建一个`fpath`带有默认值`test.txt`和描述的字符串标志。该函数返回存储标志值的字符串[变量](https://golangbot.com/variables/)的地址。`file path to read from``String`

在程序访问任何标志之前，应调用*flag.Parse（）*。

我们在行号中打印标志的值。10

使用以下命令运行该程序时

```shell
filehandling -fpath=/path-of-file/test.txt  
```

我们将其`/path-of-file/test.txt`作为标志的值传递`fpath`。

该程序输出

```shell
value of fpath is /path-of-file/test.txt  
```

如果仅使用程序运行`filehandling`而未传递任何程序`fpath`，它将打印

```shell
value of fpath is test.txt  
```

由于`test.txt`是的默认值`fpath`。

现在我们知道了如何从命令行读取文件路径，让我们继续完成文件读取程序。

```go
package main  
import (  
    "flag"
    "fmt"
    "io/ioutil"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    data, err := ioutil.ReadFile(*fptr)
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

上面的程序读取从命令行传递的文件路径的内容。使用以下命令运行该程序

```shell
filehandling -fpath=/path-of-file/test.txt  
```

请替换`/path-of-file/`为的绝对路径`test.txt`。例如，以我为例，我运行了命令

```shell
filehandling --fpath=/home/naveen/Documents/filehandling/test.txt  
```

并打印程序。

```shell
Contents of file: Hello World. Welcome to file handling in Go.  
```

### 3.将文本文件与二进制文件捆绑在一起

从命令行获取文件路径的上述选项很好，但是有一种更好的方法可以解决此问题。如果我们能够将文本文件和二进制文件捆绑在一起，那会很棒吗？这就是我们下一步要做的。

有各种[软件包](https://golangbot.com/go-packages/)可以帮助我们实现这一目标。我们将使用[packr v2，](https://github.com/gobuffalo/packr/tree/master/v2)因为它非常简单，并且在我的项目中一直使用它，没有任何问题。

第一步是安装`packr`。

在`~/Documents/filehandling/`目录中的命令提示符处键入以下命令以安装软件包

```shell
cd ~/Documents/filehandling/  
go get -u github.com/gobuffalo/packr/v2/... 
```

*packr*转换静态文件，如`.txt`给`.go`其随后直接嵌入到二进制文件。Packer具有足够的智能，可以在开发过程中从磁盘而不是从二进制文件中获取静态文件。这样，仅在静态文件发生更改时，就无需在开发过程中重新编译。

一个程序将使我们更好地理解事物。内容替换`filehandling.go`为以下

```go
package main

import (  
    "fmt"

    "github.com/gobuffalo/packr/v2"
)

func main() {  
        box :=  packr.New("fileBox", "../filehandling")
        data, err := box.FindString("test.txt")
        if err != nil {
            fmt.Println("File reading error", err)
                return
        }
        fmt.Println("Contents of file:", data)
}
```

上面程序的第10行，我们正在创建一个名为的新盒子`box`。一个框代表一个文件夹，其内容将嵌入二进制文件中。在这种情况下，我要指定`filehandling`包含的文件夹`test.txt`。在下一行中，我们使用`FindString`方法读取文件的内容并进行打印。

使用以下命令运行程序。

```shell
cd ~/Documents/filehandling  
go install  
filehandling  
```

程序将打印的内容`test.txt`。

由于我们现在处于开发阶段，因此将从磁盘读取文件。尝试更改的内容，`test.txt`然后`filehandling`再次运行。您可以看到该程序`test.txt` 无需重新编译即可打印的更新内容。完美的 ：）。

Packr还能够找到盒子的绝对路径。因此，该程序可以从任何目录运行。它不需要`test.txt`出现在当前目录中。cd到另一个目录，然后再次尝试运行该程序。

```shell
cd ~/Documents  
filehandling  
```

运行上述命令还将打印的内容`test.txt`。

现在，我们继续下一步，并将其捆绑`test.txt`到我们的二进制文件中。我们使用`packr2`命令来执行此操作。

`packr2`从`filehandling`目录运行命令。

```shell
cd ~/Documents/filehandling/  
packr2  
```

此命令将在源代码中搜索新框，并生成Go文件，该文件包含我们的test.txt文本文件，该文本文件已转换为字节，并且可以与Go二进制文件捆绑在一起。该命令将生成一个文件`main-packr.go`和一个包`packrd`。这两个是捆绑静态文件和二进制文件所必需的。

运行上述命令后，再次编译并运行该程序。该程序将打印的内容`test.txt`。

```shell
go install  
filehandling  
```

运行时，`go install`您可能会收到以下错误。

```shell
build filehandling: cannot load Users/naveen/Documents/filehandling/packrd: malformed module path "Users/naveen/Documents/filehandling/packrd": missing dot in first path element  
```

这可能是因为packr2不知道我们正在使用Go模块。如果出现此错误，请尝试通过运行命令将go modules显式设置为on `export GO111MODULE=on`。将Go模块设置为打开后，必须清除并重新生成生成的文件。

```shell
packr2 clean  
packr2  
go install  
filehandling  
```

现在`test.txt`将打印的内容，并从二进制文件中读取该内容。

如果您怀疑是从二进制文件还是从磁盘提供文件，建议您删除`test.txt`并`filehandling`再次运行该命令。您可以看到`test.txt`的内容已打印。非常棒的：D我们已经成功地将静态文件嵌入到我们的二进制文件中。

## 小块读取文件

在上一节中，我们学习了如何将整个文件加载到内存中。当文件的大小过大时，将整个文件读入内存是没有意义的，尤其是在RAM不足的情况下。更好的方法是分小块读取文件。这可以在[bufio](https://golang.org/pkg/bufio)软件包的帮助下完成。

让我们编写一个程序，该程序`test.txt`以3个字节的块为单位读取文件。运行`packr2 clean`以删除上一部分中由packr生成的文件。`test.txt`如果您删除了它，您可能还想重新创建。内容替换`filehandling.go`为以下，

```go
package main

import (  
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
            log.Fatal(err)
        }
    }()
    r := bufio.NewReader(f)
    b := make([]byte, 3)
    for {
        n, err := r.Read(b)
        if err != nil {
            fmt.Println("Error reading file:", err)
            break
        }
        fmt.Println(string(b[0:n]))
    }
}
```

在上面程序的15行中，我们使用从命令行标志传递的路径打开文件。

在行号 19，我们推迟文件关闭。

上面程序的24行将创建一个新的缓冲读取器。在下一行中，我们创建一个长度为3的字节片，文件的字节将被读入其中。

`Read` [方法 ](https://golangbot.com/methods/)第27行读取最多len（b）个字节，即最多3个字节，并返回读取的字节数。我们将返回的字节存储在变量中`n`。在32行中，将切片从index读取`0`到`n-1`，即直到该`Read`方法返回并打印的字节数为止。

一旦到达文件末尾，它将返回EOF错误。该程序的其余部分很简单。

如果我们使用命令运行上面的程序

```shll
cd ~/Documents/filehandling  
go install  
filehandling -fpath=/path-of-file/test.txt 
```

将输出以下内容

```shell
Hel  
lo  
Wor  
ld.  
 We
lco  
me  
to  
fil  
e h  
and  
lin  
g i  
n G  
o.  
Error reading file: EOF  
```

## 逐行读取文件

在本节中，我们将讨论如何使用Go逐行读取文件。这可以使用[bufio](https://golang.org/pkg/bufio/)包来完成。

请更换的内容`test.txt`与以下

```
Hello World. Welcome to file handling in Go.  
This is the second line of the file.  
We have reached the end of the file.  
```

以下是逐行读取文件所涉及的步骤。

1. 打开文件
2. 从文件创建一个新的扫描
3. 扫描文件并逐行读取。

更换的内容`filehandling.go`与以下

```go
package main

import (  
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {  
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
        log.Fatal(err)
    }
    }()
    s := bufio.NewScanner(f)
    for s.Scan() {
        fmt.Println(s.Text())
    }
    err = s.Err()
    if err != nil {
        log.Fatal(err)
    }
}
```

在上面程序的15行中，我们使用从命令行标志传递的路径打开文件。在行号 24，我们使用该文件创建一个新的扫描。`scan()`方法中第 25行将读取文件的下一行，并且所读取的字符串将通过该`text()`方法可用。

扫描返回后`false`，该`Err()`方法将返回扫描过程中发生的任何错误。如果错误是“ End of File”，`Err()`将返回`nil`。

如果我们使用命令运行上面的程序：

```shell
cd ~/Documents/filehandling  
go install  
filehandling -fpath=/path-of-file/test.txt  
```

文件的内容将逐行打印，如下所示。

```shell
Hello World. Welcome to file handling in Go.  
This is the second line of the file.  
We have reached the end of the file.  
```

这使我们到了本教程的结尾。希望你喜欢。
