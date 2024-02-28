---
title: NodeJS 镜像配置 npm yarn 淘宝镜像
tags: []
id: '803'
categories:
  - - JavaScript
comments: false
date: 2020-07-15 18:37:55
---

下载npm包经常下载不下来，还好淘宝提供了npm的镜像站，配置好之后，再使用npm或yarn就可以通过淘宝镜像站下载npm包，不用使用cnpm。

## 配置镜像

```Shell
npm config set registry https://registry.npm.taobao.org/
npm config set disturl https://npm.taobao.org/dist
npm config set electron_mirror https://npm.taobao.org/mirrors/electron/
npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/
npm config set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs/
npm config set chromedriver_cdnurl https://cdn.npm.taobao.org/dist/chromedriver/
npm config set operadriver_cdnurl https://cdn.npm.taobao.org/dist/operadriver
npm config set fse_binary_host_mirror https://npm.taobao.org/mirrors/fsevents

yarn config set registry https://registry.npm.taobao.org -g
yarn config set disturl https://npm.taobao.org/dist -g
yarn config set electron_mirror https://npm.taobao.org/mirrors/electron/ -g
yarn config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/ -g
yarn config set phantomjs_cdnurl https://npm.taobao.org/mirrors/phantomjs/ -g
yarn config set chromedriver_cdnurl https://cdn.npm.taobao.org/dist/chromedriver -g
yarn config set operadriver_cdnurl https://cdn.npm.taobao.org/dist/operadriver -g
yarn config set fse_binary_host_mirror https://npm.taobao.org/mirrors/fsevents -g
```