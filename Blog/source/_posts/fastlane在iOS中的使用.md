---
title: fastlane在iOS中的使用
date: 2018-04-11 15:11:54
tags: fastlane
categories: tool
---

`fastlane`是部署和发布app的最简单的一种方式。它能处理像生成截图、处理签名、发布应用这些枯燥的任务。

## fastlane的安装

1、确保Xcode的命令行工具已安装

```
xcode-select --install
```

2、安装fastlane

```
gem install fastlane -NV
或者
brew cask install fastlane
```
3、在项目中创建fastlane

```
fastlane init
```
## fastlane初体验

1、进入工程目录

```
cd ~/Documents/Code/LL_Workspace/DMFastlane/DMFastlane
```

2、创建fastlane

```
fastlane init
```
执行上面命令后，会出现一个选择菜单如下：

```
[18:20:51]: What would you like to use fastlane for?
1. 📸  Automate screenshots
2. 👩‍✈️  Automate beta distribution to TestFlight
3. 🚀  Automate App Store distribution
4. 🛠  Manual setup - manually setup your project to automate your tasks
?  4
```
选择不用的选项，`fastlane`会为你创建不同功能对应的 `lane`。

比如我选择`1`，自动生成的`Fastfile`文件中会出现如下代码:

```ruby
default_platform(:ios)

platform :ios do
  desc "Generate new localized screenshots"
  lane :screenshots do
    capture_screenshots(workspace: "DMFastlane.xcworkspace", scheme: "DMFastlane")
  end
end
```
`screenshots`这个`lane`的功能就是自动生成截图。

如果我选`4`,就会生成一个供我们自定`lane`,如下:

```ruby
default_platform(:ios)

platform :ios do
  desc "Description of what the lane does"
  lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actions
  end
end
```

我们可以把我们想要实现的功能写在这个`custom_lane`中，比如我们想build工程生成ipa，可以调用`fastlane`提供的一个工具`build_ios_app`,这样实现：

```ruby
lane :custom_lane do
    # add actions here: https://docs.fastlane.tools/actions
    build_ios_app(scheme: "DMFastlane",
            workspace: "DMFastlane.xcworkspace",
            configuration: "Debug",
            export_method: "development",
            output_directory: "~/Desktop/", # Destination directory. Defaults to current directory.
            output_name: "DMFastlane.ipa"       # specify the name of the .ipa file to generate (including file extension)
            )
  end
```

然后再工程目录下，运行如下命令：

```
fastlane custom_lane
```

就会在`output_directory`指定的目录下生成`output_name`指定的`ipa`包

这样fastlane的初次体验就完成了，以后生成ipa执行一个命令就可以了。

## QA
Q: `Error: fastlane gym produces error: method 'to_plist' not defined in Array`

A:
```
# 先卸载
rvm @global do gem uninstall fastlane 
rvm all do gem uninstall fastlane
gem uninstall fastlane

# 从新安装
gem install fastlane -NV
```

## 参考资料

[fastlane官网](https://docs.fastlane.tools)

[iOS中fastlane的使用](http://blog.devzeng.com/blog/ios-fastlane-in-action.html)

[小团队的自动化发布－Fastlane带来的全自动化发布](https://whlsxl.github.io/fastlane1/)