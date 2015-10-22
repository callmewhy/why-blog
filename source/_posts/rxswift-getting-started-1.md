title: RxSwift 入坑手册 Part1 - 示例实战
date: 2015-09-23 14:45:59
tags: RxSwift
categories: 开发笔记
description: 万水千山总是情，留坑在此行不行=。=
---

## Intro

这部分主要是学习 [RxSwift](https://github.com/ReactiveX/RxSwift/) 项目中的示例项目，了解 `RxSwift` 在实际 iOS 开发中的正确打开方式。

## Demo1: [GitHub Signup](https://github.com/ReactiveX/RxSwift/tree/master/RxExample/RxExample/Examples/GitHubSignup)

第一个示例是 GitHub 注册账号的例子。输入用户名、密码、重复密码，然后提交注册。

### username

在注册流程中，用户名校验是一个很常见的功能。我们一般需要对用户名做如下检查和流程：

- 是否不为空
- 是否不包含非法字符
- 是否没有被注册过
- 以上均通过，联网注册
- 是否成功连接上服务器
- 服务器是否正确处理并返回结果

要通过 RxSwift 实现以上流程，可以划分成如下几步。

#### rx_text

如果想监听文字输入，我们最好能有一个 `Observable` 的对象不断地给我们发送新的输入值。`rx_text` 是 RxSwift 针对 Cocoa 库做的各种封装中的一个，可以简单看下它的定义：

```swift
    extension UITextField {
        /**
        Reactive wrapper for `text` property.
        */
        public var rx_text: ControlProperty<String> {
            return rx_value(getter: { [weak self] in
                self?.text ?? ""
            }, setter: { [weak self] value in
                self?.text = value
            })
        }
    }
```

返回的 `ControlProperty` 遵循 `ControlPropertyType` 协议：

```swift
    public protocol ControlPropertyType : ObservableType, ObserverType {
        
        /**
        - returns: `ControlProperty` interface
        */
        func asControlProperty() -> ControlProperty<E>
    }
```


可以看到，它既是一个可被订阅者 (`ObservableType`)，又是一个订阅者 (`ObserverType`)，也就是说它是一个 [`Subject`](http://blog.callmewhy.com/2015/09/21/rxswift-getting-started-0/#Subjects) 对象。由于它是 `Observable` 的，所以我们可以通过 `map` 把它转换成想要的输出，例如下面这段代码会实现『每输入一个字就在控制台输出当前内容』的功能（记得 RAC 的系列教程似乎也是这个节奏）：

```swift
    @IBOutlet weak var usernameOutlet: UITextField!
    override func viewDidLoad() {
        super.viewDidLoad()
        let username = usernameOutlet.rx_text
        username.subscribeNext {
            print($0)
        }
}
```

#### validate

接下来就是验证用户名的阶段了。首先需要一个方法，将用户名转换成一串流。为什么是流？为了统一口径。先看代码：

```swift
    typealias ValidationResult = (valid: Bool?, message: String?)

    func validateUsername(username: String) -> Observable<ValidationResult> {
        // 如果用户名为空
        if username.characters.count == 0 {
            return just((false, nil))
        }
        
        // 如果用户名中出现非法字符
        if username.rangeOfCharacterFromSet(NSCharacterSet.alphanumericCharacterSet().invertedSet) != nil {
            return just((false, "Username can only contain numbers or digits"))
        }
        
        // 加载中的值
        let loadingValue = (valid: nil as Bool?, message: "Checking availabilty ..." as String?)
        
        // 校验用户名
        return API.usernameAvailable(username)
            .map { available in
                if available {
                    return (true, "Username available")
                }
                else {
                    return (false, "Username already taken")
                }
            }
            .startWith(loadingValue)    // 在加载结果之前插入 加载中 这个状态的事件值
    }
```
    
梳理一下：

- 如果用户名为空，返回 `(false, nil)` 完事儿
- 如果用户名有非法字符，返回 `(false, "Username can only contain numbers or digits")` 完事儿
- 如果本地检查没问题，发给服务器检查，先发送一个事件 `loadingValue` 表示正在加载，加载成功再发送结果事件

这就是我所理解的『统一口径』。虽然本地检查分分钟就能给你个结果，但是如果统一都用『流』来表述，外部处理起来会简单得多。不用管具体的结果是什么，只需要知道是一个 `Observable` 对象，并且随之而来的是一串事件，就足够了。这一串事件，有可能只有一个，提示用户名不能为空；也有可能有很多，先提示正在加载，然后再提示注册成功。在外部来看，这就是一个事件流。

试想一下如果用 UIKit 的那一套来写这个流程，肯定要监听个 `valueChanged` 事件然后在委托方法里先判断 A ，如果不符合就刷新 UI ；再判断 B ，不符合就刷新 UI ；最后发请求给服务器，刷新 UI 提示等待，然后加载完了再刷新个 UI 。。。

#### switch

把上面的验证代码用到项目中大概就是这样：

```swift
    let usernameValidation = username
        .map { username in
            return validationService.validateUsername(username)
        }
```

如果把每次的文字改动事件用 v 来表示，那整个事件序列应该是这样的：

    ----V------V--V---V---V----

而用了 `validateUsername` 之后，它会把以前的值转换成一个新的序列，就成了这样：

    ----|------|--|---|---|----
        |      |  |   |   |
        V      V  V   V   V
        |      |  |   |   |
        |      V  |   V   |
        |      |  |   |   |

瞬间变成了二维世界了，我们可以用 [`switch`](http://blog.callmewhy.com/2015/09/21/rxswift-getting-started-0/#switch) 来『降维』（这名词是我自己起的，如果有更好的称呼欢迎随时打脸(￣ε(#￣)☆╰╮(￣▽￣///))。这里我们选用 `switchLastest` ：

```swift
    let usernameValidation = username
        .map { username in
            return validationService.validateUsername(username)
        }
        .switchLatest()
```

不妨看一下 ObservableType 的定义：

```swift
    extension ObservableType where E : ObservableType {
        public func switchLatest() -> Observable<E.E> {
            return Switch(sources: self.asObservable())
        }
    }
```

这个扩展是针对『自己是 `ObservableType` 且自己监听的事件也是 `ObservableType` 』这一类对象的。通过 `switchLatest` 方法我们可以将前面的二维结构梳理成一维结构，每次自动切换到新的序列的最新事件上。

    ----|------|--|---|---|----
        V      |  |   |   |
        |------|  |   |   |
        |      V  |   |   |
        |      V  |   |   |
        |      |--|   |   |
        |      |  V   |   |
        |      |  |---|   |
        |      |  |   V   |
        |      |  |   V   |
        |      |  |   |---|
        |      |  |   |   V

执行结果如下：

```swift
    let usernameValidation = username
            .map { username in
                return validationService.validateUsername(username)
            }
            .switchLatest()
            .subscribe {
                print("Event: - \($0)")
            }

    ----- Output -----
    // 输入 12!
    Event: - Next((Optional(false), Optional("Username can only contain numbers or digits")))
    // 输出 abc
    Event: - Next((nil, Optional("Checking availabilty ...")))
    Event: - Next((Optional(false), Optional("Username already taken")))
```

#### replay

我们在 `map` 里调用了 `validateUsername` 方法，这会导致如果有多个订阅者的话，会重复调用多次。

例如这个例子：

```swift
    let sequenceOfElements = sequenceOf(0).map { r -> Int in
        print("MAP")    // twice
        return r * 2
    }    
    let subscription = sequenceOfElements
        .subscribe { event in
            print("1 - \(event)")
    }
    let subscription2 = sequenceOfElements
        .subscribe { event in
            print("2 - \(event)")
    }

    --- map example ---
    MAP
    1 - Next(0)
    1 - Completed
    MAP
    2 - Next(0)
    2 - Completed
```

我一开始很疑惑，明明订阅的是 `map` 之后的队列，但是为什么 `subscribe` 每次都会重新 `map` 一次呢？

经提醒之后又去仔细翻阅了入门文档 [Getting Started](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md) ，看到了下面这段话：

> Every subscriber upon subscription usually generates it's own separate sequence of elements. Operators are stateless by default. There is vastly more stateless operators then stateful ones.

这么一想就说得通了，一切都是为了 `stateless` 。如果 `subscribe` 的是 `map` 后的结果，那就意味着需要多存储一个状态，而状态的增加往往意味着复杂度的指数级增长。

为了解决这个多次订阅会多次执行的问题，我们需要 `shareReplay` ，看下这个示例：

```swift
    let sequenceOfInts = PublishSubject<Int>()
    let a = sequenceOfInts.map{ i -> Int in
        print("MAP---\(i)")
        return i * 2
    }.shareReplay(3)
    let b = a.subscribeNext {
        print("--1--\($0)")
    }
    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Next(2))
    let c = a.subscribeNext {
        print("--2--\($0)")
    }
    sequenceOfInts.on(.Next(3))
    sequenceOfInts.on(.Next(4))
    let d = a.subscribeNext {
        print("--3--\($0)")
    }
    sequenceOfInts.on(.Completed)

    --- shareReplay example ---
    MAP---1
    --1--2
    MAP---2
    --1--4
    --2--2
    --2--4
    MAP---3
    --1--6
    --2--6
    MAP---4
    --1--8
    --2--8
    --3--4
    --3--6
    --3--8
```

`shareReplay` 会返回一个新的事件序列，它监听底层序列的事件，并且通知自己的订阅者们。不过和传统的订阅不同的是，它是通过『重播』的方式通知自己的订阅者。就像是过目不忘的看书，但是每次都只记得最后几行的内容，在有人询问的时候就背诵出来。从上面的例子可以看到，通过 `shareReplay` 订阅的 `map` 并不会调用多次。所以我们也可以把它应用到 `validateUsername` 上：

```swift
    let usernameValidation = username
        .map { username in
            return validationService.validateUsername(username)
        }
        .switchLatest()
        .shareReplay(1)
```

这样就不会出现『多次订阅导致重复地检查用户名是否可用』的情况了。

#### usernameAvailable

前面梳理了基本的用户名校验流程，接下来看下联网检测这部分是如何实现的。

联网检测用户名是否可用主要是访问用户名对应的 github 地址然后查看是否是 404 ，如果不是那就说明已经被注册了。核心代码如下：
    
```swift
    func usernameAvailable(username: String) -> Observable<Bool> {
        let URL = NSURL(string: "https://github.com/\(URLEscape(username))")!
        let request = NSURLRequest(URL: URL)
        return self.URLSession.rx_response(request)
            .map { (maybeData, maybeResponse) in
                if let response = maybeResponse as? NSHTTPURLResponse {
                    return response.statusCode == 404
                }
                else {
                    return false
                }
            }
            .observeOn(self.dataScheduler)
            .catchErrorJustReturn(false)
    }
```

和前面的 `rx_value` 相似， `rx_response` 是针对 `NSURLSession` 的扩展。通过 `observeOn` 将监听事件绑定在了 `dataScheduler` 上。最后 `catchErrorJustReturn(false)` 表明如果出现异常就返回个 `false` 。

[`Scheduler`](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/Schedulers.md) 是一种 Rx 里的任务运行机制，类似的 `gcd` 里的 `dispatch queue` 。可以通过 `observeOn` 切换 `scheduler` ：

```swift
    sequence1
        .observeOn(backgroundScheduler)
        .map { n in
          println("This is performed on background scheduler")
        }
        .observeOn(MainScheduler.sharedInstance)
        .map { n in
          println("This is performed on main scheduler")
        }
```

### password

密码的检测相比较用户名而言就简单很多，核心代码如下：

```swift
    func validatePassword(password: String) -> ValidationResult {
        let numberOfCharacters = password.characters.count
        if numberOfCharacters == 0 {
            return (false, nil)
        }
        if numberOfCharacters < minPasswordCount {
            return (false, "Password must be at least \(minPasswordCount) characters")
        }
        return (true, "Password acceptable")
    }
```

注意这里返回了 `ValidationResult` ，因为所有校验都是本地完成的。

接下来就是重复密码的校验，这部分比较有意思，通过 [`combineLatest`](http://blog.callmewhy.com/2015/09/21/rxswift-getting-started-0/#combineLatest) 将两个序列合并起来：

```swift
    let repeatPasswordValidation = combineLatest(password, repeatPassword) { (password, repeatedPassword) in
            validationService.validateRepeatedPassword(password, repeatedPassword: repeatedPassword)
        }
        .shareReplay(1)
```
    
然后 `validateRepeatedPassword` 方法如下：

```swift
    func validateRepeatedPassword(password: String, repeatedPassword: String) -> ValidationResult {
        if repeatedPassword.characters.count == 0 {
            return (false, nil)
        }
        
        if repeatedPassword == password {
            return (true, "Password repeated")
        }
        else {
            return (false, "Password different")
        }
    }
```
    
这几个例子基本都是把事件序列进行组装然后『外包』给其他对象去处理。

### bindValidationResultToUI

检查也检查好了，接下来的就是更新 UI 了，用户名非法、两次密码不一致，这些都需要通过刷新 UI 告知用户。也就是说，需要把前面定义的『事件流』和『用户界面』绑定起来。看下这个绑定的方法：

```swift
    func bindValidationResultToUI(source: Observable<ValidationResult>,
        validationErrorLabel: UILabel) {
        source
            .subscribeNext { v in
                let validationColor: UIColor
                
                if let valid = v.valid {
                    validationColor = valid ? okColor : errorColor
                }
                else {
                   validationColor = UIColor.grayColor()
                }
                
                validationErrorLabel.textColor = validationColor
                validationErrorLabel.text = v.message ?? ""
            }
            .addDisposableTo(disposeBag)
    }
```
    
在这里出现了 `addDisposableTo(disposeBag)` ，在此需要解释一下 [`disposing`](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing)  的相关概念。

一个事件流的终结除了前面了解的各种事件之外，还有一种方法，就是 `dispose` ，释放掉所有的资源。比如这个例子：

```swift
    let subscription = interval(0.3, scheduler)
        .subscribe { (e: Event<Int64>) in
            println(e)
        }

    NSThread.sleepForTimeInterval(2)

    subscription.dispose()

    ----- Dispose Sample -----
    0
    1
    2
    3
    4
    5
```
    
然而 `dispose` 方法是不推荐使用的，推荐使用更好的解决方案， `DisposeBag` 就是一个。`addDisposableTo(disposeBag)` 有点像是 ARC ，先把分配的资源统一丢到袋子里 (有点像是 `autoreleasepool`) ，然后当 `disposeBag` 销毁的时候就一起销毁这些资源。在代码里可以看到，只要有 `subscribe` 的基本在最后都会兜上一个 `.addDisposableTo(disposeBag)` 用来处理资源自动销毁的问题。

### signupEnabled

检查完毕之后，如果所有条件都符合，那就需要把 `Signup` 按钮高亮，高亮的逻辑是把多个数据流合并在了一起：

 ```swift
    let signupEnabled = combineLatest(
         usernameValidation,
         passwordValidation,
         repeatPasswordValidation,
         signingProcess
     ) { un, p, pr, signingState in
         return (un.valid ?? false) && (p.valid ?? false) && (pr.valid ?? false) && signingState != SignupState.SigningUp
     }
 ```
     
在基本的流都构建完毕的情况下，各种需求更多的是对流的组合拼装。比如这里就再次用到了 `usernameValidation` 这个流，还好前面有 [`shareReplay`](http://blog.callmewhy.com/2015/09/23/rxswift-getting-started-1/#replay) 罩着，我们想复用多少次都没问题。


### signingProcess

在点击注册按钮之后，就是具体的注册流程了，注册流程的代码是这样的：

```swift
    let signingProcess = combineLatest(username, password) { ($0, $1) }
        .sampleLatest(signupSampler)
        .map { (username, password) in
            return API.signup(username, password: password)
        }
        .switchLatest()
        .startWith(SignupState.InitialState)
        .shareReplay(1)
```
    
这里有个 `sampleLatest` ，在了解它之前先要了解什么是 `sample` 。

#### sample

`sample` 就是一次『采样』，当收到采样事件的时候，就会从事件队列中取出一个事件作为『样本』，并发送到事件流里。如果下一次又要采样了，就会从两次采样之间的事件队列中选择最后一个事件，如果两次采集之间没有新的事件就不会进行任何操作。

可以看下这个例子帮助理解：

```swift
    let s = PublishSubject<Int>()
    let o = PublishSubject<String>()
    
    let subscription = s
        .sample(o)
        .subscribe { event in
            print(event)
    }
    
    s.on(.Next(1))
    o.on(.Next("A"))
    s.on(.Next(2))
    s.on(.Next(3))
    o.on(.Next("B"))
    o.on(.Next("C"))

    --- sample example ---
    Next(1)
    Next(3)
```
    
`sampleLatest` 就是，即使两次采样期间没有新的事件也没关系，取整个队列的最后一个事件作为输出。还是上面那个例子：

```swift
    let s = PublishSubject<Int>()
    let o = PublishSubject<String>()
    
    let subscription = s
        .sampleLatest(o)
        .subscribe { event in
            print(event)
    }
    
    s.on(.Next(1))
    o.on(.Next("1"))
    s.on(.Next(2))
    s.on(.Next(3))
    o.on(.Next("2"))
    o.on(.Next("3"))

    --- sample example ---
    Next(1)
    Next(3)
    Next(3)
```
    
所以上面的注册流程代码也就可以理解了：

- 先把 username 和 password 绑起来
- 将注册按钮的点击事件作为一个触发点，每次点击都会获取最新的账号密码走下面的流程
- 调用 `API.signup` 进行注册
- 将 `map` 之后的二维队列拍平，切换到最新的队列上
- 将状态置为初始状态
- 通过 `shareReplay` 避免重复订阅导致的反复执行的问题

#### signup

项目里的注册功能只是一个 `mock` 而已，并没有真的访问 API ：

```swift
    func signup(username: String, password: String) -> Observable<SignupState> {
        // this is also just a mock
        let signupResult = SignupState.SignedUp(signedUp: arc4random() % 5 == 0 ? false : true)
        return [just(signupResult), never()]
            .concat()
            .throttle(2, MainScheduler.sharedInstance)
            .startWith(SignupState.SigningUp)
    }
```


在这里可以看到 `never()` 的正确打开方式：用于无限等待。  `concat` 将上面两个序列首尾拼接起来，然后 `throttle` 等价于 [`debounce`](http://rxmarbles.com/#debounce) ：如果两个事件的时间间隔小于某个特定值，就会忽视掉前面一个。通过 `never` + `throttle` 伪造了一种等待加载2秒然后返回注册结果的错觉。

#### disposeBag

定义了事件流之后，我们就可以通过 `subscribeNext` 来刷新 UI 了：

```swift
    signingProcess
        .subscribeNext { [unowned self] signingResult in
            switch signingResult {
            case .SigningUp:
                self.signingUpOulet.hidden = false
            case .SignedUp(let signed):
                self.signingUpOulet.hidden = true
                
                let alertView: UIAlertView
                
                if signed {
                    alertView = UIAlertView(title: "GitHub", message: "Mock signed up to GitHub", delegate: nil, cancelButtonTitle: "OK")
                }
                else {
                    alertView = UIAlertView(title: "GitHub", message: "Mock signed up failed", delegate: nil, cancelButtonTitle: "OK")
                }
                
                alertView.show()
            default:
                self.signingUpOulet.hidden = true
            }
        }
        .addDisposableTo(disposeBag)
```
    

注意，每一次 `subscribe` 都要及时回收资源，在示例代码中是都通过 `addDisposableTo(disposeBag)` 统一处理了。在 `disposeBag` 重新赋值的时候就会自动清理资源。

项目中一共有三个地方调用了 `disposeBag = DisposeBag()` ：

- 定义变量的时候：

```swift
    var disposeBag = DisposeBag()
```

- `viewDidLoad` 里：

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        self.disposeBag = DisposeBag()
        ...
    }
```

- `willMoveToParentViewController` 里：

```swift
    // This is one of the reasons why it's a good idea for disposal to be detached from allocations.
    // If resources weren't disposed before view controller is being deallocated, signup alert view
    // could be presented on top of wrong screen or crash your app if it was being presented while
    // navigation stack is popping.
    // This will work well with UINavigationController, but has an assumption that view controller will
    // never be readded as a child view controller.
    // It it was readded UI wouldn't be bound anymore.
    override func willMoveToParentViewController(parent: UIViewController?) {
        if let parent = parent {
            assert(parent.isKindOfClass(UINavigationController), "Please read comments")
        }
        else {
            self.disposeBag = DisposeBag()
        }
    }
```

在 `UINavigationController` 中，这样的代码没有问题，但是当把这个 `view controller` 作为 `child view controller` 添加到其他界面的时候，会直接走到断言处。原因是在 `child view controller` 中，会先调用 `viewDidLoad` 再调用 `willMoveToParentViewController` ，好不容易绑定好的界面和事件流，结果直接 `self.disposeBag = DisposeBag()` 就给解绑了，自然出了问题。


## Demo2: [Wiki](https://github.com/ReactiveX/RxSwift/tree/master/RxExample/RxExample/Examples/WikipediaImageSearch)

为什么在第一篇开头我就说：我又要挖坑了？因为我预见到。。。这个系列可能还没来得及写完就出其他事情了=。=

果然。

接下来专心前端和推荐算法了。有缘我们坑里再见。。。


## Next

各种异步各种回调的好处是整个应用行云流水让人感觉十分舒适，坏处是和 RAC 一样断点调试基本就是噩梦：

![](http://ww2.sinaimg.cn/large/61d238c7gw1ewdy2ss0l9j21kw0xdtlw.jpg)



参考文献：

- [replay](http://reactivex.io/documentation/operators/replay.html)
- [Getting Started](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md)