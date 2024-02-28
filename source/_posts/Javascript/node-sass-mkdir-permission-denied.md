---
title: >-
  node-sass Error: EACCES: permission denied, mkdir
  'xxx/node_modules/node-sass/vendor'
tags: []
id: '793'
categories:
  - - pitfall
comments: false
date: 2020-04-02 10:08:06
---

Unable to save binary //node\_modules/node-sass/vendor/linux-x64-64 : { Error: EACCES: permission denied, mkdir 'xxx/node\_modules/node-sass/vendor' at Object.mkdirSync (fs.js:752:3) 使用npm打包时，发现node-sass无法安装成功,但是用yarn可以。 原来是因为 npm 为了安全禁止使用root用户或者sudo来安装node-sass，切换到普通用户就可以了，或者添加 `--unsafe-perm` 参数。

```Shell
sudo npm install --unsafe-perm -g node-sass
```

[Running with sudo or as root](https://github.com/sass/node-sass/blob/master/TROUBLESHOOTING.md#installation-problems "Running with sudo or as root")