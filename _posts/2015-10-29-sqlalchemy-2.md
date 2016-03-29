---
layout: post
title: SQLalchemmy - 2
category: 技术 
published: true
---

这个就用点记点了。

# func

```python
>>> q = session.query(User.id, User.name)
>>> q.all()
[(1, u'aaa'), (2, u'ccc'), (3, u'ed'), (4, u'zxwer'), (5, u'fake'), (6, u'dc'), (7, u'dc'), (8, u'dc')]
>>> q1 = session.query(func.sum(User.id))
>>> q1.all()
2015-11-11 22:06:24,312 INFO sqlalchemy.engine.base.Engine SELECT sum(users.id) AS sum_1
FROM users
2015-11-11 22:06:24,312 INFO sqlalchemy.engine.base.Engine ()
[(36,)]
>>> q1 = session.query(func.sum(User.id*2))
>>> q1.all()
2015-11-11 22:06:55,466 INFO sqlalchemy.engine.base.Engine SELECT sum(users.id * ?) AS sum_1
FROM users
2015-11-11 22:06:55,466 INFO sqlalchemy.engine.base.Engine (2,)
[(72,)]
>>>
```
