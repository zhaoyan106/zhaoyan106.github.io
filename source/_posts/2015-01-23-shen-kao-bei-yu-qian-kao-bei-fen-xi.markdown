---
layout: post
title: "深拷贝与浅拷贝分析"
date: 2015-01-23 09:37:21 +0800
comments: true
categories: 
---
深拷贝:开辟新的存储空间,修改copy后的对象,不会影响本身

浅拷贝:只是指针的copy,修改copy后的对象,会影响本身.

对象要具备复制功能，必须实现<NSCopying>协议或者<NSMutableCopying>协议，常用的可复制对象有：

NSNumber、NSString、NSMutableString、NSArray、NSMutableArray、NSDictionary、NSMutableDictionary

对于以上的Foundation都遵循这个规律(arrayI为不可变数组,arrayM为可变数组)
		
	
	(lldb) po arrayI
	<__NSArrayI 0x7fb620704ba0>(
	
	)
	
	(lldb) po [arrayI copy]
	<__NSArrayI 0x7fb620704ba0>(
	
	)
	
	(lldb) po [arrayI mutableCopy]
	<__NSArrayM 0x7fb62042d050>(
	
	)

	(lldb) po arrayM
	<__NSArrayM 0x7fb620605570>(
	
	)
	
	(lldb) po [arrayM copy]
	<__NSArrayI 0x7fb620704ba0>(
	
	)
	
	(lldb) po [arrayM mutableCopy]
	<__NSArrayM 0x7fb620451ce0>(
	
	)


容器类对象深浅复制,对于容器而言，其元素对象始终是指针复制。如果需要元素对象也是对象复制，就需要实现深拷贝.

	NSArray *array = [NSArray arrayWithObjects:[NSMutableString stringWithString:@"first"],[NSStringstringWithString:@"b"],@"c",nil];
	// 里面的元素浅复制
	NSArray *deepCopyArray=[[NSArray alloc] initWithArray: array copyItems: YES];
	// 里面的元素深复制
	NSArray* trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:
	[NSKeyedArchiver archivedDataWithRootObject: array]];
	    
自己实现深拷贝的方法

NSDictionaryMutableDeepCopy.h
	
	
	#import <foundation /Foundation.h>
	
	@interface NSDictionary(MutableDeepCopy)
	
	- (NSMutableDictionary *)mutableDeepCopy;
	
	@end
	

NSDictionaryMutableDeepCopy.m
	
	#import "NSDictionaryMutableDeepCopy.h"
	
	@implementation NSDictionary(MutableDeepCopy)
	
	- (NSMutableDictionary *)mutableDeepCopy {
	    NSMutableDictionary *ret = [[NSMutableDictionary alloc]
	                                initWithCapacity:[self count]];
	    NSArray *keys = [self allKeys];
	    for (id key in keys) {
	        id oneValue = [self valueForKey:key];
	        id oneCopy = nil;
	        
	        if ([oneValue respondsToSelector:@selector(mutableDeepCopy)]) {
	            oneCopy = [oneValue mutableDeepCopy];
	        }
	        else if ([oneValue respondsToSelector:@selector(mutableCopy)]) {
	            oneCopy = [oneValue mutableCopy];
	        }
	        if (oneCopy == nil) {
	            oneCopy = [oneValue copy];
	        }
	        [ret setValue:oneCopy forKey:key];
	    }
	    
	    return ret;
	}
	
	@end

一般对象的拷贝,需要实现两个方法<NSCopying,NSMutableCopying>

浅拷贝

	-（id）copyWithZone:(NSZone *)zone{  
	             
	           Person *person = [[[self Class]allocWithZone:zone]init];  
	           p.name = _name;  
	           p.age = _age;  
	           return person;  
	       }  
	       
深拷贝

	-(void)copyWithZone:(NSZone *)zone{  
	            Person *person = [[[self Class]allocWithZone:zone]init];  
	            person.name = [_name copy];  
	            person.age = [_age copy];  
	            return person;  	              
	        }  


	       
