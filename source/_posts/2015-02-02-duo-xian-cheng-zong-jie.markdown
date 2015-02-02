---
layout: post
title: "多线程总结"
date: 2015-02-02 10:15:38 +0800
comments: true
categories: 
---
##一、线程

####1、线程中的术语
线程(线程)用于指代独立执行的代码段。

进程(process)用于指代一个正在运行的可执行程序,它可以包含多个线程。

任务(task)用于指代抽象的概念,表示需要执行工作。

####2、线程的模式
线程启动之后,线程就进入三个状态中的任何一个:运行(running)、就绪(ready)、阻塞(blocked)

`(因为底层内核的支持,操作对象(Operation objectis)可能创建线程更快。它们使用内核 里面常驻线程池里面的线程来节省创建的时间,而不是每次都创建新的线程。)`

####3、线程的使用
1、隐式的开启一个线程

	[NSThread detachNewThreadSelector:@selector(myThreadMainMethod:) toTarget:self withObject:nil];
	//使用 detachNewThreadSelector:toTarget:withObject:类方法来生成一个新的线程。
	
2、显示开启线程

	[[NSThread alloc] initWithTarget:self selector:@selector(myThreadMainMethod:) object:nil];
	[myThread start];
	//创建一个新的 NSThread 对象,并调用它的 start 方法。
	
3、子类化NSthread

	initWithTarget:selector:object:
	// 并重写run方法，main方法将成为线程入口
	
4、调用NSObject方法使用NSThread对象,它的线程当前真正运行,你可以给该线程发送 消息的唯一方法是在你应用程序里面的任何对象使用

	performSelector:onThread:withObject:waitUntilDone:
	
`（注意：目标线程必须在它的 run loop 里面运行，也就是说目标线程必须受到开启runloop，后面runloop会涉及）`

5、调用NSObject方法新生成一个脱离的线程,使用指定的方法作为新线程的主体入口点

	performSelectorInBackground:withObject:
	
`（注意：在selectore内部,你必须配置线程就像你在任何线程里面一样。比如,你可能需要设置一个自动 释放池(如果你没有使用垃圾回收机制),在你要使用它的时候配置线程的 run loop）`

####4、线程的主入口需要的配置
1、创建一个自动释放池

如果应用程序使用的垃圾回收机制,而不是管理的内存模型,那么创建一个自动 释放池不是绝对必要的。在垃圾回收的应用程序里面,一个自动释放池是无害的,而 且大部分情况是被忽略。允许通过个代码管理必须同时支持垃圾回收和内存管理模 型。在这种情况下,内存管理模型必须支持自动释放池,当应用程序运行垃圾回收的 时候,自动释放池只是被忽略而已。

2、设置异常处理

3、设置一个runloop

	+ (void)httpThreadWork{
	    @autoreleasepool {
	        // 线程命名，方便调试
	        [[NSThread currentThread] setName:@"eLongHttpThread"];
	        
	        // runloop一直运行
	        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
	        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
	        [runLoop run];
	    }
	}
	##二、runloop####1、runloop的基本介绍
使用 run loop 的目的是让你的线 程在有工作的时候忙于工作,而没工作的时候处于休眠状态（节能）。应用程序不需要显式的创建这些对象(run loop objects)，每个线程,包括程序的主线程都有与之对应的 run loop object。只有辅助线程（子线程）需要显式的运行它的。输入源(input source)和定时源 (timer source)。输入源传递异步事件,通常消息来自于其他线程或程序。定时源 则传递同步事件,发生在特定时间或者重复的时间间隔。两种源都使用程序的某一特 定的处理例程来处理到达的事件。![runloop icon](http://img.blog.csdn.net/20130703215237531?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3p6dmljdG9yeV90anNk/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####2、runloop模式
run loops 会生成关于 run loop 行为的通知 (notifications)。注册 run loop 观察者(run-loop Observers)可以收到这些通知, 并在线程上面使用它们来做额外的处理。在 run loop 运行过程中,`只有和模式相关的源才会被监视并允许他们传递事件消息。`(类似的,只有和模式相关的观察者会通知 run loop 的进程)。和其他模式关联的源只有在 run loop 运行在其模式下才会运行,否则处于暂停状态。	￼NSDefaultRunLoopMode(Cocoa)  // 最常用的一种模式，大多数的时候都使用他
	￼￼kCFRunLoopDefaultMode (Core￼￼Foundation)
	￼NSRunLoopCommonModes(Cocoa) // 一般模式，注意网络请求使用这个，否则主线程刷新UI，网络请求会暂停
	￼￼kCFRunLoopCommonModes (CoreFoundation)
####3、输入源事件来源取决于输入源的种类:基于端口的输入源和自定义输入源。基于端口的输入源监听程序相应的端口。自定义输入源则监 听自定义的事件源。至于 run loop,它不关心输入源的是基于端口的输入源还是自定义的输入源。系统会实现两种输入源供你使用。

两类输入源的区别在于如何显示: 基于端口的输入源由内核自动发送,而自定义的则需要人工从其他线程发送。

1、基于端口的输入源Cocoa和CoreFoundation内置支持使用端口相关的对象和函数来创建的基于端口的源。例如,在Cocoa 里面你从来不需要直接创建输入源。你只要简单的创建端口对象,并使用NSPort的方法把该端口添加到runloop。端口对象会自己处理创建和配置输入源。

2、自定义输入源，在Core Foundation,你必须人工创建端口和它的runloop源。在两种情况下，你都可以使用端口相关的函数(CFMachPortRef,CFMessagePortRef,CFSocketRef)来创建合适的对象。

3、Cocoa执行Selector的源

	￼performSelectorOnMainThread:withObject:waitUntilDone:
	￼￼performSelectorOnMainThread:withObject:waitUntilDone:modes
	
	performSelector:onThread:withObject:waitUntilDone:
	performSelector:onThread:withObject:waitUntilDone:modes
	
	performSelector:withObject:afterDelay:
	performSelector:withObject:afterDelay:inModes:
	
	￼￼￼￼￼cancelPreviousPerformRequestWithTarget:
	￼￼￼￼￼cancelPreviousPerformRequestWithTarget:selector:object:
	
4、定时源

如果定时器被设定在某一特定时间开始并5秒重复一次,那么定时器会在那个特定时间后5秒启动,即使在那个特定的触发时间延迟了。如果定时器被延迟以至于它错过了一个或多个触发时间,那么定时器会在下一个最近的触发事件启动,而后面会按照触发间隔正常执行。

####4、何时使用runloop
1、仅当在为你的程序创建辅助线程的时候,你才需要显式运行一个 run loop

2、对于辅助线程,你需要判断一个run loop是否是必须的。如果是必须的,那么你要自己配置并启动它。你不需要在任何情况下都去启动一个线程的run loop。比如,你使用线程来处理一个预先定义的长时间运行的任务时,你应该避免启动runloop。

3、Runloop在你要和线程有更多的交互时才需要,比如以下情况:

使用端口或自定义输入源来和其他线程通信

使用线程的定时器

Cocoa中使用任何performSelector...的方法

使线程周期性工作

####5、runloop对象
1、获取runloop对象

为了获得当前线程的 run loop,Cocoa 程序中,使用 NSRunLoop 的 currentRunLoop 类方法来检索一个NSRunLoop 对象。

2、配置runloop对象

在你在辅助线程运行 run loop 之前,你必须至少添加一输入源或定时器给它。 如果 run loop 没有任何源需要监视的话,它会在你启动之际立马退出。

`（当前长时间运行的线程配置 run loop 的时候,最好添加至少一个输入源到 run loop 以接收消息。虽然你可以使用附属的定时器来进入 run loop,但是一旦定时器 触发后,它通常就变为无效了,这会导致 run loop 退出。虽然附加一个循环的定时 器可以让 run loop 运行一个相对较长的周期,但是这也会导致周期性的唤醒线程, 这实际上是轮询(polling)的另一种形式而已。与之相反,输入源会一直等待某事 件发生,在事情导致前它让线程处于休眠状态。）`

3、启动runloop
有几种方式可以启动runloop,包括以下这些:

无条件的

无条件的进入 run loop 是最简单的方法,但也最不推荐使用的。因为这样会使你的线程处在一个永久的循环中,这会让你对 run loop本身的控制很少。你可以添加或删除输入源和定时器,但是退出 run loop 的唯一方法是杀死它。没有任何办法 可以让这run loop运行在自定义模式下。

设置超时时间

替代无条件进入 run loop 更好的办法是用预设超时时间来运行 run loop,这样 run loop 运作直到某一事件到达或者规定的时间已经到期。如果是事件到达,消息 会被传递给相应的处理程序来处理,然后 run loop 退出。你可以重新启动 run loop 来等待下一事件。如果是规定时间到期了,你只需简单的重启 run loop 或使用此段 时间来做任何的其他工作。

特定模式

除了超时机制,你也可以使用特定的模式来运行你的 run loop。模式和超时不是 互斥的,他们可以在启动 run loop 的时候同时使用。

4、退出runloop

给runloop设置超时时间

通知runloop停止
