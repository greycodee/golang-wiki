## make

`make`只可以用来创建`map`，`slice`，`chan`这三个类型，而且它是直接返回类型本身。可以看看它的源码介绍：

> 贴心的翻译了一下

```go
// make内置函数分配并初始化一个切片，map或chan类型的对象。
// 与new相同点：第一个参数是类型，而不是具体值
// 与new不同点：make的返回类型与其参数相同的类型，而不是指向它的指针。
// 结果的规格取决于类型：
//
// 切片：大小指定长度。切片的容量等于其长度可以提供第二个整数参数指定其他容量；它必须不小于长度。
// 例如：make（[] int，0，10）分配一个基础数组大小为10，并返回长度为0且容量为10的切片，即由该基础数组支持。
//
// map：为空map分配了足够的空间来容纳指定的元素数。在这种情况下，可以省略容量分配的起始大小较小。
//
// Channel：使用指定的值初始化通道的缓冲区容量。如果为零或忽略大小，则通道为无缓冲
func make(t Type, size ...IntegerType) Type
```

### 使用

```go
// 切片
// 创建一个容量为0，长度为0的切片
slice1 := make([]int,0)
// 创建一个容量为10，长度为0的切片
slice2 := make([]int,0,10)

// map
// 创建一个没有指定容量的map
map1 := make(map[string]string)
// 创建一个指定容量为10的map
map2 := make(map[string]string,10)

// chan
// 创建一个缓冲容量为100的int通道
c := make(chan int,100)
// 创建一个无缓冲int通道
c2 := make(chan int)

```

## new

`new`可以用来为类型在堆中申请内存（当然也包括`map`，`slice`，`chan`）。返回的值是指向该类型新分配的零值的**指针**

源码介绍：

```go
// new是内置函数分配内存。参数是类型，而不是值，返回的值是指向该类型新分配的零值的指针。
func new(Type) *Type
```

### 使用

```go
// 创int  `*i`的默认值初始化为0
i := new(int)

// 创建一个长度为0  容量为0的切片
slice := new([]int)
```

- 创建map就比较麻烦了，创建后还要进行初始化才可以继续使用

  ```go
   //使用new创建一个map指针
  ma := new(map[string]int)                                                                                                                                          
  //第一种初始化方法
  *ma = map[string]int{}
  (*ma)["a"] = 44
  fmt.Println(*ma)
  
  //第二种初始化方法
  *ma = make(map[string]int, 0)
  (*ma)["b"] = 55
  fmt.Println(*ma)
  
  //第三种初始化方法
  mb := make(map[string]int, 0)
  mb["c"] = 66
  *ma = mb
  (*ma)["d"] = 77
  fmt.Println(*ma)
  ```

虽然官方提供了`new`这个函数，但是我们一般都比较少用它，因为确实不怎么方便。除非一些特殊的场景才会用到它。

## 比较

相同点：

- 都是在堆上分配内存空间

不同点：

- make: 只用于slice、map以及channel的初始化， 无可替代

- new: 用于类型内存分配(初始化值为0)， 不常用