---
layout: post
title: "压缩、截取上传图片,设置圆形UIImageView,App版本更新，设置tableView随textFiled焦点滑动"
date: 2015-02-11 14:47:42 +0800
comments: true
categories: 
---
####一、压缩、截取图片
	
	- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
	    if (indexPath.row == 0) {
	            UIActionSheet * actionsheet = [[UIActionSheet alloc] initWithTitle:nil delegate:self cancelButtonTitle:@"取消" destructiveButtonTitle:nil otherButtonTitles:@"拍照",@"从相册选择",nil];
	            actionsheet.actionSheetStyle = UIActionSheetStyleAutomatic;
	            [actionsheet showInView:self.view];
	        }
	}
	           
	
	- (void)actionSheet:(UIActionSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex{
	    
	    if (buttonIndex == 0) { // 拍照
	        [self takePhoto];
	    }else if(buttonIndex == 1){ // 相册
	        [self LocalPhoto];
	    }else if (buttonIndex == 2) { // 取消
	        
	    }
	}
	
	//开始拍照
	-(void)takePhoto
	{
	    UIImagePickerControllerSourceType sourceType = UIImagePickerControllerSourceTypeCamera;
	    if ([UIImagePickerController isSourceTypeAvailable: UIImagePickerControllerSourceTypeCamera]){
	        UIImagePickerController *picker = [[UIImagePickerController alloc] init];
	        picker.delegate = self;
	        picker.sourceType = sourceType;
	        picker.allowsEditing = YES;
	        [self presentViewController:picker animated:YES completion:nil];
	    }else{
	        NSLog(@"模拟器中无法打开照相机,请在真机中使用");
	    }
	}
	
	//打开本地相册
	-(void)LocalPhoto
	{
	    UIImagePickerController *picker = [[UIImagePickerController alloc] init];
	    picker.sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
	    picker.delegate = self;
	    //设置选择后的图片可被编辑
	    picker.allowsEditing = YES;
	    [self presentViewController:picker animated:YES completion:nil];
	    
	}
	
	//当选择一张图片后进入这里
	-(void)imagePickerController:(UIImagePickerController*)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
	{
	    UIImage* image = [info objectForKey:@"UIImagePickerControllerOriginalImage"];
	    
	    if (image != nil) {
	       image = [self OriginImage:image scaleToSize:CGSizeMake(120, 120)];
	        NSData *data;
	        if (UIImagePNGRepresentation(image)) {
	            //返回为png图像。
	            data = UIImagePNGRepresentation(image);
	        }else {
	            //返回为JPEG图像。压缩比率设置0.3-0.7之间最好
	            data = UIImageJPEGRepresentation(image, 0.5);
	        }
	        //此处网络请求
	       	[picker dismissViewControllerAnimated:YES completion:nil];
	}


	- (UIImage*)OriginImage:(UIImage *)image scaleToSize:(CGSize)size
	{
	    UIGraphicsBeginImageContext(size);  //size 为CGSize类型，即你所需要的图片尺寸
	    
	    [image drawInRect:CGRectMake(0, 0, size.width, size.height)];
	    
	    UIImage* scaledImage = UIGraphicsGetImageFromCurrentImageContext();
	    
	    UIGraphicsEndImageContext();
	    
	    return scaledImage;   //返回的就是已经改变的图片
	}

        
####二、设置UIImageView为圆形
	self.icon.layer.masksToBounds=YES;
    
    self.icon.layer.cornerRadius= 30.0;    //最重要的是这个地方要设成imgview高的一半
    
	self.outimgViewCusActivity.layer.borderWidth=1.0;
    self.icon.backgroundColor = [UIColor whiteColor];
    
####三、APP版本更新

	-(void)onCheckVersion
	{
	    NSDictionary *infoDic = [[NSBundle mainBundle] infoDictionary];
	    //CFShow((__bridge CFTypeRef)(infoDic));
	    NSString *currentVersion = [infoDic objectForKey:@"CFBundleVersion"];
	    // 或者@""https://itunes.apple.com/lookup?bundleId=cn.keyno.$(PRODUCT_NAME:rfc1034identifier)"
	    NSString *URL = @"http://itunes.apple.com/lookup?id=你的应用程序的ID";
	    http://itunes.apple.com/lookup?id=你的应用程序的ID
	    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];
	    [request setURL:[NSURL URLWithString:URL]];
	    [request setHTTPMethod:@"POST"];
	    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
	        [MBProgressHUD hideAllHUDsForView:self.view animated:YES];
	        NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
	        NSArray *infoArray = [dic objectForKey:@"results"];
	        if ([infoArray count]) {
	            NSDictionary *releaseInfo = [infoArray objectAtIndex:0];
	            NSString *lastVersion = [releaseInfo objectForKey:@"version"];
	            
	            if (![lastVersion isEqualToString:currentVersion]) {
	                //trackViewURL = [releaseInfo objectForKey:@"trackVireUrl"];
	                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"更新" message:@"有新的版本更新，是否前往更新？" delegate:self cancelButtonTitle:@"关闭" otherButtonTitles:@"更新", nil];
	                alert.tag = 10000;
	                [alert show];
	            }
	            else
	            {
	                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"更新" message:@"此版本为最新版本" delegate:self cancelButtonTitle:@"确定" otherButtonTitles:nil, nil];
	                alert.tag = 10001;
	                [alert show];
	            }
	        }
	        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"更新" message:@"有新的版本更新，是否前往更新？" delegate:self cancelButtonTitle:@"关闭" otherButtonTitles:@"更新", nil];
	        alert.tag = 10000;
	        [alert show];
	        
	    }];
	}
	
	- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
	{
	    if (alertView.tag==10000) {
	        if (buttonIndex==1) {
	            NSURL *url = [NSURL URLWithString:@"itms-apps://itunes.apple.com/app/id你的应用程序的ID"];
	            [[UIApplication sharedApplication]openURL:url];
	        }
	    }
	}
	
	-(void)goToAppStore{
		NSString *str = [NSString stringWithFormat:
		@"itms-apps://ax.itunes.apple.com/WebObjects/MZStore.woa/wa/viewContentsUserReviews?type=Purple+Software&id=%d",appID];
		[[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];
	} 
		iOS7下苹果对app stotr的评分页面链接做了修改，及 
		NSString *str = [NSString stringWithFormat:
		
		@"itms-apps://itunes.apple.com/app/id%@",APPID];
		
		[[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];

####四、设置tableView随textFiled焦点滑动
######第一种方法：
CGPoint origin = textField.frame.origin;

    CGPoint point = [textField.superview convertPoint:origin toView:self.tableView];

    float navBarHeight = self.navigationController.navigationBar.frame.size.height;

    CGPoint offset = self.tableView.contentOffset;

    offset.y = (point.y - navBarHeight);

    [self.tableView setContentOffset:offset animated:YES];
   
######第二种方法
    CGPoint point = [textField.superview convertPoint:textField.frame.origin toView:self.tableView];

    NSIndexPath *indexPath = [self.tableView indexPathForRowAtPoint:point];

    NSLog(@"%zd",indexPath.row);

    [self.tableView scrollToRowAtIndexPath:indexPath atScrollPosition:UITableViewScrollPositionTop animated:YES];

