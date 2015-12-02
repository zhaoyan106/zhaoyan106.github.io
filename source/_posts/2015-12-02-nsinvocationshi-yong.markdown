---
layout: post
title: "NSInvocation使用"
date: 2015-12-02 12:25:04 +0800
comments: true
categories: 
---
#####NSInvocation可以对某个对象发送消息(多参数使用),可以处理参数和返回值
1、创建NSInvocation对象需要一个方法签名

	NSMethodSignature * sig = [self methodSignatureForSelector:sel]
2、利用得到的这个方法签名创建NSInvocation对象
	
	NSInvocation *inv = [NSInvocation invocationWithMethodSignature:sig];
	
3、通过调用NSInvocation的相关方法设置消息接收对象及参数

	[inv setTarget:self];
	[inv setSelector:sel];
	
4、参数解析设置传参

	va_list args;
	va_start(args, sel);
	// 通过方法签名能够获取参数个数及参数类型
	// [sig numberOfArguments]
	// char *type = (char *)[sig getArgumentTypeAtIndex:index];
	// 根据匹配出的类型通过va_arg取出入参
	// int arg = va_arg(args, int);
	[inv setArgument:&arg atIndex:index];
5、调用执行方法

	[inv invoke]
6、获取方法返回值

	NSUInteger length = [sig methodReturnLength];
	// 如果length为0方法返回为void
    char *type = (char *)[sig methodReturnType];
    // 根据type匹配出对应的类型，得到具体返回值
    id ret = nil;
	[inv getReturnValue:&ret];
	


