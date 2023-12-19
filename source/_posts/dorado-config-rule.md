---
title: dorado ide 更新规则
tags:
  - dorado
  - spring security
id: '851'
categories:
  - - dorado
comments: false
date: 2021-04-23 20:59:03
---

> **dorado eclipse 插件** 百度网盘: [https://pan.baidu.com/s/1chalW5ebFOC3cKkYJLvkig](https://pan.baidu.com/s/1chalW5ebFOC3cKkYJLvkig  "百度网盘" target="_blank") 提取码: xobo

dorado可以自定义控件，所以dorado ide 在设计的时候并没有把控件写死，而是采用基于配置的方式来加载控件及控件对应的事件、属性。 dorado配置规则是基于当前项目动态生成的，然后保存在项目根目录的`.rules`文件里。控件对应的图标则保存在`.setting`目录里。这个设计也带来了弊端--每个新项目都要手动更新一下dorado配置规则。 dorado的配置规则更新有两方式：离线、在线。

### 离线模式

IDE的默认方式，需要指定一个dorado home目录，然后IDE通过加载Spring配置文件，解析出dorado控件信息。该模式适合简单项目，我经常会遇到有自定义控件加载不到的情况。 配置方式:

1.  在项目名上点右键，然后选择"Update Dorado Config Rules"(更新dorado配置规则)。
    
2.  在弹出框里选择Dorado Home目录。BDF2项目通常在`\web\WEB-INF\dorado-home`，BDF3项目通常在`\src\main\resources\dorado-home`位置。
    

![](https://i.loli.net/2021/04/23/6pTfaUIJARceqB4.png)

### 在线模式

个人推荐。需要先启动dorado项目。dorado ide会访问项目的`/dorado/ide/config-rules.xml`路径，获取dorado的配置规则，然后保存到本地。这个模式是加载最完整的。 配置方式：

1.  启动项目
    
2.  先在Eclipse的Preference的Dorado7 Studio，里把更新规则的模式切换成Online (在线模式).
    

Windows下菜单Window -> Preferences MacOS下，快捷键⌘+,

2.  在项目名上点右键，然后选择"Update Dorado Config Rules"(更新dorado配置规则)。
    
3.  在弹出框里填写服务器的地址。
    
    > Server Name 填域名或者IP，不带http/https前缀 Port 填端口号 AppName 填context path,如果是直接访问可以留空。
    

![](https://i.loli.net/2021/04/22/JKCSLUd2Z6yGh3j.jpg) 在线模式下 ServerName对应着域名或ip, Port就是端口号, App Name就是contextpath，如果为空可以不填。图片中是示例访问地址为: http://localhost:8080/waterdrop 。如果 项目访问地址是: http://localhost:8080

> Server Name: localhost Port: 8080 App Name:

但是也是最需要容易踩坑的：

*   系统必须可以匿名访问`/dorado/ide/config-rules.xml`路径
    
*   由于dorado IDE的bug，其更新规则时真实的请求是`POST //dorado/ide/config-rules.xml`，路径中出现双斜杠。双斜杠这种非标准的路径可能会被防火墙拦截，比如Spring Security的`HttpFirewall`。这个时候需要注册一个我们自己的`HttpFireWall`,然后允许路径中出现双斜杠。
    
    ```java
    @Bean
    public HttpFirewall looseHttpFirewall() {
      StrictHttpFirewall firewall = new StrictHttpFirewall();
      // 允许路径中带双斜杠("//")
      firewall.setAllowUrlEncodedDoubleSlash(true);
    
      // 清空所有策略，不建议使用。
      // firewall.getEncodedUrlBlacklist().clear();
      return firewall;
    }
    ```
    

附录： HttpFirewall 拦截的异常信息

```java
org.springframework.security.web.firewall.RequestRejectedException: The request was rejected because the URL contained a potentially malicious String "//"
    at org.springframework.security.web.firewall.StrictHttpFirewall.rejectedBlacklistedUrls(StrictHttpFirewall.java:369)
    at org.springframework.security.web.firewall.StrictHttpFirewall.getFirewalledRequest(StrictHttpFirewall.java:336)
    at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:194)
    at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:178)
    at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:358)
    at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:271)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
    at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
    at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
    at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
    at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:367)
    at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
    at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:868)
    at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1639)
    at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:748)
```