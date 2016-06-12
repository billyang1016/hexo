---
title: require/load/include/extend的区别
date: 2016-06-13 00:02:17
tags:
- Ruby
categories:
- 技术
---
Ruby中require/load/include/extend的区别。
<!-- more -->
# require
一般用于加载一个库，当多次使用require加载一个库时，只有第一次有效，后面的都会加载失败，也就是会返回"false"，以为require会追踪文件是否被加载。
使用require加载库文件时，可以不带后缀".rb"。一般放在文件的最前面。
test2.rb文件内容如下：
```
result=require "./test1"
puts result
result=require "./test1"
puts result
```
结果为：
```
true
false
```
## 路径的问题
上面的例子要求test1.rb和test2.rb在同一路径下，因为使用了"./"。
指定路径方法：
* require File.dirname(__FILE__)+'/test1'
File.dirname(__FILE__)返回当前文件的所在目录，在这个例子中__FILE__的值就是"test2.rb"，File.dirname(__FILE__)的返回值就是"."。
* require File.join(File.dirname(__FILE__),'test2')
和上面类似。
* require File.expand_path('../test2',__FILE__)
* $LOAD_PATH.unshift(File.dirname(__FILE__))
require 'test2.rb'
把test1所在的目录加入load path
* 直接require './test2.rb'

# load
和require类似，但是一般用于加载配置文件。和require有两大区别：
* 对一个文件可以多次load，每次都会加载。适合加载可能出现变化的文件。
* 另一个区别是不能省略".rb"。

# include
在类定义时引入module的实例方法作为当前类的实例方法，引入module的变量作为当前类的类变量。
include并不会把module的实例方法拷贝到类中，只是做了引用，包含module的不同类都指向了同一个对象。如果你改变了module的定义，即使你的程序还在运行，所有包含module的类都会改变行为。
*当被引入的类和当前类不在同一个文件时，需要先使用require或load加载*

# extend
在定义类时使用，把module的实例方法作为当前类的类方法。