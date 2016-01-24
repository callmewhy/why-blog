title: 再遇柯里化：瀑布 IM 的消息封装
date: 2016-01-16 09:33:49
tags: Curry
categories: 开发笔记
description: 柯西不是理，陪我到最后。
---

## 背景介绍

最近在做微信订阅号爬虫的时候，突然感觉可以搞这样一个报警系统：如果解析的内容出现了错误，通过『[瀑布 IM](https://pubu.im/)』发送消息给我。

有这样聪明懂事的爬虫，绝对省心不少。

## 初步实现

功能嘛很简单，就是爬虫解析网页的时候，如果发现解析的内容和期待的内容格式不相符（比如正则没匹配上），则调用报警接口，预计应该是 `pubu.error('extract item failed')` 这样的调用方式。

我们先分析一下接口需要哪些数据，瀑布的文档里是这样描述的：

```json
{
  "text": "文本",
  "attachments": [{
    "title": "标题",
    "description": "描述",
    "url": "链接",
    "color": "warning|info|primary|error|muted|success"
  }],
  "displayUser": {
    "name": "机器人名称",
    "avatarUrl": "头像地址"
  }
}
```

大概是需要：消息的内容，附件的标题、描述、链接、类型，发送者的名称、头像。

于是我们很快可以写出一个报警函数：

```js
function sendPubuMessage(type, sender, title, description, url) {
  const attachment = {
    title: title,
    description: description,
    url: url,
    color: type,
  }
  request.post('https://hooks.pubu.im/services/xxxxxxx', {
    json: {
      text: moment().format('GGGG-MM-DD HH:mm'),
      attachments: [attachment],
      displayUser: {
        name: sender,
      },
    },
  }, (err, response) => {
    if (err || response.statusCode !== 200) {
      console.error('网络异常！提交瀑布失败：' + err) // eslint-disable-line
    }
  })
}
```

然后调用方法如下：

```js
sendPubuMessage('error', '微信爬虫', 'Extract key failed!', 'I do xxxx xxxx and failed', 'http://my.url/for/this/error')
```

测试一下，木问题：

![](http://ww1.sinaimg.cn/large/61d238c7jw1f01d3ddgl6j20ds05oaa7.jpg)

## 调整函数

然而，现在这个调用方法用起来还是不方便：

- 每次需要手动输入消息的级别，比如 `error` 这种，容易手误
- 每次需要手动输入发送者的机器人名字，不易管理
- 消息发送的频道接口写死在了函数里，不方便定制

于是乎，需要把 `sender` 和 `type` 分离出来。

先用 `buildType` 来组装 `type` ，生成各种消息类型，主要是定义 `color` 属性，用于在消息中显示不同级别的颜色：

```js
function buildType(color) {
  return {
    color: color,
  }
}
const info = buildType('info')
const warning = buildType('warning')
const error = buildType('error')
const success = buildType('success')
```

再用 `buildSender` 来组装 `sender` ，生成各种发送者，主要是定义 `name` 和 `url` 属性，即发送者的名称和需要发送的频道地址：

```js
function buildSender(name, url) {
  return {
    name: name,
    url: url,
  }
}
const wechat = buildSender('微信爬虫', 'https://hooks.pubu.im/services/111111111')
const sogou = buildSender('搜狗爬虫', 'https://hooks.pubu.im/services/222222222')
const log = buildSender('系统日志', 'https://hooks.pubu.im/services/333333333')
```

最后函数稍作调整，变成了这样：

```js
function sendPubuMessage(type, sender, title, description, url) {
  const attachment = {
    title: title,
    description: description,
    url: url,
    color: type.color,
  }
  request.post(sender.url, {
    json: {
      text: moment().format('GGGG-MM-DD HH:mm'),
      attachments: [attachment],
      displayUser: {
        name: sender.name,
        avatarUrl: sender.avatar,
      },
    },
  }, (err, response) => {
    if (err || response.statusCode !== 200) {
      console.error('网络异常！提交瀑布失败：' + err) // eslint-disable-line
    }
  })
}
```

调用的地方成了这样：

```js
// 由 微信爬虫 发送一条 error 消息
sendPubuMessage(error, wechat, 'failed!', 'I xx and failed', 'http://my.url/for/this/error')
// 由 搜狗爬虫 发送给一条 warning 消息
sendPubuMessage(warn, sogou, 'failed!', 'I xx and failed', 'http://my.url/for/this/error')
```

## 封装接口

函数基本是确定了，但是这样的函数外部对象需要使用的时候，只能：

```js
const pubu = require('./lib/pubu')
pubu.sendPubuMessage(pubu.error, pubu.wechat, 'failed!')
```

这真是太丑了。我希望能够这样调用：

```js
const pubu = require('./lib/pubu')
pubu.wechat.error('failed!')
```

我们需要改造！我们希望能直接通过 `sender` 对象发送消息，所以需要改写一下 `sender` 的 `builder` 函数：

```js
function buildSender(name, url) {
  return {
    name: name,
    url: url,
    info: function(title, description, url) {
      sendPubuMessage(info, this, title, description, url)
    },
    warn: function(title, description, url) {
      sendPubuMessage(warning, this, title, description, url)
    },
    error: function(title, description, url) {
      sendPubuMessage(error, this, title, description, url)
    },
    success: function(title, description, url) {
      sendPubuMessage(success, this, title, description, url)
    },
  }
}

const wechat = buildSender('微信爬虫', 'https://hooks.pubu.im/services/111111111111111')
```

修改过后我们就可以这样调用啦：

```js
wechat.info('info test')
wechat.warn('warn test')
wechat.error('error test')
wechat.success('success test')
```

测试结果看起来还不错：

![](http://ww2.sinaimg.cn/large/61d238c7jw1f01f6ukhvjj20di0gc3z5.jpg)

## 重构实现

然而，这部分代码看得我总是慌得很：

```js
function buildSender(name, url) {
  return {
    name: name,
    url: url,
    info: function(title, description, url) {
      sendPubuMessage(info, this, title, description, url)
    },
    warn: function(title, description, url) {
      sendPubuMessage(warning, this, title, description, url)
    },
    error: function(title, description, url) {
      sendPubuMessage(error, this, title, description, url)
    },
    success: function(title, description, url) {
      sendPubuMessage(success, this, title, description, url)
    },
  }
}
```

为什么这个世界上充满了重复。

为什么？为什么？为什么？为什么？

是的，重复了四遍。

是的，上面那句是个双关。

仔细想想，其实我们要做的就是封装 `sendPubuMessage` 以便外部调用。这个函数接受三类参数：

- type，消息类型，不同类型的消息有不用的颜色区分
- sender，发送者，包括发送者名称和发送到的频道地址
- message，后面三个参数都是消息的内容，统一归为一类，`title` 是必须的， `description` 和 `url` 是可选的

每传入一个参数，其实这个函数就完善了一点点。

比如我传入了 `error` ，那后面不管传入什么，这都是个发送 `error` 消息的函数。
比如我再传入了 `wechat`，那后面不管传入什么消息，这都是个发送微信爬虫的 `error` 消息的函数。

感觉有点眼熟，这不是柯里化的思路吗？不妨用柯里化函数试试。

## 柯里化

找了一个 JS 的柯里化的库：[curry](https://github.com/dominictarr/curry)，柯里化后的调用是这样的：

```js
const curry = require('curry')
const curriedSend = curry(sendPubuMessage)

function buildSender(name, url) {
  const sender = {
    name: name,
    url: url,
  }
  sender.info = curriedSend(info)(sender)
  sender.warn = curriedSend(warning)(sender)
  sender.error = curriedSend(error)(sender)
  sender.success = curriedSend(success)(sender)
  return sender
}
const wechat = buildSender('微信爬虫', 'https://hooks.pubu.im/services/111111111111111')


```

由于不再是 `function` 了，所以 `this` 失效，只能通过这种『声明外赋值』的方式来实现。（JS 学艺不精，应该有更好的方法，欢迎指点）

看起来似乎是简洁了一些，然而，在测试的时候发现，`wechat.info` 这个函数如果接受了少于3个参数就不会执行了。

比如这样的时候：

```js
wechat.info('info test')
```

仔细一想，柯里化之后的函数应该是期待五个参数输入，而此时我才输入了三个参数： `type`、`sender`、`title`。讲道理的话，此时的执行结果，应该是一个期待输入两个参数的参数。我们打印一下，果然：

```
console.log(wechat.info('info test').length) // 2
```

这就有点辣手了啊，柯里化之后把我本来的可选参数给搞没了，而大部分情况下其实我只传个 `title` 就结束了，剩下来两个参数是不会传的。

换句话说为了省几个字母的内部实现，现在每次外部调用都需要传入两个额外的参数。

你知道什么时候我会觉得我是个天才吗？

![](http://ww3.sinaimg.cn/large/61d238c7jw1f01gosvgvhj202s02s0sj.jpg)

当我发现我以前原来是一个傻逼的时候。

整理一下思绪，柯里化显然需要把所有的参数都假设成需要输入的参数，然后再做局部应用，要不然一个 `()` 人家怎么知道是该直接调用返回运算结果，还是该局部调用返回一个新的函数呢？

那我可以在柯里化的结果外面包一层啊，根据传入参数的数量来决定生成的柯里化的结果是该有几个入参，比如这样：

```js
const buildcurriedSend = (type) => {
  return () => {
    const args = [].slice.call(arguments)
    const curriedSend = curry.to(2 + args.length, sendPubuMessage)
    return curriedSend(type)(sender)
  }
}
```

然而这方法并没有调用，虽然通过 `arguments` 知道了参数的数量，但是并没有将参数传入并调用函数。

如果要调用，我需要自己对这个生成的函数传入参数，而不是像现在这样直接返回一个函数。

『传入参数』之后才能『生成新函数』，『生成新函数』之后需要传入『传入的参数』来调用函数，那我为什么不直接把参数组装一下给这个函数呢？

想到这里的时候我的内心是崩溃的。

但是也是光明的：是啊，为什么我一定要柯里化呢？

## 去柯里化

这种参数不确定的场景，其实并不适合柯里化，个人感觉。

一开始的思路是：需要局部调用函数，生成一个新的函数供外部调用。

其实也就是：提供部分参数，然后将参数补全并调用。那我为何不用 `apply` 方法呢，将外部传入的参数把持住，然后在前面插上 `type` 和 `sender` ，然后作为参数传给那个函数就可以了。

而且由于我可以自己组装函数，`this` 指针也重新起了作用：

```js
function buildSendMessage(type) {
  return () => {
    const args = [].slice.call(arguments)
    args.unshift(type, this)
    sendPubuMessage.apply(this, args)
  }
}

function buildSender(name, url) {
  const sender = {
    name: name,
    url: url,
    info: buildSendMessage(info),
    warning: buildSendMessage(warning),
    error: buildSendMessage(error),
    success: buildSendMessage(success),
  }
  return sender
}
```

最后的完整代码是这样的：

```js
const request = require('request')
const moment = require('moment')

// ----------------------------------------------------------------------------
// 消息类型
// ----------------------------------------------------------------------------

function buildType(color) {
  return {
    color: color,
  }
}

const info = buildType('info')
const warning = buildType('warning')
const error = buildType('error')
const success = buildType('success')

// ----------------------------------------------------------------------------
// 发消息的函数定义
// ----------------------------------------------------------------------------

function sendPubuMessage(type, sender, title, description, url) {
  const attachment = {
    title: title,
    description: (typeof description === 'object') ? JSON.stringify(description) : description,
    url: url,
    color: type.color,
  }
  request.post(sender.url, {
    json: {
      text: moment().format('GGGG-MM-DD HH:mm'),
      attachments: [attachment],
      displayUser: {
        name: sender.name,
        avatarUrl: sender.avatar,
      },
    },
  }, (err, response) => {
    if (err || response.statusCode !== 200) {
      console.error('网络异常！提交瀑布失败' + err) // eslint-disable-line
    }
  })
}

// ----------------------------------------------------------------------------
// 消息的发送者
// ----------------------------------------------------------------------------

function buildSendMessage(type) {
  return () => {
    const args = [].slice.call(arguments)
    args.unshift(type, this)
    sendPubuMessage.apply(this, args)
  }
}

function buildSender(name, url) {
  const sender = {
    name: name,
    url: url,
    info: buildSendMessage(info),
    warning: buildSendMessage(warning),
    error: buildSendMessage(error),
    success: buildSendMessage(success),
  }
  return sender
}

module.exports.wechat = buildSender('微信爬虫', 'https://hooks.pubu.im/services/111111111111111')
module.exports.sogou = buildSender('搜狗爬虫', 'https://hooks.pubu.im/services/111111111111111')
module.exports.log = buildSender('系统日志', 'https://hooks.pubu.im/services/222222222222222')
```


终于可以这样调用接口了：

```js
pubu.log.warning('Test Warning')
pubu.log.error('Test Error')
pubu.log.success('Test Success')
```

## 小结

经过一通虾折腾，花了半天的时间。

JS 还是有待深入学习，感觉一旦遇到一些稍微深入一点的话题，自己的知识储备就显得乏力了。比如 `this` 比如 `apply` 比如 `call` 比如 `bind` 各种。

回想起来，学习 Swift 的过程中了解过一段时间的 FRP 并且整理了一些文章。虽然粗浅地看了一些理论知识，但是并没有什么真枪实弹的经验。今天终于在项目里实验了一次，虽然结果以失败告终，但是内心是

![](http://ww2.sinaimg.cn/large/61d238c7jw1f01illdtqoj202s02sa9x.jpg)

崩溃的。


***

相关文章：

- [Functional Reactive Programming in Swift](http://blog.callmewhy.com/2015/05/11/functional-reactive-programming-1/)
- [Swift 中的柯里化](http://blog.callmewhy.com/2014/11/23/currying-in-swift/)
