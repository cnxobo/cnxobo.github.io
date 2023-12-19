---
title: HSQL的BLOB
tags: []
id: '134'
categories:
  - - Database
date: 2012-05-17 20:50:29
---

公司BDF的Demo使用的是HsqlDB，每次启动quartz都会抛一个异常出来。 \[code lang="java"\]java.lang.RuntimeException: org.quartz.JobPersistenceException: Couldn't store job: data exception: string data, right truncation \[See nested exception: java.sql.SQLDataException: data exception: string data, right truncation\]\[/code\] 没有更多的错误提示信息也不知道怎么定位出错的地方。看到网上有人说是因为2.0之后"BINARY without the length L means a single byte"。打开quartz的HsqlDB的脚本文件tables\_hsqldb搜索binary, 发现还真有几个binary。同时打开quartz的mysql的脚本对应的数据类型是blob。这下就好办了，把binary换成LONGVARBINARY。 参考网页：[http://db.apache.org/ddlutils/databases/hsqldb.html](http://db.apache.org/ddlutils/databases/hsqldb.html "http://db.apache.org/ddlutils/databases/hsqldb.html")