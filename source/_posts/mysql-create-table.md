---
title: mysql create table
tags:
  - mysql
id: '938'
categories:
  - - MySQL
comments: false
date: 2022-01-10 20:43:07
---

## 建表

```SQL
CREATE TABLE table [IF NOT EXISTS] (
    field1 type1, 
    field2 type2,
    INDEX IDX_FIELD (field),
    PRIMARY KEY (field1)
)
```

### CREATE TABLE ... LIKE ...

基于一张表来创建另一个表 一模一样复制表结构, 包括主键、索引、默认值等信息。

```SQL
CREATE TABLE t2 LIKE t1;
```

复制表和数据，还希望完整保留主键、索引、默认值等信息。

```SQL
CREATE TABLE t2 LIKE t1;
INSERT INTO t2 SELECT * FROM t1;
```

### CREATE TABLE ... SELECT

可以认为是 CREATE TABLE 语句 追加一个 SELECT 语句用来填充数据。 CREATE TABLE 中的表定义的列 和 SELECT 列如果不存在，SELECT 中的列会追加到表的最后。 SELECT 列类型与CREATE TABLE中定义不一样，以CREATE TABLE定义为准。 创建表且复制数据，但是不需要主键、索引、默认值等信息。

```SQL
CREATE TABLE t2 [AS] SELECT * FROM t1;
```

只创建表, 其他什么都不需要。

```SQL
CREATE TABLE t2 [AS] SELECT * FROM t1 LIMIT 0;
```

复制表和数据，只想保留主键

```SQL
CREATE TABLE t2 (PRIMARY KEY (`ID`)) [AS] SELECT * FROM t1;
```

## 删表

```SQL
DROP TABLE t1;
DROP TABLE t1, t2, ...;
DROP TEMPORARY TABLE t1;
DROP TEMPORARY TABLE t1, t2, ...;
```

删除存在的表

```SQL
DROP TABLE IF EXISTS t1;
DROP TEMPORARY TABLE IF EXISTS t1;
```

## 参考链接

[CREATE TABLE](https://dev.mysql.com/doc/refman/8.0/en/create-table.html) [CREATE TABLE ... LIKE Statement](https://dev.mysql.com/doc/refman/8.0/en/create-table-like.html) [CREATE TABLE ... SELECT Statement](https://dev.mysql.com/doc/refman/8.0/en/create-table-select.html)