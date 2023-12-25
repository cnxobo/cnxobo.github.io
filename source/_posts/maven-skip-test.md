---
title: Maven 跳过测试 skip test
tags:
  - java
  - maven
categories:
  - - Java
date: 2023-12-25 20:04:22
---
跳过测试并不是一个好的做法，但是很多时候不得不这么做:D

### 通过命令行
```Shell
# 编译测试类，跳过测试类的执行
mvn clean package -DskipTests
# 跳过测试类的编译和执行
mvn clean package -Dmaven.test.skip=true

# 跳过测试类的编译和执行，PowerShell 专属写法。
mvn clean package --% -Dmaven.test.skip=true
mvn clean package `-Dmaven.test.skip=true
mvn clean package '-Dmaven.test.skip=true'
```
>PowerShell有自己特殊的参数处理方式，会截断`-Dmaven.test.skip=true`为两个参数，导致命令执行失败，需要额外的转义,也可以用\`, 单引号。


### 默认跳过测试
命令行参数的方式需要每次执行命令的时候添加额外的参数，如果希望默认情况下就跳过测试阶段，可以通过配置`skipTests`或`maven.test.skip`属性。
```XML
<properties>
	<skipTests>true</skipTests>
	<!-- 或 -->
	<maven.test.skip>true</maven.test.skip>
</properties>
```

> -Dxxx 和 \<properties\> 都是Maven配置属性的一种方式, 前者的优先级高于后者。
### 参考链接
[Maven Surefire Plugin – Skipping Tests](https://maven.apache.org/surefire/maven-surefire-plugin/examples/skipping-tests.html)
[IntegrationTestMojo.java](https://github.com/apache/maven-surefire/blob/master/maven-failsafe-plugin/src/main/java/org/apache/maven/plugin/failsafe/IntegrationTestMojo.java)
