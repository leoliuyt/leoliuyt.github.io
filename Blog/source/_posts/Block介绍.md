---
title: Block介绍
date: 2018-05-21 21:10:38
tags: runtime
categories: Objective-C
---

## Block介绍

Block是将函数及其执行上下文封装起来的对象。

下面代码的改写过程，大体能够体现出，Block是如何实现的

```
//相当于block中用到的变量
int varId = 1;

//相当于block的实现主体
void detail_func(int event){
    printf("varId:%d event = %d\n",varId,event);
}
```
改写成block形式如下：

```
int varId = 1;
void(^detail_func)(int) = ^(int event){
    printf("varId:%d event = %d\n",varId,event);
}
```
在改写为对象形式如下：

```Objective-c
@interface CallbackObject:NSObject
{
    int _varId;
}
@end

@implementation CallbackObject:NSObject
- (id)initWithVarId:(int)varId
{
    self = [super init];
    _varId = varId;
    return self;
}

- (void)callback:(int)event
{
    NSLog(@"varId:%d event = %d\n",_varId,event");
}
@end
```

之后这步的改写，就类似Block在内部为我们所做的操作：将函数及上下文封装成对象。

那么Block真是的内部实现是怎样的呢？我来通过clang来将源码编译成cpp文件后，来窥探一下Block的内部实现。

### 源码分析

```Objective-C
@implementation DMBlock1

- (void)dm_method
{
    void(^MyBlock)(void) = ^(){
        NSLog(@"my block");
    };

    MyBlock();
}
@end

```

上面这段源码通过`clang -rewrite-objc DMBlock1.m`执行后生成如下cpp文件

```CPP

struct __block_impl {
  void *isa;//isa指针，Block是对象的标志
  int Flags;
  int Reserved;//今后版本升级所需的区域
  void *FuncPtr;//函数指针
};

//Block结构体
struct __DMBlock1__dm_method_block_impl_0 {
  struct __block_impl impl;
  struct __DMBlock1__dm_method_block_desc_0* Desc;
  
  //结构体的构造函数
  __DMBlock1__dm_method_block_impl_0(void *fp, struct __DMBlock1__dm_method_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;//栈
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

//对应Block中的执行体
static void __DMBlock1__dm_method_block_func_0(struct __DMBlock1__dm_method_block_impl_0 *__cself) {

        NSLog((NSString *)&__NSConstantStringImpl__var_folders_6__qmlfhwd97q1dy9pv8qpqrkth0000gn_T_DMBlock1_56af0a_mi_0);
    }

// block内存大小
static struct __DMBlock1__dm_method_block_desc_0 {
  size_t reserved;
  size_t Block_size; //block大小
} __DMBlock1__dm_method_block_desc_0_DATA = { 0, sizeof(struct __DMBlock1__dm_method_block_impl_0)};

//与- (void)dm_method{}对应
static void _I_DMBlock1_dm_method(DMBlock1 * self, SEL _cmd) {
    /*
    void(^MyBlock)(void) = ^(){
        NSLog(@"my block");
    };

    简洁写法
    struct __DMBlock1__dm_method_block_impl_0 tmp = __DMBlock1__dm_method_block_impl_0(__DMBlock1__dm_method_block_func_0,&__DMBlock1__dm_method_block_desc_0_DATA)
    struct __DMBlock1__dm_method_block_impl_0 *MyBlock = &tmp
    */
    void(*MyBlock)(void) = ((void (*)())&__DMBlock1__dm_method_block_impl_0((void *)__DMBlock1__dm_method_block_func_0, &__DMBlock1__dm_method_block_desc_0_DATA));

    // MyBlock();
    /*
    简洁写法
    (*MyBlock->FuncPtr)(MyBlock);
    Block的调用实际就是函数的调用
    */
    ((void (*)(__block_impl *))((__block_impl *)MyBlock)->FuncPtr)((__block_impl *)MyBlock);
}
```
分析代码前，先说一下，编译器的命名规则:类名+方法名+block_impl或block_func或block_des+出现的顺序

下面开始逐一分析源码
首先看一下源Block代码块中的
```
^(){
    NSLog(@"my block");
};
```
被编译成了

```cpp
static void __DMBlock1__dm_method_block_func_0(struct __DMBlock1__dm_method_block_impl_0 *__cself) {
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_6__qmlfhwd97q1dy9pv8qpqrkth0000gn_T_DMBlock1_56af0a_mi_0);
}
```
这个函数有一个参数`__cself`，相当于Objective-C中的self,为指向Block值的变量。

```
- (void)dm_method{}
```
相应的被编译成了
```
static void _I_DMBlock1_dm_method(DMBlock1 * self, SEL _cmd) {}
```
block的显示最终变成了在栈上生成的`__DMBlock1__dm_method_block_impl_0`结构体实例的指针，这个结构体的构造函数的第一参数，就是上面的那个函数指针，第二参数就是`__DMBlock1__dm_method_block_desc_0_DATA`

再看block调用部分，被编译成
```
 (*MyBlock->FuncPtr)(MyBlock);
```

**Block的调用就是直接的函数调用。**

最后看一下，直接可以证明**block即是对象**的结构体

```
struct __block_impl {
  void *isa;//isa指针，Block是对象的标志
  int Flags;
  int Reserved;//今后版本升级所需的区域
  void *FuncPtr;//函数指针
};
```

这里`__DMBlock1__dm_method_block_impl_0`结构体相当于基于`objc_object`结构体的OC类对象的结构体。另外，对其中的成员变量isa的初始化如下:

```
isa = &_NSConcreteStackBlock;//栈
```
`_NSConcreteStackBlock`相当于`class_t`结构体实例。在将Block作为对象处理时，关于该类的信息放置于`_NSConcreteStackBlock`中。

>总结
>Block是将函数及其执行上下文封装起来的对象。
>Block的调用就是函数调用。

## Block的截获变量

- 局部变量
    - 基本数据类型  截获其值
    - 对象类型      连同对象所有权修饰符一同截获
- 静态局部变量  以指针形式截获
- 全局变量     不截获
- 全局静态变量  不截获

源码分析

```Objective-C
//全局变量
int global_var = 4;
//静态全局变量
static int static_global_var = 5;
- (void)dm_method{
    //基本数据类型的局部变量
    int var = 1;
    
    //对象类型的局部变量
    __unsafe_unretained id unsafe_obj = nil;
    __strong id strong_obj = nil;
    
    //局部静态变量
    static int static_var = 3;
    void(^Block)(void) = ^{
        NSLog(@"局部变量<基本数据类型> var %d",var);
        NSLog(@"局部变量<__unsafe_unretained 对象类型> var %@",unsafe_obj);
        NSLog(@"局部变量<__strong 对象类型> var %@",strong_obj);
        NSLog(@"静态变量 %d",static_var);
        
        NSLog(@"全局变量 %d",global_var);
        
        NSLog(@"静态全局变量 %d",static_global_var);
    };
    
    Block();
}
```

ARC `-fobjc-arc`

MRC `-fno-objc-arc`

上面这段源码通过`clang -rewrite-objc -fobjc-arc DMBlock1.m`执行后生成如下cpp文件

```
int global_var = 4;

static int static_global_var = 5;

struct __DMBlock__dm_method_block_impl_0 {
  struct __block_impl impl;
  struct __DMBlock__dm_method_block_desc_0* Desc;
  //截获局部变量的值
  int var;
    //连同所有权修饰符一起截活
  __unsafe_unretained id unsafe_obj;
  __strong id strong_obj;
  // 以指针形式截获局部变量
  int *static_var;
  //对全局变量、静态全局变量不截获
    
  __DMBlock__dm_method_block_impl_0(void *fp, struct __DMBlock__dm_method_block_desc_0 *desc, int _var, __unsafe_unretained id _unsafe_obj, __strong id _strong_obj, int *_static_var, int flags=0) : var(_var), unsafe_obj(_unsafe_obj), strong_obj(_strong_obj), static_var(_static_var) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```
Block的截获变量只针对Block中使用的局部变量。

被截获的变量，变成了`__DMBlock__dm_method_block_impl_0`结构体的成员变量


>注意:
>C语言数组的无法实现截获变量,如需使用，需要改为指针形式。

## __block修饰符

__block修饰符一般使用在对截获变量进行赋值操作时。

- 需要添加__block
    - 基本数据类型
    - 对象类型
- 不需要需要添加__block
    - 静态局部变量
    - 全局变量
    - 静态全局变量
源码分析：

```Objective-C
@implementation DMBlock2
- (void)dm_method
{
    __block int multiplier = 4;
    int(^MyBlock)(int) = ^int(int num){
        return num * multiplier;
    };
    MyBlock(2);
}
@end
```

编译后：

```cpp
struct __Block_byref_multiplier_0 {
  void *__isa;
__Block_byref_multiplier_0 *__forwarding;
 int __flags;
 int __size;
 int multiplier;
};

static void _I_DMBlock2_dm_method(DMBlock2 * self, SEL _cmd) {
    __attribute__((__blocks__(byref))) __Block_byref_multiplier_0 multiplier = {(void*)0,(__Block_byref_multiplier_0 *)&multiplier, 0, sizeof(__Block_byref_multiplier_0), 4};

    int(*MyBlock)(int) = ((int (*)(int))&__DMBlock2__dm_method_block_impl_0((void *)__DMBlock2__dm_method_block_func_0, &__DMBlock2__dm_method_block_desc_0_DATA, (__Block_byref_multiplier_0 *)&multiplier, 570425344));

    ((int (*)(__block_impl *, int))((__block_impl *)MyBlock)->FuncPtr)((__block_impl *)MyBlock, 2);
}

static void __DMBlock2__dm_method_block_copy_0(struct __DMBlock2__dm_method_block_impl_0*dst, struct __DMBlock2__dm_method_block_impl_0*src) {_Block_object_assign((void*)&dst->multiplier, (void*)src->multiplier, 8/*BLOCK_FIELD_IS_BYREF*/);}

static void __DMBlock2__dm_method_block_dispose_0(struct __DMBlock2__dm_method_block_impl_0*src) {_Block_object_dispose((void*)src->multiplier, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __DMBlock2__dm_method_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __DMBlock2__dm_method_block_impl_0*, struct __DMBlock2__dm_method_block_impl_0*);
  void (*dispose)(struct __DMBlock2__dm_method_block_impl_0*);
} __DMBlock2__dm_method_block_desc_0_DATA = { 0, sizeof(struct __DMBlock2__dm_method_block_impl_0), __DMBlock2__dm_method_block_copy_0, __DMBlock2__dm_method_block_dispose_0};

static int __DMBlock2__dm_method_block_func_0(struct __DMBlock2__dm_method_block_impl_0 *__cself, int num) {
  __Block_byref_multiplier_0 *multiplier = __cself->multiplier; // bound by ref

        return num * (multiplier->__forwarding->multiplier);
    }

```

**__block 修饰的变量变成了对象**

Block与__block变量的实质

名称|实质
---|---
Block|栈上Block的结构体实例（对象）
__block变量|栈上__block变量的结构体实例（对象）

只要Block不截获局部变量，就可以将Block用结构体实例设置在程序的数据区（_NSConcreteGlobalBlock）

虽然通过clang转化的源代码通常是`_NSConcreteStackBlock`类对象，但实际上是这样的:

- 声明的全局block变量属于`_NSConcreteGlobalBlock`
- Block语法中的表达式中没有截获变量时，该Block属于`_NSConcreteGlobalBlock`

在ARC有效的情况下，大多数情形下编译器会恰当的进行判断，自动生成将Block从栈上复制到堆上的代码。

```
typedef int (^blk_t)(int);
blk_t func(int rate){
    blk_t bk = ^(int count){
        return rate * count;
    };
    return bk;
}
```
该源码为返回配置在栈上的block函数。即程序执行中从该函数返回函数调用方时变量作用域结束，因此栈上的block被废弃。但该源码通过对应ARC的编译器转化如下:

```
blk_t func(int rate)
{
    blk_t tmp = &__func_block_impl_0(__func_block_func_0,&__func_block_desc_0_DATA,rate);

    //实际上回调用_Block_copy(tmp)
    //将栈上的block复制到堆上
    //复制后，将堆上的地址作为指针复制给变量tmp   
    tmp = objc_retainBlock(tmp);

    //将堆上的block注册到autoreleasepool中，然后返回该对象
    return objc_autoreleaseReturnValue(tmp);
}
```

其中`objc_retainBlock()`中调用了`_Block_copy(tmp)`这个方法会将在栈上的block复制到堆上。

编译器不会为我们自动添加的情况:

- 向方法或函数的参数中传递Block时

编译器会为我们自动添加的情况:

- Cocoa框架的方法且方法名中含有usingBlock等时
- GCD的API

会将栈中Block复制到堆上的操作:
- 对block调用copy
- Block作为函数返回值时
- 将Block赋值给__strong修饰的变量时
- 方法名中包括usingBlock的Cocoa框架方法或GCD的API中传递Block时


`__forwarding`可以实现无论`__block`变量配置在栈上还是堆上都可以正确访问`__block`变量。

## Block的内存管理

Block的类型

类|对象的存储区域|copy后的结果
---|---|---
_NSConcreteStackBlock|栈|堆
_NSConcreteGlobalBlock|程序的数据区（.data区）|什么都不做
_NSConcreteMallocBlock|堆|增加引用计数


block 从栈复制到堆时对__block变量产生的影响

__block变量的配置存储区域|Block从栈复制到堆时的影响
---|---
栈|从栈复制到堆并被Block持有
堆|被Block持有


调用copy函数和dispose函数的时机

函数|调用时机
copy函数|栈上的block复制到堆上时
dispose函数|堆上的block被废弃时

{% asset_img block_forwarding __block中的forwarding %}

## Block的循环引用

```Objective-C
- (void)dm_methodF
{
    int multipler = 1;
    int(^Block)(int) = ^int(int num){
        NSLog(@"%@",self.isAssign);
        return multipler * num;
    };
    
    self.copyBlock = Block;
}
```

Block被self持有，而Block中又持有self

{% asset_img block_circle_referance block的循环引用 %}

可以通过添加下面的代码解除循环
```
__weak typeof(self)weakSelf = self;
```
{% asset_img block_circle_reference_break block的循环引用破解 %}

```Objective-C
- (void)dm_methodG
{
    __block id tmp = self;
    int multipler = 1;
    int(^Block)(int) = ^int(int num){
        NSLog(@"%@",tmp);
        return multipler * num;
    };
    
    self.copyBlock = Block;
}
```

上面那段代码在MRC下不会产生循环引用，这是因为在MRC下Block不会对__block强引用
在ARC下会产生循环引用

{% asset_img block_referance block的循环引用 %}

可以通过以下方法解决循环引用

```Objective-C
int(^Block)(int) = ^int(int num){
        NSLog(@"%@",tmp);
        tmp = nil;
        return multipler * num;
    };
```

弊端如果block不调用那么循环引用就会一直存在

## QA

Q:

```Objective-C
- (void)dm_methodA{
    int multipler = 5;
    int(^Block)(int) = ^int(int num){
        return multipler * num;
    };
    multipler = 4;
    int result = Block(2);//10
    NSLog(@"%d",result);
}
```

Q:

```Objective-C
- (void)dm_methodC{
    __block int multipler = 5;
    int(^Block)(int) = ^int(int num){
        return multipler * num;
    };
    multipler = 4;
    int result = Block(2);//8
    NSLog(@"%d",result);
}

```

Q:

```Objective-C
- (void)dm_methodE
{
    __block int multipler = 1;
    int(^Block)(int) = ^int(int num){
        return multipler * num;
    };
    
    multipler = 2;
    int result1 = Block(1);//2
    NSLog(@"result1 = %d",result1);
    
//    _aBlock = Block;
    self.copyBlock = Block;
    
    multipler = 3;
    int result2 = Block(1);//3
    NSLog(@"result2 = %d",result2);
    
    int result3 = self.copyBlock(1);//3
    NSLog(@"result3 = %d",result3);
}
```

Q:

```Objective-C
- (void)dm_methodI
{
    id obj = [self getBlockArray];
    typedef void(^blk_t)(void);
    blk_t blk = (blk_t)[obj objectAtIndex:0];
    blk();
}

- (id)getBlockArray
{
    int val = 10;
    return [[NSArray alloc] initWithObjects:
            ^{NSLog(@"block0:%tu",val);},
            ^{NSLog(@"block1:%tu",val);}, nil];
}
```

## 参考
[clang](https://clang.llvm.org/docs/)