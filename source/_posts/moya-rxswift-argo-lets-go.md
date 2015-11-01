title: 通过 Moya+RxSwift+Argo 完成网络请求
date: 2015-11-01 11:27:59
tags: Swift
categories: 开发笔记
description: 网络方案千千万，换个思路玩玩看。
---

最近在新项目中尝试使用 Moya+RxSwift+Argo 进行网络请求和解析，感觉还阔以，再来给大家安利一波。

## [Moya](https://github.com/Moya/Moya)

`Moya` 是一个基于 `Alamofire` 的更高层网络请求封装，深入学习请参见官方文档：[Moya/Docs](https://github.com/Moya/Moya/tree/master/docs)。

使用 `Moya` 之后网络请求一般长了这样：

```swift
provider.request(.UserProfile("ashfurrow")) { (data, statusCode, response, error) in
    if let data = data {
        // do something with the data
    }
}
```

`Moya` 提供了很多不错的特性，其中我感觉最棒的是 `stub` ，配合 `sampleData` 分分钟就完成了单元测试：

```swift
private let provider = MoyaProvider<ItemAPI>(stubClosure: MoyaProvider.ImmediatelyStub)
```

注意这里的 `MoyaProvider.ImmediatelyStub` ，我原以为它是个枚举类型，看了 `MoyaProvider` 定义发现这里应该传个 `closure` ，看了 `ImmediatelyStub` 的定义发现原来它是个类方法：

```swift
public typealias StubClosure = Target -> Moya.StubBehavior

override public init(stubClosure: StubClosure = MoyaProvider.NeverStub, ...) {

}

public final class func ImmediatelyStub(_: Target) -> Moya.StubBehavior {
    return .Immediate
}
```

如果想打印每次请求的参数，在组装 `endpoint` 的时候打印即可：

```swift
private func endpointMapping<Target: MoyaTarget>(target: Target) -> Endpoint<Target> {
    if let parameters = target.parameters {
        log.verbose("\(parameters)")
    }
    return MoyaProvider.DefaultEndpointMapping(target)
}

private let provider = RxMoyaProvider<ItemAPI>(endpointClosure: endpointMapping)
```


## [RxSwift](https://github.com/ReactiveX/RxSwift)

`RxSwift` 前面强行[安利](http://blog.callmewhy.com/tags/RxSwift/)过两波，在此不再赘述啦，`Moya` 本身提供了 `RxSwift` 扩展，可以无缝衔接 `RxSwift` 和 `ReactiveCocoa` ，于是打开方式变成了这样：


```swift
 private let provider = RxMoyaProvider<ItemAPI>()
 private var disposeBag = DisposeBag()
 
 extension ItemAPI {
     static func getNewItems(completion: [Item] -> Void) {
         disposeBag = DisposeBag()
         provider
             .request(.GetItems())
             .subscribe(
                 onNext: { items in
                     completion(items)
                 }
             )
             .addDisposableTo(disposeBag)
     }
 }
 ```

`Moya` 的核心开发者、同时也是 [Artsy](https://github.com/Artsy/) 的成员：[Ash Furrow](https://github.com/ashfurrow)， 在 AltConf 做过一次 《[Functional Reactive Awesomeness With Swift](https://realm.io/news/altconf-ash-furrow-functional-reactive-swift/)》 的分享，推荐大家看一下，很可爱的！

## [Argo](https://github.com/thoughtbot/argo)

`Argo` 是 `thoughtbot` 开源的函数式 `JSON` 解析转换库。说到 `thoughtbot` 就不得不提他司关于 `JSON` 解析质量很高的一系列文章：

- [Efficient JSON in Swift with Functional Concepts and Generics](https://robots.thoughtbot.com/efficient-json-in-swift-with-functional-concepts-and-generics)
- [Real World JSON Parsing with Swift](https://robots.thoughtbot.com/real-world-json-parsing-with-swift)
- [Parsing Embedded JSON and Arrays in Swift](https://robots.thoughtbot.com/parsing-embedded-json-and-arrays-in-swift)
- [Functional Swift for Dealing with Optional Values](https://robots.thoughtbot.com/functional-swift-for-dealing-with-optional-values)

`Argo` 基本上就是沿着这些文章的思路写出来的，相关的库还有 [Runes](https://github.com/thoughtbot/Runes) 和 [Curry](https://github.com/thoughtbot/Curry)。

使用 `Argo` 做 `JSON` 解析很有意思，大致长这样：

```swift
struct Item {
    let id: String
    let url: String
}

extension Item: Decodable {
    static func decode(j: JSON) -> Decoded<Item> {
        return curry(Item.init)
            <^> j <| "id"
            <*> j <| "url"
    }
}
```

至于这其中各种符号的缘由，在几篇博客中都有讲解，还是挺有意思滴。

## All

说完这三者，如何把它们串起来呢？[Emergence](https://github.com/artsy/Emergence) 中的 [Observable/Networking](https://github.com/artsy/Emergence/blob/master/Emergence/Contexts/Networking/Observable%2BNetworking.swift) 给了我们答案。稍微整理后如下：


```swift
enum ORMError : ErrorType {
    case ORMNoRepresentor
    case ORMNotSuccessfulHTTP
    case ORMNoData
    case ORMCouldNotMakeObjectError
}

extension Observable {
    private func resultFromJSON<T: Decodable>(object:[String: AnyObject], classType: T.Type) -> T? {
        let decoded = classType.decode(JSON.parse(object))
        switch decoded {
        case .Success(let result):
            return result as? T
        case .Failure(let error):
            log.error("\(error)")
            return nil
            
        }
    }
    
    func mapSuccessfulHTTPToObject<T: Decodable>(type: T.Type) -> Observable<T> {
        return map { representor in
            guard let response = representor as? MoyaResponse else {
                throw ORMError.ORMNoRepresentor
            }
            guard ((200...209) ~= response.statusCode) else {
                if let json = try? NSJSONSerialization.JSONObjectWithData(response.data, options: .AllowFragments) as? [String: AnyObject] {
                    log.error("Got error message: \(json)")
                }
                throw ORMError.ORMNotSuccessfulHTTP
            }
            do {
                guard let json = try NSJSONSerialization.JSONObjectWithData(response.data, options: .AllowFragments) as? [String: AnyObject] else {
                    throw ORMError.ORMCouldNotMakeObjectError
                }
                return self.resultFromJSON(json, classType:type)!
            } catch {
                throw ORMError.ORMCouldNotMakeObjectError
            }
        }
    }

    func mapSuccessfulHTTPToObjectArray<T: Decodable>(type: T.Type) -> Observable<[T]> {
        return map { response in
            guard let response = response as? MoyaResponse else {
                throw ORMError.ORMNoRepresentor
            }
            
            // Allow successful HTTP codes
            guard ((200...209) ~= response.statusCode) else {
                if let json = try? NSJSONSerialization.JSONObjectWithData(response.data, options: .AllowFragments) as? [String: AnyObject] {
                    log.error("Got error message: \(json)")
                }
                throw ORMError.ORMNotSuccessfulHTTP
            }
            
            do {
                guard let json = try NSJSONSerialization.JSONObjectWithData(response.data, options: .AllowFragments) as? [[String : AnyObject]] else {
                    throw ORMError.ORMCouldNotMakeObjectError
                }
                
                // Objects are not guaranteed, thus cannot directly map.
                var objects = [T]()
                for dict in json {
                    if let obj = self.resultFromJSON(dict, classType:type) {
                        objects.append(obj)
                    }
                }
                return objects
                
            } catch {
                throw ORMError.ORMCouldNotMakeObjectError
            }
        }
    }
}
```


这样在调用的时候就很舒服了，以前面的 `Item` 为例：

```swift
private let provider = RxMoyaProvider<ItemAPI>()
private var disposeBag = DisposeBag()

extension ItemAPI {
    static func getNewItems(records:[Record] = [], needCount: Int, completion: [Item] -> Void) {
        disposeBag = DisposeBag()
        provider
            .request(.AddRecords(records, needCount))
            .mapSuccessfulHTTPToObjectArray(Item)
            .subscribe(
                onNext: { items in
                    completion(items)
                }
            )
            .addDisposableTo(disposeBag)
    }
}
```
    
一个 `mapSuccessfulHTTPToObjectArray` 方法，直接将 `JSON` 字符串转换成了 `Item` 对象，并且传入了后面的数据流中，所以在 `onNext` 订阅的时候传入的就是 `[Item]` 数据，并且这个转换过程还是可以复用的，且适用于所有网络请求中 `JSON` 和 `Model` 的转换。爽就一个字，我只说一次。

爽！

## Next

匆匆读了一点 [Emergence](https://github.com/artsy/Emergence) 和 [Eidolon](https://github.com/artsy/eidolon) 的项目源码，没有深入不过已经受益匪浅。通过 bundle 管理 id 和 key 直接解决了我当初纠结已久的『完整项目开源如何优雅地保留 git 记录且保护项目隐私』的问题，还有 `Moya/RxSwift` 和 `Moya/ReactiveCocoa` 这种子模块化处理也在共有模块管理这个问题上给了我一些启发。

真是很喜欢 Artsy 这样的团队，大家都一起做着自己喜欢的事情，还能站着把钱赚了。

所幸的是我也可以这样做自己喜欢的事情了，不过不赚钱。具体状况后面单独开一篇闲扯扯。

碎告。 


***

参考资料：

- [RxSwift](https://github.com/ReactiveX/RxSwift)
- [Moya](https://github.com/Moya/Moya)
- [Argo](https://github.com/thoughtbot/argo)
- [Emergence](https://github.com/artsy/Emergence)
- [Eidolon](https://github.com/artsy/eidolon)
- [Efficient JSON in Swift with Functional Concepts and Generics](https://robots.thoughtbot.com/efficient-json-in-swift-with-functional-concepts-and-generics)
- [Real World JSON Parsing with Swift](https://robots.thoughtbot.com/real-world-json-parsing-with-swift)
- [Parsing Embedded JSON and Arrays in Swift](https://robots.thoughtbot.com/parsing-embedded-json-and-arrays-in-swift)
- [Functional Swift for Dealing with Optional Values](https://robots.thoughtbot.com/functional-swift-for-dealing-with-optional-values)