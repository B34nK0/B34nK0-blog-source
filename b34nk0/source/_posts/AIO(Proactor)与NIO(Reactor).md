---
title: AIO(Proactor)与NIO(Reactor)
date: 2022-04-01 10:13:11
categories: 网络 
tags: IO模型
---

# 概要
在讲到IO时，我们会讲到阻塞/非阻塞，即BIO/NIO。那么AIO又是什么呢？

AIO即是Asynchronous IO，异步IO。这里的异步是关于用户读取数据来说是同步还是异步，回到BIO及NIO的场景：
BIO是创建一个IO句柄后，阻塞等待数据返回
NIO在创建完一个IO句柄后，不阻塞等待，而是去做其他事情，当有数据时，用户再进行数据读取，这个时候的读取动作是同步的，即如果缓冲区的数据没读取完，那么会一直读取。

比如select以及poll模型，这两种模型都是非阻塞IO，即当socket套接字创建完成之后，用户可以去做其他事情，当套接字有事件发生时，用户主动将内核数据同步拷贝到用户态。

AIO要优化的的即是，当内核数据有数据时，将内核态数据拷贝到用户态，并不需要用户做主动拷贝。

## 总结：
AIO将IO就绪事件监听和IO数据读取全部交由操作系统完成，也就不需要多路复用器去监听IO就绪事件、也不需要分配线程读取。
用户完全不需要关注IO过程，只需要关注拿到数据后的处理。

<!--more-->

# AIO的底层实现

Windows上是使用完成接口`IOCP`实现。

Linux上使用aio调用`UnixAsynchronousServerSocketChannelImpl, UnixAsynchronousSocketChannelImpl, SolarisAsynchronousChannelProvider`。

* Linux内核的AIO实现有很多问题（不在本文讨论范畴），性能在某些场景下还不如NIO，连Linux上的Java都是用epoll来模拟AIO，所以Linux上使用Java的AIO API，只是能体验到异步IO的编程风格，但并不会比NIO高效。

由于Linux上实现的缺陷，Linux平台上的Java服务端编程，目前主流依然采用NIO模型。

# Windows AIO 与 Linux NIO

关于AIO与NIO的同步异步方式，由此引生出两种通讯模型：Proactor、Reactor.

Proactor与Reactor都是基于IO多路复用的事件驱动模式，区别在于数据的同步与异步读取。
在Proactor模式下，当回调handler时，表示IO已经完成。
在Reactor模式下，当回调handler时，表示IO可以进行某个操作。

IO多路复用依赖一个事件多路分离器(Event Demultiplexer)，开发人员预先注册需要处理的事件以及事件处理器（或回调函数），事件分离器负责将来自事件源的IO事件分离出来，再分发到对应的事件处理器。

## Reactor
采用主线程（IO处理单元）只监听IO描述符是否有事件发生，有的话就通知工作线程（逻辑单元），读写数据、接收新的链接、以及处理用户请求均在工作线程处理。

### 单Reactor（主线程）、单线程（工作线程）
acceptor分发事件到handler以及handler处理数据都是在同一线程

### 单Reactor（主线程）、多线程（工作线程）
1、如果是建立链接事件，则由acceptor通过accept处理链接请求，然后创建一个handler来处理这个链接接下来的其他事件
2、如果是其他事件，则分发给对应链接的handler
3、handler只负责响应事件，不做具体的任何业务，通过read读取数据后，会分发给后面的work工作线程池进行业务处理
4、业务出来完成之后分发会handler，handler将再数据发往对应的链接

### 主从Reactor，多线程（工作线程）模式
1、主Reactor线程池中随机选择一个线程作为acceptor线程，用于绑定监听端口，接收客户端链接
2、acceptor接收客户端链接后创建新的socketchannel，同时从主Reactor线程中获取一个线程来进行客户端认证、IP黑白名单等控制层处理
3、验证通过的socketchannel在从Reactor线程池中创建一个handler来处理该链接
4、当链接有事件触发时，从Reactor会进行分发到对应的handler
5、handler响应事件时，通过read读取数据后，分发到work工作先吃进行业务处理
6、业务出来完成之后分发会handler，handler将再数据发往对应的链接

## Proactor

1、Proactor initiator负责创建 Proactor和Handler，并将Proactor和Handler都通过Asynchronous Operation Processer注册到内核
2、当Asynchronous Operation Processer完成IO操作后会通知Proctor
3、Proactor根据不同的事件分发到不同的Handler进行业务处理
4、Handler完成业务处理后，可以注册新的Handler到内核

读操作：
1、应用程序初始化一个异步读取事件，然后注册事件处理器，区别于Reactor，Proactor是异步IO，事件处理器不关注读取就绪时间，只关注读取完成事件。
2、事件分离器等待读取完成事件
3、操作系统调用内核完成读取操作，将读取到的数据放到用户传递过来的缓冲区，之后回调事件分离器
4、事件分离器捕获到读取完事件后，回调给用户注册的事件处理器，事件处理器直接从缓冲区读取数据，而不需要再进行实际的IO操作

相比于Reactor，Proactor是内核将数据读取完成之后放到用户提供的缓冲区，而Reactor需要用户多次进行读取操作