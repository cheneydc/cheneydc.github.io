---
layout: post
title: image-volume cache
category: 技术 
published: true
---

原文： [Image-Volume cache](http://docs.openstack.org/admin-guide-cloud/blockstorage_image_volume_cache.html)

Openstack块存储有一个可选的功能，镜像缓存，可以提升从镜像创建卷的性能。这个改进依赖很多因素，主要看存储的后端克隆卷的速度有多快。

当一个卷第一次从镜像创建的时候，块存储的内部租户也会创建一个image-volume缓存。之后如果再请求从这个image创建volume的话就直接clone这个缓存，不需要再下载镜像的内容然后复制数据到卷里了。

每个后端可以配置自己的缓存，可以包含大部分常用的镜像。

## 设置内部租户
image-volume缓存需要配置块存储的内部租户。这个租户拥有这些缓存并且可以进行管理。这可以保护用户不必看到这些缓存，但是也没有全局隐藏。

使块存储服务能进入内部租户，需要在conder.conf配置：

{% highlight bash %}
cinder_internal_tenant_project_id = PROJECT_ID 
cinder_internal_tenant_user_id = USER_ID
{% endhighlight %}

> 为内部租户配置的用户和项目不需要特殊的额权限。可以使块存储服务的租户或者是其他任何的项目和用户。

## 配置image-volume cache
在cinder.conf中配置如下：

{% highlight bash %}
image_volume_cache_enabled = True
{% endhighlight %}

每个后端可以独立定义或者在默认项里定义。

缓存的大小和数量限制也可以在每个后端独立定义，或者在默认项里定义。

{% highlight bash %}
image_volume_cache_max_size_gb = SIZE_GB
image_volume_cache_max_count = MAX_COUNT
{% endhighlight %}

默认被设置为0，表示不限制。

## 通知
缓存操作会触发Telemetry消息。这几种会被发送：

 - image\_volume\_cache.miss - 创建卷时，没有从缓存中找到相应的镜像。这意味着一个新的缓存会被创建
 - image\_volume_cache.hit - 从镜像创建卷时，在缓存中找到了相应的镜像并将被使用。
 - image\_volume\_cache.evict - 一个镜像的缓存被删除。

## 管理image-volume缓存
通常来说，缓存都是自动管理的，不需要手动干预。
如果需要，可以手动删除缓存。通过标准的卷删除API，块存储服务会执行清理。
 
# 注意：

代码可以看出，如果直接clone的话，就不会创建cache了


{% highlight python %}
# vim cinder/volume/flows/manager/create_volume.py
...
    def _create_from_image(self, context, volume_ref,
                           image_location, image_id, image_meta,
                           image_service, **kwargs):
        LOG.debug("Cloning %(volume_id)s from image %(image_id)s "
                  " at location %(image_location)s.",
                  {'volume_id': volume_ref['id'],
                   'image_location': image_location, 'image_id': image_id})

        # Create the volume from an image.
        #
        # First see if the driver can clone the image directly.
        #
        # NOTE (singn): two params need to be returned
        # dict containing provider_location for cloned volume
        # and clone status.
        model_update, cloned = self.driver.clone_image(context,
                                                       volume_ref,
                                                       image_location,
                                                       image_meta,
                                                       image_service)

        # Try and clone the image if we have it set as a glance location.
        if not cloned and 'cinder' in CONF.allowed_direct_url_schemes:
            model_update, cloned = self._clone_image_volume(context,
                                                            volume_ref,
                                                            image_location,
                                                            image_meta)

        # Try and use the image cache.
        should_create_cache_entry = False
        internal_context = cinder_context.get_internal_tenant_context()
        if not internal_context:
            LOG.warning(_LW('Unable to get Cinder internal context, will '
                            'not use image-volume cache.'))


----------


        if not cloned and internal_context and self.image_volume_cache:
            model_update, cloned = self._create_from_image_cache(
                context,
                internal_context,
                volume_ref,
                image_id,
                image_meta
            )
            if not cloned:
                should_create_cache_entry = True


----------
...
{% endhighlight %}
