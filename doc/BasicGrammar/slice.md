## 说明

`切片(Slice)`是`go`提供的一种便捷操作**数组**的数据结构。可以按需自动增长和缩小数组，切片通过`append`函数来动态添加底层数组数据。因为切片的底层是数组，所以在内存中是在连续内存块中分配内存的，所以切片可以和数组一样获得索引，迭代以及垃圾回收优化等好处。

## 声明切片

### make声明切片

```go
// 声明切片 容量为4  类型int初始化默认值为0
slice:=make([]int,4)
fmt.Println(slice)

// 输出
[0 0 0 0]
```

![slice1](../../../images/slice/slice1.jpg)

### 切片字面量

> 如果在`[]`运算符里指定了一个值，那么创建的就不是切片，而是数组。只有不指定值的时候才会创建切片

```go
// 创建一个容量为4的切片，并赋初始值
slice:=[]int{1,2,3,4}
fmt.Println(slice)

// 输出
[1 2 3 4]
```

![slice2](../../../images/slice/slice2.jpg)

### `nil`切片

```go
// 声明nil切片  此时不会初始化默认值
var slice []int
fmt.Println(slice)

// 输出
[]
```

### 空切片

```go
// 空切片 没有初始化默认值
slice:=[]int{}
fmt.Println(slice)

// 输出
[]
```

![slice3](../../../images/slice/slice3.jpg)

## 两个切片共用一个底层数据

### 修改数据操作

- 声明两个切片共用一个底层数组

    > **slice2 := slice[1:3]**可以看成**slice2 := slice[ｘ:y]**，因为slice底层数组容量为４，
    >
    > 所以slice2的长度为：**y-x**（3-1=2）。容量为：**4-x**（4-1=3）

    ```go
    slice := []int{1,2,3,4}
    slice2 := slice[1:3]
    fmt.Println(slice)
    fmt.Println(slice2)
    
    // 输出
    [1 2 3 4]
    [2 3]
    ```

![slice4](../../../images/slice/slice4.jpg)

- 修改slice2切片的数据

  > 同时也修改了slice的值

  ```go
  slice2[1] = 99
  fmt.Println(slice)
  fmt.Println(slice2)
  
  // 输出
  [1 2 99 4]
  [2 99]
  ```

  ![slice5](../../../images/slice/slice5.jpg)

### 添加数据操作

在上面数据基础上添加一个数据到**slice2**

> 同时会修改**slice**的值，因为底层数组是同一个

```go
slice2 = append(slice2,66)
fmt.Println(slice)
fmt.Println(slice2)

// 输出
[1 2 99 66]
[2 99 66]
```

![slice6](../../../images/slice/slice6.jpg)

###　使添加操作不影响原切片

解决这个问题就是在赋值时**使用切片字面量的第三个参数**

> **slice2 := slice[1:２:2] **可以看成**slice2 := slice[x:y:z] **因为slice底层数组容量为４，
>
> 所以slice2的长度为：**y-x**（2-1=1）， slice2的容量为：**z-x**（2-1=1）

```go
slice := []int{1,2,3,4}
//slice2 := slice[1:3]
// 设置限制容量为2-1=1 
slice2 := slice[1:２:2]
// 添加参数slice2将进行扩容　扩容到原来的２倍
slice2 = append(slice2,66)

fmt.Println(slice)
fmt.Println(slice2)

// 输出
[1 2 3 4]
[2 66]
```

![slice7](../../../images/slice/slice7.jpg)

## 扩容

在切片容量小于**1000个元素**时，总是会成倍地增加容量，一旦元素超过1000，容量的**增长因子会设为1.25**，也就是会每次增加25%的容量。随着`go`不断的迭代更新，这种增长算法可能会有所改变。

```go
slice := []int{1,2,3,4}
fmt.Printf("扩容前容量：%d\n",cap(slice))
fmt.Println("内容：",slice)
fmt.Println("------------------------")
slice = append(slice,88)
fmt.Printf("扩容后容量：%d\n",cap(slice))
fmt.Println("内容：",slice)

// 输出
扩容前容量：4
内容： [1 2 3 4]
------------------------
扩容后容量：8
内容： [1 2 3 4 88]
```

![slice8](../../../images/slice/slice8.jpg)

