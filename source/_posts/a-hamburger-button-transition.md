title: 如何用Swift做一个不错的按钮变换动画
tags: Swift
date: 2014-10-12 18:02:33
categories: 翻译笔记
description: 酷炫的动画效果，带你装逼带你飞！
---


汉堡按钮在 UI 设计中早已不是什么新鲜玩意儿了，但是某天我突然在 dribbble 上看到了这个[酷炫的动画效果](https://dribbble.com/shots/1623679-Open-Close)，效果真是棒棒哒！于是我决定把它在代码里实现一下。


先来看下 [CreativeDash](https://dribbble.com/Creativedash)  团队做出来的原始动画效果：

![](https://d13yacurqjgara.cloudfront.net/users/107759/screenshots/1623679/menu.gif)

我们可以看到 (看得我眼睛都花了)，汉堡按钮 (也就是三条横线的那个)的上下两条线分别绕着自身最优的端点旋转了45°，变成了按钮里的 X ，中间的那个线则摇身一变，成了外面的圈圈。这个效果可以用 `CAShapeLayer` 实现，但是首先，我需要为这三条直线分别创建一个 `CGPath` 。

我们可以这样手动创建一个短短的直线：

    let shortStroke: CGPath = {
        let path = CGPathCreateMutable()
        CGPathMoveToPoint(path, nil, 2, 2)
        CGPathAddLineToPoint(path, nil, 28, 2)

        return path
    }()

不过对于中间那个线，因为它有比较复杂的运动轨迹，所以我用 [Sketch](http://www.bohemiancoding.com/sketch/) 创建了一条路径，把动画融入了线条中：

![](http://robb.is/img/outline.png)

接下来我把图片导出成了 SVG 格式，然后通过 [PaintCode](http://www.paintcodeapp.com/) 1 把图片转换成代码片段，导入项目中。接下来我重写了导入的代码片段，并且创建了想要的 CGPath 对象：

    let outline: CGPath = {
        let path = CGPathCreateMutable()
        CGPathMoveToPoint(path, nil, 10, 27)
        CGPathAddCurveToPoint(path, nil, 12.00, 27.00, 28.02, 27.00, 40, 27)
        CGPathAddCurveToPoint(path, nil, 55.92, 27.00, 50.47,  2.00, 27,  2)
        CGPathAddCurveToPoint(path, nil, 13.16,  2.00,  2.00, 13.16,  2, 27)
        CGPathAddCurveToPoint(path, nil,  2.00, 40.84, 13.16, 52.00, 27, 52)
        CGPathAddCurveToPoint(path, nil, 40.84, 52.00, 52.00, 40.84, 52, 27)
        CGPathAddCurveToPoint(path, nil, 52.00, 13.16, 42.39,  2.00, 27,  2)
        CGPathAddCurveToPoint(path, nil, 13.16,  2.00,  2.00, 13.16,  2, 27)

        return path
    }()


可能有第三方的类库可以让你直接把 SVG 转换成 CGPath，但是对于这种简单的图像，没必要杀鸡用牛刀。

在我自定义的 `UIButton` 的子类里，我添加了三个 `CAShapeLayer` 属性并且把他们和前面定义的路径关联了起来：

    self.top.path = shortStroke
    self.middle.path = outline
    self.bottom.path = shortStroke

然后设置一下相关的属性：

    layer.fillColor = nil
    layer.strokeColor = UIColor.whiteColor().CGColor
    layer.lineWidth = 4
    layer.miterLimit = 4
    layer.lineCap = kCALineCapRound
    layer.masksToBounds = true


为了正确的计算出边框 (`bounds`) ，我需要对这些线条的大小进行计算。谢天谢地，用 `CGPathCreateCopyByStrokingPath` 这个函数可以创建一条和原来线条一样的路径，所以它的 `bounds` 和参数里那个 `CAShapeLayer` 类型的内容完全一致：

    let boundingPath = CGPathCreateCopyByStrokingPath(layer.path, nil, 4, kCGLineCapRound, kCGLineJoinMiter, 4)

    layer.bounds = CGPathGetPathBoundingBox(boundingPath)


因为汉堡按钮的上下两个线条接下来会绕着最右点旋转，所以在布局图层的时候，我们需要相应的设置它的锚点 (`anchorPoint`)：

    self.top.anchorPoint = CGPointMake(28.0 / 30.0, 0.5)
    self.top.position = CGPointMake(40, 18)

    self.middle.position = CGPointMake(27, 27)
    self.middle.strokeStart = hamburgerStrokeStart
    self.middle.strokeEnd = hamburgerStrokeEnd

    self.bottom.anchorPoint = CGPointMake(28.0 / 30.0, 0.5)
    self.bottom.position = CGPointMake(40, 36)


接下来就是动画的部分了。当按钮的状态改变的时候，三条直线应该会动态的移动到新的位置。对于那两个外围的直线而言，实现起来很容易。比如最上面的那条线，我先把它往里移动 4 points 让它位于中心位置，然后再逆时针旋转45度，形成了半个 X ：

    var transform = CATransform3DIdentity
    transform = CATransform3DTranslate(transform, -4, 0, 0)
    transform = CATransform3DRotate(transform, -M_PI_4, 0, 0, 1)

    let animation = CABasicAnimation(keyPath: "transform")
    animation.fromValue = NSValue(CATransform3D: CATransform3DIdentity)
    animation.toValue = NSValue(CATransform3D: transform)
    animation.timingFunction = CAMediaTimingFunction(controlPoints: 0.25, -0.8, 0.75, 1.85)
    animation.beginTime = CACurrentMediaTime() + 0.25

    self.top.addAnimation(animation, forKey: "transform")
    self.top.transform = transform


最下面的那条线的部分交给读者朋友们作为练习。

接下来，中间的那条直线似乎有点棘手。为了实现理想的效果，我们需要给 `CAShapeLayer` 的起点和终点分别设置动画效果。

首先我需要知道这两个属性在两种状态下的值分别是多少。注意，就算是在汉堡状态下，直线也没有从 O 开始。路径稍微的从左边延伸，我们可以稍候应用 `timing` 函数实现一个酷炫的效果：

    let menuStrokeStart: CGFloat = 0.325
    let menuStrokeEnd: CGFloat = 0.9

    let hamburgerStrokeStart: CGFloat = 0.028
    let hamburgerStrokeEnd: CGFloat = 0.111


接下来我们只需要创建动画并加到图层里即可：

    let strokeStart = CABasicAnimation(keyPath: "strokeStart")
    strokeStart.fromValue = hamburgerStrokeStart
    strokeStart.toValue = menuStrokeStart
    strokeStart.duration = 0.5
    strokeStart.timingFunction = CAMediaTimingFunction(controlPoints: 0.25, -0.4, 0.5, 1)

    self.middle.addAnimation(strokeStart, forKey: "strokeStart")
    self.middle.strokeStart = menuStrokeStart

    let strokeEnd = CABasicAnimation(keyPath: "strokeEnd")
    strokeEnd.fromValue = hamburgerStrokeEnd
    strokeEnd.toValue = menuStrokeEnd
    strokeEnd.duration = 0.6
    strokeEnd.timingFunction = CAMediaTimingFunction(controlPoints: 0.25, -0.4, 0.5, 1)

    self.middle.addAnimation(strokeEnd, forKey: "strokeEnd")
    self.middle.strokeEnd = menuStrokeEnd



把前面的工作整合一下，你可以看到如下的结果：

![](http://robb.is/img/hamburger-button.gif)


哈哈哈哈成功实现了，洒家真是太高兴了。你可以在[这里](https://github.com/robb/hamburger-button)下载源码。当然你也可以加我的[小鸟](https://twitter.com/ceterum_censeo)。玩的开心~

***

原文地址 

- [How to build a nice Hamburger Button transition in Swift](http://robb.is/working-on/a-hamburger-button-transition/)