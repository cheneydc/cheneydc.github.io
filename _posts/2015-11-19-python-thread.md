---
layout: post
title: python - 线程
category: 技术
published: false
---

# 进程&线程
这个概念网上太多了，具体的不说了，我对二者的理解是操作系统执行的粒度不同，进程拥有完成的堆栈内存空间，不同集成的内存空间是独立的，交互的话就要采用进程间通信了（IPC）；线程有独立的栈，只是进程中的一个执行路径，所以进程中的多个线程是共享堆空间的。

因此多线程的并发性通常来讲比较好，但是由于多线程共享的内存空间，所以多线程编程时要做好同步工作。

## GIL干嘛的？
当然python中有黑老大，GIL，这是个全局解释器锁，这个黑老大很霸道的，它保证解释器执行脚本的时候，每个时刻只会有一个线程在运行。开始我很奇怪，既然它已经保证了线程的独立运行，那么为什么还需要同步的操作呢？[这个里面][1]回答的很好，因为GIL操作的粒度是在bytecode层面，而我们在python脚本中对某一个变量的修改一般会由多个指令组成，如果在这中间让步给另一个线程，而它也来操作这个变量的话，就会出现问题了。

# python多线程
python中主要通过[thread](2)(python3中将这个模块重命名为_thread), `threading`两个模块来实现多线程。thread更底层一点，threading是类包装的，非常方便好用。

## thread

[1]: https://stackoverflow.com/questions/26873512/why-does-python-provide-locking-mechanisms-if-its-subject-to-a-gil/26873766#26873766?newreg=09f0750ce3104c6dbe2f39ad4ae4e451
[2]: https://docs.python.org/2/library/thread.html
