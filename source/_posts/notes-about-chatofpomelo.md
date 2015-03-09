title: Pomelo聊天室项目源码解析
tags: Pomelo
date: 2014-07-01 16:57:18
categories: 开发笔记
description: 一个开源游戏服务器框架的聊天室示例项目的简单解读。
---

## 背景介绍

[Pomelo](http://pomelo.netease.com/index.html)是一个网易开源的高性能、高可伸缩、分布式多进程的游戏服务器框架。

官网有一个聊天室的教程：[Getting-Source-Code](https://github.com/NetEase/pomelo/wiki/Getting-Source-Code-&-Installation)，十分适合初学者学习。

花时间深入的了解了一下这个聊天室的项目的`tutorial-starter`分支，也就是一个最简单的聊天室原型，做一些笔记记录一下学习的过程。

项目源码：[点此下载](https://github.com/NetEase/chatofpomelo-websocket/tree/tutorial-starter)。


## 源码解析

### 整体了解
聊天室项目中只有一个index.html页面，基本结构如下：

- app - 聊天主页面
    - loginView   - 登陆页面
        - loginTitle - 登陆标题
        - loginTable - 登陆信息
        - loginError - 登陆错误提示
    - chatHistory - 聊天历史
    - toolbar - 聊天工具栏
- pop - 弹窗提醒
    - popHead - 提醒的标题
    - popContent - 提醒的内容


### 登陆操作
用户点击登陆按钮，会触发`client.js`中定义的`$("#login").click`事件。

首先是对用户名和频道号的长度判断和正则校验：
```
// 获取用户名和密码
username = $("#loginUser").attr("value");
rid = $('#channelList').val();

// 长度判断，必须为0~20的字符串
if(username.length > 20 || username.length == 0 || rid.length > 20 || rid.length == 0) {
    showError(LENGTH_ERROR);
    return false;
}

// 正则校验
if(!reg.test(username) || !reg.test(rid)) {
    showError(NAME_ERROR);
    return false;
}      

```

接下来就是登陆的逻辑处理部分，会调用前面定义的`queryEntry`函数：
```
function queryEntry(uid, callback) {
    var route = 'gate.gateHandler.queryEntry';
    pomelo.init({
        host: window.location.hostname,
        port: 3014,
        log: true
    }, function() {
        pomelo.request(route, {
            uid: uid
        }, function(data) {
            pomelo.disconnect();
            if(data.code === 500) {
                showError(LOGIN_ERROR);
                return;
            }
            callback(data.host, data.port);
        });
    });
};
```
首先定义route为`gate.gateHandler.queryEntry`，pomelo会访问当前hostname的3014端口，在game-server的`servers.json`里定义了服务器的地址和端口号：
```
"connector":[
     {"id":"connector-server-1", "host":"127.0.0.1", "port":4050, "clientPort": 3050, "frontend": true, "args":"--debug=32312"}
 ],
"chat":[
     {"id":"chat-server-1", "host":"127.0.0.1", "port":6050, "args":"--debug=32313"}
],
"gate":[
   {"id": "gate-server-1", "host": "127.0.0.1", "clientPort": 3014, "frontend": true, "args":"--debug=32311"}
]
```

可以看到，其实也就是访问了gate服务器。

***有点忙，更新先停了，SORRY***



***

参考文献：

- [pomelo](https://github.com/NetEase/pomelo)
- [pomelo api](http://pomelo.netease.com/api.html)
- [chat of pomelo](https://github.com/NetEase/chatofpomelo-websocket)