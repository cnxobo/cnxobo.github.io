---
title: Windows7 安装 IE10
tags:
  - IE
  - Windows7
id: '954'
categories:
  - - ops
comments: false
date: 2022-05-25 18:40:59
---

虽然现在已经2022年，Windows7已经停止安全更新两年了。但是在跟银行或者政府打交道时，IE10有时候又是一个唯一的选择。

## 微软官方虚拟机

在低频使用的情况下微软官方虚拟机是最好的选择，而且微软官方还提供了多种类型的虚拟机供使用，不用担心病毒木马后门什么的。不过这些虚拟机会在90天后过期，**使用之前做一个快照**，等到期之后再恢复一下就好了。 支持的虚拟机类型： - VirtualBox - Vagrant - VMware (Windows, Mac) - HyperV (Windows) - Parallels (Mac) 不仅有IE10还有更多的其它IE与Windows的组合： - IE8 on Win7 (x86) - IE9 on Win7 (x86) - IE10 on Win7 (x86) - IE11 on Win7 (x86) - IE11 on Win81 (x86) - MSEdge on Win10 (x64) Stable 1809 下载地址：[Virtual Machines - Microsoft Edge Developer](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)

## 离线安装IE10

由于各种原因不希望使用微软官方虚拟机，也可以自行安装IE10。

### 前置条件

要保证Windows7已经安装了一下补丁： - [Windows 7 Service Pack 1](https://www.catalog.update.microsoft.com/Search.aspx?q=KB976932) - [KB2729094](https://support.microsoft.com/kb/2729094) 现提供对 Windows 7 和 Windows Server 2008 R2 中 Segoe UI 符号字体的更新 - [KB2731771](https://support.microsoft.com/kb/2731771) 已推出针对 Windows 7 或 Windows Server 2008 R2 中的本地时间和 UTC 转换的新 API 更新 - [KB2670838](https://support.microsoft.com/kb/2670838) 有可用的 Windows 7 SP1 和 Windows Server 2008 R2 SP1 的平台更新

> 根据文档还需要一个补丁 [KB2533623](https://support.microsoft.com/kb/2533623) 但不知道为什么微软把这个补丁下载移除了。 测试下来没有这个补丁也没影响。

### 安装IE10

IE 10的安装包也被官方删除了。只能从网上收集三方来源的。 前置条件都安装好之后，就可以安装IE10呢。如果没有安装好会停留在“正在下载所需更新。。。” 很久。 安装找到系统对应的安装包，双击就好了。 链接: [https://pan.baidu.com/s/1MPe0VCOZHGYkg46G5Uww9g?pwd=iprg](https://pan.baidu.com/s/1MPe0VCOZHGYkg46G5Uww9g?pwd=iprg) 提取码: iprg