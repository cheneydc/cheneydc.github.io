---
layout: post
title: python - greenlet
category: 技术 
published: true
---

greenlet，也就是协程，是[Stackless](http://www.stackless.com/)的副产物，是一个更加原始的微线程的概念，但是没有调度（摘自[http://www.oschina.net/p/greenlet/）](http://www.oschina.net/p/greenlet/)）。所以使用协程的话就需要自己处理调度问题。这样可以主动的在阻塞的时候可以让步给其他协程工作，提高CPU的利用率。

Python中主要通过greenlet模块来实现协程操作。greenlet.greenlet是greenlet type，一些基本操作：

- greenlet(run=None, parent=None)

  创建一个greenlet对象（不执行），run指定执行回调，parent指定parent greenlet(就叫PG吧)，默认PG是当前greenlet

- greenlet.getcurrent()

  返回当前greenlet

- greenlet.GreenletExit

  这个异常不会发送给PG，可以用来杀死一个greenlet

--------------------------------------------------------------------------------

下面看一个例子[greenlet_test.py][1]：

```python
from greenlet import greenlet

def test1():
    print "test1 - start"
    gr2.switch()
    print "test1 - end"

def test2():
    print "   test2 - start"
    gr1.switch()
    print "   test2 - end"

gr1 = greenlet(test1)
gr2 = greenlet(test2)
print gr1.parent
print gr2.parent
print greenlet.getcurrent()
gr1.switch()
```

看下执行结果：

```bash
#python greenlet_test.py
<greenlet.greenlet object at 0x7f622ec91eb0>
<greenlet.greenlet object at 0x7f622ec91eb0>
<greenlet.greenlet object at 0x7f622ec91eb0>
test1 - start
   test2 - start
test1 - end
```

从结果看出，gr1和gr2有着相同的PG，也就是当前greenlet。

- 首先`gr1.switch()`从当前greenlet切换到gr1执行，也就是`test1()`，所以首先输出`test1 - start`
- test1中通过`gr2.switch()`切换到gr2，执行test2，输出`test2 - start`
- 同上再次切换到gr1，执行test1，输出`tset1 - end`
- 然后就结束了，为什么没有输出`test2 - end`呢？

协程之间的切换不是函数之间的调用，毕竟操作的是greenlet，比如从gr2切换到gr1，实际上是将gr1入栈直接执行（点击[参考1][3],[参考2][4],[参考3][5]），函数执行结束，gr1这个协程就结束了，它返回给的不是其他任何函数，而是返回给它的PG，而PG后面也没有任务了，整个程序退出，所以gr2后面没有机会再执行了，`test2 - end`就不会输出了。

[1]: https://github.com/cheneydc/test_example/tree/master/greenlet
[2]: https://github.com/python-greenlet/greenlet/blob/master/greenlet.c
[3]: http://segmentfault.com/a/1190000000626309
[4]: https://code.google.com/p/libhjw/wiki/notes_on_greenlet#greenlet_not_stated
[5]: http://www.cnblogs.com/qiyukun/p/4754077.html
