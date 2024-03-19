---
title: git pull error git config pull.rebase
tags:
  - git
categories: ops
date: 2024-02-28
---

正常用了好久的 git 项目突然 pull 命令不好使了，提示如下。这是是因为 git 在 2.27 版本后新增了一个显示的配置式，选择 git 的默认提交合并。

<!-- more -->

> Pulling without specifying how to reconcile divergent branches is discouraged. You can squelch this message by running one of the following commands sometime before your next pull:
>
> git config pull.rebase false     # merge (the default strategy)  
> git config pull.rebase true      # rebase  
> git config pull.ff only               # fast-forward only
>
> You can replace "git config" with "git config --global" to set a default preference for all repositories. You can also pass --rebase, --no-rebase, or --ff-only on the command line to override the configured default per invocation.

作为开发者就选第一个**默认策略**就好了。

```Shell
git config pull.rebase false
```

不想每个项目单独配置就加上`--global`标记作为全局配置

```Shell
git config --global pull.rebase false
```

默认策略下 git pull 其实相当于 git fetch 然后 git merge FETCH_HEAD，会生成一个 merge 的提交。提交记录整洁没有稀奇古怪的东西，就可以使用`rebase`。

如果希望使用`rebase`替代`merge`，就配置

```Shell
git config pull.rebase true
```

> `rebase` 还是` merge` 要看项目本身的要求。

如果主要是做更新，很少做任何提交，可以配置如下，该配置下如果不能快速合并就会报错。

```Shell
git config pull.ff only
```

这个模式下不是说就不能提交代码了，可以依旧通过手动的`fetch`然后`merge`或`rebase`来更新分支。相当于自动挡换手动挡了。
