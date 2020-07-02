---
title: iOS富文本处理采坑记录(二)
category:
  - UIKit
typora-root-url: ../../source
date: 2020-07-02 00:49:33
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

系列文章

## 遍历字符串文字不连续

当从UITextView读取attributedText后，如果执行enumerateAttributes方法遍历，会出现unicode、中英文字符分开获取到的情况

### 示例

```swift
let attributeString = NSMutableAttributedString(string: "🔴🔴哈哈哈哈哈哈www.baidu.com", attributes: [.font : UIFont.systemFont(ofSize: 14)])
textView.attributedText = attributeString

if let amendRange = textView.attributedText.string.amendRange() {
    textView.attributedText.enumerateAttributes(in: amendRange, options: []) { (_, range, stop) in
        print(textView.attributedText.attributedSubstring(from: range))
    }
}
```

### 运行结果

```
🔴🔴{
    NSFont = "<UICTFont: 0x7f9c90f03ff0> font-family: \".AppleColorEmojiUI\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSOriginalFont = "<UICTFont: 0x7f9c90d08fd0> font-family: \".SFUI-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
}
哈哈哈哈哈哈{
    NSFont = "<UICTFont: 0x7f9c90e03610> font-family: \".PingFangSC-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
    NSOriginalFont = "<UICTFont: 0x7f9c90d08fd0> font-family: \".SFUI-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
}
www.baidu.com{
    NSFont = "<UICTFont: 0x7f9c90d08fd0> font-family: \".SFUI-Regular\"; font-weight: normal; font-style: normal; font-size: 14.00pt";
}
```

由运行结果可以看出，在我们进行遍历时，获取到的range是按照Unicode、中英文来断开的，正常情况这是没有什么影响的，但是对于文本编辑来说，因为有链接、普通文字、at等情况，所以需要将enumerate中属性相同的string拼接到一起。大概思路就是利用attribute来判断substring是否属于同一数据段，具体做法请看接下来介绍的富文本的选择、删除处理。

## 富文本选择、删除处理

### 需求介绍

在富文本编辑中，经常做的是链接、@信息、普通文本等编辑，这些类型当中，通常只有普通文本支持单字选择与删除，而链接与@信息的话一般是只能进行整体删除和选择（不能从segment中间开始选择）。

上文介绍到，当我们遍历从UITextView中获取到的富文本时，获取到的字符串并不是连续的，但是呢，我们的需求需要将相邻的相同的数据合并到一起。那我们根据什么条件来判断是否相邻的字符串是否是相同segment呢？有以下两个条件：

1. range连续
2. attribute相同

只要同时满足上诉两个条件，就可以认为是属于相同segment，应该合并到一起。

#### 选择

##### 效果：

![TextEditExample](/images/TextEditExample.png)

先上选择的代码：

```swift
func fixSelectRange(_ attributeString: NSAttributedString, targetRange: NSRange) -> NSRange {
    var fixRange = targetRange

    guard let range = attributeString.string.amendRange() else {
       return fixRange
    }

    var prevAttribute: AttributeInfoProtocol!

    attributeString.enumerateAttributes(in: range, options: []) { (attributes, atRange, stop) in
        guard let attribute = getAttributeInfo(attributes) else {
            return
        }

        let shoudStop = prevAttribute != nil && !prevAttribute.isEqual(attribute)
        if shoudStop {
            stop.pointee = true
            return
        }

        if fixRange.location >= atRange.location && fixRange.location + fixRange.length <= atRange.location + atRange.length {
            if attribute.type == .text {
                fixRange = targetRange
                stop.pointee = true
                return
            }
            
            fixRange.location = atRange.location + atRange.length
            fixRange.length = 0
            prevAttribute = attribute
            return
        }
    }
    return fixRange
}
```

逻辑其实挺简单的，如果fixRange的location在获取atRange中，即：

```swift
fixRange.location >= atRange.location && fixRange.location + fixRange.length <= atRange.location + atRange.length
```

如果当前的attribute是text的话，就不需要进行修改

```swift
if attribute.type == .text {
	fixRange = targetRange
	stop.pointee = true
	return
}
```

如果不是文本的话，那么退出条件为：

```
prevAttribute != nil && !prevAttribute.isEqual(attribute)
```

#### 删除

删除与选择不同，选择的话只需要将location移动到segment前即可，删除的话 ，需要将整个segment删掉

##### 效果

![textDeleteExample](/images/textDeleteExample.png)

##### 代码

```swift
func fixDeleteRange(_ attributeString: NSAttributedString, targetRange: NSRange) -> NSRange {
    var currentMaxRange: NSRange?
    guard let range = attributeString.string.amendRange() else {
       return targetRange
    }
    
    var prevAttribute: AttributeInfoProtocol?
    var didFound = false
            
    attributeString.enumerateAttributes(in: range, options: [.reverse]) { (attributes, atRange, stop) in
        print(attributeString.attributedSubstring(from: atRange).string)
        guard let attribute = getAttributeInfo(attributes) else {
            return
        }
        
        if let prevAttr = prevAttribute, prevAttr.isEqual(attribute) {
            currentMaxRange?.concatenateRange(atRange)
        } else {
            if let maxRange = currentMaxRange, targetRange.location >= maxRange.location && targetRange.location <= maxRange.location + maxRange.length {
                stop.pointee = true
                didFound = true
                return
            }
            currentMaxRange = atRange
            prevAttribute = attribute
        }
    }
    
    if didFound == false, let maxRange = currentMaxRange, targetRange.location >= maxRange.location && targetRange.location <= maxRange.location + maxRange.length {//处理删除目标在第一个info中的情况
        didFound = true
    }
    
    if didFound {
        if prevAttribute?.type == .text {
            return targetRange
        }
        return currentMaxRange ?? targetRange
    }
    return targetRange
}

extension NSRange {
    func isContinuous(_ range: NSRange) -> Bool{
        return self.location + self.length == range.location
    }
    
    /// right concatenate left eg: self:(1, 3) concatenate left:(0, 1) -> (0, 4)
    /// - Parameter left: left range
    mutating func concatenateRange(_ left: NSRange) {
        if left.location + left.length > location + length {
            return
        }
        
        if left.location + left.length < location {
            return
        }
        
        length = location + length - left.location
        self.location = left.location
//      length = (rightMax - leftMax) + (leftMax - rightMin) + (rightMin - leftMin)
    }
}
```

##### 原理：

逆序遍历attributes，将连续的attributes的range拼接起来，如果targetRange在currentMaxRange中并且不是text类型时，应该删除maxRange，否则直接删除targetRange

## 结语

以上就是富文本的选择删除处理了，你可以在[这里](https://github.com/WuKongCoo1/DemoTextView)下载demo，如果觉得对你有帮助的话，可以点一下star哦，thks