---
title: "Maven的一些技巧"
date: 2022-01-17T21:01:49+08:00
categories:
- mvn
- maven
tags:
- mvn
- maven
keywords:
- maven
thumbnailImage: /img/e3e45ad8c5ae22995c2fd77a994dfc61.webp
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/b2af2266fc1e2ccd2365cf0add7e7369.webp
metaAlignment: center
coverMeta: out
---

<!--more-->


## 构建指定模块
> ``` clean deploy -Dmaven.test.skip=true -pl project-a,project-b,project-c  ```(只构建其中三个个)
