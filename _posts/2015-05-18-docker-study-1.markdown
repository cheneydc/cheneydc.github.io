---
layout: post
title: docker学习（1）
category: 技术 
---

###1. Docker 简介 _《The Docker Book》_<br>Docker 是一个能够把开发的应用程序自动部署到容器的开源引擎。由 Docker 公司的团队编写。Docker在虚拟化的容器执行环境中增加了一个应用程序部署引擎。该引擎的目标就是提供一个轻量、快速的环境，能够运行开发者的程序，并方便高效地将程序从开发者的电脑部署到测试环境，然后再部署到生产环境。Docker上手非常快，用户只需要几分钟，就可以把自己的程序"Docker"化。

开发人员关心容器中的应用程序，而运维只需要考虑如何管理docker。

--------------------------------------------------------------------------------

###2. Docker组件
- Docker Server/Client
- Docker image
- Registry
- Docker Containers

# 2.1. Docker Server/Client
Docker是一个C/S架构的程序。Client只向Server或者守护进程发出请求，Server或者守护进程完成工作并返回结果。

# 2.2. Docker Image
Image 是构建Docker的基础。用户基于Image来运行自己的容器。Image也是Docker生命周期的"构建"部分。Image是基于联合文件系统的一种层式的结构，由一系列的指令一步一步构建出来的。例如：
- 添加一个文件
- 执行一个命令
- 打开一个终端

可以把Image当作容器的"源代码"。

> 这部分不是很理解，感觉像一个安装脚本，最后执行产生的结果是所需要的，后面操作慢慢看...

# 2.3. Docker Registry
Docker 用 Registry 来保存用户构建的Image。Registry分为公有（Docker Hub）和私有两种。

> 应该和git仓库比较像

# 2.4. Docker containers
Docker 可以帮你构建和部署容器，你只需要把自己的应用程序或者服务打包放进容器。刚刚提到，容器是基于Image启动的，容器中可以运行一个或者多个进程。Image相当于Docker生命周期的构建或者打包阶段，Containers相当于启动或者执行阶段。

总结起来，Docker container就是：
- 一个镜像格式
- 一系列标准的操作
- 一个执行环境

Docker借鉴了标准集装箱的概念。标准集装箱将货物运往世界各地，Docker将这个模型运用到设计哲学中，不同的是，集装箱运输货物，Docker运输软件。

和集装箱一样，Docker在执行操作的时候，不关心容器中的是什么，是数据库还是web服务器，还是别的应用程序。所有的容器都按照相同的方式将内容"装载"进去。

--------------------------------------------------------------------------------

## 3. Docker 的技术组件
Docker 可以运行于任何安装了现代Linux内核的x64主机上。Docker开销较低，主要包括一下几个部分：

一个原生的Linux容器格式，Docker中成为**_libcontainer_**，或者很流行的容器平台**_[lxc](http://lxc.sourceforge.net)_**。libcontainer格式现在是Docker容器的默认格式。
- 文件系统隔离： 每个容器都有自己的root文件系统
- 进程隔离： 每个容器都运行在自己的进程环境中
- 网络隔离： 容器间的虚拟网络接口和IP地址是分开的
- 资源隔离和分组： 使用**_[cgroups](http://en.wikipedia.org/wiki/Cgroups)_**将CPU和内存之类的资源独立分配给每个Docker容器
- **_[写时复制](http://en.wikipedia.org/wiki/Copy-on-write)_**： 文件系统都是通过写时复制创建的，这就意味着文件系统是分层的、快速的，而且占用的磁盘空间更小
- 日志： 容器产生的STUOUT、STDERR和STDIN这些IO流都会被收集并入日志，用来进行日志分析和故障排错
- 交互式shell： 用户可以创建一个伪tty终端，将其连接到STDIN，为容器提供一个交互式的shell

--------------------------------------------------------------------------------

## 4. 安装Docker
先决条件：  
- 64位
- 3.8版本以上的kernel，2.6.x也能运行，但是有很大不同
- 内核必须支持一种适合的存储驱动，例如：
  - [Device manager](http://en.wikipedia.org/wiki/Device_manager)
  - [AUFS](http://en.wikipedia.org/wiki/Aufs)
  - [vfs](http://en.wikipedia.org/wiki/Virtual_file_system)
  - [btrfs](http://en.wikipedia.org/wiki/Btrfs)
  - 默认通常是Device Mapper

- kernel支持并开启cgroups和namespace功能

> _环境_ Ubuntu 14.10 amd64

### 4.1. 检查环境
- 检查内核

{% highlight bash %}  $ uname -a {% endhighlight %}  
- 检查是否安装Device Mapper
- {% highlight bash %}  
- $ ls -l /sys/class/misc/device-mapper
- lrwxrwxrwx 1 root root 0  5月 18 20:26 /sys/class/misc/device-mapper -> ../../devices/virtual/misc/device-mapper
- {% endhighlight %}  

也可以

{% highlight bash %}  $ sudo grep device-mapper /proc/devices {% endhighlight %}  

如果没有出现相关信息，可以尝试加载dm_mod模块

{% highlight bash %}  $ sudo modprobe dm_mod {% endhighlight %}  
- cgroup 和 namespace 从2.6以后的版本就集成在kernel了

### 4.2. 安装
{% highlight bash %}  $ sh -c "echo deb [https://get.docker.io/ubuntu](https://get.docker.io/ubuntu) docker main > /etc/apt/sources.list.d/docker.list" $ curl -s [https://get.docker.io/gpg](https://get.docker.io/gpg) | sudo apt-key add - $ sudo apt-get update $ sudo apt-get install lxc-docker $ sudo docker info Containers: 0 Images: 2 Storage Driver: aufs  Root Dir: /var/lib/docker/aufs  Backing Filesystem: extfs  Dirs: 2  Dirperm1 Supported: true Execution Driver: native-0.2 Kernel Version: 3.16.0-37-generic Operating System: Ubuntu Kylin 14.10 CPUs: 4 Total Memory: 5.63 GiB Name: codong ID: WTRJ:5GFG:QWBJ:VKF2:KB4S:UQAE:UGPS:YDIR:L6DI:DRAE:BTRU:AMVG WARNING: No swap limit support {% endhighlight %}  

如果使用ufw，需要更改其策略，默认ufw会丢弃所有转发的数据包。

{% highlight bash %}  $ vim /etc/default/ufw {% endhighlight %}  

将<br>{% highlight bash %}  DEFAULT_FORWARD_POLICY='DROP' {% endhighlight %}  

替换成 {% highlight bash %}  DEFAULT_FORWARD_POLICY='ACCEPT' {% endhighlight %}  

重新加载UFW<br>{% highlight bash %}  $ sudo ufw reload {% endhighlight %}  
