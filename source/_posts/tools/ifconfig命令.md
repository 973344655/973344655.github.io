---
title: 网卡相关操作ifconfig
date: 2019-10-29 15:18:11
tags: [linux]
---

我一直将 ifconfig 简单的用来查看ip信息。有点太不了解它了。<br>
实际上 ifconfig 工具不仅可以被用来简单地获取网络接口配置信息，还可以修改这些配置。

```
ifconfig - configure a network interface
```

# 一.ifconfig了解

## 1.使用

```
ifconfig [-v] [-a] [-s] [interface]
ifconfig [-v] interface [aftype] options | address ...

```
-v 详细输出

-s 输出一个简要list,和netstat -i 一个效果

-a 输出全部接口信息，即使已经down

address ip地址，可以用来给网卡配置ip地址，如 ifconfig wlan0 192.168.1.65  将wlan0配置为192.168.1.65

总结下就是： <strong>ifconfig  [interface] [options]</strong>

## 2.interface

interface是网络接口设备名，例:wlan0, eth0

## 3.options

- up

开启接口设备

- down

shutdown 接口设备，这里要特别注意，在服务器上操作时，如果关闭了网卡，就断网了，导致ssh连接断开，就不能打开了，所以要小心(除非有多个网卡，关闭一个，其它的还联网)

- arp

设置是否支持arp协议

- promisc

```
Enable  or  disable  the  promiscuous mode of the interface.  If selected, all packets on the
network  will be received by the interface.

```
promiscuous模式，网卡将接收网络中发给它的  所有的数据包。

- allmulti

```
Enable or disable all-multicast mode.  If selected, all multicast packets on the network will be
re‐ceived by the interface
```
all-multicast模式，网卡将接收网络中的所有组播包。

- mtu N

设置设置网卡的最大传输单元N,字节数。

- address

设置绑定的ip地址

- dstaddr addr

```
Set the remote IP address for a point-to-point link (such as PPP).  This keyword is now obsolete;
use the pointopoint keyword instead.
```
建立点对点通讯，暂时还不太清楚用来干嘛

- add/del addr

添加/删除 ipv6 地址

- tunnel

建立IPv4与IPv6之间的隧道通信地址。

- txqueuelen length

```
Set the length of the transmit queue of the device. It is useful to set this to small values for slower devices with a high latency (modem links, ISDN) to prevent fast bulk transfers from disturbing interactive traffic like telnet too much
```
设置传输队列长度。

# 二.例子

## 1.属性


ifconfig:
```
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 3655  bytes 1594522 (1.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3655  bytes 1594522 (1.5 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vmnet1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.84.1  netmask 255.255.255.0  broadcast 192.168.84.255
        inet6 fe80::250:56ff:fec0:1  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:c0:00:01  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vmnet8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.94.1  netmask 255.255.255.0  broadcast 192.168.94.255
        inet6 fe80::250:56ff:fec0:8  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:c0:00:08  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 18  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.64  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::8702:42e3:d919:f0db  prefixlen 64  scopeid 0x20<link>
        ether 58:a0:23:27:d5:df  txqueuelen 1000  (Ethernet)
        RX packets 1944  bytes 1436200 (1.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1316  bytes 349665 (341.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

- flags=4163

The if.h header says up=0x1, running=0x40, broadcast=0x2, multicast=0x1000

4163 = 0x1043 = 0x1000 + 0x40 + 0x2 + 0x1

- <UP,BROADCAST,RUNNING,MULTICAST>  

up 网卡开启<br>
BROADCAST 支持广播<br>
RUNNING 网卡正在运行<br>
MULTICAST 支持多播

- mtu  

传输单元字节数

- txqueuelen

传输队列大小

- broadcast

组播地址

- RX packets

接收的数据包数

- TX packets

传输即发送的数据包数。


## 2.使用例子

```
#ARP协议
# ifconfig wlan0 arp  //开启
# ifconfig wlan0 -arp  //关闭
```

# 三.补充

iwconfig: 用来配置无线接口信息

iw: 用来连接wifi
