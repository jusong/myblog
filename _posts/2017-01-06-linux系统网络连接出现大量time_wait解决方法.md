---
layout: post
title:  "linux系统网络连接出现大量time_wait解决方法"
date:   2017-01-06 02:28:47 +0800
categories: linux tcp time_wait
---

##time_wait介绍
  time_wait是tcp链接过程中的一种状态，time_wait的出现是由于服务器主动断开与客户端的链接，即在tcp四次分手中向客户端发送最后一个ack包之后就进入time_wait状态，等待时间为2MSL（max segment lifetime），MSL为数据包在网络中的最大生命周期，超过MSL后当前tcp链接在网络上的数据包就全部失效了，这样主动发起断开链接的服务器方进入time_wait状态等待2MSL时间是为了保证以后新建的链接不会受到旧链接的延迟数据包的影响。

##查看当前系统各个状态的链接数量
```
$ netstat -n | awk '/^tcp/ {++S[$NF]}; END {for(a in S) print a,"\t",S[a]}'
TIME_WAIT 	 3597
CLOSE_WAIT 	 1
FIN_WAIT2 	 4
ESTABLISHED 	 151
LAST_ACK 	 1
```
可以看出当前系统time_wait数量比较多

##解决办法
通过调整系统内核tcp相关参数对tcp链接进行有效回收
修改/etc/sysctl.conf文件
```
net.ipv4.tcp_syncookies = 1  //开启SYN cookies，防止syn队列溢出
net.ipv4.tcp_tw_reuse = 1    //开启重用time_wait socket
net.ipv4.tcp_tw_recycle = 1  //开启快速回收time_wait socket
net.ipv4.tcp_fin_timeout = 30  //保持在FIN-WAIT-2状态的时间
net.ipv4.tcp_keepalive_time = 1200
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 1024 65000
##表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。
net.ipv4.tcp_max_syn_backlog = 8192
##表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000
##表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印
警告信息。
```
然后执行命令使修改生效即可
```/sbin/sysctl -p```
