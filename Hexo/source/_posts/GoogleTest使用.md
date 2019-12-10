---
title: GoogleTest使用
date: 2019-12-08 15:36:45
tags:
categories:
- [sundry]
typora-root-url: ../../source
---
# 前言

因为项目是跨平台（Mac与Win）的，且是用C++编写的，所以在单元测试方面能用的框架就不多了。公司里其他项目组都是用的GTest来写单元测试的，所以呢，我们也觉得使用GTest与GMock。本文主要是讲在Mac环境下，对GTest的环境搭建及使用

# 正文

## GTest与GMock介绍

GTest月GMock都是Google的开源C++测试框架，提供了一系列的断言与期望的宏，同时还提供了stub与mock的功能，这些都是单元测试不可或缺的。你可以在这里查看GTest的[GitHub主页](https://github.com/google/googletest)。

## 环境搭建

### 源码下载

首先需要在googletest下载最新的release源码

<img src="/images/image-20191208155702764.png"/>

### 源码编译

解压后，会看到如下的目录结构：

<img src="/images/image-20191208155950191.png" height=300/>

因为官方提供了两种编译库的方式，一种的CMake，另外一种是Bazel。这里介绍用CMake的方式，所以你需要先安装CMake需要的环境，这里就不介绍安装方式了，请自行查询。CMake的使用方式可以参考IBM的[这篇](https://www.ibm.com/developerworks/cn/linux/l-cn-cmake/index.html)文章。

CMake的环境准备好后，从命令行进入到该目录，并在命令行分别执行以下命令：

```bash
$ mkdir mybuild
$ cd mybuild
$ cmake ..
```

<img src="/images/image-20191208161040143.png"/>

执行成功后，应该是下面这个样子：

<img src="/images/image-20191208161130932.png"/>

接着执行以下指令：

```shell
make
```

执行成功后就生成了对应的.a文件了

![image-20191208161235027](/images/image-20191208161235027.png)

在finder中可以看到新生成的文件

<img src="/images/image-20191208162420154.png"/>

可以看到，一共生成了四个.a文件，其中有*_main.a表示文件中包含了main函数，所以使用此类文件时，工程里不能有其他的main函数。

### 工程创建，配置环境

在这里示例创建一个名为TestGoogletest的命令行工程。

<img src="/images/image-20191208162031530.png" width=500/>

接着导入编译好的.a文件，在这里我就不适用带有main函数的.a文件了。

直接将.a文件拖入工程

<img src="/images/image-20191208162639580.png" width=500/>

#### 导入头文件

打开finder，进入到源码目录，将googlemck与googletest目录下的include目录下的文件夹gmock与gtest加入到工程中

<img src="/images/image-20191208163307265.png" width=600/>

#### 配置头文件搜索路径

导入头文件后需要配置头文件搜索路径，打开工程文件，选择build setting，设置user header search path。

<img src="/images/image-20191208170630431.png" width=600/>

至此gtest与gmock框架添加完毕，接下来就是编写测试用例了

### 测试用例

main函数编写如下

```c++
#include <iostream>
#include "gtest/gtest.h"
#include <numeric>
#include <vector>

int main(int argc, const char * argv[]) {
    ::testing::InitGoogleTest(&argc, (char **)argv);
    return RUN_ALL_TESTS();
}

TEST(TestSuiteName, TestName) {
    EXPECT_EQ(1, 1);
}
```

接下来解释代码，GTest需要在main函数中启动，启动方式如下：需要注意的是，在mac平台下的main函数默认参数类型是==const char *==而gtest接收的类型是==char **==，所以需要做一个强制类型转换。同时return函数返回的是==RUN_ALL_TESTS==()，作用是执行所有的单元测试

```C++
int main(int argc, const char * argv[]) {
    ::testing::InitGoogleTest(&argc, (char **)argv);
    return RUN_ALL_TESTS();
}
```

TEST是GTest提供的单元测试宏，第一个参数是模块名，第二个参数是测试名称

```C++
TEST(TestSuiteName, TestName) {
    EXPECT_EQ(1, 1);
}
```

同时，GTest还提供了测试宏TEST_F。

以上就是googleTest的简单使用了，你可以在[这里](https://github.com/WuKongCoo1/Demo_TestGoogletest)下载demo

要了解更多的话可以参考以下链接：

[Googletest Primer](https://github.com/google/googletest/blob/master/googletest/docs/primer.md#simple-tests)

[Googletest README](https://github.com/google/googletest/blob/master/googletest/README.md)

