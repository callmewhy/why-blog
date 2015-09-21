title: 使用 Yeoman + AngularJS + Gulp 搭建 Web 应用
date: 2015-09-18 09:28:43
tags: Yeoman
categories: 开发笔记
description: 一个戴帽子的老男人，请带我飞！
---

最近在做一个简单的资源管理系统，以前都是 PHP+jQuery+Bootstrap 的组合，这一次想尝试一些没接触过的东西。在这里简单的总结一下在开发过程中用到的框架和工具。

总结的比较简单，新手上路，多多指教！各位如果感兴趣可以进入各个项目的主页深入了解。

另外，由于我曾经是一名 iOS 程序员（啊现在也是），在文章中可能会情不自禁地乱入一些 iOS 开发相关的东西，可以直接无视我。。。


## [npm](https://www.npmjs.com/)

npm 是 NodeJS 的包管理工具，建议使用 [n](https://github.com/tj/n) 管理 NodeJS 版本，无痛体验 NodeJS 4.0 和 0.12 版本。

同时考虑到国内的网络环境，可以考虑使用 taobao 的 [npm 镜像](https://npm.taobao.org/)。

虽然 homebrew 是神器，但是还是不推荐使用 homebrew 安装 npm ，原因可见：[Fixing npm On Mac OS X for Homebrew Users](https://gist.github.com/DanHerbert/9520689)。用 npm 去管理 npm 再好不过了。

## [Yeoman](http://yeoman.io/)

在做开发的时候，各种文件的归类和摆放是一个很常见的问题，例如业务逻辑文件放在 app 文件夹，模板文件放在 template 文件夹，每次新建项目都要做一些重复的工作。

Yeoman 是一款脚手架工具，可以轻松地为你搭建一个项目的大体框架。可以通过 npm 安装：

    npm install -g yo bower grunt-cli gulp

安装完成之后，输入 `yo` 就可以开始搭建一个项目啦。不过似乎用了 taobao 的 npm 镜像之后，用 yo 安装 generator 会出现一些问题，可以直接用 npm 安装，例如安装 webapp 模板：

    npm install -g generator-webapp

接下来可以试着用 `yo webapp` 创建一个新项目，生成的目录结构是这样的：

    ├── .bowerrc
    ...
    ├── .editorconfig
    ├── app
        ...
        ├── fonts
        ├── images
        ├── scripts
        │   └── main.js
        └── styles
            └── main.css
    ├── bower.json
    └── test

可以看到，不仅是 styles 和 scripts 这样的目录结构，包括 .editorconfig 和 .bowerrc 也是自动生成了。

在 iOS 里也有类似的脚手架：[liftoff](https://github.com/thoughtbot/liftoff)，不过似乎 Xcode7 遇到一些问题，等日后摸索一番再来汇报。


## [Bower](http://bower.io/)

两年前有幸看到了一系列很有意思的文章：[30 天学习 30 种新技术系列](http://segmentfault.com/a/1190000000349384)，当时跟着玩了一圈感觉十分好玩。就像是《七天七语言》和《七天七并发模型》一样，世界那么大，一起去看看。

第一次接触 Bower 就是缘于这其中的第一篇：[Day 1: Bower —— 管理你的客户端依赖关系](http://segmentfault.com/a/1190000000349555)。

使用方法很简单，首先通过 npm 安装 bower (前面安装 yo 的时候其实已经安装了)：

    npm install -g bower

然后就是无脑的 install 了，比如我要装个 Bootstrap ：

    bower install --save bootstrap

嗯就是这么简单。

在 iOS 中类似的依赖管理工具有 [Cocoapods](https://cocoapods.org/) 和 [Carthage](https://github.com/Carthage/Carthage) 。

## [Angular](https://angularjs.org/)

Angular 的种种好处就不多说了，双向绑定、依赖注入、指令系统、模板、路由，各种各种。再配合上 [angular-ui](http://angular-ui.github.io/)，写起项目来十分酸爽。

简单的演示一个小例子，通过 [LeanCloud](https://leancloud.cn/) 的 JS SDK 加载并显示数据：

![](http://ww3.sinaimg.cn/large/61d238c7gw1ew6t3ecbaaj21kw0gzwjs.jpg)

首先定义一个 Provider ，添加获取数据的方法：

    function getItems(className, completion) {
        var query = new AV.Query(AV.Object.extend('Article'));
        query.find({
            success: function(results) {
                var r = JSON.parse(JSON.stringify(results));
                completion(r);
            }
        })
    }

然后定义界面：

    <div class="text-center">
        <div ui-grid="article.gridOptions" ui-grid-edit class="grid"></div>
    </div>

再通过 Controller 加载数据：

    /** @ngInject */
    function ArticleController($scope, AVOS) {
        var vm = this;

        vm.gridOptions = {
            data: []
        };

        AVOS.getItems('Article', function(result) {
            $scope.$apply(function(){
                vm.gridOptions.data = result.map(function(item) {
                    return {
                        'index': item.index,
                        'content': item.content
                    };
                });
            });
        });
    }

和以前用 JS 直接操作 DOM 相比，ViewModel 的逻辑更加友好，做起数据加载刷新的时候尤其顺手。

## [Gulp](http://gulpjs.com/)

前端项目经常需要进行许多次的类似任务，例如编译 Less ，压缩代码等等。这样单调而重复的任务，我们可以通过工具来完成。 gulp 就是一个不错的选择。

gulp 是一个基于流的构建工具，个人感觉比 grunt 更简单清晰，也解决了 grunt 插件职责不明的问题。

什么是『基于流』呢？可以看下一个简单示例：

    gulp.src('client/templates/*.jade')
        .pipe(jade())
        .pipe(minify())
        .pipe(gulp.dest('build/minified_templates'));

可以看到这样一个链式调用的结构，除了开头和结尾，每一个命令的输出都是下一个命令的输入。先把任务分解成一个一个的小模块，然后再各取所需，组装起来。

具体的教程可以参考官方文档，只需要了解 `gulp.src` 、 `gulp.dest` 、 `gulp.task` 、 `gulp.watch` 这几个基础 API 即可。

## [DaoCloud ](https://daocloud.io/)

为了避免每次都重复的部署新服务器，我们可以把项目通过 Docker 打包成应用，然后直接部署。DaoCloud 提供了一个容器云平台，可以直接部署 Docker 应用。

在我的新项目 Lastone 里，只要在根目录加上 Dockerfile 即可：

    FROM ubuntu:14.04
    MAINTAINER callmewhy whywanghai@gmail.com

    RUN apt-get update
    RUN apt-get install -y nginx

    ADD ./www/lastone/dist /usr/share/nginx/html/lastone

    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]

Dockerfile 的命令可以阅读 [Dockerfile reference](https://docs.docker.com/reference/builder/)。

Daocloud 默认是每次在代码库中 push 一个 tag 就会自动构建一次，所以基本不用操什么心，而且有一个永久免费的份额，基本够玩耍使用了。

## Next

以上就是这次折腾的一些笔记啦。下一个小项目准备试一下 [React](https://facebook.github.io/react/)。

最近准备做一些小项目，到处玩一玩，如果大家有什么有趣的东西欢迎分享给我。（纯洁的微笑：]