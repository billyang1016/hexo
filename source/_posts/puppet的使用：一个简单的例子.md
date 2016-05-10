---
title: puppet的使用：puppet的hello world
date: 2016-05-08 17:14:05
tags:
- puppet
categories:
- puppet
---
一个简单的示例：将master节点上的一个文件放至agent节点上。
<!-- more -->
**这个例子完成将master节点上的一个文件放至agent节点上的功能**

## 创建要传输的文件
    echo “helloWorld” > /etc/puppet/modules/puppet-example/files/hello

## 不使用module
### 创建site.pp
直接把对agent的操作写入site.pp文件中。

    touch /etc/puppet/manifests/site.pp
内容如下：

     node 'agentHostname' {
        file { '/root/tmp/hello':
            owner => 'root',
            group => 'root',
            mode => '0440',
            source => 'puppet:///modules/puppet-example/hello'
            }
        }

### 启动agent
在agent上执行如下命令：
puppet agent --test
就会看到/root/tmp/hello文件了。

## 使用module
### 创建module
module的目录结构是固定的，目录的结构一般如下所示：
├── files
├── manifests
└── templates
* files: 属于模块的文件
* manifests: 脚本文件
* templates：模板文件

    md -p /etc/puppet/modules/puppet-example/{files,templates,manifests}


### 生成init.pp文件
init.pp是模块必须要有的文件，放至在模块的manifests路径下。

    touch /etc/puppet/modules/puppet-example/manifests/init.pp

    class puppet-example {
        file { '/root/tmp/hello':
            owner => 'root',
            group => 'root',
            mode => '0440',
            source => 'puppet:///modules/puppet-example/hello'
        }
    }

### 生成site.pp文件
    touch /etc/puppet/manifests/site.pp
内容如下：
    node 'agentHostname' {
      include puppet-example
    }
至此，文件全部创建完毕。

### 目录结构
最终的目录结构如下所示：
    /etc/puppet# tree

    ├── manifests
    │   └── site.pp
    ├── modules
    │   └── puppet-example
    │       ├── files
    │       │   └── hello
    │       ├── manifests
    │       │   └── init.pp
    │       └── templates
    └── templates

### 启动agent
在agent上执行如下命令：
puppet agent --test
就会看到/root/tmp/hello文件了。

