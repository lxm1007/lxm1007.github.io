---
title: "K8s安装"
date: 2021-01-14T11:36:19+08:00
categories:
- k8s
- k8s安装
tags:
- k8s
- 安装教程
keywords:
- k8s
thumbnailImage: /img/4x1ov3.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/zm9kpy.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->
# k8s安装教程（保姆级）

## 前期准备
> 安装docker https://developer.aliyun.com/mirror/ 选择容器 -> docker-ce 

> `yum list docker-ce --showduplicates | sort -r` 查看对应的docker可安装版本

>`yum install --setopt=obsoletes=0 docker-ce-17.03.3.ce-1.el7` 安装指定版本（docker-ce-+"指定版本"）

>配置k8s软件仓库 选择容器->kubernetes `yum install -y kubelet kubeadm kubectl` 默认不加版本为仓库最新版本 不建议使用  版本的选择可以参考上面docker的安装

> `setenforce 0` 禁用selinux

> `kubeadm config print init-defaults > config.yaml` 创建初始化文件

> `imageRepository: registry.aliyuncs.com/google_containers # 配置阿里仓库`

> networking: <br/>
  &nbsp;&nbsp;&nbsp;&nbsp;podSubnet: 10.244.0.0/16 # pod的子网网络 注意 这个和要和部署的网络插件一致  flannel 默认为这个

> 下载需要的镜像  `kubeadm config images pull --config=config.yaml`

> `kubeadm config images list` 镜像列表

> `kubeadm init --config=config.yaml` 初始化集群

> 安装完成后 按照教程操作剩下加的步骤 

> flaanel 官方仓库: `https://github.com/coreos/flannel` flannel下载地址：https://github.com/coreos/flannel/tree/master/Documentation

> 安装dashboard  `https://github.com/kubernetes/kubernetes`  cluster->addons->dashboard->dashboard.yaml

> 修改svc方式为NodePort 

> `kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token` 查看登录token

> `kubectl -n kube-system describe $(kubectl -n kube-system get secret -n kube-system -o name | grep namespace) | grep token | awk '{print $2}' | sed 1,2d ` 直接获取 

> 访问对应的nodePort端口  如果遇到证书不信任的问题没法继续 直接键盘输入 `thisisunsafe`


#### 

# metric-server的安装  

> 使用阿里云创建镜像 https://github.com/kubernetes-sigs/metrics-server 

> `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.1/components.yaml` 修改配置文件

>     spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --kubelet-insecure-tls # 需要添加的非受信 否则创建pod异常
        image: k8s.gcr.io/metrics-server/metrics-server:v0.4.1
        imagePullPolicy: IfNotPresent

