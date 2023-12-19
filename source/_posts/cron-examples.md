---
title: cron使用示例
tags: []
id: '814'
categories:
  - - ops
date: 2020-12-14 19:58:08
---

后台自动运行维护任务对Linux系统管理员是非常重要的。Linux Cron工具是一个有效的方式去安排后台定时任务。

### Linux Crontab格式

```
MIN HOUR DOM MON DOW CMD
```

字段

描述

值

MIN

分钟

0 to 59

HOUR

小时

0 to 23

DOM

天

1-31

MON

月

1-12

DOW

周

0-6

CMD

命令

要执行的命令

### 指定时间执行任务

cron的最基本用法就是如下所示的在指定时间运行任务。这将在6月10上午8点30执行完整备份的脚本。 请注意时间字段使用24小时制。因此早上8点就是8，晚上8点是20. 30 08 10 06 \* /home/ramesh/full-backup 30 – 30分 08 – 上午8点 10 – 10号 06 – 6月 \* – 周的每天

### 指定任务执行多次(比如一天执行两次)

如下的脚本每天两次的执行一个增量的备份 这个示例每天11:00和16:00执行指定的增量备份脚本。如果某个时间段需要执行多个，那么使用逗号分割，那么每个时间都会执行。 00 11,16 \* \* \* /home/ramesh/bin/incremental-backup 00 – 0分 11,16 – 11:00和16:00 \* – 每天 \* – 每月 \* – 周的每天

### 指定任务在某个时间区域执行

如果你希望任务在指定的时间段每小时执行一次可以使用下面的配置。

#### 每天的工作时间

这个例子在每天(包括周六周日)的早上9点到下午6点检查数据库状态。 00 09-18 \* \* \* /home/ramesh/bin/check-db-status 00 – 0分 09-18 – 9, 10, 11, 12, 13, 14, 15, 16, 17, 18 \* – 每天 \* – 每月 \* – 周的每天

#### 工作日的工作时间

这个例子在每天(不包括周六周日)的早上9点到下午6点检查数据库状态。 00 09-18 \* \* 1-5 /home/ramesh/bin/check-db-status 00 – 0分 09-18 – 9, 10, 11, 12, 13, 14, 15, 16, 17, 18 \* – 每天 \* – 每月 1-5 – 周一到周五

### 如何查看 crontab 条目

#### 查看当前登录用户的 crontab 条目

使用 `crontab -l` 去查看当前账号的 crontab 条目

#### 查看其它用户的 crontab 条目

要查看其它用户的 crontab 条目，需要有 root 权限然后使用 -u {username} -l

### 如何编辑 Crontab 条目

#### 编辑当前用户的 crontab 条目

使用 `crontab -e` 去编辑 crontab 条目，默认情况下会编辑当前登录用户的 crontab.

```
ramesh@dev-db$ crontab -e
@yearly /home/ramesh/centos/bin/annual-maintenance
*/10 * * * * /home/ramesh/debian/bin/check-disk-space
~
"/tmp/crontab.XXXXyjWkHw" 2L, 83C

[Note: This will open the crontab file in Vim editor for editing.
Please note cron created a temporary /tmp/crontab.XX... ]
```

当你使用`:wq`保存上面的临时文件后，它会保存 crontab 并显示下面的信息表明 crontab 已经成功地修改了。

```
~
"crontab.XXXXyjWkHw" 2L, 83C written
crontab: installing new crontab
```

#### 修改 root 用户的Crontab 条目

先使用 root 用户登录(su - root) 然后再使用 `crontab -e` 去编辑.

```Shell
root@dev-db# crontab -e
```

#### 修改其它用户的 Crontab 条目

要修改其它用户的 crontab 条目，需要有 root 权限然后使用 -u {username} -l

```
root@dev-db# crontab -u sathiya -e
@monthly /home/sathiya/fedora/bin/monthly-backup
00 09-18 * * * /home/sathiya/ubuntu/bin/check-db-status
~
~
~
"/tmp/crontab.XXXXyjWkHw" 2L, 83C
```

### 每分钟执行一次任务

正常情况下，你可能不需要一个每分钟都执行一次的任务。但是理解这个例子将会帮助你理解后面的例子。

```
* * * * * CMD
```

星号 \* 表示所有有可能的值，每一分钟、每一小时、每一天。除了直接使用星号，你还可以使用以下非常有用的场景: 当你在分钟字段指定 \*/5 意味着每5分钟 当你在分钟字段指定 0-10/2 意味着在前10分钟里，每两分钟 以上两种写法，在其它字段也是适用的。

### 每10分钟执行一次任务

如果你想每10分钟检查一次硬盘空间状态，可以使用下面的配置 \`\`\`\*/10 \* \* \* \* /home/ramesh/check-disk-space\`\`\` 除了配置这五个字段，我们也可以指定一个关键字。 有几个特殊的场景来替代这五个字段，你可以使用@+关键字例如 reboot,midnight, yearly, hourly. 关键字清单

关键字

等同于

@yearly

0 0 1 1 \*

@daily

0 0 \* \* \*

@hourly

0 \* \* \* \*

@reboot

开机后执行

### 使用 @year 在每年开始的时候执行任务

如果你希望一个任务在每年开始的时候执行，那么你就可以使用 @year 关键字。 这个将会执行系统年检通过年检脚本在每年的1月1号00:00. \`\`\`@yearly /home/ramesh/red-hat/bin/annual-maintenance\`\`\`

### 使用 @monthly 在每月开始的时候执行任务

如果你希望一个任务在每月开始的时候执行，那么你就可以使用 @monthly 关键字。 这个将会在每月的1号00:00执行备份脚本.

```
@monthly /home/ramesh/suse/bin/tape-backup
```

### 使用 @daily 在每天开始的时候执行任务

如果你希望一个任务在每天开始的时候执行，那么你就可以使用 @daily关键字。 这个将会在每月的1号00:00执行清理日志脚本. \`\`\`@daily /home/ramesh/arch-linux/bin/cleanup-logs "day started" <pre><code class="line-numbers"><br />### 使用 @reboot 在每次重启后执行任务 如果你希望一个任务在每次重启后执行，那么你就可以使用 @reboot 关键字。 这个将会在每次服务器重启后执行. \`\`\`@reboot CMD

### 如何通过MAILTO关键字禁用或重定向Crontab的邮件输出

默认情况下crontab会把任务的输出给配置定时任务的用户. 如果你希望重定向输出给一个指定的用户,只需要在crontab里添加或者修改 MAILTO 变量就可以了。

```
ramesh@dev-db$ crontab -l
MAILTO="ramesh"

@yearly /home/ramesh/annual-maintenance
*/10 * * * * /home/ramesh/check-disk-space

[Note: Crontab of the current logged in user with MAIL variable]
```

如果你不希望发送邮件，把这个变量置空就可以了.

```
MAILTO=""
```

### 如何每秒执行一次任务

你不能设置每秒执行的任务。 因为 cron 的可以配置的最小单位是分钟。在正常的业务场景下，也没有理由在系统里设置每秒都执行的任务。

### 在Crontab里设置PATH变量

上面所有的例子里我们都是指定了Linux命令或脚本的绝对路径。 如果你希望使用相对路径，那么就需要把在crontab 里把程序所在文件夹路径添加到PATH变量里。

```
ramesh@dev-db$ crontab -l

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/home/ramesh

@yearly annual-maintenance
*/10 * * * * check-disk-space

[Note: Crontab of the current logged in user with PATH variable]
```

### 从一个Cron文件里安装Crontab

除了直接编辑crontab 文件, 你也可以创建一个cron文件然后在里面添加需要的条目, 然后再把这个文件安装进 crontab。

```
ramesh@dev-db$ crontab -l
no crontab for ramesh

$ cat cron-file.txt
@yearly /home/ramesh/annual-maintenance
*/10 * * * * /home/ramesh/check-disk-space

ramesh@dev-db$ crontab cron-file.txt

ramesh@dev-db$ crontab -l
@yearly /home/ramesh/annual-maintenance
*/10 * * * * /home/ramesh/check-disk-space
```

安装cron-file.txt时，也会移除所有的旧cron条目, 所以在安装cron文件时，要特别小心。

### 每个月最后一天执行

cron原生不支持每个月的最后一天执行，但是可以通过判断明天是不是1号，来决定今天是不是这个月的最后一天。

```
55 23 28-31 * * [[ "$(date --date=tomorrow +\%d)" == "01" ]] && myjob.sh
```

### 每个月最后一个工作日执行

cron 法定节假日，每年会都变化，每个月的最后一个工作日也会不一样，需要能够动态的更新法定节假日。考虑到一年只有12个月也就是12个工作日期，我把每月工作日写入 lastworkingday.txt 然后对比当天是不是在这个文件内，如果在就执行。同时把 lastworkingday.txt 文件上传到 github 定期去摘取这个文件。如果法定节假日发生变化，我只需要提交对 lastworkingday.txt的修改就好了。

```
00 00 * * * git -C /opt/mycrontabdb pull
00 12 20-31 * * grep $(date +%F) 
/opt/mycrontabdb/lastworkingday.txt && myjob.sh
```

lastworkingday.txt sample

```
20201030
20201130
20201231
20210129
20210226
20210331
20210430
20210531
20210630
20210730
20210831
20210930
20211029
20211130
20211231
```