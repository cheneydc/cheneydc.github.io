---
layout: post
title: SQLalchemmy - 1
category: 技术 
published: true
---

Openstack最近看的有点乱套了，openstack组件都是利用sqlalchemy来操作数据库的，赶上这个机会就看看了。

[官方文档][1]

SQLAlchemy是ORM框架的一种，可以将Python中的类和数据库中的表进行映射，并将类的实例和表数据库表中的数据进行对应。 这样一个抽象的数据库表我们就可以很方便的通过一个类来记录和操作了。

[TOC]

#安装：

```bash
# pip install sqlalchemy
```

# 1 连接数据库
通过`create_engine()`可以连接到数据库:

```python
form sqlalchemy import create_engine
engine = create_engine('sqlite:///:memory:', echo=True)
```

`echo`用来设置SQLAlchemy日志，通过python的logging模块实现，如果不需要则设置为`False`。通过调用`create_engine()`得到的engine是`Engine`的一个实例，包含了连接数据库的主要接口。

> 实际上我们得到engine的时候并没有真正的连接数据库，这里采用的是lazy connecting的方式，真正连接数据库的操作发生在我们第一次访问数据库执行操作的时候。

# 2 定义数据库样式
使用ORM的话，首先我们先要确定数据库表的样式，由于表和python中的类是对应的，这里我们需要定义相应的类。 这种映射关系的类我们需要通过Declarative系统中的`declarative base class`这个基类来实现。我们可以通过`declarative_base()`函数获得：

```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

这样我们就得到了一个可以实现表-类映射关系的基类。我们尝试创建一个表`users`，保存用户信息。创建类`User`来与这个表进行关联。类中我们首先将这个表与之关联，随后定义相应的变量名和相应的类型。

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import create_engine
from sqlalchemy import Column, Integer, String

Base = declarative_base()

engine = create_engine('sqlite:///memory:', echo=True)

class User(Base):
    __tablename__ = 'users'

    id  = Column(Integer, primary_key=True)
    name = Column(String)
    fullname = Column(String)
    password = Column(String)
    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" %(
                        self.name, self.fullname, self.password
                        )
```

> 方法**repr**()是为了方便的显示格式，不是必须的

类中必须有`_tablename_`属性，至少一个primary key的`Column`。

> SQLAlchemy never makes any assumptions by itself about the table to which a class refers, including that it has no built-in conventions for names, datatypes, or constraints. But this doesn't mean boilerplate is required; instead, you're encouraged to create your own automated conventions using helper functions and mixin classes, which is described in detail at [Mixin and Custom Base Classes](http://docs.sqlalchemy.org/en/rel_1_0/orm/extensions/declarative/mixins.html#declarative-mixins).

我们通过Declarative系统创建了`User`类，在类中我们定义了表相关的信息，也就是表的元数据。SQLAlchemy中有一个`Table`对象来表示这些信息，通过`__table__`属性可以得到。

```python
>>> User.__table__
Table('users', MetaData(bind=None), Column('id', Integer(), table=<users>, primary_key=True, nullable=False), Column('name', String(), table=<users>), Column('fullname', String(), table=<users>), Column('password', String(), table=<users>), schema=None)
```

`Table`对象是`MetaData`的一部分，Metadata是Table的集合。Metadata可以发送一些命令在数据库中执行。现在我们的SQLite数据库中实际上还没有`users`表。调用`MetaData.create_all()`方法，将我们的数据库连接engine作为参数传给它。首先执行的命令是查看`users`是否存在，然后是创建表的语句`CREATE_TABLE`:

```python
>>>Base.metadata.create_all(engine)
2015-10-29 17:36:41,372 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2015-10-29 17:36:41,374 INFO sqlalchemy.engine.base.Engine ()
2015-10-29 17:36:41,375 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2015-10-29 17:36:41,375 INFO sqlalchemy.engine.base.Engine ()
2015-10-29 17:36:41,377 INFO sqlalchemy.engine.base.Engine PRAGMA table_info("users")
2015-10-29 17:36:41,378 INFO sqlalchemy.engine.base.Engine ()
2015-10-29 17:36:41,379 INFO sqlalchemy.engine.base.Engine
CREATE TABLE users (
        id INTEGER NOT NULL,
        name VARCHAR,
        fullname VARCHAR,
        password VARCHAR,
        PRIMARY KEY (id)
)

2015-10-29 17:36:41,379 INFO sqlalchemy.engine.base.Engine ()
2015-10-29 17:36:41,410 INFO sqlalchemy.engine.base.Engine COMMIT
```

--------------------------------------------------------------------------------

> **Minimal Table Descriptions vs. Full Descriptions** 熟悉CREATE_TABLE的可以看出来VARCHAR这里是没有长度的，对于数据库SQLite和Postgresql，这是允许的，但是其他的一些数据库，这个不可以的。所以如果要正确执行需要指定String的长度：

```python
Column(String(50))
```

> Firebird和Oracle会要求给主键设定sequences，SQLAlchemy在没有明确制定的情况下是不指定的，需要的话需要自己指定：

```python
from sqlalchemy import Sequence
Column(Integer, Sequence('user_id_seq'), primary_key=True)
```

> 所以一个完整的定义应该像这样：

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, create_engine, Sequence

Base = declarative_base()

engine = create_engine('sqlite:///memory:', echo=True)

class User(Base):
    __tablename__ = 'users'

    id  = Column(Integer, Sequence('user_id_seq'), primary_key=True)
    name = Column(String(60))
    fullname = Column(String(60))
    password = Column(String(16))
    def __repr__(self):
        return "<User(name='%s', fullname='%s', password='%s')>" %(
                        self.name, self.fullname, self.password
                        )
```

--------------------------------------------------------------------------------

# 3 创建实例
与数据库中的表相关的类我们已经创建好了，下面创建一个实例：

```python
user_1 = User(name='dc', fullname='cheneydc', password='nopassword')
print "user name is:    %s"  % (user_1.name)
print "user id is:      %s"  % (str(user_1.id))
```

执行后的结果：

```bash
user name is:    dc
user id is:      None
```

这里我们没有给id一个值，但id这里是`None`（Python通常对没有定义的属性会报AttributeError错误）。SQLAlchemy的[instrumentation][2] 在第一次访问的时候会为这些和数据库表有映射关系的属性一个默认值。对于我们已经赋值的属性，`instrument`系统会跟踪赋值的过程直到最后的INSERT语句在数据库中执行。

# 4 创建Session
到这里我们就可以开始与数据库交互了。ORM与数据库进行交互的句柄就是`Session`。在程序的开头，create_engine()的地方，我们定义一个`Session`类，通过它来创建session。

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
```

如果程序还没有创建engine的情况下，可以只这样写：

```python
Session = sessionmaker()
```

随后创建好engine可以用`configure()`进行关联：

```python
Session.cinfigure(bind=engine)
```

这样一个Session类创建的session对象都是与我们的数据绑定的。想要操作数据库的话就需要创建一个session对象：

```python
session = Session()
```

上面创建的session就与数据库相关联了，但像我们上面说的，并没有真正的与数据库建立市实际的连接，第一次操作数据库的时候连接才被建立，直到提交了所有变更或者关闭了session对象。

# 5 添加对象
为了保存User对象，我们需要将对象添加到Session中：

```python
>>> user_2 = User(name='cc', fullname='cheney', password='no')
>>> session.add(user_2)
```

这个时候，实际上SQL语句没有执行，对象也没有被实际写入到数据库中，session实例处于**pending**状态，只有执行了**flush**操作后，Session才会将对象写入数据库。如果当我们执行在数据库中查找`user_2`的操作的时候，所有的`pending`状态的数据会首先被`flushed`，之后才会执行查找。

比如我们创建一个`Query`对象并加载`User`。指定查找`name`为`cc`的用户，指明只需要查找结果中的第一个。查找结果就返回一个我们添加的`User`实例：

```python
>>> query_user = session.query(User).filter_by(name='cc').first()
2015-11-02 10:15:10,865 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.name = ?
 LIMIT ? OFFSET ?
2015-11-02 10:15:10,865 INFO sqlalchemy.engine.base.Engine ('cc', 1, 0)
>>> query_user is user_2
True
>>> query_user
<User(name='cc', fullname='cheney', password='no')>
```

实际上Session识别出返回的row和内部映射的对象中的一个是一样的（我们刚刚添加的一个对象），所以实际上我们得到的是和添加的同一个对象。

对于一个在Session中存在的primary key，所有在session中查询这个key，都会返回这个特定key的Python对象。

我们可以用add_all添加多个User对象：

```python
>>> user_3 = User(name='ed', fullname='eeeddd', password='qweasd')
>>> user_4 = User(name='zxc', fullname='zxcvasdf', password='098765')
>>> session.add_all([user_3, user_4])
```

我们对已经添加的对象做一些修改：

```python
>>> user_1.password = "123456"
>>> user_2.name='ccc'
```

Session会记录这些修改，新添加的对象记录在new属性中，修改过的对象记录在dirty中：

```python
>>> session.new
IdentitySet([<User(name='ed', fullname='eeeddd', password='qweasd')>, <User(name='zxc', fullname='zxcvasdf', password='098765')>])
>>> session.dirty
IdentitySet([<User(name='dc', fullname='cheneydc', password='123456')>, <User(name='ccc', fullname='cheney', password='no')>])
```

通过commit()，session会提交所有的更改，并结束与数据库的交互。

```python
>>> session.commit()
```

commit()会对所有的更改做flush操作，然后结束与数据库的事务（这里是transaction ，而不是前面说的engine的连接）。后续有其他操作的话实际都是发生在一个新的事务上，操作之前会首先自动连接数据库并获取相关资源。

# 6 回滚
由于Session的操作都是在事务（transaction）内执行的，在这个事务结束之前，我们也可以执行回滚操作。现在对对象做一些更改：

```python
>>> user_1.name='dongcong'
>>> session.dirty
IdentitySet([<User(name='dongcong', fullname='cheneydc', password='123456')>])
```

添加一个新对象：

```python
>>> fake_user = User(name='fake', fullname='TestUser', password="Nopassword")  
>>> session.add(fake_user)
```

然后执行一个查询操作，使更改有效的写入当前的事务中：

```python
>>> session.query(User).filter(User.name.in_(['dongcong', 'fake'])).all()
2015-11-02 11:11:37,085 INFO sqlalchemy.engine.base.Engine UPDATE users SET name=? WHERE users.id = ?
2015-11-02 11:11:37,085 INFO sqlalchemy.engine.base.Engine ('dongcong', 1)
2015-11-02 11:11:37,087 INFO sqlalchemy.engine.base.Engine INSERT INTO users (name, fullname, password) VALUES (?, ?, ?)
2015-11-02 11:11:37,088 INFO sqlalchemy.engine.base.Engine ('fake', 'TestUser', 'Nopassword')
2015-11-02 11:11:37,091 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.name IN (?, ?)
2015-11-02 11:11:37,092 INFO sqlalchemy.engine.base.Engine ('dongcong', 'fake')
[<User(name='dongcong', fullname='cheneydc', password='123456')>, <User(name='fake', fullname='TestUser', password='Nopassword')>]
```

目前，事务还没有结束，执行回滚操作:

```python
>>> session.rollback()
>>> user_1.name
>>> dc
>>> fake_user in session
False
```

# 7 查询
查询对象是通过Session中的[query()][3]方法创建的。采用的是可变数量的参数，任意数目的类实体或者基于列的实体均可以作为query()方法的参数`can be any combination of classes and class-instrumented descriptors`。下面加载User实例进行查询:

```python
>>> for instance in session.query(User).order_by(User.id):
...     print instance.name, instance.fullname
...
2015-11-02 13:20:31,632 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users ORDER BY users.id
2015-11-02 13:20:31,633 INFO sqlalchemy.engine.base.Engine ()
aaa cheneydc
ccc cheney
ed eeeddd
zxwer zxcvasdf
fake TestUser
```

这个例子中返回的是一组可迭代的User列表，然后通过for...in...语句输出name和fullname，如果不关心其他属性，也可以直接查询想要的类属性，前提是这个类需要时ORM映射的：

```python
>>> for name,fullname in session.query(User.name, User.fullname).all():
...     print name, fullname
...
2015-11-02 15:16:20,691 INFO sqlalchemy.engine.base.Engine SELECT users.name AS users_name, users.fullname AS users_fullname
FROM users
2015-11-02 15:16:20,691 INFO sqlalchemy.engine.base.Engine ()
aaa cheneydc
ccc cheney
ed eeeddd
zxwer zxcvasdf
fake TestUser
```

返回的仍然是元组（基于KeyedTuple类），可以当做普通的Python对象对待。属性名称归属性名称，类型名称归类型名称：

```python
>>> for row in session.query(User, User.name).all():
...     print row
...
2015-11-02 15:35:17,303 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
2015-11-02 15:35:17,303 INFO sqlalchemy.engine.base.Engine ()
(<User(name='aaa', fullname='cheneydc', password='123456')>, u'aaa')
(<User(name='ccc', fullname='cheney', password='no')>, u'ccc')
(<User(name='ed', fullname='eeeddd', password='qweasd')>, u'ed')
(<User(name='zxwer', fullname='zxcvasdf', password='098765')>, u'zxwer')
(<User(name='fake', fullname='TestUser', password='Nopassword')>, u'fake')
```

也可以通过label()方法来改变列表达式的名称:

```python
>>> for row in session.query(User.name.label('name_label')).all():
...     print row.name_label
...
2015-11-02 15:37:48,805 INFO sqlalchemy.engine.base.Engine SELECT users.name AS name_label
FROM users
2015-11-02 15:37:48,806 INFO sqlalchemy.engine.base.Engine ()
aaa
ccc
ed
zxwer
fake
```

可以发现每次查询都会用到实体类的名称User，可以用aliased()给它一个别名：

```python
>>> for row in session.query(user_alias, user_alias.name).all():
...     print row
...
2015-11-02 15:42:58,489 INFO sqlalchemy.engine.base.Engine SELECT user_alias.id AS user_alias_id, user_alias.name AS user_alias_name, user_alias.fullname AS user_alias_fullname, user_alias.password AS user_alias_password
FROM users AS user_alias
2015-11-02 15:42:58,489 INFO sqlalchemy.engine.base.Engine ()
(<User(name='aaa', fullname='cheneydc', password='123456')>, u'aaa')
(<User(name='ccc', fullname='cheney', password='no')>, u'ccc')
(<User(name='ed', fullname='eeeddd', password='qweasd')>, u'ed')
(<User(name='zxwer', fullname='zxcvasdf', password='098765')>, u'zxwer')
(<User(name='fake', fullname='TestUser', password='Nopassword')>, u'fake')
```

LIMIT和OFFSET经常用来控制查询的数量和位置，这个可以很方便的用Python中的数组分页与ORDER BY组合完成：

```python
>>> for row in session.query(User).order_by(User.id):
...     print row
...
2015-11-02 15:53:40,182 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password FROM users ORDER BY users.id
2015-11-02 15:53:40,182 INFO sqlalchemy.engine.base.Engine ()
<User(name='aaa', fullname='cheneydc', password='123456')>
<User(name='ccc', fullname='cheney', password='no')>
<User(name='ed', fullname='eeeddd', password='qweasd')>
<User(name='zxwer', fullname='zxcvasdf', password='098765')>
<User(name='fake', fullname='TestUser', password='Nopassword')>

>>> for row in session.query(User).order_by(User.id)[1:3]:
...     print row
...
2015-11-02 15:54:01,541 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users ORDER BY users.id
 LIMIT ? OFFSET ?
2015-11-02 15:54:01,542 INFO sqlalchemy.engine.base.Engine (2, 1)
<User(name='ccc', fullname='cheney', password='no')>
<User(name='ed', fullname='eeeddd', password='qweasd')>
```

筛选的功能通过filter_by()实现，使用关键词参数：

```python
>>> for row in session.query(User).filter_by(fullname='cheney'):
...     print row
...
2015-11-02 16:09:13,818 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.fullname = ?
2015-11-02 16:09:13,818 INFO sqlalchemy.engine.base.Engine ('cheney',)
<User(name='ccc', fullname='cheney', password='no')>
```

用filter()也能实现，filter使用了更加灵活的SQL语言结构。这允许在处理映射类的属性时候更多的利用python的常规操作符：

```python
>>> for row in session.query(User).filter(User.fullname=='cheney'):            
...     print row
...
2015-11-02 16:18:39,058 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.fullname = ?
2015-11-02 16:18:39,059 INFO sqlalchemy.engine.base.Engine ('cheney',)
<User(name='ccc', fullname='cheney', password='no')>
```

Query对象是完全generative，意味着大部分调用的方法返回的是一个Query对象，这样可以在此基础上增加新的条件。比如查询user的name是"ccc"，fullname是"cheney"，可以通过调用两次filter()完成，相当于两个条件AND的结果：

```python
>>> for row in session.query(User).filter(User.fullname=='cheney').filter(User.name=='ccc'):
...     print row
...
2015-11-02 16:24:51,982 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.fullname = ? AND users.name = ?
2015-11-02 16:24:51,984 INFO sqlalchemy.engine.base.Engine ('cheney', 'ccc')
<User(name='ccc', fullname='cheney', password='no')>
```

## 常用筛选操作
列举一些filter()常用的过滤操作：
- equals:

```python
query.filter(User.name == 'ccc')
```

- not equals:

```python
query.filter(User.name != 'ccc')
```

- LIKE:

```python
>>> for row in session.query(User).filter(User.fullname.like('%ene%')):
...     print row
...
2015-11-02 16:29:13,320 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.fullname LIKE ?
2015-11-02 16:29:13,321 INFO sqlalchemy.engine.base.Engine ('%ene%',)
<User(name='aaa', fullname='cheneydc', password='123456')>
<User(name='ccc', fullname='cheney', password='no')>
```

- IN:

```python
query.filter(User.name.in_(['ccc', 'cheneydc', 'aaa']))
```

- NOT IN:

```python
query.filter(~User.name.in_(['ccc', 'cheneydc', 'aaa']))

#example.
>>> for row in session.query(User).filter(~User.fullname.in_(['cheneydc', 'cheney'])):
...     print row
...
2015-11-02 16:33:49,204 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password FROM users
WHERE users.fullname NOT IN (?, ?)
2015-11-02 16:33:49,205 INFO sqlalchemy.engine.base.Engine ('cheneydc', 'cheney')
<User(name='ed', fullname='eeeddd', password='qweasd')>
<User(name='zxwer', fullname='zxcvasdf', password='098765')>
<User(name='fake', fullname='TestUser', password='Nopassword')>
```

- IS NULL:

```python
query.filter(User.name == None)

# alternatively, if pep8/linters are a concern
query.filter(User.name.is_(None))
```

- IS NOT NULL:

```python
query.filter(User.name != None)

# alternatively, if pep8/linters are a concern
query.filter(User.name.isnot(None))
```

- AND:

```python
# 使用 and_()，这个and_不是python中的and操作符
from sqlalchemy import and_
query.filter(and_(User.name == 'ccc', User.fullname == 'cheney'))

# 发送多个表达式给.filter()
query.filter(User.name == 'ccc', User.fullname == 'cheney')

# 多次连续调用filter
query.filter(User.name == 'ccc').filter(User.fullname == 'cheney')
```

- OR:

```python
# 使用or_,这里的or_不是python中的or操作符
from sqlalchemy import or_
query.filter(or_(User.name == 'ccc', User.name == 'cheney'))
```

- MATCH：

```python
# match()使用的是数据库的MATCH或CONTAINS方法，依赖于后端的数据库，所以并不是所有数据库都支持，比如SQLite
query.filter(User.name.match('wendy'))
```

## 返回列表和标量
一些Query的方法会立即发出SQL语句并返回一个包含数据库结果的值。这里做简单介绍：
- all()返回一个列表：

```python
>>> query = session.query(User).filter(~User.name.in_(['a', 'b']))
>>> query.all()
2015-11-02 17:38:55,658 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.name NOT IN (?, ?)
2015-11-02 17:38:55,659 INFO sqlalchemy.engine.base.Engine ('a', 'b')
[<User(name='aaa', fullname='cheneydc', password='123456')>, <User(name='ccc', fullname='cheney', password='no')>, <User(name='ed', fullname='eeeddd', password='qweasd')>, <User(name='zxwer', fullname='zxcvasdf', password='098765')>, <User(name='fake', fullname='TestUser', password='Nopassword')>]
```

- first()将第一个结果作为标量返回：

```python
>>> query.first()
2015-11-02 17:40:29,597 INFO sqlalchemy.engine.base.Engine SELECT users.id AS users_id, users.name AS users_name, users.fullname AS users_fullname, users.password AS users_password
FROM users
WHERE users.name NOT IN (?, ?)
 LIMIT ? OFFSET ?
2015-11-02 17:40:29,597 INFO sqlalchemy.engine.base.Engine ('a', 'b', 1, 0)
<User(name='aaa', fullname='cheneydc', password='123456')>
```

[1]: http://docs.sqlalchemy.org/en/rel_1_0/orm/tutorial.html
[2]: http://docs.sqlalchemy.org/en/rel_1_0/glossary.html#term-instrumentation
[3]: http://docs.sqlalchemy.org/en/rel_1_0/orm/session_api.html#sqlalchemy.orm.session.Session.query
