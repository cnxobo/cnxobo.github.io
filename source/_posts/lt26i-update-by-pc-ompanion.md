---
title: lt26i无法使用Sony PC Companion更新
tags:
  - lt26i
  - Sony PC Companion
  - 服务
id: '146'
categories:
  - - Android
date: 2012-06-11 15:02:40
---

我的lt26i给出提示，说有新的更新，6.0.A.3.75。虽然没有看到update log，还是想升上去看看的。让我惊讶的是，Sony的手机居然要使用PC端来升级，而不是android通常使用的OTA的方式。 无所谓了，我反正我电脑也一直不离手的。 Sony PC Companion也已经装过了，它也提示有新的更新，原以为一切正常呢，在准备的时候它提示错误“无法安装或启动手机软件更新组件”，然后就结束了。试了两次，反应都一样。我就想会不会跟豌豆荚有冲突，关掉豌豆荚的自动检测手机，重启电脑。问题依旧，Google之。发现我不是一个人，好多人都跟我一样在使用Sony PC Companion时遇到这个问题。 总结大家的解决方案，感觉就有有两条是有意义的 1. 安装jre 2. 检查Sony PC Companion的服务 我电脑已经有了jdk1.5,及jdk1.6有人说要用jre7,我抱着侥幸的心理去下载jdk7, 做为一个Java程序员，抱着Sony PC Companion只能运行于Java7的心理，想想就汗颜。 再下载jdk7的时候，我去检查了系统服务，发现Sony PC Companion处于禁用状态，改为手动启动，再启动之。再运行Sony PC Companion，可以更新。 再抱怨一句，Sony的这个更新速度真慢。 关于jre的下载，程序员就自食其力吧，其它人点这里[Java.com](http://www.java.com/zh_CN/ "Java.com")