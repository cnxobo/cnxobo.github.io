---
title: Ingress the account xxx@gmail cannot be used
tags: []
id: '356'
categories:
  - - Android
date: 2013-11-11 11:31:55
---

手机科学联网，使用wifi时正常，使用3G的时候就会提示“the account xxx@gmail.com cannot be used”. Ingress运行时，会使用Google账号同步信息，所以我们要保证Google账号同步可用。 我这个情况就是因为流量防火墙禁止了"Google Play服务”在3G下访问网络。 启用之后游戏可以正常运行了。 地址：https://groups.google.com/forum/#!topic/ingress-discuss/E3S6jVMkoTk 源文： We suspect this error occurs for two reasons: 1. \[Specified already above\] You're not actively synced to your Google account at the time you launch the app. Visit Menu > Settings > Accounts & sync and make sure your Google account is syncing successfully. You can usually test that sync is on and working by signing into the Google Talk app. 2. A network- or carrier-level issue. For #2, I'd recommend contacting your carrier for troubleshooting/resolution. -Bernini