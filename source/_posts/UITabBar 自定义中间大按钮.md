---
title: UITabBar 自定义中间大按钮
date: 2017-05-22 08:55:29   //在此处输入你编辑这篇文章的时间。
categories: UI   //在此处输入这篇文章的分类。
toc: true  //在此处设定是否开启目录，需要主题支持。
---

在项目中经常会有这种需求：在tabBar的正中间放置一个大按钮，有时候会超出tabBar的可点击范围。如下图的闲鱼：

![闲鱼TabBar](http://upload-images.jianshu.io/upload_images/760391-c8711365e3606a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

主要思路就是：
- 自定义UITabBarController
- 自定义UITabBar
- 在自定义的UITabBarController中用自定义的UITabBar替换掉原有的tabbar。

首先，利用KVC在自定义的UITabBarController中用自定义的UITabBar替换掉原有的tabbar：

```
    CustomTabBar *myTabBar = [[CustomTabBar alloc] init];
    [self setValue:myTabBar forKey:@"tabBar"];
```

大按钮好解决，用一张大点的图片即可。但是你会发现点击超出tabbar顶部的部分是不能响应的。如果你知道hit-test的工作原理，就会知道为什么超出部分不能响应。响应的四个条件是：
- 视图不是隐藏的: self.hidden == NO
- 视图是允许交互的: self.userInteractionEnabled == YES
- 视图透明度大于0.01: self.alpha > 0.01
- 视图包含这个点: pointInside:withEvent: == YES
在这里就是由于不满足第四个条件，即点击的point没有落在父视图的bounce之内，所以无法响应。

系统的hit-test方法实现大概如下:

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01)
    {
        return nil;
    }
    
    if ([self pointInside:point withEvent:event])
    {
        for (UIView *subview in [self.subviews reverseObjectEnumerator])
        {
            CGPoint convertedPoint = [subview convertPoint:point fromView:self];
            UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
            if (hitTestView)
            {
                return hitTestView;
            }
        }
    }
    return nil;
}
```

通常解决这个问题的做法是将响应区域扩大，这个我还没有研究。

不过我的思路特殊一点：在自定义的UITabBar中重写hit-test方法，将点击的point进行修改，让它落在可点击范围之内。

由于是点击的point的y坐标超出了tabbar范围，那么只要修改这个y坐标，让他落在tabbar的可点击范围内即可。

修改y坐标的代码如下所示：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01)
    {
        return nil;
    }
    
    if ([self pointInside:point withEvent:event])
    {
        return [self mmHitTest:point withEvent:event];
    }
    else
    {
        CGPoint otherPoint = CGPointMake(point.x, point.y + self.effectAreaY);
        return [self mmHitTest:otherPoint withEvent:event];
    }
    return nil;
}

- (UIView *)mmHitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    for (UIView *subview in [self.subviews reverseObjectEnumerator])
    {
        CGPoint convertedPoint = [subview convertPoint:point fromView:self];
        UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
        if (hitTestView) {
            return hitTestView;
        }
    }
    return nil;
}
```
其中effectAreaY是超出tabbar的垂直距离，比如大按钮顶部超出20，那么这个值就是20，你可以根据需求进行调整。

到这里，超出tabbar区域的大按钮点击问题就得到解决了，但是我反复点了一下，发现一个小问题，就是每个tabbar的item超出部分都能点击，这是我们不想要的。

我们想要的是落点在中间按钮范围内时才去修改这个落点，所以，加一个判断即可，只要点击的point的x坐标在中间大按钮范围之内就去修改，代码如下：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01)
    {
        return nil;
    }

    if ([self pointInside:point withEvent:event])
    {
        return [self mmHitTest:point withEvent:event];
    }
    else
    {
        CGFloat tabBarItemWidth = self.bounds.size.width/self.items.count;
        CGFloat left = self.center.x - tabBarItemWidth/2;
        CGFloat right = self.center.x + tabBarItemWidth/2;
        
        if (point.x < right &&
            point.x > left)
        {//当点击的point的x坐标是中间item范围内，才去修正落点
            CGPoint otherPoint = CGPointMake(point.x, point.y + self.effectAreaY);
            return [self mmHitTest:otherPoint withEvent:event];
        }
    }
    return nil;
}

- (UIView *)mmHitTest:(CGPoint)point withEvent:(UIEvent *)event
{
    for (UIView *subview in [self.subviews reverseObjectEnumerator])
    {
        CGPoint convertedPoint = [subview convertPoint:point fromView:self];
        UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
        if (hitTestView) {
            return hitTestView;
        }
    }
    return nil;
}
```

另外，如果想调整图片和文字位置，调整tabbarItem的属性：
 ```
        [vc.tabBarItem setImageInsets:UIEdgeInsetsMake(-30, 0, 30, 0)];//修改图片偏移量，上下，左右必须为相反数，否则图片会被压缩
        [vc.tabBarItem setTitlePositionAdjustment:UIOffsetMake(0, -30)];//修改文字偏移量
```
vc是每个根控制器。

到这里，就完美的解决了自定义大按钮及其点击问题，相当简单。


总的来说要点就是：
- 替换系统的UITabBar
- 修改点击point的落点
- 判断什么时候才去修改落点。

最后附上Demo：[HitTesting](https://github.com/flowyears/HitTesting)

end~ 如果觉得有用，请点个赞👍，谢谢~
