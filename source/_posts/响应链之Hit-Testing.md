---
title: 响应链之Hit-Testing
date: 2017-05-22 08:55:29   //在此处输入你编辑这篇文章的时间。
categories: 响应链   //在此处输入这篇文章的分类。
toc: true  //在此处设定是否开启目录，需要主题支持。
---

## Hit-Testing 是什么
Hit-Testing 是一个决定一个点（比如一个触摸点）是否落在一个给定的物理对象上（比如绘制在屏幕上的UIView）的一个过程。



## Hit-Testing 执行时机
Hit-Testing是在每次手指触摸时执行的。并且是在任何视图或者手势收到UIEvent（代表触摸属于的事件）之前。

## Hit-Testing 的实现

实现：Hit-Testing采用深度优先的反序访问迭代算法（先访问根节点然后从高到低访（从离用户近的视图或者说是后添加的视图）问低节点）。这种遍历方法可以减少遍历迭代的次数。

结束条件：**一旦找到最深的包含触摸点的后裔视图就停止遍历**（注意，是最深的）。


下面举例说明：


![图1](http://upload-images.jianshu.io/upload_images/760391-3249f23ee5246a75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，视图A\B\C依次添加到视图上，比如View B的添加比A晚比C早，而自视图B.1比B.2添加的要早。


![图2](http://upload-images.jianshu.io/upload_images/760391-241b32615852e167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过深度优先的反向遍历允许一旦找到第一个最深的后裔包含触摸点的视图就停止遍历，如上图所示，找到B.1后就停止，不会继续遍历A视图。



遍历算法以向`UIWindow`（视图层次结构的根视图）发送`hitTest:withEvent:`消息**开始**。这个方法返回的值就是包含触摸点的最前面的视图。
下面流程图说明了`hit-test`逻辑:



![图3](http://upload-images.jianshu.io/upload_images/760391-28b3f924086a454b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



下面的代码展示了原生`hitTest:withEvent:`可能的实现：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01) {
        return nil;
    }
    if ([self pointInside:point withEvent:event]) {
        for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
            CGPoint convertedPoint = [subview convertPoint:point fromView:self];
            UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
            if (hitTestView) {
                return hitTestView;
            }
        }
        return self;
    }
    return nil;
}
```

`hitTest:withEvent:`方法首先检查视图是否允许接收触摸事件。视图允许接收触摸事件的四个条件是：
- 视图不是隐藏的: self.hidden == NO
- 视图是允许交互的: self.userInteractionEnabled == YES
- 视图透明度大于0.01: self.alpha > 0.01
- 视图包含这个点: pointInside:withEvent: == YES

图2遍历顺序为：UIWindow->MainView->View C->ViewB->View B.2->View B.1.

解释一下为什么顺序是这样：
1. 首先遍历UIWindow，然后MainView
2.  MainView 有三个子视图ABC，根据算法描述中所讲，首先遍历离用户最近的视图，所以，先遍历View C
3. 每次遍历时需要判断接收触摸的四个条件，由于落点不在C中，所以在hitTest遍历C时返回nil，然后继续遍历View B,
4. 然后遍历View B的两个子视图，与第2点判断条件一样，先遍历View B.2
5. 由于落点不在B.2中，所以继续遍历B.1,由于此时满足结束条件，即接收触摸事件并且B.1没有子视图，遍历到此结束。




## 覆盖hitTest:withEvent:的一些用途
`hitTest:withEvent:`可以被覆盖,那么覆盖它可以做些什么呢？

### 1.增加视图的触摸区域

![hit-test-increase-touch-area.png](http://upload-images.jianshu.io/upload_images/760391-220f9865d5a14335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，蓝色按钮太小，如果采用设置UIButton的image来放大点击区域，调整按钮坐标的代码就很不好看了，如果用覆盖`hitTest:withEvent:`的方法来解决这个方法，就要优雅一些，自定义UIButton,覆盖hit-testing方法，代码如下：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    if (!self.isUserInteractionEnabled || self.isHidden || self.alpha <= 0.01) {
        return nil;
    }
    CGRect touchRect = CGRectInset(self.bounds, -10, -10);
    if (CGRectContainsPoint(touchRect, point)) {
        for (UIView *subview in [self.subviews reverseObjectEnumerator]) {
            CGPoint convertedPoint = [subview convertPoint:point fromView:self];
            UIView *hitTestView = [subview hitTest:convertedPoint withEvent:event];
            if (hitTestView) {
                return hitTestView;
            }
        }
        return self;
    }
    return nil;
}
```

### 2.传递触摸事件给父视图
有的时候对于一个视图忽略触摸事件并传递给下面的视图是很重要的。例如，假设一个透明的视图覆盖在应用内所有视图的最上面。覆盖层有子视图应该相应触摸事件的一些控件和按钮。但是触摸覆盖层的其他区域应该传递给覆盖层下面的视图。为了完成这个行为，覆盖层需要覆盖hitTest:withEvent:方法来返回包含触摸点的子视图中的一个，然后其他情况返回nil，包括覆盖层包含触摸点的情况：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *hitTestView = [super hitTest:point withEvent:event];
    if (hitTestView == self) {
        hitTestView = nil;
    }
    return hitTestView;
}
```
### 3.传递触摸事件给子视图

![hit-test-pass-touches-to-subviews.png](http://upload-images.jianshu.io/upload_images/760391-977751130665be15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

蓝色方框是一个图片浏览器，在蓝色方框内滑动，可以翻动图片，但是在方框之外是无法响应的，因为手指落点不在图片浏览器的bounces里面，那么如何让手指落在上图位置时，也可以滚动图片呢？方法是在图片浏览器的父视图中，重载`hitTest:withEvent:`方法，当触摸到图片浏览器自视图之外的视图时，返回图片浏览器即可：

```
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
    UIView *hitTestView = [super hitTest:point withEvent:event];
    if (hitTestView) {
        hitTestView = self.scrollView;
    }
    return hitTestView;
}
```
### 4.响应子view超出了父view的bounds事件
比如自定义Tabbar中间的大按钮，点击超出Tabbar bounds的区域也需要响应，此时重载父view的`hitTest: withEvent:`方法，**去掉点击必须在父view内的判断**，然后子view就能成为 hit-test view用于响应事件了。[这篇文章](http://www.jianshu.com/p/3126db96e8d3)详细的描述了如何实现。

参考：[iOS中的Hit-Testing](http://joywii.github.io/blog/2015/03/17/ioszhong-de-hit-testing/)
