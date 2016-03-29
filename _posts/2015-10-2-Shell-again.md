---
layout: post
title: Shell--再来一遍-1
category: 技术 
published: true
---

Linux用了好几年了，没有很熟练，也没有深入内核，平时工作发现shell还是很实用，现在重新的系统来一遍，吼吼……：）

主要看看[ABS](http://manual.51yip.com/shell/)然后多做练习是必须的

 -----------------------------
# 1. 从\#!开始
Shell脚本第一行都是#！开头的，用来指定命令解释器，当然，如果用的不是shell，python或者perl也都可以通过#！来指定，如果指定的路径错误，执行脚本的时候会报错，例如：
```
#! /bin/cat/bash

echo "hello"
```

保存脚本并执行
```
# chmod +x script.sh
# ./script.sh
bash: ./script.sh: /bin/cat/bash: bad interpreter: Not a directory
```


这里\#！也可以有其他好玩的地方

直接删除自身的脚本：
```
#! /bin/rm
```

显示自身的脚本：
```
#! /bin/cat
```

除了第一行的\#！外，脚本中其他以#开头的都是注释啦     


## 井号的其他用法 

- 表示传入脚本参数个数：

```
# cat script.sh
#! /bin/bash

echo $#
# sh script.sh 1 2 3
3
```


- ${var#Pattern}删除最小匹配：

```
# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
# echo ${PATH#*:}
/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

这里匹配的模板是*:，最小匹配的是/usr/local/sbin:，所以使用#的时候将其删除    

<br />

- ${var#Pattern}删除最长匹配：

```
# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
# echo ${PATH##*:}
/usr/local/games
```

这里使用##表示对模板*：的最长匹配，因此匹配到的应该是/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:    

<br />

- 数字常量的表示（不同的进制）

```
#  echo $(( 2#101011 ))
43
#  echo $(( 8#101011 ))
33289
#  echo $(( 10#101011 ))
101011
```

这里的三个#分别指定了2进制，8进制，10进制


# 2. 特殊字符

## 井号
上面已经说过了

## 分号
同一行中如果想写多个命令用；进行分隔

## 双分号
case的语句分支结束符
```
case "$variable" in
abc)  echo "\$variable = abc" ;;
xyz)  echo "\$variable = xyz" ;;
esac
```

## 英文句号
等同于source
```
.  ~/.bashrc
```

## 双引号
部分引用
```
echo "PATH is : $PATH"
PATH is : /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

## 单引号
完全引用
```
# echo 'PATH is : $PATH'
# PATH is : $PATH
```

## 冒号
空命令
什么也不做，返回值是0
所以死循环可以这么写
```
while :
do
    operation-1
    operation-2
    ...
    operation-n
done
```

## 反引号
命令替换 \`command\`
```
# `echo "ls"`
bash  gh-pages  hadoop_init.tar.gz  ssh-key  ubuntu.sh
```
