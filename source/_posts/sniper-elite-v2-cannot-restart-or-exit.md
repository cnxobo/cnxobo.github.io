---
title: 狙击精英V2 无法重新开始 无法正常退出
tags: []
id: '193'
categories:
  - - Something
date: 2012-08-24 19:32:11
---

在找win8 msdn镜像下载的时候找到了异次元的分身，低调异次元，这个网站太低调了。 随便下了狙击精英V2这个游戏，效果没啥说的，由于平时不怎么玩儿游戏的，个人对这个游戏还是很满意的。有些时候会在子弹射出的时候，视角跟着子弹飞出去，直击目标。击中目标之后，会显示出子弹的如何破坏人体内器官，用异次元的描述就是“慢镜的X射线细节，呈现真实的破坏效果”。由于是最简单的模式，游戏还是很轻松的。 美中不足的就是游戏中无法重新开始(restart)及退出(exit)。我已经是纯英文目录了，游戏里运行的也不卡，应该不是硬件问题了。在网上搜搜，发现有人跟我问题一样，但是没有人回答如何处理。最后在(BT之家)http://www.btbbt.com/这个网站的游戏下载的地方找到了一个解决方案。它当时给出的是一个压缩包。下载的时候这个网站竟然没有要求我登录，对该网站好感倍加。压缩包里是个一批处理，要解压到安装跟目录中运行。 批处理如下： `cd GUIMenu copy a global.gui cd.. del /F /Q SE.V2.fix.bat` 先找到游戏安装目录下的GUIMenu目录，把“a”文件复制一份，然后把已有的global.gui覆盖掉。为了保险，我把先global.gui重命名为global.gui.bak.再复制一份a文件出来，再重命名。 如此简单，试了一下，可用。 PS: 1. 游戏的下载地址：[http://www.playnext.cn/sniper-elite-v2.html](http://www.playnext.cn/sniper-elite-v2.html "http://www.playnext.cn/sniper-elite-v2.html")。以表示对低调异次元的感谢。 2. 点着ctrl键拖动文件可以快速复制文件。 3. 文件a也就是global.gui下载，下载地址：[http://pan.baidu.com/s/1jGvIEea](http://pan.baidu.com/s/1jGvIEea "百度网盘") 4. 如果本地没有global.gui文件,也所谓,因为我们要使用文件a去覆盖它，文件a的下载地址见上方。如果也没有a文件,那么通过[这里](http://www.kuaipan.cn/file/id_18429588703046879.htm "global.gui")直接下载我重命名过的global.gui文件.