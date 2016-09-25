---
title: Sequel简介
date: 2016-09-25 13:20:39
tags:
- ruby
categories:
- 技术
---
Sequel README翻译。
<!-- more -->

# Sequel: Ruby数据库工具包
## 简介
Sequel是Ruby中用于访问SQL数据库的一个简单、灵活、强大的工具包。
* Sequel能够保证线程安全，提供了连接池功能以及简洁的SDL用于创建SQL查询及表定义；
* Sequel包括一个强大的ORM层用于映射数据库记录和Ruby对象以及相关的记录；
* Sequel提供一些高级的数据库特写，比如，预处理的语句，绑定变量、存储过程、事务、两阶段提交、事务隔离、主/从结构及数据库分片。
* Sequel现在可以适配ADO, Amalgalite, CUBRID, DataObjects, IBM_DB, JDBC, MySQL, Mysql2, ODBC, Oracle, PostgreSQL, SQLAnywhere, SQLite3, Swift, and TinyTDS。

Sequel被设计用来简单的连接和操作数据库，Sequel处理所有繁杂的事情，比如，保持连接、使用正确的SQL格式以及获取数据，这样你就可以专注于自己的应用了。
Sequel使用数据库的概念来检索数据。一个数据集对象封装了一个SQL查询并且支持关联（原文是supports chainability，意思应该是关联），使你能够通过简洁灵活的Ruby DSL来方便的获取数据。
例如，下面的一行代码就可以返回中东地区国家的平均GDP：
```
DB[:countries].filter(:region => 'Middle East').avg(:GDP)
```
这条语句等价于：
```
SELECT avg(GDP) FROM countries WHERE region = 'Middle East'
```
由于数据库只有在需要的时候才取回数据，所以被缓存以便再使用。记录是作为Hash类型返回的，所以可以使用一个可枚举的接口访问：
```
middle_east = DB[:countries].filter(:region => 'Middle East')
middle_east.order(:name).each{|r| puts r[:name]}
```
Sequel还提供了简便的方法用于从数据集中提取数据，比如一个扩展的map方法：
```
middle_east.map(:name) #=> ['Egypt', 'Turkey', 'Israel', ...]
```
或者是通过.to_hash方法把结果作为一个hash，一列作为kye，另一列作为value:
```
middle_east.to_hash(:name, :area) #=> {'Israel' => 20000, 'Turkey' => 120000, ...}
```


## 安装
```
gem install sequel
```

## 小例子
```
require 'sequel'

DB = Sequel.sqlite # memory database, requires sqlite3

DB.create_table :items do
  primary_key :id
  String :name
  Float :price
end

items = DB[:items] # Create a dataset

# Populate the table
items.insert(:name => 'abc', :price => rand * 100)
items.insert(:name => 'def', :price => rand * 100)
items.insert(:name => 'ghi', :price => rand * 100)

# Print out the number of records
puts "Item count: #{items.count}"

# Print out the average price
puts "The average price is: #{items.avg(:price)}"
```

## Sequel控制台
Sequel包含一个IRB控制台用于快速的访问数据库，使用方式如下：
```
sequel sqlite://test.db # test.db in current directory
```
这样就可以获得一个IRB会话用于访问数据库。

# 入门
## 连接数据库
你可以使用Sequel.connect(URL)方法来连接数据库：
```
require 'sequel'
DB = Sequel.connect('sqlite://blog.db') # requires sqlite3
```
URL可以包含用户名、密码、端口号：
```
DB = Sequel.connect('postgres://user:password@host:port/database_name') # requires pg
```
你也可以提供可选参数，例如，连接池大小，SQL查询的logger：
```
DB = Sequel.connect("postgres://user:password@host:port/database_name",
  :max_connections => 10, :logger => Logger.new('log/db.log'))
```
你也可以为连接提供一个block，block执行完成后会从数据库连接断开：
```
Sequel.connect('postgres://user:password@host:port/database_name'){|db| db[:posts].delete}
```

## DB约定
在Sequel文档中，我们使用DB常量来表示你创建的Sequel::Database实例。
另外，一些使用Sequel的框架，可能已经为你创建好了Sequel::Database实例，但是你可能不知道怎么获取它，大部分情况下你可以通过Sequel::Model.db来获取。

## 任意的SQL命令
你可以使用Database#run来执行任意的SQL命令：
```
DB.run("create table t (a text, b text)")
DB.run("insert into t values ('a', 'b')")
```
你也可以通过SQL来获取数据集：
```
dataset = DB['select id from items']
dataset.count # will return the number of records in the result set
dataset.map(:id) # will return an array containing all values of the id column in the result set
```
通过SQL语句获取的数据集，你可以操作每一条记录：
```
DB['select * from items'].each do |row|
  p row
end
```
在SQL语句中，还可以使用占位符：
```
name = 'Jim'
DB['select * from items where name = ?', name].each do |row|
  p row
end
```

## 获取数据集实例
数据集是记录被检索和操作的主要方式，他们基本是通过Database#from和Database#[]方法获得的：
```
posts = DB.from(:posts)
posts = DB[:posts] # same
```

## 获取记录
你可以通过all方法获取所有记录：
```
posts.all
# SELECT * FROM posts
```
all方法返回一个包含hash的数组，每个hash对应一条记录。
你也可以通过each迭代器来访问记录：
```
posts.each{|row| p row}
```
或者更拽一点：
```
names_and_dates = posts.map([:name, :date])
old_posts, recent_posts = posts.partition{|r| r[:date] < Date.today - 7}
```
可以获取第一条记录：
```
posts.first
# SELECT * FROM posts LIMIT 1
```
或者获取指定的记录：
```
posts[:id => 1]
# SELECT * FROM posts WHERE id = 1 LIMIT 1
```
如果数据集是有序的，你也可以获取最后一条记录：
```
posts.order(:stamp).last
# SELECT * FROM posts ORDER BY stamp DESC LIMIT 1
```

## 过滤记录
一个过滤记录的简单方式是通过提供给where方法一个hash：
```
my_posts = posts.where(:category => 'ruby', :author => 'david')
# WHERE category = 'ruby' AND author = 'david'
```
也可以指定一个范围：
```
my_posts = posts.where(:stamp => (Date.today - 14)..(Date.today - 7))
# WHERE stamp >= '2010-06-30' AND stamp <= '2010-07-07'
```
或者一个数据：
```
my_posts = posts.where(:category => ['ruby', 'postgres', 'linux'])
# WHERE category IN ('ruby', 'postgres', 'linux')
```
Sequel也可以接受表达式：
```
my_posts = posts.where{stamp > Date.today << 1}
# WHERE stamp > '2010-06-14'
my_posts = posts.where{stamp =~ Date.today}
# WHERE stamp = '2010-07-14'
```
一些数据库还允许指定正则表达式：
```
my_posts = posts.where(:category => /ruby/i)
# WHERE category ~* 'ruby'
```

你也可以使用exclude进行反向过滤：
```
my_posts = posts.exclude(:category => ['ruby', 'postgres', 'linux'])
# WHERE category NOT IN ('ruby', 'postgres', 'linux')
```
你甚至可以提供一个WHERE语句：
```
posts.where('stamp IS NOT NULL')
# WHERE stamp IS NOT NULL
```
字符串中也可以使用参数，比如：
```
author_name = 'JKR'
posts.where('(stamp < ?) AND (author != ?)', Date.today - 3, author_name)
# WHERE (stamp < '2010-07-11') AND (author != 'JKR')
```

数据集也可以用于子查询：
```
DB[:items].where('price > ?', DB[:items].select{avg(price) + 100})
# WHERE price > (SELECT avg(price) + 100 FROM items)
```

## 汇总记录
count方法可以很方便的统计记录数量：
```
posts.where(Sequel.like(:category, '%ruby%')).count
# SELECT COUNT(*) FROM posts WHERE category LIKE '%ruby%'
```
通过max/min方法可以获取最大/最小值:
```
max = DB[:history].max(:value)
# SELECT max(value) FROM history

min = DB[:history].min(:value)
# SELECT min(value) FROM history
```
通过sum/avg方法可以计算和/平均值：
```
sum = DB[:items].sum(:price)
# SELECT sum(price) FROM items
avg = DB[:items].avg(:price)
# SELECT avg(price) FROM items
```

## 记录排序
通过order方法可以对数据集进行排序：
```
posts.order(:stamp)
# ORDER BY stamp
posts.order(:stamp, :name)
# ORDER BY stamp, name
```
链式的order和where一样，不起作用：
```
posts.order(:stamp).order(:name)
# ORDER BY name
```
但是可以使用order_append/order_prepend方法：
```
posts.order(:stamp).order_append(:name)
# ORDER BY stamp, name

posts.order(:stamp).order_prepend(:name)
# ORDER BY name, stamp
```
你也可以指定降序：
```
posts.reverse_order(:stamp)
# ORDER BY stamp DESC
posts.order(Sequel.desc(:stamp))
# ORDER BY stamp DESC
```

## 核心扩展
注意上面的例子中使用的Sequel.desc(:stamp)，大部分的Sequel DSL使用这种方式，调用Sequel module方法返回SQL表达式实例。Sequel提供了一个核心扩展将Sequel DSL和Ruby语言更好的整合在一起，所以你可以这样写：
```
:stamp.desc
```
这个下面是等价的:
```
Sequel.desc(:stamp)
```

## 选择列
select方法可以用于选择列：
```
posts.select(:stamp)
# SELECT stamp FROM posts
posts.select(:stamp, :name)
# SELECT stamp, name FROM posts
```
链式的select效果和order类似，最后一个生效，而不是像where：
```
posts.select(:stamp).select(:name)
# SELECT name FROM posts
```
类似的，select也有一个select_append方法：
```
posts.select(:stamp).select_append(:name)
# SELECT stamp, name FROM posts
```

## 删除记录
使用delete方法即可：
```
posts.where('stamp < ?', Date.today - 3).delete
# DELETE FROM posts WHERE stamp < '2010-07-11'
```
使用delete时要小心，因为delete会影响数据集中的所有行；先调用select，在调用delete：
```
# DO THIS:
posts.where('stamp < ?', Date.today - 7).delete
# NOT THIS:
posts.delete.where('stamp < ?', Date.today - 7)
```

## 插入数据
使用insert方法：
```
posts.insert(:category => 'ruby', :author => 'david')
# INSERT INTO posts (category, author) VALUES ('ruby', 'david')
```

## 更新数据
使用update方法：
```
posts.where('stamp < ?', Date.today - 7).update(:state => 'archived')
# UPDATE posts SET state = 'archived' WHERE stamp < '2010-07-07'
```
你可以引用要设置的列：
```
posts.where{|o| o.stamp < Date.today - 7}.update(:backup_number => Sequel.+(:backup_number, 1))
# UPDATE posts SET backup_number = backup_number + 1 WHERE stamp < '2010-07-07'
```
和delete类似，update会影响数据集中所有的记录，所以，显示用where，再使用update：
```
# DO THIS:
posts.where('stamp < ?', Date.today - 7).update(:state => 'archived')
# NOT THIS:
posts.update(:state => 'archived').where('stamp < ?', Date.today - 7)
```

## 事务
你可以通过Database#transaction方法把代码打包到一个数据库事务中：
```
DB.transaction do
  posts.insert(:category => 'ruby', :author => 'david')
  posts.where('stamp < ?', Date.today - 7).update(:state => 'archived')
end
```
如果代码块没有出现异常，这个事务就会被提交。如果出现了异常，事务就会回滚，异常会被抛出。如果你想回滚事务，但是不想在block外抛出异常，你可以在block里面抛出Sequel::Rollack异常：
```
DB.transaction do
  posts.insert(:category => 'ruby', :author => 'david')
  if posts.filter('stamp < ?', Date.today - 7).update(:state => 'archived') == 0
    raise Sequel::Rollback
  end
end
```

## 联接表
Sequel中连接表很简单：
```
order_items = DB[:items].join(:order_items, :item_id => :id).
  where(:order_id => 1234)
# SELECT * FROM items INNER JOIN order_items
# ON order_items.item_id = items.id 
# WHERE order_id = 1234
```
这里需要注意的是，item_id会自动使用被连接的表限定，id会自动使用上个联接的表进行限定。
**在Sequel中，默认的selection是选择所有联接表的所有列。但是Sequel返回的是一个以列名作为key的hash，所有如果有不同表包含相同的列名，返回的hash结果就会有问题。所以使用连接时，通常会使用select、select_all或者select_append。**

## Sequel中的列引用
Sequel期望指定的列名使用符号，另外，返回的hash结果也使用符号作为key。在很多情况下，这让你可以自由的混用字面值和列引用。下面两行代码产生等价的SQL语句：
```
items.where(:x => 1)
# SELECT * FROM items WHERE (x = 1)
items.where(1 => :x)
# SELECT * FROM items WHERE (1 = x)"
```
Ruby的字符串类型也被当做SQL字符串类型处理：
```
items.where(:x => 'x')
# SELECT * FROM items WHERE (x = 'x')
```

## 限定标识符
在SQL中，标识符用于表示一个列、表或者数据库的名字。标识符可以使用带双下划线的特殊符号限定:table__column:
```
items.literal(:items__price)
# items.price
```
另一种限定列的方式是使用Sequel.qualify方法：
```
items.literal(Sequel.qualify(:items, :price))
# items.price
```
通过表名来限定列是比较常见的，你也可以使用数据库名来限定表：
```
posts = DB[:some_schema__posts]
# SELECT * FROM some_schema.posts
```

## 别名
可以使用三下划线来取别名，:column___alias 或者 :table__column___alias:：
```
items.literal(:price___p)
# price AS p
items.literal(:items__price___p)
# items.price AS p
```
另一种方式是使用Sequal.as方法：
```
items.literal(Sequel.as(:price, :p))
# price AS p
```
你可以使用Sequel.as为任意表达式指定别名，而不只是标识符。

## Sequel模型
一个模型类封装了一个数据集，该类的一个实例封装了数据集的一条记录。
Model类是继承自Sequel::Model的普通Ruby类：
```
DB = Sequel.connect('sqlite://blog.db')
class Post < Sequel::Model
end
```
当一个Model类被创建后，它会处理数据库中的表，并且设置表中所有列的存取方法（Sequel::Model实现了active record模式）。
Sequel模型类假定表名是类名的复数形式：
```
Post.table_name #=> :posts
```
你可以显示的指定表名，或者为数据集指定：
```
class Post < Sequel::Model(:my_posts)
end
# or:
Post.set_dataset :my_posts
```
你可以传递给set_dataset一个符号，它假定你要关联到的是同名的表。你也可以通过数据集调用，这将为该model的所有查询设置默认值。
```
Post.set_dataset DB[:my_posts].where(:category => 'ruby')
Post.set_dataset DB[:my_posts].select(:id, :name).order(:date)
```
这一段表示没看明白：
> 
> If you call set_dataset with a symbol, it assumes you are referring to the table with the same name. You can also call it with a dataset, which will set the defaults for all retrievals for that model:
> ```
> Post.set_dataset DB[:my_posts].where(:category => 'ruby')
> Post.set_dataset DB[:my_posts].select(:id, :name).order(:date)
> ```

### 模型实例
模型实例由主键标识。大多数情况下，Sequel查询数据库来决定主键，如果没有，默认使用id。Model.[]方法可以用来根据主键提取数据。
```
post = Post[123]
```
pk方法用于返回该记录的主键值:
```
post.pk #=> 123
```
Sequel模型允许你使用任何列作为主键，即使是由多个列组成的复合键：
```
class Post < Sequel::Model
  set_primary_key [:category, :title]
end

post = Post['ruby', 'hello world']
post.pk #=> ['ruby', 'hello world']
```
通过no_primary_key，你也可以定义一个没有主键的模型类，同时你也丧失了简单的更新/删除记录的能力了：
```
Post.no_primary_key
```
一个单一模型实例也可以通过指定一个条件获取：
```
post = Post[:title => 'hello world']
post = Post.first{num_comments < 10}
```


###
一个model类将很多的方法转发给了底层的数据集，这意味着你可以使用大多数的Dataset API去创建返回模型实例的自定义查询：
```
Post.where(:category => 'ruby').each{|post| p post}
```
你也可以操作数据集中的记录：
```
Post.where{num_comments < 7}.delete
Post.where(Sequel.like(:title, /ruby/)).update(:category => 'ruby')
```

### 访问记录值
一个模型实例以hash的形式存储了它的值，hash中的key为符号形式的列名，可以通过values方法获取实例的值：
```
post.values #=> {:id => 123, :category => 'ruby', :title => 'hello world'}
```
你可以像访问对象属性一样访问一个记录的值：
```
post.id #=> 123
post.title #=> 'hello world'
```
如果记录的属性名在模式能够的数据集中不是有效的列名（比如你使用了select_append方法添加了一个通过计算得到的列），你可以使用Model#[]来访问这些值：
```
post[:id] #=> 123
post[:title] #=> 'hello world'
```
你还可以通过以下方式修改记录的值：
```
post.title = 'hey there'
post[:title] = 'hey there'
```
这只会改变model实例的值，不会更新数据库中的数据。要更新数据库中的数据，必须使用save方法：
```
post.save
```

### 多列赋值
你可以通过一个方法的调用给多个列赋值，例如，set方法更新模型的列值但不保存到数据库中：
```
post.set(:title=>'hey there', :updated_by=>'foo')
```
update方法完成类似的功能，但是会保存到数据中：
```
post.update(:title => 'hey there', :updated_by=>'foo')
```

### 创建新记录
可以通过调用Model.create方法创建新记录：
```
post = Post.create(:title => 'hello world')
```
另一种方式是先创建实例，然后再保存：
```
post = Post.new
post.title = 'hello world'
post.save
```
你也可以为Model.new和Model.create方法提供一个块：
```
post = Post.new do |p|
  p.title = 'hello world'
end

post = Post.create{|p| p.title = 'hello world'}
```

### 回调
你可以通过钩子方法在创建、更新、删除时执行特定方法。钩子方法包括：
* before_create and after_create 
* before_update and after_update 
* before_save and after_save
* before_destroy and after_destroy
* before_validation and after_validation
```
class Post < Sequel::Model
  def after_create
    super
    author.increase_post_count
  end

  def after_destroy
    super
    author.decrease_post_count
  end
end
```
自定义钩子方法时，记得调用super方法。几乎所有的Sequel::Model方法都可以被安全的重写，但一定要调用super方法，否则就有出错的风险。
对于上面的例子，你也快而已使用数据库触发器。钩子可以用于保证数据的完整性，但是只有在通过model实例修改数据库时才会保证，同时还会面临很多竞态条件。最好是使用数据库触发器和约束来保证数据完整性。

### 删除记录
你可以调用delete或destroy方法删除记录，这两个方法的唯一区别是，destroy方法会调用before_destroy和after_destroy钩子方法，但是delete不会：
```
post.delete # => bypasses hooks
post.destroy # => runs hooks
```
通过delete和destroy也可以一次删除多个符合条件的记录:
```
Post.where(:category => 32).delete # => bypasses hooks
Post.where(:category => 32).destroy # => runs hooks
```
注意，destroy方法会挨个删除记录，但是delete会在一次SQL查询中删除所有符合的记录。

### 关联
模型类之间的关系使用关联表示，用于反映数据库中表的关系，在数据库中通常通过外键指定。通过如下类方法执行关联：
```
class Post < Sequel::Model
  many_to_one :author
  one_to_many :comments
  one_to_one :first_comment, :class=>:Comment, :order=>:id
  many_to_many :tags
  one_through_one :first_tag, :class=>:Tag, :order=>:name, :right_key=>:tag_id
end
```
many_to_one和one_to_one为每个模型对象创建一个getter和setter方法：
```
post = Post.create(:name => 'hi!')
post.author = Author[:name => 'Sharon']
post.author
```
one_to_many和many_to_many方法创建一个getter方法，一个添加对象到该关联关系的方法，一个从该关联关系删除对象的方法，一个从该关联关系中删除所有关联对象的方法：
```
post = Post.create(:name => 'hi!')
post.comments

comment = Comment.create(:text=>'hi')
post.add_comment(comment)
post.remove_comment(comment)
post.remove_all_comments

tag = Tag.create(:tag=>'interesting')
post.add_tag(tag)
post.remove_tag(tag)
post.remove_all_tags
```
注意，remove_*和remove_all_*方法并不从数据库中删除数据，它们只是从接收者解除关联对象。
所有的关联有一个以dataset结尾的方法，可以用于进一步过滤、排序或者是该返回的对象：
```
# Delete all of this post's comments from the database
post.comments_dataset.destroy

# Return all tags related to this post with no subscribers, ordered by the tag's name
post.tags_dataset.where(:subscribers=>0).order(:name).all
```

### 预加载
Associations can be eagerly loaded via eager and the :eager association option. Eager loading is used when loading a group of objects. It loads all associated objects for all of the current objects in one query, instead of using a separate query to get the associated objects for each current object. Eager loading requires that you retrieve all model objects at once via all (instead of individually by each). Eager loading can be cascaded, loading association's associated objects.
```
class Person < Sequel::Model
  one_to_many :posts, :eager=>[:tags]
end

class Post < Sequel::Model
  many_to_one :person
  one_to_many :replies
  many_to_many :tags
end

class Tag < Sequel::Model
  many_to_many :posts
  many_to_many :replies
end

class Reply < Sequel::Model
  many_to_one :person
  many_to_one :post
  many_to_many :tags
end

# Eager loading via .eager
Post.eager(:person).all

# eager is a dataset method, so it works with filters/orders/limits/etc.
Post.where{topic > 'M'}.order(:date).limit(5).eager(:person).all

person = Person.first
# Eager loading via :eager (will eagerly load the tags for this person's posts)
person.posts

# These are equivalent
Post.eager(:person, :tags).all
Post.eager(:person).eager(:tags).all

# Cascading via .eager
Tag.eager(:posts=>:replies).all

# Will also grab all associated posts' tags (because of :eager)
Reply.eager(:person=>:posts).all

# No depth limit (other than memory/stack), and will also grab posts' tags
# Loads all people, their posts, their posts' tags, replies to those posts,
# the person for each reply, the tag for each reply, and all posts and
# replies that have that tag.  Uses a total of 8 queries.
Person.eager(:posts=>{:replies=>[:person, {:tags=>[:posts, :replies]}]}).all
```
In addition to using eager, you can also use eager_graph, which will use a single query to get the object and all associated objects. This may be necessary if you want to filter or order the result set based on columns in associated tables. It works with cascading as well, the API is very similar. Note that using eager_graph to eagerly load multiple *_to_many associations will cause the result set to be a cartesian product, so you should be very careful with your filters when using it in that case.

You can dynamically customize the eagerly loaded dataset by using using a proc. This proc is passed the dataset used for eager loading, and should return a modified copy of that dataset:
```
# Eagerly load only replies containing 'foo'
Post.eager(:replies=>proc{|ds| ds.where(Sequel.like(text, '%foo%'))}).all
```
This also works when using eager_graph, in which case the proc is called with dataset to graph into the current dataset:
```
Post.eager_graph(:replies=>proc{|ds| ds.where(Sequel.like(text, '%foo%'))}).all
```
You can dynamically customize eager loads for both eager and eager_graph while also cascading, by making the value a single entry hash with the proc as a key, and the cascaded associations as the value:
```
# Eagerly load only replies containing 'foo', and the person and tags for those replies
Post.eager(:replies=>{proc{|ds| ds.where(Sequel.like(text, '%foo%'))}=>[:person, :tags]}).all
```
### Joining with Associations
You can use the association_join method to add a join to the model's dataset based on the assocation:
```
Post.association_join(:author)
# SELECT * FROM posts
# INNER JOIN authors AS author ON (author.id = posts.author_id)
```
This comes with variants for different join types:
```
Post.association_left_join(:replies)
# SELECT * FROM posts
# LEFT JOIN replies ON (replies.post_id = posts.id)
```
Similar to the eager loading methods, you can use multiple associations and nested associations:
```
Post.association_join(:author, :replies=>:person).all
# SELECT * FROM posts
# INNER JOIN authors AS author ON (author.id = posts.author_id)
# INNER JOIN replies ON (replies.post_id = posts.id)
# INNER JOIN people AS person ON (person.id = replies.person_id)
```
### Extending the underlying dataset
The recommended way to implement table-wide logic by defining methods on the dataset using dataset_module:
```
class Post < Sequel::Model
  dataset_module do
    def posts_with_few_comments
      where{num_comments < 30}
    end

    def clean_posts_with_few_comments
      posts_with_few_comments.delete
    end
  end
end
```
This allows you to have access to your model API from filtered datasets as well:
```
Post.where(:category => 'ruby').clean_posts_with_few_comments
```
Sequel models also provide a subset class method that creates a dataset method with a simple filter:
```
class Post < Sequel::Model
  subset(:posts_with_few_comments){num_comments < 30}
  subset :invisible, Sequel.~(:visible)
end
```
### Model Validations
You can define a validate method for your model, which save will check before attempting to save the model in the database. If an attribute of the model isn't valid, you should add an error message for that attribute to the model object's errors. If an object has any errors added by the validate method, save will raise an error or return false depending on how it is configured (the raise_on_save_failure flag).
```
class Post < Sequel::Model
  def validate
    super
    errors.add(:name, "can't be empty") if name.empty?
    errors.add(:written_on, "should be in the past") if written_on >= Time.now
  end
end
```