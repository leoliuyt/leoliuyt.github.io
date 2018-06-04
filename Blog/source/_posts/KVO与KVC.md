---
title: KVO与KVC
date: 2018-06-04 11:05:53
tags: runtime
categories: Objective-C
---

## KVO
KVO（key-value-observing）
- 系统对观察者设计模式的一种实现
- 通过运行时的`ISA-Swizzling`技术实现

{% asset_img kvo.png kvo流程 %}

## KVC

KVC(key-value-coding)

`valueForKey`执行流程

{% asset_img kvc_valueforkey.png valueForKey调用流程 %}

访问器方法是否存在的判断标准（假设valueForKey中传入的key为@"name"）

- getName
- name
- isName

如果别观察对象中实现了上面三个方法，那么`valueForKey:@"name"`方法时就会调用到上面的三个方法，如果三方法都存在只调用其中的一个方法，优先级就是上面的顺序。

实例变量是否存在的判断标准是

类中定义字段名 | valueForKey中传入的key的名称
 --|--
name | name
_name | name 或 _name
isName | name 或 isName
_isName | name 或 _isName 或 isName


`setValueForKey`执行流程

{% asset_img kvc_setvalueforkey.png setValueForKey流程 %}

setter方法的判断标准是

是否实现了`setName`或`setIsName`

验证的[demo](https://github.com/leoliuyt/DMKVO)从这里下载。