---
title: Hibernate dynamic-update dynamic-insert 作用及影响
tags:
  - annotation
  - hibernate
  - java
id: '387'
categories:
  - - hibernate
  - - Java
date: 2014-09-19 14:00:30
---

dynamic-update 可选 默认false UPDATE SQL会在运行时生成且只包括**值发生变化的列**。

dynamic-insert 可选 默认false INSERT SQL会在运行时生成且只包括**值不为null的列**。[1](#fn1)

### 配置方式：

Hibernate3:

@org.hibernate.annotations.Entity(
dynamicUpdate = true,
dynamicInsert = true
)

Hibernate4:

@DynamicUpdate(value=true)
@DynamicInsert(value=true)

### 性能影响：

Hibernate会为每一个entity缓存它的INSERT/SELECT/UPDATE SQL语句。这样我们在保存、查找、更新entity时，不需要再去动态的计算这些SQL。

然而，当我们使用dynamic-insert、dynamic-update时, Hibernate都要重新计算一次相应的SQL语句，在Hibernate层面会有性能损耗。

所以我们需要在数据库及Hibernate的开销上做权衡。

在我看来，只有表中有庞大的blog列或者有超多的列，dynamic insert、 dynamic update才有意义；否则我不认为它们会带来性能提升。[2](#fn2)

### 使用效果

仅仅提供使用效果，但我不认同其作者对性能影响的评价。

[Dynamic-insert](http://www.mkyong.com/hibernate/hibernate-dynamic-insert-attribute-example/)

[Dynamic-update](http://www.mkyong.com/hibernate/hibernate-dynamic-update-attribute-example/ "dynamic-update")

1.  [http://docs.jboss.org/hibernate/orm/3.6/reference/en-US/html/mapping.html](http://docs.jboss.org/hibernate/orm/3.6/reference/en-US/html/mapping.html) [↩](#ffn1)
2.  [http://stackoverflow.com/questions/3404630/hibernate-dynamic-update-dynamic-insert-performance-effects](http://stackoverflow.com/questions/3404630/hibernate-dynamic-update-dynamic-insert-performance-effects) [↩](#ffn2)