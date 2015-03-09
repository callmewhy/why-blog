title: Localization 的问题和工具
tags: Localization
date: 2015-01-29 13:21:32
categories: 开发笔记
description: 和体力劳动说拜拜。
---

最近公司的一个项目要发布多语言版本，但是因为以前主要是 English 和 Chinese ，其他语言翻译了一部分，未能完全跟上翻译的步伐。如果全部重新翻译，工作量大且浪费了以前的翻译成果，如果手动对比或者用 `git diff` 这样的命令去增量翻译，又无形中增加了开发人员和翻译人员的协作成本。

具体的过程就不多复述了，简单的介绍一下我们目前在 Localization 方面的规范。


## Localizable.strings

iOS 项目中的 `Localizable.strings` 文件务必要规范化，避免重复的无意义翻译，同时在项目模块变动的时候可以及时去除无需使用的字段。

- key 值统一为驼峰规则+下划线 (历史包袱)，首单词为所属模块名，例如相册部分叫做 `Album` ，通用按钮命名为 `Button` 这种。
- 对于不同模块的翻译内容不仅要通过命名区分也要通过备注区分，便于后期维护和跟进。
- 不用翻译的一律放在文件末尾，并且通过注释表明。


良好的示例：

    // 照片
    "Album_GroupName_Camera" = "相机";
    "Album_GroupName_Stream" = "我的照片流";

    // 按钮
    "Button_Add" = "添加";
    "Button_Album" = "相册";


    // DO NOT TRANSLATE
    "Help_Image_Name_Photos" = "help_photos_chs";

## Localization Suite

[Localization Suite](http://www.loc-suite.org/) 是一组 Localization 的工具集合。有利于帮助我们更好更快的完成翻译和校对工作。

下面简单的介绍一下基本的使用方法。

### Localization Manager

打开 Manager 之后，在菜单栏选择 New -> From Xcode ，这样可以从 Xcode 的项目文件中直接读取需要进行翻译的文件。

导入之后应该是这个样子的：

![](http://callmewhy.qiniudn.com/QQ20150129-1.png)

基本的操作按钮在鼠标悬停的时候都有提示。下面演示一下用这个工具的基本用处之一：查找漏翻译的内容。

以 Korean 为例。选中 Korean 然后在右侧的按钮点击 Edit 按钮。

这时会打开我们的第二个工具： Localizer：

![](http://callmewhy.qiniudn.com/QQ20150129-2.png)

### Localizer

Localizer 是用来进行翻译的，我们可以在左侧设定好筛选条件然后在右侧面板就能看到未翻译的内容：

![](http://callmewhy.qiniudn.com/QQ20150129-3.png)

由于这个工具把 value 相同认为未翻译，而实际上可能确实存在一些待翻译的内容，比如目前图片资源没跟上，还是先显示 ChatGame 这样的。可以统一把这些放在末尾，并且备注写明不需翻译，这样在对照的时候会更清晰一些：

![](http://callmewhy.qiniudn.com/QQ20150129-4.png)

先写这么多。


## Kaleidoscope

Mac 下一个很好用的文件对比工具，就是有点贵 (几百RMB)，不过确实很方便，可以试用15天再决定是否购买。