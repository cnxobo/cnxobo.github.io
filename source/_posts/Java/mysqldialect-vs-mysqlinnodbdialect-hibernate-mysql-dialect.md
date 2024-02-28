---
title: Deprecated 过时的 MySQL5InnoDBDialect
tags:
  - dialect
  - hibernate
id: '423'
categories:
  - - hibernate
date: 2015-05-04 23:51:42
---

### 什么是Hibernate方言？

Hibernate方言是用来告诉Hibernte如何对指定的数据库生成相应的SQL语句。 尽管做了很多尝试去使SQL语句标准化，但是不同的数据库支持的SQL语句还是有很多不同的地方。 所以Hibernate使用方言来辅助生成正确的SQL语句。

### MySQL5Dialect与MySQL5InnoDBDialect有什么区别？

他们最大的区别是，在使用Hibernate创建表时MySQL5InnoDBDialect会在生成的**建表**SQL语句最后加上"ENGINE=InnoDB"。 InnoDB是一种MySQL数据库引擎.MySQL5.5及之后使用它做为默认引擎。它提供了ACID兼容的事务（Transaction）功能，并提供外键支持。

### MySQLDialect与MySQL5Dialect有什么区别？

MySQLDialect是针对MySQL5之前的版本。主要变化还是在于**建表**SQL语句。 MySQL由于4到5还是有不小的变化。比如varchar在4及之前版本最大长度限制为255，5及之后版本最大长度限制为65535。 MySQLInnoDBDialect会在生成的**建表**SQL语句最后加上"TYPE=InnoDB"。

### MySQL5InnoDBDialect过时了

升级到 Hibernate 5 的时候，就会发现 MySQL5InnoDBDialect，被标注@Deprecated也就是过时了。不仅仅是 MySQL5InnoDBDialect 过时了，所有带InnoDB的 Dialect 都被标注过时了@Deprecated；在标注有 InnoDBDialect 过时的同时 新加了 MySQL55Dialect 及 MySQL57Dialect。 如果查看源码就会发现 MySQL55Dialect 与 MySQL5InnoDBDialect 源码一模一样。 毕竟 MySQL 从 5.5 开始就默认使用 InnoDB 引擎，MySQL 8 已经移除了 MyISAM 引擎。Hibernate 的作者认为Dialect分为两类就没有什么必要了。

```Java
org.hibernate.dialect.MySQL55Dialect
org.hibernate.dialect.MySQL57Dialect
```

### 后记

Hibernate 作者最坑的地方是标注了@Deprecated之时，补了一个注释“Use "hibernate.dialect.storage\_engine=innodb" environment variable or JVM system property instead.”，导致我以为是通过配置的方式来指定引擎。我在 Spring Boot 中各种拼配置都没成功，后来发现这个配置是要写在 classpath 下的 hibernate.properties 内。