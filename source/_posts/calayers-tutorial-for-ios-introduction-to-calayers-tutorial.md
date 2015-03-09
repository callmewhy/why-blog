title: CALayers 教程：初步认识 CALayer
tags: CALayer
date: 2014-10-15 10:26:58
categories: 翻译笔记
description: 每一个酷炫的视图背后都有一群默默无闻的图层。
---

如果你是一个 iOS 开发者，对于 `UIView` 肯定不会陌生 – `UIButton`、`UITextArea`、`UIWebView`，这些都是 UIView 的子类。

但是你可能还不知道 `UIView` 是基于 `CALayer` 构建的！至少在很长的一段时间里我并不知道。

懂点 `CALayer` 的内容还是很不错的，因为你可以用它来实现一些不错的视觉效果，而且他们对于理解 `Core Animation` 很有帮助。

在这篇 `CALayer` 的教程中，我们将创建一个简单的应用，新建一个 layer 并对它的外观进行各种改动，学习 `CALayer` 的基本使用方法。

友情提示， `CALayer` 的系列教程是针对有一定 iOS 开发基础的同学们的，如果你是一个完全的新手，可以先看看[这个系列](http://www.raywenderlich.com/?p=1797)的基础教程。

好的，让我们开始吧！


## CALayer 是什么

简单来说，`CALayer` 代表的就是屏幕上的一片矩形区域。

“嗨哥们儿等下，”你可能要说，“那不是 `UIView` 干的事情吗！”啊哈是的没错，但是真相是：每个 `UIView` 都包含了一个 `Root Layer` 。你可以用下面这段代码获取到这个根图层：

    CALayer *myLayer = myView.layer;

`CALayer` 有很多很多的属性，想必是极好的，你可以通过设置它的属性实现各种视觉效果，比如：

- 调整图层的大小和位置
- 调整图层的背景颜色
- 修改图层的内容 (一个图片，或者是用 [`CoreGraphics`](http://www.raywenderlich.com/tag/core-graphics) 绘制的东西)
- 图层是否圆角
- 添加黑色投影
- 添加描边的边框
- 还有其他各种玩法！

你可以用这些属性创建一些视觉效果。比如，你想要把一张图片放在一个白色的边框里，然后再加上个黑色的阴影。不用 PS ，也不用一大串 [`CoreImage`](http://www.raywenderlich.com/tag/core-graphics) 的代码，只需要这么一小段代码，你就可以用 [`CALayer`](http://tumbljack.com/post/506095243/subtle-effects-with-calayer) 实现！

啊哈补充一点， `CALayer` 的属性绝大多数都是可以做成动画效果的。举个例子，你可以让你的图像显示圆角，然后点击一个按钮，圆角又会恢复到原来方角的样子。好好利用这个特性可以做出很多精致的效果！


你可以直接使用 `CALayer` ，你也可以用它的子类：CAGradientLayer、CATextLayer、CAShapeLayer等等。 `UIView` 默认的图层类是原始的 `CALayer` 类。

## 开始动手

没什么比自己动手更好的学习方法了！打开你的 Xcode ，新建一个名为 LayerFun 的项目，项目类型是 Single View Application，这样一开始我们就有了一个空白的 `UIView` 。


现在你可以自己动手改一改这个 UIView 中 layer 的属性了：

    override func viewWillAppear() {
        self.view.layer.backgroundColor = UIColor.orangeColor().CGColor
        self.view.layer.cornerRadius = 20.0
        self.view.layer.frame = CGRectInset(self.view.layer.frame, 20, 20)
    }



让我们一点一点分析一下上面的代码：

- 你可以用 `self.view.layer` 获取到 `UIView` 的图层的指针。你可以通过 `self.view` 获取到当前 ViewController  的根视图，你可以通过 `view.layer` 获取到视图的默认图层。默认情况下这个图层的类型是 `CALayer` (不是子类)。
- 接下来我们把背景颜色设置成了橘黄色。注意， `backgroundColor` 这个属性实际上是一个 `CGColor` 的类型，不过把 `UIColor` 转 `CGColor` 很容易。
- 然后我们通过设置 `cornerRadius` 给图层加了圆角效果。这个值是圆角效果的圆的半径，在这里我们把它设置成了20。
- 最后我们通过 `CGRectInset` 稍稍缩放一下图层的边框，这样可以更方便的展示效果。`CGRectInset` 函数有两个参数，一个是边框，另一个是缩放的值 (x方向 和 y方向) ，并且返回一个新的边框。

运行一下会发现屏幕中间出现一个橘黄色的长方形：

![](http://cdn1.raywenderlich.com/wp-content/uploads/2010/12/Simple-CALayer.jpg)

## CALayers 和 Sublayers

就像 UIView 有很多 subview 一样， CALayer 也有 sublayer 。你可以很容易的创建一个新的 CALayer ：

    var sublayer = CALayer()


创建了一个新的 CALayer 之后，你可以设置它的任何属性，不过有一个是必须设置的，那就是它的 frame (或者 bounds/position) 。毕竟，所有的图层都要知道自己该有多大 (或者自己该在哪里) 。当你设置了这些基本的属性之后，你可以通过下面的代码把它加到另一个 layer 里：

    self.view.layer.addSublayer(sublayer)

你可以给我们的视图的图层添加一个图层，添加如下代码：
    
    override func viewWillAppear(animated: Bool) {
        
        self.view.layer.backgroundColor = UIColor.orangeColor().CGColor
        self.view.layer.cornerRadius = 20.0
        self.view.layer.frame = CGRectInset(self.view.layer.frame, 20, 20)

        var sublayer = CALayer()
        sublayer.backgroundColor = UIColor.blueColor().CGColor;
        sublayer.shadowOffset = CGSizeMake(0, 3);
        sublayer.shadowRadius = 5.0;
        sublayer.shadowColor = UIColor.blackColor().CGColor;
        sublayer.shadowOpacity = 0.8;
        sublayer.frame = CGRectMake(30, 30, 128, 192);
        self.view.layer.addSublayer(sublayer)
        
    }

上面的代码创建了一个新的图层，并且设置了一些属性 - 包括一些设置阴影的代码，你以前可能没见过。可以看到这几行代码就添加了阴影效果，十分简单。

运行的效果是这个样子的：

![](http://cdn3.raywenderlich.com/wp-content/uploads/2010/12/CALayer-Sublayer-with-Shadow.jpg)



## 设置 CALayer 的图片内容

`CALayer` 最酷的事情之一就是它可以包含除了纯色之外的东西，比如它里面可以放上一张图片。

那么接下来让我们给这个图层换上图片，你可以在[这里](http://d1xzuxjlafny7l.cloudfront.net/downloads/BattleMapSplashScreen.jpg) 下载一个示例图片，当然也可以自己弄一张图片做测试。

把图片加到项目中，然后在把 sublayer 加到图层之前加上如下代码：


    sublayer.contents = UIImage(named: "BattleMapSplashScreen")?.CGImage
    sublayer.borderColor = UIColor.blackColor().CGColor
    sublayer.borderWidth = 2.0


上面的代码将图层的 `contents` 设置成了一张图片 (代码写的十分清晰) ，然后又设置了 `borderColor` 和 `borderWidth` ，添加了一个黑色的边框。

运行一下，你可以看到原本蓝色的背景以及不复存在，取而代之的是我们设置的图片的内容：

![](http://cdn4.raywenderlich.com/wp-content/uploads/2010/12/CALayer-with-Image-Content.jpg)


## 关于设置圆角和图片

设置圆角看起来非常的酷，但是如果你设置了 contents 为图片，那么图片会绘制在圆角的外面，导致圆角的设置失败。解决的方案是设置 `.masksToBounds` 属性为 `true` 。不过你如果这样设置的话，投影又不显示了，因为他们都被遮盖了！

我发现了一种折中的解决方案：创建两个图层。外围的图层是一个纯色的 `CALayer` 用来展示投影，里面的图层则是有圆角的图片图层：


    var sublayer = CALayer()
    sublayer.backgroundColor = UIColor.blueColor().CGColor
    sublayer.shadowOffset = CGSizeMake(0, 3)
    sublayer.shadowRadius = 5.0
    sublayer.shadowColor = UIColor.blackColor().CGColor
    sublayer.shadowOpacity = 0.8
    sublayer.frame = CGRectMake(30, 30, 128, 192)
    sublayer.borderColor = UIColor.blackColor().CGColor
    sublayer.borderWidth = 2.0
    sublayer.cornerRadius = 10.0
    self.view.layer.addSublayer(sublayer)
    
    var imageLayer = CALayer()
    imageLayer.frame = sublayer.bounds
    imageLayer.cornerRadius = 10.0
    imageLayer.contents = UIImage(named:"BattleMapSplashScreen.jpg")?.CGImage
    imageLayer.masksToBounds = true
    sublayer.addSublayer(imageLayer)

运行的效果是这样的，既有投影，又有图片：

![](http://cdn3.raywenderlich.com/wp-content/uploads/2010/12/CALayer-with-Image-Content-and-Corner-Radius.jpg)


## 关于自定义填充内容

如果你希望通过 `Core Graphics` 画一些自定义的内容而不是用现成的图片的话，这也很简单。

解决的方案是给这个图层设置一个委托，需要实现一个 `drawLayer:inContext` 的方法。这样可以把 `Core Graphics` 绘制的内容显示出来。

让我们试着加一个新的图层，并且在上面绘制一些纹理式样。你可以把 `ViewController` 设置为图层的委托。至于怎么使用 `Core Graphs` ，可以参考[Core Graphics 101: Patterns](http://www.raywenderlich.com/?p=2167)这篇文章。


在方法的最后加上下面这段代码：

    var customDrawn = CALayer()
    customDrawn.delegate = self
    customDrawn.backgroundColor = UIColor.greenColor().CGColor
    customDrawn.frame = CGRectMake(30, 250, 128, 40)
    customDrawn.shadowOffset = CGSizeMake(0, 3)
    customDrawn.shadowRadius = 5.0
    customDrawn.shadowColor = UIColor.blackColor().CGColor
    customDrawn.shadowOpacity = 0.8
    customDrawn.cornerRadius = 10.0
    customDrawn.borderColor = UIColor.blackColor().CGColor
    customDrawn.borderWidth = 2.0
    customDrawn.masksToBounds = true
    self.view.layer.addSublayer(customDrawn)
    customDrawn.setNeedsDisplay()



上面的大部分代码你都是见过的 (比如：创建一个图层，设置一些属性等等) 。有两个地方可能还不太熟悉：

- 首先，它把 self 设置成了这个图层的 delegate 。这意味着 self 需要实现 `drawLayer:inContext` 这个方法来绘制图案。
- 其次，在添加了图层之后，需要通过 `setNeedsDisplay` 方法告诉图层刷新内容 (调用 `drawLayer:inContext` 方法)，如果你忘了调用，内容永远都不会显示。


时间有限，最后一部分自定义纹理的还没有翻译，~~其实是 Swift 水平不够不会写了~~，可以参考原文学习。


***


原文地址 (Objective-C) ：

- [CALayers Tutorial for iOS: Introduction to CALayers](http://www.raywenderlich.com/2502/calayers-tutorial-for-ios-introduction-to-calayers-tutorial)