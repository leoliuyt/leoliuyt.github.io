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

list和tuple是Python内置的有序集合，一个可变，一个不可变。

### Dictionary

### 函数

函数名其实就是指向一个函数对象的引用，完全可以把函数名赋给一个变量，相当于给这个函数起了一个“别名”

定义函数时，需要确定函数名和参数个数；

如果有必要，可以先对参数的数据类型做检查；

函数体内部可以用return随时返回函数结果；

函数执行完毕也没有return语句时，自动return None。

函数可以同时返回多个值，但其实就是一个tuple。

```python
a = abs
a(-10)#10
```

`pass`语句什么都不做，那有什么用？实际上pass可以用来作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个`pass`，让代码能运行起来。

```python
if age >= 18:
    pass
```

#### 位置参数


#### 默认参数

可以不按顺序提供部分默认参数。当不按顺序提供部分默认参数时，需要把参数名写上。比如调用enroll('Adam', 'M', city='Tianjin')，意思是，city参数用传进去的值，其他默认参数继续使用默认值。

```python
def enroll(name,gender,age=6,city='Beijing'):
    print("name:",name)
    print("gender:",gender)
    print("age:",age)
    print("city:",city)

enroll("Jack","男",city="Shanghai")
```

>注意：
>定义默认参数要牢记一点：默认参数必须指向不变对象！

#### 可变参数
可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

```python
#可变参数
def calc(*numbers):
    print(type(numbers))#<class 'tuple'>
    sum = 0
    for x in numbers:
        sum += x
    return sum;

sum = calc(1,2,3,4,5,6,7)
print("sum = ",sum)
```
#### 关键字参数

对于关键字参数，函数的调用者可以传入任意不受限制的关键字参数

```python

# 关键字参数
def person(name,age,**kw):
    print(type(kw))
    print('name:', name, 'age:', age, 'other:', kw)

person('leo','18',city='Beijing')

extra = {'city':'Beijing','job':'Engineer'}

person('leo','18',**extra)
```

#### 命名关键字参数

如果要限制关键字参数的名字，就可以用命名关键字参数

例如，只接收city和job作为关键字参数。这种方式定义的函数如下：

```python
def nameKeyword(name,age,*,city,job):
    print('name: ', name, 'age: ', age, 'city: ', city,'job: ',job)
nameKeyword('leo','20',city='Shanghai',job='Engineer')

extraDic = {'city':'Beijing','job':'Engineer'}
nameKeyword('leoliuyt',18,**extraDic)
```
命名关键字参数需要一个特殊分隔符`*`，`*`后面的参数被视为命名关键字参数。

命名关键字参数必须传入参数名，这和位置参数不同

如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了：

```python
#如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了：

def nameKWWithArgs(name,age,*args,city,job):
    print(type(args))
    print('name: ', name, 'age: ', age, 'city: ', city,'job: ',job)

extraArr = [1,2,3,4]
nameKWWithArgs('leoliu','18',*extraArr,city = 'Yantai',job = 'Engineer')
```

#### 参数组合

在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

对于任意函数，都可以通过类似func(*args, **kw)的形式调用它，无论它的参数是如何定义的

## 高级特性

### 切片

L[0:3]表示，从索引0开始取，直到索引3为止，但不包括索引3。即索引0，1，2

只写`[:]`就可以原样复制一个list

前10个数，每2个取一个：
```python
>>> L[:10:2]
[0, 2, 4, 6, 8]
所有数，每5个取一个：
```
```python
>>> L[::5]
[0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95]
```
### 迭代

只要是可迭代对象，无论有无下标，都可以迭代

```python
dic = {'name':'leoliu','age':18}

for key in dic:
    print(key)

for value in dic.values():
    print(value)

for key,value in dic.items():
    print("key:",key,"value:",value)

for x, y in [(1, 1), (2, 4), (3, 9)]:
    print(x, y)
```

## 列表生成式

```python
import os
ds = [d for d in os.listdir('.')]
print(ds)

l = [x * x for x in range(1,5) if x % 2 == 0]#[4, 16]
print(l)

l = [m + n for m in 'ABC' for n in 'XYZ']
print(l)

d = {'x':'A','y':'B','z':'C'}
kvs = [k + '=' + v for k,v in d.items()]
print(kvs)
```

## generator

如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：`generator`

要创建一个`generator`，有很多种方法。第一种方法很简单，只要把一个列表生成式的`[]`改成`()`，就创建了一个generator：

```python
g = (x*x for x in range(10))
nextValue = next(g)
for n in g:
    print(n)
```

斐波拉契数列用列表生成式写不出来，但是，用函数把它打印出来却很容易：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'
```

fib函数实际上是定义了斐波拉契数列的推算规则，可以从第一个元素开始，推算出后续任意的元素，这种逻辑其实非常类似generator。

也就是说，上面的函数和generator仅一步之遥。要把fib函数变成generator，只需要把print(b)改为`yield` b就可以了：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```

```
杨辉三角
          1
         / \
        1   1
       / \ / \
      1   2   1
     / \ / \ / \
    1   3   3   1
   / \ / \ / \ / \
  1   4   6   4   1
 / \ / \ / \ / \ / \
1   5   10  10  5   1
```

```python
def triangles():
    N = [1]
    while True:
        yield N
        N.append(0)
        N = [N[i-1] + N[i] for i in range(len(N))]

n = 0
for x in triangles():
    print(x)
    n += 1
    if n == 10:
        break
```

## Iterator

可以被`next()`函数调用并不断返回下一个值的对象称为迭代器：`Iterator`

生成器都是`Iterator`对象，但`list`、`dict`、`str`虽然是`Iterable`，却不是`Iterator`。

把`list`、`dict`、`str`等`Iterable`变成`Iterator`可以使用`iter()`函数：

凡是可作用于for循环的对象都是Iterable类型；

凡是可作用于`next()`函数的对象都是`Iterator`类型，它们表示一个惰性计算的序列；

集合数据类型如`list`、`dict`、`str`等是`Iterable`但不是`Iterator`，不过可以通过`iter()`函数获得一个`Iterator`对象。

Python的for循环本质上就是通过不断调用`next()`函数实现的。

```python
from collections import Iterable,Iterator
isinstance([],Iterable)#True
isinstance({},Iterable)#True
isinstance('abc',Iterable)#True
isinstance((x for x in range(10)),Iterable)#True

isinstance((x for x in range(10)),Iterator)#True
isinstance([],Iterator)#False
isinstance({},Iterator)#False
isinstance('abc',Iterator)#False

isinstance(iter([]),Iterator)#True
isinstance(iter('abc'),Iterator)#True
```