---
title: puppet的使用：目录结构
date: 2016-05-11 00:09:44
tags:
- puppet
categories:
- puppet
---
介绍puppet的目录结构
<!-- more -->

# 目录结构
    root@xxx:/etc/puppet# tree -L 2
    .
    ├── environments
    │   └── example_env
    ├── manifests
    │   └── site.pp
    ├── modules
    │   └── puppet-example
    ├── ssl
    │   ├── ca
    │   ├── certificate_requests
    │   ├── certs
    │   ├── crl.pem
    │   ├── private
    │   ├── private_keys
    │   └── public_keys
    └── templates

一般puppet的目录结构如上图所示。
## site.pp
site.pp文件默认放在/etc/puppet/manifests路径下，该文件对agent进行了定义，master就是根据site.pp的内容获取对agent的操作。
一般manifests目录结构如下：

    .
    ├── nodes
    │   ├── agent1.pp
    │   └── agent2.pp
    └── site.pp
agentx.pp里是对agent操作的定义，里面引用的class一般定义在各个modules下面。