---
title: Mac 设置socket5代理
date: 2019-10-23 13:14:18
tags:
categories:
- [sundry]
typora-root-url: ../../source
---

# Mac 设置socket5代理

Mac端可以在设置->网络->高级->代理中设置http代理与socket代理，但是在这里设置的代理都是http的，并不能直接设置socket的代理。如果想要设置socket的代理的话，需要通过其他的软件来使用。在这里介绍利用==proxifier==来实现socket的代理。

步骤如下：点击proxies，进入proxy列表

<img src="/images/proxy_1.png" width=600pt/>



点击add进入到添加界面

<img src="/images/proxy_3.png" width=600pt/>

在添加界面输入代理服务器的address与port，并且选择协议类型为http5，点击ok

<img src="/images/proxy_2.png" width=600pt/>

这就设置好socket5的代理了。