---
title: 卸载千月(BlueSoleil)后,win7 无线网络图标变成红叉叉
tags: []
id: '210'
categories:
  - - Something
date: 2012-09-09 00:57:59
---

蓝牙管理软件本身就不多, windows自身功能又弱, 也就无疑问的尝试了一下千月(BlueSoleil). 但是在一次偶然的卸载操作,千月(BlueSoleil)给我带来了无尽的痛苦--无线网络图标变成红叉叉. 对于功能来说没有任何影响但是就看着那个叉叉不爽, 一定要把它处理掉, 系统还原又被我关掉了. 只能硬处理了. 本以为我卸载了,它出这个叉叉,我再装回去就好了,装回去也不成. 问百度,知道上有人说是重建一下图标缓存.依旧不成. 中文无解去google.com/ncr搜英文,说什么删除设备管理器中的隐藏驱动,依旧无解. ... 经无数次尝试均不成, 最后只好重装系统了. 那差不多是年前的事儿了,今天我又手贱的安装了千月(BlueSoleil)又卸载了,红叉叉又出现了,系统还原还是没有开启. 这次发现在千月有用户提出同样的问题,千月的客服回答说这个跟蓝牙个人局域网服务有关, 他们测试的时候没有问题.虽然对问题没有帮助,至少了解到是因为跟什么东西有关了. 继续问Google, 有人说删除驱动就可以了,"将非本地网卡、网线网卡的适配器全部右键卸载掉，可能包括蓝牙、1394、虚拟网卡都逐一卸载掉".蓝牙的驱动,本地网卡驱动,无线网卡驱动,我早不知删除重装多少遍了.但是虚拟网上的驱动我一直没有动过, 总不会跟这个有关吧. 我本机装有virtualbox及vmware两个虚拟机, 用试试看的方法, 先卸载了virtualbox的网卡驱动. 无线图标闪了闪变了回正常的了.终于好了,再把virtualbox的虚拟网卡安装进来,图标依然正常.重启电脑,也没有复发.原来这个跟虚拟网卡有关. 总结: 1. 永远不要关系统盘的系统还原 2. 尝试卸载虚拟网卡 猜测: 中招的是不是都是virtualBox用户