title: 三款开源工具让你的演示脱颖而出
tags: Tool
date: 2014-07-02 21:43:39
categories: 翻译笔记
description: 是时候和屁屁踢说拜拜啦。
---


不论是在商业圈还是在学术界，演示都是生活中不可或缺的一部分。一般来说，做一个演示就意味着做几张幻灯片，微软的PPT，Apple的Keynote，LibreOffice的Impress都是常见的选择。撇开前两个应用的闭源性质不谈，这些应用的问题在于如果你要查看你准备的内容，你就必须在演示的电脑上安装对应的软件。如果想在线展示的话，你也可以尝试用谷歌的Drive或者类似的其他服务碰碰运气，不过能否成功就看人品了。

这些年，用来创建幻灯片的演示框架数目激增，这些框架充分发挥了HTML5、CSS3和JavaScript的优势，只需要一个普通的浏览器就可以创建属于你的幻灯片。再也不需要担心文件的兼容性，不需要担心某一天文档会被某个特殊的网络服务加锁。因为这些幻灯片框架都是开源的，所以我们可以随心所欲地对这些框架进行一些自定义修改。不过说句实在话，和用PowerPoint、Keynote、Impress相比，写HTML5、CSS3和JavaScript的代码要略微复杂一点。


接下来介绍一下三款开源工具，可以让你的演示脱颖而出。


## Impress.js

[Impress.js](https://bartaz.github.io/impress.js/#/bored)是Bartek Szopka受到[Prezi](http://prezi.com/)的启发开发的一个演示工具框架，利用CSS3提供优于传统幻灯片的演示体验。 演示者可以用impress.js轻松实现各种旋转、滑动、放缩特效，足以让观众惊叹。impress.js依赖于传统的Web技术（HTML+CSS+JavaScript），意味着不会将用户捆绑到某种特定的软件或者网络服务上。因为它是遵循[MIT](http://mit-license.org/)和[GPLv2+](https://www.gnu.org/licenses/gpl-2.0.html)协议的，所以你可以对impress.js的源码做任意修改。impress.js充分利用了最新的Web技术，所以需要一个比较流行的网页浏览器，最近版本的Chrome、Firefox、Safari、IE基本就能满足要求。创建一个impress.js应用并不是很容易，即使对于有一定HTML和CSS基础的人来说也是如此。

impress.js中，基本的标记很容易懂，但是想做出很复杂的演示，需要深入思考和仔细规划。在impress.js里没有什么默认主题，需要自己设计展示效果、演示流程、幻灯片之间的切换方式以及每张幻灯片的相对布局。从零开始制作一个演示文档需要做很多工作，但是事实上有很多[样例](https://github.com/bartaz/impress.js/wiki/Examples-and-demos)可以提供灵感和指导，网上也有很多[教程](https://github.com/bartaz/impress.js/wiki/impress.js-tutorials-and-other-learning-resources)，深入讲解impress.js的使用。

如果你觉得创建一个impress.js的展示对你来说很复杂，那可以使用一些更容易使用的小工具。



## Hovercraft

[Hovercraft](https://github.com/regebro/hovercraft)简化了创建impress.js文档的过程，使用[reStructedText](http://docutils.sourceforge.net/rst.html)创建演示文档。和用HTML制作幻灯片不同，Hovercraft可以让你更加专注于写作。你可以任意改动元素而不用担心标记语言的标签封闭问题。

举个例子，我想创建了一张幻灯片，比上一张幻灯片大了五倍并且旋转了90度。那么在Hovercraft里，只需要两行代码就能完成这些工作：

```

:data-scale: 5
:data-rotate: 90

Heading
=======
* Bullet Point 1
* Bullet Point 2

```

使用Hovercraft极大的简化了impress.js的使用。Hovercraft支持四种放置幻灯片的方式，如果没有设置的话，会使用默认的切换方式，也就是向左飞出切换到下一张。如果你想让你的幻灯片更酷炫一点，你可以使用相对布局，幻灯片会基于你自定义的偏移量进行切换。如果在中间插入了一张幻灯片，接下来的其他幻灯片也会依次自动适应调整坐标。如果你想要控制其中的细节，你可以使用绝对布局，提前定义好每个幻灯片的坐标并用[SVG](http://www.w3schools.com/svg/)制定好路线。


Hovercraft的文档评价SVG布局“用起来有点繁琐”，不过它可以让你更加精确的控制幻灯片的每一个细节，让你的演示更加出彩。另外，如果你想在你的演示中插入代码，那也没有问题，Hovercraft支持代码语法高亮，并且它还提供一个专门给演讲者看的屏幕，可以显示笔记，并且还有计时功能。当你写好了一份文档，一条简单的命令就可以把rst文件转换成HTML演示文稿：

```
hovercraft [markupfile] [output directory]
```

虽然Hovercraft有很多优点，但是它依然需要使用者有一定的CSS常识。默认的主题十分的朴实，如果你想要你的演示出彩的话，还是要花一些功夫的。给幻灯片加上CSS并非难事，但是和PPT中点击就能选主题相比，还是显得复杂了一些。

如果想深入学习，你可以阅读[Hovercraft的文档](http://hovercraft.readthedocs.org/en/1.0/)。Hovercraft的作者是Lennart Regebro，遵循[CC0 1.0通用协议](https://creativecommons.org/publicdomain/zero/1.0/deed.en)。



## Strut

如果你想要一个工具，让你的工作像传统的幻灯片制作一样简单，那么[Strut](http://www.strut.io/)是一个不错的选择。Strut是一个基于网络的应用，提供了幻灯片的分类和编辑工具。图形化的界面让你轻轻松松的添加文字、图片、视屏和网页。你也可以一次性改变所有幻灯片的前景色和背景色，也可以一张一张的修改。

Strut支持[Markdown](http://daringfireball.net/projects/markdown/)的语法，而且对于有一定基础的用户，可以自定义CSS样式。当你设计好了你的幻灯片，你可以设置旋转角度和缩放比例等参数，切换不同的预览方式和页面布局。除了impress.js，Strut也可以创建基于[bespoke.js](https://github.com/markdalgleish/bespoke.js)框架的演示文档。

Strut很不错，但是依旧有一些缺点。有时候会遇到一些bug，并且这个项目的待办事项有点多，虽然都不是什么大问题。 

该项目遵循“早发布，常发布”的准则，愿意把这个项目做得更好的人可以去[Github](https://github.com/tantaman/Strut)做贡献。你可以在官网的[在线编辑器](http://strut.io/editor/index.html)试一试，或者直接去[Github](https://github.com/tantaman/Strut)下载它的源码包在本地运行。如果想在本地运行Strut，需要有[NodeJS](http://nodejs.org/)的npm工具和[Grunt](http://gruntjs.com/)来安装依赖项目。


Strut的创始人是Matthew Crinklaw-Vogt，并且遵循[Affero通用协议](https://www.gnu.org/licenses/agpl-3.0.html)。





***

原文地址：

- [3 open source tools make your presentations pop](http://opensource.com/life/14/7/3-open-source-tools-make-your-presentations-pop)