---
title: 定制Mac的启动任务
date: 2018-04-10 12:29:48
tags: Mac
categories: shell
---

在Mac上设置启动任务可以通过一下两种方式（我实验成功了这两种）:
- 通过系统设置界面设置
- 利用launchd设置
<!-- Mac下的启动服务主要有三个地方可配置： 
1，系统偏好设置->帐户->登陆项 
2，/System/Library/StartupItems 和 /Library/StartupItems/ 
3，launchd 系统初始化进程配置。  -->

## 系统设置界面

1、创建要运行的脚本 (run.sh)

```shell
#!/bin/sh
ls > ~/a.txt
```
设置脚本运行权限

```shell
chmod +x run.sh
```

2、设置

设置脚本打开方式

右击脚本->`显示简介`->`打开方式`-> 可以选terminal

{% asset_img introduce.png 简介图 %}

在`系统偏好设置`->`用户与群众`->`当前用户`->`登陆项`->`+`添加上面所创建的脚本

{% asset_img launch_config.png 启动设置图 %}

上面操作完成后，系统在每次重新启动后就会自动调用脚本

## 利用launchd设置

`launchd` 是 Mac 下用于初始化操作系统的关键进程。通过启动硬盘指定目录下的配置文件，来完成启动任务。这些文件为 plist，本质上是 XML。

### 目录配置

Mac 下 Launchd 指定目录有：

`~/Library/LaunchAgents`

`/Library/LaunchAgents`

`/Library/LaunchDaemons`

`/System/Library/LaunchAgents`

`/System/Library/LaunchDaemons`

其中的区别：

`/System/Library` 目录下存放的是系统文件

`/Library` 目录是系统管理员存放的第三方软件

`~/Library/` 目录是用户存放的第三方软件

`LaunchDaemons` 是用户未登陆前就启动的服务

`LaunchAgents` 是用户登陆后启动的服务

### Plist 配置

这里列举几个比较有用的配置关键字：

`Label` - 标识符，用来表示该任务的唯一性(与plist名称一致)

`Program` - 程序名称，用来说明运行哪个程序、脚本

`ProgramArguments` - 数组程序名，同上，只是可以运行多个程序

`WatchPaths` - 监控路径，当路径文件有变化是运行程序，也是数组

`RunAtLoad` - 是否在加载的同时

`StartInterval `间隔执行的时间，单位是秒

`StartCalendarInterval`：运行的时间，单个时间点使用`<dic>`，多个时间点使用 `<array> <dict></dict><array>`

更多配置可以查看[苹果文档](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html#//apple_ref/doc/man/5/launchd.plist)

### 实例

1. 设置定时任务

进入 `~/Library/LaunchAgents`创建一个plist文件`com.leoliu.launch.awake.plist`,
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.leoliu.launch.awake</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/lbq/Documents/Code/LL_Workspace/DMShell/awake.sh</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StartInterval</key>
	<integer>5</integer>
</dict>
</plist>
```
执行下面的`load`和`start`命令就可以了

```shell
cd ~/Library/LaunchAgents

# 加载任务, -w选项会将plist文件中无效的key覆盖掉，建议加上
$ launchctl load -w com.leoliu.launch.awake.plist

# 删除任务
$ launchctl unload -w com.leoliu.launch.awake.plist

# 查看任务列表, 使用 grep '任务部分名字' 过滤
$ launchctl list | grep 'com.leoliu'

# 开始任务
$ launchctl start  com.leoliu.launch.awake.plist

# 结束任务
$ launchctl stop   com.leoliu.launch.awake.plist
```
2. 用户输入密码登录成功后自动脚本

因为每次登录成功后，`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`文件会发生变化，所有可以通过观察该文件的变化，来确定是否是输入密码后登陆进来的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.leoliu.launch.awake</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Users/lbq/Documents/Code/LL_Workspace/DMShell/awake.sh</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>WatchPaths</key>
	<array>
		<string>/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist</string>
	</array>
</dict>
</plist>
```
然后执行：

```shell
$ launchctl load -w com.leoliu.launch.awake.plist
$ launchctl start  com.leoliu.launch.awake.plist
```
按快捷键`option+command+关机键`休眠后，在重新唤起，输入密码就会看到结果

## 参考

[mac环境下开机自启动Shell脚本](http://makaiqian.com/setting-boot/)

[利用 Launchd 定制 Mac 启动任务](https://zhuanlan.zhihu.com/p/25049770)

[Daemons and Services Programming Guide](https://developer.apple.com/library/content/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/Introduction.html#//apple_ref/doc/uid/10000172i-SW1-SW1)

[launchd.plist说明](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man5/launchd.plist.5.html#//apple_ref/doc/man/5/launchd.plist)

[Mac OS X: Creating a login hook](https://support.apple.com/de-at/HT2420)

[Running script upon login mac [closed]](https://stackoverflow.com/questions/6442364/running-script-upon-login-mac)