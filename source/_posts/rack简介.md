---
title: rack简介
date: 2016-09-24 23:45:59
tags:
- Ruby
categories:
- 技术
---
rack简介。
<!-- more -->

# 什么是rack
rack是对ruby的Net::HTTP进行封装了的包，使用rack能够方便的新建一个简单的web应用。
## what is rack
Rack describes itself as follows:
> Rack在支持Ruby和Ruby框架的web服务间提供了一个最小接口。

Rack出现以前，Ruby的web框架都是实现自己的一套接口，导致实现web服务非常的困难，同时不同框架之间也很难共享代码。现在几乎所有的Ruby web框架都实现了Rack，包括Rails和Sinatra，也就是说，现在这些应用都以类似的方式运行。

Rack核心框架提供了大量的工具让你能够很简单的构建自己的web应用或接口。一行代码就可以实现一个Rack应用，但是我们需要更屌一点。

------------------------
**rack安装**
Rack的安装很简单：
```
gem install rack
```

## helloword举例
```
#config.ru文件内容
class Helloworld
  def call(env)
    [200, {'Content-Type' => 'text/html'}, ["Hello World!"]]
  end
end
run Helloworld.new
```
上面的代码和下面的lambda表达式功能是等价的：
```
run ->(env) { [200, {"Content-Type" => "text/html"}, ["Hello World!"]] }
```

当前路径下执行rackup
```
#rackup
[2016-09-24 10:34:45] INFO  WEBrick 1.3.1
[2016-09-24 10:34:45] INFO  ruby 1.9.3 (2014-11-13) [x86_64-linux]
[2016-09-24 10:34:45] INFO  WEBrick::HTTPServer#start: pid=125538 port=9292

```
浏览器访问http://localhost:9292 或执行curl  127.0.0.1:9292就能看到内容
```
# curl  127.0.0.1:9292
hello world
```
## Rack是怎么工作的
Rack有两个简单的要求。第一，你的请求必须有一个call方法响应，这就是为什么lambda或者Proc在这里可以用。call方法需要一个类似hash的参数，参数中包含了请求以及其他的环境参数。call方法返回包含三个元素的数组，[status, header, body]。status是对request的响应状态；header是一个哈希，内容是响应的头部信息；body是响应的消息体，必须是一个可遍历的对象，例如Array或者IO对象。

## 使用Rack提供的工具
Rack提供了大量的工具用于构建Ruby应用，所以我们尝试使用一个小工具，我们来增加些路由。
```
class HelloWorld
  def call(env)
    req = Rack::Request.new(env)
    case req.path_info
    when /hello/
      [200, {"Content-Type" => "text/html"}, ["Hello World!"]]
    when /goodbye/  
      [500, {"Content-Type" => "text/html"}, ["Goodbye Cruel World!"]]
    else
      [404, {"Content-Type" => "text/html"}, ["I'm Lost!"]]
    end
  end
end

run HelloWorld.new
```
执行结果：
```
# curl  127.0.0.1:9292/hello
Hello World!
# 
# curl  127.0.0.1:9292/goodbye
Goodbye Cruel World!

```
首先我们创建了一个Rack::Request对象，并将request中传递过来的env对象提供给Rack::Request对象。然后我们就可以使用path_info属性，根据其值进行我们的路由处理。
如果此时用浏览器访问http://localhost:9292，就会得到一个404错误页面。

## 让我们更屌一点
虽然上面的方法已经能够提供简单的路由功能，但是太挫，而且case也没法实现我们所有的路由功能。
我们更进一步，构建一个能够处理GET请求的小框架。
我们先来看看应用长什么样子，然后再一点点实现我们的框架。
```
#当前目录添加到ruby的加载路径中
$:.unshift File.dirname(__FILE__)
#require我们后面要实现的框架
require 'simple_framework0'

route("/hello") do
  "Hello #{params['name'] || "World"}!"
end

route("/goodbye") do
  status 500
  "Goodbye Cruel World!"
end

run SimpleFramework.app
```
simple_framework0.rb将是我们的框架实现代码，提供了一个route方法，route方法需要一个路径和块作为参数。如果请求能够和path匹配，提供的块将会被执行，块的最后一行代码是返回值，将作为response的body。

------------------------------------------------

看看我们的SimpleFramework长啥样。
```
# 后面将要实现的action，处理具体的请求
require 'action'
class SimpleFramework

  def self.app
    @app ||= begin
      Rack::Builder.new do
        map "/" do
          run ->(env) {[404, {'Content-Type' => 'text/plain'}, ['Page Not Found!']] }
        end
      end
    end
  end

end

def route(pattern, &block)
  SimpleFramework.app.map(pattern) do
    run Action.new(&block)
  end
end
```

------------------------------

下面是Action的实现。
```
class Action
  attr_reader :headers, :body, :request

  def initialize(&block)
    @block = block
    @status = 200
    @headers = {"Content-Type" => "text/html"}
    @body = ""
  end

  def status(value = nil)
    value ? @status = value : @status
  end

  def params
    request.params
  end

  def call(env)
    @request = Rack::Request.new(env)
    @body = self.instance_eval(&@block)
    [status, headers, [body]]
  end

end
```

----------------------------

整个的代码结构是：
```
tree
.
├── action.rb
├── config.ru
└── simple_framework0.rb
```

# 参考文档
[gist:4240848](https://gist.github.com/markbates/4240848)
