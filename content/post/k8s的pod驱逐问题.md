---
title: "K8s的pod驱逐问题"
date: 2021-01-28T15:39:04+08:00
categories:
- k8s
- pod
tags:
- k8s
- pod
keywords:
- k8s
thumbnailImage: /img/kw7q5d.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/r2r51m.jpg
metaAlignment: center
coverMeta: out
---


# 关于项目中k8s的几个知识点和对应的操作

## k8s的污点和容忍

>污点（Taints）是拒绝pods的操作，而容忍（Tolerations）是配合着污点使用，允许调度的操作


## 关于调度和驱逐的问题

> 至少有 1 个未被忽略的 taint 且 effect 是 NoSchedule 时，则 k8s 不会将该 pod 调度到这个 node 上

> 但至少有 1 个未被忽略的 taint 且 effect 是 PreferNoSchedule 时，则 k8s 将尝试不把该 pod 调度到这个 node 上【注意该场景是在上一基础上不满足才会执行】

>至少有 1 个未被忽略的 taint 且 effect 是 NoExecute 时，则 k8s 会立即将该 pod 从该 node 上驱逐（如果已经在该 node 上运行），或着不会将该 pod 调度到这个 node 上（如果还没在这个 node 上运行）

>NoExecute这个Taint效果对节点上正在运行的Pod有以下影响:
``` 
◎　没有设置Toleration的Pod会被立刻驱逐。
◎　配置了对应Toleration的Pod，如果没有为tolerationSeconds赋值，则会一直留在这一节点中。
◎　配置了对应Toleration的Pod且指定了tolerationSeconds值，则会在指定时间后驱逐。
◎　Kubernetes从1.6版本开始引入一个Alpha版本的功能，即把节点故障标记为Taint（目前只针对node unreachable及node not ready，相应的NodeCondition "Ready"的值分别为Unknown和False）。激活TaintBasedEvictions功能后（在--feature-gates参数中加入TaintBasedEvictions=true），NodeController会自动为Node设置Taint，而在状态为Ready的Node上，之前设置过的普通驱逐逻辑将会被禁用。注意，在节点故障的情况下，为了保持现存的Pod驱逐的限速（rate-limiting）设置，系统将会以限速的模式逐步给Node设置Taint，这就能避免在一些特定情况下（比如Master暂时失联）大量的Pod被驱逐。这一功能兼容tolerationSeconds,允许Pod定义节点故障时持续多久才被逐出```

## k8s中的一些问题

> 1.使用deploy 创建的pod，在没有设置容忍的时候，会默认加上```
node.kubernetes.io/not-ready:NoExecute for 300s
node.kubernetes.io/unreachable:NoExecute for 300s```的两个容忍，结合着上面驱逐的策略，当节点notready时，遍不会调度到该节点。如果节点上已经运行pod，并且pod没有配置对应的容忍，那么会将pod从该节点驱逐。

## 实际中遇到的问题

> 1.在使用kubeedge的时候，使用nodeName选择节点部署deploy并且没有设置容忍，这样导致的问题是：当节点失联后，k8s会把节点故障标记为Taint,节点上如果已经存在pod，那么根据k8s的策略进行驱逐，然后deploy又会根据副本数进行维护，但是由于master无法和node通信，导致的问题就是大量的pod处于Terminating，并且周期是5分钟（k8s设置的策略）

> 2.针对该场景的解决方案：目前想到的是，1、把节点的污点删除，这样由于pod就不需要做任何修改，pod的调度正常执行，pod的驱逐也不会产生，但是对于kubedge没问题，其他集群不建议这样处理。2、由于node添加了污点，那如果在对应的pod创建时也添加对应的容忍，并且容忍时间不设置，按照上面的说法，对于已经存在的pod，则应该会保留在对应的node上，这样也不会出现大量Terminating的问题。
