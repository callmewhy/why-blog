title: CGI FastCGI WSGI 学习笔记
date: 2015-12-07 10:11:50
tags: Web
categories: 学习笔记
description: 充分了解当年原始社会的举步维艰，才能体会如今小康阶段的来之不易。
---

最近用 Flask 给以前写的 OpenCV 代码配置一个入口供前端调用，翻文档的时候发现部署服务器和在本地运行与我一开始想的并不太一样。于是花点时间了解并整理了一下，补补基础课。


## CGI

### 概念

CGI 是一种服务器和后端可执行程序之间的交互标准，它描述了『如何通过环境变量来传递请求信息』。

### 原理

最原始的服务器，简单到就是访问文件目录，每次的请求都是请求加载目录下的文件。比如文档放在 `/usr/local/apache/htdocs` 目录下，访问 http://example.com/index.html 其实就是请求 `/usr/local/apache/htdocs/index.html` 文件。

CGI 通过服务器脚本（或者二进制文件），扩展了这个基础的『访问过程』。它利用程序的标准输入输出流，完成 HTTP 通信。每次请求的文本以标准输入流的形式进入服务器端的 CGI 程序，创建进程并执行，然后将运行结果通过进程的标准输出流输出作为响应。

比如 `/usr/local/apache/htdocs/cgi-bin` 是我们的 CGI 目录，当请求了 CGI 目录里的文件的时候（比如访问 http://example.com/cgi-bin/printenv.pl ），服务器并不会返回这个文件，而会运行这个程序，然后将生成的内容返回给客户端。所以理论上，任何有输入输出能力的语言都可以用来写 CGI。

CGI 的工作原理如图：

![](http://ww1.sinaimg.cn/large/61d238c7jw1eyr31id7kqj20z40cwjtn.jpg)

解释一下，上面图中的 CGI 指的是 CGI 程序。CGI 是一种协议标准，CGI 程序是实现了 CGI 标准的程序，确保输入输出合法，这是两个相关但不同的概念。

### 优点

CGI 的优点也就是它的作用了。CGI 程序提供了很多静态网页无法实现的功能，比如加载数据、数据运算等等。早期的动态网页基本都是基于 CGI 实现的。

### 缺点

在 CGI 协议下，解析器的反复加载是性能低下的主要原因。每个发送到服务器的请求，都需要经过『启动进程、处理请求、结束进程』三个步骤，所以当访问量增大时，系统资源的开销也会增大，导致服务器性能下降甚至服务中断。

更惨的是，这种『一个请求一个进程』的模式意味着没有『状态』可言，导致很多资源无法复用，比如连接数据库、内存缓存等等。



## FastCGI

### 概念

FastCGI 是 CGI 的增强版本，用于减少 Server 与 CGI 应用之间的交互开销，从而使 Server 可以同时处理更多的请求。

### 原理

和 CGI 的 `fork-and-execute` 模式不同的是，FastCGI 以 `Daemon` 的形式运行，在初始化的时候会启动一个 `FastCGI Server` 然后长驻内存，处理一系列的请求。

Nginx+FastCGI 的工作流程是这样的：

- 初始化 FastCGI 进程管理器，启动主进程和多个 CGI 子进程。主进程主要是管理子进程，同时还需要监听端口（例如9000端口）；子进程则是处于等待连接的状态。
- 当请求到达服务器时，Nginx 通过 location 指令，将请求（例如以 `php` 为后缀的文件）分配到指定端口（例如9000端口）来处理。
- FastCGI 进程管理器选择并连接到一个子进程，服务器将环境变量和标准输入发送给子进程。
- 子进程接受请求并完成处理后，将标准输出和错误信息从同一连接返回给服务器。
- 子进程关闭连接，继续等待下一个请求。

工作原理如图：

![](http://ww4.sinaimg.cn/large/61d238c7jw1eyr2zrcgzrj218e0c4ju1.jpg)

上图中的 `wrapper` 可以理解为用于启动另一个程序的程序，因为 Nginx 本身不支持对外部程序的直接调用或者解析，所以外部程序必须通过 FastCGI 接口来调用。

### 优点

除了继承 CGI 原有的优点之外， FastCGI 还有以下特点：

- 业务分离：FastCGI 后端和 Web Server 运行在不同的进程中，后端的故障不会导致 Web Server 停止服务。
- 分布式计算：由于 FastCGI 服务器是可以独立运行的，所以 FastCGI 程序可以在服务器以外的主机上执行，并且接受来自其它服务器的请求。
- 多个可扩展角色：在 FastCGI 中，程序被赋予明确的角色，例如响应器角色、认证器角色、过滤器角色等等。



## WSGI

### 概念

WSGI 是 Web 服务器和 Web 应用程序之间的一种简单而通用的接口，最初是为 Python 量身定做。

### 原理

WSGI 属于接口规范，从层级上来讲要比 CGI/FastCGI 高级。

WSGI 中存在两种角色：接受请求的 Server 和处理请求的 Application，它们底层是通过 FastCGI 沟通的。

WSGI 的工作流程如下：

- 客户端发起 HTTP 请求到 Nginx
- Nginx 转成 FastCGI 请求发送到 WSGI 的 Server（例如 flup）
- WSGI 的 Server 通过 app(environ, start_response) 函数调用 Application
- Application 再根据 environ 里的内容，对用户代码中注册的路由和响应函数进行调用。


#### Server

Server 端从规定的输入中获取 Request 数据，然后把环境变量（environ）和回调函数（start_response）传给 Application 。

#### Application

Application 会处理请求并通过回调函数将结果返回给 Server。

和 Serve 对应，一个标准的 Application 接受两个参数：

- environ：一个包含所有 HTTP 请求信息的 dict 对象
- start_response：一个发送 HTTP 响应的函数

#### Middleware

Middleware 是一个比较特殊的存在，它是夹在二者之间的，对于 Server 端而言它是个 Application ，而对于 Application 而言它就是 Server 端。

它可以实现以下功能：

- 重写环境变量，根据目标 URL，将请求消息路由到不同的应用对象。
- 允许在一个进程中同时运行多个应用程序或应用框架。
- 负载均衡和远程处理，通过在网络上转发请求和响应消息。
- 进行内容后处理，例如应用 XSLT 样式表。

### 优点

WSGI 将请求的工作通过异步回调进行拆解，可以很方便的在一个线程空间里同时处理多个请求。

另外，方便进行负载均衡和请求转发，不会造成后端应用阻塞。


## 其他

其他一些相关的名词一起梳理一下：

- uWSGI：一个应用服务器，实现了 uwsgi 、FastCGI 和 HTTP 等协议。
- uwsgi：uWSGI 服务器的自有协议，用于定义传输信息的类型。
- flup：WSGI 中的 Server 的一种实现
- Spawn-FCGI：Python 中的管理 FastCGI 进程管理器。
- PHP-CGI： PHP 自带的 FastCGI 管理器。
- PHP-FPM：PHP 的 Fastcgi 进程管理器。

OK 就是这样。对这方面不是很了解，如果有错误还望及时指出，不胜感激：）



***

参考文献：

- [Common Gateway Interface](https://en.wikipedia.org/wiki/Common_Gateway_Interface)
- [Simple Common Gateway Interface](https://en.wikipedia.org/wiki/Simple_Common_Gateway_Interface)
- [FastCGI](https://en.wikipedia.org/wiki/FastCGI)
- [Web Server Gateway Interface](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface)
- [浅谈 Node.js 和 PHP 进程管理](http://taobaofed.org/blog/2015/11/24/nodejs-php-process-manager/)
- [Linux PHP 的运行模式与其相关名词术语](http://www.2cto.com/os/201111/111318.html)
- [I never really understood: what is CGI?](http://stackoverflow.com/questions/2089271/i-never-really-understood-what-is-cgi)
- [FastCGI/drupal](http://www.fastcgi.com/drupal/)
- [fcgi，scgi，wsgi，cgi](http://zires.info/2011/01/fcgi-scgi-wsgi-cgi/)
- [如何理解 CGI WSGI](http://www.zhihu.com/question/19998865)
- [CGI Programming 101: Learn CGI Today!](http://www.cgi101.com/learn/)
- [RFC3875: The Common Gateway Interface (CGI) Version 1.1](http://tools.ietf.org/html/rfc3875)
- [FastCGI Specification](http://www.fastcgi.com/devkit/doc/fcgi-spec.html)
- [CGI、FastCGI 区别](http://ifxoxo.com/cgi_fastcgi.html)
- [Nginx+FastCGI 运行原理](http://book.51cto.com/art/201202/314840.htm)
- [从运行原理及使用场景看 Apache 和 Nginx](http://yansu.org/2014/02/15/apache-and-nginx.html)
- [漫谈 CGI FastCGI WSGI](http://blog.51reboot.com/cgi-fastcgi-wsgi/)
- [PEP 0333](https://www.python.org/dev/peps/pep-0333/)
- [WSGI 接口](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432012393132788f71e0edad4676a3f76ac7776f3a16000)
- [WSGI 学习](http://yansu.org/2013/05/19/what-is-wsgi.html)
- [Benchmark of Python WSGI Servers](http://nichol.as/benchmark-of-python-web-servers)
- [You Should Be Using Nginx + UWSGI](http://cramer.io/2013/06/27/serving-python-web-applications/)
- [Is there a speed difference between WSGI and FCGI?](http://stackoverflow.com/questions/1747266/is-there-a-speed-difference-between-wsgi-and-fcgi)
- [区分 wsgi、uWSGI、uwsgi、php-fpm、CGI、FastCGI 的概念](http://www.itopers.com/?p=586)
