---
title: 浅谈Goroutine和Channel
author: GopherTy
top: true
cover: false
toc: true
date: 2019-11-02 14:13:20
coverImg:
password:
summary:
tags:
    - Go
    - Go 并发编程
categories: 编程语言
reprintPolicy:
---
# 浅谈 Goroutine 和 Channel

## 什么是 Goroutine

1.在说是 goroutine 之前先简单了解下进程、线程之间的关系。

一个进程简单来说就是跑在一台机器上的一个应用程序，它占有独立的内存地址空间。一个进程由一个或多个操作系统线程组成，这些线程共享该进程的内存地址空间。几乎所有的程序都是多线程的，一个并发程序可以在一个处理器或内核上使用多个线程来执行任务，但是同一个程序多个线程在某个时间点同时运行在多核或多处理器上才是真正的并行。

并行是一种通过使用多处理器以提高速度的能力。所以并发程序可以是并行的，也可以不是。

2.在 Go 中，应用程序每一个并发的执行单元被成为 goroutine （协程），在协程和操作系统线程之间并无一对一的关系：协程是根据一个或多个线程的可用性，映射（多路复用，执行于）在它们之上的；协程调度器在 Go 运行时很好的完成了这个工作。

3.当一个程序启动时，其主函数即在一个单独的 goroutine 中运行，我们叫它 main goroutine。新的goroutine 会用 go 语句来创建。在语法上，go 语句是一个普通的函数或方法调用前加上关键字 go 。go语句会使其语句中的函数在一个新创建的 goroutine 中运行，而 go 语句本身会迅速地完成。

```go
go func Hello()
```

4.在主函数返回时，所有的 goroutine 都会被直接打断，程序退出。除了从主函数退出或者直接终止程序之外，没有其它的**编程方法**能够让一个 goroutine 来打断另一个的执行，但是可以通过 channel 在不同的 goroutine 进行通信。

5.在 Go 中有一句话：**不要通过共享内存来通信，而应该通过通信来共享内存**，所以 channel 就是这句话的后者。

## 什么是 Channel

1.channel 是 goroutine 之间的通信机制。它可以让一个 goroutine 给另一个 goroutine 发送值信息。每个 channel 都有自己可以发送的数据类型，它的底层数据结构为引用类型，所以用 make 函数进行创建。

```go
ch := make(chan int)
```

2.一个 channel 有发送和接受两个主要操作，都是通信行为。一个发送语句将一个值从一个 goroutine 通过 channel 发送到另一个执行接收操作的 goroutine 。发送和接收两个操作都使用 `<-` 运算符。在发送语句中，`<-` 运算符分割 channel 和要发送的值。在接收语句中，`<-` 运算符写在 channel 对象之前。一个不使用接收结果的接收操作也是合法的。

```go
ch <- 1 // a send statement
x := <- ch // a receive expression
<- ch // a receive statement
```

3.channel 还支持 close 操作，用于关闭 channel ，关闭之后的 channel 进行发送和重复 close 操作都将导致 panic 异常。但是此时还可以进行接收操作，并且可以接受到之前已经发送成功的数据；如果 channel 中已经没有数据的话将产生一个零值数据。

```go
close(ch)
```

4.channel 还分为带缓存的 channel 和不带缓存的 channel ，不带缓存的 channel 会造成发送方所在的goroutine 阻塞直到有接收方 goroutine 在相同的 channel 上执行接收操作，才可以执行后面的语句。

```go
ch := make(chan int)
```

5.带缓存的 channel 内部持有一个元素队列。队列的最大容量是 make 函数创建 channel 时通过第二个参数指定的。创建三个字符串的带缓存的 channel:

```go
ch := make(chan string,  3)
```

![](https://go.wuhaolin.cn/gopl/images/ch8-02.png)

向缓存 channel 的发送操作就是向内部缓存队列的尾部插入元素，接收操作则是从队列的头部删除元素。如果内部缓存队列是满的，那么发送操作将阻塞直到因另一个 goroutine 执行接收操作而释放了新的队列空间。相反，如果 channel 是空的，接收操作将阻塞直到有另一个 goroutine 执行发送操作而向队列插入元素。

## Go Channel 踩坑

在 Go 中使用无缓存的 channel （同步channel）在不同的协程来进行通信时，下面这种写法会造成死锁：

```go
	no := make(chan struct{})
	no <- struct{}{} 
	go func() {
		x := <-no
		fmt.Println(x)
	}()
```

原因是无缓存的 channel 在发送（接收）数据时都会造成阻塞，直到其他协程从该 channel 中接收（发送）数据。所以上面的代码问题在于第二行发送数据一直会阻塞，从而后面开启协程的操作也执行不到，造成死锁。踩了这个坑才理解 《Go 圣经中文版》8.4.1节中下面这句话的含义：

> 基于无缓存Channels的发送和接收操作将导致两个goroutine做一次同步操作。因为这个原因，无缓存Channels有时候也被称为同步Channels。**当通过一个无缓存Channels发送数据时，接收者收到数据发生在唤醒发送者goroutine之前**（译注：*happens before*，这是Go语言并发内存模型的一个关键术语！）。

所以正确的代码应该是这样：

```go
	no := make(chan struct{})
	go func() {
		x := <-no
		fmt.Println(x)
	}()
	no <- struct{}{}
```



