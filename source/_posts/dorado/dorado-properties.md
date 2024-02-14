---
title: dorado 属性配置
tags:
  - dorado
  - propertes
id: '861'
categories:
  - - dorado
date: 2021-06-04 20:49:01
---

dorado有一套自身的属性加载机制，dorado-home下的configure.propertes和configure-debug.properties及通过dorado插件机制加载的配置文件) dorado的el表达式取属性值`${configure["core.runMode"]}`只能取`ConfigureStore`内的值。 >= 7.6.1-SNAPSHOT com.bstek.dorado.core.ConfigurePropertiesConfigurer

> com.bstek.dorado.core.ConfigureProper\[t\]iesConfigurer 7.6.1-SNAPSHOT之后修正了拼写错误，把丢失的t补上了。

### 属性的加载顺序

1) 先把dorado `ConfigureStore`的属性值存入Properties。 2) 遍历所有Spring加载到的属性，找到 `dorado.` 开头的,移除`dorado.`前缀然后重新存入`ConfigureStore`和步骤1创建的`Properties`内。 3) 然后基于`Properties`创建`PropertiesPropertySource`然后加入`propertySources`集合的最后。 简单的理解为:

1.  dorado的配置文件优先级最低。
    
2.  如果希望在Spring Boot 的application.properties里创建一个可以被dorado el表达式读取的的配置。就需要创建一个以`dorado.`开头的配置。比如配置属性`dorado.name=xobo`, 那么ConfigureStore会新增一个配置(name=xobo),Spring Boot会新增两个配置 dorado.name=xobo 和 name=xobo.
    

<7.6.1-SNAPSHOT com.bstek.dorado.core.ConfigureProperiesConfigurer 该版本省略了第二步,也就是只能dorado的配置文件导入Spring项目，但是Spring项目的配置无法导入dorado的，而且改类继承自`PropertyPlaceholderConfigurer`,Spring3.1之后就不推荐使用了,不多做分析。