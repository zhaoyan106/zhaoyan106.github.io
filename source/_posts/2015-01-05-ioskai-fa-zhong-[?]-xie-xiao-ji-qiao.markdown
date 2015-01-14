---
layout: post
title: "iOS开发中一些小技巧"
date: 2015-01-05 11:13:29 +0800
comments: true
categories: 
---
原文地址：<http://www.cocoachina.com/ios/20141231/10783.html>最近cocoachina上很火的一篇文章
####1.TableView不显示没内容的Cell

	self.tableView.tableFooterView = [[UIView alloc] init];
	
####2.自定义了leftBarbuttonItem左滑返回手势失效了
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]
                                         initWithImage:img
                                         style:UIBarButtonItemStylePlain
                                         target:self
                                         action:@selector(onBack:)];
	self.navigationController.interactivePopGestureRecognizer.delegate = (id)self;

在iOS7以后系统自动支持了左侧滑动返回功能,但当设置了左侧返回按钮,就会失效,这个方法很好地解决了问题
	
####3.ScrollView莫名其妙不能在viewController划到顶怎么办?

	self.automaticallyAdjustsScrollViewInsets = NO;
	
这个功能也是iOS7之后系统带的功能

####4.像safari一样滑动的时候隐藏navigationbar?

	navigationController.hidesBarsOnSwipe = Yes
	
####5.导航条返回键带的title太讨厌了,怎么让它消失!

	[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -60) forBarMetrics:UIBarMetricsDefault];
	
####6.怎么把tableview里cell的小对勾的颜色改成别的颜色？

	_mTableView.tintColor = [UIColor redColor];
	
####7.本来我的statusbar是lightcontent的，结果用UIImagePickerController会导致我的statusbar的样式变成黑色，怎么办？
	
	- (void)navigationController:(UINavigationController *)navigationController willShowViewController:(UIViewController *)viewController animated:(BOOL)animated
	{
   	 	[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
	}
 
####8.怎么把我的navigationbar弄成透明的而不是带模糊的效果？

	[self.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];
	self.navigationBar.shadowImage = [UIImage new];
	self.navigationBar.translucent = YES;
####9.怎么改变uitextfield placeholder的颜色和位置？
	- (void) drawPlaceholderInRect:(CGRect)rect {
    	[[UIColor blueColor] setFill];
    	[self.placeholder drawInRect:rect withFont:self.font 	lineBreakMode:UILineBreakModeTailTruncation alignment:self.textAlignment];
	}






