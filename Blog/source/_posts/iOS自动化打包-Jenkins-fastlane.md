---
title: iOS自动化打包(Jenkins+fastlane)
date: 2017-07-22 11:50:36
tags: fastlane
categories: tool
---

## Jenkins的安装

```
//通过homebrew 安装jenkins
brew install jenkins


//链接 launchd 配置文件
ln -sfv /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents

//
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist

//启动Jenkins
jenkins

```

在浏览器中输入`localhost:8080`会提示默认密码保存的路径，去路径中找到密码复制到输入框后，即可登录
```
//查看默认密码
cat /Users/leoliu/.jenkins/secrets/initialAdminPassword
```
{% asset_img jenkins_unlock 输入密码 %}



安装过程中可能会出现如下错误:

{% asset_img jenkins_offline jenkins离线错误 %}

解决办法如下：

在提示你offline的那个页面，不要动。然后打开一个新的tab，输入网址http://localhost:8080/pluginManager/advanced。 这里面最底下有个【升级站点】，把其中的链接改成http的就好了，http://updates.jenkins.io/update-center.json。 然后在服务列表中关闭jenkins，再启动，这样就能正常联网了。

{% asset_img jenkins_error_offline 升级站点 %}

依次会出现如下界面

{% asset_img jenkins_install_custom_plug 初始化推荐插件 %}

{% asset_img jenkins_create_account 创建第一个用户 %}

{% asset_img jenkins_configure jenkins配置 %}

退出jenkins 重启服务

{% asset_img jenkins_success jenkins成功 %}