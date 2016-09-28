---
title: Grape教程-params
date: 2016-09-28 22:25:43
tags:
- Ruby
- Grape
- RESTful
categories:
- 技术
---
params介绍。
<!-- more -->
# 参数
请求参数可以通过params获取，params是一个hash对象，包括GET、POST、PUT参数，以及路径字符串中的任何命名参数：
```
get :public_timeline do
  Status.order(params[:sort_by])
end
```
Parameters are automatically populated from the request body on POST and PUT for form input, JSON and XML content-types.
请求：
```
curl -d '{"text": "140 characters"}' 'http://localhost:9292/statuses' -H Content-Type:application/json -v
```
Grape中：
```
post '/statuses' do
  Status.create!(text: params[:text])
end
```
多部分的POSTs和PUTs也是支持的。
请求：
```
curl --form image_file='@image.jpg;type=image/jpg' http://localhost:9292/upload
```
Grape中：
```
post 'upload' do
  # file in params[:image_file]
end
```
在以下任何一个或两个有冲突时：
* 路径字符串参数
* GET/POST/PUT参数
* POST/PUT请求正文的内容
路径字符串中的将生效。

## 声明
Grape只允许访问在params块中声明的变量，它过滤掉传递过来但是不允许访问的变量，请看下面的API：
```
format :json

post 'users/signup' do
  { 'declared_params' => declared(params) }
end
```
如果我们没有指定任何参数，declared会返回一个空的Hashie::Mash实例。
请求：
```
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "last_name": "last name"}}'
```
应答：
```
{
  "declared_params": {}
}
```
一旦我们添加了参数requirements，Grape仅返回声明的参数:
```
format :json

params do
  requires :user, type: Hash do
    requires :first_name, type: String
    requires :last_name, type: String
  end
end

post 'users/signup' do
  { 'declared_params' => declared(params) }
end
```
请求：
```
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "last_name": "last name", "random": "never shown"}}'
```
应答：
```
{
  "declared_params": {
    "user": {
      "first_name": "first name",
      "last_name": "last name"
    }
  }
}
```
返回的hash是Hashie::Mash实例，允许你使用.访问参数：
```
  declared(params).user == declared(params)['user']
```
The #declared method is not available to before filters, as those are evaluated prior to parameter coercion.

### 包含父命名空间
默认的，declared(params)包含了在父命名空间中声明的参数，如果你只向返回当前命名空间的参数，可以把include_parent_namespaces选项设置为false。
```
format :json

namespace :parent do
  params do
    requires :parent_name, type: String
  end

  namespace ':parent_name' do
    params do
      requires :child_name, type: String
    end
    get ':child_name' do
      {
        'without_parent_namespaces' => declared(params, include_parent_namespaces: false),
        'with_parent_namespaces' => declared(params, include_parent_namespaces: true),
      }
    end
  end
end
```
请求：
```
curl -X GET -H "Content-Type: application/json" localhost:9292/parent/foo/bar
```
应答：
```
{
  "without_parent_namespaces": {
    "child_name": "bar"
  },
  "with_parent_namespaces": {
    "parent_name": "foo",
    "child_name": "bar"
  },
}
```

### Include missing
默认的，declared(params)包含值为nil的参数，如果你只想返回值为非nil的参数，你可以使用include_missing 选项。该选项默认为true，看下下面的API：
```
format :json

params do
  requires :first_name, type: String
  optional :last_name, type: String
end

post 'users/signup' do
  { 'declared_params' => declared(params, include_missing: false) }
end
```
请求：
```
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "random": "never shown"}}'
```
include_missing:false时：
```
{
  "declared_params": {
    "user": {
      "first_name": "first name"
    }
  }
}
```
include_missing:true时：
```
{
  "declared_params": {
    "first_name": "first name",
    "last_name": null
  }
}
```
在嵌套的hash中也会生效：
```
format :json

params do
  requires :user, type: Hash do
    requires :first_name, type: String
    optional :last_name, type: String
    requires :address, type: Hash do
      requires :city, type: String
      optional :region, type: String
    end
  end
end

post 'users/signup' do
  { 'declared_params' => declared(params, include_missing: false) }
end
```
请求：
```
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "random": "never shown", "address": { "city": "SF"}}}'
```
include_missing:false时：
```
{
  "declared_params": {
    "user": {
      "first_name": "first name",
      "address": {
        "city": "SF"
      }
    }
  }
}
```
 include_missing:true时
 ```
{
  "declared_params": {
    "user": {
      "first_name": "first name",
      "last_name": null,
      "address": {
        "city": "Zurich",
        "region": null
      }
    }
  }
}
 ```
 注意，值被设置为nil的变量和缺失是不同的，include_missing被设置为false时，值为nil的变量也会返回。
 请求：
 ```
curl -X POST -H "Content-Type: application/json" localhost:9292/users/signup -d '{"user": {"first_name":"first name", "last_name": null, "address": { "city": "SF"}}}'
 ```
 include_missing:false时的应答：
 ```
{
  "declared_params": {
    "user": {
      "first_name": "first name",
      "last_name": null,
      "address": { "city": "SF"}
    }
  }
}
 ```

## 参数校验和强转
你可以通过一个params block为参数定义校验选项：
```
params do
  requires :id, type: Integer
  optional :text, type: String, regexp: /\A[a-z]+\z/
  group :media do
    requires :url
  end
  optional :audio do
    requires :format, type: Symbol, values: [:mp3, :wav, :aac, :ogg], default: :mp3
  end
  mutually_exclusive :media, :audio
end
put ':id' do
  # params[:id] is an Integer
end
```

如果指定了参数类型，在参数强转后，会对强转后的参数进行隐式的校验，以保证转换后的类型是声明的类型。（When a type is specified an implicit validation is done after the coercion to ensure the output type is the one declared.）
可选参数可以提供一个默认值：
```
params do
  optional :color, type: String, default: 'blue'
  optional :random_number, type: Integer, default: -> { Random.rand(1..100) }
  optional :non_random_number, type: Integer, default:  Random.rand(1..100)
end
```
注意，默认值要满足所有的校验条件，如果没有显示提供值，下面的例子总是会失败：
```
params do
  optional :color, type: String, default: 'blue', values: ['red', 'green']
end
```
### 支持的参数类型
* Integer
* Float
* BigDecimal
* Numeric
* Date
* DateTime
* Time
* Boolean
* String
* Symbol
* Rack::Multipart::UploadedFile (alias File)
* JSON

### 自定义类型和转换
除了上面列出的类型，如果提供了显示的转换方法parse，任何类都可以当做类型使用。如果提供的是类方法，Grape会自动使用，该方法入参必须是一个字符串，并且返回一个正确的类型，或者抛出异常表示该值无效：
```
class Color
  attr_reader :value
  def initialize(color)
    @value = color
  end

  def self.parse(value)
    fail 'Invalid color' unless %w(blue red green).include?(value)
    new(value)
  end
end

# ...

params do
  requires :color, type: Color, default: Color.new('blue')
end

get '/stuff' do
  # params[:color] is already a Color.
  params[:color].value
end
```
另外，通过coerce_with可以为任意类型提供自定义的转换方法，任何实现了parse或者call方法的类或对象都可以，该方法必须接受一个字符串参数，返回类型必须能够和type给出的匹配：
```
params do
  requires :passwd, type: String, coerce_with: Base64.method(:decode)
  requires :loud_color, type: Color, coerce_with: ->(c) { Color.parse(c.downcase) }

  requires :obj, type: Hash, coerce_with: JSON do
    requires :words, type: Array[String], coerce_with: ->(val) { val.split(/\s+/) }
    optional :time, type: Time, coerce_with: Chronic
  end
end
```
下面的例子为corece_with提供了一个lambda表达式，该表达式接受一个string类型参数，返回一个整形数组，以匹配Array[Integer]类型：
```
params do
  requires :values, type: Array[Integer], coerce_with: ->(val) { val.split(/\s+/).map(&:to_i) }
end
```
### MultipatFile参数
Grape利用Rack::Request内置的对MultipatFile参数的支持，这种类型的参数可以被声明为type: File：
```
params do
  requires :avatar, type: File
end
post '/' do
  # Parameter will be wrapped using Hashie:
  params.avatar.filename # => 'avatar.png'
  params.avatar.type     # => 'image/png'
  params.avatar.tempfile # => #<File>
end
```

### JSON类型
Grape支持JSON格式的复杂类型参数，使用type: JSON声明，JSON对象和数组都可以被接受，在两种情况下，内置的校验规则对所有的对象（JSON对象或JSON数组中的所有对象）都适用。（Grape supports complex parameters given as JSON-formatted strings using the special type: JSON declaration. JSON objects and arrays of objects are accepted equally, with nested validation rules applied to all objects in either case）：
```
params do
  requires :json, type: JSON do
    requires :int, type: Integer, values: [1, 2, 3]
  end
end
get '/' do
  params[:json].inspect
end

# ...

client.get('/', json: '{"int":1}') # => "{:int=>1}"
client.get('/', json: '[{"int":"1"}]') # => "[{:int=>1}]"

client.get('/', json: '{"int":4}') # => HTTP 400
client.get('/', json: '[{"int":4}]') # => HTTP 400
```
另外，也可以使用type: Array[JSON]，用以明确表示被传递进来的参数是一个数组，如果只传递进来的是非数组形式的单个对象，会将其转换成数组。
```
params do
  requires :json, type: Array[JSON] do
    requires :int, type: Integer
  end
end
get '/' do
  params[:json].each { |obj| ... } # always works
end
```
For stricter control over the type of JSON structure which may be supplied, use type: Array, coerce_with: JSON or type: Hash, coerce_with: JSON.

### 允许多种类型
多类型参数可以使用types来声明，而不是使用type：
```
params do
  requires :status_code, types: [Integer, String, Array[Integer, String]]
end
get '/' do
  params[:status_code].inspect
end

# ...

client.get('/', status_code: 'OK_GOOD') # => "OK_GOOD"
client.get('/', status_code: 300) # => 300
client.get('/', status_code: %w(404 NOT FOUND)) # => [404, "NOT", "FOUND"]
```
作为一个特例，通过传递包含多个成员的Set或者Array给type,可以声明多成员类型：
```
params do
  requires :status_codes, type: Array[Integer,String]
end
get '/' do
  params[:status_codes].inspect
end

# ...

client.get('/', status_codes: %w(1 two)) # => [1, "two"]
```
### 校验嵌套的参数
通过group或者为requires或者optinal提供block可以使用嵌套参数：
```
params do
  requires :id, type: Integer
  optional :text, type: String, regexp: /\A[a-z]+\z/
  group :media do
    requires :url
  end
  optional :audio do
    requires :format, type: Symbol, values: [:mp3, :wav, :aac, :ogg], default: :mp3
  end
  mutually_exclusive :media, :audio
end
put ':id' do
  # params[:id] is an Integer
end
```
上面例子的意思是，需要同时提供`params[:media][:url]` 和 `params[:id]`，提供`params[:audio]`时才需要提供`params[:audio][:format]`。提供block时，group、requires、optional接受值可以是Array或者Hash的type（默认为Array）。Depending on the value, the nested parameters will be treated either as values of a hash or as values of hashes in an array.
```
params do
  optional :preferences, type: Array do
    requires :key
    requires :value
  end

  requires :name, type: Hash do
    requires :first_name
    requires :last_name
  end
end
```

### 参数依赖
对于给定一个参数时，必须提供另一个参数的情况，可以使用given方法：
```
params do
  optional :shelf_id, type: Integer
  given :shelf_id do
    requires :bin_id, type: Integer
  end
end
```
在上面的例子中，Grape会使用blank?方法检查是否提供了shelf_id参数。
give也接受一个自定义的Proc，下面的例子，description参数只有在category 的值是“foo”的时候才需要提供：
```
params do
  optional :category
  given category: ->(val) { val == 'foo' } do
    requires :description
  end
end
```

### 内置的验证方法
#### allow_blank
参数定义中也可以使用allow_blank，用以保证参数有值。默认情况下，requires只检查请求中包含了指定的参数，但是忽略它的值，通过指定allow_blank: false，空值和只包含空白符的值将是无效的。
allow_blank可以和requires/optional结合使用，如果参数是必须的，那么就必须包含一个值；如果是可选的，那么请求中如果含有这个参数时，它的值必须不能是空字符串或者空白符。
```
params do
  requires :username, allow_blank: false
  optional :first_name, allow_blank: false
end
```

#### values
通过:values选项，参数可以被限制为一组特定的值。
Default values are eagerly evaluated. Above :non_random_number will evaluate to the same number for each call to the endpoint of this params block. To have the default evaluate lazily with each request use a lambda, like :random_number above.
```
params do
  requires :status, type: Symbol, values: [:not_started, :processing, :done]
  optional :numbers, type: Array[Integer], default: 1, values: [1, 2, 3, 5, 8]
end
```
可以给:values选项提供一个范围参数：
```
params do
  requires :latitude, type: Float, values: -90.0..+90.0
  requires :longitude, type: Float, values: -180.0..+180.0
  optional :letters, type: Array[String], values: 'a'..'z'
end
```
注意，range的起始值类型必须和:type指定的匹配（如果没有提供:type选项，结束值的类型和开始值的类型必须相同），下面的例子是非法的：
```
params do
  requires :invalid1, type: Float, values: 0..10 # 0.kind_of?(Float) => false
  optional :invalid2, values: 0..10.0 # 10.0.kind_of?(0.class) => false
end
```
也可以给:values选项提供一个Proc对象，在每个请求中进行求值：
```
params do
  requires :hashtag, type: String, values: -> { Hashtag.all.map(&:tag) }
end
```
也可以通过except选项来限制不能包含哪些值，except接受相同类型的参数作为值（Procs, ranges，等）：
```
params do
  requires :browsers, values: { except: [ 'ie6', 'ie7', 'ie8' ] }
end
```
values和except可以结合使用，用以指定哪些值能接受，哪些不能接受。可以分别为except和value自定义不同的错误消息，用于指定值落在了except中或者不在value中：
```
params do
  requires :number, type: Integer, values: { value: 1..20 except: [4,13], except_message: 'includes unsafe numbers', message: 'is outside the range of numbers allowed' }
end
```
#### 正则表达式
通过:regexp选项，可以将参数限制为和正则表达式匹配，如果不匹配将会返回一个错误，正则表达式选项对requires和optinal参数都有效。
```
params do
  requires :email, regexp: /.+@.+/
end
```
当参数不包含值时，校验是能通过的。为了保证参数包含值，可以使用alow_blank: false。
```
params do
  requires :email, allow_blank: false, regexp: /.+@.+/
end
```

#### 互斥
参数可以通过mutually_exclusive定义为互斥，保证不会出现在同一个请求中：
```
params do
  optional :beer
  optional :wine
  mutually_exclusive :beer, :wine
end
```
可以定义多组：
```
params do
  optional :beer
  optional :wine
  mutually_exclusive :beer, :wine
  optional :scotch
  optional :aquavit
  mutually_exclusive :scotch, :aquavit
end
```
**警告：永远不要将两个必须的参数设置为互斥，否则将导致参数永远无效；一个必须的参数和一个可选的参数互斥，会导致后一个参数用于无效**

#### 正好有一个
通过exactly_one_of可以指定正好有一个参数被提供：
```
params do
  optional :beer
  optional :wine
  exactly_one_of :beer, :wine
end
```
#### 至少有一个
at_least_one_of选项保证至少有一个参数被提供：
```
params do
  optional :beer
  optional :wine
  optional :juice
  at_least_one_of :beer, :wine, :juice
end
```

#### 都提供或都不提供
可以通过all_or_none_of选项指定所有参数都提供或都不提供。
```
params do
  optional :beer
  optional :wine
  optional :juice
  all_or_none_of :beer, :wine, :juice
end
```

#### 嵌套的mutually_exclusive, exactly_one_of, at_least_one_of, all_or_none_of
所有的这些方法都一个在任何嵌套层使用：
```
params do
  requires :food do
    optional :meat
    optional :fish
    optional :rice
    at_least_one_of :meat, :fish, :rice
  end
  group :drink do
    optional :beer
    optional :wine
    optional :juice
    exactly_one_of :beer, :wine, :juice
  end
  optional :dessert do
    optional :cake
    optional :icecream
    mutually_exclusive :cake, :icecream
  end
  optional :recipe do
    optional :oil
    optional :meat
    all_or_none_of :oil, :meat
  end
end
```
### 命名空间校验和转换
命名空间允许在每个方法中通过命名空间定义和使用参数。
```
namespace :statuses do
  params do
    requires :user_id, type: Integer, desc: 'A user ID.'
  end
  namespace ':user_id' do
    desc "Retrieve a user's status."
    params do
      requires :status_id, type: Integer, desc: 'A status ID.'
    end
    get ':status_id' do
      User.find(params[:user_id]).statuses.find(params[:status_id])
    end
  end
end
```
namespace方法有好几个别名，包括：group, resource, resources, and segment。你可以选择一个你喜欢的。
通过route_param你可以方便的定义一个路由参数作为命名空间：
```
namespace :statuses do
  route_param :id do
    desc 'Returns all replies for a status.'
    get 'replies' do
      Status.find(params[:id]).replies
    end
    desc 'Returns a status.'
    get do
      Status.find(params[:id])
    end
  end
end
```
You can also define a route parameter type by passing to route_param's options.
```
namespace :arithmetic do
  route_param :n, type: Integer do
    desc 'Returns in power'
    get 'power' do
      params[:n] ** params[:n]
    end
  end
end
```