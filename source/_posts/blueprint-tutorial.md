title: API Blueprint 指导手册
tags: Blueprint
date: 2014-06-05 16:04:57
categories: 翻译笔记
description: 让写文档这种枯燥的事情变得无比舒畅。
---

欢迎来到*API Blueprint*指导手册！

[*API Blueprint*](http://apiary.io/)是一套*API*描述标准，和*Markdown*一样，属于一种标记语言，可以把标记文稿转换成漂亮的接口文档。


在这里我们假设开发一个关于*Gist*的接口文档：***Gist Fox API***，以此来学习*API Blueprint*的全部核心特性。

*Gist*是一个粘贴工具，用户把内容粘贴后会得到一个URL，通过这个URL可以访问先前粘帖的文本，方便地共享文本或者代码。具体可以参考*Github*提供的[**GitHub Gists**](https://gist.github.com/)服务。


我们将在我们的*API*中提供如下功能：
- 列出所有的*Gist*
- 搜索一个*Gist*
- 创建一个*Gist*
- 编辑已有的*Gist*
- 给一个*Gist*星标
- 给一个*Gist*取消星标
- 检查一个*Gist*是否星标 
- 删除一个*Gist*


## Gist Fox API

刚开始的时候，我们的*Gist Fox API*是这样的：
```
FORMAT: 1A

# Gist Fox API
Gist Fox API is a **pastes service** similar to [GitHub's Gist][http://gist.github.com].
```

我们刚刚做了什么？接下来让我们一行一行的来分析一下。


 - 元数据*Metadata*
```
FORMAT: 1A
```
你需要使用一个元数据(*metadata*)，来明确指定*API Blueprint*的版本。
比如在上述文件中我们使用的版本号是***1A***。


- 接口名称*API Name*
```
# Gist Fox API
```
每一个好的*API*都应该有一个名字，我们的API也是这样：***Gist Fox API***，使用*Markdown*中的`#`标记表示*API*的名字。


- 接口描述*API Description*
```
Gist Fox API is a **pastes service** similar to [GitHub's Gist][http://gist.github.com].
```
在*API*的名字后面可以用*Markdown*的语法加上一些*API*的解释。






## Markdown

如果要写一个*blueprint*文档，你唯一需要的就是一个文本编辑器，最好是一个*Markdown*的文本编辑器，*VI*和*Byword*都是不错的选择，在线的文本编辑器也没有任何问题。你甚至可以直接在*Github*的仓库里编辑*bluePrint*格式的文档。

如果你完全不知道*Markdown*，现在是一个学习的好机会。只需要点击[*Markdown Tutorial*](http://markdowntutorial.com/)就可以开始学习。学完了可别忘了回来看看，我们这里也很精彩！

如果你需要查阅*Markdown*的语法可以直接点击[*Gruber's Original*](http://daringfireball.net/projects/markdown/syntax)查看。







## First Resource
我们第一个要写的内容是根目录(*root*)，我们*API*的入口可以这样定义：
```
# Gist Fox API Root [/]
Gist Fox API entry point.

This resource does not have any attributes. Instead it offers the initial API affordances in the form of the HTTP Link header and HAL links.

## Retrieve Entry Point [GET]

+ Response 200 (application/hal+json)
    + Headers

            Link: <http:/api.gistfox.com/>;rel="self",<http:/api.gistfox.com/gists>;rel="gists"

    + Body

            {
                "_links": {
                    "self": { "href": "/" },
                    "gists": { "href": "/gists?{since}", "templated": true }
                }
            }
```

这些奇奇怪怪的内容是什么呢？让我们挨个来看一看。

- 资源*Resource*
```
# Gist Fox API Root [/]
```
在*Blueprint*里，所有的数据信息都是资源(*resource*)，比如用户、视频、文章。
*resource*的定义以`#`开始，中间是*resource*的名称，最后是用中括号包围的路径(*URI*)，需要注意的是*URI*是放在`[` `]`中的。*URI*是相对路径，在这里它就是个`/`。


- 资源描述*Resource Description*
```
# Gist Fox API Root [/]
Gist Fox API entry point.

This resource does not have any attributes. Instead it offers the initial API affordances in the form of the HTTP Link header and haveL links.
```
我们可以用*Markdown*的语法在*resource*名称的后面加上包含*API*名称的说明。
在这里`Gist Fox API`是*API*名称，`entry point`是说明。

- 行为*Action*
```
## Retrieve Entry Point [GET]
```
行为*action*是一个*HTTP*请求的属性之一，在发送请求的时候会随数据一起发送到服务器。
我们在这里定义了一个*action*叫做*Retrieve Entry Point (索引入口)*，它是一个*GET*类型的请求。我们可以在后面加上一些描述，但是因为这个*action*的名字(*Retrieve Entry Point*)已经把这个行为解释的很清楚了，所以我们就跳过了这一步。
一般来说常见的*action*，包括*GET*和*POST*两种。在Blueprint有以下四种*action*：
    - GET ： 获取数据
    - POST ： 添加数据
    - PUT ： 更新数据
    - DELETE ： 删除数据



- 回应*Response*
```
+ Response 200
```
在*API Blueprint*中一个*action*应该至少包括一个回应(*response*)。*response*是指当服务器收到一个请求(request)时候的响应。
在*response*中应该有一个状态码[*status code*](https://github.com/for-GET/know-your-http-well/blob/master/status-codes.md)和数据[*payload*](https://github.com/apiaryio/api-blueprint/blob/master/Glossary%20of%20Terms.md#payload)。
在这里我们定义最常见的状态码：*200*，表示请求成功。


- 响应负载*Response Payload*
```
+ Response 200 (application/hal+json)
    + Headers

            Link: <http:/api.gistfox.com/>;rel="self",<http:/api.gistfox.com/gists>;rel="gists"

    + Body

            {
                "_links": {
                    "self": { "href": "/" },
                    "gists": { "href": "/gists?{since}", "templated": true }
                }
            }
```
一个响应(response)经常包含一些负载(*payload*)。一个负载(*payload*)通常包含负载体(*body*)和负载头(*header*)两个部分。
在这个例子中，我们采用[*application/hal+json*](https://github.com/mikekelly/hal_specification)类型作为返回数据的类型。





## Complex Resource
有了前面的根路径作为入口点，我们可以继续后面的内容。既然我们的*API*是围绕*Gist*展开的，那就让我们来给*Gist*一个明确的定义，以及一个用来获取它的*action*：
```
# Group Gist
Gist-related resources of *Gist Fox API*.

## Gist [/gists/{id}]
A single Gist object. 
The Gist resource is the central resource in the Gist Fox API. It represents one paste - a single text note.

The Gist resource has the following attributes:

- id
- created_at
- description
- content

The states *id* and *created_at* are assigned by the Gist Fox API at the moment of creation.


+ Parameters
    + id (string) ... ID of the Gist in the form of a hash.

+ Model (application/hal+json)

    HAL+JSON representation of Gist Resource. In addition to representing its state in the JSON form it offers affordances in the form of the HTTP Link header and HAL links.

    + Headers

            Link: <http:/api.gistfox.com/gists/42>;rel="self", <http:/api.gistfox.com/gists/42/star>;rel="star"

    + Body

            {
                "_links": {
                    "self": { "href": "/gists/42" },
                    "star": { "href": "/gists/42/star" },
                },
                "id": "42",
                "created_at": "2014-04-14T02:15:15Z",
                "description": "Description of Gist",
                "content": "String contents"
            }

### Retrieve a Single Gist [GET]
+ Response 200

    [Gist][]
```

我们定义了一个和*Gist*相关的组*Group*，我们也定义了一个*Gist resource*以及它的模型阐述，还有一个能够获取*Gist*的*action*。

让我们来把它拆散了看一下。


- 资源分组*Groups of Resources*
```
# Group Gist
Gist-related resources of *Gist Fox API*.
```
我们的*API*最终会提供很多的资源，所以我们需要将相关的内容归类并分组。现在我们先简单的把所有的资源都归类在一个*Gist*组中。
不同的分组可以通过在开头的关键字***Group***区别。


- URI模板*URI Template*
```
## Gist [/gists/{id}]
```
在*URI*中的变量需要遵守[*URI*的模板格式](https://github.com/apiaryio/api-blueprint/blob/master/Glossary%20of%20Terms.md#uri-template)，在这个例子中，*Gist*的编号(*id*)在*URI*中就是`{id}`。


- URI参数*URI Parameters*
```
+ Parameters
    + id (string) ... ID of the Gist in the form of a hash.
```
这个*id*变量是这个*resource*中的一个参数(*parameter*)，我们定义它的类型为*string*，并且在后面加上一些解释。


- 资源模型*Resource Model*
```
+ Model (application/hal+json)

    HAL+JSON representation of Gist Resource. In addition to representing its state in the JSON form it offers affordances in the form of the HTTP Link header and HAL links.

    + Headers

            Link: <http:/api.gistfox.com/gists/42>;rel="self", <http:/api.gistfox.com/gists/42/star>;rel="star"

    + Body

            {
                "_links": {
                    "self": { "href": "/gists/42" },
                    "star": { "href": "/gists/42/star" },
                },
                "id": "42",
                "created_at": "2014-04-14T02:15:15Z",
                "description": "Description of Gist",
                "content": "String contents"
            }
```
资源模型*Resource Model*是前面定义的资源的一个样例，它可以在任何一个*request*或者*response*需要的位置引用，一个资源模型有着和前面所说的*payload*一模一样的结构。
在前面的例子中，还包含了一个额外的描述，也就是在`+ Model`和`+ Headers`中间的那部分内容。


- 引用资源模型*Referring the Resource Model*
```
### Retrieve a Single Gist [GET]
+ Response 200

    [Gist][]
```
有了资源模型(*Gist Resource Model*)，定义一个获取*Gist*的*action*就很容易了，直接写上`[Gist][]`就可以返回一个*Gist*资源。






## Modifying a Resource
接下来我们来写一个修改*Gist*的行为(*action*)：*Edit a Gist*，和删除*Gist*的行为(*action*)：*Delete a Gist*。
```
### Edit a Gist [PATCH]
To update a Gist send a JSON with updated value for one or more of the Gist resource attributes. All attributes values (states) from the previous version of this Gist are carried over by default if not included in the hash.

+ Request (application/json)

        {
            "content": "Updated file contents"
        }

+ Response 200

    [Gist][]

### Delete a Gist [DELETE]
+ Response 204
```

我们来看一看这个*payload*中的新内容：


- 请求中的负载*Request and its Payload*
```
+ Request (application/json)

        {
            "content": "Updated file contents"
        }
```
我们需要接受一些输入来修改资源(*Gist resource*)，这些输入是访问请求(*HTTP Request*)的一部分，在上面的例子中定义了这样的一个请求以及负载*payload*的内容。


- 空负载*Empty Payload*
```
### Delete a Gist [DELETE]
+ Response 204
```
在*Delete a Gist*这样一个删除的操作中，我们的返回内容只包含一个*204*的状态码而没有其他内容，这个*204*的状态码表示服务器成功执行了这个请求。






## Collection Resource
既然已经有了单个资源的接口，接下来我们来定义一个集合(*Gists Collection*)接口：

```
## Gists Collection [/gists{?since}]
Collection of all Gists.

The Gist Collection resource has the following attribute:

- total

In addition it **embeds** *Gist Resources* in the Gist Fox API.


+ Model (application/hal+json)

    HAL+JSON representation of Gist Collection Resource. The Gist resources in collections are embedded. Note the embedded Gists resource are incomplete representations of the Gist in question. Use the respective Gist link to retrieve its full representation.

    + Headers

            Link: <http:/api.gistfox.com/gists>;rel="self"

    + Body

            {
                "_links": {
                    "self": { "href": "/gists" }
                },
                "_embedded": {
                    "gists": [
                        {
                            "_links" : {
                                "self": { "href": "/gists/42" }
                            },
                            "id": "42",
                            "created_at": "2014-04-14T02:15:15Z",
                            "description": "Description of Gist"
                        }
                    ]
                },
                "total": 1
            }

### List All Gists [GET]
+ Parameters
    + since (optional, string) ... Timestamp in ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ` Only Gists updated at or after this time are returned.

+ Response 200

    [Gists Collection][]

### Create a Gist [POST]
To create a new Gist simply provide a JSON hash of the *description* and *content* attributes for the new Gist.

+ Request (application/json)

        {
            "description": "Description of Gist",
            "content": "String content"
        }

+ Response 201 (application/hal+json)

    [Gist][]
```

在这里你应该基本都熟悉这些接口了，除了其中的一个查询的参数：*since*。

- 查询参数*Query Parameters*
```
## Gists Collection [/gists{?since}]
```
作为*URI*的参数，查询参数的定义必须符合[*URI*模板格式](https://github.com/apiaryio/api-blueprint/blob/master/Glossary%20of%20Terms.md#uri-template)，而这里有所不同的是，查询参数里有一个问号，并且它总在*URI*的结尾处。


- *Action-specific Query Parameters*
```
### List All Gists [GET]
+ Parameters
    + since (optional, string) ... Timestamp in ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ` Only Gists updated at or after this time are returned.
```
通常查询的参数是针对诸多行为(*action*)中的某一个，所以我们只需要考虑和这个*action*相关的内容.






## Subsequent Resource
最后一个缺失的部分就是星标资源的功能了，利用前面所学的知识，我们可以这样定义：

```
## Star [/gists/{id}/star]
Star resource represents a Gist starred status.

The Star resource has the following attribute:

- starred


+ Parameters

    + id (string) ... ID of the gist in the form of a hash

+ Model (application/hal+json)

    HAL+JSON representation of Star Resource.

    + Headers

            Link: <http:/api.gistfox.com/gists/42/star>;rel="self"

    + Body

            {
                "_links": {
                    "self": { "href": "/gists/42/star" },
                },
                "starred": true
            }

### Star a Gist [PUT]
+ Response 204

### Unstar a Gist [DELETE]
+ Response 204

### Check if a Gist is Starred [GET]
+ Response 200

    [Star][]
```







## Complete Blueprint
你可以在*API Blueprint*的[示例源码库](https://github.com/apiaryio/api-blueprint/tree/master/examples)，找到完整的[*Gist Fox API*](https://raw.githubusercontent.com/apiaryio/api-blueprint/master/examples/Gist%20Fox%20API.md)。







## API Blueprint Tools
随着*Gist Fox Blueprint*接口文档的完工，接下来就应该让它发挥作用了。可以在*Github*中查看[渲染结果](https://github.com/apiaryio/api-blueprint/blob/master/examples/Gist%20Fox%20API.md)或者在*Apiary*中查看[渲染结果](http://docs.gistfoxapi.apiary.io/)。可以前往[*apiblueprint.org*](http://apiblueprint.org/)的[*Tooling Section*](http://apiblueprint.org/#tooling)寻找更多的相关工具包。



## Thanks
感谢董花花的勘误工作^_^



*** 
附录：
- [英文原版](http://apiary.io/blueprint)
- [Apiary官网地址](http://app.apiary.io/)
- [Blueprint官网地址](http://apiblueprint.org/)
- [案例*Gist Fox API*的源码](https://raw.githubusercontent.com/apiaryio/api-blueprint/master/examples/Gist%20Fox%20API.md)
- [案例*Gist Fox API*的文档](http://docs.gistfoxapi.apiary.io/)
- [其他*API Blueprint的案例*](https://github.com/apiaryio/api-blueprint/tree/master/examples)


















