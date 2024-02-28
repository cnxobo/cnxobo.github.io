---
title: Nginx proxy_pass 的URL Mapping
tags:
  - if
  - nginx
  - proxy_pass
id: '846'
categories:
  - - ops
date: 2021-02-28 17:19:27
---

Nginx 从Web服务器起步，现在越来越多的承担起反向代理服务器和负载均衡器的角色。`proxy_pass` 也就成了很最常用的指令之一，但是`proxy_pass`中，请求URI与传递给后端服务器的URI的转化是很容易弄错的地方，本文通过各种示例展示这个URI的转化规则。

## 一个简单的例子

假设我们有一个前后分离的应用，前端静态文件存放在`/srv/www/myapp`目录，后端API访问地址是`http://127.0.0.1:8080`。后端提供API接口`/hello`,同时前端访问后端时，统一加`/api`前缀，那这个时候，我们就可以配置为:

```
location / {
    root /srv/www/myapp;
}
location /api/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

现在，就通过 `http://127.0.0.1` 访问到`/srv/www/myapp`里的静态文件(假设Nginx监听端口80)，访问`http://127.0.0.1/api/hello`时,请求都会被转发给`http://127.0.0.1:8080/hello`。 如果我们希望保留`/api/`路径那么只需要把`proxy_pass`配置的URI的path删掉就可以了。

```nginx
proxy_pass http://127.0.0.1:8080;
```

## URI的在与不在

**URI**在这里不是指完整的URL，而是proxy\_pass 指定的URL内服务器地址或端口(如果存在)后面的地址，也就是标准URL定义中的path。 URI = scheme:\[//\[userinfo@\]host\[:port\]\]**path**\[?query\]\[#fragment\] ![uri.png](https://i.loli.net/2021/09/08/BHewEaG589vqfO6.png) 例如:

> `proxy_pass http://127.0.0.1:8080/`,URI是`/`; `proxy_pass http://127.0.0.1:8080/api`, URI是`/api`; `proxy_pass http://127.0.0.1:8080`， URI不存在。 **请求URI**发起请求的URL内服务器地址或端口(如果存在)后面的地址。例如: `http://127.0.0.1/api/hello`, 请求URI是`/api/hello`。 `上游服务器`,被代理服务器，或者说是后端服务器。

### 指定了URI

1) 如果`proxy_pass`指令**指定了URI**，`请求URI`中匹配location的部分在传递给上游服务器时被`URI`替换掉。 Case 1

```
location /api/ {
    proxy_pass http://127.0.0.1:8080/;
}
```

请求`http://127.0.0.1/api/hello`, 上游服务器就会收到`http://127.0.0.1:8080/hello`。_(/api/ -> /)_ Case 2

```
location /api/ {
    proxy_pass http://127.0.0.1/remote/;
}
```

请求`http://127.0.0.1/api/hello`, 上游服务器就会收到`http://127.0.0.1:8080/remote/hello`。 _(/api/ -> /remote/)_ Case 3

```
location /api/ {
    proxy_pass http://127.0.0.1/remote;
}
```

请求`http://127.0.0.1/api/hello`, 上游服务器就会收到`http://127.0.0.1:8080/remotehello`。_(/api/ -> /remote)_

### 没有指定URI

2) 如果`proxy_pass`指令**没有指定URI**，那么请求URI就会原样的传递给上游服务器。

```
location /api/ {
    proxy_pass http://127.0.0.1:8080;
}
```

请求`/api/hello`, 上游服务器就会收到`/api/hello`。

### 结尾的`/`

location的值是否以`/`结尾，`proxy_pass`的值是否以`/`结尾可以组合出成N多种可能。但是不管怎么组合，只有一个会影响匹配规则: **URI的在与不在**。 location的值最好以`/`结尾。如果location的值如果没有以`/`结尾,例如`/api`,那么URL`/apixxx`也会匹配上。除非明确需要匹配这种`/apixxx`场景。从最小配置的原则上，建议location的值都是以`/`结尾。 proxy\_pass的URI，根据项目实际需要赋值或留空。如果有值，那么结尾通常都需要与location的值保持一致--要么都有`/`，要么都没有，不然会拼接出黏连在一起的路径(`/remotehello`)或者出现两个`/`的路径(`/api//hello`)。

### 更多的Case

Case #

Nginx location

proxy\_pass URL

Test URL

Path received

1

/api1

http://127.0.0.1:8080

/api1/abc/api

/api1/abc/api

2

/api2

http://127.0.0.1:8080/

/api2/abc/api

//abc/api

3

/api3/

http://127.0.0.1:8080

/api3/abc/api

/api3/abc/api

4

/api4/

http://127.0.0.1:8080/

/api4/abc/api

/abc/api

5

/api5

http://127.0.0.1:8080/app1

/api5/abc/api

/app1/abc/api

6

/api6

http://127.0.0.1:8080/app1/

/api6/abc/api

/app1//abc/api

7

/api7/

http://127.0.0.1:8080/app1

/api7/abc/api

/app1abc/api

8

/api8/

http://127.0.0.1:8080/app1/

/api8/abc/api

/app1/abc/api

9

/

http://127.0.0.1:8080

/api9/abc/api

/api9/abc/api

10

/

http://127.0.0.1:8080/

/api10/abc/api

/api10/abc/api

11

/

http://127.0.0.1:8080/app1

/api11/abc/api

/app1test11/abc/api

12

/

http://127.0.0.1:8080/app2/

/api12/abc/api

/app2/api12/abc/api

13

/api

http://127.0.0.1:8080

/api13/abc/api

/api13/abc/api

14

/api

http://127.0.0.1:8080/remote

/api14/abc/api

/remote14/abc/api

## 更复杂的路径映射

### 正则表达式

使用正则表达式可以很方便的重组请求URI。例如: 对外URI是`http://localhost:8080/api/cart/items/123`,而上游的API处理的格式是`http://localhost:5000/cart_api?items=123`。这个时候就可以使用正则表达式去捕获需要的部分，然后转化成希望的格式。

```
location ~ ^/api/cart/([a-z]*)/(.*)$ {
   proxy_pass http://127.0.0.1:5000/cart_api?$1=$2;
}
```

### rewrite

也可以在location里如果使用`rewrite`修改了URI:

```
location /name/ {
    rewrite    /name/([^/]+) /users?name=$1 break;
    proxy_pass http://127.0.0.1;
}
```

在这个场景下，指定的URI会被忽略，完整的修改后的URI会传递给被代理的服务器。

### 使用变量

也可以指定Nginx变量来

```
location /name/ {
    proxy_pass http://127.0.0.1$request_uri;
}
```

如果你想把服务器地址定义为变量的话，还需要额外指定一个`resolver`也就是DNS服务器，为了安全不建议使用公共的DNS例如:8.8.8.8, 223.5.5.5等等。

```
location /proxy {
    resolver 127.0.0.1;
    set $target http://proxytarget.example.com;
    proxy_pass $target;
}
```

## If 引发的bug

nginx < 1.7.9 有个bug。如果在`location`里使用`if`，会导致`proxy_pass`不按照预期工作。proxy\_pass 的`URI`叠加`request_uri`。 就像最开始的例子访问`http://127.0.0.1/api/hello`时,请求都会被转发给`http://127.0.0.1:8080//api/hello`。