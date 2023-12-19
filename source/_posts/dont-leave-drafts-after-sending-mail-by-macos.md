---
title: 解决Mail中发完邮件，会在草稿里留一份
tags:
  - imap
  - macOS
  - Mail
  - OSX
id: '497'
categories:
  - - macOS
date: 2016-12-21 10:18:35
---

使用macOS自带的Mail写邮件时，总会有一份草稿保留到草稿箱，就算邮件发送成功它依旧在那里。当然我们可以手动删掉，但是有一个更好的解决方法。 这个问题是由于IMAP账号导致的，所以要对所有的IMAP做相同的操作。 打开Mail的偏好设置(⌘ + ,)切换至"Accounts"->"Mailbox Behaviors"。 OS 10.12 Sierra 修改Drafts Mailbox为"On My Mac" -> "Drafts". ![](http://xobo.org/wp-content/uploads/2016/12/65526839-300x204.png) ![](http://xobo.org/wp-content/uploads/2016/12/65543473-300x216.png) 其它版本 取消在服务器端保存草稿 ![](http://xobo.org/wp-content/uploads/2016/12/65609991-291x300.png) 参考链接: http://royalwise.com/do-you-have-drafts-left-in-your-folder-even-after-sending/