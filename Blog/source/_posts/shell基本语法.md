---
title: shell基本语法
date: 2018-04-08 15:19:52
tags: shell
categories: shell
---

## shell简介

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

```shell
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
$#| 传递到脚本的参数个数
$*| 以一个单字符串显示所有向脚本传递的参数。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。
$$|脚本运行的当前进程ID号
$!|后台运行的最后一个进程的ID号
$@|与$*相同，但是使用时加引号，并在引号中返回每个参数。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。
$-|显示Shell使用的当前选项，与set命令功能相同。
$?|可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行后的返回结果。0表示没有错误，其他任何值表明有错误。

`$*` 与 `$@` 区别：

相同点：都是引用所有参数。
不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

```shell
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

输出结果：

```shell
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3
```

### 运算符

Shell 和其他编程语言一样，支持多种运算符，包括：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

```shell
#!/bin/bash
val=`expr 2 + 2`
echo "两数之和：$val"
```

>注意：
>表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
>完整的表达式要被 `` 包含，注意这个字符不是常用的单引号。

#### 算数运算符

运算符|说明|举例
---|---|---
+|加法|\`expr $a + $b\` 结果为 30。
-|减法|\`expr $a - $b\` 结果为 -10。
*|乘法|\`expr $a \* $b\` 结果为  200。
/|除法|\`expr $b / $a\` 结果为 2。
%|取余|\`expr $b % $a\` 结果为 0。
=|赋值|a=$b 将把变量 b 的值赋给 a。
==|相等。用于比较两个数字，相同则返回 true。|[ $a == $b ] 返回 false。
!=|不相等。用于比较两个数字，不相同则返回 true。|[ $a != $b ] 返回 true。
>注意：条件表达式要放在方括号之间，并且要有空格，例如: [$a==$b] 是错误的，必须写成 [ $a == $b ]。

实例：

```shell
a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"

if [ $a == $b ]
then
echo "a == b"
fi

if [ $a != $b ]
then 
echo "a != b"
fi
```

结果：

```shell
a + b : 30
a - b : -10
a * b : 200
b / a : 2
b % a : 0
a != b
```

>注意：
>乘号(*)前边必须加反斜杠(\)才能实现乘法运算；
>if...then...fi 是条件语句，后续将会讲解。

#### 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。
下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

运算符|说明|举例
---|---|---
-eq|检测两个数是否相等，相等返回 true。|[ $a -eq $b ] 返回 false。
-ne|检测两个数是否相等，不相等返回 true。|[ $a -ne $b ] 返回 true。
-gt|检测左边的数是否大于右边的，如果是，则返回 true。|[ $a -gt $b ] 返回 false。
-lt|检测左边的数是否小于右边的，如果是，则返回 true。|[ $a -lt $b ] 返回 true。
-ge|检测左边的数是否大于等于右边的，如果是，则返回 true。|[ $a -ge $b ] 返回 false。
-le|检测左边的数是否小于等于右边的，如果是，则返回 true。|[ $a -le $b ] 返回 true。

实例：

```shell
if [ $a -eq $b ]
then
echo "$a -eq $b : a 等于 b"
else 
echo "$a -eq $b : a 不等于b"
fi

if [ $a -ne $b ]
then
   echo "$a -ne $b: a 不等于 b"
else
   echo "$a -ne $b : a 等于 b"
fi

if [ $a -gt $b ]
then
   echo "$a -gt $b: a 大于 b"
else
   echo "$a -gt $b: a 不大于 b"
fi

if [ $a -lt $b ]
then
   echo "$a -lt $b: a 小于 b"
else
   echo "$a -lt $b: a 不小于 b"
fi

if [ $a -ge $b ]
then
   echo "$a -ge $b: a 大于或等于 b"
else
   echo "$a -ge $b: a 小于 b"
fi

if [ $a -le $b ]
then
   echo "$a -le $b: a 小于或等于 b"
else
   echo "$a -le $b: a 大于 b"
fi
```

结果：

```shell
10 -eq 20 : a 不等于b
10 -ne 20: a 不等于 b
10 -gt 20: a 不大于 b
10 -lt 20: a 小于 b
10 -ge 20: a 小于 b
10 -le 20: a 小于或等于 b
```

#### 布尔运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

运算符|说明|举例
---|---|---
!|非运算，表达式为 true 则返回 false，否则返回 true。|[ ! false ] 返回 true。
-o|或运算，有一个表达式为 true 则返回 true。|[ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a|与运算，两个表达式都为 true 才返回 true。|[ $a -lt 20 -a $b -gt 100 ] 返回 false。

实例：

```shell
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ $a -lt 100 -a $b -gt 15 ]
then
   echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
   echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi
if [ $a -lt 100 -o $b -gt 100 ]
then
   echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi
if [ $a -lt 5 -o $b -gt 100 ]
then
   echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
   echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi
```

结果：

```shell
10 != 20 : a 不等于 b
10 小于 100 且 20 大于 15 : 返回 true
10 小于 100 或 20 大于 100 : 返回 true
10 小于 5 或 20 大于 100 : 返回 false
```

#### 逻辑运算符

以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:

运算符|说明|举例
---|---|---
`&&` | 逻辑的 AND | `[[ $a -lt 100 && $b -gt 100 ]] `返回 false
两竖杠 | 逻辑的 OR | `[[ $a -lt 100 两竖杠 $b -gt 100 ]]` 返回 true

实例：

```shell
if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

结果：

```shell
返回 false
返回 true
```

#### 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

运算符|说明|举例
---|---|---
=|检测两个字符串是否相等，相等返回 true。|[ $a = $b ] 返回 false。
!=|检测两个字符串是否相等，不相等返回 true。|[ $a != $b ] 返回 true。
-z|检测字符串长度是否为0，为0返回 true。|[ -z $a ] 返回 false。
-n|检测字符串长度是否为0，不为0返回 true。|[ -n $a ] 返回 true。
str|检测字符串是否为空，不为空返回 true。|[ $a ] 返回 true。

实例：

```shell
a="abc"
b="efg"

if [ $a = $b ]
then
   echo "$a = $b : a 等于 b"
else
   echo "$a = $b: a 不等于 b"
fi
if [ $a != $b ]
then
   echo "$a != $b : a 不等于 b"
else
   echo "$a != $b: a 等于 b"
fi
if [ -z $a ]
then
   echo "-z $a : 字符串长度为 0"
else
   echo "-z $a : 字符串长度不为 0"
fi
if [ -n $a ]
then
   echo "-n $a : 字符串长度不为 0"
else
   echo "-n $a : 字符串长度为 0"
fi
if [ $a ]
then
   echo "$a : 字符串不为空"
else
   echo "$a : 字符串为空"
fi
```

结果：

```shell
abc = efg: a 不等于 b
abc != efg : a 不等于 b
-z abc : 字符串长度不为 0
-n abc : 字符串长度不为 0
abc : 字符串不为空
```

#### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。
属性检测描述如下：

操作符|说明|举例
---|---|---
-b file|检测文件是否是块设备文件，如果是，则返回 true。|[ -b $file ] 返回 false。
-c file|检测文件是否是字符设备文件，如果是，则返回 true。|[ -c $file ] 返回 false。
-d file|检测文件是否是目录，如果是，则返回 true。|[ -d $file ] 返回 false。
-f file|检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。|[ -f $file ] 返回 true。
-g file|检测文件是否设置了 SGID 位，如果是，则返回 true。|[ -g $file ] 返回 false。
-k file|检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。|[ -k $file ] 返回 false。
-p file|检测文件是否是有名管道，如果是，则返回 true。|[ -p $file ] 返回 false。
-u file|检测文件是否设置了 SUID 位，如果是，则返回 true。|[ -u $file ] 返回 false。
-r file|检测文件是否可读，如果是，则返回 true。|[ -r $file ] 返回 true。
-w file|检测文件是否可写，如果是，则返回 true。|[ -w $file ] 返回 true。
-x file|检测文件是否可执行，如果是，则返回 true。|[ -x $file ] 返回 true。
-s file|检测文件是否为空（文件大小是否大于0），不为空返回 true。|[ -s $file ] 返回 true。
-e file|检测文件（包括目录）是否存在，如果是，则返回 true。|[ -e $file ] 返回 true。

实例：

```shell
file="/Users/lbq/Documents/Code/LL_Workspace/DMShell/param.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

结果：

```shell
文件可读
文件可写
文件可执行
文件为普通文件
文件不是个目录
文件不为空
文件存在
```

### 流程控制

和Java、PHP等语言不一样，sh的流程控制不可为空,如果else分支没有语句执行，就不要写这个else。
#### if else

if 语句语法格式：

```shell
if condition
then
    command1 
    command2
    ...
    commandN 
fi
```

写成一行就是：

```shell
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
```

if else 语法格式：

```shell
if condition
then 
command1
command2
else
command3
fi
```

if else-if else 语法格式：

```shell
if condition1
then
command1
elif condition2
then
command2
else
command3
fi
```

#### for循环

for循环一般格式为：

```shell
for var in item1  item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```

#### while语句

while循环用于不断执行一系列命令，也用于从输入文件中读取数据；命令通常为测试条件。其格式为：

```shell
while condition
do
    command
done
```

实例：

```shell

int=1
while(( $int <= 5 ))
do
echo $int
let "int++"
done
```
使用中使用了 Bash let 命令，它用于执行一个或多个表达式，变量计算中不需要加上 $ 来表示变量，具体可查阅：Bash let 命令

>注意:
>while条件要用两个括号，并且与内容之间有空格

while循环可用于读取键盘信息。下面的例子中，输入信息被设置为变量FILM，按<Ctrl-D>结束循环。

```shell
echo '按下 <CTRL-D> 退出'
echo -n '输入你最喜欢的网站名: '
while read input
do
    echo "是的！$input 是一个好网站"
done
```

#### 无限循环

无限循环语法格式：

```shell
while :
do
    command1
done
```

或

```shell
while true
do
    command1
done
```

或

```shell
for (( ; ; ))
```

#### until 循环

until 循环执行一系列命令直至条件为 true 时停止。
until 循环与 while 循环在处理方式上刚好相反。
一般 while 循环优于 until 循环，但在某些时候—也只是极少数情况下，until 循环更加有用。
until 语法格式:

```shell
until condition
do
    command
done
```

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。
以下实例我们使用 until 命令来输出 0 ~ 9 的数字：

```shell
a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

#### case

Shell case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。case语句格式如下：

```shell
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
```
case工作方式如上所示。取值后面必须为单词in，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 ;;。
取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 * 捕获该值，再执行后面的命令。
下面的脚本提示输入1到4，与每一种模式进行匹配：

```shell
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```
#### 跳出循环

在循环过程中，有时候需要在未达到循环结束条件时强制跳出循环，Shell使用两个命令来实现该功能：break和continue。

##### break命令

break命令允许跳出所有循环（终止执行后面的所有循环）。

```shell
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```
##### continue命令
continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。
对上面的例子进行修改：

```shell
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

### echo命令

Shell 的 echo 指令与 PHP 的 echo 指令类似，都是用于字符串的输出。命令格式：

```shell
echo string
```
#### 1.显示普通字符串

```shell
echo "It is a test"
echo It is a test
```

#### 2.显示转义字符

```shell
echo "\"It is a test\""
```
结果：

```shell
"It is a test"
```
#### 3.显示变量

```shell
read name 
echo "$name It is a test"
```

#### 4.显示换行

```shell
echo -e "OK! \n" # -e 开启转义
echo "It it a test"
```

#### 5.显示不换行
```shell
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```

#### 6.显示结果定向至文件

```shell
echo "It is a test" > myfile
```

#### 7.原样输出字符串，不进行转义或取变量(用单引号)

```shell
echo '$name\"'
```

#### 8.显示命令执行结果

```shell
echo `date`
```

### test命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

#### 数值测试

参数|说明
---|---
-eq|等于则为真
-ne|不等于则为真
-gt|大于则为真
-ge|大于等于则为真
-lt|小于则为真
-le|小于等于则为真

实例：

```shell
num1=100
num2=100
if test $[num1] -eq $[num2]
then
    echo '两个数相等！'
else
    echo '两个数不相等！'
fi
```

代码中的`[]`执行基本的算数运算

```shell
a=5
b=6
result=$[a+b]
```

#### 字符串测试

参数|说明
---|---
=|等于则为真
!=|不相等则为真
-z 字符串|字符串的长度为零则为真
-n 字符串|字符串的长度不为零则为真

实例演示：

```shell
num1="ru1noob"
num2="runoob"
if test $num1 = $num2
then
    echo '两个字符串相等!'
else
    echo '两个字符串不相等!'
fi
```

#### 文件测试

参数|说明
---|---
-e 文件名|如果文件存在则为真
-r 文件名|如果文件存在且可读则为真
-w 文件名|如果文件存在且可写则为真
-x 文件名|如果文件存在且可执行则为真
-s 文件名|如果文件存在且至少有一个字符则为真
-d 文件名|如果文件存在且为目录则为真
-f 文件名|如果文件存在且为普通文件则为真
-c 文件名|如果文件存在且为字符型特殊文件则为真
-b 文件名|如果文件存在且为块特殊文件则为真

实例演示：

```shell
cd /bin
if test -e ./bash
then
    echo '文件已存在!'
else
    echo '文件不存在!'
fi
```
### 函数

shell中函数的定义格式如下：

```shell
[ function ] funname [()]

{

    action;

    [return int;]

}
```
说明：
1、可以带function fun() 定义，也可以直接fun() 定义,不带任何参数。
2、参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值n(0-255)

### 输入/输出重定向

重定向命令列表如下：

命令	|说明
---|---
command > file	|将输出重定向到 file。
command < file	|将输入重定向到 file。
command >> file	|将输出以追加的方式重定向到 file。
n > file	|将文件描述符为 n 的文件重定向到 file。
n >> file	|将文件描述符为 n 的文件以追加的方式重定向到 file。
n >& m	|将输出文件 m 和 n 合并。
n <& m	|将输入文件 m 和 n 合并。
<< tag	|将开始标记 tag 和结束标记 tag 之间的内容作为输入。

>需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

#### 输出重定向

实例：

```shell
who > users.txt
echo "leoliu" > users.txt
echo "追加内容" >> users.txt
```

#### 输入重定向

```shell

wc -l < users.txt
```
`wc -l`命令用来统计 users.txt 文件的行数

#### 重定向深入讲解
一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。
- 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。
- 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。

默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。
如果希望 stderr 重定向到 file，可以这样写：

```shell
$ command 2 > file
```

如果希望将 stdout 和 stderr 合并后重定向到 file，可以这样写：

```shell
$ command > file 2>&1
或者
$ command >> file 2>&1
```
如果希望对 stdin 和 stdout 都重定向，可以这样写：

```shell
$ command < file1 >file2
```

command 命令将 stdin 重定向到 file1，将 stdout 重定向到 file2。


#### Here Document
Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。
它的基本的形式如下：

```
command << delimiter
    document
delimiter
```
它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command。

>注意：
>结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
>开始的delimiter前后的空格会被忽略掉。

实例
在命令行中通过 wc -l 命令计算 Here Document 的行数：

```shell
$ wc -l << EOF
    欢迎来到
    菜鸟教程
    www.runoob.com
EOF
```

输出结果：

```shell
3          # 输出结果为 3 行
```

```shell
cat << EOF
欢迎来到
菜鸟教程
www.runoob.com
EOF
```

#### /dev/null 文件

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，那么可以将输出重定向到 `/dev/null`：

```shell
$ command > /dev/null
```

`/dev/null` 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。但是 /dev/null 文件非常有用，将命令的输出重定向到它，会起到"禁止输出"的效果。
如果希望屏蔽 stdout 和 stderr，可以这样写：

```shell
$ command > /dev/null 2>&1
```

>注意：0 是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

### 文件包含

和其他语言一样，Shell 也可以包含外部脚本。这样可以很方便的封装一些公用的代码作为一个独立的文件。
Shell 文件包含的语法格式如下：

```shell
. filename   # 注意点号(.)和文件名中间有一空格

或

source filename
```

实例
创建两个 shell 脚本文件。
test1.sh 代码如下：

```shell
#!/bin/bash

url="http://www.runoob.com"
```

test2.sh 代码如下：

```shell
#!/bin/bash
#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo "菜鸟教程官网地址：$url"
```