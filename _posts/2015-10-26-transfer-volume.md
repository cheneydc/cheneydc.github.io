---
layout: post
title: Transfer Volume(volume过户)
category: 技术 
published: true
---

原文：[Transfer a Volume](http://docs.openstack.org/user-guide/common/cli_manage_volumes.html)

# Transfer a volume

这个功能可以使volume在不同的租户间转移。

通过`cinder transfer*`等命令可以完成volume的过户。对于volume的发送方，需要创建一个transfer，然后将transfer的id和authorization key发送给接收方。接收方得到id和key之后就能通过命令完成volume的过户。

> 适用于接收方和发送方，需要在同一个云环境内。

# 创建transfer

**volume的提供方（所有者）：**

首先要保证volume是`available`状态

{% highlight bash %}
# cinder list
+--------------------------------------+-----------+------------------+------+------+-------------+----------+-------------+-------------+
|                  ID                  |   Status  | Migration Status | Name | Size | Volume Type | Bootable | Multiattach | Attached to |
+--------------------------------------+-----------+------------------+------+------+-------------+----------+-------------+-------------+
| 220dec77-9d4c-4bb5-869e-647efef7921c | available |        -         |  t1  |  5   |      -      |   true   |    False    |             |
+--------------------------------------+-----------+------------------+------+------+-------------+----------+-------------+-------------+
# cinder transfer-create 220dec77-9d4c-4bb5-869e-647efef7921c
+------------+--------------------------------------+
|  Property  |                Value                 |
+------------+--------------------------------------+
|  auth_key  |           568b429714aae29c           |
| created_at |      2015-10-26T08:20:04.398648      |
|     id     | d93180c9-6c4e-45e3-81ac-cfdadd467428 |
|    name    |                 None                 |
| volume_id  | 220dec77-9d4c-4bb5-869e-647efef7921c |
+------------+--------------------------------------+
{% endhighlight %}

> 创建transfer的结果中输出了auth key，但是对于命令`cinder transfer-show`的话，这个key是不可见的。

{% highlight bash %}
# cinder transfer-list
+--------------------------------------+--------------------------------------+------+
|                  ID                  |              Volume ID               | Name |
+--------------------------------------+--------------------------------------+------+
| d93180c9-6c4e-45e3-81ac-cfdadd467428 | 220dec77-9d4c-4bb5-869e-647efef7921c |  -   |
+--------------------------------------+--------------------------------------+------+
# cinder transfer-show d93180c9-6c4e-45e3-81ac-cfdadd467428
+------------+--------------------------------------+
|  Property  |                Value                 |
+------------+--------------------------------------+
| created_at |      2015-10-26T08:20:04.000000      |
|     id     | d93180c9-6c4e-45e3-81ac-cfdadd467428 |
|    name    |                 None                 |
| volume_id  | 220dec77-9d4c-4bb5-869e-647efef7921c |
+------------+--------------------------------------+
{% endhighlight %}


创建好transfer后，将transfer的id和key发送给接收方。

**volume接收方：**

得到transfer的id和auth key后就可以接收了

{% highlight bash %}
# cinder transfer-accept d93180c9-6c4e-45e3-81ac-cfdadd467428 568b429714aae29c
+-----------+--------------------------------------+
|  Property |                Value                 |
+-----------+--------------------------------------+
|     id    | d93180c9-6c4e-45e3-81ac-cfdadd467428 |
|    name   |                 None                 |
| volume_id | 220dec77-9d4c-4bb5-869e-647efef7921c |
+-----------+--------------------------------------+
{% endhighlight %}

> 如果接收方的配额不满足volume的话接收请求被拒绝，但transfer不会消失，直到接收成功或被删除。

```bash
# cinder transfer-accept d93180c9-6c4e-45e3-81ac-cfdadd467428 568b429714aae29c
ERROR: Requested volume or snapshot exceeds allowed gigabytes quota. Requested 5G, quota is 0G and 0G has been consumed. (HTTP 413) (Request-ID: req-173c3f2c-8c73-4aea-8af8-ef1fb0e04d5d)
```
