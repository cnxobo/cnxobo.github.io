---
title: NoClassDefFoundError:org.apache.juli.logging.LogFactory Jetty
tags:
  - java
  - jetty
  - NoClassDefFoundError
  - tomcat
id: '433'
categories:
  - - Java
  - - Something
date: 2015-05-11 22:46:22
---

使用Jetty启动项目的时候会报NoClassDefFoundError:org.apache.juli.logging.LogFactory异常，而Tomcat可以正常启动，这种情况是因为tomcat的包配置不正确引起的 移除掉tomcat的Library就可以了 具体步骤： 右键项目 -> Build Path -> Configure Build Path... -> Libraries 移除掉tomcat。 ![移除Apache Tomcat Library](http://7xj1gb.com1.z0.glb.clouddn.com/remove library.png) 参考自：[http://www.oschina.net/question/170972\_218538](http://www.oschina.net/question/170972_218538)