title: Functional Reactive Programming in Swift - Part 2
date: 2015-05-12 21:22:22
tags: Reactive
categories: 开发笔记
description: 换个思路，换个心情，写一些有趣的代码。
---

## 背景

在这篇文章中我们将会了解到如何利用 FPR 的框架开发一些简单的功能。

在 Github 上搜索 Swift Reactive，可以找到以下结果 (排名不分先后，按照 Star 数目排序)：

- [ReactiveCocoa/ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
- [ReactKit/ReactKit](https://github.com/ReactKit/ReactKit)
- [kzaher/RxSwift](https://github.com/kzaher/RxSwift)
- [yusefnapora/ReactiveSwift](https://github.com/yusefnapora/ReactiveSwift)
- [JJJayway/ReactiveSwift](https://github.com/JJJayway/ReactiveSwift)
- [zhxnlai/ReactiveUI](https://github.com/zhxnlai/ReactiveUI)

按照顺序依次了解一下。

## ReactiveUI

ReactiveUI 是一个轻量级的 target-action 替换方案，可以使用 closure 而不是 selector 来来处理 UI 事件。

比如 UIButton 的点击事件，我们可以这样：

    var btn = UIButton(frame: CGRect(x: 0, y: 0, width: 50, height: 30))
    btn.addAction({ control in
        let c = control as! UIButton
        self.controlSectionDetailLabels[indexPath.row]?.text = "TouchUpInside"},
        forControlEvents: .TouchUpInside)

（吐槽一下，虽然 `addAction:forControlEvents` 从命名上非常可读，但是感觉写起来的时候有些别扭。用尾闭包会好看些：

    btn.addAction(forControlEvents: .TouchUpInside) { control in
        let c = control as! UIButton
        self.controlSectionDetailLabels[indexPath.row]?.text = "TouchUpInside"
    }

实现的思路很简单，其实就是创建另一个对象作为 target ，然后通过这个 target 的 selector 调用 closure 。

看下 `addAction:forControlEvents` 的源码：

    func addAction(action: UIControl -> (), forControlEvents events: UIControlEvents) {
        removeAction(forControlEvents: events)
        
        let proxyTarget = RUIControlProxyTarget(action: action)
        proxyTargets[keyForEvents(events)] = proxyTarget
        addTarget(proxyTarget, action: RUIControlProxyTarget.actionSelector(), forControlEvents: events)
    }

首先创建了 `RUIControlProxyTarget` 对象作为 target ，然后把这个对象存到了 `proxyTargets` 数组里，然后 `addTarget:action:forControlEvents` 方法添加事件。那么问题来了：

这个扩展里是怎么添加了 `proxyTargets` 这个新的数组呢？其实这并不是一个新的变量，而是一个计算量：

    var proxyTargets: RUIControlProxyTargets {
        get {
            if let targets = objc_getAssociatedObject(self, &RUIProxyTargetsKey) as? RUIControlProxyTargets {
                return targets
            } else {
                return setProxyTargets(RUIControlProxyTargets())
            }
        }
        set {
            setProxyTargets(newValue)
        }
    }

    private func setProxyTargets(newValue: RUIControlProxyTargets) -> RUIControlProxyTargets {
        objc_setAssociatedObject(self, &RUIProxyTargetsKey, newValue, UInt(OBJC_ASSOCIATION_RETAIN_NONATOMIC));
        return newValue
    }

可以看到是通过 `objc_setAssociatedObject` 添加数据，通过 `objc_getAssociatedObject` 获取数据。然后在外面封装个 `proxyTargets` 这个计算量，对外隐藏内部的实现过程。

再回头看下这个项目，其实 `ReactiveUI` 并不能算是 FRP 框架，只是把 `UIKit` 的事件用 `closure` 封装了起来。


## ReactiveSwift

ReactiveSwift 是一个很有意思的库。











- [ReactiveCocoa/ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
- [ReactKit/ReactKit](https://github.com/ReactKit/ReactKit)
- [kzaher/RxSwift](https://github.com/kzaher/RxSwift)
- [yusefnapora/ReactiveSwift](https://github.com/yusefnapora/ReactiveSwift)
- [JJJayway/ReactiveSwift](https://github.com/JJJayway/ReactiveSwift)
- [zhxnlai/ReactiveUI](https://github.com/zhxnlai/ReactiveUI)




## MVVM



## 总结

- FRP 的好处
1. 代码简洁，开发快速
函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。
Paul Graham在《黑客与画家》一书中写道：同样功能的程序，极端情况下，Lisp代码的长度可能是C代码的二十分之一。
如果程序员每天所写的代码行数基本相同，这就意味着，”C语言需要一年时间完成开发某个功能，Lisp语言只需要不到三星期。反过来说，如果某个新功能，Lisp语言完成开发需要三个月，C语言需要写五年。”当然，这样的对比故意夸大了差异，但是”在一个高度竞争的市场中，即使开发速度只相差两三倍，也足以使得你永远处在落后的位置。”
2. 接近自然语言，易于理解
函数式编程的自由度很高，可以写出很接近自然语言的代码。
前文曾经将表达式(1 + 2) * 3 - 4，写成函数式语言：
　　subtract(multiply(add(1,2), 3), 4)
对它进行变形，不难得到另一种写法：
　　add(1,2).multiply(3).subtract(4)
这基本就是自然语言的表达了。再看下面的代码，大家应该一眼就能明白它的意思吧：
　　merge([1,2],[3,4]).sort().search(“2”)
因此，函数式编程的代码更容易理解。
3. 更方便的代码管理
函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。因此，每一个函数都可以被看做独立单元，很有利于进行单元测试（unit testing）和除错（debugging），以及模块化组合。
4. 易于”并发编程”
函数式编程不需要考虑”死锁”（deadlock），因为它不修改变量，所以根本不存在”锁”线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署”并发编程”（concurrency）。
请看下面的代码：
　　var s1 = Op1();
　　var s2 = Op2();
　　var s3 = concat(s1, s2);
由于s1和s2互不干扰，不会修改变量，谁先执行是无所谓的，所以可以放心地增加线程，把它们分配在两个线程上完成。其他类型的语言就做不到这一点，因为s1可能会修改系统状态，而s2可能会用到这些状态，所以必须保证s2在s1之后运行，自然也就不能部署到其他线程上了。
多核CPU是将来的潮流，所以函数式编程的这个特性非常重要。




The big idea behind FRP in general and ReactiveCocoa in particular is that you can write your programs as a declarative set of transformations of values over time, rather than keeping track of tons of transient state in instance variables and so on. You process inputs (taps, keyboard inputs, network events, etc) as "signals", whose values can be transformed using functions such as filter and map, and combined to produce new values as output. All the logic for translating the many sources of input in your app into some action can be contained in a single set of transformations along the signal chain. It's cool.




***
参考文献：

- ReactiveCocoa
    - [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
    - [Reactive​Cocoa](http://nshipster.cn/reactivecocoa/)
    - [ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)
    - [ReactiveCocoa Tutorial – The Definitive Introduction: Part 2/2](http://www.raywenderlich.com/62796/reactivecocoa-tutorial-pt2)
    - [MVVM Tutorial with ReactiveCocoa: Part 1/2](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)
    - [MVVM Tutorial with ReactiveCocoa: Part 2/2](http://www.raywenderlich.com/74131/mvvm-tutorial-with-reactivecocoa-part-2)
    - [ReactiveCocoa](http://www.alexcurylo.com/blog/2013/03/27/reactivecocoa/)
    - [MVVM, Swift and ReactiveCocoa - It's good!](http://blog.scottlogic.com/2014/07/24/mvvm-reactivecocoa-swift.html)

- 中文博客
    - [Reactive Cocoa Tutorial [0] = Overview](http://blog.sunnyxx.com/2014/03/06/rac_0_overview/)
    - [Reactive Cocoa Tutorial [1] = 神奇的Macros](http://blog.sunnyxx.com/2014/03/06/rac_1_macros/)
    - [Reactive Cocoa Tutorial [2] = 百变RACStream](http://blog.sunnyxx.com/2014/03/06/rac_2_racstream/)
    - [Reactive Cocoa Tutorial [3] = RACSignal的巧克力工厂](http://blog.sunnyxx.com/2014/03/06/rac_3_racsignal/)
    - [Reactive Cocoa Tutorial [4] = 只取所需的Filters](http://blog.sunnyxx.com/2014/04/19/rac_4_filters/)

- 其他
    - [What is the Sink protocol in swift?](http://stackoverflow.com/questions/24164933/what-is-the-sink-protocol-in-swift)
    - [Archive for the ‘Swift’ Category](http://ericasadun.com/category/swift/page/4/)