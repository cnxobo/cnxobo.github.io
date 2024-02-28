---
title: 如何清空 DNS 缓存
tags:
  - 日志
id: '628'
categories:
  - - Something
date: 2018-02-09 21:31:10
---

### Windows

```bash
ipconfig /flushdns
```

### Linux (根据实际运行的服务执行相应的命令)

```bash
/etc/init.d/named restart
/etc/init.d/nscd restart
```

### macOS

```bash
sudo killall -HUP mDNSResponder
```

#### 例外:

OSX Yosemite 10.10.0 – 10.10.3

```bash
sudo discoveryutil mdnsflushcache
```

OSX Leopard Snow Leopard 10.5 – 10.6

```bash
sudo dscacheutil -flushcache
```