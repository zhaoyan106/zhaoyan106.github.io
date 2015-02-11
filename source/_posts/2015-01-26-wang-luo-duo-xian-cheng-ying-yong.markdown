---
layout: post
title: "网络多线程应用"
date: 2015-01-26 16:46:41 +0800
comments: true
categories: 
---
##一、发送网络请求所需


网络请求中一定会用的类:NSOperation、NSOperationQueue、NSRunLoop、NSURLRequest、NSURLConnection，这些都是发送网络请求必须得，其中NSOperation、NSOperationQueue、NSRunLoop这些是用在多线程发送网络请求中的。

####1、NSOperation
######NSOperation有三种使用方法：NSInvocationOperation、NSBlockOperation、自定义NSOperation（NSOperation的子类）

1、NSInvocationOperation：创建operation object。如果你已经有现有的方法来执行需要的任务，就可以使用这个类。

2、NSBlockOperation：可以直接使用的类，用来并发地执行一个或多个block对象。operation object使用“组”的语义来执行多个block对象，所有相关的block都执行完成之后，operation object才算完成。

3、自定义NSOperation：继承NSOperation可以完全控制operation object的实现，包括修改操作执行和状态报告的方式。
######operation objects都支持以下关键特性：

1、支持建立基于图的operation objects依赖。可以阻止某个operation运行，直到它依赖的所有operation都已经完成。

2、支持可选的completion block，在operation的主任务完成后调用。

3、支持应用使用KVO通知来监控operation的执行状态。

4、支持operation优先级，从而影响相对的执行顺序

5、支持取消，允许你中止正在执行的任务
######自定义NSOperation
NSOperation 类提供通用的子类继承点，而且实现了许多重要的基础设施来处理依赖和KVO通知。继承所需的工作量主要取决于你要实现非并发还是并发的operation。

定义非并发operation要简单许多，只需要执行主任务，并正确地响应取消事件;NSOperation 处理了其它所有事情。对于并发operation，你必须替换某些现有的基础设施代码。

串行：

1、每个operation对象至少需要实现以下方法：

2、自定义initialization方法：初始化，将operation 对象设置为已知状态

3、自定义main方法：执行你的任务

4、你也可以选择性地实现以下方法：

5、main方法中需要调用的其它自定义方法

6、Accessor方法：设置和访问operation对象的数据

7、dealloc方法：清理operation对象分配的所有内存

8、NSCoding 协议的方法：允许operation对象archive和unarchive

并行：

	- (void)start;

（`必须`）所有`并发`操作都必须覆盖这个方法，以自定义的实现替换默认行为。手动执行一个操作时，你会调用start方法。因此你对这个方法的实现是操作的起点，设置一个线程或其它执行环境，来执行你的任务。你的实现在任何时候都绝对不能调用super。

	- (void)main;

（`可选`）这个方法通常用来实现operation对象相关联的任务。尽管你可以在start方法中执行任务，使用main来实现任务可以让你的代码更加清晰地分离设置和任务代码

	isExecuting、isFinished

（`必须`）并发操作负责设置自己的执行环境，并向外部client报告执行环境的状态。因此并发操作必须维护某些状态信息，以知道是否正在执行任务，是否已经完成任务。使用这两个方法报告自己的状态。
这两个方法的实现必须能够在其它多个线程中同时调用。另外这些方法报告的状态变化时，还需要为相应的key path产生适当的KVO通知。

	isConcurrent

（必须）标识一个操作是否并发operation，覆盖这个方法并返回YES

####2、NSOperationQueue

Operation Queues是Cocoa版本的并发dispatch queue，由 NSOperationQueue 类实现。dispatch queue总是按先进先出的顺序执行任务，而 Operation Queues 在确定任务执行顺序时，还会考虑其它因素。最主要的一个因素是指定任务是否依赖于另一个任务的完成。你在定义任务时配置依赖性，从而创建复杂的任务执行顺序图。

提交到Operation Queues的任务必须是 NSOperation 对象，operation object封装了你要执行的工作，以及所需的所有数据。由于 NSOperation 是一个抽象基类，通常你需要定义自定义子类来执行任务。不过Foundation framework自带了一些具体子类，你可以创建并执行相关的任务。

Operation objects会产生key-value observing(KVO)通知，对于监控任务的进程非常有用。虽然operation queue总是并发地执行任务，你可以使用依赖，在需要时确保顺序执行

##二、封装NSURLRequest
通过一个继承自NSObject的类，提供相关的类方法，快速创建网络请求。

    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
    NSMutableString *targetUrl = [[NSMutableString alloc] init];
    [request setValue:self.channelID forHTTPHeaderField:@"channelid"];
    [request setValue:self.version forHTTPHeaderField:@"version"];
    [request setValue:self.deviceID forHTTPHeaderField:@"deviceid"];
    [request setValue:self.authCode forHTTPHeaderField:@"AuthCode"];
	return request;
##三、自定义NSOperation
重写start方法

	// 状态机
	typedef enum {
    	eLongOperationPausedState      = -1,   // 暂停
    	eLongOperationReadyState       = 1,    // 准备就绪
    	eLongOperationExecutingState   = 2,    // 正在运行
    	eLongOperationFinishedState    = 3,    // 完成
	}elongOperationState;
	
	@property (nonatomic, assign) elongOperationState state;    // 状态机
	
	@property (nonatomic, strong, readonly) NSMutableURLRequest *currentReq;    // 网络请求
	
	@property (nonatomic,retain) NSRecursiveLock *lock;    // 数据锁
	
	// 创建一个单例线程用来处理所有请求
	+ (NSThread *)httpRequest{
    	static NSThread *httpThread = nil;
    	static dispatch_once_t oncePredicate;
    	dispatch_once(&oncePredicate, ^{
        	httpThread = [[NSThread alloc] initWithTarget:self selector:@selector(httpThreadWork) object:nil];
        	[httpThread start];
    	});
    	return httpThread;
	}
	
	// 子线程显式创建runloop	
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

	// 创建网络请求，初始化数据锁，设置状态机为ready状态
	- (id)initWithRequest:(NSMutableURLRequest *)request {
	    if (self = [super init]) {
	        self.currentReq	= request;
	        NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
	        self.lock = lock;
	        self.lock.name = @"elong.httpOperation.lock";
	        _state = eLongOperationReadyState;
	    }
	    return self;
	}
	
	// 重写start方法，当自定义operation加入到队列中自动并发调用
	- (void)start{
	    [self.lock lock];
	    if ([self isCancelled]) {
	        [self performSelector:@selector(cancelConnection) onThread:[[self class] httpRequest] withObject:nil waitUntilDone:NO modes:[NSArray arrayWithObject:NSRunLoopCommonModes]];
	    }else if ([self isReady]) {
	        self.state = eLongOperationExecutingState;
	        [self performSelector:@selector(operationDidStart) onThread:[[self class] httpRequest] withObject:nil waitUntilDone:NO modes:[NSArray arrayWithObject:NSRunLoopCommonModes]];
	    }
	    [self.lock unlock];
	}
	
	// 创建NSURLConnection并开始
	- (void)operationDidStart {
    	[self.lock lock];
    	if (![self isCancelled]) {
    		//  startImmediately:NO 一定要设置为NO否则无法修改runloop
        	NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:self.currentReq delegate:self startImmediately:NO];
        	self.connection = connection;
        	NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        	// 这里需要设置RunLoop模式为NSRunLoopCommonModes，否则会影响主线程刷新UI会暂停网络,将connection加入runloop或者
        	[self.connection scheduleInRunLoop:runLoop forMode:NSRunLoopCommonModes];
        	[self.connection start];
    	}
    	[self.lock unlock];
	}
##四、利用NSOperationQueue开启并发网络请求

	- (id)init{
    	self = [super init];
    	if (!self) {
        	return nil;
    	}
    	// 创建网络请求队列
    	static dispatch_once_t oncePredicate;
    	dispatch_once(&oncePredicate, ^{
        	httpQueue = [[NSOperationQueue alloc] init];    // 创建请求队列
        	httpQueue.maxConcurrentOperationCount = 4;      // 最大并发数
    	});
    	self.callBackObjectDict = [[NSMutableDictionary alloc] initWithCapacity:3];
    	NSRecursiveLock *lock = [[NSRecursiveLock alloc] init];
    	self.lock = lock;
    	self.lock.name = @"elong.httpReq.lock";
    	return self;
	}
	
	- (void)startWith:(eLongHTTPRequestOperation *)httpRequestOperation{
    	[httpQueue addOperation:httpRequestOperation];
	}
