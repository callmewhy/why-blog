title: RxSwift 入坑手册
date: 2015-09-21 09:53:10
tags: Lastone
categories: 开发笔记
description: 万事万物皆是流。
---


[RxSwift](https://github.com/ReactiveX/RxSwift) 是我在 Github 上关注已久的一个项目，今天花点时间过了一下它的示例代码，感觉很有意思。

我主要是通过项目里的 Rx.playground 进行学习和了解的，这种方式确实便捷高效。只需要把文档用 `/*: */` 注释即可，直接用 Markdown 编写，简单方便。不过 Xcode7 中这种方式现在还不是很稳定，有个最大的问题就是阅读到中间然后切到其他文件再切回来的时候，阅读的进度条是从头开始的，并不能记录上次阅读的位置。心累。

下面是我的简单笔记，不会面面俱到，只是把学习过程中的收获记录下来，大部分内容来自于项目内的 Rx.playground 。各位如果感兴趣，建议 clone 官方项目，跑个操场玩玩。

Run the playground!

## SupportCode

在进入正题之前，先看下项目里的 `SupportCode.swift` ，主要为 playground 提供了两个便利函数。

一个是 `example` 函数，专门用来写示例代码的，统一输出 log 便于标记浏览，同时还能保持变量不污染全局：

    {% codeblock lang:swift %}
    public func example(description: String, action: () -> ()) {
        print("\n--- \(description) example ---")
        action()
    }
    {% endcodeblock %}

另一个是 `delay` 函数，通过 `dispatch_after` 用来演示延时的：

    {% codeblock lang:swift %}
    public func delay(delay:Double, closure:()->()) {
        dispatch_after(
            dispatch_time(
                DISPATCH_TIME_NOW,
                Int64(delay * Double(NSEC_PER_SEC))
            ),
            dispatch_get_main_queue(), closure)
    }
    {% endcodeblock %}

## Introduction

主要介绍了 Rx 的基础： `Observable` 。

### empty
 `empty` 是一个空的序列，它只发送 `.Completed` 消息。

    {% codeblock lang:swift %}
    example("empty") {
        let emptySequence: Observable<Int> = empty()
        
        let subscription = emptySequence
            .subscribe { event in
                print(event)
            }
    }
    {% endcodeblock %}


- `just` 是只包含一个元素的序列，它会先发送 `.Next(value)` 然后发送 `.Completed` 。

- `sequenceOf` 可以把一系列元素转换成事件序列。

- `form` 是通过 `asObservable()` 方法把 Swift 中的序列转换成事件序列。 

- `create` 可以通过闭包创建序列，通过 `.on(e: Event)` 添加事件。

- `failWith` 创建一个没有元素的序列，只会发送失败 (`.Error`) 事件。

- `deferred` 会等到有订阅者的时候再创建 `Observable` ，而且每个订阅者订阅的对象都是内容相同而独立的。



参考文献：

- [ReactiveX](http://reactivex.io/intro.html)
- [Defer](http://reactivex.io/documentation/operators/defer.html)
- [jQuery 的 deferred 对象详解](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object.html)
- [script 的 defer 和 async](http://ued.ctrip.com/blog/?p=3121)