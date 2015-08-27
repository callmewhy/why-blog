title: Functional Reactive Programming in Swift - Part 2
date: 2015-05-12 21:22:22
tags: Reactive
categories: 开发笔记
description: 换个思路，换个心情，写一些有趣的代码。
---

## 背景

在这篇文章中我们将会了解到如何利用 FPR 的框架开发一些简单的功能。

在 Github 上搜索 Swift Reactive，可以找到以下结果 (排名不分先后，按照 Star 数目排序）。


## [ReactiveUI](https://github.com/zhxnlai/ReactiveUI)

ReactiveUI 是一个轻量级的 target-action 替换方案，可以使用 closure 而不是 selector 来来处理 UI 事件。

比如 UIButton 的点击事件，我们可以这样：

    var btn = UIButton(frame: CGRect(x: 0, y: 0, width: 50, height: 30))
    btn.addAction({ control in
        let c = control as! UIButton
        self.controlSectionDetailLabels[indexPath.row]?.text = "TouchUpInside"},
        forControlEvents: .TouchUpInside)

（吐槽一下，虽然 `addAction:forControlEvents` 从命名上非常可读，但是感觉写起来的时候有些别扭。用 `Trailing Closure` 会好看些：

    btn.addAction(forControlEvents: .TouchUpInside) { control in
        let c = control as! UIButton
        self.controlSectionDetailLabels[indexPath.row]?.text = "TouchUpInside"
    }

实现的方式是：创建另一个对象作为 target ，然后通过这个 target 的 selector 调用 closure 。

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

再回头看下这个项目，其实 `ReactiveUI` 并不能算是一个完整的 FRP 框架，只是把 `UIKit` 中事件用闭包封装了一层。


## [ReactiveSwift](https://github.com/JJJayway/ReactiveSwift)

ReactiveSwift 是一个很有意思的库，它基于数据流做了一些封装，让数据流的处理更加自然。

它主要由以下几个元素组成：

### Event

`Event` 是一个枚举类，用来表示以下四种事件：

- Value: 包含具体数值的事件
- Completed: 成功完成
- Stopped: 提前结束
- Error: 意外结束

后面的三个事件都代表着事件的终结，在这之后再也不会有新的事件了。

### Sinks

`Sink` 是一个负责接收事件的容器，是 Swift 标准库中的 `SinkOf<T>` 结构体。

它定义了一个处理各种类型事件的容器，我们可以提前传入回调函数，然后生成这个容器，在有新的事件传入的时候，会自动调用相应的回调进行处理。


### Terminators

`Terminator` 其实是一个协议，提供了一个 `stop()` 方法，可以发送一个 `.Stopped` 事件。


### Emitters

`Emitter` 是一个函数，输入 `Sink` ，输出 `Terminator` 。 `Emitter` 会不断的向 `Sink` 发送事件 (`Event`) 。可以通过 `emitElements` 方法生成 `Emitter` 对象。

### 具体示例

说来说去的有些晕，看一个官网给的具体的示例：

    let myEmitter = emitElements([1, 3, 5, 7, 9])                   // Emitter: Sink -> Terminator
    let myMap     = map { (v: Int) -> Int in v + 1 }                // Sink -> Sink
    let myFilter  = filter { (v: Int) -> Bool in v < 10 }           // Sink -> Sink
    let mySink    = acceptValue { (v: Int) -> () in println(v) }    // Sink
    
    let result1 = myEmitter .. myMap        // Emitter: Sink -> Terminator
    let result2 = result1 .. myFilter
    let result3 = result2 .. mySink
    println(result3)

一行一行的进行分解：

- 首先生成了一个 `Emitter` ，它会把 [1, 3, 5, 7, 9] 这一串数据作为 `.Value` 事件发送给它的 `Sink` 。
- 后面的 `myMap` 和 `myFilter` 都是针对数据流的转换，称之为 `Intermediate` 。
- `mySink` 定义了一个 `Sink` 用来接受各种事件，在这里我们只处理了 `.Value` 事件：将它打印出来。
- `..` 这个运算符将 `Emitter` 和 `Intermediate` 进行合并，生成了新的 `Emitter` 。
- 最后的 `..` 运算符是将 `Emitter` 和 `Sink` 进行了合并，其实就是调用了 `emitter(sink)` 。


上面的步骤也可以简化成这样：

    emitElements([1, 3, 5, 7, 9])
        .. map { (v: Int) -> Int in v + 1 }
        .. filter { (v: Int) -> Bool in v < 10 }
        .. acceptValue { (v: Int) -> () in println(v) }

看得比较匆忙，更多用法可以参见作者的测试用例 [ReactiveTests.swift](https://github.com/JJJayway/ReactiveSwift/blob/master/ReactiveTests/ReactiveTests.swift) 。

## [RxSwift](https://github.com/kzaher/RxSwift)

RxSwift 是微软 [Reactive Extensions](https://msdn.microsoft.com/en-us/data/gg577609) 的 Swift 版本。


## [ReactKit](https://github.com/ReactKit/ReactKit)

ReactKit 是 Github 上小有名气的反应型框架。


## 小结

简单体验了一下目前比较主流的几个反应型编程框架，感觉很有趣。


***

参考文献：

- [ReactiveCocoa/ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
- [ReactKit/ReactKit](https://github.com/ReactKit/ReactKit)
- [kzaher/RxSwift](https://github.com/kzaher/RxSwift)
- [Interstellar](https://github.com/JensRavens/Interstellar)
- [JJJayway/ReactiveSwift](https://github.com/JJJayway/ReactiveSwift)
- [zhxnlai/ReactiveUI](https://github.com/zhxnlai/ReactiveUI)