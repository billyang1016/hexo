---
title: puppet的使用：puppet配置文件介绍
date: 2016-05-08 16:59:08
tags:
- puppet
categories:
- puppet
---
puppet master及agent配置文件简介。

<!-- more -->
## 配置文件的产生
Puppet安装完后，配置文件就产生了，名称为puppet.conf，一般在/etc/puppet路径下。
master也可以通过命令：
puppet master --genconfig > puppet.conf
产生。
agent可以通过命令：
puppet agent --genconfig > puppet.conf
产生。

puppet配置文件一般包括main、master、agent这几个小节。
### main
全局配置。
[main]
    logdir=/var/log/puppet
    vardir=/var/lib/puppet
    ssldir=/var/lib/puppet/ssl
    rundir=/var/run/puppet
    factpath=$vardir/lib/facter
    templatedir=$confdir/templates
    server=puppet.example.com
一般只需要修改server即可，server一般是master的hostname，同时也要在agent的/etc/hosts中配置。

**master和agent的配置项太多，下面只是简单的罗列出来了，只把重要的几个配置项含义列了下，其他的可以参考对应的英文说明，通过前面命令生成的配置文件都会有对应配置项含义的说明**

*agent和master默认的监听端口都是8140，如果在一台机器上master和agent都要起，需要修改其中一个的端口*

### master
[master]
    confdir = /etc/puppet #配置文件路径
    vardir = /var/lib/puppet #puppet数据的存放位置
    name = master
    logdir = /var/lib/puppet/log
    statedir = /var/lib/puppet/state
    rundir = /var/lib/puppet/run
    libdir = /var/lib/puppet/lib
    route_file = /etc/puppet/routes.yaml
    node_terminus = plain
    node_cache_terminus = write_only_yaml
    data_binding_terminus = hiera
    hiera_config = /etc/puppet/hiera.yaml
    catalog_terminus = compiler
    facts_terminus = yaml
    inventory_terminus = yaml
    default_file_terminus = rest
    httplog = /var/lib/puppet/log/http.log
    http_keepalive_timeout = 4
    filetimeout = 15
    environment_timeout = 0
    immutable_node_data = false
    preview_outputdir = /var/lib/puppet/preview
    csr_attributes = /etc/puppet/csr_attributes.yaml
    certdir = /etc/puppet/ssl/certs
    ssldir = /etc/puppet/ssl #ssl文件的存放位置，一般无需改动
    publickeydir = /etc/puppet/ssl/public_keys
    requestdir = /etc/puppet/ssl/certificate_requests
    privatekeydir = /etc/puppet/ssl/private_keys
    privatedir = /etc/puppet/ssl/private
    passfile = /etc/puppet/ssl/private/password
    hostcsr = /etc/puppet/ssl/csr_cuimiemie.pem
    hostcert = /etc/puppet/ssl/certs/cuimiemie.pem
    hostprivkey = /etc/puppet/ssl/private_keys/cuimiemie.pem
    hostpubkey = /etc/puppet/ssl/public_keys/cuimiemie.pem
    localcacert = /etc/puppet/ssl/certs/ca.pem
    hostcrl = /etc/puppet/ssl/crl.pem
    certificate_expire_warning = 5184000
    plugindest = /var/lib/puppet/lib
    pluginsource = puppet://puppet/plugins
    pluginfactdest = /var/lib/puppet/facts.d
    pluginfactsource = puppet://puppet/pluginfacts
    factpath = /var/lib/puppet/lib/facter:/var/lib/puppet/facts
    module_working_dir = /var/lib/puppet/puppet-module
    module_skeleton_dir = /var/lib/puppet/puppet-module/skeleton
    ca_name = Puppet CA: cuimiemie
    cadir = /etc/puppet/ssl/ca
    cacert = /etc/puppet/ssl/ca/ca_crt.pem
    cakey = /etc/puppet/ssl/ca/ca_key.pem
    capub = /etc/puppet/ssl/ca/ca_pub.pem
    cacrl = /etc/puppet/ssl/ca/ca_crl.pem
    capub = /etc/puppet/ssl/ca/ca_pub.pem
    cacrl = /etc/puppet/ssl/ca/ca_crl.pem
    caprivatedir = /etc/puppet/ssl/ca/private
    csrdir = /etc/puppet/ssl/ca/requests
    signeddir = /etc/puppet/ssl/ca/signed #这里会记录以前发的客户端，一般名称为agentHostname.pem
    capass = /etc/puppet/ssl/ca/private/ca.pass
    serial = /etc/puppet/ssl/ca/serial
    autosign = /etc/puppet/autosign.conf #用于控制是否自动签发，默认是false
    ca_ttl = 157680000
    cert_inventory = /etc/puppet/ssl/ca/inventory.txt
    config = /etc/puppet/puppet.conf
    pidfile = /var/lib/puppet/run/master.pid
    manifestdir = /etc/puppet/manifests
    manifest = /etc/puppet/manifests/site.pp
    masterlog = /var/lib/puppet/log/puppetmaster.log
    masterhttplog = /var/lib/puppet/log/masterhttp.log
    bucketdir = /var/lib/puppet/bucket
    rest_authconfig = /etc/puppet/auth.conf
    basemodulepath = /etc/puppet/modules:/usr/share/puppet/modules
    modulepath = /etc/puppet/modules:/usr/share/puppet/modules #模块文件的存放路径
    yamldir = /var/lib/puppet/yaml
    server_datadir = /var/lib/puppet/server_data
    reportdir = /var/lib/puppet/reports
    fileserverconfig = /etc/puppet/fileserver.conf
    storeconfigs_backend = active_record
    rrddir = /var/lib/puppet/rrd
    rrdinterval = 1800
    devicedir = /var/lib/puppet/devices
    deviceconfig = /etc/puppet/device.conf
    node_name_value = cuimiemie
    localconfig = /var/lib/puppet/state/localconfig
    statefile = /var/lib/puppet/state/state.yaml
    clientyamldir = /var/lib/puppet/client_yaml
    client_datadir = /var/lib/puppet/client_data
    classfile = /var/lib/puppet/state/classes.txt
    resourcefile = /var/lib/puppet/state/resources.txt
    puppetdlog = /var/lib/puppet/log/puppetd.log
    runinterval = 1800
    ca_server = puppet
    ca_port = 8140
    agent_catalog_run_lockfile = /var/lib/puppet/state/agent_catalog_run.lock
    agent_disabled_lockfile = /var/lib/puppet/state/agent_disabled.lock
    splaylimit = 1800
    clientbucketdir = /var/lib/puppet/clientbucket
    configtimeout = 120 
    report_server = puppet
    report_port = 8140
    inventory_server = puppet
    inventory_port = 8140
    lastrunfile = /var/lib/puppet/state/last_run_summary.yaml
    lastrunreport = /var/lib/puppet/state/last_run_report.yaml


### agent
[agent]
    confdir = /etc/puppet
    vardir = /var/lib/puppet
    name = agent
    logdir = /var/lib/puppet/log
    statedir = /var/lib/puppet/state
    rundir = /var/lib/puppet/run
    libdir = /var/lib/puppet/lib
    route_file = /etc/puppet/routes.yaml
    node_terminus = rest
    data_binding_terminus = hiera
    hiera_config = /etc/puppet/hiera.yaml
    catalog_terminus = rest
    catalog_cache_terminus = json
    facts_terminus = facter
    inventory_terminus = facter
    default_file_terminus = rest
    httplog = /var/lib/puppet/log/http.log
    http_keepalive_timeout = 4 
    filetimeout = 15
    environment_timeout = 0 
    immutable_node_data = false
    preview_outputdir = /var/lib/puppet/preview
    csr_attributes = /etc/puppet/csr_attributes.yaml
    certdir = /etc/puppet/ssl/certs
    ssldir = /etc/puppet/ssl
    publickeydir = /etc/puppet/ssl/public_keys
    requestdir = /etc/puppet/ssl/certificate_requests
    privatekeydir = /etc/puppet/ssl/private_keys
    privatedir = /etc/puppet/ssl/private
    passfile = /etc/puppet/ssl/private/password
    hostcsr = /etc/puppet/ssl/csr_cuimiemie.pem
    hostcert = /etc/puppet/ssl/certs/cuimiemie.pem
    hostprivkey = /etc/puppet/ssl/private_keys/cuimiemie.pem
    hostpubkey = /etc/puppet/ssl/public_keys/cuimiemie.pem
    localcacert = /etc/puppet/ssl/certs/ca.pem
    hostcrl = /etc/puppet/ssl/crl.pem
    certificate_expire_warning = 5184000
    plugindest = /var/lib/puppet/lib
    pluginsource = puppet://puppet/plugins
    pluginfactdest = /var/lib/puppet/facts.d
    pluginfactsource = puppet://puppet/pluginfacts
    factpath = /var/lib/puppet/lib/facter:/var/lib/puppet/facts
    module_working_dir = /var/lib/puppet/puppet-module
    module_skeleton_dir = /var/lib/puppet/puppet-module/skeleton
    ca_name = Puppet CA: cuimiemie
    cadir = /etc/puppet/ssl/ca
    cacert = /etc/puppet/ssl/ca/ca_crt.pem
    cakey = /etc/puppet/ssl/ca/ca_key.pem
    capub = /etc/puppet/ssl/ca/ca_pub.pem
    cacrl = /etc/puppet/ssl/ca/ca_crl.pem
    caprivatedir = /etc/puppet/ssl/ca/private
    csrdir = /etc/puppet/ssl/ca/requests
    signeddir = /etc/puppet/ssl/ca/signed
    capass = /etc/puppet/ssl/ca/private/ca.pass
    serial = /etc/puppet/ssl/ca/serial
    autosign = /etc/puppet/autosign.conf
    ca_ttl = 157680000
    cert_inventory = /etc/puppet/ssl/ca/inventory.txt
    config = /etc/puppet/puppet.conf
    pidfile = /var/lib/puppet/run/agent.pid
    manifestdir = /etc/puppet/manifests
    manifest = /etc/puppet/manifests/site.pp
    masterlog = /var/lib/puppet/log/puppetmaster.log
    masterhttplog = /var/lib/puppet/log/masterhttp.log
    bucketdir = /var/lib/puppet/bucket
    rest_authconfig = /etc/puppet/auth.conf
    basemodulepath = /etc/puppet/modules:/usr/share/puppet/modules
    modulepath = /etc/puppet/modules:/usr/share/puppet/modules
    yamldir = /var/lib/puppet/yaml
    server_datadir = /var/lib/puppet/server_data
    reportdir = /var/lib/puppet/reports
    fileserverconfig = /etc/puppet/fileserver.conf
    storeconfigs_backend = active_record
    rrddir = /var/lib/puppet/rrd
    rrdinterval = 1800
    devicedir = /var/lib/puppet/devices
    deviceconfig = /etc/puppet/device.conf
    node_name_value = cuimiemie
    localconfig = /var/lib/puppet/state/localconfig
    statefile = /var/lib/puppet/state/state.yaml
    clientyamldir = /var/lib/puppet/client_yaml
    client_datadir = /var/lib/puppet/client_data
    classfile = /var/lib/puppet/state/classes.txt
    resourcefile = /var/lib/puppet/state/resources.txt
    puppetdlog = /var/lib/puppet/log/puppetd.log
    runinterval = 1800 #这个时间是客户端主动向master请求数据的时间间隔，单位默认是s
    ca_server = puppet
    ca_port = 8140
    agent_catalog_run_lockfile = /var/lib/puppet/state/agent_catalog_run.lock
    agent_disabled_lockfile = /var/lib/puppet/state/agent_disabled.lock
    splaylimit = 1800
    clientbucketdir = /var/lib/puppet/clientbucket
    configtimeout = 120 
    report_server = puppet
    report_port = 8140 #客户端监听的端口号，一般也无需改动
    inventory_server = puppet
    inventory_port = 8140
    lastrunfile = /var/lib/puppet/state/last_run_summary.yaml
    lastrunreport = /var/lib/puppet/state/last_run_report.yaml
    graphdir = /var/lib/puppet/state/graphs
    waitforcert = 120 
    archive_file_server = puppet
    tagmap = /etc/puppet/tagmail.conf
    dblocation = /var/lib/puppet/state/clientconfigs.sqlite3
    railslog = /var/lib/puppet/log/rails.log
    templatedir = /var/lib/puppet/templates



