---
title: 在线更新 dorado 规则RequestRejectedException
tags: []
id: '797'
categories:
  - - dorado
  - - pitfall
date: 2020-04-05 17:41:56
---

使用在线更新 dorado 规则时，项目报错`org.springframework.security.web.firewall.RequestRejectedException: The request was rejected because the URL contained a potentially malicious String "//"`. 这个是由Spring Security 5提供的一个HTTP防火墙，拦截可疑访问导致的。在项目中注册如下的bean，即可替换系统默认的防火墙。该实现没有任何防范作用，建议仅仅再更新Dorado规则时临时打开，一定不能发布到生产环境。

```Java
 @Bean
  public HttpFirewall looseHttpFirewall() {
    StrictHttpFirewall firewall = new StrictHttpFirewall();
    firewall.getEncodedUrlBlacklist().clear();
    return firewall;
  }
```