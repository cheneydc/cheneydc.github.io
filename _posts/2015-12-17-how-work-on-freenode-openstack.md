---
layout: post
title: 怎样加入freenode的openstack频道
category: 工具
published: true
---

openstack相关开发有时候去irc里面与大神们直接交流是很方便有效的，进入频道很方便，下载个irc，ubuntu下面常用的就xchat了，也有windows的版本，不过记得貌似收费的啊。然后起好名字，临时上去的话名字无所谓啊，不过长期交流的话尽量保持名字不变，然后在freenode里面进行注册，这样很方便找人。

注册freenode：

1. 告诉NickServ，偶要注册，然后写好密码和邮箱，有的频道需要验证身份才能进入，验证的时候就要输入密码了。随后freenode会发送一个邮件到你的邮箱：```/msg NickServ REGISTER password youremail@example.com```

2. 收到邮件，意思是收到了一个验证码，验证一下，大致是这个模样：```msg NickServ VERIFY REGISTER yourname xxxxxxx```

3. 如果不想公开邮箱可以设置隐藏： ```/msg NickServ SET HIDEMAIL ON```

4. 验证用户，相当于登录吧```：/msg NickServ IDENTIFY foo password```


官方文档[看这里](http://freenode.net/faq.shtml#nicksetup)
