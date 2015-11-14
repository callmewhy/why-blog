title: 使用 Babel + React + Webpack 搭建 Web 应用
date: 2015-11-09 19:32:22
tags: React
categories: 开发笔记
description: 在我写完这篇文章的时候，文章里介绍的工具可能已经被淘汰了。
---

同志们，我又来闲扯前端了。掐指一算不务正业地折腾前端已经有将近一个月的时间了，这段时间零零散散做了三四个简单的小项目，尝试了 Grunt Gulp Webpack 之类的项目构建工具，折腾了 AngularJS ReactJS VueJS 之类的前端框架，对比了 Jade Mustache Ejs 之类的模板引擎，把玩了 Sass Less Stylus 之类的预处理器，还借助着 Babel 体验了一把 ES6+ ，心中一直有这样一种感觉：还是 iOS 好啊！（别问我为什么试这么多东西，挑一个用不就行了吗？是啊，但是不试一遍怎么知道挑哪个好Q_Q

简单整理一下最新的一个项目：一个用于微信传播的 HTML5 小活动，源码在活动上线之后会开源到 Github 上。

## [Yeoman](https://github.com/yeoman/yo)

Yeoman 在[前面的文章](http://blog.callmewhy.com/2015/09/18/use-yeoman-with-angular-and-gulp/#Yeoman)中介绍过，是一款脚手架工具。新项目使用的是 [generator-gulp-webapp](https://github.com/yeoman/generator-gulp-webapp) 这个模板，在 [recipes](https://github.com/yeoman/generator-gulp-webapp/tree/master/docs/recipes) 里介绍了一些常用的扩展方案，主要是 gulp 相关的一些配置。

## [Babel](http://babeljs.io/)

Babel 是一款 JavaScript 转译器，它能够将 ES6+ 的代码转译为主流浏览器支持的 ES5 代码，而且还可以通过插件加载 JSX 的语法。

可以在 Babel 的 [repl](http://babeljs.io/repl/) 里体验一下它的神奇之处。输入一段 ES6 的内容：

```js
var add = (a, b) => {
  console.log('Hello')
}

add(1, 2)
```

转换之后的结果是：

```js
'use strict';

var add = function add(a, b) {
  console.log('Hello');
};

add(1, 2);
```

这样就可以无忧无虑的直接使用 ES6+ 开发了 Web 项目了。

## Webpack

Webpack 是一款模块打包工具，在 Webpack 当中，所有的资源都被当作是模块，各种资源通过各种 loader 加载，比如 babel 就提供了 [babel-loader](https://github.com/babel/babel-loader) 插件，用于在 Webpack 中加载 babel 编译 js 文件。

在项目里用 npm 安装好依赖，添加一个 Webpack 任务，设置 JSX 和 Babel 的相关配置，就可以在项目里使用 React 和 Babel 了：

```js
gulp.task('webpack', cb => {
  webpack({
    entry: './app/scripts/app.jsx',
    output: {
      path: path.resolve(__dirname, '.tmp/scripts/'),
    },
    resolve: {
      extensions: ['', '.js', '.jsx']
    },
    module: {
      loaders: [{
        test: /\.jsx?$/,
        exclude: /(node_modules|bower_components)/,
        loader: 'babel',
        query: {
          presets: ['react', 'es2015', 'stage-0']
        }
      }]
    }
  }, (err, stats) => {
    if (err) {
      throw new gutil.PluginError('webpack', err);
    } else {
      console.log(stats.toString());
    }
    cb();
  });
});
```

## [React](https://github.com/facebook/react)

React 是 Facebook 开源的一款前端框架，专注于 View 层，提供数据驱动、可组合的视图组件。结合 JSX 语法写出来大概是这个样子的：

```js
var CommentBox = React.createClass({
  render: function() {
    return (
      <div className="commentBox">
        Hello, world! I am a CommentBox.
      </div>
    );
  }
});
React.render(
  <CommentBox />,
  document.getElementById('content')
);
```

从语法角度来讲，我是不太能接受 React 这种设计的。渲染函数包含了大量业务逻辑，看着像是程序片断而不能直观提供视觉呈现。虽然强调了『模块化』的概念，可以通过封装各种单位模块然后逐渐拼接成完整项目，但是，就是感觉不够优雅。

不过学习的过程中了解了不少有趣的思想，比如单向数据流、Virtual DOM，还有通过 props 和 state 强调『状态管理』，减少可变因素，这些都让我受益匪浅。

React 一般会配合 [FLUX](http://facebook.github.io/flux/) 使用，官方提供了 [Todo List](http://facebook.github.io/flux/docs/todo-list.html) 的例子可以作为入门参考。FLUX 的结构大概是这样的：

![](http://ww1.sinaimg.cn/large/61d238c7jw1exvtvfsmfcj21040joadn.jpg)

看起来很美好，不过只是照着做了边教程，还没在项目里试过，等到后面有项目可以体验一下。

## Next

以上内容相当于最近的个人学习小结，匆匆忙忙一笔带过，只是做个记录。

感觉前端是一个很有趣的领域，每个人都在尝试用各种姿势造着各种轮子，接下来的时间不再折腾别人的轮子了，花点时间补补基础知识。

然而。

![](http://ww4.sinaimg.cn/large/61d238c7gw1ettf9re22uj20fu08owf3.jpg)

教练，我想写 Swift 。

***

参考资料：

- [Facebook：MVC 不适合大规模应用，改用 Flux](http://www.infoq.com/cn/news/2014/05/facebook-mvc-flux)
- [React 虚拟 DOM 浅析](http://www.alloyteam.com/2015/10/react-virtual-analysis-of-the-dom/)
