---
title: ARC详解
date: 2019-10-14 15:03:14
categories:
- [Objective-C]
---

# 引言：

Objective-C（以下简称OC）的内存管理是通过引用计数来完成的，具体来说有两种方式。以前呢，我们是通过MRC（Manual Reference Counting 手动引用计数）的方式来管理，而现在呢，现在苹果官方是推荐我们使用ARC（Automatic Reference Counting 自动引用计数）。

# 正文

## 内存管理基础知识

oc中对对象的引用计数retain count有影响的方法有：retain、release、autorelease等，其中retain会使对象的引用计数+1，而release与autorelease会使对象的引用计数减一。当执行release后，如果对象的引用计数减少为0，则执行对象的dealloc方法，来释放内存。总结起来就是：

- retain 递增引用计数
- release 递减引用计数
- autorelease 在自动释放池 pop时，再递减引用计数

## MRC 手动引用计数

在MRC管理方式中，管理原则是谁创建谁释放，即一个对象或者一个方法alloc或者retain了对象了，在使用完毕后，一定要执行release或者autorelease方法。如：

```objective-c
NSObject *obj = [[NSObject alloc] init];
//do something ...
[obj release];
```

在平时的编码过程中，我们需要创建和销毁非常多的对象，如果通过MRC的方式来管理的话，工作量会非常的繁琐，于是乎苹果的工程师为我们提供了更便捷的管理方式ARC。

## ARC 自动引用计数

### ARC简介

简单来说ARC的作用是编译器在编译的时候，帮我们加入适当的release和retain调用。例如在ARC中的如下代码：

```objc
- (void)foo
{
    NSObject *object = [[NSObject alloc] init];
    NSLog(@"%@", object);
}
```

上面这段代码，在ARC中会被改写为如下形式：

```objc
- (void)foo
{
    NSObject *object = [[NSObject alloc] init];
    NSLog(@"%@", object);
    [object release]; //ARC添加的
}
```

由此可见，在ARC的管理方式下，引用计数任然是有效的，只是release与retain操作交给ARC自动添加了。因为ARC会自动为我们添加内存管理语义，所以在ARC下是不能主动调用内存管理语义的，具体如下：

- retain
- release
- autorelease
- dealloc

熟悉MRC的人可能会觉得ARC只是替我们调用了release与retain等方法，其实ARC并不是直接通过消息派发机制来调用的，而是直接通过调用底层的C语言版本。C语言对应的方法如下：

- retain -> objc_retain
- release -> objc_release
- autorelease -> objc_autorelease

### ARC命名规则

总的来说ARC下的内存管理是通过命名规则来决定对象的管理权的归属，具体来说通过以下名称开头产生的对象管理权归调用者所有：

- alloc
- new
- copy
- mutableCopy

如果不是通过以上语句开头产生的，那么则有被调用方管理，ARC会在创建方法末尾添加调用autorelease。

示例如下：

```objc
@implementation Student
+ (id)newStudent:(NSString *)name
{
    Student *stu = [[Student alloc] init];
    stu.name = name;
    //管理权归被调用方所有
    return stu;
}

+ (id)studentWithName:(NSString *)name
{
    Student *stu = [[Student alloc] init];
    /**
     在arc下管理权归被调用者
     return [stu autorelease];
     */
    stu.name = name;
    return stu;
}

- (void)dealloc
{
    NSLog(@"%@ dealloc", self.name);
}



@end

void foo2()
{
    NSLog(@"%s begin", __func__);
    {
        Student *stu1, *stu2;
        stu1 = [Student newStudent:@"Jacky"];
        /**
         [stu1 release];
         */
        stu2 = [Student studentWithName:@"Nacy"];
    }
        NSLog(@"%s end", __func__);
}

```

运行结果：

```text
2019-10-14 18:06:19.802799+0800 Test_ARC[9252:11351467] foo2 begin
2019-10-14 18:06:19.802865+0800 Test_ARC[9252:11351467] Jacky dealloc
2019-10-14 18:06:30.592471+0800 Test_ARC[9252:11351467] foo2 end
2019-10-14 18:06:33.308012+0800 Test_ARC[9252:11351467] Nacy dealloc
```

可能你会对这个运行结果存在疑惑，正常来说stu1与stu2的生命周期都应该在=={}==中，出了中括号这两个变量都应该销毁，变量销毁后，指向的对象也应该马上销毁。但是只有stu1在生命周期结束后销毁了，而stu2却延迟销毁了。为什么会这样呢，就像之前介绍说到的，stu2是通过==studentWithName==创建的，这不符合ARC中管理规则，所以stu2是以autorelease的方式返回的，内存管理方式是交给被调用方的。同理，就可以明白stu1会在变量作用于结束时销毁的原理了

### ARC的优化

上面讲到，如果对象是通过不符合ARC规则的方法创建的，那么ARC会为其加上autorelease，如果这时候接收的变量是强指针的话，那么ARC会为其加上retain调用。示例如下：

```objc
Student *stu = [Student studentWithName:@"Nacy"];
/**
在arc处理后代码变为
Student *stu = [Student studentWithName:@"Nacy"];
[stu retain];
*/
```

这样看来的话，在这里studentWithName中的autorelease与这里的retain就是多余的了，可以删除。但是为了兼容性，ARC没有直接删除。ARC不会直接调用autorelease，而是执行另外的函数objc_autoreleaseReturnValue。这个函数会检查返回值之后的代码，如果之后的代码有执行retain操作，那么就设置全局结构，不执行autorelease操作。同样的，在调用retain方法时，会调用objc_retainAutoreleasedReturnValue。这个方法会检查上面提到的标志位，如果标志位被设置了的话，那么就不会执行retain操作，并重置检测标志位。

