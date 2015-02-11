---
layout: post
title: "POST请求上传文件"
date: 2015-02-09 13:56:50 +0800
comments: true
categories: 
---
####利用POST请求发送文件数据,需要对数据进行拼接

	----------httppost（\r\n）
	Content-Disposition: form-data; name="img"; filename="t.txt" （\r\n）
	Content-Type: application/octet-stream （\r\n）
	（\r\n）
	----------httppost（\r\n）
	Content-Disposition: form-data; name="text" （\r\n）
	（\r\n）
	text tttt （\r\n）
	----------httppost（\r\n）
	
####使用NSURLConnection发送POST上传文件

	#define Encode(str) [str dataUsingEncoding:NSUTF8StringEncoding]
	
	- (void)upload:(NSString *)name filename:(NSString *)filename mimeType:(NSString *)mimeType data:(NSData *)data parmas:(NSDictionary *)params
	{
	    // 文件上传
	    NSURL *url = [NSURL URLWithString:@"http://127.0.0.1/upload"];
	    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
	    request.HTTPMethod = @"POST";
	    
	    // 设置请求体
	    NSMutableData *body = [NSMutableData data];
	    
	    /***************文件参数***************/
	    // 参数开始的标志
	    [body appendData:Encode(@"--ZY\r\n")];
	    // name : 指定参数名(必须跟服务器端保持一致)
	    // filename : 文件名
	    NSString *disposition = [NSString stringWithFormat:@"Content-Disposition: form-data; name=\"%@\"; filename=\"%@\"\r\n", name, filename];
	    [body appendData:Encode(disposition)];
	    NSString *type = [NSString stringWithFormat:@"Content-Type: %@\r\n", mimeType];
	    [body appendData:Encode(type)];
	    
	    [body appendData:Encode(@"\r\n")];
	    [body appendData:data];
	    [body appendData:Encode(@"\r\n")];
	    
	    /***************普通参数***************/
	    [params enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
	        // 参数开始的标志
	        [body appendData:Encode(@"--ZY\r\n")];
	        NSString *disposition = [NSString stringWithFormat:@"Content-Disposition: form-data; name=\"%@\"\r\n", key];
	        [body appendData:Encode(disposition)];
	        
	        [body appendData:Encode(@"\r\n")];
	        [body appendData:Encode(obj)];
	        [body appendData:Encode(@"\r\n")];
	    }];
	    
	    /***************参数结束***************/
	    // ZY--\r\n
	    [body appendData:Encode(@"--ZY--\r\n")];
	    request.HTTPBody = body;
	    
	    // 设置请求头
	    // 请求体的长度
	    [request setValue:[NSString stringWithFormat:@"%zd", body.length] forHTTPHeaderField:@"Content-Length"];
	    // 声明这个POST请求是个文件上传
	    [request setValue:@"multipart/form-data; boundary=ZY" forHTTPHeaderField:@"Content-Type"];
	    
	    // 发送请求
	    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
	        if (data) {
	            NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:nil];
	            NSLog(@"%@", dict);
	        } else {
	            NSLog(@"上传失败");
	        }
	    }];
	}
	
####利用AFN框架发送POST上传文件请求

	- (void)modifyUserIcon:(NSData *)data completion:(ResultInfoBlock)completion{
	    // 1.获得请求管理者
	    AFHTTPRequestOperationManager *mgr = [AFHTTPRequestOperationManager manager];
	    // 2.发送请求(做文件上传)
	    mgr.responseSerializer = [AFHTTPResponseSerializer serializer];
	    [mgr POST:@"http://123.56.113.104:8081/upload" parameters:nil
	constructingBodyWithBlock:^(id<AFMultipartFormData> formData) {
	    // 一定要在这个block中添加文件参数
	    // 拼接文件参数
	    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
	    // 设置时间格式
	    formatter.dateFormat = @"yyyyMMddHHmmss";
	    NSString *str = [formatter stringFromDate:[NSDate date]];
	    NSString *fileName = [NSString stringWithFormat:@"%@.png", str];
	    [formData appendPartWithFileData:data name:@"imgFile" fileName:fileName mimeType:@"image/png"];
	}
	      success:^(AFHTTPRequestOperation *operation, id responseObject) {
	          NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:responseObject options:NSJSONReadingMutableContainers error:nil];
	          NSLog(@"上传成功----%@", dict);
	          completion(dict,nil);
	      } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
	          NSLog(@"上传失败----%@", error);
	          completion(nil,error);
	      }];
	
	}


