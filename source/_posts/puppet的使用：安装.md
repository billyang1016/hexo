---
title: puppet的使用：安装puppet
date: 2016-05-08 16:29:20
tags:
- puppet
categories:
- 技术
---
puppet master及agent的安装。

<!-- more -->

最近项目要使用puppet，趁机赶紧学习下。
在家里的机器中搭建puppet环境，使用两台ubuntu 14.04；
## 准备工作
### 时间同步
两台设备先进行时间同步，我把要安装master的机器作为NTP服务器，client向master同步下时间；
这个不会的可以搜下NTP的配置；

### 配置/etc/hosts
把hostname改好，并添加到/etc/hosts中，master和client都添加到hosts中，这样才能根据hostname进行访问；

## 安装 

 1. apt-get update
 这个是同步源索引，后面才能下载到最新的包。
 2. apt-get install puppetmaster
 master的安装
 
 3. apt-get install puppet
 client的安装

安装完了通过puppet --version查看版本

## 配置文件
默认即可，有时间再补充；

## master的启动
puppet master --verbose --no-daemonize
这样启动后就能看到日志的输出；

## client 的启动
puppet agent --test --server=xxx
启动后会看到如下信息
Info: Creating a new SSL key for yourhostname
Info: Caching certificate for ca
Info: csr_attributes file loading from /etc/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for yourhostname
Info: Certificate Request fingerprint (SHA256): xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Info: Caching certificate for ca
Exiting; no certificate found and waitforcert is disabled

## 签发
在上一步，client向master发起了证书签批请求，在master上通过命令puppet cert list -a就能看到待签发的信息；
"yourhostname"   (SHA256) xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
如果yourhostname前面有个“+”就表示已签发；
对于未签发的可以通过命令puppet cert sign hostname或者puppet cert sign --all（签发所有）进行签发；

## 证书与ssl删除
### master节点删除证书
puppet cert --clean hostname
### 删除ssl
根据puppet.conf目录中的sslpath字段，删除ssl，然后启动agent，会重新生成ssl，master节点也一样；
### agent重新向master注册
* agent操作：
删除ssl证书，rm -rf /etc/puppet/ssl

* master操作：
清除对应agent的认证，puppet cert clean agentHostname
