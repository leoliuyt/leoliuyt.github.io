---
title: Runloop
date: 2018-05-21 14:16:55
tags: runloop
categories: Objective-C
---

什么是RunLoop？

RunLoop是通过内部维护的**事件循环**来对事件/消息进行管理的一个对象。

事件循环：

- 没有消息处理时，休眠以避免资源浪费（用户态通过系统调用进入内核态）
- 有消息需要被处理时，被立刻唤醒（内核态到用户态）

用户态：

内核态：

```Objective-C
int main(int argc, char * argv[]) {
    @autoreleasepool {
        NSLog(@"%s",__func__);
        int a = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
        NSLog(@"a = %tu",a);
        return a;
    }
}
```

打印的结果为：

```
main
```
Q: main函数为什么能够保持不退出？

UIApplicationMain 中会启动一个运行循环，用来接收消息（点击屏幕、滑动列表、处理网络请求的放回），对事件进行处理，处理完后，等待。等待不等于死循环。这里主要是发生了状态切换。

UIApplicationMain内部启动了主线程上的一个RunLoop，而RunLoop是对事件循环的维护机制，在有事做的时候做事，没事做的时候，由用户态切换到内核态，避免资源占用，使当前线程处于休眠状态。

## Runloop的数据结构

NSRunLoop是对CFRunLoop的封装，提供了面向对象的API。

- CFRunLoop
    - a
    - b
    - c

- CFRunLoopMode
- Source/Timer/Observer

```C
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;			/* locked for accessing mode list */
    __CFPort _wakeUpPort;			// used for CFRunLoopWakeUp 
    Boolean _unused;
    volatile _per_run_data *_perRunData;              // reset for runs of the run loop
    pthread_t _pthread;
    uint32_t _winthread;
    CFMutableSetRef _commonModes;
    CFMutableSetRef _commonModeItems;
    CFRunLoopModeRef _currentMode;
    CFMutableSetRef _modes;
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFTypeRef _counterpart;
};
```

### CFRunLoopObserver

source0 需要手动切换线程
source1 具备唤起线程的能力


观测时间点

- kCFRunLoopEntry//入口时机 即将进入RunLoop
- kCFRunLoopBeforeTimers//通知观察RunLoop将要对Timer事件处理
- kCFRunLoopBeforeSources//通知观察RunLoop将要对Sources事件处理
- kCFRunLoopBeforeWaiting ##重要 即将用户态到内核态切换
- kCFRunLoopAfterWaiting ## 内核态切换到用户态不久
- kCFRunLoopExit //RunLoop退出

### 数据结构之间的关系
RunLoop 与线程一对一
RunLoop与Model 是一对多的关系
Model与 Source 、Timer 、Observer 是以对多的关系

如果RunLoop运行在Mode1上面，对应Mode2中的事件是无法处理的

滑动tableView时，轮播图停止了滚动是什么原因？

一个timer想要同时加入到两个Mode中，该怎么做？


### CommonMode的特殊性

NSRunLoopCommonModes

    - CommonMode不是实际存在的一种Mode。
    - 是同步source/Timer/Observer到多个Mode中的一种技术方案。

## 事件循环的实现机制

RunLoop启动后发通知告诉Observer即将进入RunLoop



```
void CFRunLoopRun()

```

[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)