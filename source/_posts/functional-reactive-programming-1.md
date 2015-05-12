title: Functional Reactive Programming in Swift - Part 1
date: 2015-05-11 19:53:37
tags: Reactive
categories: 开发笔记
description: 换个思路，换个心情，写一些有趣的代码。
---

## 简介

大四狗在毕业前夕终于撸完了毕业论文。把论文内容整理之后拆分成了三篇博客，希望和大家一起探索函数式反应型编程 (Functional Reactive Programming ， 缩写为 FRP) 的乐趣。

在第一章里，我们先了解一下，什么是 FRP 。

## 函数式 - Functional

函数式编程是一种编程范型，也就是指导如何编写程序的方法论。它强调函数必须被当成第一等公民对待，将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。

例如 +1 这样一个简单的操作，传统的做法是这样的：

	var foo = 0
	func increment() {
		foo++
	}

函数式的写法是这样的：

	func increment(foo: Int) -> Int {
		return foo + 1
	}

从这个例子中可以看到，函数式编程不依赖于外部的数据，而且也不修改外部数据的值，而是返回一个运算之后的新值。

### 函数式的特性

函数式编程具有以下几个特性：

#### 特性1：函数是第一等公民

所谓 第一等公民 (first class) ，指的是函数与其他数据类型一样，处于平等地位。既可以赋值给其他变量，也可以作为参数传入另一个函数，或者作为别的函数的返回值。

比如我们可以用 map 将数组通过指定的函数映射成另一个数组：

	let increment = { return $0 + 1 }
	[1,2,3].map(increment)  // [2,3,4]

这里的 increment 便是作为一个函数传入的。这个技术可以让你的函数就像变量一样来使用。也就是说，你的函数可以像变量一样被创建、修改、传递，返回或是在函数中嵌套其他函数。

#### 特性2：数据是不可变的

函数式语言里面的数据是不可修改的，只会返回新的值。这使得多个线程可以在不用锁的情况下并发地访问数据，因为数据本身并不会发生变化。

在 Clojure 这样的纯函数式语言中，变量默认是不可变的。如果想改变变量的值，可以通过 binding 进行动态绑定：

	user=> (def ^:dynamic x 1)
	#’user/x
	
	user=> (def ^:dynamic y 2)
	#’user/y
	
	user=> (+ x y)
	3
	
	user=> (binding [x 4 y 5]   ; 使用动态绑定覆盖原来绑定的值
	         (+ x y))
	9
	
	user=> (+ x y)
	3

#### 特性3：函数没有副作用

副作用指的是函数内部与外部互动，产生了函数运算以外的其他结果。最典型的情况，就是修改全局变量的值：
	
	var foo = 0
	func increment() {
		foo++
	}

函数式编程强调函数运算没有副作用，意味着函数要保持独立。函数的所有功能就是返回一个新值，没有其他行为，尤其是不得修改外部变量的值。

#### 特性4：函数具有确定性

函数的运行不依赖于外部变量和系统状态，只依赖于输入的参数。任何时候只要输入的参数相同，函数返回的新值总是相同的。

不确定性的函数示例：

    let foo = 3
    var i = 0
    func increment(value: Int) -> Int {
        return value + i
    }

    i = 1
    increment(foo)    // 4
    i = 2
    increment(foo)    // 5

可以看到，不确定性函数的运行结果往往与系统状态有关，不同的状态之下，返回值是不一样的。

确定性的函数示例：

    var foo = 3
    func increment(value: Int, step: Int) -> Int{
        return value + step
    }

    increment(foo, 1)   // 4
    increment(foo, 2)   // 5

函数的确定性有利于我们观察和理解程序的行为，因为它所依赖的东西只有参数本身。

### 函数式的函数

在函数式编程中，有些函数是抬头不见低头见的常客。在合适的时机利用合适的函数，可以有效地缩短代码，并且让代码更可读。在这里我们提前了解一下他们。

#### map

`map` 可以把一个数组按照一定的规则转换成另一个数组，定义如下：

    func map<U>(transform: (T) -> U) -> U[]

也就是说它接受一个函数叫做 `transform` ，然后这个函数可以把 T 类型的转换成 U 类型的并返回 (也就是 `(T) -> U`)，最终 `map` 返回的是 U 类型的集合。

下面的表达式更有助于理解：

    [ x1, x2, ... , xn].map(f) -> [f(x1), f(x2), ... , f(xn)]

如果用 `for in` 来实现，则需要这样：

    var newArray : Array<T> = []
    for item in oldArray {
        newArray += f(item)
    }


举个例子，我们可以这样把价格数组中的数字前面都加上 ￥ 符号：

    var oldArray = [10,20,45,32]
    var newArray = oldArray.map({money in "￥\(money)"})

    println(newArray) // [￥10, ￥20, ￥45, ￥32]

如果你觉得 `money in` 也有点多余的话可以用 `$0` ：
    
    newArray = oldArray.map({"\($0)€"})



#### filter

方法如其名， `filter` 起到的就是筛选的功能，参数是一个用来判断是否筛除的筛选闭包，定义如下：

    func filter(includeElement: (T) -> Bool) -> [T]


还是举个例子说明一下。首先先看下传统的 `for in` 实现的方法：

    var oldArray = [10,20,45,32]
    var filteredArray : Array<Int> = []
    for money in oldArray {
        if (money > 30) {
            filteredArray += money
        }
    }
    println(filteredArray)

奇怪的是这里的代码编译不通过：

    Playground execution failed: <EXPR>:15:9: error: 'Array<Int>' is not identical to 'UInt8'
            filteredArray += money


发现原来是 `+=` 符号不能用于 `append` ，只能用于 `combine` ，在外面包个 `[]` 即可：


    var oldArray = [10,20,45,32]
    var filteredArray : Array<Int> = []
    for money in oldArray {
        if (money > 30) {
            filteredArray += [money]
        }
    }
    println(filteredArray) // [45, 32]

用 `filter` 可以这样实现：

    var oldArray = [10,20,45,32]
    var filteredArray  = oldArray.filter({
        return $0 > 30
    })

    println(filteredArray) // [45, 32]

少了很多代码。（你真的好短啊！

#### reduce

`reduce` 函数解决了把数组中的值整合到某个独立对象的问题。定义如下：

    func reduce<U>(initial: U, combine: (U, T) -> U) -> U

好吧看起来略抽象。我们还是从 `for in` 开始。比如我们要把数组中的值都加起来放到 `sum` 里，那么传统做法是：


    var oldArray = [10,20,45,32]
    var sum = 0
    for money in oldArray {
        sum = sum + money
    }
    println(sum) // 107

`reduce` 有两个参数，一个是初始化的值，另一个是一个闭包，闭包有两个输入的参数，一个是原始值，一个是新进来的值，返回的新值也就是下一轮循环中的旧值。写几个小例子试一下：

    var oldArray = [10,20,45,32]
    var sum = 0
    sum = oldArray.reduce(0,{$0 + $1}) // 0+10+20+45+32 = 107
    sum = oldArray.reduce(1,{$0 + $1}) // 1+10+20+45+32 = 108
    sum = oldArray.reduce(5,{$0 * $1}) // 5*10*20*45*32 = 1440000
    sum = oldArray.reduce(0,+) // 0+10+20+45+32 = 107
    println(sum)


### 函数式和指令式的比较

对于开发者们来说，大家最熟悉的编程范例之一应该是指令式编程。指令式编程是一种描述计算机所需作出的行为的编程范型。

我们通过一个简单的例子来演示两者的区别。比如我们需要将数组中的元素乘以2，然后取出大于10的结果。

指令式编程的写法如下：

    var source = [1, 3, 5, 7, 9]
    var result = [Int]()
    for i in source {
        let timesTwo = i * 2
        if timesTwo > 10 {
            result.append(timesTwo)
        }
    }
    result  // [14, 18]

函数式编程的写法如下：

    var source = [1, 3, 5, 7, 9]
    let result = source.map { $0 * 2 }
                       .filter { $0 > 10 }
    result  // [14, 18]

这个简单的例子并不是争论哪种范例更清晰，而是为了演示二者之间的区别。

在指令式编程里，我们给计算机下发了如下指令：

- 遍历数组中的所有元素
- 在遍历中取出元素并乘以2
- 比较一下看看是否大于10
- 如果大于10则将它存到 result 数组中

在函数式编程中，我们则是这样解决问题：

- 将数组元素中的每个元素乘以2
- 在结果中选出大于10的元素

指令式编程通过下达指令完成任务，侧重于具体流程以及状态变化；而函数式编程则专注于结果，以及为了得到结果需要做哪些转换。


## 反应型 - Reactive

在日常开发中，我们经常需要监听某个属性，并且针对该属性的变化做一些处理。比如以下几个场景：

- 用户在输入邮箱的时候，监测输入的内容并在界面上提示是否符合邮箱规范。
- 用户在修改用户名之后，所有显示用户名的界面都要改为新的用户名。

外部输入信号的变化、事件的发生，这些都是典型的外部环境变化。根据外部环境的变化进行响应处理，直观上来讲像是一种自然地反应。我们可以将这种自动对变化作出响应的能力称为反应能力 (Reactive) 。

那么什么是反应型编程呢？

> Reactive programming is programming with asynchronous data streams.
> 反应型编程是异步数据流的编程。

对于移动端来说，异步数据流的概念并不陌生，变量、点击事件、属性、缓存，这些就可以成为数据流。

我们可以通过一些简单的 ASCII 字符来演示如何将事件转换成数据流：

    --a---b-c---d---X---|-->

    a, b, c, d 是具体的值，代表了某个事件
    X 表示发生了一个错误
    | 是这个流已经结束了的标记
    ----------> 是时间轴

比如我们要统计用户点击鼠标的次数，那么可以这样：

      clickStream: ---c----c--c----c------c-->
                   vvvvv map(c becomes 1) vvvv
                   ---1----1--1----1------1-->
                   vvvvvvvvv scan(+) vvvvvvvvv
    counterStream: ---1----2--3----4------5-->

反应型编程就是基于这些数据流的编程。而函数式编程则相当于提供了一个工具箱，可以方便的对数据流进行合并、创建和过滤等操作。

## Swift

Swift 是苹果公司在 2014 年推出的编程语言，用于编写 iOS 和 OS X 应用程序。它吸收了很多其它语言的语法特性，例如闭包、元组、泛型、结构体等等，这使得它的语法简洁而灵活。

Swift 本身并不是一门函数式语言，不过它有一些函数式的方法和特性，这让人不禁产生了使用 Swift 进行函数式编程的遐想。

和 Objective-C 相比， Swift 更接近于函数式，它支持以下特性：

- `map` `reduce` 等函数式函数
- 函数是一等公民
- 模式匹配
- ...

但是和真正的函数式语言相比， Swift 还差很多：

- 没有 `flatmap`
- 无法迅速取出 `head` 和 `tail`
- 没有 `foldLeft`
- ...

我们并不能因为 Swift 中的一些函数式特性就把它归为函数式语言，但是我们可以利用这些特性进行函数式 Style 的编程。

## 小结

终于花时间把前阵子炒得火热的函数式编程简单的了解了一圈，最大的感想便是：“原来代码可以这样写”。

在下一章中，我们将结合 Swift 和 RAC 写一写代码，一起体验 FRP 的魅力。



***
参考文献：

- Wiki
    - [Programming paradigm](http://en.wikipedia.org/wiki/Programming_paradigm)
    - [Functional reactive programming](http://en.wikipedia.org/wiki/Functional_reactive_programming)

- Functional
    - [Input and Output](http://blog.maybeapps.com/post/42894317939/input-and-output)
    - [What's the status of current FRP implementations?](http://stackoverflow.com/questions/13341937/whats-the-status-of-current-functional-reactive-programming-implementations)
    - [Learn X in Y minutes - Where X=clojure](http://learnxinyminutes.com/docs/clojure/)
    - [Functional Programming vs. Imperative Programming](https://msdn.microsoft.com/en-us/library/bb669144.aspx)
    - [Functional Programming in Swift](http://jamesonquave.com/blog/functional-programming-in-swift/)
    - [Swift Is Not Functional](http://robnapier.net/swift-is-not-functional)
    - [Functions and Closures in Swift](http://code.martinrue.com/posts/functions-and-closures-in-swift)

- Reactive
    - [Functional Reactive Programming on iOS](https://leanpub.com/iosfrp/)
    - [Reactive Swift](https://medium.com/swift-programming/reactive-swift-3b6050375534)
    - [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
    - [A Swift Reaction](http://napora.org/a-swift-reaction/)
    - [From MVC to MVVM in Swift](http://rasic.info/from-mvc-to-mvvm-in-swift/)
    - [Bindings, Generics, Swift and MVVM](http://rasic.info/bindings-generics-swift-and-mvvm/)
    - [Model-View-ViewModel for iOS](http://www.teehanlax.com/blog/model-view-viewmodel-for-ios/)
    - [Functional Reactive Programming in Swift](https://speakerdeck.com/ashfurrow/functional-reactive-programming-in-swift)

- 中文博客
    - [函数式反应型编程(FRP) - 实时互动应用开发的新思路](http://www.infoq.com/cn/articles/functional-reactive-programming)
    - [函数式编程初探](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)
    - [函数式编程](http://coolshell.cn/articles/10822.html)
    - [Clojure入门教程: Clojure – Functional Programming for the JVM中文版](http://xumingming.sinaapp.com/302/clojure-functional-programming-for-the-jvm-clojure-tutorial/)
    - [ReactiveCocoa 与 Functional Reactive Programming](http://blog.leezhong.com/ios/2013/06/19/frp-reactivecocoa.html)
    - [异步编程与响应式框架](http://blog.zhaojie.me/2010/09/async-programming-and-reactive-framework.html)