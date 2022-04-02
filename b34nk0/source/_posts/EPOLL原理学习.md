---
title: EPOLL原理学习
date: 2022-03-31 17:45:16
categories: 网络 
tags: （Linux）Epoll
---


Epoll在ET（Edge Triggered）边缘触发模式下，Socket必须是非阻塞，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。
et模式，是当发生事件时进行一次通知，epool_wait获取该次事件后，如果用户不处理完缓冲区数据，使用epoll_wait是不会再获取到事件

LT（level Triggered）水平触发模式，支持非阻塞Socket及阻塞Socket，当一个文件句柄准备就绪时，内核会通知用户进行io操作，如果不进行任何操作时，内核还是会继续通知用户关于该句柄的事件。
通过epoll_wait获取到事件后，如果不处理完缓冲区数据，继续使用epoll_wait

epol会将用户空间与内核空间采用mmap技术映射为同一内存，减少内存拷贝

epoll相比于select/poll，在于epoll通过epoll_create创建时，内核开始准备存储句柄的空间，通过epoll_add添加句柄后，在epoll_wait开始等待事件通知，而select/poll每次都需要将句柄列表拷贝给内核。

当进程调用eopll_create()时，内核会创建一个eventpoll结构体，该结构体里采用红黑树来存储epoll_ctl添加进来的句柄，采用双向链表来记录就绪句柄。
当调用epoll_ctl的add操作添加一个句柄时，会把这个句柄存储到红黑树，同时注册一个回调函数给内核。回调函数的操作：当该句柄有中断时，内核调用回调将句柄放到链表

epoll_wait调用时，查看就绪列表有没数据即可，如果有则返回，如果没有则等待timeout后返回