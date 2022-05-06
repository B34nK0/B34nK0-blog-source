---
title: 跟踪线上java进程cpu异常
date: 2022-04-29 09:40:43
categories: 开发问题
tags: Java
---

# 概要
线上有个服务，当大量用户在请求任务时（会将任务异步持久化进行记录），服务有占用大量的cpu，导致服务不可用。

运行环境centos7，jdk1.8

<!--more-->

# 查看系统状态
使用 top命令查看cpu、内存占用情况


# 定位问题线程
使用ps -mp pid -o THREAD,tid,time命令查看该进程的线程情况，发现该进程的两个线程占用率很高

`ps -mp 15801 -o THREAD,tid,time`

# 查看问题线程堆栈

## 将线程id转换为16进制
`printf "%x\n" 15801`

## jstack查看线程堆栈信息
jstack命令打印线程堆栈信息，命令格式：jstack pid |grep tid

`jstack 15801 |grep 3db9`

## jstat查看进程内存状况
命令: jstat -gcutil

## 使用jmap保存堆栈及内存为heapdump文件

命令: jmap [option] vmid
jmap -dump:format=b,file=dump.bin 6764

## 使用jdk自带JVisualVM 分心heapdump
通过cmd控制台输入JVisualVM打开

JVisualVM 工具栏打开插件安装Visual GC

装入heapdump文件进行分析

查看类占用内存大小情况，找出异常的类或实例