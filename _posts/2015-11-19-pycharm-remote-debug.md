---
layout: post
title: pycharm 远程调试
category: 工具
published: true
---

pycharm不得不说是用的最顺手的一个IDE了，最近尝试了下pycharm的远程调试功能。

# pycharm配置如下
 ![](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/post/20151119-pycharm-debug-1.png)

 ![](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/post/20151119-pycharm-debug-2.png)

 ![](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/post/20151119-pycharm-debug-3.png)  local host name是本机ip，port选择一个空闲端口即可，实际上本机是作为server端。

# 启用调试
点击调试按钮（快捷键：Shift+F9）开启调试，IDE会弹出Debug窗口 ![](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/post/20151119-pycharm-debug-4.png)

如上图中有一段代码，里面包换了刚才设置的本机ip和port。将这段代码加入到执行机器上的代码里即可。

```python
from oslo_config import cfg
import sys

opts = [
        cfg.IntOpt('o1', default=111),
        cfg.StrOpt('o2', default='test opt'),
        ]

import pydevd
pydevd.settrace('192.168.20.198', port=51234, stdoutToServer=True, stderrToServer=True)

CONF = cfg.CONF
CONF.register_cli_opts(opts)
CONF(sys.argv[1:])

print CONF.o1
print CONF.o2
```

执行代码后，加入调试代码的地方就是断点了，执行到此就会连接到pycharm，然后就可以进行调试了。

# 调试
远程的脚本开始执行并运行到断点位置后，IDE的Console中提示

```
Connected to pydev debugger (build 139.1001)
```

此时点击Debugger选项卡，进入调试界面 ![](https://raw.githubusercontent.com/cheneydc/blog/gh-pages/assets/img/post/20151119-pycharm-debug-5.png)

`1`里面就是常用的流程控制了step over, step into等等，`2`里面就是程序的变量列表了。这样调试python就方便多了。
