---
title: "MaxCompute"
tags: []
categories: []
date: "2025-03-28"
---

| 表类型                 | UPDATE | DELETE | INSERT INTO | INSERT OVERWRITE | 分区  | 创建方式                                   |
| ------------------- | ------ | ------ | ----------- | ---------------- | --- | -------------------------------------- |
| 普通表<br>             | ❌      | ❌      | ✅           | ✅                | ✅   |                                        |
| Transactional Table | ✅      | ✅      | ✅           | ✅                | ✅   | TBLPROPERTIES ("transactional"="true") |
| Delta Table<br>     | ✅      | ✅      | ✅           | ✅                | ✅   | Transactional 基础之上指定<br>PRIMARY KEY    |
| 聚簇表                 | ❌      | ❌      | ⚠️(不支持追加)   | ✅                | ✅   | CLUSTERED BY...<br>INTO n buckets      |
