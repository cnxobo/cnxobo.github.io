---
title: MySQL 记录所有SQL和慢SQL
tags: []
id: '772'
categories:
  - - Database
comments: false
date: 2019-10-24 19:24:00
---

### 通过配置文件配置日志

#### my.cnf 位置

查看自己版本 MySQL 默认读取配置文件的路径

```Shell
mysql --help  grep my.cnf
```

我的这个版本的 MySQL 会以以下顺序读配置文件 1. /etc/my.cnf 2. /etc/mysql/my.cnf 3. /usr/local/etc/my.cnf 4. ~/.my.cnf 如果使用了自定义位置可以通过查看进程启动参数 `--defaults-file`

```Shell
ps aux  grep mysqld
```

#### 日志配置

**通用SQL日志** ([General Query Log](https://dev.mysql.com/doc/refman/5.7/en/query-log.html)) 记录所有mysqld做的事,连接、断开、查询。

```
# 通用SQL日志
general_log_file        = /var/log/mysql/mysql.log
general_log             = 1
```

**慢SQL日志** ([Slow Query Log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)) 记录慢SQL [long\_query\_time](https://dev.mysql.com/doc/refman/5.5/en/server-system-variables.html#sysvar_long_query_time) 默认10，单位秒；SQL执行时间比long\_query\_time长的都会被记录。

```
# 慢SQL日志
log_slow_queries = /var/log/mysql/mysql-slow.log
long_query_time = 10
log-queries-not-using-indexes = ON

```

### 在运行时开启日志

开启日志，登录 mysql client (mysql -u root -p) 然后执行:

```Shell
SET GLOBAL general_log = 'ON';
SET GLOBAL slow_query_log = 'ON';
```

关闭日志，登录 mysql client (mysql -u root -p) 然后执行:

```Shell
SET GLOBAL general_log = 'OFF';
SET GLOBAL slow_query_log = 'OFF';
```

即时生效，不需要重启。 查看日志文件位置

```Shell
show variables like '%log_file';
```