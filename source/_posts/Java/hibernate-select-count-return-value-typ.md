---
title: Hibernate select count(*)返回值类型Integer vs Long
tags: []
id: '205'
categories:
  - - hibernate
date: 2012-09-06 15:07:18
---

> **COUNT 返回Long对象 MAX MIN 返回类型是跟所使用的字段类型有关 AVG 返回Double SUM 如使用字段是整形，返回Long(以前是返回BigInteger),如果是浮点类型那么就返回Double.**

由于接触Hibernate较晚, 一直坚定不移的相信select count(\*) 的返回值的类型的是Long. 一次很偶然的机会发现在一段分页函数里发现在使用select count(\*)之后拿到的对象，居然先判断是Long类型还是Integer类型。心中大惊，总不会是在不同的数据库里它的表示不一样吧，不找出原因心难安。 在[csdn](http://topic.csdn.net/u/20080725/14/239ee1ee-c1c7-4489-b752-22e884476b61.html "关于Hibernate读取select count(*)的返回值到底是Long还是Integer的疑惑")上看到有人提出类似问题。其中提到

> 关于在Hibernate里使用select count(\*) 返回值的问题说明 由于我使用的是Hibernate 3.2版本，经确认，这个版本已经把以前返回 Integer的改成了 Long, 因为JPA里面的返回值规定是Long, Hibernate为了兼容这个，所以修改了返回值。 如果你从Hibernate 3.0.x/3.1.x升级到最新的3.2版，一定要注意，3.2版的很多sql函数如count(), sum()的唯一返回值已经从Integer变为Long，如果不升级代码，会得到一个ClassCastException。

应该就是正解了，但是国内的信息太多的复制粘贴了，经过大量的转发，原本正确的信息也可能变成了错误了，加上其中提到的原贴已经打不开了。我就想找到hibernate官方文档，这样才放心。 在[Migrating from 3.1 to 3.2](https://community.jboss.org/wiki/HibernateCoreMigrationGuide32 "Migrating from 3.1 to 3.2")的官方文档里找到如下信息

> Changed aggregation (count, sum, avg) function return types In alignment with the JPA specification the count, sum and avg function now defaults to return types as specified by the specification. This can result in ClassCastException's at runtime if you used aggregation in HQL queries. The new type rules are described at http://opensource.atlassian.com/projects/hibernate/browse/HHH-1538 If you cannot change to the JPA compliant type handling the following code can be used to provide "classic" Hibernate behavior for HQL aggregation: Configuration classicCfg = new Configuration(); classicCfg.addSqlFunction( "count", new ClassicCountFunction()); classicCfg.addSqlFunction( "avg", new ClassicAvgFunction()); classicCfg.addSqlFunction( "sum", new ClassicSumFunction()); SessionFactory classicSf = classicCfg.buildSessionFactory();

证明了为了兼容JPA，hibernate确实修改了 aggregation (count, sum, avg) function的返回值，但是文档中并没有提到说，把返回值由Integer改为Long. 根据文件中的链接“http://opensource.atlassian.com/projects/hibernate/browse/HHH-1538”。

> The Java type that is contained in the result of a query using an aggregate function is as follows\[33\]: • COUNT returns Long. • MAX, MIN return the type of the state-field to which they are applied. • AVG returns Double. • SUM returns Long when applied to state-fields of integral types (other than BigInteger); Double when applied to state-fields of floating point types; BigInteger when applied to state-fields of type BigInteger; and BigDecimal when applied to state-fields of type BigDecimal.

这里详细的描述了各个函数的返回值类型的变化 COUNT 返回Long对象 MAX MIN 返回类型是跟所使用的字段类型有关 AVG 返回Double SUM 如使用字段是整形，返回Long(以前是返回BigInteger),如果是浮点类型那么就返回Double... 由此可知我们之后可以放心的去用Long类型来接收count(\*)的返回值了。要是还不放心，我们还可以使用Number类型来接收. 没用过Number类型？

> java.lang.Number The abstract class Number is the superclass of classes BigDecimal, BigInteger, Byte, Double, Float, Integer, Long, and Short. Subclasses of Number must provide methods to convert the represented numeric value to byte, double, float, int, long, and short.

基本上所有数据类型都继承了这个Number类，而这个类又定义了longValue,doubleValue方法，这样管它Integer还是Long都是没问题的啦，而且我们还能取出我们需要的值。 将对象赋值给接口或者抽象类应该也是Java中经常用法吧。