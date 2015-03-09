title: 使用CodeIgniter框架搭建RESTful API服务
tags: RESTful
date: 2014-07-12 12:38:49
categories: 翻译笔记
description: RESTful不仅仅是一套协议标准更是一种设计思路。
---

在2011年8月的时候，我写了一篇博客[《使用CodeIgniter框架搭建RESTful API服务》](http://outergalactic.org/blog/building-a-restful-service-using-codeigniter/)，介绍了RESTful的设计概念，以及使用CodeIgniter框架实现RESTful API的方法。转眼两年过去了，REST在这两年里有了很大的改进。我对于前一篇博客中的某些方面不是很满意，所以希望能利用这次机会写一个更加完善的版本。

我的项目基于[Phil Sturgeon](https://github.com/philsturgeon/)的[CodeIgniter REST Server](http://github.com/philsturgeon/codeigniter-restserver)，遵循他自己的[DBAD协议](http://www.dbad-license.org/)。Phil的这个项目很棒，干净利落，简单实用，并且极具特色，解决了我自己项目中两个问题：在请求中查询语句的使用，以及复杂的身份验证方法。正如我前面所说，我的项目基于Phil的项目开发，并且只有一个方面不同：我给每个资源分别分配一个控制器，而不是用一个独立庞大的控制器进行统一管理。使用多个独立控制器的好处是维护起来更加简单方便。



完整的项目源码可以在[awhitney42/codeigniter-restserver-resources](https://github.com/awhitney42/codeigniter-restserver-resources)下载。

## RESTful

在深入探讨之前，我们先来回顾一下TESTful的概念。REST全名Representational State Transfer，中文可以译为：表现层状态转化，是一种网络服务的架构工具，而不仅仅是一套接口规范。

### 使用名词而不是动词

你的RESTful接口应该提供访问而不是方法。
所以，这样是不可取的：

```
/createCustomer
/getCustomer?id=666
```

应该这样：

```
POST /customers
GET  /customers/666
```

### 所有东西都应该有ID

REST的优点之一就是简洁，一定程度上来讲，REST只是一个URI的集合，每个资源都需要一个独一无二的ID作为一个标识。

```
GET /customers/666
GET /products/4234
```


### 使用动词进行操作

在REST里，使用不同的HTTP动词进行增删改查的操作：

|动词         | 操作     |
|------------|----------|
|POST        |新建一个资源|
|GET         |读取一个资源|
|PUT         |更新一个资源|
|DELETE      |删除一个资源|

使用这个REST服务的映射表来设计接口，在开发客户端的时候可以轻松上手，不用深入理解接口的含义。这套标准十分适用于REST，符合面向服务架构(SOA)的核心设计思想：服务抽离、松耦合、可复用、可发现、健全性。

通过这个一致的映射关系，客户端也知道哪个动词是幂等（Idempotent）的。幂等是指这个操作可以重复多次，并且每次都会得到相同的结果。参照HTTP规范，GET、PUT和DELETE操作是幂等的，POST操作不是幂等的。这也就是为什么在REST中使用POST来进行添加操作，而使用PUT来进行更新操作。


### 把东西链接起来

REST中的一个核心概念是HATEOAS（Hypermedia As The Engine Of Application State），即“超媒体即应用状态引擎”。这意味着应该始终使用链接（超媒体）来获取资源信息，然后客户端通过这个链接就可以获取到不同的资源。

所以，相关的资源应当返回一条链接，而不是返回整个资源的内容。
也就是，该这样：

```
<officer id="1">
   <name>James T. Kirk</name>
   <rank>Captain</rank>
   <subordinates>
      <link ref="http://ncc1701/api/officers/2">
      <link ref="http://ncc1701/api/officers/3">
   </subordinates>
</officer>
```

而不该这样：

```
<officer id="1">
   <name>James T. Kirk</name>
   <rank>Captain</rank>
   <subordinates>
      <officer id="2">
         <name>Spock</name>
         <rank>Commander</rank>
      </officer>
      <officer id="3">
         <name>Doctor Leonard McCoy</name>
         <rank>Commander</rank>
      </officer>
   </subordinates>
</officer>
```

返回资源的链接应该用完整的URI地址而不是相对路径，这要求客户端请求这些资源的当前状态，维持HATEOAS原则。使用完整的URI地址的好处是客户端不需要额外的了解API相关的内容，只需要简单的访问这些链接就可以了。


### 提供多种资源表现方式

REST中的R代表Representational，即表现层，这意味着REST服务应该提供不同的表现方式，以全面支持不同的客户端请求。在一次HTTP请求中，服务器可以通过指定的表现方式返回资源，REST服务应该使用标准的表现方式，以便客户端之间的信息互通。

比如可以通过如下请求XML格式的数据：

```
GET /customers/666
Accept: application/xml
```

或者vcard格式：

```
GET /customers/666
Accept: application/vcard
```

或者pdf格式：

```
GET /customers/666
Accept: application/pdf
```

或者自定义格式：

```
GET /customers/666
Accept: application/vnd.mycompany.customer-v1+json
```


### 使用状态码作为回复

HTTP的状态码提供了一套标准化方案，用来反馈请求的状态。

||含义|解释|
|------|------|------|
|200|OK|确认GET、PUT和DELETE操作成功|
|201|Created|确认POST操作成功|
|304|Not Modified|用于条件GET访问，告诉客户端资源没有被修改|
|400|Bad Request|通常用于POST或者PUT请求，表明请求的内容是非法的|
|401|Unauthorized|需要授权|
|403|Forbidden|没有访问权限|
|404|Not Found|服务器上没有资源|
|405|Method Not Allowed|请求方法不能被用于请求相应的资源|
|409|Conflict|访问和当前状态存在冲突|





## CodeIgniter

[CodeIgniter](http://ellislab.com/codeigniter)是一个流行的MVC框架，很适合用来进行RESTful API开发。控制器（Controller）处理客户端的请求并返回内容，模型（Model）进行增删改查（CRUD）的操作，视图（View）用来处理资源的表现格式。不过在这个例子里，我们没有用模型，而是直接用控制器进行格式处理，这样整个项目更干净更简单。

代码可以直接在Github中获取：[codeigniter-restserver-resources](https://github.com/awhitney42/codeigniter-restserver-resources)。

在接下来的例子里，我们用到了`REST_Controller`这个库，继承自原生的`CI_Controller`。它可以完成绝大部分繁杂的工作：处理请求、调用模块、格式化内容、返回数据。你的每个资源控制器都应该继承自`REST_Controller`这个类。

下面我们来看一个例子，我们假设要提供一个`Widgets`资源的RESTful接口：
```
class Widgets extends REST_Controller
```

### get()

`Widgets`类中的第一个函数是`get()`，用来响应HTTP的GET请求。这个函数调用了父类的protected函数`_get()`，用来获取请求中的参数ID。然后这个函数根据是否有参数ID使用`widgets_model`调用`getWidgets()`或者`getWidget($id)`方法，模型的返回值将会通过父类的`response()`函数返回，返回的内容包含对应的状态码和符合格式的数据。

```
function get()
{    
    $id = $this->_get('id');
    if(!$id)
    {
        $widgets = $this->widgets_model->getWidgets();                        
        if($widgets)
            $this->response($widgets, 200); // 200 being the HTTP response code
        else
            $this->response(array('error' => 'Couldn\'t find any widgets!'), 404);
    }
    $widget = $this->widgets_model->getWidget($id);
    if($widget)
        $this->response($widget, 200); // 200 being the HTTP response code
    else
        $this->response(array('error' => 'Widget could not be found'), 404);
}
```

`cURL`是一个很好的命令行工具，我们可以用来测试REST服务，返回一个JSON格式的数据：
```
$ curl -i -H "Accept: application/json" -X GET http://foo.com/index.php/api/widgets
```

则会返回如下内容：

```
HTTP/1.1 200 OK
Status: 200
Content-Type: application/json

{"1":{"id":1,"name":"sprocket"},"2":{"id":2,"name":"gear"}}
```

请求特定的资源也很简单，这次我们去请求ID为2的`widget`并且通过XML格式返回：

```
$ curl -i -H "Accept: application/xml" -X GET http://foo.com/index.php/api/widgets/id/2
```
返回如下内容：
```
HTTP/1.1 200 OK
Status: 200
Content-Type: application/xml

<?xml version="1.0" encoding="utf-8"?>
<xml><id>2</id><name>gear</name></xml>
```

如果请求一个不存在的资源就会返回404的错误码：
```
HTTP/1.1 404 Not Found
Status: 404
Content-Type: application/xml

<?xml version="1.0" encoding="utf-8"?>
<xml><error>Widget could not be found</error></xml>
```

### post()

接下来的函数是`post()`，用来处理创建widget的POST请求。可以通过`$this->_post_args`获取请求的数据。父类通过`Format.php`对请求的数据进行处理并把它们放到了`$this->_post_args`里。接下来`post()`方法使用`widgets_model`模型调用`createWidgets($data)`函数，如果数据非法或者请求冲突，`widgets_model`模型会抛出异常并且返回异常的内容。如果调用成功，则会调用`getWidget($id)`函数获取最新的`widget`，在返回的时候会将返回值和`201 (Created)`的状态码一起返回。

```
function post()
{
    $data = $this->_post_args;
    try {
        $id = $this->widgets_model->createWidget($data);
    } catch (Exception $e) {
        // Here the model can throw exceptions like the following:
        // * Invalid input data:
                    //   throw new Exception('Invalid request data', 400);
        // * Conflict when attempting to create, like a resubmit:
                    //   throw new Exception('Widget already exists', 409)
        $this->response(array('error' => $e->getMessage()),
                                    $e->getCode());
    }
    if ($id) {
        $widget = $this->widgets_model->getWidget($id);
        $this->response($widget, 201); // 201 is the HTTP response code
    } else
        $this->response(array('error' => 'Widget could not be created'),
                                    404);
}   
```


为了测试通过POST请求新建资源的操作，我们需要把我们的cURL包裹在PHP代码里，创建一个`rest_client.php`文件：

```
print "\n-----TESTING REST POST-----\n";
test_post();
function test_post() {
   $data = array("name" => "bolt");
   $data_string = json_encode($data);
   $ch = curl_init('http://foo.com/index.php/api/widgets');
   curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
   curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
   curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
   curl_setopt($ch, CURLOPT_HTTPHEADER, array(
       'Content-Type: application/json',
       'Content-Length: ' . strlen($data_string))
   );
   $result = curl_exec($ch);
   $httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
   $contenttype = curl_getinfo($ch, CURLINFO_CONTENT_TYPE);
   print "Status: $httpcode" . "\n";
   print "Content-Type: $contenttype" . "\n";
   print "\n" . $result . "\n";
}
```
在命令行中通过PHP命令执行：
```
$ php rest_client.php
```
返回内容如下：
```
-----TESTING REST POST-----
Status: 201
Content-Type: application/xml

<?xml version="1.0" encoding="utf-8"?>
<xml><id>3</id><name>bolt</name></xml>
```

### put()

接下来的方法是`put()`，用来处理PUT请求，更新已经存在的资源数据。处理过程和POST的处理十分相似，最大的区别就在于使用`$this->_put_args`而不是`$this->_post_args`，以及返回200而不是201。
```
public function put()
{
    $data = $this->_put_args;
    try {
        //$id = $this->widgets_model->updateWidget($data);
        $id = $data['id']; // test code
        //throw new Exception('Invalid request data', 400); // test code
    } catch (Exception $e) {
        // Here the model can throw exceptions like the following:
        // * For invalid input data: new Exception('Invalid request data', 400)
        // * For a conflict when attempting to create, like a resubmit: new Exception('Widget already exists', 409)
        $this->response(array('error' => $e->getMessage()), $e->getCode());
    }
    if ($id) {
        $widget = array('id' => $data['id'], 'name' => $data['name']); // test code
        //$widget = $this->widgets_model->getWidget($id);
        $this->response($widget, 200); // 200 being the HTTP response code
    } else
        $this->response(array('error' => 'Widget could not be found'), 404);
}
```


为了测试UPDATE更新资源的功能，我们依旧使用PHP进行cURL的操作，大多数的网络服务器默认没有开启PUT和DELETE，我们可以在header中使用`X-HTTP-Method-Override`，通过POST来发送PUT请求。这样的话，服务器会把它当做一个POST请求，而REST服务器会把它作为PUT操作处理。
```
print "\n-----TESTING REST PUT-----\n";
test_put();

function test_put() {
   $data = array("id" => "3", "name" => "nut");
   $data_string = json_encode($data);
   $ch = curl_init('http://foo.com/index.php/api/widgets');
   curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "POST");
   curl_setopt($ch, CURLOPT_POSTFIELDS, $data_string);
   curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
   curl_setopt($ch, CURLOPT_HTTPHEADER, array(
       'X-HTTP-Method-Override: PUT',
       'Content-Type: application/json',
       'Content-Length: ' . strlen($data_string))
   );
   $result = curl_exec($ch);
   $httpcode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
   $contenttype = curl_getinfo($ch, CURLINFO_CONTENT_TYPE);
   print "Status: $httpcode" . "\n";
   print "Content-Type: $contenttype" . "\n";
   print "\n" . $result . "\n";
}
```

同样使用PHP命令测试：
```
$ php rest_client.php
```
返回结果：
```
-----TESTING REST PUT-----
Status: 200
Content-Type: application/xml

<?xml version="1.0" encoding="utf-8"?>
<xml><id>3</id><name>nut</name></xml>
```


### delete()

最后的函数就是delete了，请求中必须包含ID的参数否则就会返回400(Bad Request)的状态码，因为我们不希望用户删除所有的资源内容。如果ID没有对应的资源则会返回404(Not Found) 错误码。如果对应的`widget`存在，则会尝试调用`deleteWidget($id)`删除对应资源。操作中的所有异常都会被捕获到并且返回。如果删除成功，返回200(Success)状态码。


```
function delete()
{
    $id = $this->_get('id');
    if(!$id)
    {
        $this->response(array('error' =>
                            'An ID must be supplied to delete a widget'), 400);
    }
    //$widget = $this->widgets_model->getWidget($id);
    $widget = @$widgets[$id]; // test code
    if($widget) {
        try {
            $this->widgets_model->deleteWidget($id);
        } catch (Exception $e) {
            // Here the model can throw exceptions like the following:
            // * Client is not authorized: new Exception('Forbidden', 403)
            $this->response(array('error' => $e->getMessage()),
                                    $e->getCode());
        }
            $this->response($widget, 200); // 200 being the HTTP response code
    } else
        $this->response(array('error' => 'Widget could not be found'), 404);
}
```
我们可以用命令行工具`cURL`进行测试，参数ID像GET一样放在URL地址中，和PUT相同，我们通过`X-HTTP-Method-Override`使服务器把请求当做POST处理，REST服务则会把这次请求当做DELETE操作处理。
```
$ curl -i -H "Accept: application/xml" -H "X-HTTP-Method-Override: DELETE" -X POST http://foo.com/index.php/api/widgets/id/1
```

返回200 (Success)说明id为1的资源已经被成功删除：
```
HTTP/1.1 200 OK
Status: 200
Content-Type: application/xml

<?xml version="1.0" encoding="utf-8"?>
<xml><id>1</id><name>sprocket</name></xml>
```

如果ID非法，则会返回404(Not Found)错误码，如果没有删除的权限，则会返回403 (Forbidden) 状态码。




### Authentication

正如前面提到的，Phil的REST框架和我的原来的设计相比，在用户权限认证上有了很大的改进， 提供[basic](http://en.wikipedia.org/wiki/Basic_access_authentication)和[digest](http://en.wikipedia.org/wiki/Digest_access_authentication)两种认证方式。还有很多其他特性，比如可以用LDAP字典将授权融为一体，具体可以在`config/rest.php`中设置。

除了传统的认证方式，你还可以使用API keys或者IP地址的白名单进行用户权限管理。


### DTO

在原先的设计中，我使用DTO(Data Transfer Object)在不同格式之间传输数据，Phil使用Format类来解决这个问题，从`Widget`的例子中我们可以看到，使用数据是多么的方便，只需要`$this->_post_args`或者`$this->response()`就可以解决问题。当然，这并不意味着DTO不好用，但是在很多场合下它会显得十分复杂和庞大。
在我原来的项目里，我对客户端需要使用和服务器端同样的DTO库来传输数据很不满意，因为它违反了KISS的原则。

## 总结

REST目前已经是比较成熟的网络服务的框架模型方案，是API产品目录网站[programmableweb](http://www.programmableweb.com/)的基石，是各式各样应用和服务器的数据传输的基础，同时对于[SOA](http://en.wikipedia.org/wiki/Service-oriented_architecture)来说也是十分重要，促进网络端和移动端的接口技术日趋成熟。

正如前面所看到的，用CodeIgniter框架实现RESTful接口十分简单，可以下载我的项目源码[awhitney42/codeigniter-restserver-resources](https://github.com/awhitney42/codeigniter-restserver-resources)，从现在就开始开发吧！


## 后记
REST是一种遵从传统设计模式的架构风格，在过去的几年中一直都在改进，并且会一直坚持下去。所以如果对于REST或者是本文有什么意见，或者对于其中的概念有什么困惑，请在文章后面留言。



***


原文地址：

- [RESTful Services with CodeIgniter](http://outergalactic.org/blog/restful-services-with-codeigniter/)