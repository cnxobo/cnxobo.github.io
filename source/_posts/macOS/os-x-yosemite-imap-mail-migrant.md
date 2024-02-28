---
title: OS X Yosemite IMAP邮箱迁移小记
tags:
  - imap
  - os x
  - Time Machine
id: '431'
categories:
  - - Something
date: 2015-05-07 22:57:53
---

操作系统：Yosemite 10.10.3 邮件客户端：系统自带的Mail Version 8.2 (2098) 配置好所有的邮件帐号，在Mail里，选中任一Mailboxes里选中邮件都可以随意的Move To或者Copy To其它Mailboxes。 如果想迁移邮件的话，只需要把旧的邮箱里的邮件Copy To新邮箱的对应位置就好了。Mail客户端会通过IMAP协议自动的把邮件上传到新的邮件服务器。Nice & Easy。 注意事项： **千万不要直接把旧帐号直接改成新帐号。** **千万不要直接把旧帐号直接改成新帐号。** **千万不要直接把旧帐号直接改成新帐号。** 我就是这么干的，Mail愉快地把我旧账号的千余封邮件删除了然后换成了新账号的的几封邮件。 如果你也这么干了，用Time Machine恢复 ~/Library/Mail 目录就可以了。