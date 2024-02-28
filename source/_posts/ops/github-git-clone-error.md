---
title: git clone error
tags: []
id: '671'
categories:
  - - Something
date: 2018-07-13 19:00:14
---

要在 github 上下载一个朋友做的[项目](https://github.com/muxiangqiu/bdf3.git)，试了几次都报如下错误，但是在海外的 VPS 上秒 clone 完成，所以猜测是原始 resposity 过大，网络又不好导致的。

```
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

### 解决方案:

#### 节流:

只 clone 每个文件最新的一个提交，然后再 fetch 完整的提交记录。

```Shell
$ git clone http://github.com/large-repository --depth 1
$ cd large-repository
$ git fetch --unshallow
```

#### 开源:

如果有梯子，为 git 配置加速的梯子也能解决问题。

```Shell
git config --global http.proxy http://127.0.0.1:4411
```

或者

```Shell
export http_proxy=http://127.0.0.1:4411 # 配置http访问的
export https_proxy=http://127.0.0.1:4411 # 配置https
export all_proxy=http://127.0.0.1:4411 # 配置http和https访问
```

#### 其它方式

1.  使用 SSH 的方式 Use SSH 使用该方式，需要在 github settings -> SSH and GPG keys 里上传一个 SSH 公钥。 [![上传 SSH 公钥](https://i.loli.net/2018/07/13/5b486ae9d8a06.jpg "上传 SSH 公钥")](https://i.loli.net/2018/07/13/5b486ae9d8a06.jpg "上传 SSH 公钥")

```Shell
$ git clone git@github.com:large-repository
```

2.  下载 zip 包 Download Zip 通过该方式能下载master 分支下的完整源码，但是没有任何 git 提交记录。 [![其它下载方式](https://i.loli.net/2018/07/13/5b486a712689c.jpg "其它下载方式")](https://i.loli.net/2018/07/13/5b486a712689c.jpg "其它下载方式")

#### 完成无用的方案:

该设置只影响到 push，对 clone 毫无帮助.

```Shell
git config –-global http.postBuffer 524288000
```