---
title: 详解自动布局(Masonry)实现九宫格
date: 2017-04-7 08:55:29
categories: 自动布局
toc: true
---

以前写TimeLine中照片九宫格布局是直接计算frame，今天想用自动布局实现。
<!--more-->
### 九宫格布局
使用自动布局，首先就必须知道给出了哪些条件。一般在TimeLine中照片九宫格布局给出的已知条件为：
1. 每个单元的宽cellWidth；
2. 每个单元的高cellHeight；
3. 每行有几个单元numPerRow；
4. 总共单元个数totalNum；
5. 每个单元与边界间距viewPadding；
6. 每个单元之间的间距viewPaddingCell。

![图 1](http://upload-images.jianshu.io/upload_images/760391-f9e752bfc5b8ffb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图 1是一个九宫格，黄色区域为父视图，由已知条件可知，它的大小是由里面的单元格布局决定的，所以，只要固定住里面的单元格，父视图就会自动固定住。
下面我们来添加约束：
1.所有单元格添加高度（height）和宽度（width）约束，如图 2所示，

![图 2](http://upload-images.jianshu.io/upload_images/760391-e6bb6f6d5a20d779.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.第一行相对父视图，添加top约束，如图三中紫色箭头所示，

![图 3](http://upload-images.jianshu.io/upload_images/760391-8e6d4276d6da399d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.非第一行添加对上一行单元的top约束，如图四红色箭头所示，

![图 4](http://upload-images.jianshu.io/upload_images/760391-87120ed7d6ea322c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.第一列添加对父视图的left约束，如图五墨绿色箭头所示


![图 5](http://upload-images.jianshu.io/upload_images/760391-a6130621009191d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.非第一列添加对上一个view的left约束，如图6深蓝色箭头所示


![图 6](http://upload-images.jianshu.io/upload_images/760391-3fd770e36d6a6966.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时候你会发现所有单元格都固定了，但是父视图的大小却不能固定，因为还不能得出父视图的宽和高

6.右上角（第一行&最后一列）添加对父视图right约束，如图7所示，2号单元格右侧绿色箭头

![图 7](http://upload-images.jianshu.io/upload_images/760391-e608894c34ba15d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

7.左下角（最后一行&第一列）添加对父视图的bottom约束，如图8所示，6号单元格底部紫色箭头


![图 8](http://upload-images.jianshu.io/upload_images/760391-b4360a56a4693dbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

talk is cheap show me the code.

![doge.gif](http://upload-images.jianshu.io/upload_images/760391-4e01cb9f1c2a49e5.gif?imageMogr2/auto-orient/strip)

```

/**
 九宫格布局（不限于九宫格，可以是N个格子），每个格子给定高（cellHeight）宽（cellWidth），
 每行格子数量（numPerRow），格子总数量（totalNum），格子与边界距离（viewPadding），格
 子之间的距离（viewPaddingCell）。

 @param cellWidth       格子宽度
 @param cellHeight      格子高度
 @param numPerRow       每行格子数量
 @param totalNum        格子总数量
 @param viewPadding     格子与边界距离
 @param viewPaddingCell 格子之间的距离
 @param superView       父视图
 */
- (void)gridWithCellWidth:(CGFloat)cellWidth
               cellHeight:(CGFloat)cellHeight
                numPerRow:(NSInteger)numPerRow
                 totalNum:(NSInteger)totalNum
              viewPadding:(CGFloat)viewPadding
          viewPaddingCell:(CGFloat)viewPaddingCell
                superView:(UIView *)superView

{
    
    __block UILabel *lastView = nil;// 创建一个空view 代表上一个view
    __block UILabel *lastRowView;// 创建一个空view 代表上一行view
    
    __block NSInteger lastRowNo = 0;//上一行的行号
    NSMutableArray *cells = [[NSMutableArray alloc] init];
    
    
    for (int i = 0; i < totalNum; i++) {
        
        UILabel *aLabel = [UILabel new];
        aLabel.text = [NSString stringWithFormat:@"%d",i];
        [superView addSubview:aLabel];
        aLabel.backgroundColor = [UIColor colorWithHue:(arc4random() % 256 / 256.0 ) saturation:( arc4random() % 128 / 256.0 ) + 0.5
                                          brightness:( arc4random() % 128 / 256.0 ) + 0.5 alpha:1.0];
        [cells addObject:aLabel];
    }
    
    // 循环创建view
    for (int i = 0; i < cells.count; i++)
    {
        
        UILabel *lb = cells[i];

        
        BOOL isFirstRow = [self isFirstRowWithIndex:i numOfRow:numPerRow];
        BOOL isFirstCol = [self isFirstColumnWithIndex:i numOfRow:numPerRow];
        
        BOOL isLastCol = [self isLastColumnWithIndex:i numOfRow:numPerRow totalNum:totalNum];
        BOOL isLastRow = [self isLastRowWithIndex:i numOfRow:numPerRow totalNum:totalNum];
        
        NSInteger curRowNo = i/numPerRow;
        if (curRowNo != lastRowNo)
        {//如果当前行与上一个view行不等，说明换行了
            lastRowView = lastView;
            lastRowNo = curRowNo;
        }
        
        // 添加约束
        [lb mas_makeConstraints:^(MASConstraintMaker *make) {
            make.width.equalTo(@(cellWidth));
            make.height.equalTo(@(cellHeight));
            
            if (isFirstRow)
            {
                make.top.equalTo(superView.mas_top).with.offset(viewPadding);
            }
            else
            {
                if (lastRowView)
                {
                    make.top.equalTo(lastRowView.mas_bottom).with.offset(viewPaddingCell);
                }
            }
            
            if (isFirstCol)
            {
                make.left.equalTo(superView.mas_left).with.offset(viewPadding);
            }
            else
            {
                if (lastView)
                {
                    make.left.equalTo(lastView.mas_right).with.offset(viewPaddingCell);
                }
            }
            
            if (isFirstRow && isLastCol)
            {
                make.right.equalTo(superView.mas_right).with.offset(-viewPadding);
            }
            
            if (isLastRow && isFirstCol)
            {
                make.bottom.equalTo(superView.mas_bottom).with.offset(-viewPadding);
            }
            
        }];
        
        
        
        // 每次循环结束 此次的View为下次约束的基准
        lastView = lb;
    }
}
```

代码中有一些判断，比如是否为第一行，

```
/**
 是否第一行

 @param index    当前下标
 @param numOfRow 每行个数

 @return YES OR NO
 */
- (BOOL)isFirstRowWithIndex:(NSInteger)index numOfRow:(NSInteger)numOfRow
{
    if (numOfRow != 0)
    {
        return index/numOfRow == 0;
    }
    return NO;
}
```


是否为第一列，
```
/**
 是否第一列
 
 @param index    当前下标
 @param numOfRow 每行个数
 
 @return YES OR NO
 */
- (BOOL)isFirstColumnWithIndex:(NSInteger)index numOfRow:(NSInteger)numOfRow
{
    if (numOfRow != 0)
    {
        return index%numOfRow == 0;
    }
    return NO;
}
```
是否为最后一行，
```
/**
 是否最后一行
 
 @param index    当前下标
 @param numOfRow 每行个数
 
 @return YES OR NO
 */
- (BOOL)isLastRowWithIndex:(NSInteger)index numOfRow:(NSInteger)numOfRow totalNum:(NSInteger)totalNum
{
    NSInteger totalRow = ceil(totalNum/((CGFloat)numOfRow));//总行数
    
    if (numOfRow != 0)
    {
        return index/numOfRow == totalRow - 1;
    }
    return NO;
}
```

是否为最后一列

```
/**
 是否最后一列
 
 @param index    当前下标
 @param numOfRow 每行个数
 
 @return YES OR NO
 */
- (BOOL)isLastColumnWithIndex:(NSInteger)index numOfRow:(NSInteger)numOfRow totalNum:(NSInteger)totalNum
{
    if (numOfRow != 0)
    {
        if (totalNum < numOfRow)
        {//总数小于每行最大个数时，如果index是最后一个，那么也是最后一列
            return index == totalNum-1;
        }
        return index%numOfRow == numOfRow - 1;
    }
    return NO;
}
```
**注意**：这里有个地方要注意，当你的单元格总数（totalNum ）小于每行个数 （numOfRow），比如总共有2个单元格，每行排三个，那么最后一个即为最后一列。

上面这四个判断在很多地方都可以用到，可以记下备用🙂。

然后是上一行的view判断也需要注意。

其实这不单单只是九宫格布局，N个单元格布局也是可以的，感兴趣的小伙胖可以自行测试（好吧，估计你从我写的方法名已经看出来了，每行个数和总个数我都没有写死😂）。

### 另一种九宫格
这里的九宫格布局是子视图固定，而父视图由子视图决定，还有另一种情况：父视图高宽固定，子视图与父视图边界距离给定，子视图间距给定。
知道怎么布局吗？可以先思考一下。
|
|
|
|
|
|
好吧，揭晓答案：只要按照第一种九宫格前5个步骤来添加约束即可，去掉最后两步。



### 总结
一开始我只是想布局一个九宫格，但是后来又想，如果需求扩展到了N个单元，该如何实现呢，我的办法是从九宫格开始，由小及大来推导，然后就是要知道自动布局需要添加哪些约束，能够完整的固定视图，不能多，也不要少，这是很重要的。
