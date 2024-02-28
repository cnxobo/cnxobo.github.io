---
title: High Sierra升级后记 eclipse 菜单变灰
tags:
  - 日志
id: '604'
categories:
  - - macOS
date: 2017-09-28 16:48:31
---

#### eclipse菜单都变成灰色的

我用英文的系统语言没问题，同事使用中文系统语言出现 eclipse 菜单全是灰色的，没办法使用的问题。

##### 方法一:

修改 Eclipse.app/Contents/Info.plist 文件  
1\. 把 CFBundleDevelopmentRegion 对应的 value 值由 English 改为 en。  
2\. 把 CFBundleLocalizations 对应的 array 里元素除了 en 其它都删掉。

详情参考 [eclipse.org](http://t.cn/R0OZgUU)

##### 方法二:

在eclipse.ini文件中的-product后加入如下内容（注意要换行）如：

```-product
org.eclipse.epp.package.jee.product
-nl
en_US
--launcher.defaultAction
```

或  
configuration\\config.ini文件添加：`osgi.nl=en_US`  
详情参考 [max.cn](http://t.cn/R0OwYhJ)

#### telnet 被移除

使用 brew 安装 telnet.

```
brew tap theeternalsw0rd/telnet                                                                                                                                                                                                                   
brew install telnet
```