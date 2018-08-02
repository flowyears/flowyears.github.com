
---

title: 《Auto-Layout-Guide》笔记
date: 2017-04-7 08:55:29
categories: 自动布局
toc: true
---

### 概念
#### 约束优先级
所有约束都有1-1000的优先级。优先级为1000的约束是必须的。其它约束都是可选的。

<!--more-->
>注意：不要强迫使用所有1000个优先级值。实际上，优先级通常聚集在系统定义的低（250），中（500），高（750）和必须（1000）周围。可能需要让约束的优先级比这些值高或低1-2点，来帮助阻止绑定（teis）。如果优先级远远超出这些值，可能需要重新检查布局逻辑。

#### 固有内容尺寸(Intrinsic Content Size)
有些视图根据当前给定的内容有个自然的尺寸。这个尺寸被称为`固有内容尺寸(Intrinsic Content Size)`。

不是所有视图都有固有内容尺寸。对于有固有内容尺寸的视图，**它可以定义视图的高度，宽度或者两者都定义**。下面的表格列出了一些例子（这里只标注iOS的，mac OS已删除）。

视图	|固有内容尺寸
--|--
UIView	|没有固有内容尺寸
UISlider	|只定义了宽度。
UILabel,UIbutton,UISwitch,UITextField|	同时定义了宽度和高度。
UITextView,UIImageView	| 固有内容尺寸可以改变。


#### Content hugging & Compression resistance
自动布局使用了一对约束来呈现一个视图的每个维度。content hugging向内抱紧视图，使紧贴内容（简称抱紧我）。压缩阻力（compression resistance）向外推视图，使其不剪辑内容（简称别挤我）。

Content hugging优先级越高，抱的越紧；compression resistance优先级越高，压缩阻力越大，内容越不容易被剪辑。

下面举两个例子说明一下：
* 例1（Content hugging）
通常会有这样一个功能，左侧label，提示输入名字，右侧是一个textfield，用于输入名字，如下图所示：


![](http://upload-images.jianshu.io/upload_images/760391-3843819524c2a48a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时候我们希望name足够小，能显示name足以，textfield则尽量拉伸。
这时候只需要name的Content hugging属性优先级大于textfield就可以了。

* 例2（compression resistance）
给一个label添加宽度为80的约束，如果label的文字超过80，那么显示就会这样：

![](http://upload-images.jianshu.io/upload_images/760391-e2ef14402782078f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为compression resistance默认优先级为750，而高宽这些数据的优先级默认是required（即1000）。所以我们只要将宽度的约束优先级修改一下，比750小即可，比如749.这样修改后，效果如下：

![](http://upload-images.jianshu.io/upload_images/760391-2cde8b491ccabb84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样应该就能很好理解两个属性的意义了。

常见关系优先级：

属性或关系|优先级
--|--
equal|1000
greater-than-or-equal|1000
 less-than-or-equal|1000
Content hugging|250
Compression resistance|750

### 工具
#### Align工具
Align工具可以快速对齐布局中的项。选择想要**对齐**的项，然后单击Align工具。界面生成器显示一个弹出框视图，其中包括一些可能的对齐选项。

![](http://upload-images.jianshu.io/upload_images/760391-2a0c61da751892b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### Pin工具
Pin工具可以快速定义视图相对于它**邻居**的位置，或者快速定义视图的尺寸。选择想要固定(pin)位置或尺寸的项，然后点击Pin工具。界面生成器显示一个弹出框视图，其中包括一些选项。


![](http://upload-images.jianshu.io/upload_images/760391-3c95091daf09a074.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

弹出框顶部区域可以固定选中项的开头，顶部，结尾或底部边缘到它最近的邻居。关联的数字表示画布中两个项之间的当前间隔。可以输入自定义的间隔，或者点击三角形，设置它被约束到哪个视图，或者选择标准间隔。Constrain to margins复选框决定约束使用父视图的页边留白（margin）还是它的边缘（edge）。


![](http://upload-images.jianshu.io/upload_images/760391-753104f642a87016.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Resolve Auto Layout Issues工具
Resolve Auto Layout Issues工具提供了一些选项用来修复常见的自动布局问题。菜单的上半部分只影响当前选中的视图。下半部分选项影响场景中所有视图。

![](http://upload-images.jianshu.io/upload_images/760391-7c68e65d852366cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以使用这个工具更新视图的框架（frame），基于当前的约束，或者根据视图在画布中的当前位置更新约束。还可以添加缺失的约束，清理约束，或者重置视图为界面生成器推荐的约束。

### 调试技巧
在自动布局过程中，出现错误是经常的事情，自动布局中的错误主要分为三个类型：
* **不可满足的（unsatisfiable）布局。**布局没有有效的解。更多信息请参考[不可满足的布局](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/ConflictingLayouts.html#//apple_ref/doc/uid/TP40010853-CH19-SW1)。
* **有歧义的布局。**你的布局有两个或多个可能的解。更多信息请参考[有歧义的布局](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/AmbiguousLayouts.html#//apple_ref/doc/uid/TP40010853-CH18-SW1)。
* **逻辑错误。**布局逻辑中存在bug。更多信息请参考[逻辑错误](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/LogicalErrors.html#//apple_ref/doc/uid/TP40010853-CH20-SW1)。

调试方法：
查看控制台的打印信息，有时候打印的信息比较多，不方便看，添加标识符，是一个很好的方法。

### 高级自动布局
#### 推迟的布局过程
自动布局安排一个布局过程在不久的将来取代直接更新受影响的view的frame。该推迟的过程更新布局的约束，然后计算视图层次结构中所有视图的frame。

可以通过调用[setNeedsLayout](https://developer.apple.com/reference/uikit/uiview/1622601-setneedslayout)方法或者[setNeedsUpdateConstraints](https://developer.apple.com/reference/uikit/uiview/1622450-setneedsupdateconstraints)方法，调度自己的推迟的布局过程。

推迟的布局过程实际涉及视图层级结构的两个过程：
1.更新过程根据需要更新约束。
2.布局过程根据需要重新定位视图的frame。

1. 更新过程
系统遍历视图层级结构，并在所有视图控制器上调用[updateViewConstraints](https://developer.apple.com/reference/uikit/uiviewcontroller/1621379-updateviewconstraints)方法，在所有视图上调用[updateConstraints](https://developer.apple.com/reference/uikit/uiview/1622512-updateconstraints)方法。你可以覆写这些方法，来优化约束的改变（查看[批量改变](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/ModifyingConstraints.html#//apple_ref/doc/uid/TP40010853-CH29-SW2)）。

2. 布局过程
系统再次遍历视图层级结构，并在所有视图控制器上调用[viewWillLayoutSubviews](https://developer.apple.com/reference/uikit/uiviewcontroller/1621437-viewwilllayoutsubviews)方法，在所有视图上调用[layoutSubviews](https://developer.apple.com/reference/uikit/uiview/1622482-layoutsubviews)。默认情况下，layoutSubviews方法使用自动布局引擎计算的矩形更新每个子视图的frame。你可以覆写这些方法来修改布局（查看[自定义布局](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/ModifyingConstraints.html#//apple_ref/doc/uid/TP40010853-CH29-SW4)）。


#### 批量改变
影响变化发生后，立即更新约束几乎总是更干净和容易。推迟这些改变到一个之后的方法会让代码复杂，更难理解。

然而，有些时候你可能基于性能原因，希望批量修改。只有在就地改变约束太慢，或者当视图做了很多多余的改变时，才应该这么做。

要想批量改变，在持有约束的视图上调用[setNeedsUpdateConstraints](https://developer.apple.com/reference/uikit/uiview/1622450-setneedsupdateconstraints)方法，而不是直接做出改变。然后，覆写视图的[updateConstraints](https://developer.apple.com/reference/uikit/uiview/1622512-updateconstraints)方法，来修改受影响的约束。

>**提示**:你的updateConstraints实现必须尽可能高效。不要禁用所有约束，然后启用你需要的。相反，你的应用程序必须有些方式来追踪你的约束，并在每个更新过程中验证它们。只有变化的项需要改变。在每一个更新过程中，你必须确保应用程序的当前状态有合适的约束。

**总是在你实现的updateConstraints方法的最后一步调用父类的实现**。


不要在你的updateConstraints方法中调用setNeedsUpdateConstraints。调用setNeedsUpdateConstraints调度另一个更新过程，创建了一个反馈回路（feedback loop）。


参考：
[Auto Layout Guide](https://developer.apple.com/library/content/documentation/UserExperience/Conceptual/AutolayoutPG/ModifyingConstraints.html#//apple_ref/doc/uid/TP40010853-CH29-SW1)
[自动布局指南](http://www.jianshu.com/p/5d5562106756)
