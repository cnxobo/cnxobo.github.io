---
title: 使用PreparedStatement插入Date类型数据
tags:
  - date
  - java
  - jdbc
  - sql
id: '393'
categories:
  - - Java
date: 2014-09-25 00:22:19
---

使用PreparedStatement插入Date类型数据

1.  ### 类型转换
    
    java.util.Date nowDate = new java.util.Date(); // Thu Sep 25 00:05:22 CST 2014
    java.sql.Time time = new java.sql.Time(nowDate.getTime()); // 00:05:22
    java.sql.Date date = new java.sql.Date(nowDate.getTime()); // 2014-09-25
    java.sql.Timestamp timestamp = new java.sql.Timestamp(nowDate.getTime()); //2014-09-25 00:05:22.913
    
2.  ### 类型关系
    

PreparedStatement提供了三种日期类型对应JDBC中的三种日期类型(TIME, DATE,TIMESTAMP)：

*   java.sql.Time 只保存时间；时分秒，没有任何日期信息。
*   java.sql.Date 只保存日期；年月日，没有任何时间信息。
*   java.sql.Timestamp 保存日期及时间；日期+时间+**微秒**。
    1.  所有的日期都可以通过毫秒(从1970.1.1 0h0m0s已经过去的毫秒数)来构造, 如：
        
        *   Timestamp timestamp = new java.sql.Timestamp((new java.util.Date()).getTime());
    2.  都继承自java.util.Date；
    3.  虽然java.util.Date也保存时期及时间但是没有保存微秒(nanosecond)​。