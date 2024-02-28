---
title: Spring Boot接入 apollo 后启动 dubbo 报错
tags: []
id: '745'
categories:
  - - logs
date: 2019-04-28 14:31:38
---

某Spring Boot项目接入 apollo 后启动 dubbo 报错`Caused by: java.lang.IllegalStateException: ApplicationConfig.application == null`.

## TLDR

1.  apollo-client 版本>= 1.2.0。 [点击查看Apollo Client最新版本](https://mvnrepository.com/artifact/com.ctrip.framework.apollo/apollo-client)
2.  在`application.properties`内添加配置`apollo.bootstrap.eagerLoad.enabled=true`。

## 解决步骤复盘

根据异常猜测是 dubbo 启动时未读取到 apollo 内的配置。

##### 0x01

先Google关键字 `dubbo 2.6.2 apollo`, 第二条 "继承dubbo配置不生效...".

##### 0x02

点开第二条之后发现跟我们的问题毫无关系，但是里面提到 `apollo use cases`.

##### 0x03

使用 SourceTree 下载 [apollo-use-cases](https://github.com/ctripcorp/apollo-use-cases/tree/master/spring-boot-dubbo)，并在本机跑起来，发现没问题。

##### 0x04

对比支付系统与其中的spring-boot-dubbo-consumer项目的异同。 1. dubbo 版本一致 2. dubbo-spring-boot-starter版本差 0.0.1 3. 支付系统启动类上多`@EnableJpaRepositories, @EnableCaching,@EnableSwagger2`。 4. 发现支付系统启动类里上还多一个`@ImportResource(value = {"classpath*:commons-dubbo-api.xml"}`.

##### 0x05

在发现的异同里，跟 dubbo 关系最大的就是第四条。给spring-boot-dubbo-consumer的添加`commons-dubbo-api-balcony` 依赖，然后在启动类上添加注解 `@ImportResource(value = {"classpath*:commons-dubbo-api.xml"}`问题复现。可以肯定这个问题是 apollo 与 dubbo 的兼容性问题。

##### 0x06

访问 [appollo issues](https://github.com/ctripcorp/apollo/issues), 以 `dubbo` 作为关键字搜索。结果差不多一页，`⌘ + F` 用浏览器的查找`配置`.在第三个高亮项里找到跟我一模一样的问题。 `1.2.0版本已经发布，支持把Apollo配置加载提到初始化日志系统之前，可以解决此issue中提到的问题。` ，升级 apollo 到 1.2.0 问题依旧存在。

##### 0x07

回顾该 issue, 之前提到 `不过测试了一下，#1614 合并后就可以解决这个问题。`，进入[#1614](https://github.com/ctripcorp/apollo/pull/1614) 发现配置`apollo.bootstrap.eagerLoad.enabled`项,把 `apollo.bootstrap.eagerLoad.enabled=true`放入项目application.properties,可成功启动项目。

* * *

## 相关截图

##### 0x01

![1.png](https://i.loli.net/2019/04/28/5cc53b326e1de.png)

##### 0x02

![2.png](https://i.loli.net/2019/04/28/5cc53d9382885.png)

##### 0x04

![3.png](https://i.loli.net/2019/04/28/5cc53e062bdd9.png)

##### 0x06

![4.png](https://i.loli.net/2019/04/28/5cc53e3c09dcb.png)

##### 0x07

![5.png](https://i.loli.net/2019/04/28/5cc53e90cd8c5.png) 解决问题总耗时 30 分钟左右。

## 完整异常

```Java
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'alipayController': Unsatisfied dependency expressed through field 'payService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'payServiceImpl': Unsatisfied dependency expressed through field 'commandInvokeService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'commandInvokeServiceImpl': Unsatisfied dependency expressed through method 'setCommandMap' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayBatchPayCommand': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayServiceImpl': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:846) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:863) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:546) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:142) ~[spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:775) [spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397) [spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:316) [spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1260) [spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:1248) [spring-boot-2.1.2.RELEASE.jar:2.1.2.RELEASE]
    at com.ezhiyang.pay.PayApplication.main(PayApplication.java:22) [classes/:na]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_144]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_144]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_144]
    at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_144]
    at org.springframework.boot.devtools.restart.RestartLauncher.run(RestartLauncher.java:49) [spring-boot-devtools-2.1.2.RELEASE.jar:2.1.2.RELEASE]
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'payServiceImpl': Unsatisfied dependency expressed through field 'commandInvokeService'; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'commandInvokeServiceImpl': Unsatisfied dependency expressed through method 'setCommandMap' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayBatchPayCommand': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayServiceImpl': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1244) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1164) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:593) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 24 common frames omitted
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'commandInvokeServiceImpl': Unsatisfied dependency expressed through method 'setCommandMap' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayBatchPayCommand': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayServiceImpl': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:676) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1244) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1164) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:593) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 37 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayBatchPayCommand': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayServiceImpl': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessProperties(CommonAnnotationBeanPostProcessor.java:324) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.addCandidateEntry(DefaultListableBeanFactory.java:1460) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1424) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveMultipleBeans(DefaultListableBeanFactory.java:1315) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1202) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1164) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:668) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 50 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'alipayServiceImpl': Injection of resource dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessProperties(CommonAnnotationBeanPostProcessor.java:324) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:575) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:498) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:320) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:318) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:277) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1244) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1164) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.autowireResource(CommonAnnotationBeanPostProcessor.java:518) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.getResource(CommonAnnotationBeanPostProcessor.java:496) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor$ResourceElement.getResourceToInject(CommonAnnotationBeanPostProcessor.java:630) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata$InjectedElement.inject(InjectionMetadata.java:180) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessProperties(CommonAnnotationBeanPostProcessor.java:321) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 64 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dubbo.balconyService': FactoryBean threw exception on object creation; nested exception is java.lang.IllegalStateException: ApplicationConfig.application == null
    at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:178) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.getObjectFromFactoryBean(FactoryBeanRegistrySupport.java:101) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getObjectForBeanInstance(AbstractBeanFactory.java:1674) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.getObjectForBeanInstance(AbstractAutowireCapableBeanFactory.java:1216) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:257) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:204) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.autowireResource(CommonAnnotationBeanPostProcessor.java:525) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.getResource(CommonAnnotationBeanPostProcessor.java:496) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor$ResourceElement.getResourceToInject(CommonAnnotationBeanPostProcessor.java:630) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata$InjectedElement.inject(InjectionMetadata.java:180) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    at org.springframework.context.annotation.CommonAnnotationBeanPostProcessor.postProcessProperties(CommonAnnotationBeanPostProcessor.java:321) ~[spring-context-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 80 common frames omitted
Caused by: java.lang.IllegalStateException: ApplicationConfig.application == null
    at com.alibaba.dubbo.config.AbstractConfig.appendParameters(AbstractConfig.java:246) ~[dubbo-2.6.2.jar:2.6.2]
    at com.alibaba.dubbo.config.AbstractConfig.appendParameters(AbstractConfig.java:181) ~[dubbo-2.6.2.jar:2.6.2]
    at com.alibaba.dubbo.config.ReferenceConfig.init(ReferenceConfig.java:303) ~[dubbo-2.6.2.jar:2.6.2]
    at com.alibaba.dubbo.config.ReferenceConfig.get(ReferenceConfig.java:163) ~[dubbo-2.6.2.jar:2.6.2]
    at com.alibaba.dubbo.config.spring.ReferenceBean.getObject(ReferenceBean.java:66) ~[dubbo-2.6.2.jar:2.6.2]
    at org.springframework.beans.factory.support.FactoryBeanRegistrySupport.doGetObjectFromFactoryBean(FactoryBeanRegistrySupport.java:171) ~[spring-beans-5.1.4.RELEASE.jar:5.1.4.RELEASE]
    ... 91 common frames omitted
Caused by: java.lang.IllegalStateException: ApplicationConfig.application == null
    at com.alibaba.dubbo.config.AbstractConfig.appendParameters(AbstractConfig.java:231) ~[dubbo-2.6.2.jar:2.6.2]
    ... 96 common frames omitted

```