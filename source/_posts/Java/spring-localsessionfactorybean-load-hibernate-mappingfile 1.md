---
title: 解决Spring LocalSessionFactoryBean加载Hibernate映射不支持通配符问题
tags: []
id: '198'
categories:
  - - Something
date: 2012-08-29 15:18:38
---

一直在使用公司的BDF，从来没有留意过加载的具体细节，在一次做培训的时候，因为没有使用BDF，Hibernate映射只能一个一个的添加不支持我常用的通配符，文件少了，没什么，要是文件多了，多麻烦呀，才让我正式的面对这个问题。我们BDF的实现方式比较复杂，之后单独写出来。 常用的映射加载方式有这么几种mappingResources、mappingLocations、mappingDirectoryLocations、mappingJarLocations。 mappingResources： 指定classpath下具体映射文件名也可以是一个List的具体映射文件名。

<!--具体映射文件名-->
<property name="mappingResources">
<value>org/xobo/domain/users.hbm.xml</value>
</property>
<property name="mappingResources" value="org/xobo/users.hbm.xml"/>
<!--或者是一个list-->
<property name="mappingResources">
<list>
<value>org/xobo/domain/users.hbm.xml</value>
<value>org/xobo/domain/dept.hbm.xml</value>
</list>
</property>

mappingLocations：可以指定任何文件路径，并且可以指定前缀：classpath、file等，也可以用通配符指定。

<property name="mappingLocations">
<value>/WEB-INF/users.hbm.xml</value>
</property>
<property name="mappingLocations">
<value>classpath:/org/xobo/domain/users.hbm.xml </value>
</property>
<property name="mappingLocations">
<value>classpath:/org/xobo/domain/\*.hbm.xml</value>
</property>

mappingDirectoryLocations：指定映射的文件路径

<property name="mappingDirectoryLocations"> <list>
<value>WEB-INF/HibernateMappings</value>
</list>
</property>
<!--也可以通过classpath来指出-->
<property name="mappingDirectoryLocations">
<list>
<value>classpath:/XXX/package/</value>
</list>
</property>

由此可知mappingLocations、mappingDirectoryLocations都可以轻松的满足我的需求，不知道我从网上搜索到的教程为什么会使用mappingResources。