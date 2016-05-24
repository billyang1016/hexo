---
title: puppet的使用：资源介绍
date: 2016-05-08 21:34:28
tags:
- puppet
categories:
- 技术
---
puppet资源的相关概念。

<!-- more -->
# 资源的概念

# 资源的引用
格式：Type ["title"]
其中资源类型的首字母必须大写，title可以有多个，即支持Type ["title1",...,"titlen"]的格式。
# 各类资源
## file
### 文件服务器
file资源中可以通过source属性指定从master文件服务器上获取文件的内容。
语法格式为：
source => "puppet://masterServerName/modules/modulename/realpath"或者
source => "puppet:///modules/modulename/realpath"。
对于第二种形式，没有直接指定server的名字，则认为是当前agent正在连接的master。
模块的文件都保存在files路径下，上述“realpath”的值是文件相对于files的路径。比如路径为“files/aaa/bbb”，那么“realpath”的值即为“aaa/bbb”。
