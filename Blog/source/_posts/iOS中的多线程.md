---
title: iOS中的多线程
date: 2018-04-19 14:17:55
tags: [多线程,iOS]
categories: Objective-C
---

## 多线程相关概念

### 同步、异步

同步：完成需要做的任务后才会返回，进行下一任务。不会开新的线程，会在当前线程中执行任务。

异步：不会等待任务完成才返回，会立即返回。必定会开启新的线程，线程的申请是由异步负责，起到开分支的作用。

### 串、并行、全局、主队列

串行队列： 任务依次执行，同一时间队列中只有一个任务在执行，每个任务只有在前一个任务执行完成后才能开始执行。

并行队列：任务并发执行，你唯一能保证的是，这些任务会按照被添加的顺序开始执行。但是任务可以以任何顺序完成。

全局队列：隶属于并行队列

主队列：隶属于串行队列，不能与 sync 同步方法搭配使用，会造成死循环

### 交叉关系

--- | 同步 | 异步 
-------------|-------------|-------------
串行队列 | 不会新建线程，依然在当前线程上</p>类似同步锁，是同步锁的替代方案</p>✅ 常用| 会新建线程，只开一条线程</p>一条线程就够了</p> 每次使用 createDispatch 方法就会新建一条线程，多次调用该方法，会创建多条线程，多条线程间会并行执行
并行队列 | 不会新建线程，依然在当前线程上</p> | 会新建线程，可以开多条线程</p> iOS7-SDK 时代一般是5、6条， iOS8-SDK 以后可以50、60条 </p> ✅ 常用

## iOS中的多线程技术

iOS中的三种多线程技术：

- NSThread
- NSOperation/NSOperationQueue
- GCD(Grand Central Dispatch)

### NSThead

NSThead 启动流程

```
start()->创建pthread->执行main -> [target performSelector:selector] -> exit;
```

### NSOperation/NSOperationQueue

`NSOperation`、`NSOperationQueue` 是基于 GCD 更高一层的封装，完全面向对象。但是比 GCD 更简单易用、代码可读性也更高。

使用`NSOperation`、`NSOperationQueue`有一下几种优势：

- 可添加完成的代码块，在操作完成后执行。
- 添加操作之间的依赖关系，方便的控制执行顺序。
- 设定操作执行的优先级。
- 可以很方便的取消一个操作的执行。
- 可以控制最大并发量
- 使用 KVO 观察对操作执行状态的更改：isReady、isExecuteing、isFinished、isCancelled。

`NSOperation`表示了一个独立的计算单元。作为一个抽象类，它给了它的子类一个十分有用而且线程安全的方式来建立状态、优先级、依赖性和取消等的模型。或者，你不是很喜欢再自己继承`NSOperation`的话，框架还提供`NSInvocationOperation`、`NSBlockOperation`两个继承自`NSOperation`的类。

所以我们有三种方式来封装操作：

- 使用子类 NSInvocationOperation
- 使用子类 NSBlockOperation
- 自定义继承自 NSOperation 的子类，通过实现内部相应的方法来封装操作。

但是仅仅把计算封装进一个对象而不做其他处理显然没有多大用处，我们还需要`NSOperationQueue`来大显身手。
`NSOperationQueue`控制着这些并行操作的执行，它扮演者优先级队列的角色，让它管理的高优先级操作(`NSOperation -queuePriority`)能优先于低优先级的操作运行的情况下，使它管理的操作能基本遵循先进先出的原则执行。此外，在你设置了能并行运行的操作的最大值(`maxConcurrentOperationCount`)之后，`NSOperationQueue`还能并行执行操作。

让一个`NSOperation`操作开始，你可以直接调用`-start`，或者将它添加到`NSOperationQueue`中，添加之后，它会在队列排到它以后自动执行。

现在让我们通过怎样使用和怎样通过继承实现功能来看看`NSOperation`稍微复杂的部分。

#### 状态

`NSOperation`包含了一个十分优雅的状态机来描述每一个操作的执行。

```objc
isReady → isExecuting → isFinished
```

为了替代不那么清晰的state属性，状态直接由上面那些keypath的KVO通知决定，也就是说，当一个操作在准备好被执行的时候，它发送了一个KVO通知给`isReady`的keypath，让这个keypath对应的属性`isReady`在被访问的时候返回YES。

每一个属性对于其他的属性必须是互相独立不同的，也就是同时只可能有一个属性返回YES，从而才能维护一个连续的状态：`- isReady`: 返回 YES 表示操作已经准备好被执行, 如果返回NO则说明还有其他没有先前的相关步骤没有完成。 `- isExecuting`: 返回YES表示操作正在执行，反之则没在执行。 `- isFinished` : 返回YES表示操作执行成功或者被取消了，`NSOperationQueue`只有当它管理的所有操作的`isFinished`属性全标为YES以后操作才停止出列，也就是队列停止运行，所以正确实现这个方法对于避免死锁很关键。

#### 取消

早些取消那些没必要的操作是十分有用的。取消的原因可能包括用户的明确操作或者某个相关的操作失败。

与之前的执行状态类似，当`NSOperation`的`-cancel`状态调用的时候会通过KVO通知`isCancelled`的keypath来修改`isCancelled`属性的返回值，`NSOperation`需要尽快地清理一些内部细节，而后到达一个合适的最终状态。特别的，这个时候`isCancelled`和`isFinished`的值将是YES，而`isExecuting`的值则为NO。

#### 优先级

不可能所有的操作都是一样重要，通过以下的顺序设置queuePriority属性可以加快或者推迟操作的执行：

```objc
NSOperationQueuePriorityVeryHigh
NSOperationQueuePriorityHigh
NSOperationQueuePriorityNormal
NSOperationQueuePriorityLow
NSOperationQueuePriorityVeryLow
```

此外，有些操作还可以指定`threadPriority`的值，它的取值范围可以从0.0到1.0，1.0代表最高的优先级。鉴于`queuePriority`属性决定了操作执行的顺序，`threadPriority`则指定了当操作开始执行以后的CPU计算能力的分配，如果你不知道这是什么，好吧，你可能根本没必要知道这是什么。

#### 依赖性

根据你应用的复杂度不同，将大任务再分成一系列子任务一般都是很有意义的，而你能通过NSOperation的依赖性实现。

比如说，对于服务器下载并压缩一张图片的整个过程，你可能会将这个整个过程分为两个操作（可能你还会用到这个网络子过程再去下载另一张图片，然后用压缩子过程去压缩磁盘上的图片）。显然图片需要等到下载完成之后才能被调整尺寸，所以我们定义网络子操作是压缩子操作的依赖，通过代码来说就是：

```Objective-C
[resizingOperation addDependency:networkingOperation];
[operationQueue addOperation:networkingOperation];
[operationQueue addOperation:resizingOperation];
```

除非一个操作的依赖的isFinished返回YES，不然这个操作不会开始。时时牢记将所有的依赖关系添加到操作队列很重要!!!

此外，确保不要意外地创建依赖循环，像A依赖B，B又依赖A，这也会导致死锁。

#### completionBlock

每当一个`NSOperation`执行完毕，它就会调用它的`completionBlock`属性一次，这提供了一个非常好的方式让你能在视图控制器(View Controller)里或者模型(Model)里加入自己更多自己的代码逻辑。比如说，你可以在一个网络请求操作的`completionBlock`来处理操作执行完以后从服务器下载下来的数据。

#### 自定义NSOperation

- 如果只重写main方法，底层控制变更任务执行完成状态，以及任务退出。
- 如果重写start方法，自行控制任务状态

自定义`NSOperation`时，思路都是大同小异，大致可以分为一下几步：

创建一个集成自NSOperation的类
重写NSOperation的main()方法，在main()方法中实现耗时操作
然后使用时创建自定义的NNSOperation对象，把它添加到NSOperationQueue中，这样就可以自动执行了
其实，这只是自定义NSOperation的一种方法，而且是比较简单的一种方法，不需要自己去控制NSOperation的完成，取消等。另外一种方式是重写NSOperation的start方法，这种方法就需要你自己去控制NSOperation的完成，取消，执行等，而且有许多需要注意的地方。下面着重介绍一些第二种方法。

第一种方式像下面那样就可以了，执行完耗时操作后，这个operation会被自动`dealloc`

```objc
- (void)main
{
    NSLog(@"%s",__func__);
    [NSThread sleepForTimeInterval:3.0];
    NSLog(@"%@-耗时操作执行结束",[NSThread currentThread]);
}

- (void)dealloc
{
    NSLog(@"%s",__func__);
}

/**
结果：
 -[DMOperation main]
 <NSThread: 0x60c000263dc0>{number = 3, name = (null)}-执行结束
 -[DMOperation dealloc]
*/
```

如果第二种方式就需要自己控制状态

```objc
@interface DMOperation ()
@property (assign, nonatomic, getter = isExecuting) BOOL executing;
@property (assign, nonatomic, getter = isFinished) BOOL finished;
@end
@implementation DMOperation

@synthesize finished = _finished;
@synthesize executing = _executing;
- (void)main
{
    NSLog(@"%s",__func__);
}

- (void)start
{
    NSLog(@"%s",__func__);
    @synchronized(self){
        if (self.isCancelled) {
            self.finished = YES;
            return;
        }
    }
    self.executing = YES;
    [NSThread sleepForTimeInterval:3.0];
    self.executing = NO;
    self.finished = YES;//不讲finished设置为YES 是不会被dealloc的
    NSLog(@"%@-执行结束",[NSThread currentThread]);
}

- (void)setFinished:(BOOL)finished {
    [self willChangeValueForKey:@"isFinished"];
    _finished = finished;
    [self didChangeValueForKey:@"isFinished"];
}

- (void)setExecuting:(BOOL)executing {
    [self willChangeValueForKey:@"isExecuting"];
    _executing = executing;
    [self didChangeValueForKey:@"isExecuting"];
}

- (void)dealloc
{
    NSLog(@"%s",__func__);
}
```

重写了`- (void)start`方法后，`- (void)main`方法不会被调用

`finished` 设置为YES后才会调用`dealloc`,这是只有`finished`后，才会从`NSOperationQueue`队列中移除。

### GCD(Grand Central Dispatch)

#### barrier

`dispatch_barrier_asyn`的作用是将加入并发队列中的任务，分成三部分，保证这三部分是按顺序执行的，每一部分内部可能是无序的。

实例代码如下：

```objc
- (void)dispatch_barrier_1
{
    dispatch_queue_t queue = dispatch_queue_create("com.leoliu.concurrent",DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_async(queue, ^{
        NSLog(@"1");
    });
    dispatch_async(queue, ^{
        sleep(5);
        NSLog(@"2");
    });
    dispatch_async(queue, ^{
        NSLog(@"3");
    });
    
    dispatch_barrier_async(queue, ^{
        NSLog(@"barrier");
        sleep(5);
    });
    
    dispatch_async(queue, ^{
        sleep(1);
        NSLog(@"4");
    });
    dispatch_async(queue, ^{
        sleep(2);
        NSLog(@"5");
    });
    dispatch_async(queue, ^{
        NSLog(@"6");
    });
    dispatch_async(queue, ^{
        NSLog(@"7");
    });
    
    NSLog(@"over");
    /**
     结果：
     1
     3
     2
     barrier
     6
     4
     7
     5
     结论：
     barrier：之前的执行完成之后 才会执行barrier中的代码，barrier中的代码执行完后才会执行后面的代码
     barrier函数之前和之后的操作执行顺序都不固定
     */
}

```

>注意：
>dispatch_barrier_asyn使用时对队列的类型有要求，只能是通过dispatch_queue_create方法创建的并发队列才后有上面提到的功能，否则，相当于普通的dispatch_asyn。对于dispatch_barrier_syn也是一样的


 ##### dispatch_barrier_sync与dispatch_barrier_async
 1、等待在它前面插入队列的任务先执行完
 
 2、等待他们自己的任务执行完再执行后面的任务
 
 不同点：
 
 1、dispatch_barrier_sync将自己的任务插入到队列的时候，需要等待自己的任务结束之后才会继续插入被写在它后面的任务，然后执行它们
 
 2、dispatch_barrier_async将自己的任务插入到队列之后，不会等待自己的任务结束，它会继续把后面的任务插入到队列，然后等待自己的任务结束后才执行后面任务。

###### dispatch_barrier_asyn的应用

利用dispatch_barrier_asyn我们可以实现一个读写安全的模型,这样可以保证写的过程中不能被读，以免数据不对，但是读与读之间并没有任何的冲突

代码如下:

```objc
@interface DMCache()
{
    dispatch_queue_t _queue;
}
@property (nonatomic, strong) NSMutableDictionary *cache;
@end
@implementation DMCache
+ (instancetype)shared
{
    static DMCache *share;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        share = [[DMCache alloc] init];
    });
    
    return share;
}

- (instancetype)init
{
    self = [super init];
    self.cache = [NSMutableDictionary dictionary];
    _queue = dispatch_queue_create("com.cache.concurrent", DISPATCH_QUEUE_CONCURRENT);
    return self;
}

- (id)cacheWithKey:(id)key
{
    __block id obj;
    
    // 同步'读'，立刻返回结果
    dispatch_sync(_queue, ^{
        obj = [self.cache objectForKey:key];
    });
    
    return obj;
}

- (void)setCacheObject:(id)obj withKey:(id)key
{
    // 保证同时'写'的的只有一个
    dispatch_barrier_async(_queue, ^{
        [self.cache setObject:obj forKey:key];
    });
}
```

#### group

 dispatch_group_async 用来阻塞一个线程，直到一个或多个任务完成执行。进行线程同步。

 代码如下：

 ```objc
 - (void)dispatch_group_1
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    // 把 queue 加入到 group
    dispatch_group_async(group, queue, ^{
        // 一些异步操作任务
        sleep(5);
        NSLog(@"%@:group task one",[NSThread currentThread]);
    });
    
    dispatch_group_async(group, queue, ^{
        NSLog(@"%@:group task two",[NSThread currentThread]);
    });
    
    // code 你可以在这里写代码做一些不必等待 group 内任务的操作
    NSLog(@"%@:outer",[NSThread currentThread]);
    // 当你在 group 的任务没有完成的情况下不能做更多的事时，阻塞当前线程等待 group 完工
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    NSLog(@"finish");
    
//        dispatch_group_notify(group, queue, ^{
//            NSLog(@"%@:notify group finish",[NSThread currentThread]);
//        });
    /**
     2018-04-19 17:17:22.706349+0800 DMGCD[349:54561] outer
     2018-04-19 17:17:22.710449+0800 DMGCD[349:54592] group task two
     2018-04-19 17:17:27.715335+0800 DMGCD[349:54590] group task one
     2018-04-19 17:17:27.715601+0800 DMGCD[349:54561] finish
     */
}
 ```

 `dispatch_group_enter`和`dispatch_group_leave`成对出现，可以实现异步任务的同步

 ```objc

- (void)dispatch_group_2
{
    dispatch_queue_t concurrentQueue = dispatch_queue_create("my.concurrent.queue", DISPATCH_QUEUE_SERIAL);
    dispatch_group_t group = dispatch_group_create();
    
    for (int i = 0; i < 10; i++) {
        dispatch_group_enter(group);
        dispatch_async(concurrentQueue, ^{
            if (i == 5) {
                sleep(i+1);
            }
            NSLog(@"%@:task = %tu",[NSThread currentThread],i);
            dispatch_group_leave(group);
        });
    }
    
    NSLog(@"outter");
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        NSLog(@"finished");
    });
    
    /**
      outter
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 0
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 1
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 2
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 3
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 4
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 5
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 6
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 7
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 8
      <NSThread: 0x17026ce80>{number = 4, name = (null)}:task = 9
      finished
     */
    
}
 ```

 #### dispatch_apply

 `dispatch_apply`函数是`dispatch_sync`函数和Dispatch Group的关联API,该函数按指定的次数将指定的Block追加到指定的Dispatch Queue中,并等到全部的处理执行结束

 ```objc
 - (void)dispatch_apply_1
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    NSArray *arr = @[@"a",@"b",@"c",@"d",@"e",@"f",@"g",@"h",@"i",@"j"];
    dispatch_apply(arr.count, queue, ^(size_t i) {
            NSLog(@"%@:%@",[arr objectAtIndex:i],[NSThread currentThread]);
    });
    NSLog(@"--over");
    /**
      a:<NSThread: 0x170067580>{number = 1, name = main}
      b:<NSThread: 0x170067580>{number = 1, name = main}
      c:<NSThread: 0x170067580>{number = 1, name = main}
      d:<NSThread: 0x170067580>{number = 1, name = main}
      e:<NSThread: 0x170067580>{number = 1, name = main}
      f:<NSThread: 0x170067580>{number = 1, name = main}
      g:<NSThread: 0x170067580>{number = 1, name = main}
      h:<NSThread: 0x170067580>{number = 1, name = main}
      i:<NSThread: 0x170067580>{number = 1, name = main}
      j:<NSThread: 0x170067580>{number = 1, name = main}
      --over
     */
}
 ```

 #### semaphore

 关于信号量，一般可以用停车来比喻：

 停车场剩余4个车位，那么即使同时来了四辆车也能停的下。如果此时来了五辆车，那么就有一辆需要等待。
 信号量的值就相当于剩余车位的数目，`dispatch_semaphore_wait`函数就相当于来了一辆车，`dispatch_semaphore_signal`
 就相当于走了一辆车。停车位的剩余数目在初始化的时候就已经指明了（`dispatch_semaphore_create（long value）`），
 调用一次`dispatch_semaphore_signal`，剩余的车位就增加一个；调用一次`dispatch_semaphore_wait`剩余车位就减少一个；
 当剩余车位为0时，再来车（即调用`dispatch_semaphore_wait`）就只能等待。有可能同时有几辆车等待一个停车位。有些车主
 没有耐心，给自己设定了一段等待时间，这段时间内等不到停车位就走了，如果等到了就开进去停车。而有些车主就像把车停在这，
 所以就一直等下去。

 可以使用`semaphore`进行线程同步,将异步操作转换为同步操作

```objc
- (void)dispatch_semaphore_1
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    
    dispatch_async(queue, ^{
        NSLog(@"请求中……");
        sleep(5);
        NSLog(@"请求回调中……");
        dispatch_semaphore_signal(semaphore);
    });
    
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"over");
    /**
      DMGCD[388:60678] 请求中……
      DMGCD[388:60678] 请求回调中……
      DMGCD[388:60602] over
     */
}
```

也可以加锁

```objc
//为线程加锁
- (void)dispatch_semaphore_2
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(1);
    for (int i = 0; i < 100; i++) {
        dispatch_async(queue, ^{
            long semaphorecount = dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
            NSLog(@"i = %tu, count = %ld",i,semaphorecount);
            dispatch_semaphore_signal(semaphore);
        });
    }
    
    /**
     当线程1执行到dispatch_semaphore_wait这一行时，semaphore的信号量为1，所以使信号量-1变为0，并且线程1继续往下执行；如果当在线程1NSLog这一行代码还没执行完的时候，又有线程2来访问
     执行dispatch_semaphore_wait时由于此时信号量为0，且时间为DISPATCH_TIME_FOREVER,所以会一直阻塞线程2（此时线程2处于等待状态），直到线程1执行完NSLog并执行
     dispatch_semaphore_signal使信号量为1后，线程2才能解除阻塞继续住下执行。以上可以保证同时只有一个线程执行NSLog这一行代码。
     */
}
```

还可以控制线程并发数

```objc
//使用 Dispatch Semaphore 控制并发线程数量
- (void)dispatch_semaphore_3
{
    dispatch_queue_t queue = dispatch_queue_create("com.limit.queue", DISPATCH_QUEUE_CONCURRENT);
    for (int i = 0; i < 1000; i++) {
        dispatch_async_limit(queue, 5, ^{
            NSLog(@"%tu = %@",i,[NSThread currentThread]);
        });
    }
}

void dispatch_async_limit(dispatch_queue_t queue,NSUInteger limitSemaphoreCount, dispatch_block_t block) {
    //控制并发数的信号量
    static dispatch_semaphore_t limitSemaphore;
    
    //专门控制并发等待的线程
    static dispatch_queue_t receiverQueue;
    
    //使用 dispatch_once而非 lazy 模式，防止可能的多线程抢占问题
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        limitSemaphore = dispatch_semaphore_create(limitSemaphoreCount);
        receiverQueue = dispatch_queue_create("receiver", DISPATCH_QUEUE_SERIAL);
    });
    // 如不加 receiverQueue 放在主线程会阻塞主线程
    dispatch_async(receiverQueue, ^{
        //可用信号量后才能继续，否则等待
        dispatch_semaphore_wait(limitSemaphore, DISPATCH_TIME_FOREVER);
        dispatch_async(queue, ^{
            !block ? : block();
            //在该工作线程执行完成后释放信号量
            dispatch_semaphore_signal(limitSemaphore);
        });
    });
}

```

 #### Dispatch Source

`Dispatch Source` 是BSD系内核惯有功能`kqueue`的包装。`kqueue` 是在 XNU 内核中发生各种事件时，在应用程序编程方执行处理的技术。其 CPU 负荷非常小，尽量不占用资源。kqueue 可以说是应用程序处理 XNU 内核中发生的各种事件的方法中最优秀的一种。

`Dispatch Source` 也使用在了 Core Foundation 框架的用于异步网络的API CFSocket 中。因为Foundation 框架的异步网络 API 是通过`CFSocket`实现的，所以可享受到仅使用 Foundation 框架的 `Dispatch Source` 带来的好处。

那么优势何在？使用的 `Dispatch Source` 而不使用 `dispatch_async` 的唯一原因就是利用`联结`的优势。

##### 创建dispatch_source_t

```objc
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());
```

参数:

参数|意义
--------|--------
type	|dispatch源可处理的事件类型
handle	|可以理解为索引、id或句柄，假如要监听进程，需要传入进程的ID
mask	|可以理解为描述，提供更详细的描述，让它知道具体要监听什么
queue	|自定义源需要的一个队列，用来处理所有的响应句柄（block）


Dispatch Source可处理的所有事件:

名称	|意义
--------------------------------|--------------------------------
DISPATCH_SOURCE_TYPE_DATA_ADD	|自定义的事件，变量增加
DISPATCH_SOURCE_TYPE_DATA_OR	|自定义的事件，变量OR
DISPATCH_SOURCE_TYPE_MACH_SEND	|MACH端口发送
DISPATCH_SOURCE_TYPE_MACH_RECV	|MACH端口接收
DISPATCH_SOURCE_TYPE_PROC	    |进程监听,如进程的退出、创建一个或更多的子线程、进程收到UNIX信号
DISPATCH_SOURCE_TYPE_READ	    |IO操作，如对文件的操作、socket操作的读响应
DISPATCH_SOURCE_TYPE_SIGNAL	    |接收到UNIX信号时响应
DISPATCH_SOURCE_TYPE_TIMER	    |定时器
DISPATCH_SOURCE_TYPE_VNODE	    |文件状态监听，文件被删除、移动、重命名
DISPATCH_SOURCE_TYPE_WRITE	    |IO操作，如对文件的操作、socket操作的写响应
DISPATCH_SOURCE_TYPE_MEMORYPRESSURE|内存压力监听

##### 自定义事件

上面类型中可以使用自定时间的类型只有`DISPATCH_SOURCE_TYPE_DATA_ADD`和`DISPATCH_SOURCE_TYPE_DATA_OR`

实例：

当我们更新进度条时，可能在多个线程上同时做很多任务，每个任务完成后，刷新界面，更新一点进度条的进度，因为每个任务都更新一次进度条，造成界面刷新次数太多，可能会导致界面卡顿，所以此时利用Dispatch Source能很好的解决这种情况，因为Dispatch Source在刷新太频繁的时候会自动联结起来，下面就用代码实现一下这个场景。

```objc

- (void)dispatch_source_5
{
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, dispatch_get_main_queue());
    //在主线程繁忙的时候将操作联结起来，等主线程空闲时 刷新UI 避免频繁的在主线程刷新UI
    dispatch_source_set_event_handler(source, ^{
        //当处理事件被最终执行时，计算后的数据可以通过dispatch_source_get_data来获取。这个数据的值在每次响应事件执行后会被重置，所以totalComplete的值是最终累积的值。
        NSUInteger value = dispatch_source_get_data(source);
        
        NSLog(@"value：%@", @(value));
        value = value == 100 ? 0 : value;
        self.progressView.progress = value/10.;
        
        NSLog(@"线程号：%@", [NSThread currentThread]);
    });
    
    dispatch_resume(source);
    
    self.progressView.progress = 1.0;
    __block NSInteger timeout = 10;
    dispatch_queue_t queue = dispatch_queue_create("com.yiqi.dmgcd", DISPATCH_QUEUE_SERIAL);
    dispatch_source_t timerSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    dispatch_time_t time = dispatch_walltime(NULL, 0);
    dispatch_source_set_timer(timerSource, time, 1.0*NSEC_PER_SEC, 0);
    
    dispatch_source_set_event_handler(timerSource, ^{
        if (timeout <= 0) {
            dispatch_source_cancel(timerSource);
            NSLog(@"cancel");
        } else {
            timeout--;
            NSLog(@"timeout = %tu  %f",timeout,timeout/10.);
            NSInteger a = timeout == 0 ? 100 : timeout;
            dispatch_source_merge_data(source,  a);//timeout == 0 时不会触发事件
        }
    });
    
    //开始执行
    dispatch_resume(timerSource);
}

```

联结方式为OR时

```objc
- (void)dispatch_source_6
{
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_OR, 0, 0, dispatch_get_main_queue());
    dispatch_source_set_event_handler(source, ^{
         NSLog(@"监听函数：%lu",dispatch_source_get_data(source));
    });
    dispatch_resume(source);
    
    for (int i = 1; i < 5 ; i++) {
        dispatch_source_merge_data(source, i);
        usleep(2000);
    }
}
```

`DISPATCH_SOURCE_TYPE_DATA_ADD` 与 `DISPATCH_SOURCE_TYPE_DATA_OR` 的区别：

DISPATCH_SOURCE_TYPE_DATA_ADD 带别的方式为将`dispatch_source_merge_data`联结进来的数据进行加运算

DISPATCH_SOURCE_TYPE_DATA_OR 带别的方式为将`dispatch_source_merge_data`联结进来的数据进行或运算

比如上面的那个例子，先将`usleep(2000);`这段代码注释掉以后，通过`DISPATCH_SOURCE_TYPE_DATA_ADD`方式运行的到的结果会是 10（1+2+3+4），通过`DISPATCH_SOURCE_TYPE_DATA_OR`方式运行得到的结果是7 （1|2|3|4）

如果将这段代码`usleep(2000);`放开，不管是用ADD还是OR会打印四次结果为下面所示,这正是联结的作用

```
监听函数：1
监听函数：2
监听函数：3
监听函数：4
```

##### DISPATCH_SOURCE_TYPE_VNODE

通过这种类型，可以监听文件状态

状态类型|状态
--|--
DISPATCH_VNODE_DELETE	|文件被删除
DISPATCH_VNODE_WRITE	|文件数据发生变化
DISPATCH_VNODE_EXTEND	|文件size发生变化
DISPATCH_VNODE_ATTRIB	|文件metadata发生变化
DISPATCH_VNODE_LINK		|文件链接数发生变化
DISPATCH_VNODE_RENAME	|文件被重命名
DISPATCH_VNODE_REVOKE	|文件被revoked
DISPATCH_VNODE_FUNLOCK	|文件被unlocked

通过这个属性我们可以实现一个监听指定目录下文件内容是否发生变化，如果变化，我们可以来做一些操作,这里有一个我写工具[LLDirWatchDog](https://github.com/leoliuyt/LLDirWatchDog)欢迎star

```objc

- (void)start {
    NSString* docPath = self.path;
    if (!docPath) return;
    
    int dirFD = open([docPath fileSystemRepresentation], O_EVTONLY);
    if (dirFD < 0) {return;}
    
    _dirQueueSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE, dirFD, DISPATCH_VNODE_WRITE, dispatch_get_main_queue());
    
    if (!_dirQueueSource)
    {
        close(dirFD);
    }
    _dirFD = dirFD;
    __weak typeof(self) weakSelf = self;
    dispatch_source_set_event_handler(_dirQueueSource, ^{
        __strong typeof(weakSelf) strongSelf = weakSelf;
        if (strongSelf.update) {
            strongSelf.update();
        }
    });
    dispatch_source_set_cancel_handler(_dirQueueSource, ^{close(dirFD);});
    dispatch_resume(_dirQueueSource);
}
```


##### DISPATCH_SOURCE_TYPE_MEMORYPRESSURE

```objc
//内存压力情况变化.
- (void)dispatch_source_3
{
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_MEMORYPRESSURE, 0, DISPATCH_MEMORYPRESSURE_WARN|DISPATCH_MEMORYPRESSURE_CRITICAL, dispatch_get_main_queue());
    dispatch_source_set_event_handler(source, ^{
        dispatch_source_memorypressure_flags_t pressureLevel = dispatch_source_get_data(source);
        if (pressureLevel & DISPATCH_MEMORYPRESSURE_WARN) {
            NSLog(@"memeory warn");
        }
        
        if (pressureLevel & DISPATCH_MEMORYPRESSURE_CRITICAL) {
            NSLog(@"memeory critical");
        }
    });
    dispatch_resume(source);
}
```
## 多线程与锁

iOS中锁的类型：

- @synchronized
- atomic(保证赋值操作)
- OSSpinLock（自旋锁）
    - 循环等待询问，不释放当前资源
    - 用于轻量级数据访问，简单的int值+/-1的操作

- NSLock
- NSRecursiveLock（递归锁）
- dispatch_semaphore_t(信号量)

```Objective-C
- (void)methodA
{
    [lock lock];
    [self methodB];
    [lock unlock];
}

- (void)methodB
{
    [lock lock];
    //其他逻辑
    [lock unlock];
}
```

上面代码会死锁，可用递归锁解决

## QA

看下面代码的是否有问题及执行结果,然后分析结果

Q1:

```Objective-C
- (void)demo//task1
{
    NSLog(@"outer--1");
    dispatch_sync(dispatch_get_main_queue(), ^{//task2
       NSLog(@"inner--1");
    });
    NSLog(@"outer--2");
}
```
结果： 由于队列中任务的循环等待引起的死锁

分析：

```
task1是在主队列中一个任务，被派发在主线程中执行，而task2也被加入主队列，
由于主队列是一个串行队列，而串行队列的特点是一个任务结束才会开始下一个任务，现在有两个任务，task1中的任务要等待task2完成，而task2需要等待task1，最终导致死锁。
```

Q2:
```Objective-C
- (void)demo{//task1
    dispatch_queue_t q = dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL);
    NSLog(@"outer--1");
    dispatch_sync(q, ^{//task2
        NSLog(@"inner--1");
    });
    NSLog(@"outer--2");
}
```

结果：

```
outer--1
inner--1
outer--2
```

分析：

```
task1 和 task2 分别在主队列和serial串行队列中，不存在队列中任务的循环等待的问题
```

Q3:
```Objective-C
- (void)demo{
    dispatch_queue_t q = dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL);
    NSLog(@"outer--1");
    dispatch_async(q, ^{//开辟了一条子线程 task1
        NSLog(@"inner--1--%@",[NSThread currentThread]);
        dispatch_sync(q, ^{//任务提交到串行队列中，并在当前子线程中执行 task2
            NSLog(@"inner--2");
        });
    });
    NSLog(@"outer--2");
}
```

结果：由于队列中任务的循环等待引起的死锁

分析：

```
串行队列中加入task1
在task1中又有一个任务task2被添加到了串行队列中，并要求在队列中同步执行，
任务二的完成要等待串行队列任务一task1的完成
而任务task1的完成要等待task2的完成
```
Q4:

```Objective-C
- (void)demo{
    dispatch_queue_t q = dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL);
    NSLog(@"outer--1");
    dispatch_sync(q, ^{//主线程 task1
        NSLog(@"inner--1--%@",[NSThread currentThread]);
        dispatch_async(q, ^{//子线程 task2
//            sleep(5);
            NSLog(@"inner---inner-2");
        });
        sleep(2);
        NSLog(@"inner--2--%@",[NSThread currentThread]);
    });
    
    sleep(2);
    NSLog(@"outer--2");
}

```

结果：

```
outer--1
inner--1--<NSThread: 0x17007c440>{number = 1, name = main}
inner--2--<NSThread: 0x17007c440>{number = 1, name = main}
inner---inner-2
outer--2

//如果不sleep(2) 那么结果为下面，理论上最后两项的顺序应该是不一定的
outer--1
inner--1--<NSThread: 0x17007c440>{number = 1, name = main}
inner--2--<NSThread: 0x17007c440>{number = 1, name = main}
outer--2
inner---inner-2
```

分析：

```
task1加入串行队列
task1中的task2也加入了队列中
因为task1中的task2任务是在另一条子线程中完成的，
task1不需要等待task2的完成就可以完成，
所以task1完成后出队列，然后task2就可以执行
```
Q5:

```Objectice-C
- (void)demo{
    dispatch_queue_t q = dispatch_queue_create("CONCURRENT", DISPATCH_QUEUE_CONCURRENT);
    NSLog(@"outer--1");
    dispatch_sync(q, ^{//task1
        NSLog(@"inner--1");
        dispatch_sync(q, ^{//task2
            sleep(5);
            NSLog(@"inner--inner--2");
        });
        NSLog(@"inner--2");
    });
    
    NSLog(@"outer--2");
}
```

结果：

```
outer--1
inner--1
inner--inner--2
inner--2
outer--2
```

分析：

```
task1 任务被添加到并行队列中后，开始执行
执行到task2时，把task2后添加到并行队列中，
由于task2是同步执行，所以task2中的任务开始执行
task2执行完毕后，task1后续的任务开始执行
```

Q6:
```Objective-C
- (void)demo
{
    dispatch_queue_t q = dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL);
    dispatch_async(q, ^{
        NSLog(@"1");
        [self performSelector:@selector(printLog) withObject:nil afterDelay:0];
        NSLog(@"3");
    });
    
    /*
     GCD维持的线程池 其中的线程没有开启runloop，
     performSelector 方法调用所在的当前的线程，必须开启runloop
     */
}
与demo1效果一样
- (void)demo1
{
    __weak typeof(self) weakSelf = self;
    [NSThread detachNewThreadWithBlock:^{
        NSLog(@"1:%@",[NSThread currentThread]);
        [weakSelf performSelector:@selector(printLog) withObject:nil afterDelay:0];
        NSLog(@"3:%@",[NSThread currentThread]);
    }];
}

- (void)printLog
{
    NSLog(@"2");
}
```

结果 

```
1
3
```

分析：

```
1、performSelector 调用该方法所在的线程，必须开启runloop,否则不会执行
2、GCD维持的线程池 其中的线程没有开启runloop，
```

## 参考
[Parse的底层多线程处理思路](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse的多线程处理思路/Parse的底层多线程处理思路.md#parse-ios-sdk介绍)

[IOS dispatch source 学习篇](https://www.jianshu.com/p/2f6eed90bb9d)

[Dispatch Source的自定义事件](https://www.jianshu.com/p/89943e91bee9)

[IOS Dispatch Source 笔记](https://www.jianshu.com/p/83c3b7e4af28)

[iOS多线程：『NSOperation、NSOperationQueue』详尽总结](https://www.jianshu.com/p/4b1d77054b35)

[NSHipster-NSOperation](http://nshipster.cn/nsoperation/)

[iOS多线程篇：NSThread](http://www.cocoachina.com/ios/20160412/15903.html)