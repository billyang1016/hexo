---
title: puppet的使用：依赖关系整理
date: 2016-05-08 21:45:33
tags:
- puppet
categories:
- 技术
---
puppet中的依赖关系整理。

<!-- more -->

# 概述
puppet中的依赖关系大概有如下几个：
* require
* before
* after
* notify
* subscribe

**更准确的说法是前三者表示的是依赖，后两者表示的是通知。**

# 详细介绍
## require
type1 { 'title1'
    ...
}

type2 { 'title2'
    ...,
    require => Type1['title1'],
}
表示资源title2依赖title1，即title1必须在title2之前就存在或正确执行了。


## before
表示在某个资源之前执行。
例如：
before => Type1['title1']，表示before所在资源在'title1'之前执行。

## after
和before含义相反。自行脑补。

## 小结
上面描述的三个是资源之间的依赖关系，实际就是某个资源执行前另一个要先执行了，或者要在其后执行，但并不表示每次执行该资源的动作时都会执行依赖的动作。
before、after、和require，均可用于各个资源中。

## notify
通知某个资源进行更新。
notify => Type1['title1']，表示notify所在资源执行后通知'title1'，经常用于配置文件更新后通知服务重启。

## subscribe
资源有更新时，通知另一个资源执行相应的动作。
subscribe => Type1['title1']，表示subscribe所在资源关心资源'title1'，当'title1'发生变化了会通知subscribe所在资源。
目前支持subscribe只有exec、service、mount。

notify和subscribe是对应的，在一个资源里使用了notify，就相当于在另一个资源中使用了subscribe。