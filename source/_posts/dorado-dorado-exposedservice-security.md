---
title: dorado 接口安全风险
tags: []
id: '969'
categories:
  - - dorado
comments: false
date: 2022-07-31 15:59:29
---

dorado 封装了Ajax请求，导致在权限处理上与传统Web项目还是有很大的区别。dorado下，暴露一个服务可以通过`@Expose`、`@DataProvider`、`@DataResolver`等注解。 比如，我想提供一个接口，参数为`name`, 返回值为`"Hello " + name`, 我只需要写一个Java方法，同时把类注册到Spring上下文，并在方法上标注`@Expose`。

```Java
@Service
public class DemoController {
  @Expose
  public String hello(String name) {
    return "Hello " + name;
  }
}
```

像这样的一个接口，在任何一个dorado页面都可以通过构建一个AjaxAction，并配置service为beanId#方法名的形式 `demoController#hello`， 就可以调用这个方法。

```JavaScript
new dorado.widget.AjaxAction({
    service: "demoController#hello",
    parameter: "xobo"
}).execute(function(result) {
    console.log(result)
})
```

dorado封装了HTTP请求，所有的请求统一发给URL `/dorado/view-service`，然后由报文内容决定调用哪一个接口。 dorado 的这一特性导致，无法通过拦截URL的方式管理接口的权限，只能通过AOP的方式去拦截方法从而实现接口的管理。每个页面都会提供大量的接口，不同的页面可能会使用相同的接口。会导致接口权限的配置变得特别复杂。 从简化配置的角度，可以通过解析dorado view建立dorado接口与页面的映射，通过判断是否有权限访问接口所在的页面来判断能否访问接口。然后对需要更细粒度管理的接口做一个额外的配置。 基于此思路开发了 dorado exposedservice security https://github.com/cnxobo/dorado-exposedservice-security.git ，以解决dorado接口安全问题。