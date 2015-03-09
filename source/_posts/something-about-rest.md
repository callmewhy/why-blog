title: 关于REST和Blueprint
tags: Blueprint
date: 2014-06-07 18:39:57
categories: 开发笔记
description: 编写接口文档是一项高雅而文艺的事业。
---


##*RESTful API*


最近在做*WLAN*精灵的*API*设计，因为前面一直断断续续的接触*RESTful API*，在这里系统的整理一下。


*RESTful API*是指符合*REST*原则的API设计。

> [*REST*](http://en.wikipedia.org/wiki/Representational_state_transfer) : Representational state transfer, introduced and defined in 2000 by [Roy Fielding](http://en.wikipedia.org/wiki/Roy_Fielding) in his [doctoral dissertation](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm).

对于*Representational state transfer*，[*Wiki*](http://zh.wikipedia.org/wiki/REST)上翻译为“含状态传输”，阮一峰老师在[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)中翻译为“表现层状态转化”，并且还解释了一下*Representational state transfer*中的*representational*：

> 其中，“表现层状态转化”一句省略了主语，表现层其实指的是“资源”(Resources)的表现层。



简单来说，就是把数据信息当做资源(*resource*)，通过不同的方法动词(*action*)来执行不同的操作。
下面是四种*action*的解释和举例：

- *GET* ： 获取以及检索数据
    - `/api/robots` 获得所有*robot*的数据
    - `/api/robots/search/Astro` 搜索一个名为*Astro*的*robot*
    - `/api/robots/2` 根据编号获取*robot*

- *POST* ： 添加数据
    - `/api/robots` 添加一条新的数据
- *PUT* ： 更新数据
    - `/api/robots/2` 通过*id*修改数据
- *DELETE* ： 删除数据
    - `/api/robots/2` 通过*id*删除数据


推荐一些关于*RESTful API*的网站：
- [Principles of good RESTful API Design](http://codeplanet.io/principles-good-restful-api-design/)
- [RESTful Tutorial](http://www.restapitutorial.com/)
- [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
- [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)


***


##*Blueprint*

那么什么是*Blueprint*呢？*Blueprint*是*apiary*公司开源的一个*API*文档编辑工具包，它可以把一些约定的标记转换成非常漂亮的文档，就像是*Markdown*一样。

在*Blueprint*中，基本遵循*RESTful API*的设计，*API*可以想象成是对一个资源(*resource*)的操作。
一级标签`*`是分组(*Group*)，表示某种对象资源(*resource*)，比如*Gist*这种。
二级标签`**`是某个路径(*URI*)，指一种属性，比如星标*star*、集合*collection*、等等
三级标签`***`是操作action，指针对这种属性的操作，比如获取、删除、编辑、星标。

比如官网案例中的*Gist*：

```
- Gist             # Group Gist
    - Gist              ## Gist [/gists/{id}]
        - Retrieve         ### Retrieve a Single Gist [GET]
        - Edit             ### Edit a Gist [PATCH]
        - Delete           ### Delete a Gist [DELETE]
    - Gists Collection  ## Gists Collection [/gists{?since}]
        - List             ### List All Gists [GET]
        - Create           ### Create a Gist [POST]
    - Star              ## Star [/gists/{id}/star]
        - Star             ### Star a Gist [PUT]
        - Unstar           ### Unstar a Gist [DELETE]
        - Check            ### Check if a Gist is Starred [GET]
```

推荐一些*Blueprint*的网站：
- [apiary官网](http://apiary.io/)
- [API Blueprint Tutorial](http://apiary.io/blueprint)
- [apiblueprint官网](http://apiblueprint.org/)













