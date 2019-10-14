---
title: C、C++格式化字符串
date: 2019-09-09 18:05:44
tags:
categories:
- [sundry]
typora-root-url: ../../source
---

# 引言

在C和C++开发中，我们经常会用到printf来进行字符串的格式化，例如`printf("format string %d, %d", 1, 2);`，这样的格式化只是用于打印调试信息。printf函数实现的是接收可变参数，然后解析格式化的字符串，最后输出到控制台。那么问题来了，当我们需要实现一个函数，根据传入的可变参数来生成格式化的字符串，应该怎么办呢？

# 正文

## 可变参数

首先来一个可变参数使用示例，`testVariadic`方法接收int行的可变参数，并以可变参数为-1表示结束。va_list用于遍历可变参数，`va_start`方法接收两个参数，第一个为`va_list`，第二个为可变参数前一个参数，下面的例子里该参数为a。

```c
/**
 下面是 <stdarg.h> 里面重要的几个宏定义如下：
 typedef char* va_list;
 void va_start ( va_list ap, prev_param ); // ANSI version
 type va_arg ( va_list ap, type );
 void va_end ( va_list ap );
 va_list 是一个字符指针，可以理解为指向当前参数的一个指针，取参必须通过这个指针进行。
 <Step 1> 在调用参数表之前，定义一个 va_list 类型的变量，(假设va_list 类型变量被定义为ap)；
 <Step 2> 然后应该对ap 进行初始化，让它指向可变参数表里面的第一个参数，这是通过 va_start 来实现的，第一个参数是 ap 本身，第二个参数是在变参表前面紧挨着的一个变量,即“...”之前的那个参数；
 <Step 3> 然后是获取参数，调用va_arg，它的第一个参数是ap，第二个参数是要获取的参数的指定类型，然后返回这个指定类型的值，并且把 ap 的位置指向变参表的下一个变量位置；
 <Step 4> 获取所有的参数之后，我们有必要将这个 ap 指针关掉，以免发生危险，方法是调用 va_end，他是输入的参数 ap 置为 NULL，应该养成获取完参数表之后关闭指针的习惯。说白了，就是让我们的程序具有健壮性。通常va_start和va_end是成对出现。
 */
//-1表示可变参数结束
void receiveVariadic(int a, ...) {
    va_list list;
    va_start(list, a);
    int arg = a;
    while (arg != -1) {
        arg = va_arg(list, int);
        printf("%d ", arg);
    }
    printf("\n");
    va_end(list);
}

//test
void testVari()
{
    printf("------%s------\n", __FUNCTION__);
    //-1表示可变参数结束
    receiveVariadic(1, 2, 3, 4, 5, 6, -1);
}
```

运行结果

```
------testVari------
2 3 4 5 6 -1 
```

## 格式化字符串

好了，我们已经介绍了怎样实现一个接收可变参数的C函数，接下来介绍根据接收的可变参数来格式化字符串。这里介绍两种方式，第一种是利用宏定义，第二种通过函数的方式来实现。

### 通过宏定义的方式

en…让咱们先来看看第一个版本的宏，这个宏定义对于不熟悉宏的人来说可能看着有点费劲，不过不要怕，稍后会做解释，代码如下：

```c
#define myFormatStringByMacro_WithoutReturn(format, ...) \
do { \
    int size = snprintf(NULL, 0, format, ##__VA_ARGS__);\
    size++; \
    char *buf = (char *)malloc(size); \
    snprintf(buf, size, format, ##__VA_ARGS__); \
    printf("%s", buf); \
    free(buf); \
} while(0)

```

#### 宏基础知识

首先需要介绍宏用到的知识：`\`, 这个`\`的作用是可换行定义宏，毕竟如果一行很长的宏可读性很差，使用方式在换行时加上`\`即可。第二个是介绍`(format, ...)`，这里的`...`是预定义的宏，用于接收可变参数，就像是`printf`函数一样。接着介绍`##__VA_ARGS__`，同样的`__VA_ARGS__`也是预定义的宏，表示接收到的`...`传入的可变参数。`##`的作用是用来处理未传入可变参数的情况，当没有传入可变参数的时候，编译器或通过优化将`snprintf(NULL, 0, format, ##__VA_ARGS__);`优化为`snprintf(NULL, 0, format);`。你可以理解为没有可变参数时，`##`前的逗号`,`与`__VA_ARGS__`都被“干掉了”。

你一定会觉得困惑，为什么要写`do-while`语句呢？这是为了宏的健壮性，如果使用宏的人像下面这样使用的话，就会出问题

```c
#define testMarco(a, b) \
int _a = a + 1; \
int _b = b + 1; \
printf("\n%d", _a + _b); \

void test()
{
    if (1 > 0)
        testMarco(1, 2);
}
```

上面的代码连编译都不会通过, 会报错如下:

![](/images/macro_1.png)

如果手动展开这个宏的话，会变成这个样子，问题就显而易见了。但是如果`if`语句加上了`{}`的话，就不会有问题，可以看出规范写法是多么的重要🐶（皮一下很开心）。

```c
void test()
{
    if (1 > 0)
        int _a = 1 + 1; int _b = 2 + 1; printf("\n%d", _a + _b);;
}
```

加上`do-while`以后就不一样，加上do-while后的代码如下：

```c
#define testMarco(a, b) \
do { \
int _a = a + 1; \
int _b = b + 1; \
printf("\n%d", _a + _b); \
} while(0)

void test()
{
    if (1 > 0)
        testMarco(1, 2);
}


```

预处理之后代码如下：

```c
//展开后的代码 
void test()
{
    if (1 > 0)
        do { int _a = 1 + 1; int _b = 2 + 1; printf("\n%d", _a + _b); } while(0);
}
```

好了，宏的基础知识就介绍这么多了，接下来进入正题。

#### 代码解析

为了方便阅读，原谅我在这里再贴一遍宏定义的代码：

```c
#define myFormatStringByMacro_WithoutReturn(format, ...) \
do { \
    int size = snprintf(NULL, 0, format, ##__VA_ARGS__);\
    size++; \
    char *buf = (char *)malloc(size); \
    snprintf(buf, size, format, ##__VA_ARGS__); \
    printf("%s", buf); \
    free(buf); \
} while(0)
```

首先，介绍一下`snprintf()`函数，此函数的定义如下：

```c
/**
 
 @param __str 接收格式化结果的指针
 @param __size 接收的size
 @param __format 格式化的字符串
 @param ... 可变参数
 @return 返回格式化后实际上写入的大小a，a <= __size
 */
int     snprintf(char * __restrict __str, size_t __size, const char * __restrict __format, ...) __printflike(3, 4);
```

为了方便理解，使用方式是这个样子的：

```c
void testSnprintf()
{
    printf("------%s------\n", __FUNCTION__);
    char des[50];
    int size = snprintf(des, 50, "less length %d", 50);
    printf("size:%d\n", size);
}
```

运行结果：

```
------testSnprintf------
size:14
```

`snprintf`函数还有一个用法是`__str`和`__size`分别传入NULL和0，返回值会是格式化字符串的实际长度，可以通过这个方式来获取正确的格式化size，从而避免malloc多余的空间，造成空间浪费。同时返回的size是不包含结束符`\0`的，所以真正写入要buffer时，需要对size + 1。

相信通过我的解释，你一定能看懂上面这段代码了吧。哦，对了malloc的代码一定要记得free（敲重点）。

到了这里，如果细心思考的同学一定会问？这个宏根本没有实际用途好不好，我要的是能够把格式化的字符串作为返回值返回的，仅仅打印直接用`printf`不就好了。其实，这样的宏还是有作用的，比如说当你要记录日志时，你可以像这样使用：

```c
#define Log_Debug(format, ...) \
do { \
int size = snprintf(NULL, 0, format, ##__VA_ARGS__);\
size++; \
char *buf = (char *)malloc(size); \
snprintf(buf, size, format, ##__VA_ARGS__); \
doLog(buf); \
free(buf); \
} while(0)
```

要将结果字符串返回的话，需要用到GNU C的赋值扩展，使用方式如下:

```c
int a = ({
        int b = 2;
        int c = 4;
        b + c;
    });
```

这段代码变量a最终值会是6。利用gnu这个扩展，将之前的宏改造一下就能实现我们的需求，改造完成后是这个样子的：

```c
#define myFormatStringByMacro_ReturnFormatString(format, ...) \
({ \
    int size = snprintf(NULL, 0, format, ##__VA_ARGS__);\
    size++; \
    char *buf = (char *)malloc(size); \
    snprintf(buf, size, format, ##__VA_ARGS__); \
    buf; \
});
```

调用宏的代码：

```c
void testByMacro1()
{
    printf("------%s------\n", __FUNCTION__);
    char *a = myFormatStringByMacro_ReturnFormatString("format by macro, %d %s", 123, "well done");
    printf("%s\n", a);
    free(a);
}
```

原谅我的啰嗦，malloc开辟的空间一定要记得free。运行结果：

```c
------testByMacro1------
format by macro, 123 well done
```

至此利用宏的方式就介绍完了。

### 通过函数的方式

老规矩先上代码

```c
char *myFormatStringByFun(char *format, ...)
{
    va_list list;
    //1. 先获取格式化后字符串的长度
    va_start(list, format);
    int size = vsnprintf(NULL, 0, format, list);
    va_end(list);
    if(size <= 0) {
        return NULL;
    }
    size++;
    
    //2. 复位va_list，将格式化字符串写入到buf
    va_start(list, format);
    char *buf = (char *)malloc(size);
    vsnprintf(buf, size, format, list);
    va_end(list);
    return buf;
}
```

这里利用的是`vsnprintf`函数，此函数的定义在`stdio.h`中的定义如下：

```c
/**

 @param __str 目标字符串
 @param __size 要赋值的大小
 @param __format 格式化字符串
 @param va_list 可变参数列表
 @return 返回格式化后实际上写入的大小a，a <= __size
 */
int     vsnprintf(char * __restrict __str, size_t __size, const char * __restrict __format, va_list) __printflike(3, 0);
```

`vsnprintf`的具体使用方式和之前介绍的`snprintf`是差不多的，这里就不再详细介绍了，不大明白的同学可以看看上面的介绍。哦，对了，这两个函数都是定义在stdio.h这个头文件下的

接下来就是试一下我们封装的函数了

```c
void testByFun()
{
    printf("------%s------\n", __FUNCTION__);
    char *b = myFormatStringByFun("format by fun %d %s", 321, "nice");
    printf("%s\n", b);
}
```

运行结果：

```c
------testByFun------
format by fun 321 nice
```

格式化字符串的方法差不多介绍完了，不知道善于思考的你有没想到直接用宏定义来调用我们封装的函数呢？我就在这直接给出宏定义和使用方式了

```c
#define myFormatStringByFunQuick(format, ...) myFormatStringByFun(format, ##__VA_ARGS__);
void testMyFormatStringByFunQuick() {
    printf("------%s------\n", __FUNCTION__);
    char *formatString = myFormatStringByFunQuick("amazing happen, %s", "cool");
    printf("%s\n", formatString);
}
```

运行结果：

```c
------testMyFormatStringByFunQuick------
amazing happen
```

### C++版本

对了，最初实现是用的C++版本，这里使用的是泛型，代码是这个样子的：

```c++
template< typename... Args >
std::string string_sprintf( const char* format, Args... args ) {
    int length = std::snprintf( nullptr, 0, format, args... );
    assert( length >= 0 );
    
    char* buf = new char[length + 1];
    std::snprintf( buf, length + 1, format, args... );
    
    std::string str( buf );
    delete[] buf;
    return str;
}
```

其实和C语言版本的没什么差别，只是多了泛型的东西而已，相信聪明的你一定能看懂，看不懂的话，就去看看C++的泛型知识吧，哈哈哈。

# 结语

终于介绍完了，你可以在[这里](https://github.com/WuKongCoo1/DemoFormatString)下载代码。写博客是真的有点累人，不过对于最近被面试打击的我来说，写博客能够让我对知识理解的更加透彻，毕竟要自己认真思考后才能够写的明白（至少我觉得讲明白了，哈哈哈）。如果有什么说的不对的地方，还请指出，感谢你的阅读，thks。

# 参考资料

[std::string formatting like sprintf](https://stackoverflow.com/questions/2342162/stdstring-formatting-like-sprintf)

[宏定义的黑魔法 - 宏菜鸟起飞手册](https://onevcat.com/2014/01/black-magic-in-macro/)

[整理：C/C++可变参数，“## __VA_ARGS__”宏的介绍和使用](https://blog.csdn.net/bat67/article/details/77542165)