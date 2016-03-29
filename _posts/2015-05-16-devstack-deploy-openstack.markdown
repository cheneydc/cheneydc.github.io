---
layout: post
title: devstack部署openstack
category: 技术 
---


前一阵工作需要，熟悉了下openstack，环境有限，决定采用all in one方式利用devstack进行部署.


*环境:*

`ubuntu-14.04-server-amd64`


#1. Get devstack

{% highlight bash %}
$ git clone https://git.openstack.org/openstack-dev/devstack
{% endhighlight %}  

#2. Create configure file: localrc

根据[官方文档](http://docs.openstack.org/developer/devstack/configuration.html#minimal-configuration)提示，创建配置文件'localrc'进行安装配置：

{% highlight bash %}
$ cat localrc
ENABLED_SERVICES=g-api,g-reg,key,n-api,n-crt,n-cpu,n-net,n-cond,n-sch,rabbit,mysql,horizon,sadfasfdasfda 
ADMIN_PASSWORD=redhat
DEST=/opt/stack
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
HOST_IP=XXX.XXX.XXX.XXX
SERVICE_TOKEN=123qweP
{% endhighlight %}  

#3. 安装...  

{% highlight bash %}
$ ./stach.sh
{% endhighlight %}

等待跑完看到Successfully就可以了，浏览器输入HOST_IP就可以登录了  

#4. 遇到问题  

碰到了一些问题，devstack很多包会自动安装，对版本要求也比较高，所以有时候会有包的冲突问题：

*  pycadf<0.9.0,>=0.8.0

查看当前包的版本

{% highlight bash %}
$ sudo pip show pycadf
 {% endhighlight %}

删除旧包，安装新包

{% highlight bash %} 
$ sudo pip uninstall pycadf
$ sudo pip install 'pycadf<0.9.0'
{% endhighlight %}

*  更换git  

devstack会从github抓相关代码，由于国内网络限制（你懂得），加上openstack项目较大的原因，经常会timeout导致安装失败，国内oschina有openstack的git，所以这里可以更换一下。

{% highlight bash %}
$ cat stackrc
...
#GIT_BASE=${GIT_BASE:-git://git.openstack.org}
GIT_BASE=${GIT_BASE:-https://git.oschina.net}
...
{% endhighlight %}  

不过oschina的git并不完整，keystone的git竟然木有kilo的branch，所以这种情况还需要自己调整。
