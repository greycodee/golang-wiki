

> 本文为翻译文章
>
> 原文地址：https://golangbot.com/concurrency/

## 简介

Go 是一种并发语言，而不是并行语言。在讨论如何在 Go 中处理并发之前，我们必须首先了解什么是并发以及它与并行性有何不同。

## 什么是并发？

并发是一次处理很多事情的能力。最好用一个例子来解释。

让我们考虑一个慢跑的人。可以说，在他早上跑步时，他的鞋带解开了。现在，该人停止跑步，系上鞋带，然后再次开始跑步。这是并发的经典示例。该人能够处理跑步和绑鞋带，也就是说，该人能够一次处理很多事情：）

## 什么是并行性，它与并发有何不同？

并行性可以同时做很多事情。这听起来可能类似于并发，但实际上是不同的。让我们通过相同的慢跑示例更好地理解它。在这种情况下，我们假设该人正在慢跑并且也在 iPod 上听音乐。在这种情况下，这个人同时在慢跑和听音乐，也就是说，他同时在做很多事情。这称为并行性。

## 并发和并行-技术观点

通过使用实际示例，我们了解了什么是并发以及它与并行性有何不同。现在，让我们从极客的角度从技术角度来看它们：)。

假设我们正在编程一个 Web 浏览器。 Web 浏览器具有各种组件。其中两个是网页渲染区域和用于从 Internet 下载文件的下载器。假设我们已经以这样的方式构造了浏览器的代码，使得每个组件都可以独立执行（这是使用 Java 之类的语言中的线程完成的，而在 Go 中，我们可以使用 Goroutines 来实现，稍后会详细介绍）。当此浏览器在单核处理器中运行时，处理器将在浏览器的两个组件之间进行上下文切换。它可能会下载文件一段时间，然后可能会切换以呈现用户请求的网页的 html，这称为并发。并发进程在不同的时间点开始，并且它们的执行周期重叠。在这种情况下，下载和渲染在不同的时间点开始，并且它们的执行重叠。假设同一浏览器在多核处理器上运行。在这种情况下，文件下载组件和 HTML 呈现组件可能在不同的内核中同时运行。这就是所谓的并行性。

![concurrency-parallelism-copy](http://cdn.mjava.top/blog/concurrency-parallelism-copy.png)

并行不会总是导致更快的执行时间。这是因为并行运行的组件可能必须相互通信。例如，在我们的浏览器中，文件下载完成后，这应该传达给用户，例如使用弹出窗口。这种通信发生在负责下载的组件和负责渲染用户界面的组件之间。在并发系统中，此通信开销很低。在组件在多个内核中并行运行的情况下，此通信开销很高。因此，并行程序并不总是会导致更快的执行时间！

## Go 对并发的支持

并发是 Go 编程语言的固有部分。并发在 Go 中使用 [Goroutine](https://golangbot.com/goroutines/) 和通道进行处理。我们将在即将到来的教程中详细讨论它们。