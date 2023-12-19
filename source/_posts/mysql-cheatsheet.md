---
title: MySQL CheatSheet
tags:
  - CheatSheet
  - mysql
id: '608'
categories:
  - - Cheat sheet
date: 2018-01-29 15:06:21
---

### 连接

```Shell
mysql -h <host> -u <user> -p<passwd>
mysql -h <host> -u <user> -p
Enter password: ********
mysql -u user -p
mysql
mysql -h <host> -u <user> -p <Database>
```

### 查询

```SQL
 SELECT * FROM table
 SELECT * FROM table1, table2, ...
 SELECT field1, field2, ... FROM table1, table2, ...
 SELECT ... FROM ... WHERE condition
 SELECT ... FROM ... WHERE condition GROUP BY field
 SELECT ... FROM ... WHERE condition GROUP BY field HAVING condition2
 SELECT ... FROM ... WHERE condition ORDER BY field1, field2
 SELECT ... FROM ... WHERE condition ORDER BY field1, field2 DESC
 SELECT ... FROM ... WHERE condition LIMIT 10
 SELECT DISTINCT field1 FROM ...
 SELECT DISTINCT field1, field2 FROM ...

 SELECT ... FROM t1 JOIN t2 ON t1.id1 = t2.id2 WHERE condition
 SELECT ... FROM t1 LEFT JOIN t2 ON t1.id1 = t2.id2 WHERE condition
 SELECT ... FROM t1 JOIN (t2 JOIN t3 ON ...) ON ...
 SELECT ... FROM t1 JOIN t2 USING(id) WHERE condition
```

### 条件

```SQL
 field1 = value1
 field1 <> value1
 field1 LIKE 'value _ %'
 field1 IS NULL
 field1 IS NOT NULL
 field1 IN (value1, value2)
 field1 NOT IN (value1, value2)
 condition1 AND condition2
 condition1 OR condition2
```

### 数据处理

```SQL
 INSERT INTO table1 (field1, field2, ...) VALUES (value1, value2, ...)
 INSERT table1 SET field1=value_1, field2=value_2 ...

 LOAD DATA INFILE '/tmp/mydata.txt' INTO TABLE table1
 FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' ESCAPED BY '\\'

 DELETE FROM table1 / TRUNCATE table1
 DELETE FROM table1 WHERE condition
 -- join:
 DELETE FROM table1, table2 WHERE table1.id1 = table2.id2 AND condition

 UPDATE table1 SET field1=new_value1 WHERE condition
 -- join:
 UPDATE table1, table2 SET field1=new_value1, field2=new_value2, ...
 WHERE table1.id1 = table2.id2 AND condition
```

### 浏览

```SQL
 SHOW DATABASES
 SHOW TABLES
 SHOW FIELDS FROM table / SHOW COLUMNS FROM table / DESCRIBE table / DESC table / EXPLAIN table
 SHOW CREATE TABLE table
 SHOW CREATE TRIGGER trigger
 SHOW TRIGGERS LIKE '%update%'
 SHOW PROCESSLIST
 KILL process_number
 SELECT table_name, table_rows FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = '**yourdbname**';
 $ mysqlshow
 $ mysqlshow database
```

### 数据库 CRUD

```SQL
 CREATE DATABASE [IF NOT EXISTS] mabase [CHARACTER SET charset] [COLLATE collation]
 CREATE DATABASE mabase CHARACTER SET utf8
 DROP DATABASE mabase
 USE mabase

 ALTER DATABASE mabase CHARACTER SET utf8
```

### 表 CRUD

```SQL
CREATE TABLE table (field1 type1, field2 type2, ...)
CREATE TABLE table (field1 type1 unsigned not null auto_increment, field2 type2, ...)
CREATE TABLE table (field1 type1, field2 type2, ..., INDEX (field))
CREATE TABLE table (field1 type1, field2 type2, ..., PRIMARY KEY (field1))
CREATE TABLE table (field1 type1, field2 type2, ..., PRIMARY KEY (field1, field2))
CREATE TABLE table1 (fk_field1 type1, field2 type2, ...,
FOREIGN KEY (fk_field1) REFERENCES table2 (t2_fieldA)
[ON UPDATE] [CASCADESET NULLRESTRICT]
[ON DELETE] [CASCADESET NULLRESTRICT])
CREATE TABLE table1 (fk_field1 type1, fk_field2 type2, ...,
FOREIGN KEY (fk_field1, fk_field2) REFERENCES table2 (t2_fieldA, t2_fieldB))
CREATE TABLE table IF NOT EXISTS (...)

CREATE TABLE new_tbl_name LIKE tbl_name
[SELECT ... FROM tbl_name ...]

CREATE TEMPORARY TABLE table (...)

CREATE table new_table_name as SELECT [ *column1, column2 ] FROM table_name

DROP TABLE table
DROP TABLE IF EXISTS table
DROP TABLE table1, table2, ...
DROP TEMPORARY TABLE table

ALTER TABLE table MODIFY field1 type1 
ALTER TABLE table MODIFY field1 type1 NOT NULL ... 
ALTER TABLE table CHANGE old_name_field1 new_name_field1 type1
ALTER TABLE table CHANGE old_name_field1 new_name_field1 type1 NOT NULL ...
ALTER TABLE table ALTER field1 SET DEFAULT ...
ALTER TABLE table ALTER field1 DROP DEFAULT
ALTER TABLE table ADD new_name_field1 type1
ALTER TABLE table ADD new_name_field1 type1 FIRST
ALTER TABLE table ADD new_name_field1 type1 AFTER another_field
ALTER TABLE table DROP field1
ALTER TABLE table ADD INDEX (field);
ALTER TABLE table ADD PRIMARY KEY (field);

-- Change field order:
ALTER TABLE table MODIFY field1 type1 FIRST
ALTER TABLE table MODIFY field1 type1 AFTER another_field
ALTER TABLE table CHANGE old_name_field1 new_name_field1 type1 FIRST
ALTER TABLE table CHANGE old_name_field1 new_name_field1 type1 AFTER another_field

ALTER TABLE old_name RENAME new_name;
```

### 主键

```SQL
 CREATE TABLE table (..., PRIMARY KEY (field1, field2))
 CREATE TABLE table (..., FOREIGN KEY (field1, field2) REFERENCES table2 (t2_field1, t2_field2))
 ALTER TABLE table ADD PRIMARY KEY (field);
 ALTER TABLE table ADD CONSTRAINT constraint_name PRIMARY KEY (field, field2);
### create/modify/drop view
 CREATE VIEW view AS SELECT ... FROM table WHERE ...
```

### 事件调度

```SQL
-- 事件调度状态
SHOW VARIABLES LIKE 'event_scheduler'
-- 查看事件任务
SHOW EVENTS ;
-- 开启事件调度
SET GLOBAL event_scheduler = ON;
-- 关闭事件调度
SET GLOBAL event_scheduler = OFF;
-- 禁用指定事件
ALTER EVENT myevent DISABLE;
-- 启用指定事件
ALTER EVENT myevent ENABLE;
-- 事件重命名
ALTER EVENT myevent RENAME TO yourevent;
-- 创建事件
CREATE EVENT myevent ON SCHEDULE EVERY 1 MINUTE
DO
   INSERT INTO messages(message,created_at)
   VALUES('Test ALTER EVENT statement',NOW());

-- 修改事件调度
ALTER EVENT myevent ON SCHEDULE EVERY 2 MINUTE;

-- 修改事件体
ALTER EVENT myevent
DO
   INSERT INTO messages(message,created_at)
   VALUES('Message from event',NOW());

-- 创建事件(多行)
DELIMITER $$

CREATE EVENT myevent 
    ON SCHEDULE 
        EVERY 1 MINUTE
    DO
        BEGIN
            INSERT INTO messages(message,created_at)
            VALUES('Test ALTER EVENT statement',NOW());
    END $$

DELIMITER ;

-- 删除事件
DROP EVENT IF EXISTS myevent;

```

### 权限

```SQL
 CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';

 GRANT ALL PRIVILEGES ON base.* TO 'user'@'localhost' IDENTIFIED BY 'password';
 GRANT SELECT, INSERT, DELETE ON base.* TO 'user'@'localhost' IDENTIFIED BY 'password';
 REVOKE ALL PRIVILEGES ON base.* FROM 'user'@'host'; -- one permission only
 REVOKE ALL PRIVILEGES, GRANT OPTION FROM 'user'@'host'; -- all permissions

 SET PASSWORD = PASSWORD('new_pass')
 SET PASSWORD FOR 'user'@'host' = PASSWORD('new_pass')
 SET PASSWORD = OLD_PASSWORD('new_pass')

 DROP USER 'user'@'host'

 -- MySQL 分配权限时, 只支持库名的模糊匹配不支持表名的模糊匹配，可以用下面的 SQL 来生成批量的授权语句
SELECT   CONCAT('GRANT update, insert ON myDB.', TABLE_NAME, ' to ''user'';') grantSQL
FROM     INFORMATION_SCHEMA.TABLES
WHERE    TABLE_SCHEMA = 'myDB' and TABLE_NAME like 'MyTable%';
```

### 忘记 root 密码

```Shell
$ /etc/init.d/mysql stop
$ mysqld_safe --skip-grant-tables &
$ mysql # on another terminal
mysql> UPDATE mysql.user SET password=PASSWORD('nouveau') WHERE user='root';
## Kill mysqld_safe from the terminal, using Control + \
$ /etc/init.d/mysql start
```

### 异常关闭后,表修复

```Shell
mysqlcheck --all-databases
mysqlcheck --all-databases --fast
```

### 加载数据

```Shell
mysql> SOURCE input_file
$ mysql database < filename-20120201.sql
$ cat filename-20120201.sql  mysql database
```

### 参考链接

https://en.wikibooks.org/wiki/MySQL/CheatSheet