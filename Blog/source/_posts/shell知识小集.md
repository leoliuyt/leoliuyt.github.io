---
title: shell知识小集
date: 2018-04-09 17:20:26
tags: shell
categories: shell
---

## 四种纯数字字符串转数字的方式

- $[]
- $(())
- `expr`
- let

实例:

```shell
#!/bin/sh
string="   12  "
number=$[string]
# number=$((string))
# number=`expr $string`
# let number="string"
echo -e "number=$number\n"
echo -e "string=$string\n"
```
结果：

```shell
-e number=12

-e string=   12
```