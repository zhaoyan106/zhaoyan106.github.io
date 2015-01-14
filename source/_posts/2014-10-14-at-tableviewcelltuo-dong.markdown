---
layout: post
title: "tableViewCell拖动"
date: 2014-10-14 17:54:15 +0800
comments: true
categories: 
---
1. 为tableView添加长按手势
 
 		UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPressGestureRecognized:)];
		[self.tableView addGestureRecognizer:longPress];
2. 监听方法,根据手势状态进行Cell的复制，移动

		- (IBAction)longPressGestureRecognized:(id)sender {
			UILongPressGestureRecognizer *longPress = (UILongPressGestureRecognizer *)sender;
			UIGestureRecognizerState state = longPress.state;
			CGPoint location = [longPress locationInView:self.tableView];
 	 		NSIndexPath *indexPath = [self.tableView indexPathForRowAtPoint:location];
			static UIView       *snapshot = nil;        ///< A snapshot of the row user is moving.
  			static NSIndexPath  *sourceIndexPath = nil; ///< Initial index path, where gesture begins.
  			switch (state) {
    			case UIGestureRecognizerStateBegan: {
      				if (indexPath) {
        			sourceIndexPath = indexPath;
       				UITableViewCell *cell = [self.tableView cellForRowAtIndexPath:indexPath];
        			// Take a snapshot of the selected row using helper method.
       				 snapshot = [self customSnapshoFromView:cell]; 
      				  // Add the snapshot as subview, centered at cell's center...
       				 __block CGPoint center = cell.center;
       				 snapshot.center = center;
        			 snapshot.alpha = 0.0;
        			 [self.tableView addSubview:snapshot];
        			 [UIView animateWithDuration:0.25 animations:^{
         				 // Offset for gesture location.
          					center.y = location.y;
          					snapshot.center = center;
         					snapshot.transform = CGAffineTransformMakeScale(1.05, 1.05);
          					snapshot.alpha = 0.98;
          					cell.alpha = 0.0;  
        			} completion:^(BOOL finished) {
          					cell.hidden = YES；
       	 			}];
     	 		}
     					 break;
   	 		}
  	 			case UIGestureRecognizerStateChanged: {
      				CGPoint center = snapshot.center;
      				center.y = location.y;
      				snapshot.center = center;
      				// Is destination valid and is it different from source?
     		 		if (indexPath && ![indexPath isEqual:sourceIndexPath]) {
        			// ... update data source.
        			[self.objects exchangeObjectAtIndex:indexPath.row withObjectAtIndex:sourceIndexPath.row];        
        			// ... move the rows.
        			[self.tableView moveRowAtIndexPath:sourceIndexPath toIndexPath:indexPath];
        			// ... and update source so it is in sync with UI changes.
        			sourceIndexPath = indexPath;
      		}
      					break;
   			}
    			default: {
      			// Clean up.
     					UITableViewCell *cell = [self.tableView cellForRowAtIndexPath:sourceIndexPath];
     		 			cell.hidden = NO;
      		 			cell.alpha = 0.0;
     		 			[UIView animateWithDuration:0.25 animations:^{
       							snapshot.center = cell.center;
        						snapshot.transform = CGAffineTransformIdentity;
        						snapshot.alpha = 0.0;
        						cell.alpha = 1.0;
      					} completion:^(BOOL finished) {
        					sourceIndexPath = nil;
       						[snapshot removeFromSuperview];
       	 					snapshot = nil;
      					}];
     				 	break;
   		 		}
  		  	}
		}

3. 生产镜像Cell

		- (UIView *)customSnapshoFromView:(UIView *)inputView {
 			UIView *snapshot = [inputView snapshotViewAfterScreenUpdates:YES];
  			snapshot.layer.masksToBounds = NO;
  			snapshot.layer.cornerRadius = 0.0;
  			snapshot.layer.shadowOffset = CGSizeMake(-5.0, 0.0);
  			snapshot.layer.shadowRadius = 5.0;
  			snapshot.layer.shadowOpacity = 0.4;
  			return snapshot;
		}
