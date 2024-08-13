# 核心概念
## 认识k8s
### 什么是k8s
#### k8s是一个开源的，用于管理云平台中多个主机上的容器化的应用，k8s目标是为了让部署容器化的应用简单高效，k8s提供了应用部署、规划、更新和维护的一种机制。
### 为什么需要k8s
#### 在我们熟悉了容器化部署与物理机或虚拟机部署的前提下，为什么需要k8s？主要从以下几方面展开
##### 1.自我修复-出现故障k8s拥有自愈功能
##### 2.弹性伸缩-可灵活添加或减少服务数量
##### 3.自动部署与回滚-保证服务一直存在，用户层面感知不到，或回退到之前版本
##### 4.服务发现和负载均衡-类比nginx
##### 5.机密和配置管理-统一配置或密码管理
##### 6.存储编排-就像管理员提前准备好的一块“存储盘”，可以使用各种存储后端（例如云存储、本地存储、网络存储）
##### 7.批处理-批量处理或实现任务

## 集群架构与组件
![image](https://github.com/user-attachments/assets/333bbf26-8681-4b06-be5f-49b22fd0d336)

### 相关组件
![image](https://github.com/user-attachments/assets/e37253b5-3e5c-4d0b-994d-7b8c5d016e65)

#### 控制面板组件（master）
##### api-server-接口服务：基于REST风格开放k8s接口服务
##### kube-controller-manager-控制器管理器：管理各个类型的控制器进程，针对k8s中的各种资源进行管理
###### 节点控制器：负责在节点出现故障时进行通知和响应
###### 任务控制器：检测代表一次性任务的Job对象，然后创建Pods来运行这些任务直至完成
###### 端电分片控制器：填充端电分片对象
###### 服务账号控制器：为新的命名空间创建默认的服务账号
##### cloud-coontroller-manager-云控制器管理器：第三方云平台提供的控制器API对接管理功能
##### kube-scheduler-调度器：将pod基于一定算法，将其调度到更合适的节点上
##### etcd-k8s的键值对分布式数据库，提供了基于Raft算法实现自主的集群高可用
#### 节点组件(node)
##### kubelet-管理pod生命周期，会在集群中每个节点（node）上运行。 它保证容器（containers）都运行在 Pod 中。kubelet 接收一组通过各类机制提供给它的 PodSpec，确保这些 PodSpec 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器
##### kube-proxy 是集群中每个节点（node）上所运行的网络代理， 实现 Kubernetes 服务（Service） 概念的一部分。kube-proxy 维护节点上的一些网络规则， 这些网络规则会允许从集群内部或外部的网络会话与 Pod 进行网络通信。如果操作系统提供了可用的数据包过滤层，则 kube-proxy 会通过它来实现网络规则。 否则，kube-proxy 仅做流量转发。
##### container runtime-容器运行时环境，例如docker、CRI-O
#### 附加组件
##### kube-dns 负责为整个集群提供 DNS 服务
##### Ingress Controller 为服务提供外网入口
##### Prometheus 提供资源监控
##### Dashboard 提供 GUI控制台
##### Federation 提供跨可用区的集群
##### Fluentd-elasticsearch 提供集群日志采集、存储与查询

### 分层架构
![image](https://github.com/user-attachments/assets/719e6880-97d6-4255-bb75-75138b5f9889)


#### 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴：Kubernetes 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等。Kubernetes 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等
#### 接口层：kubectl 命令行工具、客户端 SDK 以及集群联邦
#### 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）
#### 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）
#### 核心层：Kubernetes 最核心的功能，对外提供 API 构建高层的应用，对内提供插件式应用执行环境

## 核心概念与专业术语
### 服务的分类
#### 无状态：需要在服务运行环境下存储数据，操作磁盘相关服务
##### 代表应用：nginx，apache
##### 优点：对客户端透明、无依赖关系，可以高效实现扩展迁移
##### 缺点：不能存储数据，需要额外的数据服务支撑
#### 有状态：与无状态相反
##### 代表应用：MySQL，redis
##### 优点：可以独立存储数据，实现数据管理
##### 缺点：集群环境需要实现主从数据同步、备份、水平扩容复杂

### 资源和对象
#### 资源的分类
##### 元数据：对于资源的描述（控制）
一、Horizontal Pod Autoscaler（HPA）：Pod 自动扩容：可以根据 CPU 使用率或自定义指标（metrics）自动对 Pod 进行扩/缩容。控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况，  

支持三种metrics类型   

1.预定义metrics（比如Pod的CPU）以利用率的方式计算  

2.自定的Pod metrics，以原始值（raw value）的方式计算  

3.自定义的object metrics  

支持两种metrics查询方式：Heapster和自定义的REST API  

支持多metrics  

二、PodTemplate：Pod Template 是关于 Pod 的定义，但是被包含在其他的 Kubernetes 对象中（例如 Deployment、StatefulSet、DaemonSet 等控制器）。控制器通过 Pod Template 信息来创建 Pod。  

三、LimitRange：可以对集群内 Request 和 Limits 的配置做一个全局的统一的限制，相当于批量设置了某一个范围内（某个命名空间）的 Pod 的资源使用限制。  

##### 集群
一、Namespace（以及默认命名空间）：Kubernetes 支持多个虚拟集群，它们底层依赖于同一个物理集群，这些虚拟集群被称为命名空间。作用是用于实现多团队/环境的资源隔离。命名空间 namespace 是 k8s 集群级别的资源，可以给不同的用户、租户、环境或项目创建对应的命名空间。  

1.kube-system 主要用于运行系统级资源，存放 k8s 自身的组件  

2.kube-public 此命名空间是自动创建的，并且可供所有用户（包括未经过身份验证的用户）读取。此命名空间主要用于集群使用，关联的一些资源在集群中是可见的并且可以公开读取。此命名空间的公共方面知识一个约定，但不是非要这么要求。  

3.default 未指定名称空间的资源就是 default，即你在创建pod 时如果没有指定 namespace，则会默认使用 default  

4.kube-node-lease 该名字空间包含用于与各个节点关联的 Lease（租约）对象。 节点租约允许 kubelet 发送心跳， 由此控制面能够检测到节点故障。  

二、Node（可以立即为某个服务器，k8s只是服务调度）：不像其他的资源（如 Pod 和 Namespace），Node 本质上不是Kubernetes 来创建的，Kubernetes 只是管理 Node 上的资源。虽然可以通过 Manifest 创建一个Node对象（如下 json 所示），但 Kubernetes 也只是去检查是否真的是有这么一个 Node，如果检查失败，也不会往上调度 Pod。  

三、ClusterRole ClusterRole 是一组权限的集合，但与 Role 不同的是，ClusterRole 可以在包括所有 Namespace 和集群级别的资源或非资源类型进行鉴权。  

四、ClusterRoleBinding 将 Subject 绑定到 ClusterRole，ClusterRoleBinding 将使规则在所有命名空间中生效。  

##### 命名空间
一、工作负载  
1.Pod（容器组）是指一个或多个容器，具有共享存储和网络资源，以及如何运行容器的规范。Pod 的内容始终位于同一位置、共同调度，并在共享上下文中运行。Pod 模拟特定于应用程序的“逻辑主机”：它包含一个或多个相对紧密耦合的应用程序容器。在非云环境中，在同一物理或虚拟机上执行的应用程序类似于在同一逻辑主机上执行的云应用程序  

（1）副本（replicas）一个 Pod 可以被复制成多份，每一份可被称之为一个“副本”，这些“副本”除了一些描述性的信息（Pod 的名字、uid 等）不一样以外，其它信息都是一样的，譬如 Pod 内部的容器、容器数量、容器里面运行的应用等的这些信息都是一样的，这些副本提供同样的功能。  

（2）控制器  
· ReplicationController（RC）：Replication Controller 简称 RC，RC 是 Kubernetes 系统中的核心概念之一，简单来说，RC 可以保证在任意时间运行 Pod 的副本数量，能够保证 Pod 总是可用的。如果实际 Pod 数量比指定的多那就结束掉多余的，如果实际数量比指定的少就新启动一些Pod，当 Pod 失败、被删除或者挂掉后，RC 都会去自动创建新的 Pod 来保证副本数量，所以即使只有一个 Pod，我们也应该使用 RC 来管理我们的 Pod。可以说，通过 ReplicationController，Kubernetes 实现了 Pod 的高可用性。  
· ReplicaSet（RS）：RC （ReplicationController ）主要的作用就是用来确保容器应用的副本数始终保持在用户定义的副本数 。即如果有容器异常退出，会自动创建新的 Pod 来替代；而如果异常多出来的容器也会自动回收（已经成为过去时），在 v1.11 版本废弃。​Kubernetes 官方建议使用 RS（ReplicaSet ） 替代 RC （ReplicationController ） 进行部署，RS 跟 RC 没有本质的不同，只是名字不一样，并且 RS 支持集合式的 selector。label （标签）是附加到 Kubernetes 对象（比如 Pods）上的键值对，用于区分对象（比如Pod、Service）。 label 旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。 label 可以用于组织和选择对象的子集。label 可以在创建时附加到对象，随后可以随时添加和修改。可以像 namespace 一样，使用 label 来获取某类对象，但 label 可以与 selector 一起配合使用，用表达式对条件加以限制，实现更精确、更灵活的资源查找。
label 与 selector 配合，可以实现对象的“关联”，“Pod 控制器” 与 Pod 是相关联的 —— “Pod 控制器”依赖于 Pod，可以给 Pod 设置 label，然后给“控制器”设置对应的 selector，这就实现了对象的关联。  
· Deployment 为 Pod 和 Replica Set 提供声明式更新。

你只需要在 Deployment 中描述你想要的目标状态是什么，Deployment controller 就会帮你将 Pod 和 Replica Set 的实际状态改变到你的目标状态。你可以定义一个全新的 Deployment，也可以创建一个新的替换旧的 Deployment。
二、服务发现  

三、存储  

四、特殊类型配置  

五、其他  

#### 资源清单





















