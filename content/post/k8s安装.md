---
title: "K8s总结"
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

> `kubeadm config images list --kubernetes-version=v1.18.7` 镜像列表 --kubernetes-version 指定版本

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

# k8s主机通过service名访问service配置

>在pod内可以通过service名访问对应的服，这是因为在创建pod的时候coredns将需要的配置添加到/etc/resolv.conf 

>类似``` search default.svc.cluster.local svc.cluster.local cluster.local ``` 其中  search 域中 default.svc.cluster.local 格式为【命名空间】.svc.【k8s安装时指定的域】

>可以在k8s初始化时指定 ``` --service-dns-domain ``` 默认为 cluster.local 当多集群时需要进行配置，该信息会存储到对应的configmap中

# k8s中使用技巧
## 端口转发
> 使用```kubectl port-forward nginx-598b589c46-ffn25 8090:80```将pod的容器端口80映射到宿主机8090
## 显示pod指定标签 
> ```kubectl get pod  -L pod-template-hash -Listio.io/rev -Lservice.istio.io/canonical-name``` 通过-L指定标签将在查询的时候将该列的值显示出来，kubectl get 可以接多个-L。同时也可以使用逗号分隔所要显示的标签 ```kubectl get pod  -L pod-template-hash,istio.io/rev,service.istio.io/canonical-name```

## 显示所有标签
> ```kubectl get pod  --show-labels ```

## 根据标签过滤要显示的pod 
> ```kubectl get pod  -l service.istio.io/canonical-name=details``` 只显示标签带有指定key和value的pod，```kubectl get pod  -l service.istio.io/canonical-name``` 则显示标签带有该key的pod
## 删除命名空间下所有资源
> 在删除命名空间时 将删除命名空间下所有的对象资源

> ```kubectl delete all --all -n lxm-test``` 该操作也会删除对应的命名空间下所有的资源【尽量删除】

## 日志打印前一个容器

> ```kubectl logs lee-nginx-779d8f74b8-wf9jw -n logging  --previous``` --previous 查看前一个容器的日志，不存在是会报错

## 容器退出码
> 容器退出码为128+x，当退出码为137时为128+9，则为强制退出，这时候容器不是重启，应该是重新创建
## 标签选择器(operator)
> In:label 的值必须与其中的一个指定的values匹配

>NotIn: label值与任何指定的values都不匹配

>Exists: pod要包含一个指定的标签名，value无所谓，同时values值不应该设置

>DoesNotExist: pod不能包含指定的标签，values不应该指定

>matchExpressions中指定的规则要都满足才可以，matchExpressions和matchLabels都指定时，要求两者对应的所有规则都满足才可以

## job
> job类型的pod不能使用默认的重启策略 Always,需要设置为OnFailure或者Never

> activeDeadlineSeconds：设置job可以运行的最大时间。backoffLimit：设置失败的重试次数

> startingDeadlineSeconds: 在计划任务开始时间后执行【最迟】