---
title: Arch Linux 使用小结
tags:
  - arch linux
  - flash
  - 关闭独显
  - 字体
id: '109'
categories:
  - - Cheat sheet
date: 2011-11-13 11:40:38
---

### Flash中文字体

Arch Linux中Flash中文字体显示看不清，模糊，总之不正常就是了 方法一，修改/etc/fonts/conf.d/49-sansserif.conf，把里面的sans-serif改成sans 方法二，pacman -S ttf-fireflysung （推荐） PS：我尝试过修改配置文件但是没有效果，不过安装好ttf-fireflysung的时候，网址一刷新就好了。又把配置文件还原，字体依旧正常。 [原贴地址](http://lilacblog.appspot.com/2011/10/30/archlinux-flash-font.html "不一定能打开")

### 关闭独显

BIOS竟然不支持关闭显卡，不经常玩游戏的我，真是后悔买这样的本本。 原本想跟Ubuntu一样解决呢，结果 /sys/kernel/debug/ 目录下是空的。 原来是因为没有加载debugfs。Mount上去就好了。

> \# mount -t debugfs debugfs /sys/kernel/debug

让他自动加载就更好了/etc/fstab:

> debugfs /sys/kernel/debug debugfs 0 0

参考：[https://wiki.archlinux.org/index.php/Acer\_TimelineX#Switchable\_Graphics](https://wiki.archlinux.org/index.php/Acer_TimelineX#Switchable_Graphics "太喜欢arch linux")