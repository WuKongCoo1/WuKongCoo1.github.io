---
title: iOS富文本处理踩坑记录（一）
tag:
  - - 富文本编辑
category:
  - - UIKit
date: 2020-06-20 16:38:28
tags:
---
<!--categories:-->
<!--- [Objective-C]-->
<!--- [net]-->
<!--- [OpenGL]-->
<!--- [sundry]-->
<!--- [Cocoa]-->
<!--- [DesignPattern]-->



# 前言

最近在做基于UITextView的富文本编辑功能，遇到了不少的坑，总感觉起来呢有以下几点：

- 当字符串中含有Unicode编码时，直接获取的range不对
- 在中英文混合的情况下，遍历字符串会不连续
- 富文本中的选择、删除处理
- 富文本中的链接点击处理

# 正文

## range不对

### 场景

在富文本中有时候需要高亮文本，但是遇到了文本只有部分高亮的问题，代码如下：

```swift
let string = "🔴1🔴2🔴3内部文档1"
let attributeString = NSMutableAttributedString(string: string)
let range = NSRange(location: 0, length: string.count)
attributeString.addAttribute(.foregroundColor, value: UIColor.blue, range: range)
textView.attributedText = attributeString
```

效果：

<img src="/Users/bytedance/Documents/Personal/blog/Hexo/source/images/image-20200620155224529.png" width=300>

上面的代码看起来是没有问题的，既然颜色值没有全部生效，那么问题一个是出在了rang上。通过查资料发现，在swift中使用了扩展字型集群，所以swift中会对可以组合在一起的两个字符，以下为示例:

```swift
let precomposed: Character = "\u{D55C}" // 한
let characters = ["\u{1112}", "\u{1161}", "\u{11AB}"] // ᄒ, ᅡ, ᆫ
var string = String()
for character in characters {
    string.append(character)
    print("string:\(string) count:\(string.count)")
}
```

运行结果：

```swift
string:ᄒ count:1
string:하 count:1
string:한 count:1
```

可以看出虽然上面的string已经append了三个character，但是count任然是1。那么我们应该怎么样做呢？主要有以下两种办法：

1. string as NSString
2. range convert to NSRange

示例代码如下：

```swift
let string = "🔴1🔴2🔴3内部文档1"
//method 1  string as NSString
print("NSString length: \((string as NSString).length)")
print("String length: \(string.count)")

let wrongRange = NSRange(location: 0, length: string.count)
//method 2 range convert to NSRange
let r1 = string.range(of: string)
var correctRange: NSRange?
if let tR1 = r1 {
    correctRange = NSRange(tR1, in: string)
}
print("wrongRange\(wrongRange)")
print("correctRange: \(String(describing: correctRange))")
```

运行结果：

```
NSString length: 14
String length: 11
wrongRange{0, 11}
correctRange: Optional({0, 14})
```

## 参考资料

https://stackoverflow.com/questions/25138339/nsrange-to-rangestring-index/30404532#30404532

https://stackoverflow.com/questions/42114872/nsattributedstring-and-emojis-issue-with-positions-and-lengths

