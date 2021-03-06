## 为什么这么快？

为什么`go`能支持高并发，它和`Java`的多线程有什么不同？

常规的多线程是由`CPU`直接调度的，其中大部分时间花在了上下文切换上面，所以后面就了了`协程(co-routine)`，用于减少上下文切换。

![协程](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/goroutine.jpg)

## GMP模型

`go`的高并发特性秘密就是`GMP模型`

![gmpall](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/gmpall.jpg)

## M0和G0

**M0**

`M0`是启动程序后的编号为0的主线程，这个M对应的实例会在全局变量runtime.m0中，不需要在`heap`上分配，`M0`负责执行初始化操作和启动第一个`G`， 在之后`M0`就和其他的`M`一样了。

**G0**

`G0`是每次启动一个M都会第一个创建的`gourtine`，`G0`仅用于负责调度的`G`，`G0`不指向任何可执行的函数, 每个`M`都会有一个自己的`G0`。在调度或系统调用时会使用`G0`的栈空间, 全局变量的`G0`是`M0`的`G0`。

![gmpfirst](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/gmpfirst.jpg)

## 新的协程被创建和执行

当有新的协程G被创建时，**会优先放入被创建的当前P本地队列**，如果本地P队列满了，则放入全局G队列。然后P通过`G0`调度到新的G，然后`G0`退出，P执行新的G。

![gmp_create_g](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/gmp_create_g.jpg)

## 自旋线程

当本地P队列为空，当前线程就会变成自旋线程，此时`G0`不断的寻找可执行的G(优先从全局G队列查找)，然后找到后放入本地P队列，然后P切换到新找到的G继续执行。

![gmp_p_empty](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/gmp_p_empty.jpg)

## work stealing机制

上面的自旋线程是优先从全局G队列里查找可执行的G，当全局G队列也为空时，他就会从其它的P队列里偷取G，然后放入本地P队列。![gmp_workst](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/gmp_workst.jpg)

## hand off机制

如果当前线程的G进行系统阻塞调用时，如进行`time.sleep`，则当前线程就会释放P，然后把P转交给其它空闲的线程执行，如果没有闲置的线程，则创建新的线程

![hand_off](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/hand_off.jpg)

## 本地P队列已满

如果本地P队列满了，将本地P队列前一半打乱顺序，然后和新的G一起放入全局G队列![p_m2](https://cdn.jsdelivr.net/gh/greycodee/golang-wiki@main/images/gmp/p_m2.jpg)