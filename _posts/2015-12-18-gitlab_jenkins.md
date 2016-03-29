---
layout: post
title: Gitlab与jenkins集成
category: 工具
published: true
---

熟悉了一下openstack社区的工作流程，初觉繁琐，用下来觉得异常清晰，CI虽然稳定欠佳，但是一些基本的单元测试等苦累之活可以交与它的。所以想着怎么把gitlab和jenkins集成一下。

# 偷懒搭建jenkins
只是一次探索，就偷个懒，用docker拉一个镜像搞定

```
$ docker pull jenkins

```

# 安装Gitlab plugin
在jenkins的插件管理界面，在可选插件中搜索gitlab hook plugin，然后安装

# 任务中配置触发器
任务中的构建触发器中，开启"Poll SCM"

# Gitlab hook
在gitlab的项目设置界面，有一个web hook配置选项，这里可以配置触发类型和触发链接，触发链接：

```
http://$jenkins_url/git/notifyCommit?url=$project_git_url
```

将$jenkins_url替换成自己的jenkin地址，$project_git_url替换成项目的git地址。添加成功后会有一个测试按钮，可以点击“TEST HOOK”,看看jenkins的任务是否被触发。
