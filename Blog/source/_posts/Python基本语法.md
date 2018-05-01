---
title: Python3基本语法
date: 2018-04-28 16:09:42
tags: Python
categories: Python
---

## Python 工具

### Pylint

`Pylint` 是一个 `Python` 代码分析工具，它分析 `Python` 代码中的错误，查找不符合代码风格标准（`Pylint` 默认使用的代码风格是 `PEP 8`，具体信息，请参阅参考资料）和有潜在问题的代码。

## pip

`pip` 是一个现代的，通用的 `Python` 包管理工具。提供了对 `Python` 包的查找、下载、安装、卸载的功能。

## Python 基本数据类型

Python3中有六个标准的数据类型：

- Number（数字）
    - int
    - float
    - bool
    - complex
- String (字符串)
- List (列表)
- Tuple（元组）
- Sets（集合）
- Dictionary（字典）

其中不可变数据类型有四个：Number、String、Tuple、Sets
不可变数据（两个）：List、Dictionary

### Number

`//` 向下取整

```python
intDivide = 1 // 2
print(intDivide) 
// 0
```

`**` 操作来进行幂运算
 

```python
powerInt = 5 ** 2
print(powerInt)
// 25
```

在交互模式中，最后被输出的表达式结果被赋值给变量 `_` 

```python
>>> tax = 12.5 / 100
>>> price = 100.50
>>> price * tax
12.5625
>>> price + _
113.0625
```

数字相关函数

```python
# 函数
print("绝对值：abs(-10) = %s" % abs(-10))#10
print("绝对值：fabs(-10) = %s" % math.fabs(-10))#10.0
print("向上取整：ceil(4.1) = %s" % math.ceil(4.1))#5
print("向下取整：ceil(4.9) = %s" % math.floor(4.9))#4
print("返回浮点数x的四舍五入值 round(4.9164,2) = %s" % round(4.9164,2))#4.92
print("最大值：max(10,11,9) = %s" % max(10,11,9))#11
print("最小值：min(10,11,9) = %s" % min(10,11,9))#9
print("返回x的整数部分与小数部分modf(9.12) = " , math.modf(9.12))#(0.11999999999999922, 9.0)
print("x**y 运算后的值pow(2,3) = %s" % pow(2,3))#8
print("e的x次幂：exp(1) = %s" % math.exp(1))#2.718281828459045
print("x的平方根：sqrt(x) = %s" % math.sqrt(9))#3.0
print("log(x) = %s" % math.log(100,10))#2.0
print("log10(x) = %s" % math.log10(100))#2.0
```

### String

### List

### Tuple

### Dictionary
