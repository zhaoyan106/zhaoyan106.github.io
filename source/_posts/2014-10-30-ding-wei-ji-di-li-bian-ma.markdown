---
layout: post
title: "定位及地理编码"
date: 2014-10-30 17:04:55 +0800
comments: true
categories: 
---
####定位
在iOS8中,取消了之前的定位调用方式,需要对设备版本进行判断

    // 定位管理者
    _locManager = [[CLLocationManager alloc] init];
    [_locManager setDelegate:self];
    [_locManager setDesiredAccuracy:kCLLocationAccuracyBest];
	#if __IPHONE_OS_VERSION_MAX_ALLOWED > __IPHONE_7_1 
    if ([_locManager respondsToSelector:@selector(requestWhenInUseAuthorization)]) {
        [_locManager requestWhenInUseAuthorization];
        [_locManager requestAlwaysAuthorization];
    }
	#endif
    CLAuthorizationStatus status = [CLLocationManager authorizationStatus];
    if (kCLAuthorizationStatusDenied == status || kCLAuthorizationStatusRestricted == status) {
        [LSHelper alertWithTitle:@"定位失败" message:@"您的本次拜访将无法记录位置信息，请在设置中打开您的位置服务，并重新进入该页面！"];
    }
    else {
        [_locManager startUpdatingLocation];
        [self showProgressViewWithTitle:@"定位中..."];
    }
 
####地理编码
以下是代理方法:

	- (void)locationManager:(CLLocationManager *)manager didUpdateToLocation:(CLLocation *)newLocation fromLocation:(CLLocation *)oldLocation {
    	[_locManager stopUpdatingLocation];
    	[self hideProgressView];
    	CLLocationCoordinate2D loc = [newLocation coordinate];
    	_locLatitude = [[NSString stringWithFormat:@"%f",loc.latitude] floatValue] + 0.00130;
    	_locLongitude = [[NSString stringWithFormat:@"%f",loc.longitude] floatValue] + 0.00625;
    
    	CLLocation *location = [[CLLocation alloc] initWithLatitude:_locLatitude longitude:_locLongitude];
    	[self.geoCoder reverseGeocodeLocation:location completionHandler:^(NSArray *placemarks, NSError *error) {
        	if (error || placemarks.count == 0) {
            	[self.view makeToast:@"定位失败，请稍后重试！"];
            	[self.locBtn setBackgroundImage:[UIImage imageNamed:@"MapPinGray"] forState:UIControlStateNormal];
            	[self.locBtn addTarget:self action:@selector(locationIconBtnClick) forControlEvents:UIControlEventTouchUpInside];
            	self.locLab.textColor = [UIColor redColor];
            	self.locLab.text = @"未定位成功";
        	} else {
            	[self.view makeToast:@"定位成功！"];
            	self.locBtn.enabled = NO;
            	[self.locBtn setBackgroundImage:[UIImage imageNamed:@"MapPin@2x"] forState:UIControlStateNormal];
            	self.locLab.textColor = [UIColor blackColor];
            	CLPlacemark *firstPlacemark = [placemarks firstObject];
            	self.locLab.text = firstPlacemark.name;
        	}
    	}];
	}

	- (void)locationManager:(CLLocationManager *)manager didFailWithError:(NSError *)error {
    	[_locManager stopUpdatingLocation];
    	[self hideProgressView];
    	[LSHelper alertWithTitle:@"定位失败，请稍后重试！" message:nil];
    
    	[self.locBtn setBackgroundImage:[UIImage imageNamed:@"MapPinGray"] forState:UIControlStateNormal];
    	[self.locBtn addTarget:self action:@selector(locationIconBtnClick) forControlEvents:UIControlEventTouchUpInside];
    	self.locLab.textColor = [UIColor redColor];
    	self.locLab.text = @"未定位成功";
	}
	
高德地图的火星坐标需要进行处理,在此代理方法中对坐标进行地理编码操作,注意: 定位可以在没有网络的情况下进行,但是地理编码需要依靠网络.

