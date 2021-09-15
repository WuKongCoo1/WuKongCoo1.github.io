---
title: perfor selector实践
tags:
---



**Student.h**

```objc
@interface Student : NSObject

@property (nonatomic, copy) NSString *name;


@end

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
```

测试代码

```objc
void foo(void)
{
    Student *student = [Student performSelector:NSSelectorFromString(@"newStudent:") withObject:@"jacky"];
    Student *student2 = [Student performSelector:NSSelectorFromString(@"studentWithName:") withObject:@"cook"];
    Student *student3 = [Student newStudent:@"nacy"];
    Student *student4 = [Student newStudent:@"ttt"];
    [Student studentWithName:@"232323232"];
}
```

执行结果

```
2021-09-15 18:41:22.781820+0800 DemoPerformSelector[65943:5388280] ttt dealloc
2021-09-15 18:41:22.781982+0800 DemoPerformSelector[65943:5388280] nacy dealloc
2021-09-15 18:41:22.782092+0800 DemoPerformSelector[65943:5388280] cook dealloc
```

结论：`NSSelectorFromString(@"newStudent:`的方式会出现内存泄露

AutoRelease测试

```objc
- (void)foo
{
    @autoreleasepool {
        NSLog(@"autoreleasepool begin");
        {
            NSLog(@"bracket begin");
            Student *stu1, *stu2;
            [Student studentWithName:@"nonAllocFamily_WithRefer_Nacy"];//autorelease
            stu1 = [Student studentWithName:@"nonAllocFamily_NoneRefer_Nacy"];
            stu2 = [Student newStudent:@"AllocFamily_WithRefer_jacky"];
            NSLog(@"before AllocFamily_NoneRefer_jacky");
            [Student newStudent:@"AllocFamily_NoneRefer_jacky"];
            NSLog(@"after AllocFamily_NoneRefer_jacky");
        }
        NSLog(@"after bracket");
    }
}

```

结果

```
2021-09-15 18:41:22.782293+0800 DemoPerformSelector[65943:5388280] autoreleasepool begin
2021-09-15 18:41:22.782375+0800 DemoPerformSelector[65943:5388280] bracket begin
2021-09-15 18:41:22.782486+0800 DemoPerformSelector[65943:5388280] nonAllocFamily_WithRefer_Nacy dealloc
2021-09-15 18:41:22.782563+0800 DemoPerformSelector[65943:5388280] before AllocFamily_NoneRefer_jacky
2021-09-15 18:41:22.782655+0800 DemoPerformSelector[65943:5388280] AllocFamily_NoneRefer_jacky dealloc
2021-09-15 18:41:22.782758+0800 DemoPerformSelector[65943:5388280] after AllocFamily_NoneRefer_jacky
2021-09-15 18:41:22.782855+0800 DemoPerformSelector[65943:5388280] AllocFamily_WithRefer_jacky dealloc
2021-09-15 18:41:22.782950+0800 DemoPerformSelector[65943:5388280] nonAllocFamily_NoneRefer_Nacy dealloc
2021-09-15 18:41:22.802374+0800 DemoPerformSelector[65943:5388280] after bracket
```

结论：非alloc家族的调用会放到AutoreleasePool中

