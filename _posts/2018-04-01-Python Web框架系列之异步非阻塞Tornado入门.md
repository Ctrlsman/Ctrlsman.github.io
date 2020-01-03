---
title: <Python Web框架系列>之异步非阻塞Tornado入门
---

## 简介

Tornado是一个Python web框架和异步网络库，最初是在FriendFeed开发的。通过使用非阻塞网络I/O, Tornado可以扩展到数以万计的开放连接，这使它成为长轮询、WebSockets和其他需要与每个用户进行长时间连接的应用程序的理想选择。   -----翻译自tornado官网


Tornado和现在的主流Web服务器框架（包括大多数 Python 的框架）有着明显的区别：它是非阻塞式服务器，而且速度相当快。得利于其 非阻塞的方式和对 epoll 的运用，Tornado 每秒可以处理数以千计的连接，这意味着对于实时 Web 服务来说，Tornado 是一个理想的 Web 框架。

#### 安装
```shell
pip3 install tornado==5.1.1
```

## 快速入门

```python
# run.py
import tornado.ioloop
import tornado.web
import time

class MainHandler(tornado.web.RequestHandler):
    def get(self):
				time.sleep(5)
        self.write("Hello, world")

class MessageHandler(tornado.web.RequestHandler):
		def get(self):
				self.write('message')

def make_app():
    return tornado.web.Application([
        (r"/", MainHandler),
        (r"/message", MessageHandler),
    ])

if __name__ == "__main__":
    app = make_app()
    app.listen(8888)
    tornado.ioloop.IOLoop.current().start()
```

#### 运行
```shell
python run.py
curl http://localhost:8888
>>> Hello World!
```

#### 执行过程
短短的几行代码就搭建了一个web服务,可谓神奇,那么就让我们一起来一探究竟,看看到底发生了什么

+ make_app函数通过Application类创建了一个app对象,大概看一下传入了对应url规则和处理逻辑的类
+ app对象监听8888端口
+ 通过IOLoop启动一个I/O事件循环,并开始监听请求(后续源码分析的时候会深度解析)

#### 疑问🤔️
有同学要问了:这TM异步在哪了?同时发起两个请求不还是阻塞的吗?别着急呀,人家tornado官方也说了,这个示例没有使用任何异步的功能.那么下面我们将上面的示例稍加修改变成一个异步的例子

```python
import tornado.ioloop
import tornado.web
from tornado import gen
from tornado.concurrent import Future
import time


class AsyncHandler(tornado.web.RequestHandler):
    @gen.coroutine
    def get(self):
        future = Future()
        tornado.ioloop.IOLoop.current().add_timeout(time.time() + 10, self.done)
        yield future

    def done(self):
        self.write('this is async request')
        self.finish()


class Message(tornado.web.RequestHandler):
    def get(self):
        self.write(str(int(time.time())))


application = tornado.web.Application([
    (r"/index", AsyncHandler),
    (r"/message", Message),
])


if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()

```

这样再同时访问/index和/message你试试它香不香?
当发送GET请求时，由于方法被@gen.coroutine装饰且yield 一个 Future对象，那么Tornado会等待，等待用户向future对象中放置数据或者发送信号，如果获取到数据或信号之后，就开始执行done方法。

异步非阻塞体现在当在Tornaod等待用户向future对象中放置数据时，还可以处理其他请求。

注意：在等待用户向future对象中放置数据或信号时，此连接是不断开的。


## 路由

路由说白了就是URL和处理逻辑类的对应关系,支持正则表达式

## 模板引擎

Tornao中的模板语言和django中类似，模板引擎将模板文件载入内存，然后将数据嵌入其中，最终获取到一个完整的字符串，再将字符串返回给请求者。

Tornado 的模板支持“控制语句”和“表达语句”，控制语句是使用 {% 和 %} 包起来的 例如 {% if len(items) > 2 %}。表达语句是使用 {{ 和 }} 包起来的，例如 {{ items[0] }}。

控制语句和对应的 Python 语句的格式基本完全相同。我们支持 if、for、while 和 try，这些语句逻辑结束的位置需要用 {% end %} 做标记。还通过 extends 和 block 语句实现了模板继承。这些在 template 模块 的代码文档中有着详细的描述。

**注：在使用模板前需要在setting中设置模板路径："template_path" : "tpl"**


```html
<!DOCTYPE html>
    <div>
        <ul>
            {% for item in list_info %}
                <li>{{item}}</li>
            {% end %}
        </ul>
    </div>
```

在模板中默认提供了一些函数、字段、类以供模板使用：

+ escape: tornado.escape.xhtml_escape 的別名
+ xhtml_escape: tornado.escape.xhtml_escape 的別名
+ url_escape: tornado.escape.url_escape 的別名
+ json_encode: tornado.escape.json_encode 的別名
+ squeeze: tornado.escape.squeeze 的別名
+ linkify: tornado.escape.linkify 的別名
+ datetime: Python 的 datetime 模组
+ handler: 当前的 RequestHandler 对象
+ request: handler.request 的別名
+ current_user: handler.current_user 的別名
+ locale: handler.locale 的別名
+ _: handler.locale.translate 的別名
+ static_url: for handler.static_url 的別名
+ xsrf_form_html: handler.xsrf_form_html 的別名


### 母版和母版导入
```html

<!--母版定义-->
{% extends 'layout.html'%}
{% block CSS %}
    <link href="{{static_url("css/index.css")}}" rel="stylesheet" />
{% end %}

{% block RenderBody %}
    <h1>Index</h1>

    <ul>
    {%  for item in li %}
        <li>{{item}}</li>
    {% end %}
    </ul>

{% end %}

{% block JavaScript %}
    
{% end %}


<!--导入母版-->
{% include 'header.html' %}

```

### UIMethod和UIModule

1. 定义
```python
# uimethods.py
def tab(self):
    return 'UIMethod'


#uimodules.py
from tornado.web import UIModule
from tornado import escape

class custom(UIModule):

    def render(self, *args, **kwargs):
        return escape.xhtml_escape('<h1>Hello!</h1>')
```

2. 注册

```python
import tornado.ioloop
import tornado.web
from tornado.escape import linkify
import uimodules as md
import uimethods as mt

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.render('index.html')

settings = {
    'template_path': 'template',
    'static_path': 'static',
    'static_url_prefix': '/static/',
    'ui_methods': mt,
    'ui_modules': md,
}

application = tornado.web.Application([
    (r"/index", MainHandler),
], **settings)


if __name__ == "__main__":
    application.listen(8009)
    tornado.ioloop.IOLoop.instance().start()
```

3. 使用
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <link href="{{static_url("commons.css")}}" rel="stylesheet" />
</head>
<body>
    <h1>hello</h1>
    {% module custom(123) %}
    {{ tab() }}
</body>
```

## 静态文件

可以配置静态文件的目录和前端使用时的前缀，并且Tornaodo还支持静态文件缓存

```python
settings = {
    'template_path': 'template',
    'static_path': 'static',
    'static_url_prefix': '/static/',
}
```

为了提高性能，在浏览器主动缓存静态文件是个不错的主意。这样浏览器就不需要发送 不必要的 If-Modified-Since 和 Etag 请求，从而影响页面的渲染速度。 Tornado 可以通过内建的“静态内容分版(static content versioning)”来直接支持这种功能。

要使用这个功能，在模板中就不要直接使用静态文件的 URL 地址了，你需要在 HTML 中使用 static_url() 这个方法来提供 URL 地址:
```html
<html>
   <head>
      <title>FriendFeed - {{ _("Home") }}</title>
   </head>
   <body>
     <div><img src="{{ static_url("images/logo.png") }}"/></div>
   </body>
 </html> 
 ```
 static_url() 函数会将相对地址转成一个类似于 /static/images/logo.png?v=aae54 的 URI，v 参数是 logo.png 文件的散列值， Tornado 服务器会把它发给浏览器，并以此为依据让浏览器对相关内容做永久缓存。

由于 v 的值是基于文件的内容计算出来的，如果你更新了文件，或者重启了服务器 ，那么就会得到一个新的 v 值，这样浏览器就会请求服务器以获取新的文件内容。 如果文件的内容没有改变，浏览器就会一直使用本地缓存的文件，这样可以显著提高页 面的渲染速度。

```python
def get_content_version(cls, abspath):
        """Returns a version string for the resource at the given path.

        This class method may be overridden by subclasses.  The
        default implementation is a hash of the file's contents.
        """
        data = cls.get_content(abspath)
        hasher = hashlib.md5()
        if isinstance(data, bytes):
            hasher.update(data)
        else:
            for chunk in data:
                hasher.update(chunk)
        return hasher.hexdigest()
```

## Cookie
Tornado中可以对cookie进行操作，并且还可以对cookie进行签名以放置伪造

```python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        if not self.get_cookie("mycookie"):
            self.set_cookie("mycookie", "myvalue")
            self.write("Your cookie was not set yet!")
        else:
            self.write("Your cookie was set!")
```
Cookie 很容易被恶意的客户端伪造。加入你想在 cookie 中保存当前登陆用户的 id 之类的信息，你需要对 cookie 作签名以防止伪造。Tornado 通过 set_secure_cookie 和 get_secure_cookie 方法直接支持了这种功能。 要使用这些方法，你需要在创建应用时提供一个密钥，名字为 cookie_secret。 你可以把它作为一个关键词参数传入应用的设置中：
```python
class MainHandler(tornado.web.RequestHandler):
    def get(self):
        if not self.get_secure_cookie("mycookie"):
            self.set_secure_cookie("mycookie", "myvalue")
            self.write("Your cookie was not set yet!")
        else:
            self.write("Your cookie was set!")
             
application = tornado.web.Application([
    (r"/", MainHandler),
], cookie_secret="61oETzKXQAGaYdkL5gEmGeJJFuYh7EQnp2XdTP1o/Vo=")
```

## CSRF
Tornado中的夸张请求伪造和Django中的相似

1. 配置
```python
settings = {
    "xsrf_cookies": True,
}
application = tornado.web.Application([
    (r"/", MainHandler),
    (r"/login", LoginHandler),
], **settings)
```

2. 基本使用
```html
<form action="/new_message" method="post">
  {{ xsrf_form_html() }}
  <input type="text" name="message"/>
  <input type="submit" value="Post"/>
</form>
```

3. Ajax使用
```html
function getCookie(name) {
    var r = document.cookie.match("\\b" + name + "=([^;]*)\\b");
    return r ? r[1] : undefined;
}

jQuery.postJSON = function(url, args, callback) {
    args._xsrf = getCookie("_xsrf");
    $.ajax({url: url, data: $.param(args), dataType: "text", type: "POST",
        success: function(response) {
        callback(eval("(" + response + ")"));
    }});
};
```

## 异步客户端AsyncHTTPClient

Tornado提供了httpclient类库用于发送Http请求

```python
import tornado.web
from tornado import gen
from tornado import httpclient
 
# 方式一：
class AsyncHandler(tornado.web.RequestHandler):
    @gen.coroutine
    def get(self, *args, **kwargs):
        http = httpclient.AsyncHTTPClient()
        data = yield http.fetch("http://www.google.com")
        self.finish('over')
 
# 方式二：
class AsyncHandler(tornado.web.RequestHandler):
    @gen.coroutine
    def get(self):
        http = httpclient.AsyncHTTPClient()
        yield http.fetch("http://www.google.com", self.done)

    def done(self, response):
        self.finish('over')
 
 
 
application = tornado.web.Application([
    (r"/async", AsyncHandler),
])
 
if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```
