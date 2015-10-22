title: RxSwift 入坑手册 Part0 - 基础概念
date: 2015-09-21 09:53:10
tags: RxSwift
categories: 开发笔记
description: 滚滚长江都是水，万事万物皆是流。
---

（前面的坑还没填完，我又来开坑了。。。Part 0 走起！）

[RxSwift](https://github.com/ReactiveX/RxSwift) 是我在 Github 上关注已久的一个项目，今天花点时间过了一下它的示例代码，感觉很有意思。

我主要是通过项目里的 [Rx.playground](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground) 进行学习和了解的，这种方式确实便捷高效。只需要把文档用 `/*: */` 注释即可，直接用 Markdown 编写，简单方便。不过 Xcode7 中这种方式现在还不是很稳定，会有大量的空行，而且有个最大的问题就是阅读到中间然后切到其他文件再切回来的时候，阅读的进度条是从头开始的，并不能记录上次阅读的位置。心累。

下面是我的简单笔记，只是把学习过程中的收获记录下来，大部分内容来自于项目内的 playground 。注意！是**很大部分**！而且操场里图文并茂，很容易理解。所以，各位如果感兴趣，建议 clone 官方项目，跑个操场玩玩。

参考文献中罗列了我在学习过程中查阅的相关资料，可以作为补充阅读。

## SupportCode

在进入正题之前，先看下项目里的 `SupportCode.swift` ，主要为 playground 提供了两个便利函数。

一个是 `example` 函数，专门用来写示例代码的，统一输出 log 便于标记浏览，同时还能保持变量不污染全局：

    public func example(description: String, action: () -> ()) {
        print("\n--- \(description) example ---")
        action()
    }

另一个是 `delay` 函数，通过 `dispatch_after` 用来演示延时的：

    public func delay(delay:Double, closure:()->()) {
        dispatch_after(
            dispatch_time(
                DISPATCH_TIME_NOW,
                Int64(delay * Double(NSEC_PER_SEC))
            ),
            dispatch_get_main_queue(), closure)
    }

## Introduction

主要介绍了 Rx 的基础： `Observable` 。 `Observable<Element>` 是观察者模式中被观察的对象，相当于一个事件序列 (`GeneratorType`) ，会向订阅者发送新产生的事件信息。事件信息分为三种： 

- `.Next(value)` 表示新的事件数据。
- `.Completed` 表示事件序列的完结。
- `.Error` 同样表示完结，但是代表异常导致的完结。

（打个岔：协议命名，想起来上午汤哥在[微博](http://weibo.com/1747002695/CBH3I4a1X?from=page_1005051747002695_profile&wvr=6&mod=weibotime&type=comment)说的一段话：

> 另外，我觉得 protocol 名字用形容词会更加语义分明，比如 Swift : Flyable, Killable, Visible。全用名词的话显得比较生硬，比如 Swift : Head, Wings, Ass。


### empty

`empty` 是一个空的序列，它只发送 `.Completed` 消息。

    example("empty") {
        let emptySequence: Observable<Int> = empty()
        
        let subscription = emptySequence
            .subscribe { event in
                print(event)
            }
    }

    --- empty example ---
    Completed


### never

`never` 是没有任何元素、也不会发送任何事件的空序列。

    example("never") {
        let neverSequence: Observable<String> = never()

        let subscription = neverSequence
            .subscribe { _ in
                print("This block is never called.")
            }
    }

    --- never example ---


### just

`just` 是只包含一个元素的序列，它会先发送 `.Next(value)` ，然后发送 `.Completed` 。

    example("just") {
        let singleElementSequence = just(32)

        let subscription = singleElementSequence
            .subscribe { event in
                print(event)
            }
    }

    --- just example ---
    Next(32)
    Completed

### sequenceOf

`sequenceOf` 可以把一系列元素转换成事件序列。

    example("sequenceOf") {
        let sequenceOfElements/* : Observable<Int> */ = sequenceOf(0, 1, 2, 3)
        
        let subscription = sequenceOfElements
            .subscribe { event in
                print(event)
            }
    }

    --- sequenceOf example ---
    Next(0)
    Next(1)
    Next(2)
    Next(3)
    Completed

### form

`form` 是通过 `asObservable()` 方法把 Swift 中的序列 (`SequenceType`) 转换成事件序列。 

    example("from") {
        let sequenceFromArray = [1, 2, 3, 4, 5].asObservable()

        let subscription = sequenceFromArray
            .subscribe { event in
                print(event)
            }
    }

    --- from example ---
    Next(1)
    Next(2)
    Next(3)
    Next(4)
    Next(5)
    Completed

### create

`create` 可以通过闭包创建序列，通过 `.on(e: Event)` 添加事件。

    example("create") {
        let myJust = { (singleElement: Int) -> Observable<Int> in
            return create { observer in
                observer.on(.Next(singleElement))
                observer.on(.Completed)
                
                return NopDisposable.instance
            }
        }
        
        let subscription = myJust(5)
            .subscribe { event in
                print(event)
            }
    }

    --- create example ---
    Next(5)
    Completed

### failWith

`failWith` 创建一个没有元素的序列，只会发送失败 (`.Error`) 事件。

    example("failWith") {
        let error = NSError(domain: "Test", code: -1, userInfo: nil)
        
        let erroredSequence: Observable<Int> = failWith(error)
        
        let subscription = erroredSequence
            .subscribe { event in
                print(event)
            }
    }

    --- failWith example ---
    Error(Error Domain=Test Code=-1 "The operation couldn’t be completed. (Test error -1.)")


### deferred

`deferred` 会等到有订阅者的时候再通过工厂方法创建 `Observable` 对象，每个订阅者订阅的对象都是内容相同而完全独立的序列。

    example("deferred") {
        let deferredSequence: Observable<Int> = deferred {
            print("creating")
            return create { observer in
                print("emmiting")
                observer.on(.Next(0))
                observer.on(.Next(1))
                observer.on(.Next(2))

                return NopDisposable.instance
            }
        }

        print("go")

        deferredSequence
            .subscribe { event in
                print(event)
        }

        deferredSequence
            .subscribe { event in
                print(event)
            }
    }

    --- deferred example ---
    go
    creating
    emmiting
    Next(0)
    Next(1)
    Next(2)
    creating
    emmiting
    Next(0)
    Next(1)
    Next(2)

为什么需要 `defferd` 这样一个奇怪的家伙呢？其实这相当于是一种延时加载，因为在添加监听的时候数据未必加载完毕，例如下面这个例子：

    example("TestDeferred") {
        var value: String? = nil
        var subscription: Observable<String?> = just(value)

        // got value
        value = "Hello!"

        subscription.subscribe { event in
            print(event)
        }
    }

    --- TestDeferred example ---
    Next(nil)
    Completed

如果使用 `deffered` 则可以正常显示想要的数据：

    example("TestDeferred") {
        var value: String? = nil
        var subscription: Observable<String?> = deferred {
            return just(value)
        }

        // got value
        value = "Hello!"

        subscription.subscribe { event in
            print(event)
        }
        
    }

    --- TestDeferred example ---
    Next(Optional("Hello!"))
    Completed


## Subjects

接下来是关于 `Subject` 的内容。 `Subject` 可以看做是一种代理和桥梁。它既是订阅者又是订阅源，这意味着它既可以订阅其他 `Observable` 对象，同时又可以对它的订阅者们发送事件。

如果把 `Observable` 理解成不断输出事件的水管，那 `Subject` 就是套在上面的水龙头。它既怼着一根不断出水的水管，同时也向外面输送着新鲜水源。如果你直接用水杯接着水管的水，那可能导出来什么王水胶水完全把持不住；如果你在水龙头下面接着水，那你可以随心所欲的调成你想要的水速和水温。

（好吧上面一段文档里没有，是我瞎掰的，如果理解错了还望打脸(￣ε(#￣)☆╰╮(￣▽￣///))

在开始下面的代码之前，先定义一个辅助函数用于输出数据： 

    func writeSequenceToConsole<O: ObservableType>(name: String, sequence: O) {
        sequence
            .subscribe { e in
                print("Subscription: \(name), event: \(e)")
            }
    }

### PublishSubject

`PublishSubject` 会发送订阅者从订阅之后的事件序列。

    example("PublishSubject") {
        let subject = PublishSubject<String>()
        writeSequenceToConsole("1", sequence: subject)
        subject.on(.Next("a"))
        subject.on(.Next("b"))
        writeSequenceToConsole("2", sequence: subject)
        subject.on(.Next("c"))
        subject.on(.Next("d"))
    }


    --- PublishSubject example ---
    Subscription: 1, event: Next(a)
    Subscription: 1, event: Next(b)
    Subscription: 1, event: Next(c)
    Subscription: 2, event: Next(c)
    Subscription: 1, event: Next(d)
    Subscription: 2, event: Next(d)


### ReplaySubject

`ReplaySubject` 在新的订阅对象订阅的时候会补发所有已经发送过的数据队列， `bufferSize` 是缓冲区的大小，决定了补发队列的最大值。如果 `bufferSize` 是1，那么新的订阅者出现的时候就会补发上一个事件，如果是2，则补两个，以此类推。

    example("ReplaySubject") {
        let subject = ReplaySubject<String>.create(bufferSize: 1)

        writeSequenceToConsole("1", sequence: subject)
        subject.on(.Next("a"))
        subject.on(.Next("b"))
        writeSequenceToConsole("2", sequence: subject)
        subject.on(.Next("c"))
        subject.on(.Next("d"))
    }

    --- ReplaySubject example ---
    Subscription: 1, event: Next(a)
    Subscription: 1, event: Next(b)
    Subscription: 2, event: Next(b) // 补了一个 b
    Subscription: 1, event: Next(c)
    Subscription: 2, event: Next(c)
    Subscription: 1, event: Next(d)
    Subscription: 2, event: Next(d)

### BehaviorSubject

`BehaviorSubject` 在新的订阅对象订阅的时候会发送最近发送的事件，如果没有则发送一个默认值。

    example("BehaviorSubject") {
        let subject = BehaviorSubject(value: "z")
        writeSequenceToConsole("1", sequence: subject)
        subject.on(.Next("a"))
        subject.on(.Next("b"))
        writeSequenceToConsole("2", sequence: subject)
        subject.on(.Next("c"))
        subject.on(.Completed)
    }

    --- BehaviorSubject example ---
    Subscription: 1, event: Next(z)
    Subscription: 1, event: Next(a)
    Subscription: 1, event: Next(b)
    Subscription: 2, event: Next(b)
    Subscription: 1, event: Next(c)
    Subscription: 2, event: Next(c)
    Subscription: 1, event: Completed
    Subscription: 2, event: Completed

### Variable

`Variable` 是基于 `BehaviorSubject` 的一层封装，它的优势是：不会被显式终结。即：不会收到 `.Completed` 和 `.Error` 这类的终结事件，它会主动在析构的时候发送 `.Complete` 。

    example("Variable") {
        let variable = Variable("z")
        writeSequenceToConsole("1", sequence: variable)
        variable.value = "a"
        variable.value = "b
        writeSequenceToConsole("2", sequence: variable)
        variable.value = "c"
    }

    --- Variable example ---
    Subscription: 1, event: Next(z)
    Subscription: 1, event: Next(a)
    Subscription: 1, event: Next(b)
    Subscription: 2, event: Next(b)
    Subscription: 1, event: Next(c)
    Subscription: 2, event: Next(c)
    Subscription: 1, event: Completed
    Subscription: 2, event: Completed


## Transform

我们可以对序列做一些转换，类似于 Swift 中 `CollectionType` 的各种转换。在以前的坑中曾经提到过，可以参考：[函数式的函数](http://blog.callmewhy.com/2015/05/11/functional-reactive-programming-1/#函数式的函数/)。

### map

`map` 就是对每个元素都用函数做一次转换，挨个映射一遍。

    example("map") {
        let originalSequence = sequenceOf(1,2,3)

        originalSequence
            .map { $0 * 2 }
            .subscribe { print($0) }
    }

    --- map example ---
    Next(2)
    Next(4)
    Next(6)
    Completed

### flatMap

`map` 在做转换的时候很容易出现『升维』的情况，即：转变之后，从一个序列变成了一个序列的序列。

什么是『升维』？在集合中我们可以举这样一个例子，我有一个好友列表 `[p1, p2, p3]`，那么如果要获取我好友的好友的列表，可以这样做：

    myFriends.map { $0.getFriends() }

结果就成了 `[[p1-1, p1-2, p1-3], [p2-1], [p3-1, p3-2]]` ，这就成了好友的好友列表的列表了。这就是一个『升维』的例子。

（以上内容文档中依旧没有，依旧是我瞎掰的，依旧欢迎有错误当面打脸(￣ε(#￣)☆╰╮(￣▽￣///))

在 Swift 中，我们可以用 `flatMap` 过滤掉 `map` 之后的 `nil` 结果。在 Rx 中， `flatMap` 可以把一个序列转换成一组序列，然后再把这一组序列『拍扁』成一个序列。

    example("flatMap") {
        let sequenceInt = sequenceOf(1, 2, 3)
        let sequenceString = sequenceOf("A", "B", "--")

        sequenceInt
            .flatMap { int in
                sequenceString
            }
            .subscribe {
                print($0)
            }
    }

    --- flatMap example ---
    Next(A)
    Next(B)
    Next(--)
    Next(A)
    Next(B)
    Next(--)
    Next(A)
    Next(B)
    Next(--)
    Completed

### scan

`scan` 有点像 `reduce` ，它会把每次的运算结果累积起来，作为下一次运算的输入值。

    example("scan") {
        let sequenceToSum = sequenceOf(0, 1, 2, 3, 4, 5)

        sequenceToSum
            .scan(0) { acum, elem in
                acum + elem
            }
            .subscribe {
                print($0)
            }
    }

    --- scan example ---
    Next(0)
    Next(1)
    Next(3)
    Next(6)
    Next(10)
    Next(15)
    Completed

## Filtering

除了上面的各种转换，我们还可以对序列进行过滤。

### filter

`filter` 只会让符合条件的元素通过。

    example("filter") {
        let subscription = sequenceOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
            .filter {
                $0 % 2 == 0
            }
            .subscribe {
                print($0)
            }
    }

    --- filter example ---
    Next(0)
    Next(2)
    Next(4)
    Next(6)
    Next(8)
    Completed


### distinctUntilChanged

`distinctUntilChanged` 会废弃掉重复的事件。

    example("distinctUntilChanged") {
        let subscription = sequenceOf(1, 2, 3, 1, 1, 4)
            .distinctUntilChanged()
            .subscribe {
                print($0)
            }
    }

    --- distinctUntilChanged example ---
    Next(1)
    Next(2)
    Next(3)
    Next(1)
    Next(4)
    Completed


### take

`take` 只获取序列中的前 n 个事件，在满足数量之后会自动 `.Completed` 。

    example("take") {
        let subscription = sequenceOf(1, 2, 3, 4, 5, 6)
            .take(3)
            .subscribe {
                print($0)
            }
    }

    --- take example ---
    Next(1)
    Next(2)
    Next(3)
    Completed


## Combining

这部分是关于序列的运算，可以将多个序列源进行组合拼装成一个新的事件序列。

### startWith

`startWith` 会在队列开始之前插入一个事件元素。

    example("startWith") {
        let subscription = sequenceOf(4, 5, 6)
            .startWith(3)
            .subscribe {
                print($0)
            }
    }

    --- startWith example ---
    Next(3)
    Next(4)
    Next(5)
    Next(6)
    Completed


### combineLatest

如果存在两条事件队列，需要同时监听，那么每当有新的事件发生的时候，`combineLatest` 会将每个队列的最新的一个元素进行合并。

    example("combineLatest 1") {
        let intOb1 = PublishSubject<String>()
        let intOb2 = PublishSubject<Int>()

        combineLatest(intOb1, intOb2) {
            "\($0) \($1)"
            }
            .subscribe {
                print($0)
            }

        intOb1.on(.Next("A"))
        intOb2.on(.Next(1))
        intOb1.on(.Next("B"))
        intOb2.on(.Next(2))
    }

    --- combineLatest 1 example ---
    Next(A 1)
    Next(B 1)
    Next(B 2)


### zip

`zip` 人如其名，就是合并两条队列用的，不过它会等到两个队列的元素一一对应地凑齐了之后再合并，正如[百折不撓的米斯特菜](http://weibo.com/mrgreenhand)所提醒的， `zip` 就像是拉链一样，两根拉链拉着拉着合并到了一根上：

    example("zip 1") {
        let intOb1 = PublishSubject<String>()
        let intOb2 = PublishSubject<Int>()
        zip(intOb1, intOb2) {
            "\($0) \($1)"
            }
            .subscribe {
                print($0)
            }
        intOb1.on(.Next("A"))
        intOb2.on(.Next(1))
        intOb1.on(.Next("B"))
        intOb1.on(.Next("C"))
        intOb2.on(.Next(2))
    }

    --- zip 1 example ---
    Next(A 1)
    Next(B 2)

### marge

`merge` 就是 merge 啦，把两个队列按照顺序组合在一起。

    example("merge 1") {
        let subject1 = PublishSubject<Int>()
        let subject2 = PublishSubject<Int>()

        sequenceOf(subject1, subject2)
            .merge()
            .subscribeNext { int in
                print(int)
            }

        subject1.on(.Next(1))
        subject1.on(.Next(2))
        subject2.on(.Next(3))
        subject1.on(.Next(4))
        subject2.on(.Next(5))
    }

    --- merge 1 example ---
    1
    2
    3
    4
    5

### switch

当你的事件序列是一个事件序列的序列 (`Observable<Observable<T>>`) 的时候，（可以理解成二维序列？），可以使用 `switch` 将序列的序列平铺成一维，并且在出现新的序列的时候，自动切换到最新的那个序列上。和 `merge` 相似的是，它也是起到了将多个序列『拍平』成一条序列的作用。

    example("switchLatest") {
        let var1 = Variable(0)

        let var2 = Variable(200)

        // var3 is like an Observable<Observable<Int>>
        let var3 = Variable(var1)

        let d = var3
            .switchLatest()
            .subscribe {
                print($0)
            }

        var1.value = 1
        var1.value = 2
        var1.value = 3
        var1.value = 4

        var3.value = var2
        var2.value = 201
        var1.value = 5
        
        var3.value = var1
        var2.value = 202
        var1.value = 6
    }

    --- switchLatest example ---
    Next(0)
    Next(1)
    Next(2)
    Next(3)
    Next(4)
    Next(200)
    Next(201)
    Next(5)
    Next(6)

注意，虽然都是『拍平』，但是和 `flatmap` 是不同的， `flatmap` 是将一条序列变成另一条序列，而这变换过程会让维度变高，所以需要『拍平』，而 `switch` 是将本来二维的序列（序列的序列）拍平成了一维的序列。


## Error Handling

在事件序列中，遇到异常也是很正常的事情，有以下几种处理异常的手段。

### catchError

`catchError` 可以捕获异常事件，并且在后面无缝接上另一段事件序列，丝毫没有异常的痕迹。

    example("catchError 1") {
        let sequenceThatFails = PublishSubject<Int>()
        let recoverySequence = sequenceOf(100, 200)

        sequenceThatFails
            .catchError { error in
                return recoverySequence
            }
            .subscribe {
                print($0)
            }

        sequenceThatFails.on(.Next(1))
        sequenceThatFails.on(.Next(2))
        sequenceThatFails.on(.Error(NSError(domain: "Test", code: 0, userInfo: nil)))
    }

    --- catchError 1 example ---
    Next(1)
    Next(2)
    Next(100)
    Next(200)
    Completed

### retry

`retry` 顾名思义，就是在出现异常的时候会再去从头订阅事件序列，妄图通过『从头再来』解决异常。

    example("retry") {
        var count = 1 // bad practice, only for example purposes
        let funnyLookingSequence: Observable<Int> = create { observer in
            let error = NSError(domain: "Test", code: 0, userInfo: nil)
            observer.on(.Next(0))
            observer.on(.Next(1))
            if count < 2 {
                observer.on(.Error(error))
                count++
            }
            observer.on(.Next(2))
            observer.on(.Completed)

            return NopDisposable.instance
        }

        funnyLookingSequence
            .retry()
            .subscribe {
                print($0)
            }
    }

    --- retry example ---
    Next(0)
    Next(1)
    Next(0)
    Next(1)
    Next(2)
    Completed

## Utility

这里列举了针对事件序列的一些方法。

### subscribe

`subscribe` 在前面已经接触过了，有新的事件就会触发。

    example("subscribe") {
        let sequenceOfInts = PublishSubject<Int>()

        sequenceOfInts
            .subscribe {
                print($0)
            }

        sequenceOfInts.on(.Next(1))
        sequenceOfInts.on(.Completed)
    }

    --- subscribe example ---
    Next(1)
    Completed

### subscribeNext

`subscribeNext` 也是订阅，但是只订阅 `.Next` 事件。

    example("subscribeNext") {
        let sequenceOfInts = PublishSubject<Int>()

        sequenceOfInts
            .subscribeNext {
                print($0)
            }

        sequenceOfInts.on(.Next(1))
        sequenceOfInts.on(.Completed)
    }

    --- subscribeNext example ---
    1

### subscribeCompleted

`subscribeCompleted` 是只订阅 `.Completed` 完成事件。

    example("subscribeCompleted") {
        let sequenceOfInts = PublishSubject<Int>()

        sequenceOfInts
            .subscribeCompleted {
                print("It's completed")
            }

        sequenceOfInts.on(.Next(1))
        sequenceOfInts.on(.Completed)
    }

    --- subscribeCompleted example ---
    It's completed

### subscribeError

`subscribeError` 只订阅 `.Error` 失败事件。

    example("subscribeError") {
        let sequenceOfInts = PublishSubject<Int>()

        sequenceOfInts
            .subscribeError { error in
                print(error)
            }

        sequenceOfInts.on(.Next(1))
        sequenceOfInts.on(.Error(NSError(domain: "Examples", code: -1, userInfo: nil)))
    }

    --- subscribeError example ---
    Error Domain=Examples Code=-1 "The operation couldn’t be completed. (Examples error -1.)"

### doOn

`doOn` 可以监听事件，并且在事件发生之前调用。

    example("doOn") {
        let sequenceOfInts = PublishSubject<Int>()

        sequenceOfInts
            .doOn {
                print("Intercepted event \($0)")
            }
            .subscribe {
                print($0)
            }

        sequenceOfInts.on(.Next(1))
        sequenceOfInts.on(.Completed)
    }

    --- doOn example ---
    Intercepted event Next(1)
    Next(1)
    Intercepted event Completed
    Completed

## Conditional

我们可以对多个事件序列做一些复杂的逻辑判断。

### takeUntil

`takeUntil` 其实就是 `take` ，它会在终于等到那个事件之后触发 `.Completed` 事件。

    example("takeUntil") {
        let originalSequence = PublishSubject<Int>()
        let whenThisSendsNextWorldStops = PublishSubject<Int>()

        originalSequence
            .takeUntil(whenThisSendsNextWorldStops)
            .subscribe {
                print($0)
            }

        originalSequence.on(.Next(1))
        originalSequence.on(.Next(2))

        whenThisSendsNextWorldStops.on(.Next(1))

        originalSequence.on(.Next(3))
    }

    --- takeUntil example ---
    Next(1)
    Next(2)
    Completed

### takeWhile

`takeWhile` 则是可以通过状态语句判断是否继续 `take` 。

    example("takeWhile") {
        let sequence = PublishSubject<Int>()
        sequence
            .takeWhile { int in
                int < 2
            }
            .subscribe {
                print($0)
            }
        sequence.on(.Next(1))
        sequence.on(.Next(2))
        sequence.on(.Next(3))
    }

    --- takeWhile example ---
    Next(1)
    Completed

## Aggregate

我们可以对事件序列做一些集合运算。

### concat

`concat` 可以把多个事件序列合并起来。 

    example("concat") {
        let var1 = BehaviorSubject(value: 0)
        let var2 = BehaviorSubject(value: 200)
        
        // var3 is like an Observable<Observable<Int>>
        let var3 = BehaviorSubject(value: var1)
        
        let d = var3
            .concat()
            .subscribe {
                print($0)
            }
        
        var1.on(.Next(1))
        var1.on(.Next(2))
        
        var3.on(.Next(var2))
        
        var2.on(.Next(201))
        
        var1.on(.Next(3))
        var1.on(.Completed)
        
        var2.on(.Next(202))
    }

    --- concat example ---
    Next(0)
    Next(1)
    Next(2)
    Next(3)
    Next(201)
    Next(202)

### reduce

这里的 `reduce` 和 `CollectionType` 中的 `reduce` 是一个意思，都是指通过对一系列数据的运算最后生成一个结果。

    example("reduce") {
        sequenceOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
            .reduce(0, +)
            .subscribe {
                print($0)
            }
    }

    --- reduce example ---
    Next(45)
    Completed


## Connectable

坑待填，Xcode 里这个操场跑不起来了。

## Next

基础入门大概就是这些了，有了前面 《[Functional Reactive Programming in Swift - Part 1](http://blog.callmewhy.com/2015/05/11/functional-reactive-programming-1/)》 的铺垫，似乎理解起来十分愉快，不过还是不够深入，在下一章会在具体项目中操练起来。

操练起来！跑个操场吧少年！

Run the playground in your Xcode!


***

参考文献：

- [ReactiveX](http://reactivex.io/intro.html)
- [RAC Marbles](http://neilpa.me/rac-marbles/)
- [defer](http://reactivex.io/documentation/operators/defer.html)
- [jQuery 的 deferred 对象详解](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object.html)
- [jQuery deferred 对象的 promise 方法](http://blog.allenm.me/2012/01/jquery_deferred_promise_method/)
- [Deferring Observable code until subscription in RxJava](http://blog.danlew.net/2015/07/23/deferring-observable-code-until-subscription-in-rxjava/)
- [To Use Subject Or Not To Use Subject?](http://davesexton.com/blog/post/To-Use-Subject-Or-Not-To-Use-Subject.aspx)
- [flatmap](http://reactivex.io/documentation/operators/flatmap.html)
- [Grokking RxJava, Part 2: Operator, Operator](http://blog.danlew.net/2014/09/22/grokking-rxjava-part-2/)
- [Transformation - Select Many](http://www.introtorx.com/Content/v1.0.10621.0/08_Transformation.html#SelectMany)
- [RxJava Observable tranformation: concatMap() vs flatMap()](http://fernandocejas.com/2015/01/11/rxjava-observable-tranformation-concatmap-vs-flatmap/)
- [Recursive Observables with RxJava](http://jschneider.io/2014/11/26/Recursive-Observables-with-RxJava.html)
- [Combining sequences](http://www.introtorx.com/Content/v1.0.10621.0/12_CombiningSequences.html)
- [switch](http://reactivex.io/documentation/operators/switch.html)
- [RxSwift Getting Started](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md)