---
title: 查看Linux发行版
tags:
  - linux
id: '886'
categories:
  - - ops
comments: false
date: 2021-09-28 22:44:02
---

## TL;DR

```Shell
cat /etc/*-release
lsb_release -a
hostnamectl
```

我们在讲`Linux`的时候，我们通常是在说`Linux 发行版`. 严格来说，`Linux`只是一个内核, 操作系统最核心的部分,是软件应用与硬件的桥梁。一个Linux发行版是由Linux内核、GNU工具及库和软件集合构成。通常Linux发行版包括桌面环境、包管理系统还有一系列的预安装的软件。 比较流行的Linux发行版有: Debian、Red Hat、Ubuntu、Arch Linux、Fedora、CentOS、Kali Linux、OpenSUSE、Linux Mint、Alpine等等。 当你第一次登入一个Linux系统时，最好在做事之前先看看系统的Linux的版本。至少知道了是什么发行版，我们才知道应该用什么包管理工具。 以下就是演示如何通过命令行来获取Linux发行版及版本信息。

### /etc/os-release 文件

```Shell
cat /etc/os-release
```

输出示例:

```
NAME="Ubuntu"
VERSION="20.04.1 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.1 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

### lsb\_release 命令

```Shell
lsb_release -a
```

输出示例:

```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.1 LTS
Release:        20.04
Codename:       focal
```

`/etc/lsb_release` 文件是一些Linux发行版为了兼容旧程序而创建的。lsb是Linux Standard Base的缩写, 它叫做Linux基础标准，但实际它并不是标准。只有部分发行版在使用它。 然而`/etc/os-release` 文件是标准，一种基于systemd的标准。任何一个基于systemd的Linux发行版都有这个文件。 所以CentOS6上有`/etc/lsb_release`, CentOS7上有`/etc/os-release`，所以在不确定是什么系统时可以直接

```Shell
cat /etc/*-release
```

如果使用了什么小众的发行版,上面的命令都不好用，还可以尝试这个命令:

```Shell
echo /etc/*_ver* /etc/*-rel*; cat /etc/*_ver* /etc/*-rel*
```

[release-files](http://linuxmafia.com/faq/Admin/release-files.html)

### hostnamectl 命令

```Shell
hostnamectl
```

输出示例:

```
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 7c5b5acceb4844b1be6d38ffe8701c30
           Boot ID: 8e3a6b1d077e46759b094f0c610222a2
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1062.el7.x86_64
      Architecture: x86-64
```

参考链接: https://linuxize.com/post/how-to-check-linux-version/ https://stackoverflow.com/questions/47838800/etc-lsb-release-vs-etc-os-release