---
title: TCP keepalive
tags:
  - keepalive
  - linux
  - tcp/ip
id: '892'
categories:
  - - ops
comments: false
date: 2021-10-13 19:23:54
---

TCP是面向连接的。创建一个TCP连接，客户端和服务端需要经历三次握手，一旦建立它的期限就是**永久**，除非其中一方发起关闭。 理想状态下，客户端与服务端建立连接就可以一直用下去，但是在真实的互联网世界没有这么理想: NAT网关、防火墙、代理服务器等网络设备可能会清理空闲的连接; 也有可能其中一方突然崩溃、网络断开来不急通知对方关闭连接；或者其他意外，导致其中一方或双方的连接就一直永久保留在那里。 客户端，随着应用的关闭，系统会自动回收应用创建的连接。这个不处理似乎影响也不太大。 服务端，软件都是长期运行的，没办法通过软件的关闭来实现连接的回收。在Linux里一个空闲的TCP连接占用将近4k内存，大量的空闲连接积攒下来消耗不少内存。而且服务端一直不释放可能会导致TCP创建连接失败。所以在**应用层**基本都会有一个超过一定时间不使用主动关闭TCP连接的机制。

## TCP keepalive 可以做什么

这个时候我们就会发现TCP keepalive似乎是可有可无的，确实在TPC连接中keepalive是默认不开启的。不过我们可以通过 TCP keepalive做这些事情:

### 1\. 检测断掉的连接

keepalive在一定的间隔里通过连接发送一个信号，如果在指定的时间内收到回应就可以确认连接还是通的；如果指定的时候还没收到回应就启动重试；如果重试了最大重试次数内还是没收到回应，那么就判定连接不通了，就主动关闭连接。

### 2\. 防止因为没有网络活动而导致连接的断开

NAT网关、代理服务器、防火墙等等会清理他认为不活动的连接，keepalive的周期性发送信号可以保持连接处于持续活动状态。Linux NAT网关的连接保持时间默认是1天，阿里云NAT网关连接的老化时间是900秒也就是15分钟，这也产生过一个生产事故。

## 如何在Linux下使用 TCP keepalive

Linux 原生支持TCP keepalive,并提供了三个用户参数:

*   `tcp_keepalive_time` 默认值7200s(2小时)，就是连接空闲多久，开始发keepalive 探测包。
    
*   `tcp_keepalive_intvl` 默认值75s，后续 keepalive探测包的时间间隔。
    
*   `tcp_keepalive_probes` 默认值9，发多少keepalive探测包没有回应，就认为连接已经死了需要通知应用层。
    

### 简易流程是这样的:

1.  客户端创建启用keepalive的TCP连接
2.  如果连接在`tcp_keepalive_time`时间内一直静默，那么发一个空的ACK包，也就是keepalive探测包。
3.  服务端是否回应这个ACK？
    *   没有回应
        1.  等待`tcp_keepalive_intvl`秒，然后发另一个ACK包
        2.  重复上一步，直到发了`tcp_keepalive_probes`次ACK包。
        3.  到了这个点还是没收到回应，那么发送一个`RST`包，然后关闭连接。
    *   收到回应，那么回到第二步

### 如何修改参数

procfs是一个伪文件系统，通过它可以查看或修改内核参数。 查看

```Shell
# cat /proc/sys/net/ipv4/tcp_keepalive_time
7200
# cat /proc/sys/net/ipv4/tcp_keepalive_intvl
75
# cat /proc/sys/net/ipv4/tcp_keepalive_probes
9
```

修改

```Shell
# echo 600 > /proc/sys/net/ipv4/tcp_keepalive_time
# echo 60 > /proc/sys/net/ipv4/tcp_keepalive_intvl
# echo 20 > /proc/sys/net/ipv4/tcp_keepalive_probes
```

简单明了，但是不太适合大量的参数的管理，所有Linux还提供了一个辅助的工具`sysctl`。 把procfs路径里的`/proc/sys/`删掉剩下的 **/** 替换成 **.** 就可以了。 `sysctl variable`查看变量的值。 `sysctl -a`打印所有变量的值

```Shell
# sysctl net.ipv4.tcp_keepalive_time
net.ipv4.tcp_keepalive_time = 7200

# 也可能一次查多个变量名
# sysctl net.ipv4.tcp_keepalive_intvl net.ipv4.tcp_keepalive_probes
net.ipv4.tcp_keepalive_intvl = 75
net.ipv4.tcp_keepalive_probes = 9 

# 也可以打印出所有的变量，然后通过grep等工具过滤
# sysctl -a  grep net.ipv4.tcp_keepalive
net.ipv4.tcp_keepalive_intvl = 75
net.ipv4.tcp_keepalive_probes = 9
net.ipv4.tcp_keepalive_time = 7200
```

需要修改的时候就用`sysctl -w variable=value`的方式

```Shell
# sysctl -w net.ipv4.tcp_keepalive_time=600 \
    net.ipv4.tcp_keepalive_intvl=60 \
    net.ipv4.tcp_keepalive_probes=20
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 20
```

要注意的是这些修改系统重启之后都会失效的，如果希望重启之后依然生效。就需要把配置写入`/etc/sysctl.conf`或者在`/etc/sysctl.d/`目录内创建一个`.conf`文件并写入配置。这样Linux系统再启动时就会加载并应用这个配置。

### 应用内

keepalive默认是不开启的，需要主动指定.

#### Java

```Java
// BIO
Socket client = new Socket(hostName, port);
client.setKeepAlive(true);

// NIO
socketChannel.setOption(StandardSocketOptions.SO_KEEPALIVE, true);

```

Does a TCP socket connection have a "keep alive"? https://newbedev.com/does-a-tcp-socket-connection-have-a-keep-alive TCP Keepalive HOWTO https://tldp.org/HOWTO/html\_single/TCP-Keepalive-HOWTO/