---
title: dorado unkown package
tags:
  - dorado
categories: dorado
date: 2024-01-18
---

正常部署的dorado项目在启动一段时间之后访问会报`unkown package [widget]`。因为dorado在启动时会把资源文件解压到临时目录里，可能是资源文件被清理掉导致的错误。

可以通过属性`core.tempDir`指定临时文件目录。
>core.tempDir=./.dorado

或者关闭临时文件加速`core.supportsTempFile=false`。

#### 参考源码
* com.bstek.dorado.web.loader.DoradoLoader.preload(boolean)
* com.bstek.dorado.view.resolver.BootPackagesResolver.PackageResource.PackageResource(PackagesConfig, String, int, Locale)