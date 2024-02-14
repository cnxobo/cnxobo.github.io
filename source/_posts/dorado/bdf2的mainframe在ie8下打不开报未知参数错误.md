---
title: BDF2的MainFrame在IE8下打不开报未知参数错误
tags:
  - bdf2
  - dorado
  - ie8
  - mainframe
id: '415'
categories:
  - - dorado
date: 2015-03-19 17:58:19
---

BDF2的MainFrame在IE8下打不开，提示“未知参数”错误。 打开“开发人员工具”也只是得到“未知参数”，在Chrome却是正常的。 最后排除发现是因为用户配置的菜单图标里存在 “url(>skin>common/icons.gif) -NaNpx -NaNpx”。 Chrome下可以将该错误图标抛弃,而IE8就拒绝。