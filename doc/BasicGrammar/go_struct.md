## 结构体组成

`go`结构体的结构体主要由`type`和`struct`两个关键字组成

> 当字段首字母为大写时，可以被外部包访问，相当于`Java`的`public`字段，
>
> 当字段首字母为小写时，只能被当前包访问，外部包不能访问，相当于`Java`的`private`字段
>
> 结构体的名称也一样，小写时外部包就无法访问到这个结构体了。

```go
type Address struct {
	Province string
	City	 string
	Country  string
	Detail   string
}
```

![struct](http://cdn.mjava.top/blog/struct.jpg)

## 声明结构体

- 字面量声明

  ```go
  addr:=Address{
      Province: "浙江省",
      City:     "杭州市",
      Country:  "滨江区",
      Detail:   "阿里巴巴",
  }
  ```

- `var`声明

  ```go
  var addr Address
  addr.Province="浙江省"
  addr.City="杭州市"
  addr.Country="滨江区"
  addr.Detail="阿里巴巴"
  ```

  

## 嵌套

一个结构体也可以包含另一个结构体，例如：

```go
type People struct {
	Name string
	Addr Address
	Age  int
	alias string
}

type Address struct {
	Province string
	City	 string
	Country  string
	Detail   string
}
```

上面的`People`结构体包含了`Address`结构体，然后在声明的时候可以这样

```go
zheng:=People{
    Name:  "",
    Addr:  Address{
        Province: "",
        City:     "",
        Country:  "",
        Detail:   "",
    },
    Age:   0,
    alias: "",
}
```

## 方法

`go`的函数的接收器就是对应这个接收器的方法

![method](http://cdn.mjava.top/blog/method.jpg)

### 值接收器

使用值接收器时，在方法里对接收器进行修改时，**不会影响**源接收器的数据

```go
func (p People) Info()  {
    
}
```

### 指针接收器

使用指针接收器时，在方法里对接收器进行修改**会影响**源接收器的数据

```go
func (p *People) Update()  {
    
}
```

## 多态

当多个方法实现同一个接口的方法时，就可以使用多态了

```go
import "fmt"

func main() {

	var li Lilei
	speak(li)
    
	var z ZhangSan
	speak(z)
}
// 函数
func speak(say Say)  {
	say.Say()
}
// 定义接口
type Say interface {
	Say()
}
// 结构体
type Lilei struct {

}
type ZhangSan struct {

}
// 实现方法
func (l Lilei) Say()  {
	fmt.Println("我是李雷")
}
func (z ZhangSan) Say()  {
	fmt.Println("我是张三")
}
```

## 标签

在我们定义结构体时，同时还可以给字段设置标签。一般常用的标签有`json`和`xml`。然后可以通过反射把字段映射到设置的标签上。

> 设置标签时 字段首字母要大写

- 单个标签

```go
type People struct {
    Name string `json:name`
}
```

- 多个标签

  同时也可以定义多个标签

  > 两个类型的标签中间用空格隔开

  ```go
  type People struct {
      Name string `json:"province" xml:"province"`
  }
  ```

  