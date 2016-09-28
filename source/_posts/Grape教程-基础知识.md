---
title: Grape简介
date: 2016-09-27 00:27:44
tags:
- Ruby
- Grape
- RESTful
categories:
- 技术
---
Grape介绍。
<!-- more -->
# 什么是Grape
Grape是Ruby中的一个类REST API框架，被设计用于运行在Rack上或弥补已有的web应用框架（比如Rails或者Sinatra），Grape提供了一个简单的DSL用于方便的开发RESTful APIs。Grape支持common conventions，包括多种格式，子域/前缀限制，内容协商，版本控制等。

# 安装
通过gem安装：
```
gem install grape
```

# 基本用法
你的应用一般要继承自Grape::API，下面的例子暂时了Grape中常用的一些特性：
```
#config.ru 内容
require "grape"

module Twitter
  class API < Grape::API
    version 'v1', using: :header, vendor: 'twitter'
    format :json
    prefix :api

    helpers do
      def current_user
        @current_user ||= User.authorize!(env)
      end

      def authenticate!
        error!('401 Unauthorized', 401) unless current_user
      end
    end

    resource :statuses do
      desc 'Return a public timeline.'
      get :public_timeline do
        Status.limit(20)
      end

      desc 'Return a personal timeline.'
      get :home_timeline do
        authenticate!
        current_user.statuses.limit(20)
      end

      desc 'Return a status.'
      params do
        requires :id, type: Integer, desc: 'Status id.'
      end
      route_param :id do
        get do
          Status.find(params[:id])
        end
      end

      desc 'Create a status.'
      params do
        requires :status, type: String, desc: 'Your status.'
      end
      post do
        authenticate!
        Status.create!({
          user: current_user,
          text: params[:status]
        })
      end

      desc 'Update a status.'
      params do
        requires :id, type: String, desc: 'Status ID.'
        requires :status, type: String, desc: 'Your status.'
      end
      put ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).update({
          user: current_user,
          text: params[:status]
        })
      end

      desc 'Delete a status.'
      params do
        requires :id, type: String, desc: 'Status ID.'
      end
      delete ':id' do
        authenticate!
        current_user.statuses.find(params[:id]).destroy
      end
    end
  end
end

run Twitter::API
```


# 安装
## Rack
上面的例子创建了一个简单的Rack应用，将上面的内容放在config.ru文件中，通过如下命令启动：
```
rackup -o 0.0.0.0
```
*在我的环境中直接执行rackup之后，在其他机器上无法访问*
现在可以访问如下路径：
```
GET /api/statuses/public_timeline
GET /api/statuses/home_timeline
GET /api/statuses/:id
POST /api/statuses
PUT /api/statuses/:id
DELETE /api/statuses/:id
```
所有的GET选项，Grape也会自动应答HEAD/OPTIONS选项，但是其他路径，只会应答OPTIONS选项。

# 脱离Rails使用ActiveRecord
如果你想在Grape中使用ActiveRecord，你要保证ActiveRecord的连接池处理正确。保证这一点的最简单的方式是在安装你的Grape应用前使用ActiveRecord的ConnectionManagement中间件：
```
use ActiveRecord::ConnectionAdapters::ConnectionManagement

run Twitter::API
```

# 结合Sinatra(或者其他框架)
如果使用Grape的同时想结合使用其他Rack框架，比如Sinatra，你可以很简单的使用Rack::Cascade：
```
# Example config.ru

require 'sinatra'
require 'grape'

class API < Grape::API
  get :hello do
    { hello: 'world' }
  end
end

class Web < Sinatra::Base
  get '/' do
    'Hello world.'
  end
end

use Rack::Session::Cookie
run Rack::Cascade.new [API, Web]
```
# Rails
把所有的API文件都放置到app/api路径下，Rails期望一个子文件夹能够匹配一个Ruby模块，一个文件名能够匹配一个类名。在我们的例子中，Twitter::API的路径和文件名应该是app/api/twitter/api.rb。
修改application.rb：
```
config.paths.add File.join('app', 'api'), glob: File.join('**', '*.rb')
config.autoload_paths += Dir[Rails.root.join('app', 'api', '*')]
```
修改config/routes：
```
mount Twitter::API => '/'
```
另外，如果使用的Rails是4.0+，并且使用了ActiveRecord默认的model层，你可能会想使用 hashie-forbidden_attributes gem包。该gem去使能model层strong_paras的安全特性，允许你使用Grape自己的参数校验。
```
# Gemfile
gem 'hashie-forbidden_attributes'
```

# Modules
在一个应用内部你可以mount多个其他应用的API实现，它们不需要是不同的版本，但是可能是同一接口的组件：
```
class Twitter::API < Grape::API
  mount Twitter::APIv1
  mount Twitter::APIv2
end
```
你也可以挂在到一个路径上，就好像在被mount的API内部使用了prefix一样：
```
class Twitter::API < Grape::API
  mount Twitter::APIv1 => '/v1'
end
```

# 版本控制
版本信息可以通过四种方式提供：
```
:path、:header、:accept_version_header、:param
```
默认的方式是:path。

## path
```
version 'v1', using: :path
```
通过这种方式，URL中必须提供版本信息：
```
curl http://localhost:9292/v1/statuses/public_timeline
```
## Header
```
version 'v1', using: :header, vendor: 'twitter'
```
现在，Grape仅支持如下格式的版本控制信息：
```
vnd.vendor-and-or-resource-v1234+format
```
最后的-和+之间的所有信息都会被解释为版本信息（上面的版本信息就是v1234）。
通过这种方式，用户必须通过HTTP的Accept头传递版本信息：
```
curl -H Accept:application/vnd.twitter-v1+json http://localhost:9292/statuses/public_timeline
```
默认的，如果没有提供Accept头信息，将会使用第一个匹配的版本，这种行为和Rails中的路径相似。为了规避这个默认的行为，可以使用:strict选项，该选项被设置为true时，如果没有提供正确的Accept头信息，会返回一个406错误。
当提供了一个无效的Accept头时，如果:cascade选项设置为false，会返回一个406错误。否则，如果没有其他的路径匹配，Rack会返回一个404错误。

## Accept-Version Header
```
version 'v1', using: :accept_version_header
```
使用该方式时，用户必须在HTTP头中提供Accept-Version信息：
```
curl -H "Accept-Version:v1" http://localhost:9292/statuses/public_timeline
```
默认的，如果没有提供Accept头信息，将会使用第一个匹配的版本，这种行为和Rails中的路径相似。为了规避这个默认的行为，可以使用:strict选项，该选项被设置为true时，如果没有提供正确的Accept头信息，会返回一个406错误。
当提供了一个无效的Accept头时，如果:cascade选项设置为false，会返回一个406错误。否则，如果没有其他的路径匹配，Rack会返回一个404错误。

## Param
```
version 'v1', using: :param
```
使用这种方式，用于必须在URL字符串或者请求的body中提供版本信息：
```
curl http://localhost:9292/statuses/public_timeline?apiver=v1
```
参数名称默认是“apiver”，你也可以通过:parameter 选项指定：
```
version 'v1', using: :param, parameter: 'v'
curl http://localhost:9292/statuses/public_timeline?v=v1
```

# 描述方法
你可以为API方法和名称空间添加一个描述:
```
desc 'Returns your public timeline.' do
  detail 'more details'
  params  API::Entities::Status.documentation
  success API::Entities::Entity
  failure [[401, 'Unauthorized', 'Entities::Error']]
  named 'My named route'
  headers XAuthToken: {
            description: 'Valdates your identity',
            required: true
          },
          XOptionalHeader: {
            description: 'Not really needed',
            required: false
          }

end
get :public_timeline do
  Status.limit(20)
end
```
* detail
更详细的描述。
* params
直接从一个Entity定义参数。
* success: (former entity) The Entity to be used to present by default this route
* failure: (former http_codes) A definition of the used failure HTTP Codes and Entities
* named: A helper to give a route a name and find it with this name in the documentation Hash
* headers: A definition of the used Headers

