---
title: UI绘制原理
date: 2018-05-26 22:17:08
tags: iOS
categories: Cocoa Touch
---

## 图像显示原理

CPU与GPU是通过总线连接起来，通过CPU输出一个位图，经由总线传输在合适的时机上传到GPU，GPU获取到位图后，会进行图层渲染、纹理合成，将结果放入帧缓存区中，由视频控制器根据VSyn信号，在指定时间之前提取帧缓存区中的屏幕显示内容，最终显示到手机屏幕上。

{% asset_img display_principle.png 显示原理 %}

CPU与GPU分别做了什么？

创建UIView后，显示部分由CALayer负责，CALayer有一个contents属性，就是我们最终要绘制到屏幕上的位图，如果我们创建的是UILabel，那么contents上最终要绘制就是“Hello”的位图,系统在合适的时候回调给我们一个drawRect的方法，然后我们可以在此基础上绘制一些我们想自定义绘制的内容，绘制好的这个位图，会经过Core Animation提交给GPU部分的OpenGL渲染管线，进行最终的位图的渲染以及文理的合成，最终显示到屏幕上。

{% asset_img display_sepereate.png CPU与GPU分工 %}

CPU要做的工作

1. Layout
    - UI布局
    - 文本计算
2. Display
    - 绘制（drawRect）
3. Prepare
    - 图片解码
4. Commit
    - 提交位图

GPU 渲染管线

- 顶点着色
- 图元装配
- 光栅化
- 片段着色
- 片段处理

提交到帧缓冲区中

## UI卡顿、掉帧的原因

一般来说，页面滑动的流畅性是60FPS，代表着每秒钟有六十帧画面更新，换种说法就是每一帧画面的生成时间在1/60秒即16.7毫秒以内就不会卡顿。在16.7毫秒内需要CPU和GPU共同来生成一帧的数据。如果时间超过了16.7ms就会产生卡顿。

{% asset_img frame_off.png 掉帧原因 %}

在 VSync 信号到来后，系统图形服务会通过 CADisplayLink 等机制通知 App，App 主线程开始在 CPU 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 CPU 会将计算好的内容提交到 GPU 去，由 GPU 进行变换、合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待下一次 VSync 信号到来时显示到屏幕上。由于垂直同步的机制，如果在一个 VSync 时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

从上面的图中可以看到，CPU 和 GPU 不论哪个阻碍了显示流程，都会造成掉帧现象。所以开发时，也需要分别对 CPU 和 GPU 压力进行评估和优化。

## 滑动优化方案

CPU

- 对象的创建、调整、销毁
- 预排版（布局计算、文本计算）
- 预渲染（文本等异步绘制，图片编解码等）

GPU

- 文理渲染（避免离屏渲染）
- 视图混合（减少视图层级）


## UIView的绘制原理

当调用`UIView`的`setNeedsDisplay`方法时，会调用`CALayer`的同名方法`setNeedsDisplay`,这时并没有立即发生绘制，而只是相当于在当前layer打上了脏标记， 会在Runloop即将结束时才会调用`[CALayer display]`,而这个方法的内部会判断是否实现了displayLayer这个方法，如果没有实现，那么走系统调用，如果实现了，就为我们异步绘制提供了入口。

{% asset_img display_flow.png 绘制流程 %}

### 系统绘制实现

{% asset_img system_display_new.png 系统绘制流程 %}

### 异步绘制实现

通过实现layer的代理方法displayLayer

- 代理负责生成对应的bitmap
- 设置该bitmap作为layer.contents属性的值

{% asset_img asyn_display.png 异步绘制流程 %}

### 离屏渲染

On-Screen Rendering 意为当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行渲染操作

Off-Screen Rendering 意为离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作

当设置某些UI视图的图层属性，如果标记为未合成之前不能用于显示，那么就会触发离屏渲染

触发离屏渲染的条件

- 圆角（和maskToBounds一起使用时）
- 图层蒙版
- 阴影
- 光栅化

为何要避免离屏渲染？

触发离屏渲染，增加GPU工作量，而GPU的工作的增加可能导致CPU+GPU产生一帧画面的时间超过16.7ms，就会产生卡顿掉帧，所以要避免离屏渲染。

- 创建新的渲染缓存区
- 上下文切换



## 参考

[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[谈谈 iOS 中图片的解压缩](http://blog.leichunfeng.com/blog/2017/02/20/talking-about-the-decompression-of-the-image-in-ios/#jtss-tsina)


