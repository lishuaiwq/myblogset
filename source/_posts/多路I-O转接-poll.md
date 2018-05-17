---
title: 多路I/O转接-poll
date: 2018-03-13 20:35:19
tags: 多路I/O转接
categories: 
- Linux
- 网络编程
---
首先多路I/O是在select的基础上进行升级改版的。既然是升级版的那肯定就比select好用。下面我来说说poll相对于select的优点
<!--more-->

1.突破了select受FD_SETSIZE限制只能最多只能监控1024个文件描述符的缺点。我们可以通过修改当前配置文件来改变这个文件数，但是不能超过硬件允许的最大的数量。
（我们可以通过cat /proc/sys/fs/file-max这个命令来查看当前硬件允许最大的打开文件的的数量）
![](https://i.imgur.com/HeHtmN4.jpg)

可以看到我这里允许最大的数是101015，在用ulimit -a查看一下当前默认的打开数值上限
![](https://i.imgur.com/6wgy48n.jpg)

可以看到上限是1024

现在我们将其修改到4096，vim /etc/security/limits.conf，在文档处添加如下内容
![](https://i.imgur.com/yNSzzve.jpg)
然后注销用户或者重启就生效了。

2.select的监听事件和返回事件是一个集合，但是在poll中，它将这两个事件分开了。

3.poll的搜范围变小了，仅限制于我们自己所定义的数组的范围。

但是poll还是有不足的地方，比如我监听了500个文件描述，但是只有3个文件描述符满足动作条件，但是poll还是会遍历整个500个文件描述符，效率有些低，所以我们的理想状态是直接将三个满足条件的文件描述符找出来，这个在epoll中可以做到，我们后面再介绍。

下面来看poll函数：

![](https://i.imgur.com/DHQ9TJn.jpg)
当然返回值和select是一样的。