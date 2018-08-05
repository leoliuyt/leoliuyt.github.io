---
title: Python环境搭建（Nginx+Flup+spawn-fcgi+web.py）
date: 2018-08-04 23:23:06
tags: Python
categories: Python
---

## 安装python
CentOS 6.8 中默认安装的python版本是2.6.6

首先我们将安装python2.7.10并设为默认版本

现在python2.7.10
```
wget http://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz
```

解压
```
tar -zxvf Python-2.7.10.tgz
```
进入解压目录
```
cd Python-2.7.10
```
编译
```
./configure --prefix=/usr/local
make && make install
```
配置软连接，让系统pyhon默认指向python2.7.10
```
//备份原来的2.6.6版本
mv /usr/bin/python /usr/bin/python2.6.6
//创建新版本python的软连接
ln -s /usr/local/bin/python2.7 /usr/bin/python
```
查看python版本

```
python -V
```
修复yum与python2.7不兼容的问题

```
vim /usr/bin/yum
将第一行修改为
#!/usr/bin/python2.6.6
```
## 升级pip

下载pip安装包

```
wget https://bootstrap.pypa.io/get-pip.py
```

安装
```
python get-pip.py
```
创建软连接

```
ln -s /usr/local/python27/bin/pip2.7 /usr/bin/pip
```
如果上面那句提示已经存在pip，那么需要如下操作:

```
#备份原pip
mv /usr/bin/pip /usr/bin/pip2.6.6
#重新创建软连接
ln -s /usr/local/python27/bin/pip2.7 /usr/bin/pip
```

## 安装Flup

下载源码安装包

```
wget http://www.saddi.com/software/flup/dist/flup-1.0.2.tar.gz
```

解压
```
tar xvzf flup-1.0.2.tar.gz
```

安装

```
cd flup-1.0.2
python setup.py install
```

## 安装spawn-fcgi

```
wget http://www.lighttpd.net/download/spawn-fcgi-1.6.3.tar.gz
```

解压
```
tar xvzf spawn-fcgi-1.6.3.tar.gz
```

安装

```
cd spawn-fcgi-1.6.3
./configure --prefix=/usr/bin/
make && make install
```

关于`./configure`

```
源码的安装一般由3个步骤组成：配置(configure)、编译(make)、安装(make install)。

Configure是一个可执行脚本，它有很多选项，在待安装的源码路径下使用命令./configure –help输出详细的选项列表。

其中--prefix选项是配置安装的路径，如果不配置该选项，安装后可执行文件默认放在/usr /local/bin，库文件默认放在/usr/local/lib，配置文件默认放在/usr/local/etc，其它的资源文件放在/usr /local/share，比较凌乱。

如果配置--prefix，如：

./configure --prefix=/usr/local/test
可以把所有资源文件放在/usr/local/test的路径中，不会杂乱。
用了—prefix选项的另一个好处是卸载软件或移植软件。当某个安装的软件不再需要时，只须简单的删除该安装目录，就可以把软件卸载得干干净净；移植软件只需拷贝整个目录到另外一个机器即可（相同的操作系统）。

当然要卸载程序，也可以在原来的make目录下用一次make uninstall，但前提是make文件指定过uninstall。
```

## 安装web.py

使用pip安装

```
pip install web.py
```

## Nginx安装和配置
进入nginx 配置目录

```
cd /etc/nginx/conf.d/default.conf
```
编辑配置文件

```
#
# The default server
#

server {
    listen       80 default_server;
    #listen       [::]:80 default_server;
    server_name  localhost;
   # root         /usr/share/nginx/html;
    root /root/www;#代码路径
    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        #root /root/www;
        #index index.html index.htm index.php;
        include fastcgi_params;
        #fastcgi_index app.py;
        fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_script_name;
        fastcgi_pass 127.0.0.1:9002;
    }
    location /static/ {
         if (-f $request_filename) {
             rewrite ^/static/(.*)$  /static/$1 break;
        }
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
    #location ~ \.php$ {
    #        root           /www;
    #        fastcgi_pass   127.0.0.1:9000;
    #        fastcgi_index  index.php;
    #        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    #        include        fastcgi_params;
    # }
}
```

### nginx相关命令

开启nginx
```
nginx
```

重启nginx
```
nginx -s reload
```

停止nginx

```
nginx -s stop
```

## 启动一个spawn-fcgi

运行脚本与app.py在同一目录下
```
#/bin/sh

APP_NAME=app.py
APP_PATH="`pwd`/$APP_NAME"

SPAWN_PATH=spawn-fcgi

PID=`ps ax | grep $APP_NAME | grep python | awk '{print $1}'`
if [ "$PID" != "" ]
then
	kill -9 $PID
	sleep 1
fi

# -a 绑定的ip地址
# -d 目录
# -f fcgi应用文件目录
# -p 绑定的端口号
# spawn-fcgi -d /root/www -f /root/www/app.py -a 127.0.0.1 -p 9002 -n
$SPAWN_PATH -a "127.0.0.1" -p 9090 -f $APP_PATH -n

```

### 编写fcgi应用app.py

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import web
urls = (
    '/.*','index'
)
class index(object):
    def GET(self):
        return 'Hello python!'

if __name__ == '__main__':
    app = web.application(urls,globals())
    web.wsgi.runwsgi = lambda func, addr=None: web.wsgi.runfcgi(func, addr)
    app.run()
```

## 参考

[centeros 升级 python 版本,以及添加pip](https://blog.csdn.net/u010856284/article/details/78049685)

[nginx python spawn-fcgi Flup webpy搭建python的web环境](https://blog.csdn.net/wzm112/article/details/7922520)

[Webpy + Nginx with FastCGI搭建Web.py](http://webpy.org/cookbook/fastcgi-nginx.zh-cn)

[webpy + nginx + fastcgi 构建python应用](https://www.cnblogs.com/LD-linux/p/4089205.html)