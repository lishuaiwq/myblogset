---
title: 多路I/O转接-select
date: 2018-03-10 22:32:04
tags: 多路I/O转接
categories: 
- Linux
- 网络编程
---
为什么会出现多路I/O转接服务器呢？
<!--more-->
因为虽然多线程和多进程服务器可以实现多个客户端同时连接服务器进行数据交流，但是这种实现方式都是进程来监听客户端是否有动静，比如accept,read，如果进程或者线程执行到这些函数的时候没有相关的动作发生则就会发生阻塞等待，函数就不能返回,这样大大的影响了程序的效率。为了解决这个问题，我们提出了一种方法，就是把监听的任务交给内核来帮我们做，当内核发现有相关动作发生以后则给当前进程一个反馈，这个时候进程在进行相关函数的调用去处理这个动作，这样的就不会产生阻塞了。
实现这个要求的就是select函数，下面我来详细介。

## select() ##
   select()，本函数用于确定一个或多个套接口的状态，对每一个套接口，调用者可查询它的可读性、可写性及错误状态信息，用fd_set结构来表示一组等待检查的套接口，在调用返回时，这个结构存有满足一定条件的套接口组的子集，并且select()返回满足条件的套接口的数目。有一组宏可用于对fd_set的操作，这些宏与Berkeley Unix软件中的兼容，但内部的表达是完全不同的。

函数原型和相关参数的解释：

	int select(int nfds, fd_set *readfds, fd_set *writefds,fd_set *exceptfds, struct timeval *timeout);


	nfds:监控的文件描述符集里最大文件描述符+1，因为此参数会告诉内核检测前多少个文件描述符的状态
	readfds：	监控有读数据到达文件描述符集合，传入传出参数
	writefds：	监控写数据到达文件描述符集合，传入传出参数
	exceptfds：	监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数
	timeout：	定时阻塞监控时间，3种情况
				1.NULL，永远等下去
				2.设置timeval，等待固定时间
				3.设置timeval里时间均为0，检查描述字后立即返回，轮询
	struct timeval {
		long tv_sec; /* seconds */
		long tv_usec; /* microseconds */
	};
   
	void FD_CLR(int fd, fd_set *set); 	//把文件描述符集合里fd清0
	int FD_ISSET(int fd, fd_set *set); 	//测试文件描述符集合里fd是否置1
	void FD_SET(int fd, fd_set *set); 	//把文件描述符集合里fd位置1
	void FD_ZERO(fd_set *set); 			//把文件描述符集合里所有位清0

select有三个可能的返值：

1）返回值－1表示出错。例在未有描述符准备好数据时捕捉到一个信号时

2）返回值0表示没有描述符准备好。若指定的描述符都没有准备好，并且指定的时间已到，则发生这种情况。

3）返回一个正数，说明已经准备好的描述符数，在这种情况下。三个描述符集中仍旧打开的位是已准备好的描述符位。

另外：
1.select能监听的文件描述符个数受限于FD_SETSIZE,一般为1024，单纯改变进程打开的文件描述符个数并不能改变select监听文件个数

2.解决1024以下客户端时使用select是很合适的，但如果链接客户端过多，select采用的是轮询模型，会大大降低服务器响应效率，不应在select上投入更多精力。

下面给出select的编程大概思路

1.首先监听int sfd=socket（）

2.然后判断sfd是否有行动，如果有行动则调用int cfd=accept()函数

3.将cfd加入allfd

4.然后判断是否有写入动作，有的话进行写入。

代码已经上传至github
[https://github.com/lishuaiwq/linux-](https://github.com/lishuaiwq/linux- "https://github.com/lishuaiwq/linux-")