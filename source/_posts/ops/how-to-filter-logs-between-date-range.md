---
title: 抓取指定时间区间的日志
tags:
  - awk
  - grep
  - linux
  - sed
id: '890'
categories:
  - - ops
comments: false
date: 2021-10-09 22:07:29
---

服务器日志有时候比较大，需要抓取某一个时间区间的日志，比如抓取2021年10月1日，12点到12点半的日志。

### grep

```Shell
grep "^2021-10-01 12:[0-3]" nohup.log
```

通过grep过滤出以如下字符串开头的记录。

> 2021-10-01 12:0 2021-10-01 12:1 2021-10-01 12:2 2021-10-01 12:3

`^` 匹配行首，如果不加这个有可能匹配到日志内容 要注意的是： 1. grep是基于行过滤的，如果日志存在不是以时间戳开头的日志会被过滤掉。 2. 简单的时间区间正则表达式比较好写，如果时间区间比较复杂就麻烦，比如想查看2021年10月1日12:45到13:04日志。 这时候可以尝试使用sed

### sed

```Shell
sed -n '/^2021-10-01 12:45/,/^2021-10-01 13:04/p' nohup.log
```

`-n` sed不打印它读入的数据 `/^2021-10-01 12:45/,/^2021-10-01 13:04/p` '/pattern1/,/pattern2/' 是sed的区域地址 `p`是打印匹配到的内容。 要注意的是： 1. 区域地址是匹配符合pattern1的第一行到符合pattern2的第一行区域的记录。即使符合pattern2的日志有多条也只显示第一条。 2. 如果pattern1没匹配到记录那么区域地址为空。也就是说在pattern1时间点必须有日志，如果没有日志匹配不到任何日志了。 3. 如果pattern2没匹配到那么返回pattern1到文件末尾的日志。也就是说在pattern2时间点没有日志，那么会返回从pattern1开始一直到日志文件结束的日志记录。

### awk

```Shell
awk '$1=="2021-10-01" && $2>="12:45" && $2<"13:05"' nohup.log　
```

其中单引号中的被大括号括着的就是awk的语句，注意，其只能被单引号包含。 其中的$1..$n表示第几例。注：$0表示整个行。这里我取的第二列。 要注意的是： 1. awk是基于行过滤的，如果日志存在不是以时间戳开头的日志同样会被过滤掉。

## 性能差异

5G日志，每次执行命令前使用`sync; echo 1 > /proc/sys/vm/drop_caches`清理cache. grep、sed、awk无明显性能差异，耗时均在38s.

```Shell
sync; echo 1 > /proc/sys/vm/drop_caches
time grep "2021-10-01 12:[0-3]" nohup.log  > /dev/null
```

real 0m38.968s user 0m2.579s sys 0m2.126s

```Shell
sync; echo 1 > /proc/sys/vm/drop_caches
time sed -n '/^2021-10-01 12:00/,/^2021-10-01 12:30/p' nohup.log > /dev/null
```

real 0m38.865s user 0m4.547s sys 0m2.354s

```Shell
sync; echo 1 > /proc/sys/vm/drop_caches
time awk '$1=="2021-10-01" && $2>="12:00" && $2<="12:30"' nohup.log > /dev/null
```

real 0m38.718s user 0m9.138s sys 0m2.209s