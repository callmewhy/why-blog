title: Functional Reactive Programming in Swift
date: 2015-03-21 22:25:37
tags: Reactive
categories: 开发笔记
description: 换个思路，换个心情，写一些有趣的代码。
---

## 背景

### 函数式 - Functional

#### 函数式的介绍

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

#### 函数式的特性

函数式编程具有以下几个特性：

##### 特性1：函数是第一等公民

所谓 第一等公民 (first class) ，指的是函数与其他数据类型一样，处于平等地位。既可以赋值给其他变量，也可以作为参数传入另一个函数，或者作为别的函数的返回值。

比如我们可以用 map 将数组通过指定的函数映射成另一个数组：

	let increment = { return $0 + 1 }
	[1,2,3].map(increment)  // [2,3,4]

这里的 increment 便是作为一个函数传入的。这个技术可以让你的函数就像变量一样来使用。也就是说，你的函数可以像变量一样被创建、修改、传递，返回或是在函数中嵌套其他函数。

##### 特性2：数据是不可变的

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

##### 特性3：函数没有副作用

副作用指的是函数内部与外部互动，产生了函数运算以外的其他结果。最典型的情况，就是修改全局变量的值：
	
	var foo = 0
	func increment() {
		foo++
	}

函数式编程强调函数运算没有副作用，意味着函数要保持独立。函数的所有功能就是返回一个新值，没有其他行为，尤其是不得修改外部变量的值。

##### 特性4：函数具有确定性

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


#### 函数式和指令式的比较

在编程范型中，与函数式编程 (Functional) 相对应的便是指令式编程 (Imperative) 。指令式编程是一种描述计算机所需作出的行为的编程范型。

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

指令式编程通过下达指令完成任务，侧重于具体流程以及状态变化；而函数式编程则专注于注结果，以及为了得到结果需要做哪些转换。


### 反应型 - Reactive

在日常开发中，我们经常需要监听某个属性，并且针对该属性的变化做一些处理。比如以下几个场景：

- 用户在输入邮箱的时候，监测输入的内容并在界面上提示是否符合邮箱规范。
- 用户在修改用户名之后，所有显示用户名的界面都要改为新的用户名。

外部输入信号的变化、事件的发生，这些都是典型的外部环境变化。随着外部环境的变化，进行响应，直观上来讲像是一种自然地反应。我们可以将这种自动对变化作出响应的能力称为反应能力 (Reactive) 。

### Swift
Swift 的语言特点，Swift 为什么适合 FRP 。
- Generic
- Closure

## 为什么
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


- 真实案例对比
- MVVM 结合

*****上面都不涉及代码，纯理论扯淡*****

## 实战
- RAC3.0 的一个登陆注册的实例
- 结合 RAC 的 MVVM 实例


## 实现原理
- ReactiveCocoa 实现原理
- Swift RX 库的实现原理







最后引用耗子叔的一段话：

> 我希望这篇浅显易懂的文章能让你感受到函数式编程的思想，就像面向对象编程、泛型编程、过程式编程一样。不用太纠结于程序是面向对象的还是函数式的，更重要的是品味其中的味道。


***
参考文献：

- [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
- [Input and Output](http://blog.maybeapps.com/post/42894317939/input-and-output)
- [Functional Reactive Programming on iOS](https://leanpub.com/iosfrp/)
- [Reactive​Cocoa](http://nshipster.cn/reactivecocoa/)
- [ReactiveCocoa Tutorial – The Definitive Introduction: Part 1/2](http://www.raywenderlich.com/62699/reactivecocoa-tutorial-pt1)
- [ReactiveCocoa Tutorial – The Definitive Introduction: Part 2/2](http://www.raywenderlich.com/62796/reactivecocoa-tutorial-pt2)
- [MVVM Tutorial with ReactiveCocoa: Part 1/2](http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1)
- [MVVM Tutorial with ReactiveCocoa: Part 2/2](http://www.raywenderlich.com/74131/mvvm-tutorial-with-reactivecocoa-part-2)
- [函数式反应型编程(FRP) - 实时互动应用开发的新思路](http://www.infoq.com/cn/articles/functional-reactive-programming)
- [](https://medium.com/swift-programming/reactive-swift-3b6050375534)
- [](http://www.infoq.com/cn/articles/functional-reactive-programming)
- [](http://en.wikipedia.org/wiki/Programming_paradigm)
- [](http://blog.leezhong.com/ios/2013/06/19/frp-reactivecocoa.html)
- [](http://en.wikipedia.org/wiki/Functional_reactive_programming)
- [](http://blog.zhaojie.me/2010/09/async-programming-and-reactive-framework.html)
- [](http://lambdor.net/?p=171)
- [](http://www.alexcurylo.com/blog/2013/03/27/reactivecocoa/)
- [](http://stackoverflow.com/questions/13341937/whats-the-status-of-current-functional-reactive-programming-implementations)
- [](http://rasic.info/from-mvc-to-mvvm-in-swift/)
- [](http://rasic.info/bindings-generics-swift-and-mvvm/)
- [](http://napora.org/a-swift-reaction/)
- [](http://jamesonquave.com/blog/functional-programming-in-swift/)
- [](http://napora.org/a-swift-reaction/)
- [](http://rasic.info/bindings-generics-swift-and-mvvm/)
- [](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)
- [](http://coolshell.cn/articles/10822.html)
- [](http://xumingming.sinaapp.com/302/clojure-functional-programming-for-the-jvm-clojure-tutorial/)
- [](http://learnxinyminutes.com/docs/clojure/)
- [Functional Programming vs. Imperative Programming](https://msdn.microsoft.com/en-us/library/bb669144.aspx)