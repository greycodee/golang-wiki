
## 指针的两个关键符号

- `*`：返回所引用变量内存地址上的值
- `&`：返回变量的内存地址

## 代码示例

首先可以看看下面这段代码的两个输出分别是什么？

```go
package main

import (
	_ "embed"
	"fmt"
)

type person struct {
	Name 		string
	Alias  		string
	Summary   	string
}

// 值接收器
func (p person) SetSummary() {
	p.Summary = fmt.Sprintf("我的名字叫%s,大家都叫我%s",p.Name,p.Alias)
}

// 指针接收器
func (p *person) SetSummaryByPointer() {
	p.Summary = fmt.Sprintf("我的名字叫%s,大家都叫我%s",p.Name,p.Alias)
}

func (p person) SaySummary() string {
	return p.Summary
}

func main(){
	per1 := person{Name:"程序员", Alias:"码农"}
	per1.SetSummary()
	fmt.Printf("第一段：%s\n",per1.SaySummary())
	per1.SetSummaryByPointer()
	fmt.Printf("第二段：%s\n",per1.SaySummary())

}
```

输出：

```shell
第一段：
第二段：我的名字叫程序员,大家都叫我码农
```

可以看到第一次`SetSummary()`的方法并为成功设置`Summary`字段的值，所以导致输出为空。而`SetSummaryByPointer()`却成功设置了`Summary`的值。

## 什么是接收器

首先我们要了解一下什么是接收器，看下面这张图就明白了。每个方法只能有一个接收器。

> 接收器类型可以是（几乎）任何类型，不仅仅是结构体类型，任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型，但是**接收器不能是一个接口类型**，因为接口是一个抽象定义，而方法却是具体实现，如果这样做了就会引发一个编译错误`invalid receiver type…`。



![](http://cdn.mjava.top/blog/20210327162529.png)

## 值传递

为什么上面的`SetSummary()`方法没有成功设置`Summary`的值呢？

我们了解到在默认情况下Go 语言使用的是**值传递**,在调用函数时**将实际参数复制一份传递到函数中**，这样在函数中如果对参数进行修改，将不会影响到实际参数。

所以接收器接收方（`p`在上面的示例中）的行为就好像它是该方法的参数一样。拥有和**值传递**一样的特性，方法里的修改对`p`来说是不可见的，因为它修改的只是`p`的副本数据。

而指针接收器也是接受了`p`的副本，但是它是`p`的地址副本，所以方法内修改是直接在对应的内存地址上进行了修改，所以原先`p`数据对应的地址发生了改变，`p`随之也发生改变。

## 总结

1. 如果要修改接收器的值，则使用指针接收器
2. 如果接收器很大，`struct`例如很大，优先考虑指针接收器
3. 对于基本类型，切片和small之类的类型，`structs`值接收器效率更高
4. 因此，除非该方法的语义要求使用指针，否则值接收器将高效且清晰。