---
title: Spring Security 3 升级导致 Request method 'POST' not supported
tags: []
id: '965'
categories:
  - - Java
comments: false
date: 2022-07-25 21:58:39
---

一个上古项目由于安全原因需要升级依赖，其中Spring 版本需要由3升级到5。完成版本升级之后GET请求的接口都是正常的，POST请求的接口都会报Request method 'POST' not supported。

> \[AbstractHandlerExceptionResolver:199\] - Resolved \[org.springframework.web.HttpRequestMethodNotSupportedException: Request method 'POST' not supported\]

在debug跟踪代码，发现请求会在CsrfFilter里被转发到`/`然后报错。通过查阅[Spring迁移文档3to4](https://docs.spring.io/spring-security/site/migrate/current/3-to-4/html5/migrate-3-to-4-xml.html#m3to4-xmlnamespace-csrf)发现，Spring Security4开始默认启用Csrf。由于之前项目并不支持Csrf，导致权限框架把请求拦截下来。给项目配置禁用Csrf就可以了。`<csrf disabled="true"/>`

```XML
<http>
    ...
    <csrf disabled="true"/>
</http>
```