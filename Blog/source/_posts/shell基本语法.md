---
title: shell基本语法
date: 2018-04-08 15:19:52
tags: shell
categories: shell
---
## shell 简介 
Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。
Ken Thompson 的 sh 是第一种 Unix Shell，Windows Explorer 是一个典型的图形界面 Shell。

### shell环境
Shell 编程跟 java、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。
Linux 的 Shell 种类众多，常见的有：
- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）

### shell与其他脚本语言

理论上讲，只要一门语言提供了解释器（而不仅是编译器），这门语言就可以胜任脚本编程，常见的解释型语言都是可以用作脚本编程的，如：`Perl`、`Tcl`、`Python`、`PHP`、`Ruby`。`Perl`是最老牌的脚本编程语言了，`Python`这些年也成了一些linux发行版的预置解释器。

编译型语言，只要有解释器，也可以用作脚本编程，如`C shell`是内置的（`/bin/csh`），`Java`有第三方解释器`Jshell`，`Ada`有收费的解释器`AdaScript`。

如下是一个`PHP Shell Script`示例（假设文件名叫`test.php`）：
```php
#!/usr/bin/php
<?php
for ($i=0; $i < 10; $i++)
        echo $i . "\n";
```
执行：
```php
/usr/bin/php test.php
```
或者：
```php
chmod +x test.php
./test.php
```

shell脚本与其他脚本语言的优势在于环境兼容性强，劣势在于不适合复杂度较高，要操作的数据结构比较复杂的操作

### 第一个shell脚本

```shell
#!/bin/bash
echo "my first shell"
```
`#!`是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行。
第一行可以是`#!/bin/bash`、`#!/bin/sh`、`#!/usr/bin/php`等形式

#### 运行
shell 运行有两种方法：
1.作为可执行程序

```shell
chmod +x demo1.sh
./demo1.sh
```
2.作为解释器参数
```
/bin/sh demo1.sh
/usr/bin/php test.php
```
## shell基本语法

### 变量
```shell
your_name="leoliu"
```
>注意:
>变量定义时`=`两边不能有空格

除了显式地直接赋值，还可以用语句给变量赋值，如
```shell
# for file in `ls /etc`
for file in $(ls /etc)
do
echo $file
done
```

#### 使用变量

```shell
your_name="qinjx"
echo $your_name
echo ${your_name}
```
使用变量的时候要在变量前加`$`,变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界比如下面这种情况：
```shell
for skill in Ada Coffe Action Java; 
do
    echo "I am good at ${skill}Script"
done
```

#### 只读变量
使用`readonly`命令可以将变量定义为只读变量，只读变量的值不能被改变。

```shell
myUrl="http://www.w3cschool.cc"
readonly myUrl
myUrl="http://www.baidu.com"
//报错
myUrl: readonly variable
```

#### 删除变量
使用`unset`命令可以删除变量。语法：
```shell
unset your_name
```
变量被删除后不能再次使用。`unset` 命令不能删除只读变量。


#### 变量类型
运行shell时，会同时存在三种变量：
1) 局部变量 局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
2) 环境变量 所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
3) shell变量 shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

### 字符串
字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。

#### 单引号
单引号字符串的限制：
- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

#### 双引号
双引号的优点：
- 双引号里可以有变量
- 双引号里可以出现转义字符

#### 拼接字符串

```shell
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting $greeting_1
```

#### 获取字符串长度
```shell
string="abcd"
echo ${#string}
```
#### 提取子字符串
以下实例从字符串第 2 个字符开始截取 4 个字符：
```shell
string="abcdefg"
echo ${string:1:4} #bcde
```
#### 查找子字符串
查找字符 "i 或 s" 的位置：
```shell
string="runoob is a great company"
echo `expr index "$string" is`  # 输出 8(Mac OS 上没有index这个命令)
```
### 数组
bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

#### 定义数组
在Shell中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：
```shell
数组名=(value1 value2 value3)
```

#### 读取数组
读取数组元素值的一般格式是：
```shell
${数组名[下标]}
```

使用`@`符号可以获取数组中的所有元素
```shell
echo ${array_name[@]}
```

#### 获取数组的长度
获取数组长度的方法与获取字符串长度的方法相同，例如：
```shell
#取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
#取得数组单个元素的长度
length=${#array_name[n]}
```

### 传递参数
我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：`$n`。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……之中`$0` 固定为执行的文件名

参数处理  | 说明   
---|---
$#  | 传递到脚本的参数个数
$*  | 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$$	|脚本运行的当前进程ID号
$!	|后台运行的最后一个进程的ID号
$@	|与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
$-	|显示Shell使用的当前选项，与set命令功能相同。
$?	|显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。