---
layout: post
title: "weakSelf与strongSelf"
date: 2015-08-03
comments: true
categories: iOS
tags: [iOS]
keywords: Carthage
publish: true
description:  weakSelf与strongSelf
---
对于weakSelf与strongSelf的一些理解。
<!-- more -->

## weakSelf

`weakSelf`的作用是防止`retain`环的出现而导致内存泄露，通过将一方设置为弱引用就可防止`retain`环的出现。比如说像代理，一般都是用的`weak`来进行声明的，这样就是防止循环应用它的写法大致有这几种：
```
// 1.AFNetworking的写法
__weak __typeof(&*self) weakSelf = self;
// 2.
__weak __typeof(self) weakSelf = self;
// 3.
__weak TTViewController *weakSelf = self;
// 4.
__weak id weakSelf = self;	
```

上述的四种写法在现有的版本都是正确的，可以随意使用。

## strongSelf

`strongSelf`就是强引用的`self`，使用该关键字声明后可以保证在使用时不会被销毁掉，来看一下下面的代码：
```
// 弱引用防止retain环
__weak __typeof(self)weakSelf = self;
AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
// 强引用防止使用时被销毁掉
   __strong __typeof(weakSelf)strongSelf = weakSelf;
   strongSelf.networkReachabilityStatus = status;
   if (strongSelf.networkReachabilityStatusBlock) {
       strongSelf.networkReachabilityStatusBlock(status);
   }
};
```

上面是开源网络框架`AFNetworking`的一段源代码，在处理网络请求时，因为网络请求的延时性，异步性很可能我们的`self`已经被销毁了，但是我们还需要请求继续执行下去，那么我们就不能让`self`被销毁掉，`strongSelf`正是来解决这一问题的，它保证了`self`不会在`block`执行时被销毁。

`strongSelf`的一般写法：
```
__strong __typeof(weakSelf)strongSelf = weakSelf;

```

