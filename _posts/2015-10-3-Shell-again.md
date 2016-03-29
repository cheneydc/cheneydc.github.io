---
layout: post
title: Shell--再来一遍-2
category: 技术 
published: true
---

# 3.变量和参数


##变量的赋值

$var、var，区别好这两个，就基本能正常使用变量了，var只是变量的名字，好比一个人的名字，狗蛋啊、狗剩啊……$var才是这个变量的值，实际上$var只是**${var}**的简化方式，有的地方写成$var会不好理解，就要写成${var}的形式了。
{% highlight bash %}
# a="hello"
# echo $a
#------------------------
变量的复制用=，左右两边不能有空格
声明变量的时候不需要$，使用的时候需要$
#------------------------
{% endhighlight %}


##变量是没有类型的

{% highlight bash %}
a=123
a=${a/3/b}
#------------------------
将a中的数字直接替换成字符，这是a便是一个字符串了
实际上变量没有类型区别，再次将字符b替换成数字，
a又变成了数字
#------------------------
a=${a/b/4}
{% endhighlight %}

这样增加了脚本的灵活性，但是很容易出现错误，写脚本时要做好一定的
变量检查


##特殊的变量

###环境变量

大多数情况下，一个进程会有一个“进程表”，它是一组进程使用的环境变量。一个Shell启动时，会创建一个相应的“进程表”，当更新一个变量时，这个表会立即更新，shell的子进程可以继承它的环境变量。

如果一个脚本要设置一个环境变量，需要将它导出，通知给执行这个脚本的进程，使用export进行导出
```
export VAR_TEST=1
```
####set、export和env区别
检查环境变量常用到的几个命令有set，export和env，举个例子看一下
{% highlight bash %}
#------------------------
set --- 当前shell的变量，包括当前用户变量
env --- 显示当前用户变量
export --- 显示当前导出成用户变量的shell变量
#------------------------

# var_test=1
# env | grep var_test
# export | grep var_test
# set | grep var_test
var_test=1
# export var_test
# set | grep var_test
_=var_test
var_test=1
# env | grep var_test
var_test=1
# export | grep var_test
declare -x var_test="1"
{% endhighlight %}

###位置参数
$0是脚本的名字，$1是第一个参数，$2是第二个参数，$3是第三个，以此类推。在位置参数$9之后的参数必须用括号括起来，例如：${10}, ${11}, ${12}
{% highlight bash %}
#------------------------
$* $@都表示命令行中所有的参数（不包括$0，区别在于使用被“”修饰的时候，$*会将所有参数作为一个整体，用下面的脚本测试一下：
#------------------------
#!/bin/bash

# cat test.sh
echo "---------With \"\"--------"
echo "\"\$*\":"
for var in "$*"
do
    echo $var
done
echo 

echo "\"\$@\":"
for var in "$@"
do
    echo $var
done
echo 


echo "---------Without \"\"--------"
echo "\$*:"
for var in $*
do
    echo $var
done
echo 

echo "\$@:"
for var in $@
do
    echo $var
done
echo 
{% endhighlight %}

执行脚本：

{% highlight bash %}
# chmod +x test.sh
# ./test.sh a b c d
---------With ""--------
"$*":
a b c d

"$@":
a
b
c
d

---------Without ""--------
$*:
a
b
c
d

$@:
a
b
c
d

{% endhighlight %}

####shift命令

shift命令会使命令行参数移位，删除$1位置的参数，将后面的参数均向前移位

测试脚本：

{% highlight bash %}
#!/bin/bash

until [ -z "$1" ]
do
    echo "$1"
    shift
done

echo 
exit 0
{% endhighlight %}

执行脚本：

{% highlight bash %}
# ./b.sh a b c d
a
b
c
d
{% endhighlight %}
