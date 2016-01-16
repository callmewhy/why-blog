title: 使用NodeJS和AngularJS创建一个TODO的单页网站
tags: AngularJS
date: 2014-07-17 15:11:00
categories: 翻译笔记
description: Mongo+Express+Angular+Node的超级大乱炖。
---

今天我们来用MEAN(Mongo,Express,Angular,Node)来搭建一个简单的TODO列表的网站应用。

首先简单的列一下任务列表：

- 创建和完成TODO列表的主页面
- 通过Mongoose把TODO列表存到MongoDB数据库
- 使用Express框架
- 创建RESTful的接口
- 使用Angular作为前端框架并连接API

这个项目本身很简单，是为新手准备的中级进阶教程，其中的很多概念可以应用到其他项目中。通过这个项目我们可以学习到如何使用NodeJS作为接口然后使用Angular作为前端。如何把各个部分整合起来是一件比较头疼的事情，希望这个教程可以给你答案。

准备好了吗少年们，我们要起航了！

![](http://callmewhy.qiniudn.com/todoaholic.png)


## 基础安装

### 文件结构
我们让文件结构尽量简单，把大多数的代码放在NodeJS项目中的server.js文件中。在大型项目里，我们可以把代码按照功能细分，让各个部分各司其职。比如[Mean.io](http://mean.io/)就是这样一个样例项目，很好的演示了按照功能分工的文件结构。接下来我们创建一个简单的项目，文件结构如下：

```
- public        <!-- holds all our files for our frontend angular application -->
    - core.js       <!-- all angular code for our app -->
    - index.html    <!-- main view -->
- package.json  <!-- npm configuration to install dependencies/modules -->
- server.js     <!-- Node configuration -->
```

### 安装模块
在NodeJS里，`package.json`文件包含了应用相关的设置，NodeJS的包管理工具会根据这个文件安装我们需要使用的依赖模块。在我们的项目里，我们将使用[Express](http://expressjs.com/)和[Mongoose](http://mongoosejs.com/)这两个依赖项。

`package.json` 如下：

```json
{
    "name"         : "node-todo",
    "version"      : "0.0.0",
    "description"  : "Simple todo application.",
    "main"         : "server.js",
    "author"       : "Scotch",
    "dependencies" :
    {
        "express"    : "~3.4.4",
        "mongoose"   : "~3.6.2"
    }
}
```

现在运行`npm install`，npm会自动安装依赖项目。


### NodeJS设置
在我们的`package.json`中，我们设置项目主文件为`server.js`，也就是说整个项目通过这个js文件启动。

在这个文件里我们可以完成如下工作：

- 设置项目
- 连接数据库
- 创建Mongoose模型
- 定义RESTful API的路径
- 定义Angular前端应用的路径
- 设置监听的端口号

现在，我们只需要初始化其中的Express和MongoDB数据库并且监听端口：

```js
var express  = require('express');
var app      = express();
var mongoose = require('mongoose');

mongoose.connect('mongodb://node:node@mongo.onmodulus.net:27017/uwO3mypu');

app.configure(function() {
    // set the static files location /public/img will be /img for users
    app.use(express.static(__dirname + '/public'));
    // log every request to the console
    app.use(express.logger('dev'));
    // pull information from html in POST
    app.use(express.bodyParser());                  
});

app.listen(8080);
console.log("App listening on port 8080");
```

这样，我们现在就有了一个HTTP的服务器，在`app.configure`里，我们使用express模块来添加功能模块。


### 数据库安装

我们在[Modulus.io](https://modulus.io/)建立远程数据库，Modulus为新注册的用户提供15美元的试用资金，方便我们在云服务器上测试项目。

Modulu也提供了一个数据库的URL地址，你只需要使用`mongoose.connet`连接数据库。就是这么简单。


### 运行应用
现在我们设置好了`package.json`和`server.js`文件，接下来我们可以通过如下命令运行服务器，看看效果如何：

```
node sercer.js
```
现在我们的服务器就启动了，并且监听8080端口。现在打开浏览器你什么都看不见，但是它确实成功启动了！

注意，运行会提示如下警告：

```
connect.multipart() will be removed in connect 3.0
visit https://github.com/senchalabs/connect/wiki/Connect-3.0 for alternatives
```

如果在3.0中使用`connect.bodyParser()`会在程序启动时收到弃用警告,但并不影响程序的正常工作。由于Express使用了connect中间件，参照官网给的解决方案,用`json`和`urlencoded`替换`bodyParser`即可。

找到如下的内容：

```js
app.use(express.bodyParser());
```
替换为：

```js
app.use(express.urlencoded());
app.use(express.json());
```
这样再运行就没有警告了。

另外，默认情况下NodeJS在服务器启动之后不会检测到文件修改，所以当你改了文件之后必须重启服务器。你可以装一个nodemon的模块：

```js
nom install -g nodemon
```

这样使用nodemon的命令就可以启动服务器了：

```js
nodemon server.js
```


## 应用流程
现在整个项目的概要基本有了个雏形，接下来我们来看看各个部分是如何协同工作的。在这个项目里有很多不同的想法和技术，我们很容易把它们结合在一起。在下面的流程图中，可以看出任务的划分以及各部分是如何合作的：

![](http://callmewhy.qiniudn.com/mean.jpg)

Angular在前端独当一面，它通过NodeJS的API获取所有它想要的数据。NodeJS访问数据库并且基于RESTful给Angular返回JSON格式的数据。

这样我们就可以把前端应用和真实API分离。如果你想要扩展API只需要添加更多的路由和函数即可，不影响前端AngularJS的正常使用。这样我们可以基于不同平台开发不同应用，只需要访问API就可以了。

## 创建接口
我们的目标是完成一个TODO的网站，在写前端应用之前，我们需要先创建RESTful接口，提供获取TODO、创建TODO、完成TODO和删除TOTO的功能，并且会返回JSON格式的数据。


### TODO模型
我们尽量简单的建立TODO模型，在config和listen之间，我们添加模型相关代码：

```js
// ----- define model
var Todo = mongoose.model('Todo', {
    text : String
});
```

没错就是这么简单，只需要TODO的任务内容就够了。MongoDB将会自动为每个TODO对象生成`_id`属性。


### RESTful路由
我们用Express来实现API访问：

```js
// ----- define routes
// get all todos
app.get('/api/todos', function(req, res) {
    // use mongoose to get all todos in the database
    Todo.find(function(err, todos) {
        // if there is an error retrieving, send the error. nothing after res.send(err) will execute
        if (err) {
            res.send(err);
        }
        res.json(todos); // return all todos in JSON format
    });
});

// create Todo and send back all todos after creation
app.post('/api/todos', function(req, res) {
    // create a Todo, information comes from AJAX request from Angular
    Todo.create({
        text : req.body.text,
        done : false
    }, function(err, todo) {
        if (err) {
            res.send(err);
        }
        // get and return all the todos after you create another
        Todo.find(function(err, todos) {
            if (err) {
                res.send(err);
            }
            res.json(todos);
        });
    });

});

// delete a todo
app.delete('/api/todos/:todo_id', function(req, res) {
    Todo.remove({
        _id : req.params.todo_id
    }, function(err, todo) {
        if (err)
            res.send(err);
        // get and return all the todos after you create another
        Todo.find(function(err, todos) {
            if (err) {
                res.send(err);
            }
            res.json(todos);
        });
    });
});
```

基于上面的路由，我们可以通过这个简单的表格来展示一下前端应用如何向服务器端请求数据：

|HTTP动词|URL地址|描述|
|-------|-------|---|
|GET|/api/todos|Get all of the todos|
|POST|/api/todos|Create a single todo|
|DELETE|/api/todos/:todo_id|Delete a single todo|


在每个API里，我们使用Mongoose来帮助我们和数据库交互。在前面的代码中，我们使用`var Todo = mongoose.model`创建了模型，并且可以通过它来实现`find`、`create`和`remove`功能。当然还有很多其他功能，建议查看[官方文档](http://mongoosejs.com/docs/guide.html)进一步学习。

OK，那么我们的API就搞定了！如果你开启项目，你可以在`localhost:8080/api/todos/`里获取所有的todos，不过现在里面什么也没有因为我们还没有往里添加数据。



## 前端框架

现在我们搭建了NodeJS应用，设置了数据库，生成了API路径，并且还开了服务器，已经做了不少工作，目前完成的工作可以当做一个单独的项目，为应用提供API接口。

接下来我们看看如何使用AngularJS搭建前端框架。

我们先调用一下刚刚创建的API接口，用一个我这个月刚学会的谚语来说：[吃自家的狗粮](http://zh.wikipedia.org/wiki/Eating_your_own_dog_food)，中文译为自产自销可能比较合适。我们把自己当做自己的首批用户，调用我们自己开发的API。为了让项目尽量简单，我们只需要`index.html`和`core.js`这两个文件。


### 定义前端路由

我们刚刚定义了API的URL路径：`/api/todos`，但是前端呢？如何让服务器在首页显示index.html页面？

我们在`server.js`中加一条路径：

```js
// get the index.html
app.get('*', function(req, res) {
    res.sendfile('./public/index.html'); // load the single view file (angular will handle the page changes on the front-end)
});
```

现在再访问`localhost:8080`就可以访问index.html了。

### 设置AngularJS

我们先来看一看AngularJS的安装，我们需要创建一个模块，一个控制器，然后定义处理TODO的函数。修改后的`core.js`如下：

```js
var scotchTodo = angular.module('scotchTodo', []);

function mainController($scope, $http) {
    $scope.formData = {};
    // when landing on the page, get all todos and show them
    $http.get('/api/todos')
        .success(function (data) {
            $scope.todos = data;
            console.log(data);
        })
        .error(function (data) {
            console.log('Error: ' + data);
        });
    // when submitting the add form, send the text to the node API
    $scope.createTodo = function () {
        $http.post('/api/todos', $scope.formData)
            .success(function (data) {
                $scope.formData = {};
                $scope.todos = data;
                console.log(data);
            })
            .error(function (data) {
                console.log('Error: ' + data);
            });
    };
    // delete a todo after checking it
    $scope.deleteTodo = function (id) {
        $http.delete('/api/todos/' + id)
            .success(function (data) {
                $scope.todos = data;
                console.log(data);
            })
            .error(function (data) {
                console.log('Error: ' + data);
            });
    };
}
```

我们创建了一个模块（scotchApp）和一个控制器（mainController）。

我们也创建了获取TODO、创建TODO、删除TODO的函数，这些都会访问我们前面创建的API接口。在页面加载的时候，我们通过GET访问了`/api/todos`并且将从服务器获取到的JSON数据绑定到了`$scope.todos`上，接下来就可以在视图中遍历生成TODO列表。

### 网站页面开发

我们依旧采用尽量简单的方法开发网站页面。主要分为以下几个步骤：

- 注册Angular的模块和控制器
- 获取所有的TODO列表并且初始化
- 遍历所有的TODO列表
- 有一个表单创建TODO
- 当TODO被选中时执行删除操作


`index.html`中源码如下：

```html
<!-- index.html -->
<!doctype html>
<!-- ASSIGN OUR ANGULAR MODULE -->
<html ng-app="scotchTodo">
<head>
    <!-- META -->
    <meta charset="utf-8">
    <!-- Optimize mobile viewport -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Node/Angular Todo App</title>
    <!-- SCROLLS -->
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.0/css/bootstrap.min.css"><!-- load bootstrap -->
    <style>
        html                    { overflow-y:scroll; }
        body                    { padding-top:50px; }
        #todo-list              { margin-bottom:30px; }
    </style>
    <!-- SPELLS -->
    <script src="//cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script><!-- load jquery -->
    <script src="//lib.sinaapp.com/js/angular.js/angular-1.2.19/angular.min.js"></script><!-- load angular -->
    <script src="core.js"></script>
</head>
<!-- SET THE CONTROLLER AND GET ALL TODOS -->
<body ng-controller="mainController">
<div class="container">
    <!-- HEADER AND TODO COUNT -->
    <div class="jumbotron text-center">
        <h1>I'm a Todo-aholic <span class="label label-info">{{ todos.length }}</span></h1>
    </div>
    <!-- TODO LIST -->
    <div id="todo-list" class="row">
        <div class="col-sm-4 col-sm-offset-4">
            <!-- LOOP OVER THE TODOS IN $scope.todos -->
            <div class="checkbox" ng-repeat="todo in todos">
                <label>
                    <input type="checkbox" ng-click="deleteTodo(todo._id)"> {{ todo.text }}
                </label>
            </div>
        </div>
    </div>
    <!-- FORM TO CREATE TODOS -->
    <div id="todo-form" class="row">
        <div class="col-sm-8 col-sm-offset-2 text-center">
            <form>
                <div class="form-group">
                    <!-- BIND THIS VALUE TO formData.text IN ANGULAR -->
                    <input type="text" class="form-control input-lg text-center" placeholder="I want to buy a puppy that will love me forever" ng-model="formData.text">
                </div>
                <!-- createToDo() WILL CREATE NEW TODOS -->
                <button type="submit" class="btn btn-primary btn-lg" ng-click="createTodo()">Add</button>
            </form>
        </div>
    </div>
</div>
</body>
</html>
```

## 开发总结

现在我们完整的开发了一个应用，实现了通过API创建、展示和删除TODO列表的功能。简单的回顾一下我们完成的工作：

- 使用Express搭建RESTful接口
- 使用mongoose连接MongoDB接口
- Angular中的AJAX调用`$http`
- 没有刷新的单页项目
- 吃自家狗粮(抱歉我真是太爱这个词了)




### 测试应用细节

可以前往[Github](https://github.com/scotch-io/node-todo)下载完整的项目源码。为了保证它能运行需要：

- 确定安装了Node和NPM
- 复制仓库`git clone git@github.com:scotch-io/node-todo`
- 安装依赖`npm install`
- 启动服务`node server.js`
- 在浏览器中打开`http://localhost:8080`

希望这个教程能够帮助你理解如何将多个部分协同工作。在将来的教程中我们将会学习如何将`server.js`分解成多个文件。



### 后期学习

如果你对MEAN(Mongo,Express,Angular,Node)集成开发很感兴趣，可以看一看这篇关于如何创建属于自己的MEAN项目有的教程：

[Setting Up a MEAN Stack Single Page Application](http://scotch.io/bar-talk/setting-up-a-mean-stack-single-page-application)



这篇文章是[Node and Angular TODO](http://scotch.io/series/node-and-angular-to-do-app)系列教程中的一篇。


整个系列如下：

-  [Creating a Single Page To-do App with Node and Angular](http://scotch.io/tutorials/javascript/creating-a-single-page-todo-app-with-node-and-angular)
- [Node Application Organization and Structure](http://scotch.io/tutorials/javascript/node-and-angular-to-do-app-application-organization-and-structure)
- [Angular Modules: Controllers and Services](http://scotch.io/tutorials/javascript/node-and-angular-to-do-app-application-organization-and-structure)


***



原文地址：

- [Creating a Single Page Todo App with Node and Angular](http://scotch.io/tutorials/javascript/creating-a-single-page-todo-app-with-node-and-angular)
