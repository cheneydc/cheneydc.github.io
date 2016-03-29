---
layout: post
title: Mistral小记
category: 技术
published: true
---

组里开始是拿来当crontab来使用的，这也是我最早对mistral的印象，云环境下的很多新项目其实也就是传统服务器上的应用的一个迁移。Mistral可以当作crontab来使用，但是远不止这些，Mistral实际是workflow-as-a-service，提供云环境下的工作流服务。更像是crontab+ansible的东西。

I think,工作流是一些动作或者操作的集合，按照一定的规则或者顺序执行，实现期望的结果。Mistral就是提供了这样的服务。下面的图是官方文档中的架构图：

# 结构
![Architecture](http://docs.openstack.org/developer/mistral/_images/mistral_architecture.png)

- engine: 处理工作流以及其中的数据流。判断任务状态，将任务放入任务队列，在任务间传递数据。
- Task Executor:执行任务动作，从对立中取出任务并执行具体动作，最后将结果返回给engine
- API server：提供REST API
- Scheduler:保存并执行延时任务，与engine和executor进行交互，并根据事件以触发workflow
- Persistence：存储数据

# Mistral的DSL
Mistral有自己的工作流描述语言，汲取了[YAQL](https://pypi.python.org/pypi/yaql/1.0.0)优点，主要包括两个部分：

- Workflow
- Action

## Wrokflow
Workflow是Mistral的主体部分，一个Workflow由至少一个task组成，task描述了具体的执行步骤，Workflow则描述了task之间的执行顺序、方式，以及输入、输出等。

看一个例子：

```
---
version: '2.0'

create_vm:
  description: Simple workflow example
  type: direct

  input:
    - vm_name
    - image_ref
    - flavor_ref
  output:
    vm_id: <% $.vm_id %>

  tasks:
    create_server:
      action: nova.servers_create name=<% $.vm_name %> image=<% $.image_ref %> flavor=<% $.flavor_ref %>
      publish:
        vm_id: <% task(create_server).result.id %>
      on-success:
        - wait_for_instance

    wait_for_instance:
      action: nova.servers_find id=<% $.vm_id %> status='ACTIVE'
      retry:
        delay: 5
        count: 15
```

Wrokflow名字定义为"create_vm",需要输入三个参数：`vm_name`,`image_ref`,`flavor_ref`,执行结束后输出`vm_id`,整个过程需要两个任务完成，首先`create_server`,通过调用openstack相应的action，创建虚机，task可以使用workflow的输入进行传参，执行结束将`vm_id`返回给workflow，`on-success`指定如果task执行成功的动作，这里跳转到`wait_for_instance`,通过`retry`在设定时间和次数内轮询虚机的状态，确保虚机状态为ACTIVE后执行结束。通过这个例子可以大致看出workflow的基本结构。查看Workflow定义的具体关键字和属性列表请[点击这里](http://docs.openstack.org/developer/mistral/dsl/dsl_v2.html#common-workflow-attributes)。

对任务的流程主要有两种处理方式也很清晰，主要有两种方式：

- Direct Workflow
- Reverse Workflow

### Direct Workflow
这种方式可以简单理解为正向流程，一个任务结束，触发另一个或多个任务，直到流程结束。主要通过三个属性完成：

- on-success:此任务执行成功后需要执行的任务列表。
- on-error:此任务执行出错后需要执行的任务列表。
- on-complete:此任务执行结束后（不管成功还是失败）需要执行的任务列表

比如：

```
tasks:
  A:
    action: action.x
    on-success:
      B

  B:
    action: action.y
    on-success:
      C

  C:
    action: action.z
    publish:
      ret
```

简单写一个例子，任务A成功后执行B，B成功后执行C，这就是一个最简单的顺序执行的方式。多个任务有可以有很多种触发方式。

如果一个任务结束，需要触发其他多个任务，称为`Fork`。

```
tasks:
  A:
    action: action.x
    on-success:
      - B
      - C
  B:
    action: action.y

  C:
    action: action.z
    publish:
      ret
```

任务A执行成功后需要触发Task B和Task C，这里B和C的执行是并行的。再换一种方式：

```
tasks:
  A:
    action: action.x
    on-success:
      C

  B:
    action: action.y
    on-success:
      C

  C:
    join: all
    action: action.z
    publish:
      ret
```

这种方式Task A执行成功会执行C，然而任务B执行成功也会执行C，C的执行可以通过`join`来指定，这个例子中`join`的值是`all`，这种情况下，C会在A、B都执行成功后再执行，如果指定`join`是一个值N，则会在上有任务至少有N个满足执行条件的时候执行。这种方式就是`Join`。

### Reverse Wrokflow
在这个类型的Wrokflow中，任务的关系是被动依赖的，比如要执行A，指定必须先执行B，这样的关系可以在这种类型的Workflow中进行处理。可以使用

 - requires: 在执行此任务前需要执行的任务列表。比如：

```
 tasks:
   A:
     action: action.x

   B:
     action: action.y

   C:
     action: action.z
       requires: [A，B]
```

## Action
Action是每一步要执行的内容，Mistra提供了一些标准的Action，并对openstack的各个组件的API进行了处理，都提供了相应的Action，众多的Action，如果可以方便的查询，就基本解决了使用的最大问题，通过mistral提供的工具可以将这些Action导入到数据库：

```
mistral-db-manage --config-file /path/to/mistral.conf populate
```

导入后，可以获得action的列表：

```
mistral action-list
```

查询某一个action的具体参数，返回值等信息可以通过：

```
mistral action-get xxx
```

## WorkBook
WorkBook相当于一个命名空间，可以将workflow和task归到一个workbook中，导入一个workbook后，其中的workflow和action以及trigger也都会独立的被创建，与普通创建他们是一样的，但是通过workbook创建的，名字会以workbook的名字为前缀。

先整理到这，像cron-trigger之类的查查命令行就比较容易看懂了，mistral看起来功能比较单一，但是放在云环境中，应用点可以很多，对于业务的部署，甚至虚拟数据中心的管理都是很有帮助的，可以通过Mistral提供统一的操作，并且也很方便对工作流进行迁移。

各个部分具体的属性可以参考[官方文档](http://docs.openstack.org/developer/mistral/dsl/dsl_v2.html#mistral-dsl-v2-specification)。
