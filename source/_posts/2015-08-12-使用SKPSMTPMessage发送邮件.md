---
layout: post
title: "使用SKPSMTPMessage发送邮件"
date: 2015-08-12
comments: true
categories: iOS
tags: [iOS]
keywords: SMTP
publish: true
description: 使用SKPSMTPMessage发送邮件
---
在iOS开发里，可以发送邮件的第三方框架几乎没有，而iOS自带的框架又有诸多的不便，所以今天就来研究一下唯一算是比较好用的`SKPSMTPMessage `。

## SMTP协议
先来看一下什么是`SMTP`：

`SMTP`（Simple Mail Transfer Protocol）即简单邮件传输协议,它是一组用于由源地址到目的地址传送邮件的规则，由它来控制信件的中转方式。`SMTP`协议属于TCP/IP协议簇，它帮助每台计算机在发送或中转信件时找到下一个目的地。通过`SMTP`协议所指定的服务器,就可以把E-mail寄到收信人的服务器上了，整个过程只要几分钟。`SMTP`服务器则是遵循`SMTP`协议的发送邮件服务器，用来发送或中转发出的电子邮件。
	
简而言之`SMTP`就是邮件发送协议，而`SKPSMTPMessage`则是对协议的实现。
## SKPSMTPMessage
[SKPSMTPMessage](https://github.com/jetseven/skpsmtpmessage)是开源的邮件发送框架，由于框架比较老，所以没有使用`ARC`,但这丝毫不影响我们的使用。`SKPSMTPMessage`支持后台发送邮件，支持各种类型附件，功能还是相当强大的，我们可以随意定制UI，不受限制。
### 如何使用

* 首先是初始化以及邮件发送的基本设置

```
	SKPSMTPMessage *testMsg = [[SKPSMTPMessage alloc] init];
    //发送者
    testMsg.fromEmail = @"passion_wxm@163.com";
    //发送给
    testMsg.toEmail = @"413085129@qq.com";
    //抄送联系人列表，如：@"664742641@qq.com;1@qq.com;2@q.com;3@qq.com"
    testMsg.ccEmail = @"lanyuu@live.cn";
    //密送联系人列表，如：@"664742641@qq.com;1@qq.com;2@q.com;3@qq.com"
    testMsg.bccEmail = @"664742641@qq.com";
    //发送有些的发送服务器地址
    testMsg.relayHost = @"smtp.163.com";
    //需要鉴权
    testMsg.requiresAuth = YES;
    //发送者的登录账号
    testMsg.login = @"passion_wxm";
    //发送者的登录密码
    testMsg.pass = @"your_password";
    //邮件主题
    testMsg.subject = [NSString stringWithCString:"来自iphone socket的测试邮件" encoding:NSUTF8StringEncoding ];
    testMsg.wantsSecure = YES; // smtp.gmail.com doesn't work without TLS!
    // Only do this for self-signed certs!
    // testMsg.validateSSLChain = NO;
    testMsg.delegate = self;
```

* 然后我们可以设置主题附件等,这里需要注意的是，由于邮件发送操作受网络影响，比较耗时，所以发送操作要放在异步线程内。

```
	//正文
    NSDictionary *plainPart = [NSDictionary dictionaryWithObjectsAndKeys:@"text/plain",kSKPSMTPPartContentTypeKey,
                               @"This is a test message.\r\n支持中文。",kSKPSMTPPartMessageKey,@"8bit",kSKPSMTPPartContentTransferEncodingKey,nil];
    
    //设置文本附件
    NSData *mailData = [NSData dataWithContentsOfFile:self.mailPath];
    NSDictionary *txtPart = [[NSDictionary alloc ]initWithObjectsAndKeys:@"text/plain;\r\n\tx-unix-mode=0644;\r\n\tname=\"bug.txt\"",kSKPSMTPPartContentTypeKey, @"attachment;\r\n\tfilename=\"bug.txt\"", kSKPSMTPPartContentDispositionKey, [mailData encodeBase64ForData], kSKPSMTPPartMessageKey, @"base64", kSKPSMTPPartContentTransferEncodingKey,nil];
    
    //附件图片文件（联系人）
    NSString *vcfPath = [[NSBundle mainBundle] pathForResource:@"video.jpg" ofType:@""];
    NSData *vcfData = [NSData dataWithContentsOfFile:vcfPath];
    NSDictionary *vcfPart = [[NSDictionary alloc ]initWithObjectsAndKeys:@"text/directory;\r\n\tx-unix-mode=0644;\r\n\tname=\"video.jpg\"",kSKPSMTPPartContentTypeKey,
                             @"attachment;\r\n\tfilename=\"video.jpg\"",kSKPSMTPPartContentDispositionKey,[vcfData encodeBase64ForData],kSKPSMTPPartMessageKey,@"base64",kSKPSMTPPartContentTransferEncodingKey,nil];
    //附件音频文件
    NSString *wavPath = [[NSBundle mainBundle] pathForResource:@"push" ofType:@"wav"];
    NSData *wavData = [NSData dataWithContentsOfFile:wavPath];
    NSDictionary *wavPart = [[NSDictionary alloc ]initWithObjectsAndKeys:@"text/directory;\r\n\tx-unix-mode=0644;\r\n\tname=\"push.wav\"",kSKPSMTPPartContentTypeKey,
                             @"attachment;\r\n\tfilename=\"push.wav\"",kSKPSMTPPartContentDispositionKey,[wavData encodeBase64ForData],kSKPSMTPPartMessageKey,@"base64",kSKPSMTPPartContentTransferEncodingKey,nil];
    testMsg.parts = [NSArray arrayWithObjects:plainPart,txtPart,vcfPart,wavPart, nil];
    //发送
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [mailMsg send];
    });
```
* 最后就是一些代理方法了

```
- (void)messageSent:(SKPSMTPMessage *)message
{    
    //发送成功
    NSLog(@"delegate - message sent");
}

- (void)messageFailed:(SKPSMTPMessage *)message error:(NSError *)error
{    
    //发送失败 
    NSLog(@"delegate - error(%d): %@", [error code], [error localizedDescription]);
}
```

### 我遇到的坑
* 使用上面的写法会出现一个严重的问题，那就是发送等待过程中，APP会crash掉，起初我以为是发送数据量太大，后来发现其实不是这么回事，最后我在`stackoverflow`上面找到了[答案](http://stackoverflow.com/questions/16397120/app-crashes-after-sending-mail-using-smtp)，原因是在发送过程中，`SKPSMTPMessage`实例被释放掉，所以导致crash，最好的做法是将`SKPSMTPMessage`实例声明为强引用的属性，这样就没有问题了。

## 最后
愿大家玩的高兴。^_^

