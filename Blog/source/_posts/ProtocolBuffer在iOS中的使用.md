---
title: ProtocolBuffer在iOS中的使用
date: 2018-05-10 18:44:07
tags: protobuf
categories: tool
---

## Mac下环境搭建

在Mac上安装protobuf有两种方式
- Homebrew的方式安装
- 下载源文件自行编译

homebrew的方式很简单，就一条命令:

```
brew install protobuf
```

验证是否成功

```
protoc --version
libprotoc 3.5.1
```

如果报错如下

```
zsh: command not found: protoc
```

那么再执行如下命令

```
brew link --overwrite protobuf
```

编译源文件的方式，先下载[源文件](https://github.com/google/protobuf/releases)，解压后进入文件目录

执行`brew list`查看是否已安装过`automake`和`libtool`如果没有安装，需要先用brew来安装这两个工具
```
brew install automake
brew install libtool
```
然后在解压后工程目录下，执行如下操作
```
./autogen.sh
./configure
make check
make
make install
```
这样就安装完成了

## 文件编写

首先要创建一个以`.proto`结尾的文件，然后在文件中编写模型内容

指定protobuf版本

```
syntax = "proto3";
```

为消息添加统一的前缀

```
option objc_class_prefix = "Art";
```

具体消息

```
message TypeTest {
	enum State {
    	STATE_NONE 	=	0;
    	// 按下
 	    DOWN 	= 	1;
 	    // 移动
        MOVE 	= 	2;
        // 抬起
        UP 	= 	3;
	}
    float float_value = 1;
    double double_value = 2;
    bool bool_value = 3;
    int32 int32_value = 4;
    string string_value = 5;
    State state = 6;
    bytes data = 7;//NSData
    TestObject testObj = 8;// 缩放位置信息
    repeated TestObject objList = 9; // NSMutableArray
}
message TestObject {
    string name = 1;
}
```

编译后的文件

```
@interface ArtTypeTest : GPBMessage

@property(nonatomic, readwrite) float floatValue;

@property(nonatomic, readwrite) double doubleValue;

@property(nonatomic, readwrite) BOOL boolValue;

@property(nonatomic, readwrite) int32_t int32Value;

@property(nonatomic, readwrite, copy, null_resettable) NSString *stringValue;

@property(nonatomic, readwrite) ArtTypeTest_State state;

/** NSData */
@property(nonatomic, readwrite, copy, null_resettable) NSData *data_p;

/** 缩放位置信息 */
@property(nonatomic, readwrite, strong, null_resettable) ArtTestObject *testObj;
/** Test to see if @c testObj has been set. */
@property(nonatomic, readwrite) BOOL hasTestObj;

/** NSMutableArray */
@property(nonatomic, readwrite, strong, null_resettable) NSMutableArray<ArtTestObject*> *objListArray;
/** The number of items in @c objListArray without causing the array to be created. */
@property(nonatomic, readonly) NSUInteger objListArray_Count;

@end

#pragma mark - ArtTestObject

@interface ArtTestObject : GPBMessage

@property(nonatomic, readwrite, copy, null_resettable) NSString *name;

@end
```
## 生成所需文件

```
protoc --plugin=/usr/local/bin/protoc-gen-objc message.proto --objc_out="./"
或简写成
protoc message.proto --objc_out="./"
```

## 使用

```Objective-C
ArtSearchRequest *request = [ArtSearchRequest new];
request.query = @"aaa";
request.pageNumber = 10;
request.resultPerPage = 20;
//编码
NSData *data = [request data];

//解码
ArtSearchRequest *search = [ArtSearchRequest parseFromData:data error:nil];

```

## 参考

[Objective C Generated Code](https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated)

