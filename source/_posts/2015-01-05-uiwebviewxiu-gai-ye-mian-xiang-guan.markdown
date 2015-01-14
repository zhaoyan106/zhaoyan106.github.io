---
layout: post
title: "UIWebView修改页面相关"
date: 2015-01-05 17:56:03 +0800
comments: true
categories: 
---
UIWebView打开html后，UIWebView有左右滚动条，要去掉左右滚动效果； 

方法：通过js截获UIWebView中的html，然后修改html标签内容； 

实例代码： 服务器端html

	<html><head>  
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8">  
	<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">   
	<title>网曝四川省一考场时钟慢半小时 老师称这就是命</title></head<body>网曝四川省一考场时钟慢半小时 老师称这就是命</body></html> 
	
这样显示的结果网页的最小宽度会是device-width；但有时候不需要这个宽度，就需要修改width=device-width为width=myWidth; 
客户端代码

	- (void)webViewDidFinishLoad:(UIWebView *)webView  
	{     
	    //修改服务器页面的meta的值  
	    NSString *meta = [NSString stringWithFormat:@"document.getElementsByName(\"viewport\")[0].content = \"width=%f, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no\"", webView.frame.size.width];  
	    [webView stringByEvaluatingJavaScriptFromString:meta];  
	}  

给网页增加utf-8编码 

	[webView stringByEvaluatingJavaScriptFromString:  
	 @"var tagHead =document.documentElement.firstChild;"  
	  "var tagMeta = document.createElement(\"meta\");"   
	  "tagMeta.setAttribute(\"http-equiv\", \"Content-Type\");"   
	  "tagMeta.setAttribute(\"content\", \"text/html; charset=utf-8\");"   
	  "var tagHeadAdd = tagHead.appendChild(tagMeta);"]; 
	  
给网页增加css样式  

	[webView stringByEvaluatingJavaScriptFromString:  
	     @"var tagHead =document.documentElement.firstChild;"  
	     "var tagStyle = document.createElement(\"style\");"   
	     "tagStyle.setAttribute(\"type\", \"text/css\");"   
	     "tagStyle.appendChild(document.createTextNode(\"BODY{padding: 20pt 15pt}\"));"  
	     "var tagHeadAdd = tagHead.appendChild(tagStyle);"];
	     
拦截网页图片  并修改图片大小  

	[webView stringByEvaluatingJavaScriptFromString:  
	 @"var script = document.createElement('script');"   
	 "script.type = 'text/javascript';"   
	 "script.text = \"function ResizeImages() { "   
	     "var myimg,oldwidth;"  
	     "var maxwidth=380;" //缩放系数   
	     "for(i=0;i <document.images.length;i++){"   
	         "myimg = document.images[i];"  
	         "if(myimg.width > maxwidth){"   
	             "oldwidth = myimg.width;"   
	             "myimg.width = maxwidth;"   
	             "myimg.height = myimg.height * (maxwidth/oldwidth);"   
	         "}"   
	     "}"   
	 "}\";"   
	 "document.getElementsByTagName('head')[0].appendChild(script);"];   
	[webView stringByEvaluatingJavaScriptFromString:@"ResizeImages();"]; 
	
参考网址： 

stringByEvaluatingJavaScriptFromString的使用方法:

<http://www.uml.org.cn/mobiledev/201108181.asp>

iphone 获取UIWebView内Html方法

<http://blog.csdn.net/diyagoanyhacker/article/details/6564897>

IOS UIWebView引用外部CSS样式

<http://hi.baidu.com/jwq359699768/item/780879e5c98bfb3e4ddcaf22>
<http://blog.csdn.net/xdonx/article/details/6973521>
