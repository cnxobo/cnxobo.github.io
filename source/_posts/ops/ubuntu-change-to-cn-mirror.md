---
title: Ubuntu切换国内镜像(通用版)
tags: []
id: '823'
categories:
  - - ops
date: 2020-12-24 13:38:54
---

## 镜像模式

使用镜像模式，apt 命令会自动根据选择服务器所在的国家的镜像。如下脚本默认源使用镜像协议并备份原始文件`sources.list到`sources.list.backup\`

```Shell
sudo sed -E -i.backup 's#^(debdeb-src) ([^ ]*) (.*)#\1 mirror://mirrors.ubuntu.com/mirrors.txt \3#' /etc/apt/sources.list
```

## 手动模式

我的服务器阿里云机房，所以希望固定使用阿里云镜像。如下脚本修改默认源为阿里云并备份原始文件`sources.list`到`sources.list.backup`

```Shell
sudo sed -E -i.backup 's#^(debdeb-src) ([^ ]*) (.*)#\1 https://mirrors.aliyun.com/ubuntu \3#' sources.list
```

### 国内常用镜像源

根据自己需要，选择自己最快的源。

名称

地址

阿里镜像源

https://mirrors.aliyun.com/ubuntu

清华大学镜像源

https://mirrors.tuna.tsinghua.edu.cn/ubuntu/

网易镜像源

https://mirrors.163.com/ubuntu/

查看更多的镜像

```Shell
wget -qO - mirrors.ubuntu.com/mirrors.txt
```

我这里返回值是

> http://mirrors.aliyun.com/ubuntu/ https://mirrors.hit.edu.cn/ubuntu/ http://mirrors.huaweicloud.com/repository/ubuntu/ http://mirrors.sohu.com/ubuntu/ http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ http://linux.xjtuns.cn/ubuntu/ http://mirrors.cqu.edu.cn/ubuntu/ https://mirror.bjtu.edu.cn/ubuntu/ http://mirrors.nju.edu.cn/ubuntu/ http://mirrors.ustc.edu.cn/ubuntu/ http://mirrors.yun-idc.com/ubuntu/ https://mirrors.bfsu.edu.cn/ubuntu/ http://mirror.lzu.edu.cn/ubuntu/ http://mirrors.dgut.edu.cn/ubuntu/ http://ftp.sjtu.edu.cn/ubuntu/ http://mirrors.njupt.edu.cn/ubuntu/ http://archive.ubuntu.com/ubuntu/