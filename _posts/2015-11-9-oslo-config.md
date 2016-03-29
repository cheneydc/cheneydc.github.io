---
layout: post
title: openstack中的oslo_config
category: 技术 
published: true
---

oslo_config 主要处理用来读取配置项，配置项可以从命令行或者配置文件中读取。但从命令行中获取的配置型需要由`register_cli_opts`来注册。

配置项都是由[Opt][1]类或者其子类定义的。 配置项有多种类型：

- strings
- integers
- floats
- booleans
- lists
- multi strings
- dictionary

# 读取配置文件中的配置项
如果是在配置文件中指定了配置项的值，oslo_config定义了两个命令行参数`--config-file`和`--config-dir`分别来指定配置文件或者配置目录。

编辑[conf_test.py][2]

```python
from oslo_config import cfg
import sys

# 定义配置项，并指定默认值
opts = [
        cfg.IntOpt('o1', default=111),
        cfg.StrOpt('o2', default='test opt'),
        ]

CONF = cfg.CONF

# 注册配置项
CONF.register_opts(opts)

print CONF.o1
print CONF.o2
```

执行配置文件：

```bash
# python conf_test.py
111
test opt
```

可以正确读出默认值。

现在将配置项在一个配置文件中指定相应的值，编辑配置文件[test.conf][2]

```bash
[DEFAULT]
o1 = 123
o2 = test2
```

编辑脚本[conf_file.py][2]

```python
from oslo_config import cfg
import sys

opts = [
        cfg.IntOpt('o1', default=111),
        cfg.StrOpt('o2', default='test opt'),
        ]

CONF = cfg.CONF
CONF.register_opts(opts)

# 从命令行参数出读取配置文件并解析
CONF(sys.argv[1:])

print CONF.o1
print CONF.o2
```

执行查看结果：

```bash
# python conf_file.py --config-file test.conf
123
test2
```

# 组
这次没有返回配置项的默认值，而是读取到了配置文件中配置项的指定值。配置项可以进行分组，分组后先注册配置组，然后注册配置项，注册配置项的时候讲配置项添加到相应的组中。如果没有指定组的话，配置项默认都是`DEFAULT`组中的。

编辑[conf_group.py][2]

```python
from oslo_config import cfg
import sys

# 定义配置项o1, o2, o3, o4
opts = [
        cfg.IntOpt('o1', default=111),
        cfg.StrOpt('o2', default='test opt'),
        ]

opt_o3 = cfg.IntOpt('o3', default=333)
opt_o4 = cfg.StrOpt('o4', default='test opt4')

opt_group = cfg.OptGroup(name='Group_test', title='Group 1')

CONF = cfg.CONF
# 注册配置项o1, o2
CONF.register_opts(opts)

# 注册配置项o3, o4并添加到组Group_test中
CONF.register_opt(opt_o3, group = opt_group)
CONF.register_opt(opt_o4, group = 'Group_test')

CONF(sys.argv[1:])

# 输出结果
print "[DEFAULT]: o1, o2"
print CONF.o1
print CONF.o2
print ""

print "[Group_test]: o3, o4"
print CONF.Group_test.o3
print CONF.Group_test.o4
print ""
```

添加`Group_test`组到配置文件[test.conf][2]

```bash
[DEFAULT]
o1 = 123
o2 = test2

[Group_test]
o3 = 98
o4 = laksdfa
```

执行：

```bash
# python conf_group.py --config-file test.conf
[DEFAULT]: o1, o2
123
test2

[Group_test]: o3, o4
98
laksdfa
```

成功读取配置文件中`Group_test`组中的配置项的值。不同组中可以有相同的配置项，比如在`Group_test`中可以添加o1, o2配置项，

编辑[test.conf][2]

```bash
[DEFAULT]
o1 = 123
o2 = test2

[Group_test]
o1 = 234
o2 = test4
o3 = 98
o4 = laksdfa
```

在[conf_group.py][2]中添加：

```python
...
# 将o1, o2添加到Group_test组中
CONF.register_opts(opts, group = opt_group)
...
# 输出Group_test组中o1, o2的值
print "[Group_test]: o1, o2"
print CONF.Group_test.o1
print CONF.Group_test.o2
print ""
```

结果：

```bash
# python conf_group.py --config-file  test.conf
[DEFAULT]: o1, o2
123
test2

[Group_test]: o3, o4
98
laksdfa

[Group_test]: o1, o2
234
test4
```

可以看出，不同组的相同配置项可以指定不同的值，这样使配置更加灵活。

# 配置目录
除了可以在命令行中指定配置文件，也可以指定一个包含配置文件的目录，比如

```bash
# ls ./conf/
test1.conf  test2.conf
# cat ./conf/test1.conf
[DEFAULT]
o1 = 123
o2 = test2
# cat ./conf/test2.conf
[Group_test]
o1 = 234
o2 = test4
o3 = 98
o4 = laksdfa
```

同样用配置组例子中的脚本[conf_group.py][2]来执行，不过这次指定配置目录，查看结果：

```bash
# python conf_group.py --config-dir ./conf
[DEFAULT]: o1, o2
123
test2

[Group_test]: o3, o4
98
laksdfa

[Group_test]: o1, o2
234
test4
```

得到的结果和指定配置文件是一样的。

# 命令行中指定配置项
如果需要在命令行中指定配置项，相应的配置项需要由`register_cli_opts`进行注册。

编辑[conf_cli.py][2]

```python
from oslo_config import cfg
import sys

opts = [
        cfg.IntOpt('o1', default=111),
        cfg.StrOpt('o2', default='test opt'),
        ]

CONF = cfg.CONF
# 注册配置项，由命令行指定配置项的值
CONF.register_cli_opts(opts)
# 解析配置项
CONF(sys.argv[1:])

print CONF.o1
print CONF.o2
```

执行：

```bash
# python conf_cli.py --o1 98
98
test opt
```

在命令行中指定`o1`的值为98，可以看到o1的值被读取，没有指定值的o2读取的默认值。

例子代码：[test_example][2]

[1]: http://docs.openstack.org/developer/oslo.config/opts.html#oslo_config.cfg.Opt
[2]: https://github.com/cheneydc/test_example/tree/master/oslo_config
