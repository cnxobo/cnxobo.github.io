---
title: 不听话的Ubuntu
tags:
  - gnome-shell
  - mysql
  - root
  - ubuntu
id: '105'
categories:
  - - Something
date: 2011-11-09 21:42:19
---

换了新电脑，arch linux下，Gnome死活不启动。只好用Ubuntu了。最新版11.10。

## 折腾

### 1\. 习惯了Gnome Shell.

安装gnome-shell

> sudo apt-get install gnome-shell #gnome-shell界面 sudo apt-get install gnome-session-fallback #gnome传统界面

首先要系统设置用户帐户里启用自动登录 默认启动gnome shell

> sudo /usr/lib/lightdm/lightdm-set-defaults -s gnome-shell

改回unity

> sudo /usr/lib/lightdm/lightdm-set-defaults -s ubuntu

更多的问题可以来这里 [本帖集中了11.10中多次出现的各种问题，和解决方法。](http://forum.ubuntu.com.cn/viewtopic.php?f=120&t=349186&sid=3df09530be5e5c73681f3f53716959bb)

### 2\. mysql-workbench无法启动，卡在启动画面

好像是当前的mysql-workbench跟ubuntu 11.10不兼容引起的，要打上补丁重新编译一下。 更多信息参考[http://mysqlworkbench.org/?p=1217](http://mysqlworkbench.org/?p=1217)。 我找到一个64bit的，直接安装就可以了。 下载地址：[Dropbox](http://dl.dropbox.com/u/773778/mysql-workbench-gpl_5.2.35-2_amd64.deb "Dropbox") [115网盘](http://115.com/file/clflb3or#mysql-workbench-gpl_5.2.35-2_amd64.deb "115网盘")

### 3\. 一进ubuntu，风扇狂转，发热变大

查资料发现是因为两显卡都在工作原因。关闭不在使用的显卡就好了。不过要先换root帐户，sudo好像都不好使。

> echo OFF > /sys/kernel/debug/vgaswitcheroo/switch

原理待补充。 可以期待xobo版的双显卡切换。 参考： [Linux下双显卡切换](http://blog.csdn.net/geyongqi/article/details/6691360%20 "csdn")  [双显卡笔记本ubuntu下禁用独显](http://zhyu.me/linux/ubuntu-disabled-independent-graphics-card.html "zhyu") [HybridGraphics](https://help.ubuntu.com/community/HybridGraphics "ubuntu")

### 4\. 接上个问题，如何启用root帐户

> sudo passwd root

此命令将会重新设置 root 的密码，按照提示输入新的密码，并加以确认。之后，重启系统时，就可以用 root 登录了。 如果你想要禁用 root 帐号，则执行下列命令：

> sudo passwd -l root

引自 [linuxtoy.org](http://linuxtoy.org/archives/howto_enable_ubuntu_root_account.html "linuxtoy")