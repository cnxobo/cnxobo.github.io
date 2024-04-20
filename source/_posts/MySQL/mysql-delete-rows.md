---
title: mysql 删除行数据
tags:
  - mysql
categories: 
date: 2024-04-20
---
### 删除单表数据
```SQL
DELETE FROM table_name WHERE column_name  = 'value'    
```
### 删除单表数据带别名
```SQL
DELETE t FROM table_name t WHERE t.column_name  = 'value'    
```
### 删除单表数据带 LEFT JOIN
```SQL
DELETE t1 FROM table_name1 t1 LEFT JOIN table_name2 ON t2 t1.key = t2.key WHERE t2.column_name2  = 'value'    
```

### 删除多表数据带 INNER JOIN
```SQL
DELETE t1, t2 FROM table_name1 t1 INNER JOIN table_name2 ON t2 t1.key = t2.key WHERE t2.column_name2  = 'value'    
```
### 删除单表数据带同表子查询
>如果直接使用同表子查询, SQL会无法执行 "You can't specify target table for update in FROM clause"

可以通过把子查询包括在另一个子查询里，强制MySQL生成一个临时表来解决。
```SQL
DELETE FROM table_name WHERE id IN (
	SELECT tp.id FROM (SELECT id FROM table_name WHERE column_name = 'value2') tp
)
```
也可以尝试使用关联自身(LEFT/INNER JOIN)的方式来删除。

### 删除所有数据
```SQL
DELETE FROM table_name
```
也可以使用
```SQL
TRUNCATE TABLE table_name;
```
DELETE 语句可以生成binlog，在事务内可以还能回退，会触发Triggers，但不会自动释放表空间。
TRUNCATE 不会产生binlog，也不能回退，不会触发Triggers，但是能释放表空间，还有他很快。。。