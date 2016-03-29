---
layout: post
title: docker学习（2）
category: 技术 
---

###5. 运行第一个容器

第一步，查看docker程序是否存在，功能是否正常  

{% highlight bash %}  $ sudo docker info iContainers: 0 Images: 2 Storage Driver: aufs  Root Dir: /var/lib/docker/aufs  Backing Filesystem: extfs  Dirs: 2  Dirperm1 Supported: true Execution Driver: native-0.2 Kernel Version: 3.16.0-37-generic Operating System: Ubuntu Kylin 14.10 CPUs: 4 Total Memory: 5.63 GiB Name: codong ID: WTRJ:5GFG:QWBJ:VKF2:KB4S:UQAE:UGPS:YDIR:L6DI:DRAE:BTRU:AMVG WARNING: No swap limit support {% endhighlight %}  

这里调用了docker可执行程序的info命令，该命令会返回所有容器和镜像的数量、docker使用的执行驱动和存储驱动以及docker的基本配置。  

Docker是基于C/S架构的。它有一个docker程序，既能作为客户端也能作为服务端。作为客户端，docker程序向Docker守护进程发送请求，然后再对返回的结果进行处理。

现在，开始启动第一个docker容器。使用docker run命令创建容器，docker run命令提供了Docker容器的创建到启动的功能。  

{% highlight bash %}  $ sudo docker run -i -t ubuntu /bin/bash Unable to find image 'ubuntu:latest' locally latest: Pulling from ubuntu

e9e06b06e14c: Pull complete  a82efea989f9: Pull complete  37bea4ee0c81: Pull complete  07f8e8c5e660: Already exists  ubuntu:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.

Digest: sha256:8126991394342c2775a9ba4a843869112da8156037451fc424454db43c25d8b0 Status: Downloaded newer image for ubuntu:latest root@b924caf80ca5:/# ls {% endhighlight %}  

这里指定了-i和-t两个参数，-i标志保证容器中STDIN是开启的，尽管我们没有附着到容器中。持久的标准输入是交互式shell的"半边天"，-t则是另外一半，它告诉Docker为要创建的容器分配一个伪tty终端。这样，新创建的容器才能提供一个交互式shell。若要在命令行下创建一个我们能与之进行交互的容器，而不是一个运行后台的服务的容器，则这两个参数已经是最基本的参数了。  

> [官方文档](http://docs.docker.com/reference/commandline/cli/#run)列出了docker run命令的所有标志，还可以用docker help run查看这些标志。或者可以勇Docker的man page

示例中使用的是ubuntu镜像。这是一个"基础"（base）镜像，它由docker公司提供，保存在[Docker Hub](http://hub.docker.com) Registry上。  

Dokcer首先会检查本地是否存在ubuntu镜像，如果本地没有的话，就连接Docker Hub Registry，查看Docker Hub是否有该镜像。一旦找到，就会下载镜像并保存到本地宿主机中。  

随后，Docker在文件系统内部使用这个镜像创建了一个新容器。该容器拥有自己的网络、IP地址，以及一个用来和宿主机进行通信的桥接网络接口。最后，我们告诉Docker在新容器中执行什么命令，上面的例子，我们运行了一个Bash shell。  

###6. 使用容器 Docker 启动后可以像正常宿主机一样操作。

{% highlight bash %} root@b924caf80ca5:/# hostname  b924caf80ca5 root@b924caf80ca5:/# cat /etc/hosts  172.17.0.1  b924caf80ca5 127.0.0.1   localhost ::1 localhost ip6-localhost ip6-loopback fe00::0 ip6-localnet ff00::0 ip6-mcastprefix ff02::1 ip6-allnodes ff02::2 ip6-allrouters root@b924caf80ca5:/# ip a 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00     inet 127.0.0.1/8 scope host lo        valid_lft forever preferred_lft forever     inet6 ::1/128 scope host         valid_lft forever preferred_lft forever 9: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default      link/ether 02:42:ac:11:00:01 brd ff:ff:ff:ff:ff:ff     inet 172.17.0.1/16 scope global eth0        valid_lft forever preferred_lft forever     inet6 fe80::42:acff:fe11:1/64 scope link         valid_lft forever preferred_lft forever root@b924caf80ca5:/# apt-get install vim Reading package lists... Done Building dependency tree<br>Reading state information... Done The following extra packages will be installed:   libgpm2 libpython2.7 libpython2.7-minimal libpython2.7-stdlib vim-runtime Suggested packages:   gpm ctags vim-doc vim-scripts The following NEW packages will be installed:   libgpm2 libpython2.7 libpython2.7-minimal libpython2.7-stdlib vim   vim-runtime 0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded. Need to get 9083 kB of archives. After this operation, 42.9 MB of additional disk space will be used. Do you want to continue? [Y/n] y Get:1 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main libgpm2 amd64 1.20.4-6.1 [16.5 kB] Get:2 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main libpython2.7-minimal amd64 2.7.6-8 [307 kB] Get:3 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main libpython2.7-stdlib amd64 2.7.6-8 [1872 kB] Get:4 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main libpython2.7 amd64 2.7.6-8 [1044 kB] Get:5 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main vim-runtime all 2:7.4.052-1ubuntu3 [4888 kB] Get:6 [http://archive.ubuntu.com/ubuntu/](http://archive.ubuntu.com/ubuntu/) trusty/main vim amd64 2:7.4.052-1ubuntu3 [956 kB] Fetched 9083 kB in 12min 2s (12.6 kB/s)<br>Selecting previously unselected package libgpm2:amd64. (Reading database ... 11528 files and directories currently installed.) Preparing to unpack .../libgpm2_1.20.4-6.1_amd64.deb ... Unpacking libgpm2:amd64 (1.20.4-6.1) ... Selecting previously unselected package libpython2.7-minimal:amd64. Preparing to unpack .../libpython2.7-minimal_2.7.6-8_amd64.deb ... Unpacking libpython2.7-minimal:amd64 (2.7.6-8) ... Selecting previously unselected package libpython2.7-stdlib:amd64. Preparing to unpack .../libpython2.7-stdlib_2.7.6-8_amd64.deb ... Unpacking libpython2.7-stdlib:amd64 (2.7.6-8) ... Selecting previously unselected package libpython2.7:amd64. Preparing to unpack .../libpython2.7_2.7.6-8_amd64.deb ... Unpacking libpython2.7:amd64 (2.7.6-8) ... Selecting previously unselected package vim-runtime. Preparing to unpack .../vim-runtime_2%3a7.4.052-1ubuntu3_all.deb ... Adding 'diversion of /usr/share/vim/vim74/doc/help.txt to /usr/share/vim/vim74/doc/help.txt.vim-tiny by vim-runtime' Adding 'diversion of /usr/share/vim/vim74/doc/tags to /usr/share/vim/vim74/doc/tags.vim-tiny by vim-runtime' Unpacking vim-runtime (2:7.4.052-1ubuntu3) ... Selecting previously unselected package vim. Preparing to unpack .../vim_2%3a7.4.052-1ubuntu3_amd64.deb ... Unpacking vim (2:7.4.052-1ubuntu3) ... Setting up libgpm2:amd64 (1.20.4-6.1) ... Setting up libpython2.7-minimal:amd64 (2.7.6-8) ... Setting up libpython2.7-stdlib:amd64 (2.7.6-8) ... Setting up libpython2.7:amd64 (2.7.6-8) ... Setting up vim-runtime (2:7.4.052-1ubuntu3) ... Processing /usr/share/vim/addons/doc Setting up vim (2:7.4.052-1ubuntu3) ... update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vim (vim) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vimdiff (vimdiff) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rvim (rvim) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rview (rview) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vi (vi) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/view (view) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/ex (ex) in auto mode update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/editor (editor) in auto mode Processing triggers for libc-bin (2.19-0ubuntu6.6) ... root@b924caf80ca5:/# vim root@b924caf80ca5:/# ps -aux USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND root         1  0.0  0.0  18180  3436 ?        Ss   14:09   0:00 /bin/bash root       171  0.0  0.0  15572  2180 ?        R+   14:26   0:00 ps -aux {% endhighlight %}  

结束工作后，exit就可以退出，回到宿主机命令行提示符了。

[未完...]
