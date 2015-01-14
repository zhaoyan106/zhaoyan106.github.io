---
layout: post
title: "瀑布流（模仿TableView）"
date: 2014-11-21 16:22:24 +0800
comments: true
categories: 
---
在一些应用中也许会用到一瀑布流来展示信息。以下是用的模仿TableView的写法，之后会写一篇iOS6之后能使用的collcetionView。

首先模仿tableView的数据源方法（用来控制有多少数据，多少列，返回的具体cell）：

	/**
	 *  数据源方法
	*/
	@protocol HMWaterflowViewDataSource <NSObject>
	@required
	/**
	 *  一共有多少个数据
	*/
	- (NSUInteger)numberOfCellsInWaterflowView:(HMWaterflowView *)waterflowView;
	/**
	 *  返回index位置对应的cell
	 */
	- (HMWaterflowViewCell *)waterflowView:(HMWaterflowView *)waterflowView cellAtIndex:(NSUInteger)index;
	
	@optional
	/**
	*  一共有多少列
	*/
	- (NSUInteger)numberOfColumnsInWaterflowView:(HMWaterflowView *)waterflowView;
	@end
	代理方法(用来控制每个cell的高度，间距和监听cell的点击):
	
	/**
	 *  代理方法
	*/
	@protocol HMWaterflowViewDelegate <UIScrollViewDelegate>
	@optional
	/**
	 *  第index位置cell对应的高度
	*/
	- (CGFloat)waterflowView:(HMWaterflowView *)waterflowView heightAtIndex:(NSUInteger)index;
	/**
	 *  选中第index位置的cell
	 */
	- (void)waterflowView:(HMWaterflowView *)waterflowView didSelectAtIndex:(NSUInteger)index;
	/**
	 *  返回间距
	 */
	- (CGFloat)waterflowView:(HMWaterflowView *)waterflowView marginForType:(HMWaterflowViewMarginType)type;
	
	@end
首选写瀑布流有一点是必须清楚的，那就是每个cell的宽度是相同的，每个cell的高度是服务器返回的数据（因此有些图片需要根据宽高比进行缩放），由于需要考虑到每个cell的复用，还需要提供具体对应的dequeueReusableCellWithIdentifier和reloadData方法。

说到复用就必须要有一个缓存池，还有一个展示池（正在显示的cell），因为展示池需要判断cell的位置，这个池使用字典：

	/**
	*  正在展示的cell
	*/
	@property (nonatomic, strong) NSMutableDictionary *displayingCells;
	缓存池只负责存放因为不需要顺序，使用集合
	
	/**
	 *  缓存池（用Set，存放离开屏幕的cell）
	*/
	@property (nonatomic, strong) NSMutableSet *reusableCells;
	另外还需要一个存放所有cell的Frame的数组
	
	/**
	*  所有cell的frame数据
	*/
	@property (nonatomic, strong) NSMutableArray *cellFrames;
	
通过reloadData方法计算出每个cell的Frame，并且计算出需要展示的contentSize
	
	- (void)reloadData
	{
	    // 清空之前的所有数据
	    // 移除正在正在显示cell
	    [self.displayingCells.allValues makeObjectsPerformSelector:@selector(removeFromSuperview)];
	    [self.displayingCells removeAllObjects];
	    [self.cellFrames removeAllObjects];
	    [self.reusableCells removeAllObjects];
	
	    // cell的总数
	    int numberOfCells = [self.dataSource numberOfCellsInWaterflowView:self];
	
	    // 总列数
	    int numberOfColumns = [self numberOfColumns];
	
	    // 间距
	    CGFloat topM = [self marginForType:HMWaterflowViewMarginTypeTop];
	    CGFloat bottomM = [self marginForType:HMWaterflowViewMarginTypeBottom];
	    CGFloat leftM = [self marginForType:HMWaterflowViewMarginTypeLeft];
	    CGFloat columnM = [self marginForType:HMWaterflowViewMarginTypeColumn];
	    CGFloat rowM = [self marginForType:HMWaterflowViewMarginTypeRow];
	
	    // cell的宽度
	    CGFloat cellW = [self cellWidth];
	
	    // 用一个C语言数组存放所有列的最大Y值
	    CGFloat maxYOfColumns[numberOfColumns];
	    for (int i = 0; i<numberOfColumns; i++) {
	        maxYOfColumns[i] = 0.0;
	    }
	
	    // 计算所有cell的frame
	    for (int i = 0; i<numberOfCells; i++) {
	        // cell处在第几列(最短的一列)
	        NSUInteger cellColumn = 0;
	        // cell所处那列的最大Y值(最短那一列的最大Y值)
	        CGFloat maxYOfCellColumn = maxYOfColumns[cellColumn];
	        // 求出最短的一列
	        for (int j = 1; j<numberOfColumns; j++) {
	            if (maxYOfColumns[j] < maxYOfCellColumn) {
	                cellColumn = j;
	                maxYOfCellColumn = maxYOfColumns[j];
	            }
	        }
	
	        // 询问代理i位置的高度
	        CGFloat cellH = [self heightAtIndex:i];
	
	        // cell的位置
	        CGFloat cellX = leftM + cellColumn * (cellW + columnM);
	        CGFloat cellY = 0;
	        if (maxYOfCellColumn == 0.0) { // 首行
	            cellY = topM;
	        } else {
	            cellY = maxYOfCellColumn + rowM;
	        }
	
	        // 添加frame到数组中
	        CGRect cellFrame = CGRectMake(cellX, cellY, cellW, cellH);
	        [self.cellFrames addObject:[NSValue valueWithCGRect:cellFrame]];
	
	        // 更新最短那一列的最大Y值
	        maxYOfColumns[cellColumn] = CGRectGetMaxY(cellFrame);
	    }
	
	    // 设置contentSize
	    CGFloat contentH = maxYOfColumns[0];
	    for (int j = 1; j<numberOfColumns; j++) {
	        if (maxYOfColumns[j] > contentH) {
	            contentH = maxYOfColumns[j];
	        }
	    }
	    contentH += bottomM;
	    self.contentSize = CGSizeMake(0, contentH);
	}
当屏幕滚动的时候会调用layoutSubviews方法，在这里我们根据cell是否显示对缓存池和展示池进行相关操作：
	
	
	- (void)layoutSubviews
	{
	    [super layoutSubviews];
	    // 向数据源索要对应位置的cell
	    NSUInteger numberOfCells = self.cellFrames.count;
	    for (int i = 0; i<numberOfCells; i++) {
	        // 取出i位置的frame
	        CGRect cellFrame = [self.cellFrames[i] CGRectValue];
	
	        // 优先从字典中取出i位置的cell
	        HMWaterflowViewCell *cell = self.displayingCells[@(i)];
	
	        // 判断i位置对应的frame在不在屏幕上（能否看见）
	        if ([self isInScreen:cellFrame]) { // 在屏幕上
	            if (cell == nil) {
	                cell = [self.dataSource waterflowView:self cellAtIndex:i];
	                cell.frame = cellFrame;
	                [self addSubview:cell];
	
	                // 存放到字典中
	                self.displayingCells[@(i)] = cell;
	            }
	        } else {  // 不在屏幕上
	            if (cell) {
	                // 从scrollView和字典中移除
	                [cell removeFromSuperview];
	                [self.displayingCells removeObjectForKey:@(i)];
	
	                // 存放进缓存池
	                [self.reusableCells addObject:cell];
	            }
	        }
	    }
	}
	
下面方法判断cell的frame是否在屏幕上：
	
	- (BOOL)isInScreen:(CGRect)frame
	{
	    return (CGRectGetMaxY(frame) > self.contentOffset.y) &&
	    (CGRectGetMinY(frame) < self.contentOffset.y + self.bounds.size.height);
	}
	下面方法根据标记取出缓存池中的cell
	
	- (id)dequeueReusableCellWithIdentifier:(NSString *)identifier
	{
	    __block HMWaterflowViewCell *reusableCell = nil;
	
	    [self.reusableCells enumerateObjectsUsingBlock:^(HMWaterflowViewCell *cell, BOOL *stop) {
	        if ([cell.identifier isEqualToString:identifier]) {
	            reusableCell = cell;
	            *stop = YES;
	        }
	    }];
	
	    if (reusableCell) { // 从缓存池中移除
	        [self.reusableCells removeObject:reusableCell];
	    }
	    return reusableCell;
	}
下面方法处理cell的点击

	- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event
	{
	    if (![self.delegate respondsToSelector:@selector(waterflowView:didSelectAtIndex:)]) return;
	
	    // 获得触摸点
	    UITouch *touch = [touches anyObject];
	    //    CGPoint point = [touch locationInView:touch.view];
	    CGPoint point = [touch locationInView:self];
	
	    __block NSNumber *selectIndex = nil;
	    [self.displayingCells enumerateKeysAndObjectsUsingBlock:^(id key, HMWaterflowViewCell *cell, BOOL *stop) {
	        if (CGRectContainsPoint(cell.frame, point)) {
	            selectIndex = key;
	            *stop = YES;
	        }
	    }];
	
	    if (selectIndex) {
	        [self.delegate waterflowView:self didSelectAtIndex:selectIndex.unsignedIntegerValue];
	    }
	}
这样一个模仿tableView的瀑布流就写好了，下篇博客会介绍利用collectionView写瀑布流的方法。