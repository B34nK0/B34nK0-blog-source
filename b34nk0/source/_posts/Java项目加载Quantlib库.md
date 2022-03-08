---
title: Java项目加载Quantlib库
date: 2022-02-14 18:50:05
categories: 量化技术
tags: Quantlib
---

# 概要

QuantLib(https://github.com/lballabio/QuantLib) 是一个免费、开源的软件库，旨在为量化金融计算提供一个统一的、综合的软件框架。QuantLib 的源代码由 C++ 编写，得利于 C++ 在面向对象和泛型编程方面强大的表现力，以及C++对贴近底层所带来的出众执行效率，QuantLib 一方面可以清晰地描述各种复杂的金融产品，同时兼顾了计算速度。

主要功能
QuantLib 所提供的功能聚焦在两大领域：

期权定价以及相关计算；
固定收益产品定价以及相关计算。

<!--more-->  

因为Quantlib是c++编写的，而为了面向多语言场景，作者采用SWIG方式进行了封装，提供给Java、Python、CSharp等语言使用

本章主要讲解如何配置IDEA，让Java项目加载Quantlib提供的JNI包, JNI包下载（http://www.bnikolic.co.uk/ql/java.html）

选择对应的环境及版本进行下载后：
![](JNI包.png)

* 打开提供的Example项目
![](Example项目.png)

* IDEA导入JNI及Dll：  File -> Project Struct -> Libraries -> Java
![](选择ProjectStruct.png)
![](导入jar.png)

需要导入Jar 及 dll

* 运行Example提供的demo

总结：
生产时我们构建Java 分布式应用来提供Saas服务，底层采用的即是Quantlib库，金融数据有着量大需要实时计算的特点，所以需要性能优势明显的c++进行计算。