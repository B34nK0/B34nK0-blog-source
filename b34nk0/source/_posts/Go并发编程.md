---
title: Go并发编程
date: 2022-04-12 10:01:26
categories: 编程语言
tags: Golang
---

# 概要
从线程模型到协程模型，讲解Go的并发编程

# 内核线程
我们知道线程是内核的最小调度单位，应用程序对线程的创建、终止和同步都必须通过内核提供的系统调用来完成，内核可以分别对每一个线程进行调度。
又因为线程的创建、切换和同步都需要花费更多的内核资源和时间，因此基于这种模型会经常采用`线程池`的方式来进行优化。

# 用户级线程（协程）
尽管是线程池，也只是优化了创建调度的性能，因此产生了协程的开发概念，协程更多是在用户级维护`调度单位`(线程)。协程库基本是语言层面提供的，一个进程内的多个`线程`（协程）对应的是一个内核级线程，而上下文的切换是在用户态进行的。相较于内核级线程的内存占用（MB）来说其内存占用（KB）更小，且不需要cpu在用户态和内核态之间切换。

此模型下的多线程并不能真正的并发运行。例如，如果某个线程在 I/O 操作过程中被阻塞，那么其所属进程内的所有线程都被阻塞，整个进程将被挂起。

# 两级线程模型
集合内核级线程以及用户级线程模型，让一个进程内的`线程`（用户级线程）可以绑定到不同的内核级线程，如果用户级线程某个操作阻塞后，绑定的内核线程需要让出CPU时，用户级其他线程绑定的内核线程可以得到cpu获得执行，即整个用户级可以实现并发。

# Go的并发机制
传统的协程即用户级线程模型，而Go的协程模型区别于传统的协程，其成为Goroutine，其实现的是两级线程模型，依赖的是Go实现的调度器。

Go 的调度器使用 G、M、P 三个结构体来实现 Goroutine 的调度，也称之为GMP 模型。

## G
表示 Goroutine。
每个 Goroutine 对应一个 G 结构体，G 存储 Goroutine 的运行堆栈、状态以及任务函数，可重用。
当 Goroutine 被调离 CPU 时，调度器代码负责把 CPU 寄存器的值保存在 G 对象的成员变量之中。
当 Goroutine 被调度起来运行时，调度器代码又负责把 G 对象的成员变量所保存的寄存器的值恢复到 CPU 的寄存器。

## P
表示逻辑处理器。
对 G 来说，P 相当于 CPU 核，G 只有绑定到 P(在 P 的 local runq 中)才能被调度。
对 M 来说，P 提供了相关的执行环境(Context)，如内存分配状态(mcache)，任务队列(G)等。
它维护一个局部 Goroutine 可运行 G 队列，工作线程优先使用自己的局部运行队列，只有必要时才会去访问全局运行队列，这可以大大减少锁冲突，提高工作线程的并发性，并且可以良好的运用程序的局部性原理

## M
OS 底层线程的抽象，它本身就与一个内核线程进行绑定，每个工作线程都有唯一的一个 M 结构体的实例对象与之对应，它代表着真正执行计算的资源，由操作系统的调度器调度和管理。
M 结构体对象除了记录着工作线程的诸如栈的起止位置、当前正在执行的 Goroutine 以及是否空闲等等状态信息之外，还通过指针维持着与 P 结构体的实例对象之间的绑定关系

![](GPM任务调度模型.jpg)


可以总结为G即是我们的用户任务，我们需要依赖于P和M来执行我们的任务。


### G
g 结构体部分源码（src/runtime/runtime2.go）：
```Golang
type g struct {
    stack    stack  // Goroutine的栈内存范围[stack.lo, stack.hi)
    stackguard0   uintptr // 用于调度器抢占式调度
    m     *m  // Goroutine占用的线程
    sched    gobuf  // Goroutine的调度相关数据
    atomicstatus  uint32 // Goroutine的状态
    ...
}

type gobuf struct {
    sp  uintptr  // 栈指针
    pc  uintptr  // 程序计数器
    g  guintptr  // gobuf对应的Goroutine
    ret  sys.Uintewg // 系统调用的返回值
    ...
}
```

g结构中的atomicstatus表示当前g的执行状态，状态描述表：
![](G任务状态.jpg)

其可归纳为三种：`等待中，可运行，运行中`。
* 等待中：Goroutine 正在等待某些条件满足，例如：系统调用结束等，包括_Gwaiting、_Gsyscall 和_Gpreempted 几个状态

* 可运行：Goroutine 已经准备就绪，可以在线程运行，如果当前程序中有非常多的 Goroutine，每个 Goroutine 就可能会等待更多的时间，即_Grunnable

* 运行中：Goroutine 正在某个线程上运行，即_Grunning

状态转换图：
![](G任务的状态转换.jpg)

### M
前文讲到M实际上是系统线程，在大多数情况下，我们都会使用 Go 的默认设置，也就是线程数等于 CPU 数。
默认的设置不会频繁触发操作系统的线程调度和上下文切换，所有的调度都会发生在用户态，由 Go 语言调度器触发，能够减少很多额外开销。

m 结构部分源码：
```Golang
type m struct {
    g0   *g   // 一个特殊的goroutine，执行一些运行时任务
    gsignal  *g   // 处理signal的G
    curg  *g   // 当前M正在运行的G的指针
    p   puintptr // 正在与当前M关联的P
    nextp  puintptr // 与当前M潜在关联的P
    oldp  puintptr // 执行系统调用之前使用线程的P
    spinning bool  // 当前M是否正在寻找可运行的G
    lockedg  *g   // 与当前M锁定的G
}
```

`g0`区别于正在当前m线程上执行的g任务`curg`,其由 Go 运行时系统在启动之处创建，它会深度参与运行时的调度过程，包括 Goroutine 的创建、大内存分配和 CGO 函数的执行。
`puintptr`即当前与m绑定的p（g任务组）

### P

通过处理器 P 的调度，每一个M都能够执行多个 Goroutine，它能在 Goroutine 进行一些 I/O 操作时及时让出计算资源，提高线程的利用率。

p 结构体部分源码：
```Golang
type p struct {
 // p 的状态
 status   uint32
 // 对应关联的 M
 m        muintptr
 // 可运行的Goroutine队列，可无锁访问
 runqhead uint32
 runqtail uint32
 runq     [256]guintptr
 // 缓存可立即执行的G
 runnext   guintptr
 // 可用的G列表，G状态等于Gdead
 gFree struct {
  gList
  n int32
 }
 ...
}
```

p结构的status状态描述：
![](P状态.jpg)
