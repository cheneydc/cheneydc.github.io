---
layout: post
title: '通过Mongo, Docker和Rancher创建Node.js应用集群'
category: 技术 
---

自从Rancher的Beta版本发布的几周以来，平台新的调度和服务发现的功能让我略嗨。为了帮助大家理解这些功能，今天我将展示如何利用这些功能部署一个且实现了HA的Node.js应用集群。我会用[Let's Chat](https://github.com/sdelements/lets-chat)作为示例程序，这是一个八错的开源的聊天工具，有点像Slack。Rancher的首席架构师Darren Shepherd在上次的Rancher Meetup上展示了一点Let's Chat，如果有兴趣可以看一下[会议记录](http://rancher.com/recorded-july-online-meetup-implementing-docker-as-a-service/)，但是今天我主要介绍的是部署的一些细节。Let's Chat 是一个自托管的聊天应用，比较适用于小团队，支持很多功能比如：聊天室，通知，警报，XMPP（[可扩展通讯和表示协议](http://baike.baidu.com/link?url=TunroNALJg1pEY_itiAvp1BKshwytPW4zC6ShZaIe8EBHb-_aernJV1n1AbQPpRk4Ot4bgos8QXicwTLqaeara)）聊天，并且还支持多种认证方式。可以从Let's Chat的[github主页](https://github.com/sdelements/lets-chat)上得到更多的信息。

原文： [http://rancher.com/clustering-a-node-js-application-with-mongo-docker-and-rancher/](http://rancher.com/clustering-a-node-js-application-with-mongo-docker-and-rancher/)

[Hussein Galal](http://rancher.com/author/hgalal/) on August 18, 2015

##1 应用所需组件

Let's Chat正常运行需要以下几个组件：
- **Node.js(0.10+)**执行应用代码
- **MongoDB(2.6+)**存储应用数据
- **Python(2.7.X)**支持应用中的一些工具

值得庆幸的是，Let's Chat有自己的Docker image（[sdelements/lets-chat](https://registry.hub.docker.com/u/sdelements/lets-chat/)）了，镜像中安装了应用所有的依赖并且定义了应用所需的环境变量，比如：**LCB_DATABASE_URI**，用来定义单个或者多个MongoDB服务的地址。

部署一个Mongo后端的Let's Chat超简单的，因为已经有可用的Docker compose file了。但我们的目的是建立一个高可用的应用集群，这样可以解决主机故障（host failure咋翻译？感觉是单点故障），所以，我需要起多个MongoDB容器，并创建一个副本集备份多个容器的数据，当主服务节点挂掉的时候副本集可以指定一个新的节点作为主服务节点。

Let's Chat使用[mongoose](https://www.npmjs.com/package/mongoose)驱动来连接MongoDB服务器，通过配置，可以用它连接一个副本集并把数据发送给新的朱服务节点。

##2 Rancher环境

如果还没有Rancher环境，不用担心，很轻松就可以搞定的。总的来说，我们需要用4个机器来部署：一个用来运行Rancher，其他三个用来运行应用的容器。 Rancher可以在任何一个linux（Docker 1.6 or later）上运行。 执行命令：

```
# docker run -d --restart=always -p 8080:8080 rancher/server
```

Rancher将会花几分钟时间启动，使用的是主机的8080端口（命令行中指定的）。一点Rancher启动并开始运行，通过"Add Host"按钮添加其他三个主机到Rancher，可以是通过Docker Machine部署所支持的云上的主机，也可以通过"custom"添加本地主机。（ 翻译的有点纠结，待改进[Once Rancher is up and running, add three new hosts to your environment by clicking the"Add Host" button, and using either the Docker Machine drivers to deploy hosts on one of the supported clouds, or click on "custom" to add any host to your environment.]）

![注册主机](http://rancher.com/wp-content/uploads/2015/08/letschat2.png)

只需复制上图中Step4的命令复制粘贴到需要添加的机器上执行即可将主机注册到Rancher上。命令中包含了Rancher服务端的信息以及注册需要的key，这样就可以将机器自动的注册到Rancher的环境中了。

另外，也可以通过任何一个自动化工具创建Rancher环境。我曾经创建了一个Ansible roles用来在所有的机器上安装Docker并且运行Rancher，也能实现注册机器到Rancher上，有这样几个role：
- [Docker Role](https://github.com/galal-hussein/Docker-Ansible.git)：安装并配置Docker在Ubuntu 14.04上
- [Rancher Role](https://github.com/galal-hussein/Rancher-Server-Ansible.git)：安装并启动Rancher 服务端
- [Rancher Agent](https://github.com/galal-hussein/Rancher-Agent-Ansible.git)：在机器上运行Rancher agent容器并注册到Rancher服务端

可以在playbook中使用这些role来部署Rnacher环境，最后应该有一个3个主机的环境（这里应该说的是Rancher上注册了三个主机[不确定]）。

##3 构建应用集群

环境准备好，就可以开始部署应用了。在Rancher界面的最上面，点击"Application"-> 点击"add stack"，一个stack就是一组相关的service的集合，一个service就是执行特定功能的容器，可以在一个或多个主机上调度。

每一个stack定义了服务发现的范围，应用可以用来连接不同的service，比如我们后面就会在一个新的stack中连接Let's Chat与Mongo数据库。

阅读[Rancher官方文档](http://docs.rancher.com/rancher/rancher-ui/applications/stacks/adding-services/#scheduling-services)可以了解更多service以及其配置信息。我们这次安装主要包括两个service：
- MongoDB数据库 service.
- 应用程序service.

###3.1 部署MongoDB数据库 service

我已经修改了Let's Chat的MongoDB镜像，使每一个容器都能自动的连接到副本集，为了实现这个功能，我写了一个python的小脚本，把它加到了**entrypoint.sh**里面，在原始的镜像中，entrypoint.sh负责启动MongoDB服务，你可以在[Github](https://github.com/galal-hussein/rancher-mongo)中找到。

这个脚本首先检测运行MongoDB的容器的IP，然后将其连接到MongoDB主服务，并把自己加到副本集里面，如果它发现MongoDB服务的数量小于3个，就不会进行连接，因为这表示没有已经启动的副本。（有点费解......[The script will first detect the IP of the running MongoDB container and then tries to connect to the primary MongoDB server and add itself to the existing replica set, if it finds that less than three MongoDB servers exist it will not initiate a connection because it means that no replication has been started.]）

这个镜像需要一个环境变量：**MONGO_SERVICE_NAME**，它的默认值是"mongo"，它的目的是用来查找已经启动的MongoDB容器的IP，这个功能是由Rancher中的DNS服务提供的，所有的容器的IP都会被加到DNS服务中的同一个service名下。

那我们先从MongoDB service开始，点击"Add New Service"进入service的配置页面：

![配置service](http://rancher.com/wp-content/uploads/2015/08/letschat3.png)

注意，配置页面中的command中"-replSet letschat"指定了服务是作为副本集"letschat"的一部分，也可以分配一个主机上的卷，保证数据的安全。

![挂在卷](http://rancher.com/wp-content/uploads/2015/08/letschat4.png)

我们想要在每一个主机上运行一个MongoDB容器，所以在scale里选择**Always run one instance of this containeron every host**，可以使每一个主机上都运行一个容器，实现可扩展。

现在启动这个service就可以看到在每一个主机上都运行了一个容器：

![](http://rancher.com/wp-content/uploads/2015/08/letschat5.png)

###3.2 部署应用程序service

现在创建一个运行应用程序的service，也就是在每一个主机上都运行一个"Let's Chat"容器。我如果用原始的Docker镜像连接分布式的Mongo后端，就会产生些问题。

**_问题_** 第一个问题是当应用程序的service连接到mongo service，比如叫"mongo"，会使let's chat只连接一个MongoDB服务器，但这个服务器可能不是主服务几点，这种情况下写入的数据没有备份。

**_解决办法_** 为了解决这个问题我调整了[let's chat容器](https://github.com/galal-hussein/lets-chat/tree/master/docker/letschat-mod)来执行一个**python**脚本连接Rancher的DNS服务，并查找到所有的MongoDB容器，然后修改Let's Chat的环境变量**LCB_DATABASE_URI**来添加MongoDB容器的IP列表，现在mongoose驱动就会将请求重新路由到主服务器，比如新的LCB_DATABASE_URI：

```
mongodb://x.x.x.x:27017,y.y.y.y:27017/letschat
```

会代替原来的：

```
mongodb://mongo:27017/letschat
```

需要给脚本提供两个环境变量：MONGO_SERVICE_NAME，提供要搜索的mongo服务的名字，用来得到所有相关的容器IP；RANCHER_CLUSTER，如果不想原来的Docker镜像改变可以不使用。

下面开始配置应用程序的service：

![配置应用程序service](http://rancher.com/wp-content/uploads/2015/08/letschat6.png)

注意我添加了mongo as a service links，因为一旦一个service链接另一个的话，一个DNS记录就会自动的被映射到每一个容器上，也可以被其他服务的容器发现。 环境变量的设置：

![环境变量设置](http://rancher.com/wp-content/uploads/2015/08/letschat7.png)

想要看到整个service的配置，可以查看自动生成的docker-compose.yml和rancher-compose.yml文件： docker-compose.yml:

![docker-compose.yml](http://rancher.com/wp-content/uploads/2015/08/letschat8.png)

rancher_compose.yml:

![rancher_compose.yml](http://rancher.com/wp-content/uploads/2015/08/letschat9.png)

###3.3 启动应用

我们启动的第一个service是MongoDB，启动后我们需要对MongoDB服务器间的备份做一些配置：

![](http://rancher.com/wp-content/uploads/2015/08/letschat10.png)

进入到一个mongo服务器，然后执行下面的命令进行初始化：

```
> config = {
    "_id" : "letschat",
    "members" : [
      {"_id" : 0, "host" : "<private-ip-of-the-1st-container>:27017"},
      {"_id" : 1, "host" : "<private-ip-of-the-2nd-container>:27017"},
      {"_id" : 2, "host" : "<private-ip-of-the-3rd-container>:27017"}
   ] }
> rs.initiate(config)
letschat:PRIMARY>
```

这个步骤不需要每一次我们添加或者移除主机的时候都做，修改的MongoDB镜像会处理这个的。启动副本集后，我们可以启动其他的service，可以看到这样的效果：

![应用启动效果](http://rancher.com/wp-content/uploads/2015/08/letschat11.png)

现在如果可以查看下每一个主机上**letschat**容器的日志就可以看到：

```
mongodb://10.42.249.151:27017,10.42.236.117:27017,10.42.211.234:27017/letschat
lets-chat@0.4.2 prestart /usr/src/app
migroose
> lets-chat@0.4.2 start /usr/src/app
> node app.js
██╗     ███████╗████████╗███████╗     ██████╗██╗  ██╗ █████╗ ████████╗
██║     ██╔════╝╚══██╔══╝██╔════╝    ██╔════╝██║  ██║██╔══██╗╚══██╔══╝
██║     █████╗     ██║   ███████╗    ██║     ███████║███████║   ██║
██║     ██╔══╝     ██║   ╚════██║    ██║     ██╔══██║██╔══██║   ██║
███████╗███████╗   ██║   ███████║    ╚██████╗██║  ██║██║  ██║   ██║
══════╝╚══════╝   ╚═╝   ╚══════╝     ╚═════╝╚═╝  ╚═╝╚═╝  ╚═╝   ╚═╝
Release [33m0.4.2[39m
```

这表明修改过的letschat镜像能够检测到所有服务名为**mongo**的已经启动的MongoDB容器。

###3.4 添加Load Balancer

另一个不错的功能是load balancer，下面的例子中我将添加一个load balance，更多的关于Load Balancers的信息，可以参考[文档](http://docs.rancher.com/rancher/rancher-ui/applications/stacks/adding-balancers/)。

首先在stack中点击"Add Load Balancer", 下面的选项使得负载均衡以轮询的方式处理请求，但是会做会话的**Stickiness**配置（the following options will make the load balancer to distribute requests in round robin fashion but with a session stickiness翻译的有点难受啊）

![添加负载均衡](http://rancher.com/wp-content/uploads/2015/08/letschat12.png)

示例中的load balancer连接到了我们已经创的letschat service上，并且会轮询service下的所有容器，,[]下面是stickiness选项，保证同一个会话的请求会路由到相同的服务器上，应用使用了一个名叫**connect.sid**的cookie。

![Stickiness配置](http://rancher.com/wp-content/uploads/2015/08/letschat13.png)

创建好load balancer后，就可以通过访问load balancer的地址来使用应用了：

![访问应用](http://rancher.com/wp-content/uploads/2015/08/letschat14.png)

###3.5 测试扩展性

为了确保应用是可伸缩的，所以创建一个新机器，注册到rancher上，检查配置好的容器会自动运行并添加到集群中：

![这里写图片描述](http://rancher.com/wp-content/uploads/2015/08/letschat15.png) ![这里写图片描述](http://rancher.com/wp-content/uploads/2015/08/letschat16.png)

这表示新的主机已经被成功添加进去，容器也自动的在其上运行了，也可以检查新的mangodb容器，会显示已经添加到了副本集中：

```
2015-07-27T02:09:12.139+0000 I REPL     [WriteReplSetConfig] Starting replication applier threads
2015-07-27T02:09:12.141+0000 I REPL     [ReplicationExecutor] New replica set config in use: { _id: "letschat", version: 6, members: [ { _id: 0, host: "10.42.236.117:27017", arbiterOnly: false, ……...
```

我希望这会对理解怎么使用Rancher调度mongo副本集有帮助。Letschat对于学习部署容器化的，带容错的应用，是一个非常好的例子。如果有兴趣用手上的Rancher尝试一下，可以在github上下载，加入Beta可以获得更多的支持，有任何问题都可以访问我们的论坛，或者注册我们下个月的线上会议提问，和开发们聊。
