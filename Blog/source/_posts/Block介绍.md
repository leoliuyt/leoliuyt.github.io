---
title: Block介绍
date: 2018-05-21 21:10:38
tags: runtime
categories: Objective-C
---

## Block介绍

Block是将函数及其执行上下文封装起来的对象。

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
  int Reserved;
  void *FuncPtr;
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

static struct __DMBlock1__dm_method_block_desc_0 {
  size_t reserved;
  size_t Block_size;
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
从上面源码可以看出

Block是将函数及其执行上下文封装起来的对象。
Block的调用就是函数调用。

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

思考下段代码输出的结果


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

static int __DMBlock2__dm_method_block_func_0(struct __DMBlock2__dm_method_block_impl_0 *__cself, int num) {
  __Block_byref_multiplier_0 *multiplier = __cself->multiplier; // bound by ref

        return num * (multiplier->__forwarding->multiplier);
    }

```

__block 修饰的变量变成了对象

Block与__block变量的实质
名称|实质
---|---
Block|栈上Block的结构体实例（对象）
__block变量|栈上__block变量的结构体实例（对象）

## Block的内存管理

Block的类型

类|对象的存储区域|copy后的结果
---|---|---
_NSConcreteStackBlock|栈|堆
_NSConcreteGlobalBlock|程序的数据区（.data区）|什么都不做
_NSConcreteMallocBlock|堆|增加引用计数

`__forwarding`可以实现无论__block变量配置在栈上还是堆上时都能够正确的访问__block变量。

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