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

### NSOperation/NSOperationQueue

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
    
    // 任意线程都可以'读'
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


##参考
[Parse的底层多线程处理思路](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse的多线程处理思路/Parse的底层多线程处理思路.md#parse-ios-sdk介绍)