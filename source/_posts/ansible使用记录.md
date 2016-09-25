---
title: ansible使用记录
date: 2016-09-25 12:47:26
tags:
- 技术
categories:
- ansible
---
ansible使用记录。
<!-- more -->
# ansible 使用记录
## ssh密钥管理
* master上生成密钥
ssh-keygen -t rsa -P ''

* 公钥放到其他节点上
~/.ssh# ssh-copy-id -i /root/.ssh/ansible_master_rsa.pub root@host_ip
会在host_ip主机的~/.ssh/生成authorized_keys，内容为上面的公钥内容。
也可以把公钥放到对应的节点上之后，通过cat xxx.put >> authorized_keys追加进去。

## ansible 配置
### 配置文件修改
vim /etc/ansible/ansible.cfg
……
remote_port = 22 #被管理节点ssh服务端口
private_key_file = /root/.ssh/id_rsa_storm1 #master ssh私钥路径
……
### hosts文件修改
/etc/ansible/hosts记录了被管理的主机，格式：
[分组]
ip1
ip2
