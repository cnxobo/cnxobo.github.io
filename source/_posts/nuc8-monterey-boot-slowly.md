---
title: NUC8 黑果Monterey启动特别慢
tags:
  - Hackintosh
  - macOS
id: '967'
categories:
  - - macOS
comments: false
date: 2022-07-27 21:43:33
---

# NUC8 黑果Monterey启动特别慢

三星的某些型号的SSD，比如我的970 EVO plus，执行TRIM的操作特别的慢。而在APFS上，如果启用了TRIM，macOS会在启动的时候执行一次TRIM操作释放未使用的空间，就会导致启动速度特别的慢。 可以通过升级OpenCore到 0.7.9 及以上版本，然后设置`SetApfsTrimTimeout`值为0（默认为-1）关闭启动时的TRIM操作以提升启动速度，我的是从启动时间5分钟提升到20s。

> macOS 12.0及以上版本 `SetApfsTrimTimeout`超时功能失效只有0禁用，及其值开启。

## macOS的APFS

macOS的APFS针对SSD做了特别的优化，它会自动管理和优化SSD的存储空间，从而减少了对TRIM的依赖。所以macOS也就没有提供手动的命令来触发TRIM，只有系统启动或者发现新的设备连接的时候才会触发一次TRIM。 如果系统关机启动比较频繁的话，或者系统更新时需要频繁重启，建议`SetApfsTrimTimeout`设置为0，关闭开机时TRIM，以加速开机时间。如果常年不关机，可以保持默认值-1。

## 如何修改OpenCore

### Shell

OpenCore配置是写在EFI分区的。所以要先看看EFI分区在哪里。 1. 找到EFI分区的IDENTIFIER。

```Shell
diskutil list
```

会有一个类似下面的输出:

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk2         1000.0 GB  disk0s2
```

在这个例子里，EFI分区的IDENTIFIER是disk0s1。

2.  挂载EFI分区

```Shell
sudo diskutil mount disk0s1
```

3.  修改配置 配置文件位于`/Volumes/efi/EFI/OC/config.plist`。 搜索`SetApfsTrimTimeout`找到如下位置

```
<key>SetApfsTrimTimeout</key>
<integer>-1</integer>
```

把其中的-1改为0, 保存编辑器并退出。

4.  卸载之前挂载的EFI

```
diskutil umount /dev/disk0s1
```

也可以通过 Hackintool 软件来挂载EFI分区。具体的操作步骤可以参考这个 NUC8黑苹果更新OpenCore引导过程记录 https://zhuanlan.zhihu.com/p/431713988

## 其他信息

我更新的OpenCore是从这里下载的 OpenCore for NUC8i5BEH \[https://github.com/Jiangmenghao/NUC8i5BEH\](https://github.com/Jiangmenghao/NUC8i5BEH target="\_blank")

## reference

OpenCore Configuration https://dortania.github.io/docs/latest/Configuration.html SSD hall of fame needs new entries · Issue #192 · dortania/bugtracker https://github.com/dortania/bugtracker/issues/192