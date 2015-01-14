---
layout: post
title: "常用工具方法总结"
date: 2014-11-24 10:02:28 +0800
comments: true
categories: 
---
###一.文件操作常用方法总结:
####1.写入文件
    + (BOOL)writeFileName:(NSString *)fileName data:(id)result{
       NSString *pathName =  [LSCacheFile filePath:fileName];
        if ([LSCacheFile isExistsFile:pathName]) {
            [[NSFileManager defaultManager] removeItemAtPath:pathName error:nil];
        }
        return [LSCacheFile writeFile:pathName object:result];
    }

####2.读出文件
    + (id)readFileName:(NSString *)fileName{
        
        if (!fileName) {
            return nil;
        }
        if(![LSCacheFile isExistsFile:fileName]){
            return nil;
        }
    
        id object = [LSCacheFile readFile:[LSCacheFile filePath:fileName]];
        if (!object) {
            return nil;
        }
        return object;
    }
####3.计算文件大小
    + (NSString *)folderSizeStringAtPath:(NSString *)folderPath
    {
        long long folderSize = [LSCacheFile folderSizeAtPath:folderPath];
        return [self sizeStringFromSizeLong:folderSize];
    }
    
    // 计算文件夹下文件的总大小
    + (long long)folderSizeAtPath:(NSString*)folderPath{
        NSFileManager* manager = [NSFileManager defaultManager];
        if (![manager fileExistsAtPath:folderPath]) return 0;
        NSEnumerator *childFilesEnumerator = [[manager subpathsAtPath:folderPath] objectEnumerator];
        NSString* fileName;
        long long folderSize = 0;
        while ((fileName = [childFilesEnumerator nextObject]) != nil){
            NSString* fileAbsolutePath = [folderPath stringByAppendingPathComponent:fileName];
            folderSize += [LSCacheFile fileSizeAtPath:fileAbsolutePath];
        }
        return folderSize;
    }
    // 根据文件大小返回对应单位
    +(NSString*)sizeStringFromSizeLong:(long long) folderSize{
        if (folderSize < 1024) {
            return @"0K";
        }else if(folderSize/1024.0 < 1024){
            return [NSString stringWithFormat:@"%.2fK",folderSize/1024.0];
        }else if(folderSize/1024.0/1024.0 < 1024){
            return [NSString stringWithFormat:@"%.2fM",folderSize/1024.0/1024.0];
        }else if(folderSize/1024.0/1024.0/1024.0 < 1024){
            return [NSString stringWithFormat:@"%.2fG",folderSize/1024.0/1024.0/1024.0];
        }
        return @"";
    }
####4.格式化size单位
    + (NSString *)formatFileSize:(int)fileSize {
        float size = fileSize;
        if (fileSize < 1023) {
            return([NSString stringWithFormat:@"%i bytes",fileSize]);
        }
        
        size = size / 1024.0f;
        if (size < 1023) {
            return([NSString stringWithFormat:@"%1.2f KB",size]);
        }
        
        size = size / 1024.0f;
        if (size < 1023) {
            return([NSString stringWithFormat:@"%1.2f MB",size]);
        }
        
        size = size / 1024.0f;
        return [NSString stringWithFormat:@"%1.2f GB",size];
    }
####5.获取目录下的文件大小(返回字节)
    + (long long)folderSizeAtPath:(NSString*)folderPath{
        NSFileManager* manager = [NSFileManager defaultManager];
        if (![manager fileExistsAtPath:folderPath]) return 0;
        NSEnumerator *childFilesEnumerator = [[manager subpathsAtPath:folderPath] objectEnumerator];
        NSString* fileName;
        long long folderSize = 0;
        while ((fileName = [childFilesEnumerator nextObject]) != nil){
            NSString* fileAbsolutePath = [folderPath stringByAppendingPathComponent:fileName];
            folderSize += [LSCacheFile fileSizeAtPath:fileAbsolutePath];
        }
        return folderSize;
    }
####6.判断文件是否存在
    + (BOOL)isExistsFile:(NSString *)filepath{
        NSFileManager *filemanage = [NSFileManager defaultManager];
        return [filemanage fileExistsAtPath:[LSCacheFile filePath:filepath]];
    }
####7.删除缓存文件
    +(void)deleteCacheFile:(NSString *)filepath
    {
        if([[NSFileManager defaultManager] fileExistsAtPath:filepath isDirectory:NO])
        {
            [[NSFileManager defaultManager] removeItemAtPath:filepath error:nil];
        }
    }
####8.创建文件夹
    + (BOOL)createFolder:(NSString*)folderPath isDirectory:(BOOL)isDirectory {
        NSString *path = nil;
        if(isDirectory) {
            path = folderPath;
        } else {
            path = [folderPath stringByDeletingLastPathComponent];
        }
        
        if(folderPath && [[NSFileManager defaultManager] fileExistsAtPath:path] == NO) {
            NSError *error = nil;
            BOOL ret;
            
            ret = [[NSFileManager defaultManager] createDirectoryAtPath:path
                                            withIntermediateDirectories:YES
                                                             attributes:nil
                                                                  error:&error];
            if(!ret && error) {
                NSLog(@"create folder failed at path '%@',error:%@,%@",folderPath,[error localizedDescription],[error localizedFailureReason]);
                return NO;
            }
        }
        
        return YES;
    }
####9.得到用户document中的一个路径
    + (NSString*)getPathInUserDocument:(NSString*) aPath{
        NSString *fullPath = nil;
        NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        if ([paths count] > 0)
        {
            fullPath = (NSString *)[paths objectAtIndex:0];
            if(aPath != nil && [aPath compare:@""] != NSOrderedSame)
            {
                fullPath = [fullPath stringByAppendingPathComponent:aPath];
            }
        }
        return fullPath;
    }
####10.文件创建日期
    + (NSDate*)dateOfFileCreateWithFolderName:(NSString *)folderName cacheName:(NSString *)cacheName
    {
        NSString *folder = [[NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:folderName];
        NSString *filePath = [folder stringByAppendingPathComponent:cacheName];
        NSError *error;
        NSDictionary *attributes = [[NSFileManager defaultManager] attributesOfItemAtPath:filePath error:&error];
        
        if(!error) {
            return [attributes objectForKey:NSFileCreationDate];
        }
        
        return nil;
    }
####11.获取NSBundele中的资源图片
    + (UIImage *)imageAtApplicationDirectoryWithName:(NSString *)fileName {
        if(fileName) {
            NSString *path = [[[NSBundle mainBundle] resourcePath] stringByAppendingPathComponent:[fileName stringByDeletingPathExtension]];
            path = [NSString stringWithFormat:@"%@@2x.%@",path,[fileName pathExtension]];
            if(![[NSFileManager defaultManager] fileExistsAtPath:path]) {
                path = nil;
            }
            
            if(!path) {
                path = [[[NSBundle mainBundle] resourcePath] stringByAppendingPathComponent:fileName];
            }
            return [UIImage imageWithContentsOfFile:path];
        }
        return nil;
    }
####12.移除某个文件夹下的所有文件(非遍历)并重新创建被删除的文件夹
    + (void) deleteContentsOfFolder:(NSString *)folderPath {
        NSFileManager *fileManager = [NSFileManager defaultManager];
        [fileManager removeItemAtPath:folderPath error:nil];
        BOOL isDir = NO;
        BOOL existed = [fileManager fileExistsAtPath:folderPath isDirectory:&isDir];
        if ( !(isDir == YES && existed == YES) ) {
            [fileManager createDirectoryAtPath:folderPath withIntermediateDirectories:YES attributes:nil error:nil];
        }
    }
###二.文件操作常用方法总结:
####1.获取请求sign
    +(NSString *)getSign{
        NSString *username = @"zhaoyan106";
        NSString *pwd = @"zhaoyan106";
        NSString *pwd_md5 = [pwd md5];
        NSString *time_key_md5 = [[NSString stringWithFormat:@"%@%@",[self getTimeStamp],@"zhaoyan106"] md5];
        NSString *sign = [[NSString stringWithFormat:@"%@%@%@",username, pwd_md5,time_key_md5] md5];
        return sign;
    }
    
    - (NSString*)md5 {
        const char* string = [self UTF8String];
        unsigned char result[16];
        CC_MD5(string, strlen(string), result);
        NSString* hash = [NSString stringWithFormat:@"%@%@@%%@%@@@%@@%@@%@@@@@%@@@@%@@@%@@%@@@%@@%",
                          result[0], result[1], result[2], result[3], result[4], result[5], result[6], result[7],
                          result[8], result[9], result[10], result[11], result[12], result[13], result[14], result[15]];
        return [hash lowercaseString];
    }
    
    // 获取时间戳
	+(NSString *)getTimeStamp{
	    NSDate* dat = [NSDate dateWithTimeIntervalSinceNow:0];
	    NSTimeInterval a=[dat timeIntervalSince1970];
	    NSString *timeString = [NSString stringWithFormat:@"%.0f", a];
	    return timeString;
	}
####2.获取时间戳
	+(NSString *)getTimeStamp{
	    NSDate* dat = [NSDate dateWithTimeIntervalSinceNow:0];
	    NSTimeInterval a=[dat timeIntervalSince1970];
	    NSString *timeString = [NSString stringWithFormat:@"%.0f", a];
	    return timeString;
	}
####3.将对象转化为json字符串
	+ (NSString *)jsonStringFromObject:(id)object{
	    if([NSJSONSerialization isValidJSONObject:object]){
	        NSData *data = [NSJSONSerialization dataWithJSONObject:object options:NSJSONWritingPrettyPrinted error:nil];
	        NSString *jsonString = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
	        return jsonString;
	    }
	    return @"";
	}
####4.去除字典元素(去除NSNull)
	+ (id)valueForKey:(NSString *)key object:(NSDictionary *)object{
	    if([object isKindOfClass:[NSDictionary class]]){
	        id value = [object objectForKey:key];
	        if([value isKindOfClass:[NSNull class]]){
	            return nil;
	        }
	        return value;
	    }
	    return nil;
	}
####5.http request header 访问信息 

关于UDID等标识不清楚的可以参考下这篇文章<http://www.cocoachina.com/industry/20130715/6597.html>

	+ (NSString *)getTraceInfo {
	    //版本号
	    NSString *versionname  = [NSString stringWithFormat:@"%.1f", ((NSString *)[[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleVersion"]).integerValue-1.0+7.0];
	    //内部版本
	    NSString *versioncode  = ([[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleShortVersionString"]);
	    //发布版本
	    NSString *buildversion = ([[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleVersion"]);
	    //系统版本
	    NSString *osversion = [[UIDevice currentDevice] systemVersion];
	    //设备型号
	    NSString *model = [self deviceType];
	    //软件名称
	    NSString *appname = @"stem";
	    //软件平台
	    NSString *clientname = @"iphone";
	    //广告唯一标识符
	    NSString *idfa = [self getAdvertisingIdentifier];
	    //UDID
	    NSString *clientid = [LSHelper getCurrentDeviceUDID];
	//    NSString *traceInfo = [NSString stringWithFormat:@"versionname=%@;appname=%@;clientname=%@;",versionname,appname,clientname];
	    NSString *traceInfo = [NSString stringWithFormat:@"versionname=%@;versioncode=%@;buildversion=%@;osversion=%@;model=%@;appname=%@;clientname=%@;idfa=%@;clientid=%@;",versionname,versioncode,buildversion,osversion,model,appname,clientname,idfa,clientid];
	    return traceInfo;
	}

	// 获取当前设备类型如ipod，iphone，ipad
	+ (NSString *)deviceType {
	    struct utsname systemInfo;
	    uname(&systemInfo);
		return [NSString stringWithCString:systemInfo.machine encoding:NSUTF8StringEncoding];
	}
	
	// 获取用户的ADFA
	+ (NSString *) getAdvertisingIdentifier{
	    return [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
	}

	// 获取当前设备的UDID
	+(NSString*) getCurrentDeviceUDID
	{
	    if ([self getCurrentDeviceVersion]>=7.0) {
	    // OpenUDID
	        return [OpenUDID value];
	    }else{
	        return [self UUID];
	    }
	    
	}
	
	
	// 获取当前设备的UDID(ios7.0以下系统)
	+ (NSString *)UUID {
	    NSInteger mib[6];
	    size_t              len;
	    char                *buf;
	    unsigned char       *ptr;
	    struct if_msghdr    *ifm;
	    struct sockaddr_dl  *sdl;
	    
	    mib[0] = CTL_NET;
	    mib[1] = AF_ROUTE;
	    mib[2] = 0;
	    mib[3] = AF_LINK;
	    mib[4] = NET_RT_IFLIST;
	    
	    if ((mib[5] = if_nametoindex("en0")) == 0) {
	        printf("Error: if_nametoindex error\n");
	        return NULL;
	    }
	    
	    if (sysctl(mib, 6, NULL, &len, NULL, 0) < 0) {
	        printf("Error: sysctl, take 1\n");
	        return NULL;
	    }
	    
	    if ((buf = malloc(len)) == NULL) {
	        printf("Could not allocate memory. error!\n");
	        return NULL;
	    }
	    
	    if (sysctl(mib, 6, buf, &len, NULL, 0) < 0) {
	        free(buf);
	        printf("Error: sysctl, take 2");
	        return NULL;
	    }
	    
	    ifm = (struct if_msghdr *)buf;
	    sdl = (struct sockaddr_dl *)(ifm + 1);
	    ptr = (unsigned char *)LLADDR(sdl);
	    NSString *macaddress = [NSString stringWithFormat:@"%02X:%02X:%02X:%02X:%02X:%02X",
	                            *ptr, *(ptr+1), *(ptr+2), *(ptr+3), *(ptr+4), *(ptr+5)];
	    free(buf);
	    
	    
	    if (macaddress) {
	        NSString *uniqueIdentifier = [macaddress stringByReplacingOccurrencesOfString:@":" withString:@""];
	        return uniqueIdentifier;
	    } else {
	        NSUserDefaults *handler = [NSUserDefaults standardUserDefaults];
	        NSString *result = (NSString *)[handler objectForKey:kUUID];
	        
	        if (NULL == result || 46 > [result length]) {
	            CFUUIDRef uuid = CFUUIDCreate(NULL);
	            CFStringRef uuidStr = CFUUIDCreateString(NULL, uuid);
	            
	            result = [NSString stringWithFormat:@"%@", uuidStr];
	            CFRelease(uuidStr);
	            CFRelease(uuid);
	            
	            [handler setObject:result forKey:kUUID];
	            [handler synchronize];
	        }
	        
	        return result;
	    }
	}
####6.(参数 + url)md5加密
	+(NSString *)keyMD5StringWithParamDic:(NSDictionary *)paramDic methodName:(NSString *)methodName
	{
	    NSMutableString *keyMutableString = nil;
	    if (paramDic == nil) {
	        return nil;
	    }
	    
	    NSArray *keys = [paramDic allKeys];
	    NSArray *sortedArray = [keys sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
	        return [obj1 compare:obj2 options:NSNumericSearch];
	    }];
	    
	    keyMutableString = [NSMutableString string];
	    for (NSInteger index = 0; index < sortedArray.count; index++) {
	        NSString *key = [sortedArray objectAtIndex:index];
	        NSString *value = [paramDic objectForKey:key];
	        if (index == 0) {
	            [keyMutableString appendFormat:@"%@=%@",key,value];
	        } else {
	            [keyMutableString appendFormat:@"|%@=%@",key,value];
	        }
	    }
	    NSString *url = [NSString stringWithFormat:@"%@%@",kServerAddress,methodName];
	    [keyMutableString appendString:url];
	    return [keyMutableString md5];
	}
####7.cookie操作
	// 设置cooike
	+ (void)setHttpCookieValue:(NSString *)value{
	    //过期时间一个小时
	    NSDate *date = [NSDate dateWithTimeIntervalSinceNow:60*60];
	    //设置了过期时间，如果过期，则NSURLRequest就不会使用此cookie
	    NSHTTPCookie *cookie = [NSHTTPCookie cookieWithProperties:@{NSHTTPCookiePath:@"/", NSHTTPCookieName:@"JSESSIONID", NSHTTPCookieValue:value, NSHTTPCookieOriginURL:kServerAddress, NSHTTPCookieExpires:date}];
	    [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:cookie];
	    [self writeFileName:kSpecialCookieFileName data:@[cookie]];
	}
	// 读取cookie
	+ (NSString *)valueFromHttpCookie{
	    NSArray *cookieArr = [self readFileName:kSpecialCookieFileName];
	    NSMutableArray *mutableArr = [NSMutableArray arrayWithArray:cookieArr];
	    NSString *value = nil;
	    for(NSHTTPCookie *cookie in cookieArr){
	        if([cookie.name isEqual:@"JSESSIONID"] && cookie.value.length > 0){
	            //如果过期了，则删除本地
	            if([cookie.expiresDate compare:[NSDate date]] == NSOrderedAscending){
	                [mutableArr removeObject:cookie];
	            }
	            else{
	                [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:cookie];
	                value = cookie.value;
	            }
	        }
	    }
	    [self writeFileName:kSpecialCookieFileName data:mutableArr];
	    return value;
	}
	// 删除cookie
	+ (void)removeHttpCookieCache{
	    [self deleteCacheFile:kSpecialCookieFileName];
	}
###三.时间常用方法总结:
####1.获取当前时间
	+ (NSString *)getCurrentTime
	{
	    NSDate *senddate = [NSDate date];
	    
	    NSDateFormatter *dateformatter = [[NSDateFormatter alloc] init];
	    
	    [dateformatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
	    
	    NSString *locationString = [dateformatter stringFromDate:senddate];
	    
	    return locationString;
	    
	}
####2.字符时间转date
	+ (NSDate *)dateFromString:(NSString *)dateString {
	    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
	    [formatter setDateFormat:@"eee, dd MMM yyyy HH:mm:ss ZZZZ"];
	    NSDate *date = [formatter dateFromString:dateString];
	    return date;
	}
####3.date转字符串
	+ (NSString *)stringFromDate:(NSDate *)date {
	    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
	    //zzz表示时区，zzz可以删除，这样返回的日期字符将不包含时区信息。
	    [dateFormatter setDateFormat:@"yyyy-MM-dd"];
	    NSString *destDateString = [dateFormatter stringFromDate:date];
	    return destDateString;
	}
####4.自1970年距离现在时间转换
	+ (NSString *)timeFactorySeconds:(NSNumber *)seconds andFormat:(TimeFormat)type {
	    float second = [seconds doubleValue];
	    if (second == 0) {
	        return nil;
	    }
	    NSString *result = nil;
	    NSDate *date = [NSDate dateWithTimeIntervalSince1970:[seconds doubleValue]/1000];
	    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
	    switch (type) {
	        case YearMonthDayHourMin: {
	            [formatter setDateFormat:@"yyyy-MM-dd HH:mm"];
	            result = [formatter stringFromDate:date];
	        }
	            break;
	        case YearMonthDay: {
	            [formatter setDateFormat:@"yyyy-MM-dd"];
	            result = [formatter stringFromDate:date];
	        }
	            break;
	        case MonthDayHourMin: {
	            [formatter setDateFormat:@"MM-dd HH:mm"];
	            result = [formatter stringFromDate:date];
	        }
	            break;
	        case YearMonthDayHZ: {
	            [formatter setDateFormat:@"yyyy年MM月dd日"];
	            result = [formatter stringFromDate:date];
	        }
	            break;
	        default:
	            break;
	    }
	    return result;
	}
####5. 数组按时间排序
	//数组按时间排序
	+ (NSArray *)resortHomePageData:(NSArray *)data {
	// NSSortDescriptor可以直接对对象的某个属性进行排列
	    NSArray *sortDescriptors = [NSArray arrayWithObject:[NSSortDescriptor sortDescriptorWithKey:@"mtime" ascending:YES]];
	    return [data sortedArrayUsingDescriptors:sortDescriptors];
	}
###四.图片操作
####1.去最大的正方形
	+ (UIImage *)cutToRect:(UIImage *)image {
	    double width = ((UIImage *)image).size.width * image.scale;
	    double height = ((UIImage *)image).size.height * image.scale;
	    UIImage *img = nil;
	    if (width > height) {
	        CGImageRef imageRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake((width - height)/2.0f, 0, height, height));
	        img = [UIImage imageWithCGImage:imageRef scale:image.scale orientation:image.imageOrientation];
	        CGImageRelease(imageRef);
	    }
	    else {
	        CGImageRef imageRef = CGImageCreateWithImageInRect([image CGImage], CGRectMake(0, (height - width)/2.0f, width, width));
	        img = [UIImage imageWithCGImage:imageRef scale:image.scale orientation:image.imageOrientation];
	        CGImageRelease(imageRef);
	    }
	    return img;
	}
####2.按比例取最大图片
	+ (UIImage *)cutToBigRect:(UIImage *)image size:(CGSize)size {
	    UIImage *imageRemove = [self imageResize:image size:image.size];//去掉拍照 图片带的方向信息
	    double width = imageRemove.size.width * imageRemove.scale;
	    double height = imageRemove.size.height * imageRemove.scale;
	    UIImage *img = nil;
	    CGImageRef imageRef;
	    if (height / width <= size.height / size.width) {//height小
	        imageRef = CGImageCreateWithImageInRect([imageRemove CGImage], CGRectMake((width - height * (size.width / size.height))/2.0f, 0, height * (size.width / size.height), height));
	    }
	    else {
	        imageRef = CGImageCreateWithImageInRect([imageRemove CGImage], CGRectMake(0, (height - width * (size.height / size.width))/2.0f, width, width * (size.height / size.width)));
	        
	    }
	    img = [UIImage imageWithCGImage:imageRef scale:imageRemove.scale orientation:imageRemove.imageOrientation];
	    CGImageRelease(imageRef);
	    return img;
	}
####3.将图片放缩到指定大小
	+ (UIImage*)imageResize:(UIImage*)image size:(CGSize)size {
	    UIGraphicsBeginImageContext(size);
	    [image drawInRect:CGRectMake(0,0,size.width, size.height)];
	    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
	    UIGraphicsEndImageContext();
	    return newImage;
	}