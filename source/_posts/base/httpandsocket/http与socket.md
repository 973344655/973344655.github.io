---
title: Http与Socket关系
date: 2018-12-24 10:11:07
tags: [protocol]
---
#### 1.问题
今天发现网关程序中，java监听了8083端口(Http)，同时，又在程序中使用了8083端口作为服务端，接收来自短信网关给我们发送的回执（Sgip1.2)。<br>
不知道两者之间是否有冲突.了解了http与socket区别后，就知道了。

#### 2.OSI七层模型
![OSI](./osi_七层模型.jpg)
图片来自:https://www.jianshu.com/p/a18a5ba78fad

#### 3.Http与Socket所在层

Http：Http是应用层的一种协议，用来对Http协议的数据进行读取。<br>

Tcp/Udp: 传输层。<br>

Ip: 网络层.<br>

Socket: Socket是对 TCP/IP 协议的封装，Socket 只是个接口不是协议，通过 Socket 我们才能使用 TCP/IP 协议，除了 TCP，也可以使用 UDP 协议来传递数据。<br>

Port: 端口是Tcp/Udp建立连接所用，也是在传输层，表现为Socket监听Port。<br>

#### 4.数据的处理流程
- 1.ClientA向服务端发一个数据，先根据所在层次，层层向下，按照所在层次的协议对数据进行包装。
- 2.数据到达服务端后，从底层(物理层)开始，将数据按照对应层协议，进行处理，然后传到上一层，直到得到可用数据。
- 3.http协议的数据，从客户端应用层，到达服务端应用层，一次传输完成。
- 4.Sgip的数据，在ClientA时，先将数据按照Sgip协议进行包装，在传输层通过Socket发送(依然往下包装到达物理层，Sgip数据包),服务端接收到时，层层解包装，在传输层，Socket接收到数据包后，此时包装层已经只剩下了Sgip包装，因此，在这里按照Sgip协议解包装后，就已经得到了想要的数据。不需要再往上层传递.
- 5.因此，在这里，虽然Http与Sgip都使用的统一端口，但是互不影响。

#### 5.总结
- 1.Http是应用层协议，Socket是对Tcp/Ip的封装
- 2.Http发送数据也会经过Socket(Tcp或者Udp建立的连接)
- 3.数据的传输和接收，是通过Socket,但是得到数据，是按照其包装和解包装的协议来的，Socket只是单纯的传输数据(bit)，并不关心数据是什么，有什么意义。
- 4.Socket传输时，发送字符串，接收时也能直接接收到字符串，是因为包装/解包装层级一样，得到了发送时的数据。

#### 6.各协议所对应层次 2019/09/23

层级 | 协议
---- | ----
应用层 |	DHCP · DNS · FTP · Gopher · HTTP · IMAP4 · IRC · NNTP · XMPP · POP3 · SIP · SMTP ·SNMP · SSH · TELNET · RPC · RTCP · RTP ·RTSP · SDP · SOAP · GTP · STUN · NTP · SSDP
表示层|	HTTP/HTML · FTP · Telnet · ASN.1（具有表示层功能）
会话层	|ADSP·ASP·H.245·ISO-SP·iSNS·NetBIOS·PAP·RPC·RTCP·SMPP·SCP·SSH·ZIP·SDP（具有会话层功能）
传输层|	TCP · UDP · TLS · DCCP · SCTP ·RSVP · PPTP
网络层|	IP (IPv4 · IPv6) · ICMP · ICMPv6 · IGMP ·IS-IS · IPsec · BGP · RIP · OSPF ·ARP · RARP
数据链路层|	Wi-Fi(IEEE 802.11) · WiMAX(IEEE 802.16) ·ATM · DTM · 令牌环 · 以太网路 ·FDDI · 帧中继 · GPRS · EVDO · HSPA · HDLC · PPP · L2TP · ISDN ·STP
物理层	|以太网路卡 · 调制解调器 · 电力线通信(PLC) · SONET/SDH（光同步数字传输网） ·G.709（光传输网络） · 光导纤维 · 同轴电缆 · 双绞线

注意: 应用层，表示层，会话层可以不用那个严格区分，都可以当作应用层协议来看待
