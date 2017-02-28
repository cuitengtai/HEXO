---
layout: post
title: "UITableView的分割线"
date: 2015-08-26
comments: true
categories: iOS
tags: [iOS]
keywords: Cell的分割线
publish: true
description: UITableView的分割线
---
在iOS7与iOS8中的UITableView的分割线的一些设置问题。

##首先
在iOS中UITableView的分割线默认是左边距离边距15像素，但是有时我们需要它左对齐。
这是正常的分割线：

![正常](http://img17.poco.cn/mypoco/myphoto/20150826/20/17832875620150826201621072.png?375x689_130
)

我们的需求：

![需求](http://img17.poco.cn/mypoco/myphoto/20150826/20/17832875620150826201715025.png?375x689_130
)
##Do It

###在iOS7

在iOS7上，我们有多种方式来解决这个问题。

* 使用代码的方式
	
	```
	self.tableView.separatorInset = UIEdgeInsetsZero;
	```
	
* 如果使用了xib或者是SB

![xib-sb](http://img17.poco.cn/mypoco/myphoto/20150826/20/17832875620150826202654013.png?239x59_130)

* 或者直接把分割线样式设置为`UITableViewCellSeparatorStyleNone`,然后自己在`cell`的底部添加一个高度为`1`的`UIView`

...

###在iOS8
不幸的是在iOS8上，上述的两种方法都不行，为什么呢？其实是`layoutMargins`这个属性在作怪，这是iOS8新添加的一个属性，下面是官方的介绍：

![layoutMargins](http://img17.poco.cn/mypoco/myphoto/20150826/20/17832875620150826203706040.png?710x429_130)

那么，我们就需要使用下面这段代码了：
	
	-(void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath
	{
    
    	if ([tableView respondsToSelector:@selector(setSeparatorInset:)]) {
        	[tableView setSeparatorInset:UIEdgeInsetsZero];
    	}
    
    	if ([tableView respondsToSelector:@selector(setLayoutMargins:)]) {
       	 [tableView setLayoutMargins:UIEdgeInsetsZero];
   		}
    
    	if ([cell respondsToSelector:@selector(setLayoutMargins:)]) {
        	[cell setLayoutMargins:UIEdgeInsetsZero];
   		}
	}
	
`layoutMargins`在子控件与父控件布局中起了决定性的作用。

但是这一段代码会在每个cell绘制时候频繁调用，这也就意味着性能的下降，所以还有一种方法来实现：

    self.tableView.separatorInset = UIEdgeInsetsZero;
    self.tableView.layoutMargins = UIEdgeInsetsZero;
##最后
愿大家玩的高兴。^_^

