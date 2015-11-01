title: Yoapp 小记 - 简单方便的扩展们
date: 2015-10-21 12:03:32
tags: Swift
categories: 开发笔记
description: 用扩展给你的代码们装备上铠甲吧！
---

### SppedUp

我们团队每隔两周会有一次 SppedUp ，两天时间开发一款全新的应用。从前天凌晨到昨天深夜整整48小时，第一次 SpeedUp 理论上应该已经结束，但是事实上并没有想象中那么理想，一直拖到今天还在继续，而且还存在诸如『登录时键盘遮挡输入框』这种低级错误。今天花了一天时间完善了细节，并且整理一下代码，整理一些 Swift 中的代码优化细节。

### UIStoryboardSegue

作为一名纯粹的 IB 党，少不了和 `UIStoryboardSegue` 打交道。主要的使用场景是以下两个方法：

- `performSegueWithIdentifier(_:sender:)`
- `prepareForSegue(_:sender:)`

一开始我们是这么写的：

```swift
performSegueWithIdentifier("show_detail", sender: nil)

```
    
后来发觉这种 `hard code` 并不科学，于是用 struct 封装了一下定义成常量，这样合理多了：

```swift
struct SegueIdentifier {
    static let showDetail = "show_detail"
}
performSegueWithIdentifier(SegueIdentifier.showDetail, sender: dataSource[indexPath.row])
```
    
再后来感觉 `prepareForSegue` 里不能 `switch` 不开心啊！于是改成用 `enum` 封装：

```swift
enum SegueIdentifier: String {
        case Detail = "show_detail"
    }

    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        guard
            let identifier = segue.identifier,
            let segueIdentifier = SegueIdentifier(rawValue: identifier) else {
                fatalError("INVALID SEGUE IDENTIFIER \(segue.identifier)")
        }

        switch segueIdentifier {
        case .Detail:
            // DO SOMETHING
        }
    }
```
    

但是在调用 `performSegue` 的时候还是有点难受，不能直接传 `enum` 进去，必须要 `rawValue` 才行：

```swift
performSegueWithIdentifier(SegueIdentifier.Detail.rawValue, sender: nil)
```
    


后来昨天看 [Swift in Practice](https://developer.apple.com/videos/play/wwdc2015-411/) 的时候发现，其实可以利用 `protocol extension` 把通用的部分封装起来：

```swift
protocol SegueHandlerType {
    typealias SegueIdentifier: RawRepresentable
}

extension SegueHandlerType where Self: UIViewController, SegueIdentifier.RawValue == String {
    func performSegueWithIdentifier(identifier: SegueIdentifier, sender: AnyObject?) {
        performSegueWithIdentifier(identifier.rawValue, sender: sender)
    }
    
    func segueIdentifierForSegue(segue: UIStoryboardSegue) -> SegueIdentifier {
        guard
            let identifier = segue.identifier,
            let segueIdentifier = SegueIdentifier(rawValue: identifier) else {
                fatalError("INVALID SEGUE IDENTIFIER \(segue.identifier)")
        }
        return segueIdentifier
    }
}
```
    

这样只要 `UIViewController` 实现 `SegueHandlerType` 协议便可获得封装好的方法，比如更好用的 `performSegueWithIdentifier` ：

```swift
performSegueWithIdentifier(.LoginViewController, sender: nil)
```


而且 `prepareForSegue` 可以直接通过 `switch segueIdentifierForSegue(segue)` 对不同 `segue` 做处理：

```swift
extension RankingViewController: SegueHandlerType {
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        switch segueIdentifierForSegue(segue) {
        case .Detail:
            guard
                let controller = segue.destinationViewController as? DetailViewController,
                let cell = sender as? RankingCell,
                let app = cell.appDetail else {
                    return
            }
            controller.appDetail = app
        }
    }
}
```


### dequeueReusableCellWithIdentifier

在写 `UITableView` 的时候总会有一段很难记但是又不得不记的代码：

```swift
let cell = tableView.dequeueReusableCellWithIdentifier("XxxxxCell") as! XxxxxCell
```

注意这里又出现了 `"XxxxxCell"` 这样的 `hard code` ，我们可以用 `Class` 的名称作为它的 `identifier` ，这样只要能获取到 `className` 就行了。在 Swift 里可以这样获取 `className` ：

```swift
extension NSObject {
    static var className: String {
        get {
            return self.description().componentsSeparatedByString(".").last!
        }
    }
}
```

于是乎我们的代码变成了这样：

```swift
let cell = tableView.dequeueReusableCellWithIdentifier(XxxxxCell.className) as! XxxxxCell
```

接下来我们可以给 `UITableView` 添加个扩展，封装一下上面的代码：

```swift
extension UITableView {
    func dequeueCell<T: UITableViewCell>(cell: T.Type) -> T {
        return dequeueReusableCellWithIdentifier(T.className) as! T
    }
}
```
    
这样就可以这样生成 `cell` 了：

```swift
let cell = tableView.dequeueCell(RankingCell)
```

### AVObject

由于在使用 `LeanCloud` 做后台服务，所以不可避免要和 `AVObject` 打交道，`AVObject` 基本和 `NSDictionary` 接口一致，所以做了如下封装：


```swift
import Foundation
import AVOSCloud

extension AVObject {
    func stringForKey(key: String) -> String? {
        return self.objectForKey(key) as? String
    }
    func doubleForKey(key: String) -> Double? {
        let o = self.objectForKey(key)
        if let r = o as? Double {
            return r
        } else if let r = o as? String {
            return Double(r)
        }
        return nil
    }
    ...
}
```

### setToRouded

将图片设为圆角是一个比较常见的操作，可以通过 `extension` 简单封装一下：

```swift
extension UIView {
    func setToRounded(radius: CGFloat? = nil) {
        let r = radius ?? min(self.frame.size.width, self.frame.size.height) / 2
        self.layer.cornerRadius = r
        self.clipsToBounds = true
    }
}
```
    
如果指定半径就设置圆角半径，如果没有值则取高和宽的最小值，实现『半圆』效果。


### QUESTION

这里有个问题没太理解。请看下面两段代码：

```swift
import UIKit

extension UITableView {
    static func dequeueCell1<T: UITableViewCell>(name: String, cell: T.Type) {
        print(cell)
    }
    
    static func dequeueCell2<T: UITableViewCell>(cell: T.Type) {
        print(cell)
    }
}
```
    
唯一的区别就是参数数目不同，但是调用的时候：

```swift
UITableView.dequeueCell1("", cell: UITableViewCell.self)
UITableView.dequeueCell2(UITableViewCell)
```

两个参数的需要 `.self` 而另一个则不需要。=。=没太搞懂这是哪出，还望赐教！

***

参考文献：

- [Swift in Practice](https://developer.apple.com/videos/play/wwdc2015-411/)