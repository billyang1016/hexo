---
title: puppet的使用：ERB模板
date: 2016-06-13 23:51:16
tags:
- puppet
categories:
- 技术
---
puppet中使用ERB模板。
<!-- more -->
## ERB介绍
全称是Embedded RuBy，意思是嵌入式的Ruby，是一种文本模板技术，用过JSP的话，会发现两者语法很像。
我们项目中一般用ERB来产生各模块的配置文件。ERB模板也可以用来产生Web页面（之前搞过一段时间ROR开发，模板用的haml），也可以用来产生其他文件。
### <%  %>与<%=  %> 
<%Ruby脚本%>，一般是Ruby的逻辑脚本，但是不会写入到目标文件中。
<%= Ruby脚本%> ，脚本的执行结果会写入到目标文件中。 
举例如下（[代码来源](http://lj6684.iteye.com/blog/410424)）：
这段代码是从Hash中读取信息创建sql语句保存到文件中。
```
require "erb"  
domains = {...}  
sqlTemplate = ERB.new %q{  
<%for organization in domains.keys%>  
    insert into org_domain(Domain, organization) values('<%=domains[organization]%>','<%=organization%>');  
<%end%>  
}  
sqlFile = File.new("./sql.sql", "w")  
sqlFile.puts sqlTemplate.result 
```
### 使输出紧凑
#### <%= -%>
You can trim line breaks after expression-printing tags by adding a hyphen to the closing tag delimiter.

* -%> — If the tag ends a line, trim the following line break.

意思是如果<%= -%>表达式中，-%>是一行的结尾，会忽略后面的换行。

#### <%- -%>
You can trim whitespace surrounding a non-printing tag by adding hyphens (-) to the tag delimiters.

* <%- — If the tag is indented, trim the indentation.
* -%> — If the tag ends a line, trim the following line break.

意思是<%-前面是空白符时，会删除这些空白符；-%> 含义同上。

## puppet中一般怎么使用erb
举例如下：
### 目录结构
```
~/tmp# tree -L 2 puppet
puppet
├── manifests
│     ├── nodes
│     └── site.pp
└── modules
    ├── xxx
    └── nginx
```

这里是大的目录结构。

- - -
manifests的内容如下：
```
~/tmp/puppet# tree -L 2 manifests
manifests
├── nodes
│   ├── controller-192-41-1-185.pp
│   ├── controller-192-41-1-186.pp
│   ├── controller-192-41-1-187.pp
│   ├── controller-192-41-1-191.pp
│   └── controller-192-41-1-202.pp
└── site.pp
```
这里每个.pp文件是对应每个agent节点的定义，具体内容见下一节。

------
每个模块下的内容类似，现在只以nginx举例。
```
~/tmp/puppet/modules# tree -L 3 nginx/
nginx/
├── files
│   ├── xxxClientCert.jks
│   ├──xxxServerCert.jks
│   ├── README.md
│   ├── server.crt
│   └── server.key
├── manifests
│   ├── config.pp
│   ├── init.pp
│   ├── params.pp
│   └── service.pp
└── templates
    └── conf.d
        ├── nginx.conf.erb
        ├── ssl_restart_oms.sh.erb
        ├── web-hedex.xml.erb
        └── web.xml.erb
```
具体模块的manifests目录下是各个类的定义，templates定义了各个模板，模板会在manifests下的类中被引用。

### 类的定义
-------
```
~/tmp/puppet/manifests# cat site.pp 
import 'nodes/*.pp'
$puppetserver='controller-192-41-1-191'
```

--------
下面是一个node的定义：
```
~/tmp/puppet/manifests/nodes# cat controller-192-41-1-191.pp 
node 'controller-192-41-1-191' {
  $xxx = "true"
 #这里是各种变量的定义
  Class['aaa']->Class['bbb']->Class['ccc']->Class['nginx']->Class['ddd']
  include aaa,bbb,ccc,nginx,ddd
}
```

模板中使用的变量就是来自这里的定义。

------
**总结如下**：
nginx/manifests下的各个pp文件是各个类的定义，类的定义中使用了nginx/templates中定义的模板，这些类被controller-192-41-1-191.pp 文件引用，controller-192-41-1-191.pp 文件中定义了模板中使用的变量。

## 访问puppet 变量
模板可以访问puppet的变量，这是模板的主要数据来源。
一个ERB模板有自己的本地作用域，它的父作用域是引用该模板的类。这意味着，模板可以使用短名称访问父类中的变量，但是不能向父类中添加新的变量。
在ERB模板中，有两种方式访问变量。
* @variable
* scope['variable'] (旧的形式为：scope.lookupvar('variable'))

### @variable
原文：
> All variables in the current scope (including global variables) are passed to templates as Ruby instance variables, which begin with “at” signs (@). If you can access a variable by its short name in the surrounding manifest, you can access it in the template by replacing its $ sign with an @. So $os becomes @os, $trusted becomes @trusted, etc. 

当前作用域的所有变量（包括全局变量）都会以Ruby实例变量的形式传给模板。如果你在模板中可以使用短名称访问一个变量，你就可以使用"@"代替"$"。所以，$os就变成了@os，$trusted就变成了@trusted。
这是访问变量最直接清晰的方式，但是这种方式不能访问其他作用域的变量，需要访问其他作用域的变量时，你需要使用下面的方式。
### scope['variable'] or scope.lookupvar('variable')
Puppet可以使用对象scope来以散列表的方式访问变量。例如，访问$ntp::tinker你可以使用scope['ntp::tinker']这种形式。
还有另一种方式使用scope对象，就是通过它的lookupvar方法来访问变量，该方法需要传入要访问的变量名，例如，scope.lookupvar('ntp::tinker')。这和上面的方式效果是一样的，但是使用起来没有scope['ntp::tinker']更简洁直观。

### puppet类型和Ruby类型的对应关系
这部分表格，就不翻译了，直接贴图。
![puppet-data-type-in-ruby](/images/puppet-data-type-in-ruby.jpg)

### 未定义变量
如果一个puppet变量没有定义的话，它的值是undef，意思是，在模板中它的值是nil。
含义是Puppet变量值为undef，对应Ruby变量值为nil。

### 在模板中使用puppet函数
* 所有的函数都以scope对象的方法调用；
* 函数名的开头都必须加上前缀"function_"；
* 函数的参数必须以数组的形式传递，即使只有一个参数。
例如，在一个模板中引用另一个模板：

> `<%= scope.function_template(["my_module/template2.erb"]) %>`

### 模板示例

```
# ntp.conf: Managed by puppet.
#
<% if @tinker == true and (@panic or @stepout) -%>
# Enable next tinker options:
# panic - keep ntpd from panicking in the event of a large clock skew
# when a VM guest is suspended and resumed;
# stepout - allow ntpd change offset faster
tinker<% if @panic -%> panic <%= @panic %><% end %><% if @stepout -%> stepout <%= @stepout %><% end %>
<% end -%>

<% if @disable_monitor == true -%>
disable monitor
<% end -%>
<% if @disable_auth == true -%>
disable auth
<% end -%>

<% if @restrict != [] -%>
# Permit time synchronization with our time source, but do not
# permit the source to query or modify the service on this system.
<% @restrict.flatten.each do |restrict| -%>
restrict <%= restrict %>
<% end -%>
<% end -%>

<% if @interfaces != [] -%>
# Ignore wildcard interface and only listen on the following specified
# interfaces
interface ignore wildcard
<% @interfaces.flatten.each do |interface| -%>
interface listen <%= interface %>
<% end -%>
<% end -%>

<% if @broadcastclient == true -%>
broadcastclient
<% end -%>

# Set up servers for ntpd with next options:
# server - IP address or DNS name of upstream NTP server
# iburst - allow send sync packages faster if upstream unavailable
# prefer - select preferrable server
# minpoll - set minimal update frequency
# maxpoll - set maximal update frequency
<% [@servers].flatten.each do |server| -%>
server <%= server %><% if @iburst_enable == true -%> iburst<% end %><% if @preferred_servers.include?(server) -%> prefer<% end %><% if @minpoll -%> minpoll <%= @minpoll %><% end %><% if @maxpoll -%> maxpoll <%= @maxpoll %><% end %>
<% end -%>

<% if @udlc -%>
# Undisciplined Local Clock. This is a fake driver intended for backup
# and when no outside source of synchronized time is available.
server   127.127.1.0
fudge    127.127.1.0 stratum <%= @udlc_stratum %>
restrict 127.127.1.0
<% end -%>

# Driftfile.
driftfile <%= @driftfile %>

<% unless @logfile.nil? -%>
# Logfile
logfile <%= @logfile %>
<% end -%>

<% unless @peers.empty? -%>
# Peers
<% [@peers].flatten.each do |peer| -%>
peer <%= peer %>
<% end -%>
<% end -%>

<% if @keys_enable -%>
keys <%= @keys_file %>
<% unless @keys_trusted.empty? -%>
trustedkey <%= @keys_trusted.join(' ') %>
<% end -%>
<% if @keys_requestkey != '' -%>
requestkey <%= @keys_requestkey %>
<% end -%>
<% if @keys_controlkey != '' -%>
controlkey <%= @keys_controlkey %>
<% end -%>

<% end -%>
<% [@fudge].flatten.each do |entry| -%>
fudge <%= entry %>
<% end -%>

<% unless @leapfile.nil? -%>
# Leapfile
leapfile <%= @leapfile %>
<% end -%>
> ```

**参考**
[Language: Embedded Ruby (ERB) template syntax](https://docs.puppet.com/puppet/latest/reference/lang_template_erb.html)