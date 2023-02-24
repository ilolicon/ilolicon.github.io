---
title: "Kubernetes"
description: Kubernetes, also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.
date: 2020-09-20 14:21:14
categories: 
- CloudNative
tags:
- Cloud Native
- Kubernetes
- Pod
- Service
- Kubeadm
---

[![kubernetes readme](https://img.shields.io/badge/Kubernetes-README-356DE3)](https://kubernetes.io/zh/docs/home/)

[![kubernetes](icons/kubernetes.svg)](https://kubernetes.io/)

`container evolution`

![container_evolution](icons/container_evolution.svg)

## 容器编排

- Ansible/Saltstack **传统应用**编排工具
- Docker
  - docker compose(docker单机编排)
  - docker swarm(docker主机加入docker swarm资源池)
  - docker machine(完成docker主机加入docker swarm资源池的先决条件/预处理工具)
- Mesos(IDC OS) + marathon(面向容器编排的框架)
- kubernetes(Borg)
  - 自动装箱(基于依赖 自动完成容器部署 不影响其可用性)
  - 自我修复
  - 水平扩展
  - 服务发现和负载均衡
  - 自动发布和回滚
  - 密钥和配置管理
  - 存储编排
  - 任务批量处理运行

概念: `DevOps` `MicroServices` `Blockchain`

- CI: 持续集成
- CD: 持续交互 Delivery
- CD: 持续部署 Deployment

## Kubernetes架构概述

![kubernetes-cluster](icons/kubernetes_cluster.png)

- 有中心节点的集群架构系统(抽象各主机资源 统一对外提供计算)
- Master/Node(Worker)
  - Master
    - etcd: 兼具一致性和高可用性的键值数据库 可以作为保存Kubernetes所有集群数据的后台数据库
    - API Server(公开kubernetes API)
    - Scheduler(调度器)
    - Controller-Manager(控制器)
      - 节点控制器(Node Controller): 负责在节点出现故障时进行通知和响应
      - 任务控制器(Job controller): 监测代表一次性任务的Job对象 然后创建Pods来运行这些任务直至完成
      - 端点控制器(Endpoints Controller): 填充端点(Endpoints)对象(即加入Service与Pod)
      - 服务帐户和令牌控制器(Service Account & Token Controllers): 为新的命名空间创建默认帐户和API访问令牌
  - Node
    - kubelet: 集群代理 并不运行容器
    - kube-proxy: 网络代理 管理Service的创建 更新(iptables/ipvs)
    - 容器运行时(容器引擎): 运行容器 支持[Docker](https://kubernetes.io/zh/docs/reference/kubectl/docker-cli-to-kubectl/)/[containerd](https://containerd.io/docs/)/[CRI-O](https://cri-o.io/#what-is-cri-o)以及任何实现 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md)

[kubernetes组件](https://kubernetes.io/zh/docs/concepts/overview/components/)

![kuberbetes-cluster02](icons/kubernetes_cluster02.png)

![components-of-kubernetes](icons/components-of-kubernetes.svg)

## Pod

- Pod
  - 自主式Pod
  - 控制器管理的Pod(建议使用)
    - ReplicationController(副本控制器)
    - ReplicaSet(副本集控制器 不直接使用 有一个声明式更新的控制器Deployment)
    - Deployment(管理无状态应用)
      - HPA(HorizontalPodAutoscaler) 水平Pod自动伸缩控制器
    - StatefulSet(有状态副本集)
    - DaemonSet
    - Job/CronJob
  - Label
    - Label Selector
    - Label: key=value

## Service

- iptables的DNAT规则(调度多个service后端 ipvs规则)
- 靠标签选择器 关联Pod对象
- service 名称 可以被解析(DNS Pod实现解析 - 基础架构级的Pod)
  - 基础架构级Pod `AddOns`集群附加组件 支撑其他服务需要使用到的服务
  - 动态更新DNS解析

## 网络通信

- 同一个Pod内的多个容器间: lo
- 各Pod之间的通信(不直接通信 通过Service通信)
  - 二层广播 -> 广播风暴
  - Overlay Network 叠加网络
    - 需要确保每一个Pod网络地址不会冲突
- Pod和Service之间的通信

依赖第三方插件(附件) 通过CNI(容器网络接口)接入

- flannel: 网络配置 Pod网络-10.244.0.0/16
- calico: 网络配置 网络策略(Pod之间、Namespace...之间的访问或隔离)
- canel: 上面两种搭配使用
- ...

![k8s-network](icons/k8s-network.png)

## Namespace

- 管理的边界
- 隔离不同名称空间的网络

## 初始化Kubernetes集群

![structure](icons/structure.png)

![structure02](icons/k8s-network-structure02.png)

- [kubeadm](https://github.com/kubernetes/kubeadm)
  
  - master/nodes: 安装`kubetlet` `kubeadm` `docker`
  
  - master: `kubeadm init`
  
    ```bash
    # master init
    sudo kubeadm init \ 
      --kubernetes-version=v1.20.15 \     # k8s版本
      --pod-network-cidr=10.244.0.0/16 \  # pod网段
      --service-cidr=10.96.0.0/12 \       # service网段
      --ignore-preflight-errors=Swap      # 忽略预检错误
    ```
  
  - nodes: `kubeadm join`
  
    ```bash
    # nodes join
    sudo kubeadm join \
      10.211.55.57:6443 \
      --token hewmhd.fro3dttm001dohys \
      --discovery-token-ca-cert-hash sha256:826faddde57dc07d0fbf31e7aa809eb5fc22613787a2fc8ab50f55c4e706cd45
    ```
  
  - Images
  
    ```bash
    ilolicon@master:~$ sudo docker image ls
    REPOSITORY                           TAG        IMAGE ID       CREATED         SIZE
    k8s.gcr.io/kube-apiserver            v1.20.15   3ecdeee1255c   7 months ago    113MB
    k8s.gcr.io/kube-controller-manager   v1.20.15   403106abce42   7 months ago    108MB
    k8s.gcr.io/kube-scheduler            v1.20.15   bfc1b1725466   7 months ago    44.1MB
    k8s.gcr.io/kube-proxy                v1.20.15   3c1b1abd329d   7 months ago    93.6MB
    k8s.gcr.io/etcd                      3.4.13-0   05b738aa1bc6   24 months ago   312MB
    k8s.gcr.io/coredns                   1.7.0      db91994f4ee8   2 years ago     42.8MB
    k8s.gcr.io/pause                     3.2        2a060e2e7101   2 years ago     484kB  # 创建基础架构容器使用 为Pod提供底层基础结构 无需启动和运行
    ```

## Kuberntes资源概述

- RESTful风格API
  - GET、POST、DELETE、PUT...
  - kubectl run、get、expose、edit...
- 资源(对象)
  - workload: Pod、ReplicaSet、Deployment、StatefulSet、DaemonSet、Job、Cronjob...
  - 服务发现与均衡: Service、Ingress...
  - 配置与存储: Volume、CSI...
    - ConfigMap、Secret
    - DownwardAPI
  - 集群级资源
    - Namespace、Node、Role、ClusterRole、RoleBinding、ClusterRoleBinding
  - 元数据型资源
    - HPA、PodTemplate、LimitRange

### 资源清单定义

- 创建资源的方法

  - apiserver仅接收json格式的资源定义

  - yaml格式提供配置清单 apiserver可自动将其转换为json格式 而后再提交

- 大部分资源的配置清单 主要都有五个主要的部分组成

  - apiversion
    - `kubectl api-versions` # 所属API群组
    - 标识方式: `group/version` 省略组名则为core group

  - kind: 资源类别
  - metadata: 元数据
    - name: 同一类别中 name需要唯一
    - namespace: 所属k8s的哪个名称空间
    - labels
    - annotations
    - 每个资源的引用PATH
      - `/api/${GROUP/VERSION}/namespace/${NAMESPACE}/${TYPE}/${NAME}`
  - spec(重要): 定义用户期望的状态 disired state
  - status: 当前状态 current state 本字段由Kubernetes集群维护

- 字段太多 可以借助`kubectl explain --help`命令查看详细信息

  - kubectl explain pod
  - kubectl explain pod.metadata

### Pod概述

[pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)

#### 自主式Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: myapp
      image: nginx:alpine
    - name: busybox
      image: busybox:latest
      command:
        - "/bin/sh"
        - "-c"
        - "sleep 3600"
```

#### Pod资源

- spec.containers <[]object>

  ```yaml
  # kubectl explain pod.spec.containers
  - name: <string>
    image: <string>
    imagePullPolicy: <string>  # Always Never IfNotPresent
    ...

  # 修改镜像中的默认应用
  - command/args
  https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
  ```

- 标签
  - key = value
    - key: 字母 数字 _ - .
    - value: 可以为空 只能字母或数字开头及结尾

- 标签选择器
  - 等值关系: =、==、!=(不等于会筛选出不具有该标签的资源)
  - 集合关系
    - KEY in (VALUE1,VALUE2,...)
    - KEY notin (VALUE1,VALUE2,...)
    - KEY    # 存在这个KEY就行
    - !KEY   # 不存在此键的资源

- 许多资源支持内嵌字段来使用标签选择器
  - matchLabels: 直接给定键值
  - matchExpressions: 基于给定的表达式来定义使用标签选择器
    - {key: "KEY", operator: "OPERATOR",values:[VAL1,VAL2,VAL3,...]
    - 操作符
      - In、NotIn: values字段的值必须为非空列表
      - Exists、NotExists: values字段的值必须为空列表

- spec.nodeSelector <map[striong]string>
  - 节点标签选择器
- spec.nodeName `<string>`  # 直接指定运行node
- annotations
  - 与label不同的地方在于 它不能用于挑选资源对象 仅用于为对象提供**元数据**
  - 没有键长度/值长度限制
- spec.restartPolicy
  - 重启策略: One of Always, OnFailure, Never. Default to Always

#### Pod生命周期

- 状态

  - Pending  # 调度尚未完成
  - Running  # 运行状态
  - Failed
  - Succeeded
  - Unknown
  - ...

- Pod生命周期中的重要行为

  - 初始化容器
  - 容器探测(自定义命令/TCP套接字发请求/HTTP应用层请求)
    - liveness probe: 探测容器是否存活
    - readiness probe: 探测容器是否准备就绪 能对外提供服务
  - 钩子
    - post start
    - pre stop

  ![pod-lifecycle](icons/pod-lifecycle.png)

#### Pod容器探针类型

`kubectl explain pod.spec.containers.livenessProbe`

`kubectl explain pod.spec.containers.readinessProbe`

- exec Action
- httpGet Action
- tcpSocket Action

#### Pod控制器

- ReplicaSet

  - Kubectl explain replicaset
  - 用户期望副本数 标签选择器 Pod资源模版
  - 不建议直接使用ReplicaSet

  ```yaml
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: myapp
    namespace: default
  spec:
    replicas: 2
    selector:
      matchLabels:
        app: myapp
        release: canary
    template:
      metadata:
        name: myapp-pod
        labels:
          app: myapp
          release: canary
          environment: qa
      spec:
        containers:
        - name: myapp-container
          image: nginx:alpine
          ports:
          - name: http
            containerPort: 80
  ```

- Deployment
  - 构建在ReplicaSet之上 而非Pod
    - 实现滚动更新(多出或少于N个副本 控制更新粒度)、回滚
    - 通常管理10个历史版本(ReplicaSet)
  - 管理无状态应用最好的控制器
  - 无状态(只关注群体、不关注个体)、持续运行应用
  - 声明式管理(即可以创建 也可以更新 `kubectl apply -f deployment.yaml`)
  - `kubectl rollout history` 查看滚动历史

![deployment-update](icons/deployment-update.png)

- DaemonSet
  
  - `kubectl explain deployment.spec.strategy.rollingUpdate` # 滚动更新策略
  
  - 确保集群中的每一个节点(或部分满足条件的节点)精确运行一个Pod副本
  - 通常用于一些系统级的后台任务
  - 无状态、持续运行应用
  
- Job
  - 只做一次 只要完成就正常退出 没完成才进行重构
  - 执行一次性的作业
  - 不需要持续在后台运行 执行完成就退出
  
- Cronjob
  - 周期性Job
  
- StatefulSet
  - 管理有状态应用
  - 每一个pod副本单独管理 拥有自己独有的标识和独有的数据集
  
- TPR: Third Party Resources 1.2+ - 1.7

- CDE: Custom Defined Resources 1.8+

- Operator

### Service概述

#### Service代理

[services-networking](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)

- userspace
  - kube-proxy会监视Kubernetes控制平面对Service对象和Endpoints对象的添加和移除操作
  - 对每个Service它会在本地Node上打开一个端口(随机选择)  任何连接到**代理端口**的请求 都会被代理到Service的后端`Pods`中的某个上面
    - 使用哪个后端Pod是 kube-proxy 基于`SessionAffinity`来确定的
  - 最后 它配置 iptables 规则 捕获到达该 Service 的`clusterIP`(是虚拟 IP)和`Port`的请求 并重定向到代理端口 代理端口再代理请求到后端Pod
    - 默认情况下 用户空间模式下的kube-proxy通过轮转算法选择后端
  - 流量来回在内核和用户空间切换 效率较低

![services-userspace-overview](icons/services-userspace-overview.svg)

- iptables
  - `kube-proxy`会监视Kubernetes控制节点对Service对象和Endpoints对象的添加和移除
  - 对每个Service 它会配置iptables规则 从而捕获到达该Service的`clusterIP`和端口的请求 进而将请求重定向到 Service 的一组后端中的某个Pod上面
  - 对于每个Endpoints对象 它也会配置iptables规则 这个规则会选择一个后端组合
    - 默认的策略是 kube-proxy在iptables模式下随机选择一个后端

  - 使用iptables处理流量具有较低的系统开销 因为流量由Linux netfilter处理 而无需在用户空间和内核空间之间切换 这种方法也可能更可靠
  - 如果kube-proxy在iptables模式下运行 并且所选的第一个 Pod 没有响应 则连接失败
    - 这与用户空间模式不同: 在这种情况下 kube-proxy将检测到与第一个Pod的连接已失败 并会自动使用其他后端Pod 重试
    - 可以使用Pod[就绪探测器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes) 验证后端Pod可以正常工作 以便iptables模式下的kube-proxy仅看到测试正常的后端 避免将流量通过kube-proxy发送到已知已失败的 Pod

![services-iptables-overview](icons/services-iptables-overview.svg)

- ipvs
  - 特性状态： Kubernetes v1.11 [stable]
  - 在`ipvs`模式下 kube-proxy监视Kubernetes服务和端点 调用`netlink`接口创建相应的IPVS规则 并定期将IPVS规则与Kubernetes服务和端点同步 该控制循环可确保 IPVS 状态与所需状态匹配
  - 访问服务时 IPVS将流量定向到后端Pod之一
  - IPVS代理模式基于类似于iptables模式的netfilter挂钩函数 但是使用哈希表作为基础数据结构 并且在内核空间中工作
    - 这意味着 与iptables模式下的kube-proxy相比 IPVS模式下的kube-proxy重定向通信的延迟要短 并且在同步代理规则时具有更好的性能
    - 与其他代理模式相比 IPVS模式还支持更高的网络流量吞吐量

  - IPVS提供了更多选项来平衡后端Pod的流量
    - `rr`: 轮询(Round-Robin)
    - `lc`: 最少链接(Least Connection) 即打开链接数量最少者优先
    - `dh`: 目标地址哈希(Destination Hashing)
    - `sh`: 源地址哈希(Source Hashing)
    - `sed`: 最短预期延迟(Shortest Expected Delay)
    - `nq`: 从不排队(Never Queue)
  - 备注
    - 要在IPVS模式下运行kube-proxy必须在启动kube-proxy之前使IPVS在节点上可用
    - 当kube-proxy以IPVS代理模式启动时 它将验证IPVS内核模块是否可用
      - 如果未检测到IPVS内核模块 则kube-proxy将退回到以iptables代理模式运行

![services-ipvs-overview](icons/services-ipvs-overview.svg)

#### Service类型

- ClusterIP: 通过集群的内部IP暴露服务 选择该值时服务只能够在集群内部访问 这也是默认的`ServiceType`
- [NodePort](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport): 通过每个节点上的IP和静态端口(NodePort)暴露服务  NodePort服务会路由到自动创建的ClusterIP服务
  - 通过请求 `<节点IP>:<节点端口>` 你可以从集群的外部访问一个NodePort服务
  - Client -> NodeIP:NodePort -> ClusterIP:ServicePort -> PodIP:containerPort
  - 为避免单Node压力过大 会在外面再加一层负载均衡
    - 公有云环境: LBaaS(参考下面LoadBalancer类型)
- [LoadBalancer](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer): 使用云提供商的负载均衡器向外部暴露服务 外部负载均衡器可以将流量路由到自动创建的NodePort服务和ClusterIP服务上
- [ExternalName](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname): 通过返回CNAME和对应值 可以将服务映射到externalName字段的内容(例如`foo.bar.example.com`) 无需创建任何类型代理
  - FQDN(CoreDNS 内部解析)
    - CNAME -> FQDN(外部真正的FQDN )

#### Headless Services(无头Service)

- 有时不需要或不想要负载均衡 以及单独的Service IP 遇到这种情况 可以通过指定Cluster IP(`spec.clusterIP`)的值为 `"None"`来创建 `Headless` Service
- 你可以使用一个无头Service与其他服务发现机制进行接口 而不必与Kubernetes的实现捆绑在一起
- 对于无头`Services`并不会分配Cluster IP kube-proxy不会处理它们 而且平台也不会为它们进行负载均衡和路由 DNS如何实现自动配置 依赖于Service是否定义了选择算符

### Ingress

![ingress-flow](icons/ingress-flow.png)

[ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/)

- Service对后端特定类型Pod分类(label selector)
- Ingress基于上面的分类识别后端Pod 并生成配置信息注入到nginx(需要重载配置)/envoy/traefik等

[ingress-controllers](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/)

### 存储卷

[kubernetes-storage](https://kubernetes.io/zh-cn/docs/concepts/storage/)

`kubectl explain pods.spec.volumes`

- emptyDir # 临时目录 随pod删除而消失(生命周期同pod)
  - gitRepo(clone到机器 修改不会同步 需要同步可以自己再做一个sidecar)
- hostPath # 宿主机路径
- SAN(iSCSI...)、NAS(nfs、cifs...)
- 分布式存储
  - glusterfs、rdb、cephfs
- 云存储
  - EBS、Azure Disk...

#### PV/PVC

![pvc](icons/pvc.png)

#### StorageClass

- 存储设备需支持RESTful风格的创建请求
- 根据请求动态创建PV

#### ConfigMap

- ConfigMap是一种 API 对象 用来将非机密性的数据保存到键值对中
- 使用时[Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)可以将其用作环境变量、命令行参数或者存储卷中的配置文件
- ConfigMap将你的环境配置信息和[容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image)解耦 便于应用配置的修改

##### 容器化配置应用方式

- 自定义命令行参数
  - args: []
- 把配置文件直接打包至镜像
- 环境变量
  - CloudNative的应用程序一般可直接通过环境变量加载配置
  - 通过entrypoint脚本来预处理变量为配置文件中的配置信息
- 存储卷

#### Secret

- Secret是一种包含少量敏感信息例如密码、令牌或密钥的对象 这样的信息可能会被放在[Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)规约中或者镜像中
- 使用Secret意味着你不需要在应用程序代码中包含机密数据
- Secret类似于[ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/)但专门用于保存敏感数据

## StatefulSet控制器

- CoreOS Operator
- cattle/pet # 一个关注群体 一个关注个体(和无状态应用的区别)
- PetSet(1.3) -> StatefulSet(1.5+)
- StatefulSet主要用于管理有以下特性的应用程序
  - 稳定且唯一的网络标识符
  - 稳定且持久的存储
  - 有序、平滑的部署和扩展
  - 有序、平滑的终止和删除
  - 有序的滚动更新
- 一般来说 一个典型的StatefulSet由三个组件组成
  - handless service              # 无头服务 确保名称唯一
  - StatefulSet                         # 控制器
  - volumeClaimTemplate # 存储卷申请模版(不能使用同一存储卷 pod模版创建的存储卷都是一样的 所以需要卷申请模版)

- `kubelet explain sts.spec.updateStrategy.rollingUpdate`
  - partition \<inter\> # 控制更新的Pod
  - partition: N         # 大于等于编号N的Pod将被更新 默认值: 0

## 认证及ServiceAccount

### 认证授权

- 认证(支持多种认证方式) # 认证插件
  - 令牌认证 bearer token
  - ssl认证(确认服务端/客户端身份) 双向证书认证(https)
  - ...
- 授权检查(权限) # 授权插件
  - RBAC # kubeadm部署的集群强制开启RBAC
  - ...
- 准入控制(关联的其他资源或操作 是否有权限 进一步补充授权机制)
- API Server需要信息去识别客户端的操作
  - user: username + uid
  - group
  - extra
  - API(请求的Kubernetes API)
    - Request Path
      - `kubectl proxy --port=8080`
      - `curl http://localhost:8080/api/v1/namespaces`
      - `curl http://localhost:8080/apis/apps/v1/namespaces/default/deployments/myapp-deploy/`
    - HTTP request verb
      - GET POST PUT DELETE
      - get list create update patch watch proxy redirect delete deletecollection
    - Resources
    - SubResources
    - Namespace
    - API Group

### ServiceAccount

- 访问APIServer的两种客户端
  - kubectl/dashborad 集群外部客户端(userAccount)
  - pod 集群内部客户端(serviceAccount)
    - `kubectl explain pods.spec.serviceAccountName`
- kubeconfig
  - `kubectl config view`

### RBAC授权

- 授权插件
  - Node
  - ABAC(*Attribute-based access control*)
  - RBAC(*Role-based access contro*)
  - Webhook

![k8s-RBAC](icons/k8s-RBAC.png)

- K8S-RBAC
  - role
    - operations
    - objects
  - rolebinding
    - user account OR service account
    - role
  - `kubectl create role pods-reader --verb=get,list,watch --resource=pods --dry-run -o yaml`

![role-binding](icons/role-binding.png)

## Helm

- 类似yum

## Reference

- ubuntu

[apt install speciffic version](https://askubuntu.com/questions/428772/how-to-install-specific-version-of-some-package/428778#428778?newreg=d056242a7a0340598dd1d2d4fad63e81)

[set-proxy-on-ubuntu-docker](https://www.serverlab.ca/tutorials/containers/docker/how-to-set-the-proxy-for-docker-on-ubuntu/)

[apt-get-like-yum-whatprovides](https://askubuntu.com/questions/2493/apt-get-or-aptitude-equivalent-to-yum-whatprovides)

- kubetnetes

[kubectl-get-commponentstatus-shows-unhealthy](https://stackoverflow.com/questions/63136175/kubectl-get-componentstatus-shows-unhealthy)
