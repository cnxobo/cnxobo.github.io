---
title: MacOS 微信获取视频号视频地址
tags:
  - macOS
  - 视频号
id: '957'
categories:
  - - macOS
comments: false
date: 2022-06-26 10:28:47
---

> 原理参考自[WeChatDownloader](https://github.com/xuncv/WeChatDownloader)，基于内存搜索实现的视频号的视频地址获取。Windows也可以去这里下载工具。

在视频号播放视频的时候，在微信的内存中搜索`https://finder.video.qq.com/251/20302/stodownload?encfilekey=` 找到的URL大概率就是视频的地址呢。 MacOS可以使用免费开源的[Bit Slicer](https://github.com/zorgiepoo/Bit-Slicer) ，类似金山游侠用来查看和修改内存的。 先选择进程`WeChat`, 然后类型选`16-bit String`, 符号保持 equals 不变，然后搜索框内输入https://finder.video.qq.com/251/20302/stodownload?encfilekey= ，视频号开始播放视频之后，点回车开始搜索。 ![bitslicer-wechat.png](https://s2.loli.net/2022/06/16/hq3iuyJP6V5Qs89.png) 很快就会搜索出来所有包含上述URL的内存。在第一条上点右键，选择Copy，然后打开一个文本编辑器，粘贴，会得到类似如下的内容, 其中从 https开始到 taskid=0就是视频的URL了，复制一下去浏览器打开试试。如果第一条不是，可以试试其它的。

> TCMalloc, rw- 0x12D08A214 https://finder.video.qq.com/251/20302/stodownload?encfilekey=Cvvj5Ix3eewK0tHtibORqcsqchXNh0Gf3sJcaYqC2rQDvjcweUjy5QItPF1I1icTWMxBdcY8vqvS5YJiaNiao2e7Zu0NlibsWqcgtYlNJjJS9EYNxXSFVVBDpnUkaYUYCs8tG&token=AxricY7RBHdXticrYWAcrAvaXDczqzOBqdibg2ueNM5zDjo7uVRklgO4HdEFjHunibSkYbOEAm1ic5kc&idx=1&adaptivelytrans=0&bizid=1023&dotrans=2991&hy=SH&m=&X-snsvideoflag=xV3&taskid=0杴慥㵲⸱〰〰〰