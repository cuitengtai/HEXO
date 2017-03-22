---
layout: post
title: "iOS开发神器Reveal使用指南"
date: 2017-03-04
comments: true
categories: Mac工具
tags: [Mac工具]
keywords: iOS开发神器Reveal使用指南
publish: true
description:  iOS开发神器Reveal使用指南
---
>作为一名iOS开发者，想必Reveal大家都是知道的。它提供的视图层次浏览功能，给开发过程中定位错误带来了极大的便利，甚至可以通过越狱设备逆向别人的APP，浏览其他APP的视图结构。自从前阵子升级到Xcode8.x之后，Reveal1.6.3就再也用不了了，起初以为是系统兼容性问题，后来等到了Reveal2这个蛋疼的问题依然没解决，最近总算是成功的解决了这个问题，现在Reveal7已经出来了，不得不说，这个版本号真的是醉了，跟坐火箭一样。Reveal的官网在[这里](https://revealapp.com/)，由于Reveal版本号更新太快，不同版本之间的接入操作会有一些差异，如果使用中遇到麻烦，一定要找对应版本的解决办法，这里就以Reveal4为例介绍一下使用步骤。

![Reveal](http://om6homgqk.bkt.clouddn.com/WX20170304-180852@2x.png)

<!-- more -->
## 安装

Reveal可以免费试用14天，如果你不差钱可以买[正版](https://revealapp.com/buy/)支持一下，也就是一年59刀。当然了，如果你不打算这样做，可以去下载[破解版](http://xclient.info/s/reveal.html)。
## 接入自己项目
Reveal可以接入自己的项目，也可以通过逆向查看其它APP，这里主要介绍Reveal如何连接模拟器及真机。
Reveal接入项目有三种方法：

```
1.通过CocoaPods导入SDK接入
2.通过Xcode断点接入
3.通过导入framework接入
```
这里的第一种和最后一种都不太合适，对项目造成了侵入，所以我毫不犹豫选择了第二种，只需要添加一个断点，并简单配置一下就可以。

### 通过断点连接
首先，打开Xcode，添加一个断点，如下：

![添加断点](http://om6homgqk.bkt.clouddn.com/断点.png
)
然后编辑断点，配置成如下图所示：

![编辑断点](http://om6homgqk.bkt.clouddn.com/编辑断点.png)

`Debugger Command`下面一栏需要填上

```
 expr (Class)NSClassFromString(@"IBARevealLoader") == nil ? (void *)dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework/RevealServer", 0x2) : ((void*)0)
```
这里需要注意如果你使用的是Reveal4，你可以使用这一段代码，如果是其他版本会不一样。
使用模拟器运行项目，打开Reveal，如果看到这种情况，那就成功一大步。

![关联成功图标](http://om6homgqk.bkt.clouddn.com/success.png)

如果你很不幸，没有看到这个图标，也不要灰心，点击下面这个，会有官方提供的解决办法。

![官方解决问题](http://om6homgqk.bkt.clouddn.com/点击去官网.png)

这里需要注意的是，不同的Reveal版本，点击后跳转的官方网页内容并不一样，连接方法稍有区别，仔细看官方的说明即可。
![官方解决问题指导](http://om6homgqk.bkt.clouddn.com/reveal解决问题指导.png)

![Hosts问题](http://om6homgqk.bkt.clouddn.com/最后一个错误.png)

如果你遇到了这个问题，说明你本机的hosts文件被更改了，可以使用[SwitchHosts](https://github.com/oldj/SwitchHosts/downloads)更改一下，如图。

![更改Hosts](http://om6homgqk.bkt.clouddn.com/hosts.png)
具体配置：

```
127.0.0.1   localhost
255.255.255.255 broadcasthost
::1             localhost 
```
具体的内容可以看一下[这个博客](http://www.cnblogs.com/fengtengfei/p/5100005.html)。上面说的这些可以连接模拟器，至于真机，Reveal4暂时没有发现使用断点连接的办法，Reveal7则可以，需要在项目中增加一段脚本。可以仔细阅读官网的[Load the Reveal Server via an Xcode Breakpoint](http://support.revealapp.com/kb/getting-started/load-the-reveal-server-via-an-xcode-breakpoint)，这篇文章是关于Reveal7的，跟上面所述会有些不同。
## 其他
由于之前用Reveal试用版30天，最后导致破解版无法使用，一打开APP就弹出激活框，折腾了好久，用Clean my mac删除也不行，后来无意间我发现在`/Users/Shared/Reveal`这里居然有个`Reveal`文件夹，打开一看，卧槽，这里放的就是激活许可相关的那个文件，心里一万只可爱的小动物飞过😭，小婊砸，二话不说，直接把这个文件干掉，从此又可以愉快的使用破解版啦😝。

## 最后
最后，特别想吐槽一下Reveal，虽然好用，但是配置起来很闹心，各种踩坑，如果你遇到什么坑，一定要告诉我哦，祝大家踩坑愉快！

