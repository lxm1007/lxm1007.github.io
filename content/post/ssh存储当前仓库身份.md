---
title: "Ssh存储当前仓库身份"
date: 2021-01-28T16:22:37+08:00
categories:
- ssh
- 账号密码
tags:
- ssh
- 账号密码
keywords:
- ssh
thumbnailImage: /img/438w60.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/13oq3v.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->


当git操作pull或者push代码时，如果没有存储账号和密码，则每次都需要手动输入，通过配置```git config credential.helper store``` 可以保存当前仓库的账户和密码，避免每次输入，同时在 .git/config的文件中会添加 ```[credential] helper = store```