---
layout: post
title: 配置horizon支持keystone v3 api
category: 技术 
published: true
---

From:
[https://docs.hpcloud.com/commercial/GA1/1.1commerical.services-identity-configure-v3.html](https://docs.hpcloud.com/commercial/GA1/1.1commerical.services-identity-configure-v3.html)

To switch Horizon from Keystone v2 to Keystone v3:

1. In each controller node, navigate to the local_settings.py file located in /opt/stack/venvs/openstack/lib/python2.7/site-packages/openstack_dashboard/local/local_settings.py

2. Edit the file as follows:

a. Set the OpenStack API version to version 3:

```
OPENSTACK_API_VERSIONS = {
"identity": 3,
}
```

b. Enable multi-domain support:

```
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
```

c. Point to Keystone V3 endpoint:

```
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
```

The Keystone v3 endpoint is in the format: http://<host>:<port>/v3 and is the same host/port as the v2 endpoint. The local_settings.py file will have the endpoint for v2 by default.

3. Restart the apache server in each controller nodes:

```
sudo service apache2 restart
```
