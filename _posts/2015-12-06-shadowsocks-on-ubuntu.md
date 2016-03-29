---
layout: post
title: ubuntu配置shadowsocks
category: 工具
published: true
---

还是用ubuntu舒服，不过发现装完系统shadowsocks不好链接啊，想办法配置一下：

先安装shadowsocks

```bash
$ sudo apt-get install shadowsocks
```

写个配置文件：

```bash
$ cat config.conf
{
"server":"",
"server_port":,
"local_address":"127.0.0.1",
"local_port":1080,
"password":"",
"timeout":300,
"method":"rc4-md5"
}
```

做成系统启动服务， 将[启动脚本](https://github.com/cheneydc/linux_profile/blob/master/shadowsocks/sslocal)放到`/etc/init.d/`
