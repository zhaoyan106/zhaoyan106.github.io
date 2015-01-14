---
layout: post
title: "利用crash堆栈信息定位典型的野指针错误"
date: 2015-01-13 17:13:23 +0800
comments: true
categories: 
---
通过堆栈信息，确定了代码的具体位置

	[self.delegate respondsToSelector:@selector(httpConnectionDidFailed:withError:)]
	
通过这个信息，猜测是delegate这个对象在网络请求回来之前已经释放掉，导致野指针错误。
控制器的具体代码如下：

	@interface TrainSearchListVC ()
	// 这个HttpUtil是封装的网络工具类
	@property (nonatomic, strong) HttpUtil *checkOrderUtil;
	@property (nonatomic, strong) HttpUtil *cancelOrderUtil;
	
	@end
	

	- (void)dealloc
	{
	    [self unregisterNotification];
		// 首先要取消请求
		// 然后设置为nil (MRC需要release)
		***因为没有设置为nil 导致野指针为题***
	    [self.checkOrderUtil cancel];
	    [self.cancelOrderUtil cancel];
	    self.checkOrderUtil = nil;
	    self.cancelOrderUtil = nil;
	    [_tableViewList setDataSource:nil];
	    [_tableViewList setDelegate:nil];
	}
	
	[_checkOrderUtil requestWithURLString:url Content:paramJson StartLoading:YES EndLoading:YES Delegate:self]; 设置了网络请求代理，但是当网络数据返回的时候，代理已经销毁了，出现也只真为题。
	
提供一个计算进制的python方法：

	EBJ1297:~ zhaoyan$ python
	Python 2.7.6 (default, Sep  9 2014, 15:04:36) 
	[GCC 4.2.1 Compatible Apple LLVM 6.0 (clang-600.0.39)] on darwin
	Type "help", "copyright", "credits" or "license" for more information.
	>>> hex(0x000f45f5 - 136693)
	'0xd3000'
	>>> hex(0x00604963 - 5712227)
	'0x92000'
	>>> hex(0x0059e963 - 5712227)
	'0x2c000'
	>>> 


