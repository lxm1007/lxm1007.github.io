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
> 当k8s节点需要使用svc名访问对应的服务时需要注意修改/etc/resolv.conf文件，nameserver 添加coredns的svc对应的clusterIp而在search 中添加```default.svc.cluster.local svc.cluster.local cluster.local localdomain``` 其中coredns的nameserver要在search上面

>可以在k8s初始化时指定 ``` --service-dns-domain ``` 默认为 cluster.local 当多集群时需要进行配置，该信息会存储到对应的configmap中

# k8s中使用技巧

## 查看组件状态

> ``` kubectl get cs ``` 查看组件状态 componentstatus 

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

## exec 使用

> ```kubectl exec -it -- 命令``` 其中双横杠是指kubectl命令项的结束，如果要执行的命令中没有- 那双横杠也不是必须的

> ```kubectl exec -it bash ```可以进入容器
## 会话亲和性

> 因为k8s服务不在http层面工作，cookie是http协议的一部分，所以会话亲和性只能设置none或者clientIp 默认是none

## 服务暴露端口问题

> 当服务暴露不只一个端口时，必须给每个端口指定名字

## service中的targetPort

> targetPort是容器对应的端口，当pod的端口更改后，如果service设置的是数值类型的端口，那么也要进行修改，但是当给pod的端口起了别名时，这时候service就可以使用pod的端口名，即使pod的端口修改了，对应的service也不用修改
## 关于服务的ip和端口
> 当先创建服务再创建pod的时候，再pod的环境变量中机会加入service的集群ip和端口，所以创建service和pod的步骤应该为：```先创建service，再创建pod```
## pod中能访问到对应svc的原理

>在创建的pod的/etc/resolv.conf serarch域中包含对应的```logging.svc.cluster.local svc.cluster.local cluster.local default.svc.cluster.local localdomain```  要查找的顺序
## svc和endpoint的其他妙用

> 除了通过选择器创建svc和对应的endpoint 还可以手动创建，手动创建svc和同名的enmdpoiont，这样的好处是可以自定义endpint的地址列表，变相可以在集群内访问其他集群外地址
> 通过设置别名的方式可以将一个公网服务变成集群内服务，此时设置```svc.sepc.type=ExternalName``` 这样就可以不用暴露公网服务了，该操作仅在dns层实施，为服务创建了canme记录，因为绕过了服务代理，所以不会获得集群ip

## headless类型svc
> 当设置svc的clusterIP为none时，此时svc没有对应的clusterIP，通过访问svc的地址可以发现此时返回的是pod的ip而不是svc的ip，此时是基于dns轮询实现的访问不同的pod，该类型的svc也用于statefulset中

## nodePort情况下减少调数
> 当svc为nodePort时 3个节点起4个服务时可能存在当访问到一个节点时实际的pod是在另外一个节点上，这时候可以设置```externalTrafficPolicy=local```保证只在本地节点访问，从而减少跳数，但是这样是有缺陷的，当存在负载均衡时，就会导致实际访问的pod分布不均
## 发现未就绪pod
> 通过设置svc的```publishNotReadyAddresses``` 可以发现未就绪pod

## 使用命令创建pod而不是deployment
>```kubectl run --image=nginx --generator=run-pod/v1```

## 卷相关
> pv和storgeClass并非命名空间内资源

>动态创建pv：先添加一个storageClass，然后再pvc声明中个指定stroageClass的name，此时就可以自动创建pv和pvc

> 如果想要使用预先创建的pv而不是自动创建，那么storageClass 应该设置为"",否则就算存在满足的pv时，系统也会动态创建一个新的

## exec和shell
> dockerfile中的entrypoint为程序入口，而cmd为默认执行参数，一般是cmd提供默认的参数，也就是镜像在运行中就算不提供参数也可以使用cmd的默认参数，而如果想要覆盖也是可以的。

> shell的形式形如```/bin/bash -c 'sleep 10' ``` 而exec的形式为```sleep 10 ``` 虽然这个例子不恰当但是确实是这个意思 shell的始终会比exec的多一个bash进程

> k8s的pod中command和args等同docker的entrypoint和cmd 

## configmap
> 当pod引用的configmap不存在时则容器无法启动，等configmap创建之后容器就会自动创建

> subpath： 在docker中-v挂载会对已经存在文件的目录清空然后将需要挂载的文件挂载进去，这样的影响无疑是巨大的。k8s支持subpath只将需要的文件挂载到指定目录而不是全部

## secret 
> secret 挂载到容器的数据始终在内存中而不会持久化到硬盘

## 挂载到容器的secret
> default-token 挂载到 ```/var/run/secrets/kubernetes.io/serviceaccount``` 当前目录下有三个文件ca.crt、token、namespace，使用```curl --cacert ca.crt  https://kubernetes``` 访问k8s的master，虽然使用-k也可以，但是这样会导致中间人攻击，所以不要盲目地相信连接的服务是可信的,在请求头加上token即可以访问
>可以通过定义serviceAccount添加imagepullsecret来解决每个pod都需要指定的问题
## dowanward
> 将pod的元信息传递给容器：1、通过环境变量 2、通过挂载目录 3、调用api 。其中挂载目录的方式虽然相对来说比较麻烦，但是如果可能，可以在一个pod中将容器1的信息挂载到容器2

> 使用挂载的方式时需要注意指定容器名，因为pod中不一定只有一个容器

## 开启api-server的swagger-ui
> 使用 --enable-swagger-ui=true 开启swagger 可以在swagger-ui创建资源

## 显示kubectl 和master 之间通信
> ```kubectl get pod -A --v 6```

## rolling-update 
> 此操作的步骤是在v1版本对应的pod添加所属deploy的label，同时在rc标签选择器添加这个标签，然后将v2的rc副本数逐渐增加，v1版本的rc的副本数逐渐减少。该操作存在的问题是，修改副本数是由kubectl通过掉用master的api实现的，也就是说在kubectl和master失去联系时将不知道会发生什么，这也是deployment出现的原因，因为该操作不符合声明式

## 回滚
> ```kubectl rollout undo deployment``` 该操作在滚动升级时同样有效，滚动升级立即结束，所有pod恢复为旧版本

> 在创建deploy 时使用--record 可以在回滚历史中查看操作原因，这对回滚非常有用 

> ```kubectl rollout history deployment ``` 查看回滚历史 

> ``` kubectl rollout undo deployment  --to-revision```回滚到指定版本

## 暂停和恢复
> ```kubectl rollout pause ```进行暂停，当想要在旧版本先运行一个新版本时，可以在达到一个新pod的时候暂停升级，这样就会有一部分新的一部分旧的,当确认新的没有问题，可以使用```kubectl rollout resume ```恢复

## statefulset 

> 当出现节点故障时导致节点失联这时候pod状态是未知，那么只有强制删除pod（因为等待到一定时间该pod会被驱逐，本来状态是已删除但是由于节点失联所以普通删除无效）之后，那个节点对应的pod会在其他节点创建同名的pod

>当出现缩容时，会删除最高索引值的实例

>不允许在实例不健康的时候进行缩容

>pv在缩容时不会删除，所以在不小心操作缩容的时候，可以再执行一次扩容，那么由于pv并没有删除，而且新增和pod和之前缩容前一致

> statefulset at-most-one 保证只有在准确确认一个pod不再运行时才会创建一个新的pod

> statefulset 在修改模板：比如修改镜像版本为2并且数量加1，镜像这时是不会修改之前的两个副本的镜像的，会新增一个使用新版本的镜像的pod

## 查看组件状态
> kubectl get componentstatus

## 关于etcd
> 只有apiserver 才能和etcd交互，这样有助于实现乐观锁（提交数据的时候服务端需要检查客户端提供的版本号，如果版本号不同则不需要更新）

> ```etcdctl get /registry --prefix=true``` 查询给定前缀的key

>etcd使用raft算法保证一致性

> etcd集群数量最好为奇数那是因为在有一个状态变成另外一个状态的时候，需要节点超过半数，否则不进行更新

## 调度器
> 因为在k8s集群中同一时间不能有两个控制器在工作，所以就要有对应的选举机制，在调度器中时通过更新kube-scheduler的endpoint实现的：如果没有则创建一个endponit，通过设置annotion的```control-plane.alpha.kubernetes.io/leader``` 值来设置leader，实例间会竞争但最终只有一个，一旦成为leader将会有2s（默认）的更新资源任务

## RBAC
> 角色和集群角色指定了在资源上可以执行哪些动作，角色绑定和集群角色绑定决定了哪些谁可以做这些动作。角色和角色绑定和集群角色和集群角色绑定的区别是定义在集群还是命名空间

>注意一个roleBinding不能访问整个集群资源，即使它引用了一个clusterRole

> 组合：clusterRole和roleBinding 在多个命名空间重用clusterRole。role和roleBinding适合

## 安全
> 容器的运行运行用户时在构建镜像的时候指定的 USER 命令 指定，如果没有指定则默认使用root

>在linux下可以使用id 查看当前运行的用户和用户组

>使用containers.securityContext.runAsUser 设置运行容器的用户而不是使用镜像默认的用户

>containers.securityContext.runAsNonRoot 在不知道当前镜像是哪个用户的情况下，禁止使用root用户

>添加内核功能而不是使用```privileged:true```

>podSecurityPolicy （psp）集群级别的控制pod的特权

> 通过设置networkPolicy设置pod的出入方向访问权限

## request和limit
>如果没有指定request只指定了limit 则request则会设置和limit一样

>pod中所有容器的request和limit的组合决定了pod的qos等级 BestEffort、Burstable、Guaranteed

>在相同等级下会优先释放已运行内存/内存申请量 值大的pod

>使用limitRange限制单独的pod container pvc的大小（创建的pod如果不指定则使用limitRange的值），配合resourceQuota限制整个命名空间的资源 

## hpa 
> 通过绑定deployment 进行pod的缩容和扩容，其中最小副本数可以是0，但是需要设置HPAScaleToZero特性开关为开

## 亲和性

>topologyKey在设置pod亲和性时必须的，其中zone为区域，region为地区。其中pod具有亲和性和反亲和性但是node只有亲和性

## 关于停止前钩子
> 停止前钩子的原理是向进程发送信号量，但是如果容器的启动命令不是shell方式，那么可能导致信号量被淹没而无法执行，那么就要注意在dockerfile中使用entrypoint [""] 代替 entrypoint "",即使用shell主线程起二进制的方式，而不是直接运行二进制

## 容器终止前消息
>在使用describe的时候，last state的message字段可以通过将信息写入/dev/termination-log实现，当然这个文件地址还也可以进行配置
