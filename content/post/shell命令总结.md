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

## 解决磁盘无法挂载的问题

> 1、查看日志 ``` dmesg 【对应磁盘】```

> 2、发现是uuid重复 提示不能挂载

> 3、```blkid``` 查看所有磁盘的uuid

> 4、临时解决无法挂载问题 ```mount -o rw,nouuid 【磁盘】  【挂载点】``` 该方式只是临时挂载，重启后失效

> 出现该问题的可能原因是：卸载挂载磁盘的时候 由于有文件在使用，所以不能卸载，于是openstack 强制卸载等再挂载的时候由于已经存在对应的uuid，所以挂载失败。解决方案可以将影响卸载的进程kill，然后正常卸载磁盘，再将新的盘挂载到对应的挂载点，然后写入fstab，使其永久生效

## vim编辑器的常用操作

> 1、```set nu``` 设施行数

> 2、 ```set paste``` 解决格式冲突

> 3、```set ff=unix ```使用unix换行

## ps获取不显示行信息

> ``` ps -C nginx --no-heading```
