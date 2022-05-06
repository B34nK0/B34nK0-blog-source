---
title: GDB调试
date: 2022-04-24 14:09:52
categories: 开发问题
tags: C++
---

# 概要
GDB作为Linux下的C++调试利器，是每位开发者必备的。
GDB支持断点、单步执行、打印变量、观察变量、查看寄存器、查看堆栈等调试手段。
不仅可调试C/C++，也支持Go等其他语言，采用vscode remote开发模式的话，可以在windows下开发C/C++程序的同时也可以在windows下使用GDB进行调试（环境安装了GDB server）。

# GDB如何断点
break 命令（可以用 b 代替）常用的语法格式有以下 2 种。
```shell
(gdb) break location     // b location
(gdb) break ... if cond   // b .. if cond
```

# 栈帧被破坏后如何调试

```shell

gdb xxx.out/xxx.exe  core

```
需要原程序及coredump

1、通过`bt`可以看到栈帧被破坏后，函数名都显示为 ??()
2、通过`info reg` 在gdb命令查看当前的寄存器情况
3、获取当前栈底寄存器rbp的值
4、通过`x /128ag $rbp` 跳过被破坏的栈区域，往前查找128个内存单元，以地址格式输出
5、通过`addr2line`获取地址对应函数


# 什么情况会导致C++程序崩溃，如何确认问题