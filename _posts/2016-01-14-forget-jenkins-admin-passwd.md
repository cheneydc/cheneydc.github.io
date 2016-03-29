---
layout: post
title: 忘了jenkins管理员密码了
tag: 技术
publish: true
---

好不容易配置好了jenkins，走的LDAP，然后脑残的设置了安全矩阵，进不去了吧...还好可以改配置文件救回来

```
# vim $JENKINS_HOME/config.xml

  <useSecurity>true</useSecurity>

```

把true改成false，然后重启jenkins就好了
