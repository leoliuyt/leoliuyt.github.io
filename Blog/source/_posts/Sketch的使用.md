---
title: Sketch的使用
date: 2018-04-21 20:55:35
tags: Sketch
categories: tool
---

## 快捷键

显示`Layer List`

```
cmd + opt + 1
```

显示 `Inspector`

```
cmd + opt + 2
```

显示`Toolbar`

```
cmd + opt + t
```

显示`Canvas`

```
cmd + .
```

打开配置

```
cmd + ,
```

显示尺标

```
ctrl + R
```

显示网格

```
ctrl + G
```

使用辅助线

```
当我们在Ruler上移动鼠标的时候，辅助线的最小移动单位是1像素，大多数情况下，我们不需要这样的精度。这时，我们就可以按住shift，辅助线就会以10像素为单位移动了。
```

## 使用ArtBoards

`Artboard`是`Sketch`中的一个组织内容的工具，它可以在画布上先定一个指定大小的区域，我们可以把这个区域理解为我们设计时使用的屏幕。

按快捷键A，Sketch指针会变成一个十字，我们就可以在画布中创建Artboards了。此时，Inspector的部分会如下图所示

{% asset_img sketch_artboard@2x artboard %}

Sketch内置了常用操作系统上使用的常用尺寸，我们可以直接点击这些方案，就可以创建对应大小的artboard了。每一个artboard都由两部分组成，分别是标题和背景色：

一般情况下，我们用一个artboard来显示一个功能页，或者一屏UI，因此我们可以用artboards展示的功能来命名它们。而artboard的背景色则显示在Inspector部分，这里需要注意的是，虽然在画布中artboard显示为白色，但是默认情况下，artboard是没有背景色的，我们把artboard导出成png的时候，就会是透明背景。因此，如果你需要使用背景色，要记得选中Background color，然后设置成需要的颜色。

这里，给大家推荐一个Sketch插件，叫`Artboard Manager`。当我们有多个artboards的时候，默认情况下，拖动artboard会让它们彼此重叠覆盖在一起，我们得手工去处理它们的位置。但有了`Artboard Manager`就方便多了，我们直接选择一个或多个artboard，然后拖动到目标位置。它总是能依据我们对内容的直觉，帮我们自动排列好所有artboards。

## 使用Pages

`Pages`是`Sketch`中第二个组织内容的工具，它位于Layer List的顶部：

{% asset_img sketch_pages@2x pages %}

我们可以点击顶部的加号新建`Page`，也可以点击下面的箭头图标收起`Pages`列表。简单来说，`Page`就是不同的画布，在`Sketch`里，通常我们用`Pages`管理逻辑上相关，但是维度不同的内容。

## 使用Sketch UI模板

选择`File -> New from Template`：

可以看到Sketch默认带了5个模板，分别是：

Android Icon Design
Web Design
iOS App Icon
Material Design
Prototyping Tutorial

添加其他模板的方法是 下载以`.sketch`结尾的模板文件，直接用Sketch打开之后，选择`File -> Save as Template...`，给它起个名字。下次再新建项目的时候，就可以把它当作模板使用了。

从这里可以下载[iOS的模板文件](https://iosdesignkit.io)

## DP

Pixel Density(PD、像素密度)的概念，它指的是一英寸屏幕上的物理像素个数。显然，这个数值越大，屏幕就越精细。

```
dp = (width in pixels * 160) / screen density
```