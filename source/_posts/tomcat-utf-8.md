---
title: Tomcat与UTF-8，告别中文乱码
tags:
  - java
  - tomcat
  - utf-8
id: '654'
categories:
  - - Java
date: 2018-04-15 19:59:30
---

## 简介

UTF-8是网页应用中最常用的字符编码。它支持世界上正在使用的所有语言，包括中日韩。 本文我们会展示所有的配置以确保在Tomcat中使用 UTF-8。

## 连接器(Connector) 配置

一个连接器在指定的端口上监听连接。我们需要确保我们所有的连接器都使用UTF-8来编码请求。 给TOMCAT\_ROOT/conf/server.xml里的所有的连接器添加一个参数 URIEncoding=”UTF-8″ 。

```XML
<Connector URIEncoding="UTF-8"  port="8080"  redirectPort="8443"  connectionTimeout="20000"  protocol="HTTP/1.1"/>

<Connector
  URIEncoding="UTF-8" port="8009" redirectPort="8443" protocol="AJP/1.3"/>
```

## 字符集过滤器

配置过连接器之后， 我们该强制网页程序使用UTF-8去处理所有的请求与响应。 如果我们在使用Spring，直接注册CharacterEncodingFilter到web.xml中, 同时要确保该过滤器是web.xml中的第一个过滤器。

```XML
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

不然的话就需要自己定义一个CharacterSetFilter的类。

```JAVA
public class CharacterSetFilter implements Filter {
    // ...
    public void doFilter(
      ServletRequest request, 
      ServletResponse response, 
      FilterChain next) throws IOException, ServletException {
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=UTF-8");
        response.setCharacterEncoding("UTF-8");
        next.doFilter(request, response);
    }
    // ...
}
```

然后我们把这个过滤器添加到web.xml里:

```XML
<filter>
    <filter-name>CharacterSetFilter</filter-name>
    <filter-class>com.baeldung.CharacterSetFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>CharacterSetFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## Servlet 页编码

我们的网页程序另一部分需要配置的是Servlet 页。 确保Servlet使用UTF-8的最好方法是添加这个标签到每个JSP页的最头部:

```JSP
<%@page pageEncoding="UTF-8" contentType="text/html; charset=UTF-8"%>
```

## HTML页编码

Servlet页编码是告诉JVM如何处理字符，HTML页编码是告诉浏览器如何处理字符。 我们应该添加这个标签  到所有HTML的 head 区域。

```HTML
<meta http-equiv='Content-Type' content='text/html; charset=UTF-8' />
```

参考链接： [Making Tomcat UTF-8-Ready](http://www.baeldung.com/tomcat-utf-8 "Making Tomcat UTF-8-Ready") [How to use UTF-8 everywhere](https://wiki.apache.org/tomcat/FAQ/CharacterEncoding#Q8 "How to use UTF-8 everywhere")