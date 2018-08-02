---
title: UITableView-Auto-Layout-(iOS 8+)
date: 2017-04-7 08:55:29
categories: 自动布局
tags : iOS,autolayout
toc: true
---

>翻译自stack overflow的一个[回答](http://stackoverflow.com/questions/18746929/using-auto-layout-in-uitableview-for-dynamic-cell-layouts-variable-row-heights/18746930#18746930)，对于iOS自动布局很有用处。翻译水平有限，且现在公司的app都是iOS 8+，所以只翻译了iOS 8+的部分。如有错误，欢迎指正！谢谢！

<!--more-->

## 概念描述
无论你是哪个iOS开发版本，下面前2个步骤都适用。

### 设置 & 添加约束
>content-hugging and compression-resistance 以下简称CHCR；
固有内容尺寸：intrinsic content size；

在你的`UITableViewCell `子类中，添加约束使cell的subviews的边缘固定到cell的contentView的边缘（最重要的是顶部(top)和底部(bottom)边缘）。**注意：不要把subviews固定到cell自身；只能添加到cell的 contentView！**通过确保CHCR在每个subview的垂直方向不被你添加的更高优先级约束重写，来让这些subviews的固有内容尺寸驱动表单元的内容视图高度（[嘿？点击这里](http://stackoverflow.com/questions/22599069/what-is-the-content-compression-resistance-and-content-hugging-of-a-uiview)）。

Remember, the idea is to have the cell's subviews connected vertically to the cell's content view so that they can "exert pressure" and make the content view expand to fit them.使用一个例子，带有几个子视图的cell，这里有张直观的插图上面有一些（不是全部）你好像需要的约束：


![](http://upload-images.jianshu.io/upload_images/760391-8e0ac86893b18698.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

想象一下，随着更多文本被添加到在上面例子cell中的多行label，它将要垂直增长以适应文本，这将有效地强制cell增长高度（当然，为了正确地工作，你需要得到约束！）。

获取正确的约束绝对是自动布局获取动态cell高度最难且最重要的一部分。如果你在这里犯了错，它将阻止你的一切工作，所以抓紧时间！我建议在代码中设置你的约束，因为你确切知道哪些约束被添加到哪里，并且当事情出错时调试起来容易得多.在代码中添加约束和IB一样简单并且当你使用一个神奇的开放源码API的时候回更强大--这里就有一个我专门设计和维护和使用的：[https://github.com/smileyborg/PureLayout](https://github.com/smileyborg/PureLayout).
 
* 如果你在代码中添加约束，你应该在UITableViewCell 子类updateConstraints方法中执行一次添加。注意updateConstraints可能会不止调用一次，所以为了避免添加同样的约束多次，可以用一个bool值比如didSetupConstraints来检查包含的约束只添加一次（在约束添加之后设置为YES）。另一方面，如果你又代码要更新已存在的约束（比如调整constant属性相同的约束），将更新约束的代码发在updateConstraints中但是在didSetupConstraints的检查之外，这样更新的代码就可以在每次调用的时候执行了。

### Determine Unique Table View Cell Reuse Identifiers

对于在cell中的每一组唯一约束，请使用唯一的cell reuse identifier。换句话说，如果你的单元格有多个唯一的布局，每个独特的布局应该得到它自己的重用标识符（当你的cell发生变化有不同数量的子视图或是以一种独特的方式布置子视图时，你需要使用一个新的reuse identifier）。

比如，例如，如果你在每个单元格中显示一个电子邮件消息，你可能有4个独特的布局：只有subject的消息；带subject和一个body的消息；带subject和照片附件的消息；带subject，body和照片附件的消息。每种布局都有完全不同的约束来实现它，所以一旦cell被初始化并且这些cell 类型之一添加了约束，cell就需要为指定的cell type获取一个唯一的reuse identifier。这意味着当你复用cell出列时，为这个cell type的约束已经被添加并且准备执行了。

注意，由于固有内容尺寸的差异，具有相同约束（类型）的单元格可能仍有不同的高度！不要因为不同的内容尺寸混淆根本不同的布局（不同的约束）和不同的算出来的视图frame。

* 不要将具有完全不同约束集（sets of constraints）的cell添加到同一重用池（ reuse pool）中（比如使用相同的reuse identifier）并且在后面的队列中尝试删除旧约束从头开始添加新约束。内部自动布局引擎不是设计来处理大规模的约束变化，你会看到大量的性能问题。



### Enable Row Height Estimation

在iOS 8中，苹果已经把许多在iOS 8之前需要由你完成的工作内部化实现了。为了让 self-sizing cell 机制工作，你必须首先设置tableview的`rowHeight `属性值为`UITableViewAutomaticDimension `.然后，你仅需要通过设置tableview的estimatedRowHeight属性值为一个非零值就可以开启行高估算（Row Height Estimation）。例如：


```
self.tableView.rowHeight = UITableViewAutomaticDimension;
self.tableView.estimatedRowHeight = 44.0; // set to whatever your "average" cell height is
```

这个所做的是给tableview还未出现在屏幕上的cell的豪赌提供一个临时估算或占位。然后，当这些cell要滚动到屏幕上时实际的行高将会被计算。为了确定每行的实际高度，tableview自动询问每个cell它的contentview基于已知的content view固定宽度和你添加到cell的content view 及 subviews的自动布局约束所需要的高度。一旦cell的真实高度确定，估算高度就会更新为真实高度（任何tableview的contentSize/contentOffset调整需要你来做）。

一般来讲，你提供的估算并不需要非常准确--它只是用来合适的排列在tableview中的scroll indicator，在滚动屏幕上的单元格时，表视图可以很好地调整滚动指示器的估计值。你应该将表视图（在viewDidLoad或类似方法中）中的estimatedRowHeight属性设置为一个常量值，即“平均”行高。只有当你的行高有极大的可变性（例如相差一个数量级），你注意到滚动指示器“跳跃”，如果你麻烦执行`tableView：estimatedHeightForRowAtIndexPath：`做最小的计算，为每行返回更准确的估算值。

[iOS的8示例项目](https://github.com/smileyborg/TableViewCellWithAutoLayoutiOS8) -需要iOS 8
