---
title: HTTP keep-alive
tags:
  - keepalive
  - nginx
  - tcp/ip
id: '899'
categories:
  - - ops
comments: false
date: 2021-10-18 21:41:58
---

## 什么是HTTP keep-alive

HTTP keep-alive, 准确的说是HTTP持久连接(HTTP persistent connection),是使用同一个TCP连接来发送和接收多个HTTP请求和响应。 持久连接的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销。 在 HTTP/1.1 中，所有的连接默认都是持久连接，但在 HTTP/1.0 内并未标准化。 对于支持keep-alive的HTTP服务器，回应的HTTP头里:

```HTTP
Connection: keep-alive
```

当然我们并不希望这个期限是永远，需要给他加一个期限, 服务端在回应的HTTP头里加入`Keep-Alive`，单位是秒。

```HTTP
Connection: keep-alive
Keep-Alive: timeout=60
```

## 在Nginx下配置keep-alive

做为HTTP Server Nginx默认已经支持keep-alive。 但是根据实际需要，可以调整以下参数:  
keepalive\_disable

> 针对某些浏览器禁用`keep-alive`功能，可选的值有: msie6, safari等等。

keepalive\_requests

> 一个TCP连接可以处理的HTTP请求的数量限制，达到限制就关闭连接。默认值1000 (从版本1.19.10开始，默认值调整为100)

keepalive\_time

> 一个TCP连接的最长存活时间，默认1小时(从版本1.19.10开始新增的参数)

keepalive\_timeout

> `keep-alive`的超时时间，一个TCP连接。

### keepalive\_timeout

keepalive\_timeout 有两个参数：第一个参数是服务端的TCP连接的超时时间，如果值是0则关闭`Keep-Alive`功能；第二个参数是可选的，是在控制响应头里的`Keep-Alive: timeout=time`的值,这两个时间值可以不一样。建议在设置时第一值略大于第二个。

> Syntax: keepalive\_timeout timeout \[header\_timeout\]; Default:  
> keepalive\_timeout 75s; Context: http, server, location

```Nginx
keepalive_timeout 75s 60s
```

[Module ngx\_http\_core\_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_disable)