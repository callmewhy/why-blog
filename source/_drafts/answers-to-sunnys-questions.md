title: Sunny 面试题试答
date: 2015-07-15 21:39:57
tags: iOS
categories: 开发笔记
description: 在膜拜中学习，成就达成。
---

## [※]

#### @property 中有哪些属性关键字？

原子性相关： 

- `atomic` ：意味着读写操作都是原子的，同时只有一个线程访问实例变量。默认属性。
- `nonatomic` ：非原子操作，可同时被多个线程访问，效率比 `atomic` 快。

读写控制相关的：

- `readwrite` ：表示可以读写，同时拥有 `getter` 和 `setter` 方法。默认属性。
- `readonly` ：表示只读，只有 `getter` 没有`setter` 。可以通过 `getter = xx` 来指定 `getter` 方法名。

内存管理相关的：

- `assign` ：常用于值类型，如 `int` 、 `float` 、`double` 等，表示单纯的复制。是值类型的默认属性。如果把引用类型设为了 `assign` ，那么在对象被释放掉时候会变成野指针，导致应用崩溃。
- `retain` ：在 `setter` 方法中对传入的对象进行引用计数加1的操作。和 `strong` 是完全等价的操作，会自动转换成 `strong` 。
- `strong` ：强引用， `ARC` 中的关键字，表示实例变量持有传入的对象。在 `ARC` 中是引用类型的默认属性。
- `weak` ：弱引用，表示在 `setter` 方法中，对传入的对象不进行引用计数加1的操作。对象被释放后，用 `weak` 声明的实例变量指向 `nil` 。
- `copy` ：与 `strong` 类似，也是强引用，但强引用了传入对象的副本，而非对象本身。使用 `copy` 关键字的属性必须遵循 `NSCopying` 协议。一般用于使用类簇的类，例如 `NSString/NSMutableString` 、 `NSArray/NSMutableArray` ，以确保引用的对象是不可变的。
- `unsafe_unretained` ：在 `Cocoa` 和 `Cocoa Touch` 中，有些类还不支持弱引用类型，所以需要使用 `unsafe_unretained` 关键字。它和 `weak` 类似，但是如果引用的对象被释放了它并不会被置为 `nil` ，而是指向以前的地址，也就是说成了野指针。所以是 `unsafe` 的。



#### weak 属性需要在 dealloc 中置 nil 么？

不需要， `weak` 属性在对象销毁的时候会自动置为 `nil` 。

#### objc 使用什么机制管理对象内存？

引用计数。


## [※※]

#### @synthesize 和 @dynamic 分别有什么作用？

- `@synthesize` 可以生成 `setter` 和 `getter` 方法。 
- `@dynamic` 是告诉编译器： `setter` 和 `getter` 方法会有的，但是不在这里。


#### objc 中向一个 nil 对象发送消息将会发生什么？

`objc_msgSend` 方法会检查 `receiver` 是否为 `nil` ，如果为 `nil` 就不做操作直接返回。


#### IBOutlet 连出来的视图属性为什么可以被设置成 weak ?

因为它是根视图的 `subview` ，只要根视图是 `strong` 的，子视图就不需要设置 `strong` 。 

#### 使用block时什么情况会发生引用循环，如何解决？




#### 在block内如何修改block外部变量？

#### GCD的队列（dispatch_queue_t）分哪两种类型？

#### addObserver:forKeyPath:options:context:各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？

## [※※※]

#### ARC 下，不显示指定任何属性关键字时，默认的关键字都有哪些？

#### 用 @property 声明的 NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

#### @synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？

#### objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

#### 什么时候会报unrecognized selector的异常？

#### runloop和线程有什么关系？
#### runloop的mode作用是什么？


#### 如何调试BAD_ACCESS错误
#### lldb（gdb）常用的调试命令？
#### 使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题？

#### 如何手动触发一个value的KVO
#### 若一个类有实例变量NSString *_foo，调用setValue:forKey:时，可以以foo还是_foo作为key？



## [※※※※]

#### 一个objc对象如何进行内存布局？（考虑有父类的情况）
#### 一个objc对象的isa的指针指向什么？有什么作用？
#### runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）
#### 使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？

#### 以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？

#### ARC通过什么方式帮助开发者管理内存？
#### 不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）
#### BAD_ACCESS在什么情况下出现？

#### 如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）
#### dispatch_barrier_async的作用是什么？

#### KVC的keyPath中的集合运算符如何使用？
#### KVC和KVO的keyPath一定是属性么？


#### 下面的代码输出什么？

    @implementation Son : Father
    - (id)init
    {
        self = [super init];
        if (self) {
            NSLog(@"%@", NSStringFromClass([self class]));
            NSLog(@"%@", NSStringFromClass([super class]));
        }
        return self;
    }
    @end






## [※※※※※]

#### 在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？

#### objc中的类方法和实例方法有什么本质区别和联系？

#### _objc_msgForward函数是做什么的，直接调用它将会发生什么？

#### runtime如何实现weak变量的自动置nil？
#### 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

#### 猜想runloop内部是如何实现的？

#### 苹果是如何实现autoreleasepool的？

#### 苹果为什么要废弃dispatch_get_current_queue？
#### 以下代码运行结果如何？

    - (void)viewDidLoad {
        [super viewDidLoad];
        NSLog(@"1");
        dispatch_sync(dispatch_get_main_queue(), ^{
            NSLog(@"2");
        });
        NSLog(@"3");
    }

#### 如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？
#### apple用什么方式实现对一个对象的KVO？
#### IB中User Defined Runtime Attributes如何使用？






***

参考文献

- [Objective-C中的@property](http://www.devtalking.com/articles/you-should-to-know-property/)
- [Encapsulating Data](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html)
- [@synthesize vs @dynamic, what are the differences?](http://stackoverflow.com/questions/1160498/synthesize-vs-dynamic-what-are-the-differences)
- [Messaging nil in Objective-C](http://unixjunkie.blogspot.com/2006/02/messaging-nil-in-objective-c.html)
- [Nil](http://ridiculousfish.com/blog/posts/nil.html)