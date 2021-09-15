---
title: LLDB 调试技巧
tags:
---

# 调试技巧

1. 利用objc
2. 利用swift

其中利用oc语法、在任意断点都可以使用。swift语法的话只能在swift代码断点才有效

## objc

### 基本使用

```lldb
e -l objc -O -- [oc表达式]
eg: e -l objc -O -- ['self.view' setHidden:true]
```

### 定义别名

```bash
e -l objc -O -- [oc表达式]
eg: e -l objc -O -- ['self.view' setHidden:true]
//定义别名
command alias poc expression -l objc -O --
po unsafeBitCast(0x7fe1cf8035f0, to: UIView.self) //利用swift转换

poc [self.view addSubview:(UIView *)self.label]
poc [self.label setFrame:CGRectMake(100, 100, 200, 200)]
poc [(id)0x7fcfda704700 setFrame:CGRectMake(100, 100, 200, 300)]
//刷新
e [CATransaction flush]

```

## Swift

### 基本使用

```
po unsafeBitCast(0x7fe1cf8035f0, to: UIView.self)
```

Tricks: 立刻刷新UI

```
(lldb) po unsafeBitCast(0x7fe1cf8035f0, to: UIView.self)
<UIView: 0x7fe1cf8035f0; frame = (0 0; 414 896); autoresize = W+H; layer = <CALayer: 0x600001ce87c0>>

(lldb) expression CATransaction.flush()
(lldb) po unsafeBitCast(0x7fe1cf8035f0, to: UIView.self).isHidden = true
(lldb) e CATransaction.flush()
(lldb) po unsafeBitCast(0x7fe1cf8035f0, to: UIView.self).isHidden = false
(lldb) e CATransaction.flush()

```

```
(lldb) e -l objc -O -- [`self.view` setHidden:true] //注意这里是反引号
0x00000000cff00b00

(lldb) e CATransaction.flush()

```

## 符号断点

## 观察断点(watch points)

```lldb
//跳过代码
thread jump --by 1
```



```
var $data = try JSONSerialization.data(withJSONObject: data, options: .prettyPrinted)
var $string = String(data: $data, encoding: .utf8)
$string
```

## 私有方法

```
-[NSObject _ivarDescription]：返回实例对象所有属性的值

+[NSObject _shortMethodDescription]：返回类的所有方法

-[UIView recursiveDescription]：打印 UIView 的层级结构

-[UIViewController _printHierarchy]：打印 UIViewController 的层级结构
```

## Image

### Lookup

Look up information for a raw address in the executable or any shared libraries.

```shell
(lldb) image lookup --address 0x1ec4
(lldb) im loo -a 0x1ec4

```

Look up functions matching a regular expression in a binary.

```shell
This one finds debug symbols:
(lldb) image lookup -r -n <FUNC_REGEX>

This one finds non-debug symbols:
(lldb) image lookup -r -s <FUNC_REGEX>
```

Find full source line information.

```shell
This one is a bit messy at present. Do:

(lldb) image lookup -v --address 0x1ec4

and look for the LineEntry line, which will have the full source path and line range information.
```

Look up information for an address in **a.out** only.

```
(lldb) image lookup --address 0x1ec4 a.out
(lldb) im loo -a 0x1ec4 a.out
```

## BreakPoint

```
//设置正则表达式断点
rb "-\[UIImageView setHighlighted:\]"
//类方法打断点
breakpoint set -n "[ACCRepoContextModel setVideoType:]"
```

参数打印

```
po $arg1
```



