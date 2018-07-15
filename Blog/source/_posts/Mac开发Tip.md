---
title: Mac开发Tip
date: 2018-04-13 09:38:16
tags: Mac 
categories: Cocoa
---

{% asset_img Cocoa@2x.png Cocoa %}

## 删除`Main.Storyboard`,纯代码开发

如果想纯代码开发Mac应用，删除`Main.Storyboard`,用代码构建初始化窗口，步骤如下：

1、删除`Main.Storyboard`,修改`info.plist`

{% asset_img configure@2x.png 修改配置 %}

2、修改`main.m`中代码如下：

```objc
#import "AppDelegate.h"
int main(int argc, const char * argv[]) {
    NSApplication*applicaton = [NSApplication sharedApplication];
    id delegete = [[AppDelegate alloc]init];
    
    applicaton.delegate= delegete;
    return NSApplicationMain(argc, argv);
}
```

3、修改`AppDelegate.m`中代码如下:

```objc
- (void)applicationDidFinishLaunching:(NSNotification *)aNotification {
    // Insert code here to initialize your application
    
    //创建Window的两种方式
    DMRootViewController *vc = [[DMRootViewController alloc] init];
    //创建Window方式一：
//    self.window = [NSWindow windowWithContentViewController:vc];
    
    //创建Window方式二：
    self.window = [[NSWindow alloc] initWithContentRect:CGRectMake(0,0,800,600)
                                              styleMask:NSWindowStyleMaskTitled|NSWindowStyleMaskMiniaturizable|NSWindowStyleMaskResizable|NSWindowStyleMaskClosable
                                                backing:NSBackingStoreRetained
                                                  defer:NO];
    self.window.contentViewController = vc;
  
    
    //window 显示出来也有两种方式：
    
    [self.window center];
    //window显示方式一：
//    [self.window makeKeyAndOrderFront:nil];
    //window显示方式二：
    [self.window makeKeyWindow];
    //将window与APP绑定，如果不绑定，无法显示Window。
    [NSApp beginModalSessionForWindow:self.window];
}
```

这里要注意`DMRootViewController`中再没使用nib的情况下，要通过实现`loadView`来加载`Controller`中的View

## 给NSView设置背景颜色

`NSView`没有`backgroudColor`这个属性，要设置背景色要通过layer来设置，并且要开开启`wantsLayer`这个属性，代码如下：

```objc
    self.view.wantsLayer = YES;
    self.view.layer.backgroundColor = [NSColor orangeColor].CGColor;
```

## 给NSScrollView设置背景颜色

`NSScrollView`有`backgroundColor`这个属性，可以通过设置这个属性和`drawsBackground`来设置`NSScrollView`的背景颜色,`drawsBackground`不设置的话也是无法显示背景颜色的
```objc
_scrollView.backgroundColor = [NSColor redColor];
_scrollView.drawsBackground = YES;

//clear color 要设置为NO
_scrollView.backgroundColor = [NSColor clearColor];
_scrollView.drawsBackground = NO;
```

## 给NSCollectionView设置透明背景

```objc
[_collectionView setBackgroundColors:@[[NSColor clearColor]]];
```

## 访问网络

Mac应用要想访问网络，必须要手动配置一下才可以，配置如图：

{% asset_img network@2x.png 访问网络配置 %}

勾选上`Outgoing Connections(tClien)`就可以访问网络了，如有想要使用摄像头、麦克风等硬件设备，那么可以勾选上`Camera`、`Microphone`等配置项

## window被close后，点击`Dock`重新显示出来

当你的界面被关闭后，想点击`Dock`重新显示(类似QQ、微信的效果)，可以分两种情况实现：

1、使用`Main.Storyboard`的情况

实现`AppDelegate`中的方法，如下：

```objc
-(BOOL)applicationShouldHandleReopen:(NSApplication *)theApplication hasVisibleWindows:(BOOL)flag
{
    if (!flag){//为了让close后的window，点dock中应用图标显示出来
        NSStoryboard *sb = [NSStoryboard mainStoryboard];
        NSWindowController *windowController = [sb instantiateInitialController];
        [windowController.window makeKeyAndOrderFront:self];
        return YES;
    }
    return NO;
}
```

2、使用纯代码的情况

```objc
-(BOOL)applicationShouldHandleReopen:(NSApplication *)theApplication hasVisibleWindows:(BOOL)flag
{
    if (!flag){//为了让close后的window，点dock中应用图标显示出来
        [self.window makeKeyAndOrderFront:self];
        return YES;
    }
    return NO;
}
```

如果想点击`x`直接退出程序,可以通过实现`NSWindowDelegate`的代理来处理

```objc
- (BOOL)windowShouldClose:(NSWindow *)sender
{
    [NSApp terminate:nil];
    return YES;
}
```

## NSView的动画

要实现NSView从底部出现的动画，要注意必须设置两个神奇属性，否则根本没有动画效果：

```objc
    self.oneView = [[CustomView alloc] init];
    self.oneView.wantsLayer = YES;
    [self.view addSubview:self.oneView];
    __block MASConstraint *bottomContraint = nil;
    [self.oneView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.right.equalTo(self.view);
        bottomContraint=make.bottom.equalTo(self.view).offset(60.);
        make.height.equalTo(@60.);
    }];
    //保证初始状态正确
    [self.view layoutSubtreeIfNeeded];
    
    bottomContraint.offset(0.);

    [NSAnimationContext runAnimationGroup:^(NSAnimationContext * _Nonnull context) {
        context.duration = 5;
        context.allowsImplicitAnimation=YES;
        [self.view layoutSubtreeIfNeeded];
    } completionHandler:nil];
```

其中`NSView`的`wantsLayer`和 `NSAnimationContext`的`allowsImplicitAnimation`这两个属性是必须要设置的！

## 设置阴影+圆角

```Objective-C
//设置阴影和圆角
    self.baseView.layer.cornerRadius = 4;
    self.baseView.layer.masksToBounds = YES;
    NSShadow *shadow = [[NSShadow alloc] init];
    //设置阴影为白色
    [shadow setShadowColor:[NSColor colorWithWhite:0 alpha:0.4]];
    //设置阴影为右下方
    [shadow setShadowOffset:NSMakeSize(0, 4)];
    [shadow setShadowBlurRadius:12];
    [self.baseView setShadow:shadow];
```
