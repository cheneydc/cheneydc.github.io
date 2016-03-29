---
layout: post
title: VDI之ovirt
category: 技术
published: true
---

最早接触虚拟化的东西是08、09左右上学玩虚拟机，搭个环境弄下ubuntu什么的，后来学习编程，虚拟机天天跑，那时候觉得挺神奇的，点一点就在我的电脑里弄出来一个机器了，但是性能比较差，图形界面一般是不敢开的;后来到红帽也是意外的很，做了虚拟化相关的测试，接触了libvirt、kvm之类的，从技术上对虚拟化有了些了解，然后接触了RHEV，如果说从一个用户角度去看的话，这是让我惊艳的地方了。

RHEV是红帽的虚拟化管理平台，社区相应的是ovirt，在体验了迁移的效果后我表示很happy，通过spice使用windows虚机的效果也不错，脑洞放大点的话，以后每个人就不需要电脑了，如果突破了显示和能源的显示，只需要一个小小的终端，随时随地都可以访问自己远程的机器（主机 or 手机也没有分别），计算资源统一管理，从环保角度看也是不错的。

这个YY不是不可能的，但是当时接触的ovirt功能还是比较单一的，对于资源的调度和管理相对于云平台要弱势很多，最近又看了下ovirt，发现ovirt也是加了些好玩的东西啊。ovirt可以加入外部服务这个功能可以弥补N多弱点，比如ovirt可以直接加入openstack的neutron服务，如果没有虚拟网络，ovirt上的虚机就会占用IP资源，加入neutron，虚机可以使用虚机网络，对于隔离和管理都很方便了;加入cinder后，ovirt的卷也可以通过cinder进行管理，也可以很方便的使用ceph存储。

# Neutron in ovirt
Neutron在openstack中提供网络即服务的组建，ovirt加入neutron服务，正可以弥补在对于虚机网络的配置和管理功能的缺陷，目前ovirt3.5版本后加入了这个功能，官方有非常详细的文档[http://www.ovirt.org/develop/release-management/features/cloud/neutronvirtualappliance/](http://www.ovirt.org/develop/release-management/features/cloud/neutronvirtualappliance/),也可以[点击这里下载](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/document/ovirt-over-all-docs.tar.gz)。文档非常详细，整体的网络拓扑：

![neutron appliance拓扑图](http://7xqb88.com1.z0.glb.clouddn.com/20160301-ovirt-over-all-1.png)

这里ovirt提供了一个集成了keystone以及neutron服务的镜像，用这个镜像启动虚机(neutron appliance )就可以直接提供neutron服务，主机两块网卡分别eth0、eth1,eth0相当于管理网络也可以提供外部网络，eth1走数据网络，用于虚机之间的通信。另外提一点，镜像应该用的是红帽RDO搭建的服务，基于I版本的。

不足之处，ovirt管理界面中只提供了子网的建立，如果要顺利使用需要通过命令行手动进行路由的配置。不过后面应该会集成进来的吧。

# Cinder in ovirt
cinder在openstack中提供块存储服务，对于卷和快照的管理也很方便，支持N多种后端，如果想在ovirt中使用ceph存储，通过接入cinder服务的方式简单的不得了啊，不过此功能在ovirt3.6开始支持。官方文档是好东西，不过ovirt最近网站更新改版了，好多格式没法看啊，幸好之前留了pdf，可以[点击这里下载](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/document/ovirt-over-all-docs.tar.gz)。

添加cinder服务方便，如果ceph配置认证的话还需要配置key，这个可以在UI直接操作了。

# ovirt HA
ovirt-engine是ovirt的核心服务了，关键服务的HA功能也是一个产品的可靠保证，ovirt提供了ovirt-host-engine-setup的工具，进行HA部署，ovirt的HA是虚机层的HA，工具会先启动一个虚拟环境，在这个虚拟环境里安装一个虚机，把ovirt-engine装到虚机里面，然后服务再管理这个主机，用同样的工具将其他主机添加到服务中，如果ovirt-engine被检测挂了，会在其他主机启动这个虚机，所以恢复服务时间稍微长了一些，如果做到了服务层的HA就好了。具体的部署也请看文档，[点击这里下载](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/document/ovirt-over-all-docs.tar.gz)

“云”的发展快的吓人啊，不过从公有云到私有云到混合云，其实更多的在追求的是“简”，大道至简啊，消除差异化，提供标准统一，稳定的平台和服务。将来openstack与VDI可能也会渐渐的打破界限，甚至融为一体，但是从目前ovirt的效果来看，性能以及稳定性是更首要的问题。
