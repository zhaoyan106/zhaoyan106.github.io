---
layout: post
title: "Best Practices"
date: 2015-04-30 16:11:22 +0800
comments: true
categories: 
---
Preferred:

	if (!error) {
		return success;
	}
	
Not preferred:

	if (!error)
		return success;
		
	or
	
	if (!error) return success;
	

Preferred:

	if ([myValue isEqual:@42]) { ...

Not preferred:

	if ([@42 isEqual:myValue]) { ...
	

Preferred:

	if (nil == myValue) { ...
	
Not preferred:

	if (myValue == nil) { ...
	
Preferred:

	if (someObject) { ...
	if (![someObject boolValue]) { ...
	if (!someObject) { ...
	
Not preferred:

	if (someObject == YES) { ... // Wrong
	if (myRawValue == YES) { ... // Never do this.
	if ([someObject boolValue] == NO) { ...

Preferred:

	- (void)someMethod {
	  if (![someOther boolValue]) {
	      return;
	  }
	
	  //Do something important
	}

Not preferred:

	- (void)someMethod {
	  if ([someOther boolValue]) {
	    //Do something important
	  }
	}

Preferred:

	BOOL nameContainsSwift  = [sessionName containsString:@"Swift"];
	BOOL isCurrentYear      = [sessionDateCompontents year] == 2014;
	BOOL isSwiftSession     = nameContainsSwift && isCurrentYear;
	
	if (isSwiftSession) {
	    // Do something very cool
	}
	
Not preferred:

	// 不建议写在一起，可读性差，容易出错
	
Preferred:

	result = object ? : [self createObject];
	
Not preferred:

	result = object ? object : [self createObject];
	
Preferred:

	NSError *error = nil;
	if (![self trySomethingWithError:&error]) {
	    // Handle Error
	}
	
Preferred:

	switch (menuType) {
	    case ZOCEnumNone:
	        // ...
	        break;
	    case ZOCEnumValue1:
	        // ...
	        break;
	    case ZOCEnumValue2:
	        // ...
	        break;
	}
	// 如果有枚举没处理，会提示
	
Not preferred:

	switch (condition) {
	    case 1:
	        // ...
	        break;
	    case 2: {
	        // ...
	        // Multi-line example using braces
	        break;
	       }
	    case 3:
	        // ...
	        break;
	    default: 
	        // ...
	        break;
	}

Preferred:

	typedef NS_ENUM(NSUInteger, ZOCMachineState) {
	    ZOCMachineStateNone,
	    ZOCMachineStateIdle,
	    ZOCMachineStateRunning,
	    ZOCMachineStatePaused
	}

Preferred:

	static NSString * const ZOCCacheControllerDidClearCacheNotification = @"ZOCCacheControllerDidClearCacheNotification";
	static const CGFloat ZOCImageThumbnailHeight = 50.0f;

Not preferred:

	#define CompanyName @"Apple Inc."
	#define magicNumber 42

Preferred:

	NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
	NSDictionary *productManagers = @{@"iPhone" : @"Kate", @"iPad" : @"Kamal", @"Mobile Web" : @"Bill"};
	NSNumber *shouldUseLiterals = @YES;
	NSNumber *buildingZIPCode = @10018;
	
Not preferred:

	NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
	NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
	NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
	NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
	
Preferred:

	+ (instancetype)sharedInstance
	{
	   static id sharedInstance = nil;
	   static dispatch_once_t onceToken = 0;
	   dispatch_once(&onceToken, ^{
	      sharedInstance = [[self alloc] init];
	   });
	   return sharedInstance;
	}
	
Not preferred:

	+ (instancetype)sharedInstance
	{
	    static id sharedInstance;
	    @synchronized(self) {
	        if (sharedInstance == nil) {
	            sharedInstance = [[MyClass alloc] init];
	        }
	    }
	    return sharedInstance;
	}

Preferred:
	
	view.backgroundColor = [UIColor orangeColor];
	[UIApplication sharedApplication].delegate;

Not preferred:

	[view setBackgroundColor:[UIColor orangeColor]];
	UIApplication.sharedApplication.delegate;
	
Lazy Loading

	- (NSDateFormatter *)dateFormatter {
	  if (!_dateFormatter) {
	    _dateFormatter = [[NSDateFormatter alloc] init];
	        NSLocale *enUSPOSIXLocale = [[NSLocale alloc] initWithLocaleIdentifier:@"en_US_POSIX"];
	        [dateFormatter setLocale:enUSPOSIXLocale];
	        [dateFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ss.SSSSS"];
	  }
	  return _dateFormatter;
	}
	
