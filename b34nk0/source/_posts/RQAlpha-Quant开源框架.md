---
title: RQAlpha Quant开源框架
date: 2022-01-19 11:25:09
categories: 量化技术
tags: RQAlpha
---

# 概要
RQAlpha虽然是开源的，但得益于米筐的公司化运做，其因为稳定以及产品化而在市面上受广大量化开发者们的青睐，本文也主要学习RQAlpha的框架以及代码结构，以便后续能对接数字货币进行回测分析以及实盘交易等。

<!--more-->  

# RQAlpha代码框架图
![](代码框架图.png)

# 回测系统结构图
![](回测系统结构图.png)


RQAlpha为python框架，其__init__.py提供了多种策略运行方式

```python

# 加载ipython用于执行ipython cell
def load_ipython_extension(ipython):

# 加载配置进行运行
def run(config, source_code=None):

# 运行ipython cell
def run_ipython_cell(line, cell=None):

# 运行策略文件
def run_file(strategy_file_path, config=None):

# 运行策略代码
def run_code(code, config=None):

# 运行函数
def run_func(**kwargs):
```

## 策略运行方式
![](策略运行方式.png)

执行策略时运行main入口的run函数
开始进行Environment环境初始化、mod模块加载已经链接rqdata服务器等操作

![](回测流程图.png)

## 回测事件驱动模型

![](事件驱动模型.png)
以上为模拟日线回测时，一个交易日所产生的事件