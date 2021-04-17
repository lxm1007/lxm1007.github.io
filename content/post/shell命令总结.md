---
title: "Shell命令总结"
date: 2021-01-28T17:52:53+08:00
categories:
- shell
- linux
tags:
- shell
- nginx
keywords:
- shell
thumbnailImage: /img/ymgvlk.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/011zk3.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->

# 不常见但好用shell命令总结

## ```xargs``` 配合管道符使用

> ``` kubectl get pod -n a5bcbee0-d55f-11ea-806a-0242ac110004 |grep Terminating | awk '{print $1}'| xargs kubectl delete  pod -n a5bcbee0-d55f-11ea-806a-0242ac110004 --grace-period=0 --force  ``` 删除某个命令空间下所有停止中的pod

## ```ps aux | pgrep []``` 获取进程号

> 获取进程号时，使用```ps aux | grep kubelet | awk '{print $2} 通常会多一个没有用的或者多个进程'``` 可以使用```ps aux | pgrep kubelet```替换 这样只有一个并且是对的

## ```CTRL+R``` 然后输入指定命令可以直接查看历史命令

## ```netstat -rn ```或 ```route -n``` 查看本地路由表

## ``` host -a|-v 123 ```查看域名搜索顺序，可以配合着/etc/resolv.conf search域

> host -a 【要搜索的域名】

> 如果类型 123或者abc这种没有.并且也不是以.结尾的 那么在查询的时候会直接在dns进行查找


## 使用```ls -lah /proc/[pid]/exe``` 查看详细的进程所在位置