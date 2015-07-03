---
layout: post
title: "Effective Objective-C 2.0总结"
date: 2015-05-13 15:32:57 +0800
comments: true
categories: 
---
######第三条：多用字面量语法，少用预制等价的方法。
`需要注意 NSMutableArray *arrayM = [@[@"1",@"2",@"3"] mutableCopy];`
######第四条：多用类型常量，少用#define预处理指令
	#define ANIMATION_DURATION 0.3
	Static const NSTimeInterval KAnimationDuration = 0.3;

######第七条：在对象内部尽量直接访问实例变量
`在对象内部读取时直接访问实例变量,设置时使用self.（懒加载除外）`

  1、如果不使用self.可能会造成copy属性不能实现。
  
  2、KVO不触发
  
  3、直接读取效率较高
  
  4、子类可能重写get方法，使用_更安全
  
######第12条：理解消息转发机制 (✨✨✨✨✨✨)
`unrecongiezed selector send to instance 0x87,当找不到对应方法时会实现消息转发机制`

对象在收到无法解读的消息后，首先会调用

	+ （BOOL)resolveInstanceMehtod:(SEL)selector



######第13条：用“方法调配技术” 调试 “黑盒方法”(✨✨✨✨✨✨)
	// 交换方法实现
	void method_exchangeImplementations(<#Method m1#>, <#Method m2#>)
	// 获取对象方法
	Method class_getInstanceMethod(<#__unsafe_unretained Class cls#>, <#SEL name#>)

######第14条：理解“类对象”的用意

	typedef struct objc_object{
	    Class isa;
	} *id;

######第15条：多用派发队列，少用同步锁
同步锁效率较低
######第16条：多用GCD,少用performSelector系列方法
在使用performSelector方法时容易引起ARC不自动使用，造成内存泄露。



