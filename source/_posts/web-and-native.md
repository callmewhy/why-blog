title: Hybrid App 探索之旅
date: 2015-05-31 13:13:57
tags: Web
categories: 开发笔记
description: 蛋糕看起来太诱人，实在忍不住咬一口。
---

最近公司的项目想尝试在 Native App 中嵌入 Web 界面，以实现一些简单功能的多终端复用。刚好周末花点时间整理一下以前学习 Hybrid App 时的笔记。

## 背景介绍

Hybrid 的原意是混合的， Hybrid App 就是指 Web 和 Native 杂交的产物。愿景是能兼具 Native App 的良好交互体验和 Web App 的跨平台优势。

Hybrid App 和网站有些相似，也是基于 HTML 、 CSS 、 JS 这类 Web 技术进行开发。唯一的区别在于 Hybrid App 是利用各个平台的 WebView 嵌入到原生应用中。

## HTML + JS + CSS

我们可以利用 HTML + JS + CSS 开发一个最简单的 Hybrid App ，直接在 Native App 里放个 WebView ，然后用 WebView 加载存储在本地的网页就可以了。先在项目中新建一个 index.html ，里面的代码如下：

    <html>
        <body>
            <h1>Hello, WHY!</h1>
        </body>
    </html>

然后在某个 `UIViewController` 中加载这个 html 文件：

    class ViewController: UIViewController {

        @IBOutlet weak var webview: UIWebView!
        
        override func viewDidLoad() {
            super.viewDidLoad()

            let path = NSBundle.mainBundle().pathForResource("index", ofType: "html")
            let url = NSURL(fileURLWithPath: path!)
            let req = NSURLRequest(URL: url!)
            webview.loadRequest(req)
        }
    }

加载结果如下：

![](http://ww4.sinaimg.cn/large/61d238c7jw1esnfigb6naj20lu0d1tba.jpg)

OK! 看起来很棒！一切都是 So Easy 啊！但是有种略无力的感觉，比如想做本地缓存，比如获取通讯录中的联系人，这些在 Native 里很常见的功能，在 Web 中并不是很好实现。


## [Cordova](http://cordova.apache.org/)

Cordova 是一套设备 API 的集合，允许移动应用的开发者使用 JavaScript 来访问本地设备的功能，比如摄像头、加速计等等。如今大部分 Hybrid App 是利用 Cordova 进行开发的。Cordova 的前身是大名鼎鼎的 PhoneGap 。 PhoneGap 被 Adobe 收购了，但是剥离了核心代码贡献给 Apache ， Apache 将这个项目命名为 Cordova 。

我们用 Cordova 搞个 Hello World 感受一下，具体的使用方法请参见[官方文档](http://cordova.apache.org/docs/en/5.0.0/index.html)。

在这里推荐使用淘宝镜像 [cnpm](https://npm.taobao.org/) 安装：

    sudo cnpm install -g cordova

安装完成之后创建一个 HelloWorld 项目：

    cordova create hello com.example.hello HelloWorld

添加 iOS 平台：

    cordova platform add ios

这时的项目目录是这样的：

![](http://ww1.sinaimg.cn/large/61d238c7jw1esnji3u54gj20aw0cndh7.jpg)

我们可以用 iOS 模拟器跑一下这个项目。首先安装 ios-sim ：

    npm install -g ios-sim

然后在模拟器上运行：

    cordova emulate ios

运行效果截图：

![](http://ww3.sinaimg.cn/large/61d238c7jw1esnk01di6uj20fn0sfq41.jpg)

看起来还不错。开发的时候我们只需要开发 www 文件夹中的内容即可：

![](http://ww1.sinaimg.cn/large/61d238c7gw1esnor3viccj210x0ihn4n.jpg)

以上便是如何从零创建 Cordova 应用的步骤。如果想在现有项目中使用 Cordova 可以参见这篇 《[Embedding CordovaLib in Your iOS App Project](http://outof.me/embedding-cordovalib-in-your-iosphonegap-app-project/)》

不过至此我们并不能满足，虽然 Cordova 帮我们省了很多工作，比如创建项目、添加平台支持、各种原生接口的封装，但是当真正用于 Hybrid App 开发的时候还是比较乏力，比如导航栏、开关按钮、提示框，这些移动应用中很常见的 UI 控件，在 Web 中并不常见。

## [Ionic](http://ionicframework.com/)

Ionic 是一个基于 Cordova 和 HTML5 的 UI 框架，类似于前端的 [Bootstrap](http://getbootstrap.com/) 和 [Foundation](http://foundation.zurb.com/) ，封装了移动应用中的一些常见控件，例如导航栏、侧滑菜单等等。得益于 HTML5 ，我们可以轻松的调用一些系统的接口。

有了 Ionic 我们可以快速实现一些在移动应用中十分常见的功能。

例如时间选择器：

![](http://ww2.sinaimg.cn/large/61d238c7jw1esnovctp90j20xe0r6agk.jpg)

例如简单的网格布局：

![](http://ww1.sinaimg.cn/large/61d238c7jw1esnoxgc128j20yi0m07c0.jpg)

例如做一个选项卡菜单：

![](http://ww4.sinaimg.cn/large/61d238c7jw1esnozxfm8cj21080pbtga.jpg)


并且还提供了海量 [icon](http://ionicons.com/) ，可以说基本满足了一个移动应用的入门级需求。


## 小结

在我看来， Hybrid App 的优势是开发成本低，在团队人力资源有限又急着上线，或者确实对交互要求不高 (比如只是做个简单的论坛浏览) 的时候， Hybrid App 确实是个不错的选择。

然而 Web App 说到底还是 Web ，它可以用来开发移动应用，但它并不擅长开发移动应用。性能上的瓶颈始终是不可逾越的鸿沟。虽然有 Ionic 和 jQuery Mobile 这样的 UI 框架，省了很多重复的界面开发工作，但是如果仔细对比，在交互体验上和 Native 还是有着比较大的差距。

所以 iOS 和 Android 的程序员们不用紧张，饭碗暂时还没丢。

***

参考文献：

- [What is a Hybrid Mobile App?](http://developer.telerik.com/featured/what-is-a-hybrid-mobile-app/)
- [The Top 7 Hybrid Mobile App Frameworks](http://www.sitepoint.com/top-7-hybrid-mobile-app-frameworks/)
- [Hybrid App开发实战](http://www.infoq.com/cn/articles/hybrid-app-development-combat)
- [手机游戏技术走向之我见: Native vs. HTML5](http://www.infoq.com/cn/presentations/trend-of-mobile-phone-game-technology-native-html5-2d-3d)
