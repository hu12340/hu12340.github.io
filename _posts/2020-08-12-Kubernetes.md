---
layout: post
title: "Kubernetes"
date: 2020-08-12 
description: "Kubernetes"
tag: 区块链 
---

------

# Kubernetes简介

Kubernetes（简称k8s）是一个基于容器的分布式架构的系统支撑平台。它具备完备的集群管理能力：

-   多层次的安全防护和准入机制
-   多租户应用支撑能力（同时运行多种应用，相互影响小）
-   透明的服务注册能力和服务发现机制
-   强大的故障发现和自我修复能力
-   服务滚动升级和在线调度机制
-   可扩展的资源自动调度机制
-   多粒度的资源配额管理能力

k8s还具备完善的管理工具，可以在上面进行部署测试、运维监控等各个环节。

## 架构

![image-20200812104849028](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\Kubernetes架构.png)

![image-20200812105819079](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\Kubernetes架构2.png)

-   kubectl：k8s的sdk（对接k8s）

-   API Server：信息交流站（唯一的准入入口）

    -   authentication authorization：权限验证（token）
    -   schedule actuator：调度机制

-   Node：

    -   kubelet：接收处理master发出的指令和任务
    -   proxy：网络通信代理

-   Pod：容器集群，调度的基本单位。（机器、容器，一般是单个服务）

-   Service：网络代理，pod的访问代理

-   Replication Controller：RC，控制Pod数量

-   Scheduler：Pod的调度

### API Server

特性：

1.  提供了各类资源对象（如Pod、RC、Service）的增删改查以及监控等RESTful接口。是模块间的数据交互和通讯枢纽
2.  k8s的集群管理和API入口
3.  提供完善的集群安全机制，包括认证授权、数据校验以及集群状态变更等

安全机制：

1.  可以开启HTTPS安全端口（--sercure-port=6443）启动安全机制
2.  选择性的暴露部分REST接口，启动一个内部代理，过滤掉不想被访问的REST API
3.  开启白名单，限制特定IP访问。--accept-hosts=“”

集群模块之间的通信：

1.  集群内的各个功能模块通过API Server将信息存入ETCD，当需要获取和操作这些数据时，则通过API Server提供的REST接口（GET、LIST、WATCH方法）来实现，从而实现各个模块之间的信息交互。
2.  为了缓解集群中各模块对API Server的访问压力，模块都采用了缓存机制来缓存数据，模块从API Server获取到数据后，会保存到本地缓存，在某些情况下将不会之间访问API Server，二十访问缓存数据来间接访问API Server。

### Controller Manager

​		集群内部的管理控制中心，负责集群内的Node、Pod、Endpoint、Namespace、Service、资源定额（Resource Quota）等的管理，也担当了高可用的负责人，当某个Node意外宕机时，Controller Manager会及时发现故障并自动化修复，确保集群始终处于可用的工作状态。

![image-20200818152703501](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\ControllerManager架构.png)

#### Replication Controller

​		简称RC，称为副本控制器，作用是保证集群中一个RC所关联的Pod副本数始终保持预设值。

职责：

1.  确保集群中有且仅有N个Pod实例，N是RC中定义的Pod副本数量。
2.  通过调整RC中的spec Replicas属性值来实现系统扩容或缩容。
3.  通过改变RC中的Pod模板来实现系统的滚动升级。

#### Node Controller

通过API Server实时获取Node的相关信息，实现管理和监控集群中的各个Node节点的相关控制功能。

![image-20200818153441399](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\NodeController.png)

#### Resource Quota Controller

资源配额管理确保指定的资源对象在任何时候都不会超量占用系统物理资源，支持三个层次的资源配置管理。

1.  容器级别：对CPU和Memory进行限制。
2.  Pod级别：对一个Pod内所有容器的可用资源进行限制。
3.  Namespace级别：包括Pod数量、Replication Controller数量、Service数量、Resource Quota数量、Secret数量和可持有的PV（Persistent Volum）数量。

#### NameSpace Controller

​		用户通过API Server可以创建新的Namespace并保存在etcd中，Namespace Controller定时通过API Server读取这些Namespace信息。如果Namespace被API标记为优雅删除（即设置删除期限，Deletion Timestamp），则将该Namespace状态设置为“Termination”，并保存到etcd中。同时Namespace Controller删除该Namespace下的Service、RC、Pod、PVC等所有的资源对象。

主要职责：创建和删除Namespace。

#### Endpoint Controller

​		EndPoints表示了一个Service对应的所有Pod副本的访问地址，而Endpoints Controller负责生成和维护所有Endpoints对象的控制器。它负责监听Service和对应的Pod副本的变化。

1.  如果检测到Service被删除，则删除和该Service同名的Endpoints对象。
2.  如果检测到新的Service被创建或修改，则根据该Service信息获得相关的Pod列表，然后创建或更新Service对应的Endpoints对象
3.  如果检测到Pod事件，则更新它对应的Service的Endpoints对象。

#### ServiceController

​		属于kubernetes集群与外部的云平台之间的一个接口控制器。监听Service变化，如果是一个LoadBalancer类型的Service，则确保外部的云平台上对该Service对应的LoadBalancer实例被相应地创建、删除以及更新路由转发表。

Service、Endpoint、Pod的关系：

![image-20200818155208189](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\三者关系.png)

### Scheduler

主要负责Pod调度。在整个系统中起承上启下的作用。

承上：负责接收Controller Manager创建的新Pod，为其选择一个合适的Node。通过调度算法为待调度Pod列表的每个Pod从Node列表中选择一个最合适的Node，并将信息写入etcd中。

启下：Node上的kubelet接管Pod的生命周期。kubelet通过API Server监听到kubernets Scheduler产生的Pod绑定信息，然后获取对应的Pod清单，下载Image，并启动容器。

调度流程：

![image-20200818155649694](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\Scheduler调度.png)

### Kubelet

​		在kubernetes集群中，每个Node节点都会启动kubelet进程，用来处理Master节点下发到本节点的任务，管理Pod和其中的容器。kubelet会在API Server上注册节点信息，定期向Master回报节点资源使用情况，并通过cAdvisor监控容器和节点资源。可以吧kubelet理解成 Server-Agent 架构中的agent，是Node上的Pod管家。

主要职责：节点管理、Pod管理

#### 节点管理

​		节点通过设置kubelet的启动参数“--register-node”，来决定是否向API Server注册自己，默认为true。可以通过kubelet --help或者查看kubernetes源码 cmd/kubelet/app/server.go中来查看该参数。

默认配置文件在 /etc/kubernetes/kubelet中，其中

-   -kubeconfig：用来配置kubeconfig的路径，kubeconfig文件常用来指定证书。
-   -hostname-override：用来配置该节点在集群中显示的主机名。
-   -node-status-update-frequency：配置kubelet心跳上报的频率，默认为10s。

#### Pod管理

Kubelet通过API Server获取etcd中自身节点需要的Pod信息，同步到缓存中，根据需求去创建或删除pod。

kubelet创建和修改Pod流程：

1.  为该Pod创建一个数据目录。
2.  从API Server读取该Pod清单。
3.  为Pod挂载外部卷（External Volume）
4.  下载Pod用到的Secret。
5.  检查运行的Pod，执行Pod未完成的任务。
6.  先创建一个Pause容器，该容器接管Pod的网络，再创建其它容器。
7.  Pod中容器的处理流程：
    1.  比较容器hash值并做相应处理。
    2.  如果容器被终止了且没有指定重启策略，则不做处理。
    3.  调用DockerClient下载容器镜像，调用Docker Client运行容器。

Kubelet执行健康检查：Liveness Probe、Readiness Probe

Liveness Probe：

1.  用于判断容器是否存活（running状态）。
2.  如果Liveness Probe探针探测到容器非健康，则kubelet将杀掉该容器，并根据容器的重启策略做相应处理。
3.  如果容器不包含LivenessProbe探针，则kubelet认为该探针的返回值永远为“success”

Readiness Probe：

1.  用于判断容器是否启动完成（read状态），可以接受请求。
2.  如果ReadnessProbe探针检测失败，Pod将会被删除。Endpoint Controller将从Service的Endpoint中删除包含该容器所在Pod的endpoint。

探针的三种实现方式：

1.  Exec Action：在一个容器内部执行一个命令，如果该命令状态返回值为0，则表明容器健康。
2.  TCP Socket Action：通过容器IP地址和端口号执行TCP检查，如果能够建立TCP连接，则表容器健康。
3.  HTTP Get Action：通过容器的IP地址、端口号及路径调用HTTP Get方法，如果相应的状态码大于等于200且小于等于400，则认为容器健康。

### Kube-proxy

1.  kube-proxy是管理service的访问入口，包括集群内Pod到Service的访问和集群外访问service。
2.  kube-proxy管理service的Endpoints，该service对外暴露一个Virtual IP，也成为Cluster IP，集群内通过访问这个Cluster IP：Port就能访问到集群内对应的service下的Pod。
3.  service是通过Selector选择的一组Pods的服务抽象，其实就是一个微服务，提供了服务的LB和反向代理的能力，而kube-proxy的主要作用就是负责service的实现。
4.  service另外一个重要作用是，一个服务后端的Pods可能会随着生存灭亡而发生IP的改变，service的出现，给服务提供了一个固定的IP，而无视后端Endpoint的变化。

代理模式：userspace、iptables、ipvs

>userspace mode是v1.1及之前版本的默认模式。
>
>iptables mode是v1.10及之前版本的默认模式。
>
>ipvs mode是v1.11及之前版本的默认模式。

#### userspace

userspace是在用户空间，通过kube-proxy来实现service的代理服务。

![image-20200818164331952](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\userspacemode.png)

​		该模式请求在到达iptables进行处理时就会进入内核，而kube-proxy监听则是在用户态，请求就形成了从用户态到内核态再返回到用户态的传递过程，一定程度降低了服务性能。

#### iptables

iptables模式完全利用内核iptables来实现service的代理和LB。

![image-20200818164848269](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\iptablesmode.png)

​		该模式相比userspace模式，克服了请求在用户态-内核态反复传递的问题，性能上有所提升，但使用iptables NAT来完成转发，存在不可忽视的性能损耗，而且在大规模场景下，iptables规划的条目会十分巨大，性能上还要再打折扣。

>   另外，如果集群中存在上万的service和endpoint，那么node上的iptables rules将会非常庞大，性能还会再大打折扣。这也导致目前大部分企业用k8s上生产时，都不会直接用kube-proxy作为服务代理，二十通过自己开发或者通过Ingress Controller来集成HAProxy，Nginx来代替kube-proxy。

#### ipvs

​		在kubernetes 1.8 以上版本中，新增ipvs模式。kube-proxy ipvs是基于NAT实现的，通过ipvs的NAT模式，对访问k8s service的请求进行虚IP的POD IP的转发。当创建一个service后，kubernetes会在每一个节点上创建一个网卡，同时帮你将Service IP（VIP）绑定上，此时相当于每个Node都是一个dns服务器。

![image-20200818170012876](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\ipvsmode.png)

​		ipvs是目前kube-proxy所支持的最新代理面膜是，相比使用iptables，使用ipvs具有更高的性能。

>   与iptables、userspace模式一样，kube-proxy 依然监听Service以及Endpoints对象的变化，不过它并不创建反向代理，也不创建大量的iptables规则，二十通过netlink创建ipvs规则，并使用k8s Service与Endpoints信息，对所在节点的ipvs规则进行定期同步；netlink与iptables底层都是基于netfilter钩子，但是netlink由于采用了hash table 而且直接工作在内核态，在性能上比iptables更优。

## Kubernetes的网络实现

​		Kubernetes采用的是基于扁平地址空间的、非NAT的网络模型，每个Pod都有自己唯一的IP地址，网络是由系统管理员或CNI（container network interface）插件建立的，而非k8s本身，k8s并不会要求用户使用指定的网络技术，但是会授权Pod（容器）在不同节点上的相互通信。

1.  容器到容器之间的通信。
2.  Pod到Pod之间的通信。
3.  Pod到service之间的通信。
4.  集群外部与内部服务之间的通信

### Kubernetes的网络模型

一、同一个POD内容器与容器之间的通信：

​		Kubernetes使用了一种“IP-per-pod”网络模型：为每一个Pod分配了一个IP地址，Pod内部的容器共享Pod的网络空间，即他们共享Pod的网卡和IP。在同一个Pod内的容器（Pod内的容器是不会跨宿主机的）之间对于网络的各类操作，就和它们在同一台机器上一样，他们甚至可以用localhost地址访问彼此的端口。

![image-20200818173247729](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\同一个pod.png)

二、Pod与Pod之间的通信

两个Pod即有可能运行在同一个Node上，也有可能运行在不同的Node上。所以，可以吧Pod间通信分为两类：

1、同一个Node内的Pod之间的通信

![image-20200818173351691](D:\MyBlog\hu12340.github.io\images\posts\Kubernetes\同Node不同pod.png)

​		可以看到，Pod1和Pod2都是通信veth pair链接到同一个docker0网桥上，他们的IP地址IP1、IP2都是从docker0网段上动态获取的，他们和网桥本身的IP3是同一个网段的。由于Pod1和Pod2处于同一局域网内，他们之间可以通过docker0作为路由量进行通信。

2、不同Node上的Pod之间的通信











​		这个是使用github搭建博客所需使用的文章模板，新建文章都可以在此基础上复制并修改。并且博客文章的名字也必须是 yyyy-mm-dd-文章标题 （如：2019-11-01-博客文章模板）

<div align="center">
	<img src="/images/posts/图片路径" />  
</div> 