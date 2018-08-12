---
title: VSCode调试Python
date: 2018-08-12 16:50:35
tags: vscode
categories: [Python,tool]
---

## 本地调试

本地调试比较简单，请参考[VSCode官方文档](https://code.visualstudio.com/docs/python/debugging)

## 远程调试

进行远端调试的需要满足一下几个条件：

- 本地和远端有相同的源码
- 本地和远端都安装`ptvsd`版本是`3.0.0`
- 配置远端服务器，开启调试端口权限
- 远端源代码中添加 `ptvsd`相关代码
- 本地源代码中添加 注释了的`ptvsd`相关代码，保证与远端行数相同
- 本地VSCode配置

具体步骤如下：

### 本地虚拟环境配置

将源代码下载到本地,在本地搭建虚拟环境

```
#进入项目根目录，创建虚拟环境 
# --python=python2.7 用于指定python版本
virtualenv ydcvenv --python=python2.7

#进入虚拟环境
source ./ydcvenv/bin/activate

这是后命令行中前部会变成,这代表正处在虚拟环境中
(ydcvenv) ➜  ydc

#安装依赖 (这是我项目需要的依赖)
brew install spawn-fcgi

pip install ptvsd==3.0.0
pip install flup==1.0.2
pip install web.py
pip install wechatpy
pip install wechatpy[cryptography]

```

配置VSCode

在虚拟环境下运行下面两条命令，用来获取当前虚拟环境下的 Python 解释器路径和 Python 包安装路径 

```

which python
结果如下：
/Users/leoliu/Desktop/ydc/ydcvenv/bin/python

python -c "from distutils.sysconfig import get_python_lib; print get_python_lib()"

结果如下：
/Users/leoliu/Desktop/ydc/ydcvenv/lib/python2.7/site-packages

```

将上面两条命令的结果加入到VSCode的配置文件中，让 VSCode 知道用哪个 Python 解释器，去哪里寻找模块做代码索引和补全提示

打开VSCode配置文件的步骤如下图所示:

{% asset_img vscode_setting.png 工作区配置 %}

将下面内容添加到`settings.json`中

```
{
    "python.pythonPath": "/Users/leoliu/Desktop/ydc/ydcvenv/bin/python",
    "python.autoComplete.extraPaths": ["/Users/leoliu/Desktop/ydc/ydcvenv/lib/python2.7/site-packages"]
}
```
配置完上面的操作后，就可以在本地运行下载的python程序了

### 阿里云开启一个端口用于调试

远端代码是放在阿里云服务器上的，调试的时候需要一个调试的端口，如果你不开启这个端口的访问权限默认是无法访问的，所以这里需要配置一下阿里云的安全规则

{% asset_img aliyun_setting.png 阿里云配置 %}

上图显示我配置了8000端口用于python的debug

### 添加远程调试的代码

在远端的源码的入口文件中添加如下代码:
```
import ptvsd
# Allow other computers to attach to ptvsd at this IP address and port, using the secret
ptvsd.enable_attach("aaa", address = ('0.0.0.0', 8000))#也可以把0.0.0.0换成服务器的私有ip
# Pause the program until a remote debugger is attached
ptvsd.wait_for_attach()

import time
time.sleep(5)
print('test')
```

如果只是添加上面的代码无法调试子线程的代码，所以要改成下面的：

```
import ptvsd
import socket
try:
    address = ('127.0.0.1', 9090)#服务器上python服务的端口号
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    s.bind(address)
except socket.error:
    print 'socket.error:'
    # Allow other computers to attach to ptvsd at this IP address and port, using the secret
    ptvsd.enable_attach("aaa", address = ('0.0.0.0', 8000))#也可以把0.0.0.0换成服务器的私有ip
    # Pause the program until a remote debugger is attached
    ptvsd.wait_for_attach()

import time
time.sleep(5)
print('test')
```
相应的本地代码也要加上上面的代码，不过要将他们都注释掉，在本地添加这些代码只是为了保证行数的一致性

### 本地VSCode中DEBUG的配置

打开VSCode debug的配置

{% asset_img debug_setting.png debug配置 %}

在配置文件中添加如下配置:

```
{
    "name": "Python: Attach(Remote Debug)",
    "type": "python",
    "request": "attach",
    "localRoot": "${workspaceFolder}",
    "remoteRoot":"/root/www/ydc",
    // "remoteRoot": "${workspaceFolder}",
    "port": 8000,
    "secret": "aaa",
    "host": "47.11.11.11" # 服务器的公开IP地址
},
```
至此配置完成，服务器启动程序后，就可以在本地通过`fn+F5`进行调试了

## 参考
[Python debugging configurations in VS Code](https://code.visualstudio.com/docs/python/debugging)
[使用 VS Code 远程调试 Python 程序](https://blog.jamespan.me/2016/06/30/remote-debug-python-with-vscode)