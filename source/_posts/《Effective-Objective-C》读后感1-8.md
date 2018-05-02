---
title: 《Effective Objective-C》读后感1-8
date: 2018-05-02 13:30:55
tags: [Objective-C]
categories: "Effective Objective-C"
---

#### 1. Familiarize Yourself with Objective-C's Roots
OC是C的完全超集，其中一个很大的区别是C语言为函数调用，而OC为消息传递。这其中一个主要的区别在于消息传递会在运行时决定哪些消息被发送到哪些对象，而函数调用在编译时就确定了这些事情。也正因为此，OC的运行时就必须包含所有数据和函数。也正因为这个，只要运行时（系统）升级之后，使用OC的App也就自动地享有了这些升级带来的优化。而更注重编译时的语言，则需要重新编译之后才能享有这些优化。

OC的一个对象数据如： `NSString* string = @ "abc";` 它的内存情况其实是一个在Stack中开辟的一个指针空间（在32位的设备上是4个bytes，在64位上是8个bytes）指向了一个在Heap上开辟的存abc这个字符串的空间。

#### 2. Minimize Importing Headers in Headers
尽量使用@class来避免在头文件中import 另一个类的头文件

#### 3. Prefer Literal Syntax over the Equivalent Methods
尽量使用类似如下的代码：

```
NSNumber *intNumber = @1;
NSArray *animals = @[@"cat", @"dog", @"mouse", @"badger"];
NSDictionary *dic = @{@"key1": @"value1", @"key2": @"value2"};
```

#### 4. Prefer Typed Constants to Preprocessor #define
尽量少用#define来定义常量，而是使用如下的方式定义常量：
`static const NSTimeInerval kAnimationDuration = 0.3;`
这种方式保证了为编译器提供了类型检查（新版本的Xcode似乎在#define中也能做类型检测）。另外，const保证了定义的是一个常量。 static保证了这个常量是在一个.m中local的，不会和其他文件中相同的名字的变量产生冲突，因为本质上，如果加了static关键字后编译器就会在编译的时候想#define一样，将定义的内容直接展开到对应的代码中。
对于NSNotificaiton中使用的name，建议使用的定义方式如下：
`extern NSString *const XXXXDidFinishLoginNotification;`

#### 5. Use Enumerations for States, Options, and Status Codes
在switch case代码段中，不要去写default，这样的话，当你在枚举中添加新的元素的时候，编译器就给出提示，因为代码中并没有将所有枚举中的情形都覆盖全。

#### 6. Understand Properties
#### 7. Access Instance Variables Primarily Directly When Accessing Them Internally
当在类的内部访问成员变量的时候，直接访问成员变量和使用property访问有几点区别：
直接访问变量可以直接获得希望获得的数据或直接修改对应的值，而使用property访问将会出发消息传递的机制，使得可能出现因运行时而产生的意外，因此，从某种程度上说，直接访问将会更加“安全”。
直接访问将会绕过变量的内存管理机制。（似乎最新的版本中并不会绕过）
直接访问将不会触发KVO通知。（存疑）
使用property会更加方便调试，可以在setter和getter中加断点。（直接访问的话，watch似乎也能解决一部分问题）

类的内部直接访问成员变量有几个需要注意的点：
在初始化函数中给成员变量赋值的时候，使用直接访问的方式进行赋值：子类可能会重载setter函数，从而产生意外的情况。
对于一些Lazy初始化的变量，如果使用直接访问的话，可能导致这个变量无法被正常初始化。

#### 8. Understand Object Equality