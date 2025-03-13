![image](https://github.com/user-attachments/assets/2d4addb4-75f3-4d47-af92-b7815b15ff22)
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
#### 无状态：不需要在服务运行环境下存储数据，不操作磁盘相关服务
##### 代表应用：nginx，apache
##### 优点：对客户端透明、无依赖关系，可以高效实现扩展迁移
##### 缺点：不能存储数据，需要额外的数据服务支撑
#### 有状态：与无状态相反
##### 代表应用：MySQL，redis
##### 优点：可以独立存储数据，实现数据管理
##### 缺点：集群环境需要实现主从数据同步、备份、水平扩容复杂

### 资源和对象
| **资源控制器**            | **描述**                                                                                       | **使用场景**                                           |
|--------------------------|------------------------------------------------------------------------------------------------|------------------------------------------------------|
| **Pod**                   | 基本的 Kubernetes 资源单元，代表一个或多个容器的集合，通常只创建一个容器。                                      | 用于创建单一容器的基本应用，或者测试和调试。                           |
| **Deployment**            | 管理无状态应用的副本，确保特定数量的 Pod 副本运行，提供自动滚动更新和回滚功能。                        | 无状态应用的部署，如 Web 应用和前端服务。                             |
| **StatefulSet**           | 管理有状态应用的副本，为每个 Pod 提供稳定的网络标识符和持久存储卷，按顺序启动和停止 Pod。                 | 有状态应用，如数据库、消息队列、缓存系统。                          |
| **ReplicaSet**            | 确保指定数量的 Pod 副本始终运行，但不提供滚动更新功能，通常由 Deployment 创建和管理。                     | 用于确保指定数量的 Pod 副本运行，但较少直接使用。                     |
| **ReplicationController** | 确保指定数量的 Pod 副本始终运行，通常用于旧版本的 Kubernetes，功能类似于 ReplicaSet。                     | 较早的 Kubernetes 版本中使用，基本被 ReplicaSet 替代。                   |
| **DaemonSet**             | 确保每个节点上都运行指定的 Pod 副本，适用于需要在所有节点上运行的应用。                                | 节点级别的服务，如日志收集、监控代理、网络插件等。                    |
| **Job**                   | 用于管理批处理任务，确保任务按照指定数量执行完毕并成功终止。                                            | 一次性任务，如数据迁移、批处理、定时任务等。                         |
| **CronJob**               | 定期运行任务，类似于 Linux 中的 cron 作业，支持按时间调度任务。                                          | 定时任务，如备份、日志处理、定时数据更新等。                          |
| **HorizontalPodAutoscaler** | 根据 CPU 或其他指标自动调节 Pod 的副本数。                                                              | 自动扩展/缩减服务负载，如自动调节 Web 应用的副本数。                    |
| **PodDisruptionBudget**   | 确保在 Pod 被中断时（例如节点维护、升级等），保持一定数量的可用 Pod。                                    | 用于高可用应用，确保在维护过程中不影响服务的可用性。                    |
| **NetworkPolicy**         | 定义 Pod 间的网络访问规则，控制哪些 Pod 可以通信。                                                      | 网络安全和隔离，限制不同 Pod 之间的通信。                              |
| **PersistentVolume**      | 描述持久化存储资源，提供给 Pod 使用，通常与 PersistentVolumeClaim 配合使用。                                | 提供持久存储，确保数据在 Pod 重启后仍然保留。                           |
| **PersistentVolumeClaim** | 用户请求存储的资源，指定存储的大小和访问模式，并与 PersistentVolume 配合使用。                            | 申请存储资源，用于持久化数据存储。                                     |
| **StorageClass**          | 定义如何动态创建 PersistentVolume，用于描述不同类型的存储。                                               | 用于动态存储供应，指定不同的存储类型和配置。                          |
| **Service**               | 定义一组 Pod 的访问方式，提供负载均衡、服务发现等功能。                                                 | 对外暴露应用或服务，如 Web 服务或数据库服务。                          |
| **Ingress**               | 管理 HTTP 和 HTTPS 路由，允许外部访问集群内的服务。                                                    | 外部流量控制，如 Web 应用的路由和负载均衡。                           |
| **ConfigMap**             | 存储非机密的配置信息，允许将配置信息与 Pod 解耦。                                                     | 用于存储应用配置信息，如环境变量、配置文件等。                        |
| **Secret**                | 存储敏感信息，如密码、Token 等，避免明文存储。                                                        | 用于存储密码、密钥、令牌等敏感信息。                                   |
| **Namespace**             | 为资源创建虚拟集群，允许在同一物理集群内进行多租户隔离。                                               | 资源隔离，支持多租户或开发/生产环境的分离。                          |
| **ResourceQuota**         | 定义命名空间内资源使用的限制，如 CPU、内存、存储等的最大值。                                           | 控制资源使用，避免一个命名空间使用过多资源。                          |
| **LimitRange**            | 限制命名空间内 Pod、容器的资源请求和限制。                                                              | 控制资源分配，确保 Pod 和容器不会使用过多的资源。                      |
| **PodSecurityPolicy**     | 定义 Pod 的安全配置，控制 Pod 是否能访问敏感资源、是否可以使用特权模式等。                                | 加强安全性，控制 Pod 的权限和访问控制。                               |
| **ClusterRole**           | 定义集群范围的权限控制，允许对集群内所有命名空间的资源进行管理。                                          | 集群范围的权限控制，通常与 RoleBinding 配合使用。                      |
| **ClusterRoleBinding**    | 将 ClusterRole 绑定到用户或服务账户上，控制集群范围内的访问权限。                                        | 控制集群范围内的访问权限，允许用户或服务账户访问集群资源。             |
| **Role**                  | 定义命名空间范围内的权限控制，允许对命名空间内的资源进行管理。                                           | 命名空间范围的权限控制，通常与 RoleBinding 配合使用。                   |
| **RoleBinding**           | 将 Role 绑定到用户或服务账户上，控制命名空间内的访问权限。                                              | 控制命名空间内的访问权限，允许用户或服务账户访问命名空间资源。          |

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

1.1 副本（replicas）一个 Pod 可以被复制成多份，每一份可被称之为一个“副本”，这些“副本”除了一些描述性的信息（Pod 的名字、uid 等）不一样以外，其它信息都是一样的，譬如 Pod 内部的容器、容器数量、容器里面运行的应用等的这些信息都是一样的，这些副本提供同样的功能。  

1.2 控制器  

1.2.1 适用无状态服务  

1.2.1.1 ReplicationController（RC）：Replication Controller 简称 RC，RC 是 Kubernetes 系统中的核心概念之一，简单来说，RC 可以保证在任意时间运行 Pod 的副本数量，能够保证 Pod 总是可用的。如果实际 Pod 数量比指定的多那就结束掉多余的，如果实际数量比指定的少就新启动一些Pod，当 Pod 失败、被删除或者挂掉后，RC 都会去自动创建新的 Pod 来保证副本数量，所以即使只有一个 Pod，我们也应该使用 RC 来管理我们的 Pod。可以说，通过 ReplicationController，Kubernetes 实现了 Pod 的高可用性。  

![image](https://github.com/user-attachments/assets/51123021-0bed-456d-91a6-5a91fc3d3886)  



1.2.1.2 ReplicaSet（RS）：RC （ReplicationController ）主要的作用就是用来确保容器应用的副本数始终保持在用户定义的副本数 。即如果有容器异常退出，会自动创建新的 Pod 来替代；而如果异常多出来的容器也会自动回收（已经成为过去时），在 v1.11 版本废弃。​Kubernetes 官方建议使用 RS（ReplicaSet ） 替代 RC （ReplicationController ） 进行部署，RS 跟 RC 没有本质的不同，只是名字不一样，并且 RS 支持集合式的 selector。label （标签）是附加到 Kubernetes 对象（比如 Pods）上的键值对，用于区分对象（比如Pod、Service）。 label 旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。 label 可以用于组织和选择对象的子集。label 可以在创建时附加到对象，随后可以随时添加和修改。可以像 namespace 一样，使用 label 来获取某类对象，但 label 可以与 selector 一起配合使用，用表达式对条件加以限制，实现更精确、更灵活的资源查找。
label 与 selector 配合，可以实现对象的“关联”，“Pod 控制器” 与 Pod 是相关联的 —— “Pod 控制器”依赖于 Pod，可以给 Pod 设置 label，然后给“控制器”设置对应的 selector，这就实现了对象的关联。  

1.2.1.3 Deployment 为 Pod 和 Replica Set 提供声明式更新.你只需要在 Deployment 中描述你想要的目标状态是什么，Deployment controller 就会帮你将 Pod 和 Replica Set 的实际状态改变到你的目标状态。你可以定义一个全新的 Deployment，也可以创建一个新的替换旧的 Deployment。  

![image](https://github.com/user-attachments/assets/c14b8544-1315-4e56-ba54-c83c0d15f127)  

1.2.1.3.1 创建 Replica Set / Pod  

1.2.1.3.2 滚动升级/回滚  

1.2.1.3.3 平滑扩容和缩容  

1.2.1.3.4 暂停与恢复 Deployment  

1.2.2 适用有状态服务：StatefulSet 中每个 Pod 的 DNS 格式为 statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local；serviceName 为 Headless Service 的名字，0..N-1 为 Pod 所在的序号，从 0 开始到 N-1，statefulSetName 为 StatefulSet 的名字，namespace 为服务所在的 namespace，Headless Servic 和 StatefulSet 必须在相同的namespace，.cluster.local 为 Cluster Domain;StatefulSet 是用来管理有状态应用的工作负载 API 对象。StatefulSet 用来管理某 Pod 集合的部署和扩缩， 并为这些 Pod 提供持久存储和持久标识符。和 Deployment 类似， StatefulSet 管理基于相同容器规约的一组 Pod。但和 Deployment 不同的是， StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID。这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。如果希望使用存储卷为工作负载提供持久存储，可以使用 StatefulSet 作为解决方案的一部分。 尽管 StatefulSet 中的单个 Pod 仍可能出现故障， 但持久的 Pod 标识符使得将现有卷与替换已失败 Pod 的新 Pod 相匹配变得更加容易。  

![image](https://github.com/user-attachments/assets/7d437095-c5c6-4fef-a5b0-10f8b0057dfe)  

1.2.2.1 主要特点  

1.2.2.1.1 稳定的、唯一的网络标识符：稳定的网络标志，即 Pod 重新调度后其 PodName 和 HostName 不变，基于 Headless Service（即没有 Cluster IP 的 Service）来实现  

1.2.2.1.2 稳定的、持久的存储：即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 来实现  

1.2.2.1.3 有序的、优雅的部署和扩缩：有序部署，有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从 0到 N-1，在下一个Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于 init containers 来实现  

1.2.2.1.4 有序的、自动的滚动更新：有序收缩，有序删除（即从 N-1 到 0）  

1.2.2.2 组成  

1.2.2.2.1 Headless Service：用于定义网络标志（DNS domain）Domain Name Server：域名服务将域名与 ip 绑定映射关系；服务名 => 访问路径（域名） => ip  

1.2.2.2.2 volumeClaimTemplate：用于创建 PersistentVolumes持久化模板  

1.2.2.3 注意事项  

1.2.2.3.1 kubernetes v1.5 版本以上才支持  

1.2.2.3.2 所有Pod的Volume必须使用PersistentVolume或者是管理员事先创建好  

1.2.2.3.3 为了保证数据安全，删除StatefulSet时不会删除Volume  

1.2.2.3.4 StatefulSet 需要一个 Headless Service 来定义 DNS domain，需要在 StatefulSet 之前创建好  

1.2.3 守护进程：DaemonSet 保证在每个 Node 上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。  

![image](https://github.com/user-attachments/assets/70aa9eb1-063a-45c5-ac9d-0eddb6e77f6a)  



1.2.3.1 日志收集 比如 fluentd，logstash 等  

1.2.3.2 系统监控 比如 Prometheus Node Exporter，collectd，New Relic agent，Ganglia gmond 等  

1.2.3.3 系统程序 比如 kube-proxy, kube-dns, glusterd, ceph 等  

1.2.4 任务或定时任务  

1.2.4.1 Job：一次性任务，运行完成后Pod销毁，不再重新启动新容器。  

1.2.4.2 CronJob: CronJob 是在 Job 基础上加上了定时功能。  

二、服务发现  

![image](https://github.com/user-attachments/assets/58159446-5bdd-4dd9-a55b-c4502fb0ee64)  


1 service:“Service” 简写 “svc”。Pod 不能直接提供给外网访问，而是应该使用 service。Service 就是把 Pod 暴露出来提供服务，Service 才是真正的“服务”，它的中文名就叫“服务”。可以说 Service 是一个应用服务的抽象，定义了 Pod 逻辑集合和访问这个 Pod 集合的策略。Service 代理 Pod 集合，对外表现为一个访问入口，访问该入口的请求将经过负载均衡，转发到后端 Pod 中的容器。  

2 ingress: Ingress 可以提供外网访问 Service 的能力。可以把某个请求地址映射、路由到特定的 service。ingress 需要配合 ingress controller 一起使用才能发挥作用，ingress 只是相当于路由规则的集合而已，真正实现路由功能的，是 Ingress Controller，ingress controller 和其它 k8s 组件一样，也是在 Pod 中运行。  



三、存储
1 Volume:数据卷，共享 Pod 中容器使用的数据。用来放持久化的数据，比如数据库数据。  

2 Csi:Container Storage Interface 是由来自 Kubernetes、Mesos、Docker 等社区成员联合制定的一个行业标准接口规范，旨在将任意存储系统暴露给容器化应用程序。CSI 规范定义了存储提供商实现 CSI 兼容的 Volume Plugin 的最小操作集和部署建议。CSI 规范的主要焦点是声明 Volume Plugin 必须实现的接口。  


四、特殊类型配置  
1 ConfigMap:用来放配置，与 Secret 是类似的，只是 ConfigMap 放的是明文的数据，Secret 是密文存放。  

2 Secret:Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec 中。Secret 可以以 Volume 或者环境变量的方式使用。  

2.1 Service Account：用来访问 Kubernetes API，由 Kubernetes 自动创建，并且会自动挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中；  

2.2 Opaque：base64 编码格式的 Secret，用来存储密码、密钥等；  

2.3 kubernetes.io/dockerconfigjson：用来存储私有 docker registry 的认证信息。  

3 downwardAPI: downwardAPI 这个模式和其他模式不一样的地方在于它不是为了存放容器的数据也不是用来进行容器和宿主机的数据交换的，而是让 pod 里的容器能够直接获取到这个 pod 对象本身的一些信息。downwardAPI 提供了两种方式用于将 pod 的信息注入到容器内部：环境变量：用于单个变量，可以将 pod 信息和容器信息直接注入容器内部,volume 挂载：将 pod 信息生成为文件，直接挂载到容器内部中去.  

五、其他  
1 Role: Role 是一组权限的集合，例如 Role 可以包含列出 Pod 权限及列出 Deployment 权限，Role 用于给某个 Namespace 中的资源进行鉴权。  
2 RoleBinding ：将 Subject 绑定到 Role，RoleBinding 使规则在命名空间内生效。  


### 对象规约和状态
#### 规约（Spec）：“spec” 是 “规约”、“规格” 的意思，spec 是必需的，它描述了对象的期望状态（Desired State）—— 希望对象所具有的特征。当创建 Kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。  

#### 状态（Status）： 表示对象的实际状态，该属性由 k8s 自己维护，k8s 会通过一系列的控制器对对应对象进行管理，让对象尽可能的让实际状态与期望状态重合。  






# 实战

## 搭建k8s集群（kubeadm方案）

### 服务器要求 

三台服务器（centos7.9）

#### 配置

四核四G 20G硬盘容量

#### 软件环境 

Docker20 k8s-1.23.6

#### 安装步骤

一、初始操作  
1.关闭防火墙  

```shell
systemctl stop firewalld
systemctl disable firewalld
```

2.关闭selinux

```shell
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时
```

3.关闭swap

```shell
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久
```

4.在master添加hosts

```shell
cat >> /etc/hosts << EOF
192.168.30.161 k8s-master
192.168.30.162 k8s-node1
192.168.30.163 k8s-node2
EOF
```

5.将桥接的IPv4流量传递到iptables的链（该配置确保流经这些桥接设备的数据包会经过 iptables 规则处理，否则 Kubernetes 可能无法正确管理网络流量，如：
Pod 之间的网络通信可能失败。
Service 的 ClusterIP 或 NodePort 可能无法正确转发流量）

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

6.sysctl --system  # 生效

7.时间同步

```shell
yum install ntpdate -y
ntpdate time.windows.com
```

二、安装基础软件（所有节点）

1.安装docker 20+（这里我安装的Docker version 20.10.14, build a224086，docker版本过新的话 k8s 1.23.6会不适配报错）

2.添加阿里云镜像源 

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

3.安装kubeadm、kubelet、kubectl

```shell
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6

systemctl enable kubelet

# 配置关闭 Docker 的 cgroups，修改 /etc/docker/daemon.json，加入以下内容
"exec-opts": ["native.cgroupdriver=systemd"]

# 重启 docker
systemctl daemon-reload
systemctl restart docker
```

三、部署k8s master

1.在master节点下执行

```shell
kubeadm init \
      --apiserver-advertise-address=192.168.30.161 \
      --image-repository registry.aliyuncs.com/google_containers \
      --kubernetes-version v1.23.6 \
      --service-cidr=10.96.0.0/12 \
      --pod-network-cidr=10.244.0.0/16
```

初始化好后确认kubelet可用systemctl status kubelet

2.安装成功后，复制如下配置并执行（确保 kubectl 具备正确的配置文件，以便访问 Kubernetes API 服务器。
允许当前用户（非 root）使用 kubectl 运行 Kubernetes 命令。
验证 Kubernetes 节点的状态，检查集群是否正常运行。）

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

四、加入node1与node2

1.下方命令可以在 k8s master 控制台初始化成功后复制 join 命令（token与sha256在步骤二与步骤三获取）

```shell
kubeadm join 192.168.30.161:6443 --token 65bgi6.x5t814c192j2g3f2 --discovery-token-ca-cert-hash sha256:b833c6c86fa042470ecd9fd2833e1bc944567db97f2092d5d1fd18b40f151f2c
```

2.如果初始化的 token 不小心清空了，可以通过如下命令获取或者重新申请

```shell
# 如果 token 已经过期，就重新申请
kubeadm token create

# token 没有过期可以通过如下命令获取
kubeadm token list
```

3.获取 --discovery-token-ca-cert-hash 值，得到值后需要在前面拼接上 sha256:

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'
```

五、部署CNI网络插件

1.在 master 节点上执行（我这边在/opt/k8s）

```shell
# 下载 calico 配置文件，可能会网络超时
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O

# 修改 calico.yaml 文件中的 CALICO_IPV4POOL_CIDR 配置，修改为与初始化的 cidr 相同

# 修改 IP_AUTODETECTION_METHOD 下的网卡名称

# 删除镜像 docker.io/ 前缀，避免下载过慢导致失败
sed -i 's#docker.io/##g' calico.yaml

# 执行部署
kubectl apply -f calico.yaml
```

2.注意，我这里遇到一个问题，就是国内各大厂商的镜像源已经在24年6月份左右由于管制，所以现在一般的镜像都用不了了，我这边通过代理docker pull直接拉下来的，代理地址为 [swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io]()

```shell
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/cni:v3.25.0
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/cni:v3.25.0  docker.io/calico/cni:v3.25.0



docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/node:v3.25.0
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/node:v3.25.0  docker.io/calico/node:v3.25.0


docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/kube-controllers:v3.25.0
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/kube-controllers:v3.25.0  docker.io/calico/kube-controllers:v3.25.0



docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/upgrade-ipam:v3.25.0
docker tag  swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/upgrade-ipam:v3.25.0  docker.io/calico/upgrade-ipam:v3.25.0
```

3.或者用自己的可以用的镜像仓库，去calico.yaml文件中全局搜索docker，把地址替换一个可用的即可

4.安装好网络插件可查看集群状态
```shell
[root@master k8s]# kubectl get po -n kube-system  
NAME                                      READY   STATUS    RESTARTS       AGE
calico-kube-controllers-6d768559b-rmctg   1/1     Running   1 (21h ago)    22h
calico-node-458jq                         1/1     Running   1 (21h ago)    22h
calico-node-8d65m                         1/1     Running   1 (57m ago)    22h
calico-node-m7vn9                         1/1     Running   1 (57m ago)    22h
coredns-6d8c4cb4d-8fxt4                   1/1     Running   1 (21h ago)    23h
coredns-6d8c4cb4d-xpgpb                   1/1     Running   1 (21h ago)    23h
etcd-master                               1/1     Running   22 (21h ago)   26h
kube-apiserver-master                     1/1     Running   23 (21h ago)   26h
kube-controller-manager-master            1/1     Running   22 (21h ago)   26h
kube-proxy-2mnsw                          1/1     Running   21 (21h ago)   26h
kube-proxy-sqcwg                          1/1     Running   2 (57m ago)    26h
kube-proxy-vd2vp                          1/1     Running   2 (57m ago)    26h
kube-scheduler-master                     1/1     Running   22 (21h ago)   26h
```

```
[root@master ~]#  kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS    RESTARTS      AGE    IP               NODE     NOMINATED NODE   READINESS GATES
default       nginx-664975df4-ksfz2                     1/1     Running   0             33h    10.244.104.3     node2    <none>           <none>
kube-system   calico-kube-controllers-6d768559b-rmctg   1/1     Running   6             126d   10.244.219.84    master   <none>           <none>
kube-system   calico-node-929rq                         1/1     Running   0             13m    192.168.30.163   node2    <none>           <none>
kube-system   calico-node-9qz8b                         1/1     Running   0             12m    192.168.30.161   master   <none>           <none>
kube-system   calico-node-dgd9r                         1/1     Running   0             13m    192.168.30.162   node1    <none>           <none>
kube-system   coredns-6d8c4cb4d-8fxt4                   1/1     Running   6             126d   10.244.219.86    master   <none>           <none>
kube-system   coredns-6d8c4cb4d-xpgpb                   1/1     Running   6             126d   10.244.219.85    master   <none>           <none>
kube-system   etcd-master                               1/1     Running   10            126d   192.168.30.161   master   <none>           <none>
kube-system   kube-apiserver-master                     1/1     Running   3             126d   192.168.30.161   master   <none>           <none>
kube-system   kube-controller-manager-master            1/1     Running   1             126d   192.168.30.161   master   <none>           <none>
kube-system   kube-proxy-2mnsw                          1/1     Running   1             126d   192.168.30.161   master   <none>           <none>
kube-system   kube-proxy-sqcwg                          1/1     Running   4 (33h ago)   126d   192.168.30.163   node2    <none>           <none>
kube-system   kube-proxy-vd2vp                          1/1     Running   4 (33h ago)   126d   192.168.30.162   node1    <none>           <none>
kube-system   kube-scheduler-master                     1/1     Running   9             126d   192.168.30.161   master   <none>           <none>
```
如果有长时间pending或者err的话，可以产看一下pod的详细信息
```shell
 kubectl describe po calico-node-458jq -n kube-system
Name:                 calico-node-458jq
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 master/192.168.30.161
Start Time:           Wed, 02 Oct 2024 19:57:37 +0800
Labels:               controller-revision-hash=8bb9f5d9d
                      k8s-app=calico-node
                      pod-template-generation=2
Annotations:          <none>
Status:               Running
IP:                   192.168.30.161
IPs:
  IP:           192.168.30.161
Controlled By:  DaemonSet/calico-node
```


六、测试集群

```shell
# 创建部署
kubectl create deployment nginx --image=nginx

kubectl create secret docker-registry myregistrykey --docker-server=swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/

# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看 pod 以及服务信息
kubectl get pod,svc

# 如果有问题也可通过查看详细日志
journalctl -u docker
journalctl -u docker

kubectl logs nginx-664975df4-wvk8v

```


![image](https://github.com/user-attachments/assets/e07e6bc8-dcfd-4337-a1e4-bc8ad0e58740)


![image](https://github.com/user-attachments/assets/a670a876-182c-4488-9822-3335870c7161)

## 命令行工具 (kubectl)Kubernetes 提供 kubectl 是使用 Kubernetes API 与 Kubernetes 集群的控制面进行通信的命令行工具，实际上也是通过apiserver对node进行控制，
```
[root@master .kube]# cat ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJME1UQXdNakEzTVRrME4xb1hEVE0wTURrek1EQTNNVGswTjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTFRCCnZnMXIwaWNwUnB3VWRQVW15YkJ0UTRUdDR6WnJrQjA5d05xVWdWcnByaUk3WldFTGFYc0EyVVNhNEpZSW4rR0UKNjlBZy9VeTJXSjVBaHE3SWhZYUZubGNEcWNhSHhXd2pvakhtMDVGVllqZVhUb3ZOQ2dlUlB1REZYUEdsM0dsQgpkWlNNS3ZLYjVIU0h1aHluUktRbU1abGhTWmt5c1hneXhSU2FCdmRSaTByU3hwcStUeUFaTFA3MVVnNXVMYmx1CkpLTm9taUFtVHdmRFE2eVIwOVJtWUp6bDZ2enIxazNwU1ZSUlIzbU5kSGJPMmVaK3QvOVFFL1JIUXFUMThkM0kKTG9yV3RNeE9zVnZLd3dlR1ZRRVVMaHRNWGY0Y0VXR3AxRFFaY09RalNCTisrZzR2bjY3aFEwZEN3OWdMdnR1eQpla2R2SVlZTjdwcE5nUTVIdDVFQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZJT0pnQU1RcndmMWZwMXQ1N3ZQVWJpK042d0lNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBRGxubFlrRWlGT2dYS3oxVTFRbgpsaHF4MGxUTFNUYkhYYm9vMFJwWVBpR1J5ZlRPYWRQUU5uQXhLVllFUjZUbTlVeDhuNU9WY3hlN3VSOTRQeHZlCjBpeEpWTGRTQ1hTcEVCU01CY2p3c1NyRlAvblFueGpCemtPV3owZ0ZxeDR2WVduVS9MeFVDQW5rck5uOWNDSS8KQlFCNTNFOHV3dlN0V1FXTndrZnBERXh1QkNMWnVId2gySDdLNVlxdTlROVJ6K1kvUFFhRFQ3cXBqWnI2RjU4cAp2OXU0dHFzakh2RVdvWW4zWXJhV09OdTI3YWNNaDhxT2NqU3NPMElqc1VERnV6ektpaW45Wjl5S0tGUG9NNHhWCm5yOWR5d0p3cTUzSkViSC9RVHhXYmMxaXpxQXc0MngzNkw5b1Noc0M0WllOK2lkRkhGT2xVY1hoais0WVdJVncKZGtrPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.30.161:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJVERNTU9mUkZTWll3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRFd01ESXdOekU1TkRkYUZ3MHlOVEV3TURJd056RTVORGhhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTJFS3I5SE16RFJQb3pPRC8Kblh6dWRyandYbGlWek1VUFhTcmh3TDZWdzNmSkdaY2RZNStxT0lCVDhGeW9veEJKYVhKK1YyS3prMUd0d1dLWQpLM3lKdCt5T0dMdS9hSndYU1krQmlZc0pVeXFRbzJXbUdORlYyaUZpWUJUMGNSa0ppTmc1QjJHb2kwUGFmL3FTClV6ZnJIVi9nMmhhNDZ2YzJ0azVJTTJUOUdoZGsrM0owTjZCZXUwYktGOTdibCsxWERWQmt4Q01wbkRvQXJteXMKRk5tRlNtemErd0hmU3F4eWVObElldUhhZ3U5bTA5b1BFaTVHSzlOeDRSbU9LbVdvTHNDZkNiTUE4TE1BWkg5QwoxOGc0bUhCSEVJWURBVWQ5M3FrczVKWEk5cDdUS21SZm5qaFpXN3lWdUE0czVZWTljYm56RFN2L1c1K0E1dU5ZCitiOEhPd0lEQVFBQm8xWXdWREFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RBWURWUjBUQVFIL0JBSXdBREFmQmdOVkhTTUVHREFXZ0JTRGlZQURFSzhIOVg2ZGJlZTd6MUc0dmplcwpDREFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBSk5aVGp2TVJHcXhqbXIzVGFOcVU4TVpWT0VBNzZLZ2NzOVE1CkhOdytKMEwraW11ajA4TzU3MkVtY09XdWtXTWthSVJkcUs5ZUNJY2I2ZDRkRjE5NDBlaURjUkk0aUpBbU90dWMKQTk1TWp2WHRHZGJDaFRUWGF6dGV4Njlub2VlT0RBNmsxbGVua3VKWGFZZzByd2crclZ2d1lQUmpVS0ZrSUd2QQpmQVA5ZW9LamNROFM5Mmo5WkloaHJkOElXVjkxNm1WbmZVTm1ZMWxIUVpSZ2VnbnN2K1ppMUxEWWJ6aWFsck1ECjZMUEExaVIxUGRmZVgyK1krU3JlcXhRQzUzdExCbWw2YTkyNjdaWjdTakFRSGlLRnRHdlozSm9LWEI3TWhqdEoKMEc3MmJTeGRudTJwQU1nSVZsc0NjUERjSm1OdW9hZGRyeG8wU3NYeXh4OCtINWo1UHc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBMkVLcjlITXpEUlBvek9EL25YenVkcmp3WGxpVnpNVVBYU3Jod0w2VnczZkpHWmNkClk1K3FPSUJUOEZ5b294QkphWEorVjJLemsxR3R3V0tZSzN5SnQreU9HTHUvYUp3WFNZK0JpWXNKVXlxUW8yV20KR05GVjJpRmlZQlQwY1JrSmlOZzVCMkdvaTBQYWYvcVNVemZySFYvZzJoYTQ2dmMydGs1SU0yVDlHaGRrKzNKMApONkJldTBiS0Y5N2JsKzFYRFZCa3hDTXBuRG9Bcm15c0ZObUZTbXphK3dIZlNxeHllTmxJZXVIYWd1OW0wOW9QCkVpNUdLOU54NFJtT0ttV29Mc0NmQ2JNQThMTUFaSDlDMThnNG1IQkhFSVlEQVVkOTNxa3M1SlhJOXA3VEttUmYKbmpoWlc3eVZ1QTRzNVlZOWNibnpEU3YvVzUrQTV1TlkrYjhIT3dJREFRQUJBb0lCQVFDTXkvdUFEM0J4VjBEKwpBbzdwVGVsRFNtelVRQUJuYlBUdngzZHJNYW4zdFFrc3JBSHFWbzFRYzl3eVpXRTFxT2ppeWpRUFdLZHBob2dGCm05ZE9tS3BoSUpYeTdHbFdCaW56TG9jN3NWWEUyN1dIYkNxVGhkYkxTV2p1L1RXWVhLQ2tnNEc5bUl0UEFFSkYKaURHMEZIZnlpL1dzaDVhbEE0YVBjcS8zSmd6UzZxMGg1clJrbGQvNEZvNzZTSkJrci9OVzcrWlB1eFhJTzZaNApEUW96S1h0VUZITVVHdXlqcXBhcHVySE1NYytYdXBlQytkZzFNUHZZS3dTQmhiN1h4ditKWEh2ejlIeXVtcXZTCkxvc1dlTjNVM0ZzQXh0Qm94V2xGRXZqeHFKQUcxRGYveUt2SFMvYVNyc2pFY3Y2WW41RmhvKzRWYjlJSkNiNVEKbHlIT3JtNmhBb0dCQVB5UmxMK1hvcWE1ZTUrVnhOcjkzb05LTjVrYzB4MjROUE53NVZ0cE00YjQzODZkaFdBQgpGdEtGWVhvbzZkU2V4OXVSV2dqZEZDWFlFZjVmd2IzUGpXc254b2lkcng2S0k1bnVQbGFuWGZtbW90ZUlGSlJ5Ck1USWk3bC9JcW9jTTROR2JoczVLWm1NclNQZTBVTGp6QTFQdVlxM0huTHBabUJiUm14TGJFV00vQW9HQkFOc3kKMEE5ckcyMHdoMnY1SGRta3ZFc3FRQTBCZURBb1hHcXgrZE4zRjRyK0o2Z1JxU0dnZDJWUnFDclI2VVBHcTVlbwpjTlp1UXRqSFMzMm5XQ1hPbXlQeUpUcjRYREtpdE1iS01GVGVKVmVhKyt4TVhMQjh3OFl2dzcybVZtMlNRM0x1Ck5lTTJpdW1sVWVDcVJiUFB4WCtBU1A0aE5wNlhpNjJhWDV4Y3ZTa0ZBb0dBY21XdUxpbU1ibC9NOHJkdmRwRk8KVzhFZDlhZnNwNlZydG1nSU9xTW54NWFxS0hlSWxiZG9rdW4vQU1uUFA1SzdpMlFHbDhVcS93a2kvVWg2QkhNaAo1c3NaVFgrK2RlS3p4V0QzczJBVFhLUnhWWlk1WEJOczNQeWRZNTBNUUNkQkhTK0ltNTl2U0xPdVZTUEMvRUoxCjIybzZIK1F0eE9vWHpSNGJVeXNPY1JrQ2dZQjBoV0Jnc2RrVWhCV1k2Z1phS2Q0R1B1RnBpSHh4YlNNamZKU0gKT3VtQzgzUDFQZDRnaUFLd0UyWkh6T29wSXpVWUcyeFFNTERNTjdVRGlLK3MrVlV6R0lkOS80UlRUbmEycmNoZgpkTzk0MEdSV3lva0RNRytKck41cXREK0JZNTBEUFduYjdLU1BhMWhKQzNxZUNUYTlmbDVPNlN6MXhTMTFEWGtCCno3S21XUUtCZ1FEc2NQNFMxWWdNeVRYZk5pdHovWXc0alozeElMUGcrU0xpRXZrMkJYakpqSzhPbWYvWHl6WmQKRllTLzBNN1Q2VFVQcks1U0YyaGJ4NW10V3FNNkd0THVqT3BTWEdxcHZETzMwWHhtY0Y3eW8xc0dvMTJ0WXV1bgpXTko1d3p0bEtxWmZIOVZnblpEamZSZ3F6d08xMGxiOXVLOGRwbUd1b3NPMktYc0VzekVpQlE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```



### 在任意节点使用kubectl
#### 将 master 节点中 /etc/kubernetes/admin.conf 拷贝到需要运行的服务器的 /etc/kubernetes 目录中 scp /etc/kubernetes/admin.conf root@192.168.30.162:/etc/kubernetes
#### 在对应的服务器上配置环境变量 echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile 配置生效source ~/.bash_profile
```
[root@node1 ~]# kubectl get po
NAME                    READY   STATUS    RESTARTS      AGE
nginx-664975df4-ksfz2   1/1     Running   1 (14m ago)   2d8h
[root@node1 ~]#
```

### 资源操作
#### 创建对象
```
$ kubectl create -f ./my-manifest.yaml           # 创建资源
$ kubectl create -f ./my1.yaml -f ./my2.yaml     # 使用多个文件创建资源
$ kubectl create -f ./dir                        # 使用目录下的所有清单文件来创建资源
$ kubectl create -f https://git.io/vPieo         # 使用 url 来创建资源
$ kubectl run nginx --image=nginx                # 启动一个 nginx 实例
$ kubectl explain pods,svc                       # 获取 pod 和 svc 的文档

# 从 stdin 输入中创建多个 YAML 对象
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# 创建包含几个 key 的 Secret
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo "s33msi4" | base64)
  username: $(echo "jane" | base64)
EOF


```

#### 显示和查找资源
```
# Get commands with basic output
$ kubectl get services                          # 列出所有 namespace 中的所有 service
$ kubectl get pods --all-namespaces             # 列出所有 namespace 中的所有 pod
$ kubectl get pods -o wide                      # 列出所有 pod 并显示详细信息
$ kubectl get deployment my-dep                 # 列出指定 deployment
$ kubectl get pods --include-uninitialized      # 列出该 namespace 中的所有 pod 包括未初始化的

# 使用详细输出来描述命令
$ kubectl describe nodes my-node
$ kubectl describe pods my-pod

$ kubectl get services --sort-by=.metadata.name # List Services Sorted by Name

# 根据重启次数排序列出 pod
$ kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# 获取所有具有 app=cassandra 的 pod 中的 version 标签
$ kubectl get pods --selector=app=cassandra rc -o \
  jsonpath='{.items[*].metadata.labels.version}'

# 获取所有节点的 ExternalIP
$ kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# 列出属于某个 PC 的 Pod 的名字
# “jq”命令用于转换复杂的 jsonpath，参考 https://stedolan.github.io/jq/
$ sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
$ echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# 查看哪些节点已就绪
$ JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# 列出当前 Pod 中使用的 Secret
$ kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

```
#### 更新资源
```
$ kubectl rolling-update frontend-v1 -f frontend-v2.json           # 滚动更新 pod frontend-v1
$ kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2  # 更新资源名称并更新镜像
$ kubectl rolling-update frontend --image=image:v2                 # 更新 frontend pod 中的镜像
$ kubectl rolling-update frontend-v1 frontend-v2 --rollback        # 退出已存在的进行中的滚动更新
$ cat pod.json | kubectl replace -f -                              # 基于 stdin 输入的 JSON 替换 pod

# 强制替换，删除后重新创建资源。会导致服务中断。
$ kubectl replace --force -f ./pod.json

# 为 nginx RC 创建服务，启用本地 80 端口连接到容器上的 8000 端口
$ kubectl expose rc nginx --port=80 --target-port=8000

# 更新单容器 pod 的镜像版本（tag）到 v4
$ kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

$ kubectl label pods my-pod new-label=awesome                      # 添加标签
$ kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # 添加注解
$ kubectl autoscale deployment foo --min=2 --max=10                # 自动扩展 deployment “foo”


```

#### 修补资源

```
$ kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}' # 部分更新节点

# 更新容器镜像； spec.containers[*].name 是必须的，因为这是合并的关键字
$ kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# 使用具有位置数组的 json 补丁更新容器镜像
$ kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# 使用具有位置数组的 json 补丁禁用 deployment 的 livenessProbe
$ kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'


```
#### 编辑资源

```
$ kubectl edit svc/docker-registry                      # 编辑名为 docker-registry 的 service
$ KUBE_EDITOR="nano" kubectl edit svc/docker-registry   # 使用其它编辑器

```
#### scale 资源
```
$ kubectl scale --replicas=3 rs/foo                                 # Scale a replicaset named 'foo' to 3
$ kubectl scale --replicas=3 -f foo.yaml                            # Scale a resource specified in "foo.yaml" to 3
$ kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # If the deployment named mysql's current size is 2, scale mysql to 3
$ kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # Scale multiple replication controllers
```

#### 删除资源
```
$ kubectl delete -f ./pod.json                                              # 删除 pod.json 文件中定义的类型和名称的 pod
$ kubectl delete pod,service baz foo                                        # 删除名为“baz”的 pod 和名为“foo”的 service
$ kubectl delete pods,services -l name=myLabel                              # 删除具有 name=myLabel 标签的 pod 和 serivce
$ kubectl delete pods,services -l name=myLabel --include-uninitialized      # 删除具有 name=myLabel 标签的 pod 和 service，包括尚未初始化的
$ kubectl -n my-ns delete po,svc --all                                      # 删除 my-ns namespace 下的所有 pod 和 serivce，包括尚未初始化的


```

### Pod 与集群
#### 与运行的 Pod 交互
```
$ kubectl logs my-pod                                 # dump 输出 pod 的日志（stdout）
$ kubectl logs my-pod -c my-container                 # dump 输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
$ kubectl logs -f my-pod                              # 流式输出 pod 的日志（stdout）
$ kubectl logs -f my-pod -c my-container              # 流式输出 pod 中容器的日志（stdout，pod 中有多个容器的情况下使用）
$ kubectl run -i --tty busybox --image=busybox -- sh  # 交互式 shell 的方式运行 pod
$ kubectl attach my-pod -i                            # 连接到运行中的容器
$ kubectl port-forward my-pod 5000:6000               # 转发 pod 中的 6000 端口到本地的 5000 端口
$ kubectl exec my-pod -- ls /                         # 在已存在的容器中执行命令（只有一个容器的情况下）
$ kubectl exec my-pod -c my-container -- ls /         # 在已存在的容器中执行命令（pod 中有多个容器的情况下）
$ kubectl top pod POD_NAME --containers               # 显示指定 pod 和容器的指标度量
```
#### 与节点和集群交互
```
$ kubectl cordon my-node                                                # 标记 my-node 不可调度
$ kubectl drain my-node                                                 # 清空 my-node 以待维护
$ kubectl uncordon my-node                                              # 标记 my-node 可调度
$ kubectl top node my-node                                              # 显示 my-node 的指标度量
$ kubectl cluster-info                                                  # 显示 master 和服务的地址
$ kubectl cluster-info dump                                             # 将当前集群状态输出到 stdout                                    
$ kubectl cluster-info dump --output-directory=/path/to/cluster-state   # 将当前集群状态输出到 /path/to/cluster-state

# 如果该键和影响的污点（taint）已存在，则使用指定的值替换
$ kubectl taint nodes foo dedicated=special-user:NoSchedule
```

### 资源类型与别名

#### pods->po, deployments->deploy,services->svc,namespace->ns,nodes->no

### 格式化输出

#### 输出 json 格式 -o json

#### 仅打印资源名称 -o name

#### 以纯文本格式输出所有信息 -o wide

#### 输出 yaml 格式 -o yaml






## 深入pod

### 探针
#### 类型
##### StartupProbe k8s 1.16 版本新增的探针，用于判断应用程序是否已经启动了。当配置了 startupProbe 后，会先禁用其他探针，直到 startupProbe 成功后，其他探针才会继续。作用：由于有时候不能准确预估应用一定是多长时间启动成功，因此配置另外两种方式不方便配置初始化时长来检测，而配置了 statupProbe 后，只有在应用启动成功了，才会执行另外两种探针，可以更加方便的结合使用另外两种探针使用。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 20
```
##### LivenessProbe 用于探测容器中的应用是否运行，如果探测失败，kubelet 会根据配置的重启策略进行重启，若没有配置，默认就认为容器启动成功，不会执行重启策略。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 60
      periodSeconds: 10
      failureThreshold: 5
      successThreshold: 1
      timeoutSeconds: 5
```
##### ReadinessProbe 用于探测容器内的程序是否健康，它的返回值如果返回 success，那么就认为该容器已经完全启动，并且该容器是可以接收外部流量的。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    readinessProbe:
      httpGet:
        path: /ready
        port: 8181
        scheme: HTTP
      periodSeconds: 10 # 探测间隔时间
      failureThreshold: 3 # 连续失败次数
      successThreshold: 1 # 连续成功次数
      timeoutSeconds: 1 # 探测超时时间
```
#### 探测方式
##### ExecAction 在容器内部执行一个命令，如果返回值为 0，则任务容器时健康的。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    livenessProbe:
      exec:
        command:
          - cat
          - /health
      initialDelaySeconds: 10  # 容器启动后等待 10 秒再开始探测
      periodSeconds: 5         # 每 5 秒探测一次
      failureThreshold: 3      # 失败 3 次后重启容器
      successThreshold: 1      # 成功 1 次就认为健康
      timeoutSeconds: 2        # 每次探测的超时时间
```
##### TCPSocketAction 通过 tcp 连接监测容器内端口是否开放，如果开放则证明该容器健康
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 10  # 容器启动后等待 10 秒再开始探测
      periodSeconds: 5         # 每 5 秒探测一次
      failureThreshold: 3      # 失败 3 次后认为容器不健康
      successThreshold: 1      # 成功 1 次就认为健康
      timeoutSeconds: 2        # 每次探测的超时时间
```
##### HTTPGetAction 生产环境用的较多的方式，发送 HTTP 请求到容器内的应用程序，如果接口返回的状态码在 200~400 之间，则认为容器健康。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: my-image
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
        scheme: HTTP
        httpHeaders:
          - name: xxx
            value: xxx
      initialDelaySeconds: 10  # 容器启动后等待 10 秒再开始探测
      periodSeconds: 5         # 每 5 秒探测一次
      failureThreshold: 5      # 失败 5 次后认为容器不健康
      successThreshold: 1      # 成功 1 次就认为健康
      timeoutSeconds: 3        # 每次探测的超时时间
```
#### 如果不知道探针这种如何配置，可以找一个已有的查看其配置是如何配置的
```
[root@master k8sfile]# kubectl get pod -n kube-system
NAME                                      READY   STATUS    RESTARTS        AGE
calico-kube-controllers-6d768559b-rmctg   1/1     Running   10 (123m ago)   128d
calico-node-929rq                         1/1     Running   2 (127m ago)    47h
calico-node-9qz8b                         1/1     Running   4 (123m ago)    47h
calico-node-dgd9r                         1/1     Running   2 (127m ago)    47h
coredns-6d8c4cb4d-8fxt4                   1/1     Running   10 (123m ago)   128d
coredns-6d8c4cb4d-xpgpb                   1/1     Running   10 (123m ago)   128d
etcd-master                               1/1     Running   15 (123m ago)   128d
kube-apiserver-master                     1/1     Running   8 (123m ago)    128d
kube-controller-manager-master            1/1     Running   5 (123m ago)    128d
kube-proxy-2mnsw                          1/1     Running   5 (123m ago)    128d
kube-proxy-sqcwg                          1/1     Running   6 (127m ago)    128d
kube-proxy-vd2vp                          1/1     Running   6 (127m ago)    128d
kube-scheduler-master                     1/1     Running   14 (123m ago)   128d
[root@master k8sfile]# kubectl get deploy -n kube-system
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           128d
coredns                   2/2     2            2           128d
[root@master k8sfile]# kubectl edit deploy coredns -n kube-system
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-10-02T07:19:56Z"
  generation: 1
  labels:
    k8s-app: kube-dns
  name: coredns
  namespace: kube-system
  resourceVersion: "104875"
  uid: 2b3ce74a-2e07-4d7e-a06d-98d17b8d041a
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kube-dns
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        k8s-app: kube-dns
    spec:
      containers:
      - args:
        - -conf
        - /etc/coredns/Corefile
        image: registry.aliyuncs.com/google_containers/coredns:v1.8.6
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        name: coredns
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /ready
            port: 8181
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        securityContext:
```

#### 测试探针 kubectl create -f test2.yaml
```
apiVersion: apps/v1  # API 版本
kind: Deployment  # 资源类型
metadata:  # 元数据
  name: nginx-demo2  # Deployment 名称
  namespace: default  # 命名空间
  labels:  # 自定义标签
    type: app
    test: 1.0.0
spec:  # Deployment 规范
  replicas: 1  # Pod 副本数
  selector:
    matchLabels:
      type: app  # Selector 匹配标签
  template:  # Pod 模板
    metadata:
      labels:
        type: app  # Pod 标签
        test: 1.0.0
    spec:  # Pod 规范
      containers:  # 容器描述
      - name: nginx  # 容器名称
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest  # 镜像
        imagePullPolicy: IfNotPresent  # 镜像拉取策略
        # startupProbe:  # 启动探针
        #   httpGet: 
        #     path: /index.html
        #     port: 80
        #   failureThreshold: 3  # 失败 3 次认为启动失败
        #   periodSeconds: 10  # 探测间隔
        #   successThreshold: 1  # 成功 1 次认为启动成功
        #   timeoutSeconds: 5  # 超时时间
        livenessProbe:  # 存活探针
          tcpSocket:
            port: 80  # 通过 TCP 连接端口 80 检查存活状态
          failureThreshold: 3
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        # readinessProbe:  # 就绪探针
        #   exec:
        #     command:
        #       - sh
        #       - -c
        #       - "sleep 5; echo success > /inited"
          failureThreshold: 3
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        command:
        - nginx
        - -g
        - 'daemon off;'  # 启动命令
        workingDir: /usr/share/nginx/html  # 工作目录
        ports:
        - name: http  # 端口名称
          containerPort: 80  # 容器端口
          protocol: TCP  # 协议
        env:  # 环境变量
        - name: JVM_OPTS
          value: '-Xms128m -Xmx128m'  # 设置 JVM 内存选项
        resources:
          requests:  # 最低资源要求
            cpu: 100m
            memory: 128Mi
          limits:  # 最大资源限制
            cpu: 200m
            memory: 256Mi
```
启动起来，然后通过livenessProbe404回一直重启，中间把kubectl cp index222.html nginx-demo2-d798dccff-rfphr:/usr/share/nginx/html/ index222.html 拷贝到容器中，进而实现一开始一直尝试，直到不在404启动成功

```
[root@master k8sfile]# kubectl get po
NAME                          READY   STATUS    RESTARTS        AGE
nginx-demo2-d798dccff-rfphr   1/1     Running   2 (3m50s ago)   5m10s

---------------------------可以看到上面重启了两次，后发现了index222.html，就不再重启了

apiVersion: apps/v1  # API 版本
kind: Deployment  # 资源类型
metadata:  # 元数据
  name: nginx-demo2  # Deployment 名称
  namespace: default  # 命名空间
  labels:  # 自定义标签
    type: app
    test: 1.0.0
spec:  # Deployment 规范
  replicas: 1  # Pod 副本数
  selector:
    matchLabels:
      type: app  # Selector 匹配标签
  template:  # Pod 模板
    metadata:
      labels:
        type: app  # Pod 标签
        test: 1.0.0
    spec:  # Pod 规范
      containers:  # 容器描述
      - name: nginx  # 容器名称
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest  # 镜像
        imagePullPolicy: IfNotPresent  # 镜像拉取策略
        startupProbe:  # 启动探针
          exec: 
            command: 
            - sh
            - -c
            - "sleep 3; echo success > /inited"
          failureThreshold: 3  # 失败 3 次认为启动失败
          periodSeconds: 10  # 探测间隔
          successThreshold: 1  # 成功 1 次认为启动成功
          timeoutSeconds: 5  # 超时时间
        livenessProbe:  # 存活探针
          httpGet: 
            path: /index222.html 404
            port: 80  # 通过 TCP 连接端口 80 检查存活状态
          failureThreshold: 3
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        command:
        - nginx
        - -g
        - 'daemon off;'  # 启动命令
        workingDir: /usr/share/nginx/html  # 工作目录
        ports:
        - name: http  # 端口名称
          containerPort: 80  # 容器端口
          protocol: TCP  # 协议
        env:  # 环境变量
        - name: JVM_OPTS
          value: '-Xms128m -Xmx128m'  # 设置 JVM 内存选项
        resources:
          requests:  # 最低资源要求
            cpu: 100m
            memory: 128Mi
          limits:  # 最大资源限制
            cpu: 200m
            memory: 256Mi
```


### Pod生命周期
![image](https://github.com/user-attachments/assets/15e7d2f5-da8c-4550-98e6-31732a18dad5)
#### Pod退出流程-删除操作
##### Endpoint 删除 pod 的 ip 地址
##### Pod 变成 Terminating 状态(变为删除中的状态后，会给 pod 一个宽限期，让 pod 去执行一些清理或销毁操作。)
```
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-life # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  terminationGracePeriodSeconds: 30 # 当pod删除时设置的宽限时间
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    lifecycle: # 生命周期配置
      postStart: 
        exec: 
          command: 
            - sh
            - -c
            - "echo '<h1>pre stop</h1>' > /usr/share/nginx/html/prestop.html"
      preStop: 
        exec: 
          command: 
            - sh
            - -c
            - "sleep 50; echo 'sleep finished...' >> /usr/share/nginx/html/prestop.html"
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    resources:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启

-------------------------------------------------------------
[root@master ~]# kubectl get po -w
NAME         READY   STATUS    RESTARTS   AGE
nginx-life   1/1     Running             0          2s
nginx-life   1/1     Terminating         0          12s
nginx-life   1/1     Terminating         0          42s
nginx-life   0/1     Terminating         0          44s
nginx-life   0/1     Terminating         0          44s
nginx-life   0/1     Terminating         0          44s

```
##### 执行 preStop 的指令


#### Pod退出流程-删除操作(但是需要注意，由于 k8s 默认给 pod 的停止宽限时间为 30s，如果我们停止操作会超过 30s 时，不要光设置 sleep 50，还要将 terminationGracePeriodSeconds: 30 也更新成更长的时间，否则 k8s 最多只会在这个时间的基础上再宽限几秒，不会真正等待 50s)
##### 注册中心下线
##### 数据清理
##### 数据销毁



## 资源调度

### Label 和 Selector
#### 标签（Label）在各类资源的 metadata.labels 中进行配置
```
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-life # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  terminationGracePeriodSeconds: 30 # 当pod删除时设置的宽限时间
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    lifecycle: # 生命周期配置
      postStart: 
        exec: 
          command: 
            - sh
            - -c
            - "echo '<h1>pre stop</h1>' > /usr/share/nginx/html/prestop.html"
      preStop: 
        exec: 
          command: 
            - sh
            - -c
            - "sleep 50; echo 'sleep finished...' >> /usr/share/nginx/html/prestop.html"
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    resources:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启

------------------------------------------------------------------------
[root@master k8sfile]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-life   1/1     Running   0          7s    test=1.0.0,type=app
```
##### 临时创建label
```
[root@master k8sfile]# kubectl label pod nginx-life author=rlw
pod/nginx-life labeled
[root@master k8sfile]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE    LABELS
nginx-life   1/1     Running   0          6m6s   author=rlw,test=1.0.0,type=app
```

##### 修改已经存在的标签
```
[root@master k8sfile]# kubectl label pod nginx-life author=rlw2
error: 'author' already has a value (rlw), and --overwrite is false
[root@master k8sfile]# kubectl label pod nginx-life author=rlw2 --overwrite
pod/nginx-life labeled
[root@master k8sfile]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE    LABELS
nginx-life   1/1     Running   0          8m8s   author=rlw2,test=1.0.0,type=app
```
##### 查看label
```
[root@master k8sfile]# kubectl get po -A -l author=rlw2
NAMESPACE   NAME         READY   STATUS    RESTARTS   AGE
default     nginx-life   1/1     Running   0          10m
[root@master k8sfile]# kubectl  get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-life   1/1     Running   0          10m   author=rlw2,test=1.0.0,type=app
```
#### 选择器（Selector）
```
[root@master k8sfile]#
------------------------------------------------------------------------------
[root@master k8sfile]# kubectl  get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-life   1/1     Running   0          10m   author=rlw2,test=1.0.0,type=app
[root@master k8sfile]# kubectl get po -l author!=rlw2,test=1.0.1
No resources found in default namespace.
[root@master k8sfile]# kubectl get po -l author!=rlw2,test=1.0.0
No resources found in default namespace.
[root@master k8sfile]# kubectl get po -l author!=rlw,test=1.0.0
NAME         READY   STATUS    RESTARTS   AGE
nginx-life   1/1     Running   0          16m
[root@master k8sfile]# kubectl get po -l 'author!=rlw,test=1.0.0,type in (app,app2)'
NAME         READY   STATUS    RESTARTS   AGE
nginx-life   1/1     Running   0          17m
```


### Deployment

#### 创建
```
[root@master deployments]# kubectl create deploy nginx-deploy --image=swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
deployment.apps/nginx-deploy created

[root@master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           4m36s
[root@master deployments]# kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-7c94c74f9d   1         1         1       4m41s
[root@master deployments]# kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-7c94c74f9d-sd2qn   1/1     Running   0          4m44s

#提示->从pod名称可以看出关联关系
```

```
apiVersion: apps/v1 # deployment api 版本
kind: Deployment # 资源类型为 deployment
metadata: # 元信息
  labels: # 标签
    app: nginx-deploy # 具体的 key: value 配置形式
  name: nginx-deploy # deployment 的名字
  namespace: default # 所在的命名空间
spec:
  replicas: 1 # 期望副本数
  revisionHistoryLimit: 10 # 进行滚动更新后，保留的历史版本数
  selector: # 选择器，用于找到匹配的 RS
    matchLabels: # 按照标签匹配
      app: nginx-deploy # 匹配的标签key/value
  strategy: # 更新策略
    rollingUpdate: # 滚动更新配置
      maxSurge: 25% # 进行滚动更新时，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25% # 进行滚动更新时，最大不可用比例更新比例，表示在所有副本数中，最多可以有多少个不更新成功
    type: RollingUpdate # 更新类型，采用滚动更新
  template: # pod 模板
    metadata: # pod 的元信息
      labels: # pod 的标签
        app: nginx-deploy
    spec: # pod 期望信息
      containers: # pod 的容器
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest # 镜像
        imagePullPolicy: IfNotPresent # 拉取策略
        name: nginx # 容器名称
      restartPolicy: Always # 重启策略
      terminationGracePeriodSeconds: 30 # 删除操作最多宽限多长时间
```

#### 滚动更新(只有修改了 deployment 配置文件中的 template 中的属性后，才会触发更新操作)
```
修改 nginx 版本号
kubectl set image deployment/nginx-deployment nginx=swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:1.9.1

或者通过 kubectl edit deployment/nginx-deployment 进行修改

查看滚动更新的过程
kubectl rollout status deploy <deployment_name>

查看部署描述，最后展示发生的事件列表也可以看到滚动更新过程
kubectl describe deploy <deployment_name>

通过 kubectl get deployments 获取部署信息，UP-TO-DATE 表示已经有多少副本达到了配置中要求的数目

通过 kubectl get rs 可以看到增加了一个新的 rs

通过 kubectl get pods 可以看到所有 pod 关联的 rs 变成了新的


假设当前有 5 个 nginx:1.7.9 版本，你想将版本更新为 1.9.1，当更新成功第三个以后，你马上又将期望更新的版本改为 1.9.2，那么此时会立马删除之前的三个，并且立马开启更新 1.9.2 的任务


----------------------------------------------------------------------------------------------------------

[root@master deployments]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
nginx-deploy   2/2     2            2           4m36s   app=nginx-deploy
[root@master deployments]# kubectl get rs --show-labels
NAME                      DESIRED   CURRENT   READY   AGE     LABELS
nginx-deploy-548f464cb4   0         0         0       4m42s   app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-9f498cf54    2         2         2       37s     app=nginx-deploy,pod-template-hash=9f498cf54
[root@master deployments]# kubectl get pod --show-labels
NAME                           READY   STATUS    RESTARTS   AGE   LABELS
nginx-deploy-9f498cf54-mpff9   1/1     Running   0          48s   app=nginx-deploy,pod-template-hash=9f498cf54
nginx-deploy-9f498cf54-r65mg   1/1     Running   0          50s   app=nginx-deploy,pod-template-hash=9f498cf54
[root@master deployments]# 
```



#### 回滚（有时候你可能想回退一个Deployment，例如，当Deployment不稳定时，比如一直crash looping。）

```
[root@master deployments]# kubectl rollout history deploy nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@master deployments]# kubectl rollout history deploy nginx-deploy --revision=1
deployment.apps/nginx-deploy with revision #1
Pod Template:
  Labels:       app=nginx-deploy
        pod-template-hash=548f464cb4
  Containers:
   nginx:
    Image:      swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
    Port:       <none>
    Host Port:  <none>
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

[root@master deployments]# ^C
[root@master deployments]# kubectl rollout undo deploy nginx-deploy --to-revisioon=1
error: unknown flag: --to-revisioon
See 'kubectl rollout undo --help' for usage.
[root@master deployments]# kubectl rollout undo deploy nginx-deploy --to-revision=1
deployment.apps/nginx-deploy rolled back
[root@master deployments]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx-deploy   2/2     2            2           15m   app=nginx-deploy
[root@master deployments]# kubectl get rs --show-labels
NAME                      DESIRED   CURRENT   READY   AGE   LABELS
nginx-deploy-548f464cb4   2         2         2       15m   app=nginx-deploy,pod-template-hash=548f464cb4 #现在是这个reday
nginx-deploy-9f498cf54    0         0         0       11m   app=nginx-deploy,pod-template-hash=9f498cf54  #原本是这个reday
[root@master deployments]# kubectl get po --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
nginx-deploy-548f464cb4-gswlv   1/1     Running   0          44s   app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-qcn66   1/1     Running   0          42s   app=nginx-deploy,pod-template-hash=548f464cb4
[root@master deployments]# 
```

#### 扩容缩容 (通过 kube scale 命令可以进行自动扩容/缩容，以及通过 kube edit 编辑 replcas 也可以实现扩容/缩容)
```
[root@master deployments]# kubectl scale --replicas=6 deploy nginx-deploy
deployment.apps/nginx-deploy scaled
[root@master deployments]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx-deploy   6/6     6            6           24m   app=nginx-deploy
[root@master deployments]# kubectl get rs --show-labels
NAME                      DESIRED   CURRENT   READY   AGE   LABELS
nginx-deploy-548f464cb4   6         6         6       24m   app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-9f498cf54    0         0         0       20m   app=nginx-deploy,pod-template-hash=9f498cf54
[root@master deployments]# kubectl get pod --show-labels
NAME                            READY   STATUS    RESTARTS   AGE     LABELS
nginx-deploy-548f464cb4-5rbc7   1/1     Running   0          31s     app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-7grwj   1/1     Running   0          31s     app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-8xwwn   1/1     Running   0          31s     app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-gswlv   1/1     Running   0          9m29s   app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-l8vzj   1/1     Running   0          31s     app=nginx-deploy,pod-template-hash=548f464cb4
nginx-deploy-548f464cb4-qcn66   1/1     Running   0          9m27s   app=nginx-deploy,pod-template-hash=548f464cb4
```
#### 暂停与恢复（由于每次对 pod template 中的信息发生修改后，都会触发更新 deployment 操作，那么此时如果频繁修改信息，就会产生多次更新，而实际上只需要执行最后一次更新即可，当出现此类情况时我们就可以暂停 deployment 的 rollout通过 kubectl rollout pause deployment <name> 就可以实现暂停，直到你下次恢复后才会继续进行滚动更新）
```
[root@master deployments]# kubectl get deploy 
\NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           30m
[root@master deployments]# kubectl edit deploy nginx-deploy
deployment.apps/nginx-deploy edited
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/6     3            5           31m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/6     6            5           31m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/6     6            5           31m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           31m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           31m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           31m
[root@master deployments]# kubectl rollout pause deploy nginx-deploy
deployment.apps/nginx-deploy paused
[root@master deployments]# kubectl edit deploy nginx-deploy
deployment.apps/nginx-deploy edited
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     0            6           32m
[root@master deployments]# kubectl rollout resume deploy nginx-deploy
deployment.apps/nginx-deploy resumed
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/6     3            5           33m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/6     6            5           33m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   7/6     6            7           33m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           33m
[root@master deployments]#
```

### StatefulSet
#### 创建
```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector: 
    matchLabels: 
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        ports:
        - containerPort: 80
          name: web
        # volumeMounts:
        # - name: www
        #   mountPath: /usr/share/nginx/html
  # volumeClaimTemplates:
  # - metadata:
  #     name: www
  #     annotations:
  #       volume.alpha.kubernetes.io/storage-class: anything
  #   spec:
  #     accessModes: [ "ReadWriteOnce" ]
  #     resources:
  #       requests:
  #         storage: 1Gi

```
```
[root@master statefulset]# kubectl apply -f web.yaml
service/nginx created
statefulset.apps/web created
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    1/2     3s
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    1/2     5s
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    2/2     6s
[root@master statefulset]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   132d
nginx        ClusterIP   None         <none>        80/TCP    8m42s
[root@master statefulset]# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          119s
web-1   1/1     Running   0          116s
[root@master statefulset]# kubectl run -i --tty --image swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/busybox:1.28.4 dns-test --restart=Never --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup web-0.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.244.166.174 web-0.nginx.default.svc.cluster.local
/ # pod "dns-test" deleted
[root@master statefulset]# kubectl get pod -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
web-0   1/1     Running   0          16m   10.244.166.174   node1   <none>           <none>
web-1   1/1     Running   0          16m   10.244.104.37    node2   <none>           <none>
[root@master statefulset]# kubectl run -i --tty --image swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/busybox:1.28.4 dns-test --restart=Never --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup web-1.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 10.244.104.37 web-1.nginx.default.svc.cluster.local

/ # ping web-0.nginx
PING web-0.nginx (10.244.166.174): 56 data bytes
64 bytes from 10.244.166.174: seq=0 ttl=62 time=0.313 ms
64 bytes from 10.244.166.174: seq=1 ttl=62 time=1.002 ms
```
#### 扩容缩容
```
[root@master statefulset]# kubectl scale sts web --replicas=5
statefulset.apps/web scaled
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    3/5     21m
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    4/5     21m
[root@master statefulset]# kubectl patch sts web -p '{"spec":{"replicas":3}}'
statefulset.apps/web patched
[root@master statefulset]# kubectl get sts
NAME   READY   AGE
web    3/3     22m
[root@master statefulset]#
```
#### 镜像更新
##### 目前还不支持直接更新 image，需要 patch 来间接实现
```
[root@master statefulset]# kubectl patch sts web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"nginx:自己版本号"}]'
statefulset.apps/web patched
[root@master statefulset]# kubectl get pod
NAME    READY   STATUS              RESTARTS   AGE
web-0   1/1     Running             0          79s
web-1   0/1     ContainerCreating   0          2s
```
##### 灰度发布（RollingUpdate）利用滚动更新中的 partition 属性，可以实现简易的灰度发布的效果例如我们有 5 个 pod，如果当前 partition 设置为 3，那么此时滚动更新时，只会更新那些 序号 >= 3 的 pod,利用该机制，我们可以通过控制 partition 的值，来决定只更新其中一部分 pod，确认没有问题后再主键增大更新的 pod 数量，最终实现全部 pod 更新
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"StatefulSet","metadata":{"annotations":{},"name":"web","namespace":"default"},"spec":{"replicas":2,"selector":{"matchLabels":{"app":"nginx"}},"serviceName":"nginx","template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest","name":"nginx","ports":[{"containerPort":80,"name":"web"}]}]}}}}
  creationTimestamp: "2025-02-11T13:42:05Z"
  generation: 2
  name: web
  namespace: default
  resourceVersion: "161867"
  uid: f9ad1b4f-1043-4d73-80a3-dede720150df
spec:
  podManagementPolicy: OrderedReady
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        imagePullPolicy: Always
        name: nginx
        ports:
        - containerPort: 80
          name: web
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
  updateStrategy:
    rollingUpdate:
      partition: 4
    type: RollingUpdate
status:
  availableReplicas: 5
  collisionCount: 0
  currentReplicas: 5
  currentRevision: web-5458f7c8b7
  observedGeneration: 2
  readyReplicas: 5
  replicas: 5
  updateRevision: web-5458f7c8b7
  updatedReplicas: 5
```
一、 将上述配置文件的partition 改为4，那么一共有5个pod，这时候edit会更新大于4的那个pod
```
updateStrategy:
    rollingUpdate:
      partition: 4
    type: RollingUpdate
```

```
[root@master statefulset]# kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          18m
web-1   1/1     Running   0          18m
web-2   1/1     Running   0          6m1s
web-3   1/1     Running   0          5m59s
web-4   1/1     Running   0          27s
[root@master statefulset]# kubectl edit pod web-4
Edit cancelled, no changes made.
[root@master statefulset]# kubectl edit pod web-3
Edit cancelled, no changes made.
[root@master statefulset]#
```
1. web-4
![image](https://github.com/user-attachments/assets/42428d05-fe32-4eb3-895c-a47a8b02e381)
2. web-3
![image](https://github.com/user-attachments/assets/2867815d-b11f-4da7-b0ed-88aad3011313)

##### OnDelete(只有在 pod 被删除时会进行更新操作)
```
# 以上省略
spec:
  podManagementPolicy: OrderedReady
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  serviceName: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        imagePullPolicy: Always
        name: nginx2
        ports:
        - containerPort: 80
          name: web
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
  updateStrategy:
    type: OnDelete
```
```
[root@master statefulset]# kubectl get po
NAME    READY   STATUS    RESTARTS        AGE
web-0   1/1     Running   1 (9m38s ago)   22h
web-1   1/1     Running   1 (9m33s ago)   22h
web-2   1/1     Running   1 (9m38s ago)   22h
web-3   1/1     Running   1 (9m33s ago)   22h
web-4   1/1     Running   1 (9m33s ago)   22h
[root@master statefulset]# kubectl delete pod web-4
pod "web-4" deleted
^[[A[root@master statefulset]# kubectl get po
NAME    READY   STATUS              RESTARTS        AGE
web-0   1/1     Running             1 (9m48s ago)   22h
web-1   1/1     Running             1 (9m43s ago)   22h
web-2   1/1     Running             1 (9m48s ago)   22h
web-3   1/1     Running             1 (9m43s ago)   22h
web-4   0/1     ContainerCreating   0               1s
[root@master statefulset]# kubectl get po
NAME    READY   STATUS    RESTARTS        AGE
web-0   1/1     Running   1 (9m51s ago)   22h
web-1   1/1     Running   1 (9m46s ago)   22h
web-2   1/1     Running   1 (9m51s ago)   22h
web-3   1/1     Running   1 (9m46s ago)   22h
web-4   1/1     Running   0               4s
```


#### 删除（有状态服务删除有级联删除->删除sts->删除pod | 与不级联删除->删除sts->不会删除pod）
```
[root@master statefulset]# kubectl delete sts web --cascade=false
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
statefulset.apps "web" deleted
[root@master statefulset]# kubectl get pod
NAME    READY   STATUS    RESTARTS      AGE
web-0   1/1     Running   1 (19m ago)   22h
web-1   1/1     Running   1 (19m ago)   22h
web-2   1/1     Running   1 (19m ago)   22h
web-3   1/1     Running   1 (19m ago)   22h
web-4   1/1     Running   0             9m25s
[root@master statefulset]# kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS      AGE     LABELS
web-0   1/1     Running   1 (19m ago)   22h     app=nginx,controller-revision-hash=web-5458f7c8b7,statefulset.kubernetes.io/pod-name=web-0
web-1   1/1     Running   1 (19m ago)   22h     app=nginx,controller-revision-hash=web-5458f7c8b7,statefulset.kubernetes.io/pod-name=web-1
web-2   1/1     Running   1 (19m ago)   22h     app=nginx,controller-revision-hash=web-5458f7c8b7,statefulset.kubernetes.io/pod-name=web-2
web-3   1/1     Running   1 (19m ago)   22h     app=nginx,controller-revision-hash=web-5458f7c8b7,statefulset.kubernetes.io/pod-name=web-3
web-4   1/1     Running   0             9m38s   app=nginx,controller-revision-hash=web-84496dd4dc,statefulset.kubernetes.io/pod-name=web-4
[root@master statefulset]# 
```


### DaemonSet(用于确保集群中的每个（或部分）节点都运行一个特定的 Pod。它适用于需要在所有节点上运行的后台任务或守护进程，创建一个daemonset会为所有相关节点创建pod，如果有新加入的节点，也会为其创建相同pod)

#### 配置文件
```
apiVersion: apps/v1
kind: DaemonSet # 创建 Daemonset资源控制
metadata:
  name: fluentd #名称
spec:
  selector: 
    matchLabels:
      app: logging
  template:
    metadata:
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      nodeSelector: 
        type: microservices
      containers:
      - name: fluentd-es
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        env: # 环境变量
         - name: FLUENTD_ARGS # 环境变量的key
           value: -qq # 环境变量的value
        volumeMounts: # 加载数据卷，避免数据丢失
         - name: containers # 数据卷名称
           mountPath: /var/lib/docker/containers # 将数据卷挂载到容器内的哪个目录
         - name: varlog
           mountPath: /varlog
      volumes: # 定义数据卷
         - hostPath: # 数据卷类型，主机路径的模式，也就是与node共享目录
             path: /var/lib/docker/containers # node中的共享目录
           name: containers # 定义的数据卷名称
         - hostPath:
             path: /var/log
           name: varlog
```
#### 指定node节点（DaemonSet 会忽略 Node 的 unschedulable 状态，有两种方式来指定 Pod 只运行在指定的 Node 节点上）

##### nodeSelector（只调度到匹配指定 label 的 Node 上）
```
kubectl get no -o wide --show-labels
NAME     STATUS   ROLES                  AGE    VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME   LABELS
master   Ready    control-plane,master   133d   v1.23.6   192.168.30.161   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.14   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
node1    Ready    <none>                 133d   v1.23.6   192.168.30.162   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.14   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node1,kubernetes.io/os=linux # 注意这里这时候没有标记type
node2    Ready    <none>                 133d   v1.23.6   192.168.30.163   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   docker://20.10.14   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node2,kubernetes.io/os=linux,type=microservices
```
```
[root@master daemonset]# kubectl apply -f fluentd.yaml 
daemonset.apps/fluentd created
[root@master daemonset]# kubectl get ds
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR        AGE
fluentd   1         1         1       1            1           type=microservices   15s
[root@master daemonset]# kubectl get pod  -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP              NODE    NOMINATED NODE   READINESS GATES
fluentd-cwljg   1/1     Running   0          37s   10.244.104.20   node2   <none>           <none>
```
```
[root@master daemonset]# kubectl label no node1 type=microservices --overwrite
node/node1 labeled
[root@master daemonset]# kubectl get po -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
fluentd-cwljg   1/1     Running   0          91s   10.244.104.20    node2   <none>           <none>
fluentd-mt8fn   1/1     Running   0          12s   10.244.166.144   node1   <none>           <none>
```

一、查看ds配置文件kubectl edit ds name,需要注意的是这里也存在滚动更新，updateStrategy:rollingUpdate:maxSurge: 0maxUnavailable: 1type: RollingUpdate，如果需要把所有的节点都更新的话可以使用RollingUpdate，否则使用OnDelete
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  annotations:
    deprecated.daemonset.template.generation: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"DaemonSet","metadata":{"annotations":{},"name":"fluentd","namespace":"default"},"spec":{"selector":{"matchLabels":{"app":"logging"}},"template":{"metadata":{"labels":{"app":"logging","id":"fluentd"},"name":"fluentd"},"spec":{"containers":[{"env":[{"name":"FLUENTD_ARGS","value":"-qq"}],"image":"swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest","name":"fluentd-es","volumeMounts":[{"mountPath":"/var/lib/docker/containers","name":"containers"},{"mountPath":"/varlog","name":"varlog"}]}],"nodeSelector":{"type":"microservices"},"volumes":[{"hostPath":{"path":"/var/lib/docker/containers"},"name":"containers"},{"hostPath":{"path":"/var/log"},"name":"varlog"}]}}}}
  creationTimestamp: "2025-02-12T13:58:30Z"
  generation: 1
  name: fluentd
  namespace: default
  resourceVersion: "173358"
  uid: b3b21f0c-40db-455e-b1ed-009a170a9208
spec:
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: logging
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      containers:
      - env:
        - name: FLUENTD_ARGS
          value: -qq
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        imagePullPolicy: Always
        name: fluentd-es
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/docker/containers
          name: containers
        - mountPath: /varlog
          name: varlog
      dnsPolicy: ClusterFirst
      nodeSelector:
        type: microservices
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /var/lib/docker/containers
          type: ""
        name: containers
      - hostPath:
          path: /var/log
          type: ""
        name: varlog
  updateStrategy:
    rollingUpdate:
      maxSurge: 0
      maxUnavailable: 1
    type: RollingUpdate
status:
  currentNumberScheduled: 2
  desiredNumberScheduled: 2
  numberAvailable: 2
  numberMisscheduled: 0
  numberReady: 2
  observedGeneration: 1
  updatedNumberScheduled: 2
```

### HPA自动扩容缩容（通过观察 pod 的 cpu、内存使用率或自定义 metrics 指标进行自动的扩容或缩容 pod 的数量。通常用于 Deployment，不适用于无法扩/缩容的对象，如 DaemonSet控制管理器每隔30s（可以通过–horizontal-pod-autoscaler-sync-period修改）查询metrics的资源使用情况）
#### 配置文件
```
apiVersion: apps/v1 # deployment api 版本
kind: Deployment # 资源类型为 deployment
metadata: # 元信息
  labels: # 标签
    app: nginx-deploy # 具体的 key: value 配置形式
  name: nginx-deploy # deployment 的名字
  namespace: default # 所在的命名空间
spec:
  replicas: 2 # 期望副本数
  revisionHistoryLimit: 10 # 进行滚动更新后，保留的历史版本数
  selector: # 选择器，用于找到匹配的 RS
    matchLabels: # 按照标签匹配
      app: nginx-deploy # 匹配的标签key/value
  strategy: # 更新策略
    rollingUpdate: # 滚动更新配置
      maxSurge: 25% # 进行滚动更新时，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25% # 进行滚动更新时，最大不可用比例更新比例，表示在所有副本数中，最多可以有多少个不更新成功
    type: RollingUpdate # 更新类型，采用滚动更新
  template: # pod 模板
    metadata: # pod 的元信息
      labels: # pod 的标签
        app: nginx-deploy
    spec: # pod 期望信息
      containers: # pod 的容器
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest # 镜像
        imagePullPolicy: IfNotPresent # 拉取策略
        name: nginx # 容器名称
        resources: 
          limits: 
            cpu: 200m
            memory: 128Mi
          requests: 
            cpu: 100m
            memory: 128Mi
      restartPolicy: Always # 重启策略
      terminationGracePeriodSeconds: 30 # 删除操作最多宽限多长时间
```
```
# 修改已存在的deployment然后replace yaml
[root@master deployments]# kubectl replace -f nginx-deploy.yaml
deployment.apps/nginx-deploy replaced
[root@master deployments]# 
```
#### 开启指标监控服务-metrics-server:v0.7.2集群的资源监控工具，主要用于收集和聚合集群中各个节点和容器的资源使用情况（如 CPU、内存等），并提供这些信息给 Kubernetes API 服务器。它是 Kubernetes 集群的一个核心监控组件，通常用于自动扩容、水平扩展和诊断问题。具体功能包括：资源监控：提供集群各节点和容器的 CPU 和内存使用数据。API 支持：通过 Kubernetes Metrics API 接口提供实时监控数据，供 Horizontal Pod Autoscaler（HPA）等资源自动扩展机制使用。集成支持：支持与 Kubernetes Dashboard、Prometheus 等工具的集成，增强对集群资源的可视化和监控。

#### metrics-server yaml配置文件
```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.7.2
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```
#### 添加HPA
```
[root@master deployments]# kubectl apply -f service.yaml 
[root@master deployments]# kubectl top pods # 测试metrics-server
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-deploy-788875c698-g9t2n   0m           3Mi             
nginx-deploy-788875c698-tjr7k   0m           3Mi             
nginx-deploy-788875c698-tzk5j   0m           3Mi  
[root@master deployments]# kubectl autoscale deploy nginx-deploy --cpu-percent=20 --min=3 --max=5 # cpu打到百分之二十，最少三个pod最多五个
horizontalpodautoscaler.autoscaling/nginx-deploy autoscaled
[root@master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2/2     2            2           14m
[root@master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           14m
```

#### 创建nginx-deploy的service
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
  
spec:
  selector: 
    app: nginx-deploy
  ports:
  - port: 80
    targetPort: 80
    name: web
  type: NodePort
```

```
[root@master deployments]# kubectl apply -f service.yaml 
service/nginx-svc created
[root@master deployments]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        135d
nginx-svc    NodePort    10.97.41.167   <none>        80:31952/TCP   5s
```
#### 测试扩容
```
### node1下访问servier
[root@master deployments]# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        135d
nginx-svc    NodePort    10.97.41.167   <none>        80:31952/TCP   4m51s

[root@node1 ~]# while true; do wget -q -O- 10.97.41.167  > /dev/null ; done
^C
```

#### 结果
```
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   20%/20%   3         5         4          11m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   20%/20%   3         5         4          11m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   23%/20%   3         5         4          11m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   23%/20%   3         5         4          11m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   23%/20%   3         5         4          11m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   33%/20%   3         5         5          12m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   33%/20%   3         5         5          12m


[root@master deployments]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/5     5            5           17m
[root@master deployments]# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-788875c698-9fqcx   1/1     Running   0          8m28s
nginx-deploy-788875c698-h5kjx   1/1     Running   0          17m
nginx-deploy-788875c698-r9tx5   1/1     Running   0          17m
nginx-deploy-788875c698-v28g7   1/1     Running   0          17m
nginx-deploy-788875c698-w5g7z   1/1     Running   0          4m58s
[root@master deployments]# kubectl get  pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
nginx-deploy-788875c698-9fqcx   1/1     Running   0          4m41s   10.244.166.154   node1   <none>           <none>
nginx-deploy-788875c698-h5kjx   1/1     Running   0          13m     10.244.104.27    node2   <none>           <none>
nginx-deploy-788875c698-r9tx5   1/1     Running   0          13m     10.244.166.151   node1   <none>           <none>
nginx-deploy-788875c698-v28g7   1/1     Running   0          13m     10.244.104.26    node2   <none>           <none>
nginx-deploy-788875c698-w5g7z   1/1     Running   0          71s     10.244.104.30    node2   <none>           <none>
[root@master deployments]# kubectl top pods
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-deploy-788875c698-9fqcx   36m          3Mi             
nginx-deploy-788875c698-h5kjx   21m          3Mi             
nginx-deploy-788875c698-r9tx5   38m          3Mi             
nginx-deploy-788875c698-v28g7   21m          3Mi             
nginx-deploy-788875c698-w5g7z   21m          3Mi


```
#### 测试缩容
```
# 停止压力测试后
[root@master deployments]# kubectl top pods
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-deploy-788875c698-9fqcx   0m           3Mi             
nginx-deploy-788875c698-h5kjx   0m           3Mi             
nginx-deploy-788875c698-r9tx5   0m           3Mi             
nginx-deploy-788875c698-v28g7   0m           3Mi             
nginx-deploy-788875c698-w5g7z   0m           3Mi

[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/20%    3         5         5          14m

[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/20%    3         5         5          14m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/20%    3         5         3          14m
[root@master deployments]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/20%    3         5         3          19m
[root@master deployments]# kubectl get deploy 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           20m
[root@master deployments]# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-788875c698-9fqcx   1/1     Running   0          11m
nginx-deploy-788875c698-r9tx5   1/1     Running   0          20m
nginx-deploy-788875c698-v28g7   1/1     Running   0          20m
[root@master deployments]#
```




## 服务发布
### Service（负责东西流量（同层级/内部服务网络通信）的通信）
![image](https://github.com/user-attachments/assets/51e1b8d7-b1d8-4484-bd46-7dccd9e358f8)

#### 步骤说明
##### 第一步 （Deployment 是管理应用的容器副本集（Pod）的一种方式。它定义了应用容器的模板，并指定了副本数（Pod 数量），并确保所需数量的 Pod 在任何时候都在运行。在 Deployment 中，你会给 Pod 配置一些 标签（Labels）。）
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
```
##### 第二步 （Service 会根据选择器（selector）来定位与之关联的 Pod。它通常会使用 标签选择器（Label Selector） 来选择 Deployment 创建的 Pod）
```
apiVersion: v1
kind: Service # 资源类型为Service
metadata:
  name: my-app-service # service名称
spec:
  selector:
    app: my-app # 匹配哪些pod会被改service代理
  ports:
    - port: 80 # service自己的端口，在内网ip访问使用
      targetPort: 8080 # 目标pod的端口
  name: web # 为端口起个名字
type: NodePort # 随机启动一个端口30000-32767.映射到ports中的端口，该端口是直接绑定在node上的，且集群中的每一个node都会绑定这个端口， 也可以用于将服务暴露给外部访问，但是不推荐生成环境使用
```
##### 第三步 （Endpoints：Service 通过标签选择器匹配到与之关联的 Pod 后，Kubernetes 会自动为 Service 创建相应的 Endpoints。Endpoints 是一个资源，它保存着符合 Service 标签选择器的 Pod 的 IP 地址和端口。例如，Service my-app-service 会将请求转发到所有标签为 app: my-app 的 Pod）

#### 操作service(集群内布)
```
[root@node1 ~]# kubectl describe svc nginx-svc
Name:                     nginx-svc
Namespace:                default
Labels:                   app=nginx
Annotations:              <none>
Selector:                 app=nginx-deploy
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.97.41.167
IPs:                      10.97.41.167
Port:                     web  80/TCP
TargetPort:               80/TCP
NodePort:                 web  31952/TCP
Endpoints:                10.244.104.28:80,10.244.104.29:80,10.244.104.31:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@node1 ~]#
```
```
kubectl exec -it nginx-deploy-788875c698-4scf4 -- sh

# 通过svc名称访问
# curl http://nginx-svc
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
# 跨命名空间访问
# curl http://nginx-svc.default
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 代理k8s集群外部服务
```
apiVersion: v1
kind: Service 
metadata:
  name: nginx-svc-external
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    name: web
  type: ClusterIP
```
```
[root@master service]# kubectl apply -f service.yaml 
service/nginx-svc-external created
[root@master service]# kubectl get svc
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes           ClusterIP   10.96.0.1       <none>        443/TCP        137d
nginx-svc            NodePort    10.97.41.167    <none>        80:31952/TCP   44h
nginx-svc-external   ClusterIP   10.105.133.94   <none>        80/TCP         4s
[root@master service]# kubectl get ep
NAME         ENDPOINTS                                            AGE
kubernetes   192.168.30.161:6443                                  137d
nginx-svc    10.244.104.28:80,10.244.104.29:80,10.244.104.31:80   44h
```
##### 手动创建ep
```
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: nginx # 与 service 一致
  name: nginx-svc-external # 与 service 一致
  namespace: default # 与 service 一致
subsets:
- addresses:
  - ip: 120.78.159.117 # 目标 ip 地址
  ports: # 与 service 一致
  - name: web
    port: 80
    protocol: TCP
```
```
[root@master service]# kubectl apply -f service.yaml 
service/nginx-svc-external created
[root@master service]# kubectl get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP        137d
nginx-svc            NodePort    10.97.41.167     <none>        80:31952/TCP   45h
nginx-svc-external   ClusterIP   10.109.128.238   <none>        80/TCP         3s
[root@master service]# kubectl apply -f nginx-ep.yaml

[root@master service]# kubectl exec -it nginx-deploy-788875c698-4scf4 -- sh
# curl http://nginx-svc-external
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx/1.8.1</center>
</body>
</html>
 
```

#### 常用类型
| **Service 类型**    | **访问范围**                           | **使用场景**                                                       | **示例** |
|---------------------|----------------------------------------|--------------------------------------------------------------------|----------|
| **ClusterIP**       | 仅集群内部可访问                       | - 集群内部的微服务通信<br>- 数据库、缓存等不需要外部访问的服务   | ```yaml<br>apiVersion: v1<br>kind: Service<br>metadata:<br>  name: my-service<br>spec:<br>  selector:<br>    app: my-app<br>  ports:<br>    - port: 80<br>      targetPort: 8080``` |
| **NodePort**        | 通过集群节点的 IP 和端口访问           | - 开发/测试环境<br>- 简易暴露服务给外部访问，不需要负载均衡的场景 | ```yaml<br>apiVersion: v1<br>kind: Service<br>metadata:<br>  name: my-service<br>spec:<br>  selector:<br>    app: my-app<br>  ports:<br>    - port: 80<br>      targetPort: 8080<br>      nodePort: 30001<br>  type: NodePort``` |
| **LoadBalancer**    | 外部可访问，自动创建外部负载均衡器     | - 需要高可用、外部访问的服务<br>- 适合生产环境，尤其是大规模流量的服务 | ```yaml<br>apiVersion: v1<br>kind: Service<br>metadata:<br>  name: my-service<br>spec:<br>  selector:<br>    app: my-app<br>  ports:<br>    - port: 80<br>      targetPort: 8080<br>  type: LoadBalancer``` |
| **ExternalName**    | 外部 DNS 服务，流量转发到外部服务      | - 访问集群外的服务（如外部数据库、第三方 API）                     | ```yaml<br>apiVersion: v1<br>kind: Service<br>metadata:<br>  name: my-external-service<br>spec:<br>  type: ExternalName<br>  externalName: example.com``` |

##### 说明

一. **ClusterIP**（默认类型）:
   - 该类型的服务只能在集群内部访问，适用于需要集群内部通信的服务，如数据库、缓存服务等。

二. **NodePort**:
   - 该类型会在每个节点上开放一个相同的端口，适用于开发、测试环境或者简单的外部访问需求。

三. **LoadBalancer**:
   - 该类型的服务会为你自动创建一个外部负载均衡器，适用于需要从外部访问并且需要负载均衡的服务，通常用于生产环境。

四. **ExternalName**:
   - 该类型的服务会将请求转发到指定的外部 DNS 名称，适用于需要访问集群外部服务的场景。

---



### Ingress（Ingress 大家可以理解为也是一种 LB 的抽象，它的实现也是支持 nginx、haproxy 等负载均衡服务的）
![image](https://github.com/user-attachments/assets/27ba5495-9b1d-4177-8824-10298335a701)
#### ingress的所有版本，我这里选择的ingress-nginx，还可以安装其他的
```
AKS 应用程序网关 Ingress 控制器 是一个配置 Azure 应用程序网关 的 Ingress 控制器。
阿里云 MSE Ingress 是一个 Ingress 控制器，它负责配置阿里云原生网关， 也是 Higress 的商业版本。
Apache APISIX Ingress 控制器 是一个基于 Apache APISIX 网关 的 Ingress 控制器。
Avi Kubernetes Operator 使用 VMware NSX Advanced Load Balancer 提供第 4 到第 7 层的负载均衡。
BFE Ingress 控制器是一个基于 BFE 的 Ingress 控制器。
Cilium Ingress 控制器是一个由 Cilium 出品支持的 Ingress 控制器。
Citrix Ingress 控制器 可以用来与 Citrix Application Delivery Controller 一起使用。
Contour 是一个基于 Envoy 的 Ingress 控制器。
Emissary-Ingress API 网关是一个基于 Envoy 的入口控制器。
EnRoute 是一个基于 Envoy 的 API 网关，可以用作 Ingress 控制器。
Easegress IngressController 是一个基于 Easegress 的 API 网关，可以用作 Ingress 控制器。
F5 BIG-IP 的 用于 Kubernetes 的容器 Ingress 服务 让你能够使用 Ingress 来配置 F5 BIG-IP 虚拟服务器。
FortiADC Ingress 控制器 支持 Kubernetes Ingress 资源，并允许你从 Kubernetes 管理 FortiADC 对象。
Gloo 是一个开源的、基于 Envoy 的 Ingress 控制器，能够提供 API 网关功能。
HAProxy Ingress 是一个针对 HAProxy 的 Ingress 控制器。
Higress 是一个基于 Envoy 的 API 网关， 可以作为一个 Ingress 控制器运行。
用于 Kubernetes 的 HAProxy Ingress 控制器 也是一个针对 HAProxy 的 Ingress 控制器。
Istio Ingress 是一个基于 Istio 的 Ingress 控制器。
用于 Kubernetes 的 Kong Ingress 控制器 是一个用来驱动 Kong Gateway 的 Ingress 控制器。
Kusk Gateway 是一个基于 Envoy 的、 OpenAPI 驱动的 Ingress 控制器。
用于 Kubernetes 的 NGINX Ingress 控制器 能够与 NGINX 网页服务器（作为代理）一起使用。
ngrok Kubernetes Ingress 控制器 是一个开源控制器，通过使用 ngrok 平台为你的 K8s 服务添加安全的公开访问权限。
OCI Native Ingress Controller 是一个适用于 Oracle Cloud Infrastructure 的 Ingress 控制器，可帮助你管理 OCI 负载均衡。
OpenNJet Ingress Controller 是一个基于 OpenNJet 的 Ingress 控制器。
Pomerium Ingress 控制器 基于 Pomerium，能提供上下文感知的准入策略。
Skipper HTTP 路由器和反向代理可用于服务组装，支持包括 Kubernetes Ingress 这类使用场景，是一个用以构造你自己的定制代理的库。
Traefik Kubernetes Ingress 提供程序 是一个用于 Traefik 代理的 Ingress 控制器。
Tyk Operator 使用自定义资源扩展 Ingress，为之带来 API 管理能力。Tyk Operator 使用开源的 Tyk Gateway & Tyk Cloud 控制面。
Voyager 是一个针对 HAProxy 的 Ingress 控制器。
Wallarm Ingress Controller 是提供 WAAP（WAF） 和 API 安全功能的 Ingress Controller。
```
#### 安装ingress-nginx
##### 安装helm（有点类似maven或者yum，通过helm安装ingress）
```
[root@master helm]# wget https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz
--2025-02-17 20:20:43--  https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz
正在解析主机 get.helm.sh (get.helm.sh)... 13.107.246.73, 2620:1ec:bdf::73
正在连接 get.helm.sh (get.helm.sh)|13.107.246.73|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度：12924654 (12M) [application/x-tar]
正在保存至: “helm-v3.2.3-linux-amd64.tar.gz”

100%[========================================================================================>] 12,924,654  68.8KB/s 用时 2m 37s 

2025-02-17 20:23:20 (80.4 KB/s) - 已保存 “helm-v3.2.3-linux-amd64.tar.gz” [12924654/12924654])

[root@master helm]# tar -zxvf helm-v3.2.3-linux-amd64.tar.gz 
linux-amd64/
linux-amd64/README.md
linux-amd64/LICENSE
linux-amd64/helm
[root@master helm]# ll
总用量 12624
-rw-r--r-- 1 root root 12924654 6月   8 2020 helm-v3.2.3-linux-amd64.tar.gz
drwxr-xr-x 2 3434 3434       50 6月   8 2020 linux-amd64
[root@master helm]# cd linux-amd64/
[root@master linux-amd64]# ll
总用量 39448
-rwxr-xr-x 1 3434 3434 40378368 6月   8 2020 helm
-rw-r--r-- 1 3434 3434    11373 6月   8 2020 LICENSE
-rw-r--r-- 1 3434 3434     3319 6月   8 2020 README.md
[root@master linux-amd64]# cp helm /usr/local/bin
[root@master linux-amd64]# llcd /
-bash: llcd: 未找到命令
[root@master linux-amd64]# cd /
[root@master /]# helm version
version.BuildInfo{Version:"v3.2.3", GitCommit:"8f832046e258e2cb800894579b1b3b50c2d83492", GitTreeState:"clean", GoVersion:"go1.13.12"}
[root@master /]# 
```

##### 添加 helm的ingress仓库
```
[root@master /]# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
[root@master /]# helm repo list
NAME            URL                                       
ingress-nginx   https://kubernetes.github.io/ingress-nginx
[root@master /]# helm search repo ingress-nginx
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     4.12.0          1.12.0          Ingress controller for Kubernetes using NGINX a...
[root@master /]#
```
##### 下载包
```
[root@master helm]# helm search repo ingress-nginx --versions
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     4.12.0          1.12.0          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.11.4          1.11.4          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.11.3          1.11.3          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.11.2          1.11.2          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.11.1          1.11.1          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.11.0          1.11.0          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.6          1.10.6          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.5          1.10.5          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.4          1.10.4          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.3          1.10.3          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.2          1.10.2          Ingress controller for Kubernetes using NGINX a...
ingress-nginx/ingress-nginx     4.10.1          1.10.1          Ingress controller for Kubernetes using NGINX a...

# 下载4.4.2版本
[root@master helm]# helm pull ingress-nginx/ingress-nginx --version 4.4.2
[root@master helm]# ll
总用量 12672
-rw-r--r-- 1 root root 12924654 6月   8 2020 helm-v3.2.3-linux-amd64.tar.gz
-rw-r--r-- 1 root root    45268 2月  17 21:30 ingress-nginx-4.4.2.tgz
drwxr-xr-x 2 3434 3434       50 6月   8 2020 linux-amd64
[root@master helm]# tar -zxvf ingress-nginx-4.4.2.tgz 
ingress-nginx/Chart.yaml
ingress-nginx/values.yaml
ingress-nginx/templates/NOTES.txt
ingress-nginx/templates/_helpers.tpl
ingress-nginx/templates/_params.tpl

[root@master helm]# ll
总用量 12672
-rw-r--r-- 1 root root 12924654 6月   8 2020 helm-v3.2.3-linux-amd64.tar.gz
drwxr-xr-x 4 root root      191 2月  17 21:31 ingress-nginx
-rw-r--r-- 1 root root    45268 2月  17 21:30 ingress-nginx-4.4.2.tgz
drwxr-xr-x 2 3434 3434       50 6月   8 2020 linux-amd64
[root@master helm]# cd ingress-nginx
[root@master ingress-nginx]# ll
总用量 132
-rw-r--r-- 1 root root 25165 12月 30 2022 CHANGELOG.md
-rw-r--r-- 1 root root   452 12月 30 2022 changelog.md.gotmpl
-rw-r--r-- 1 root root   853 12月 30 2022 Chart.yaml
drwxr-xr-x 2 root root  4096 2月  17 21:31 ci
-rw-r--r-- 1 root root   213 12月 30 2022 OWNERS
-rw-r--r-- 1 root root 39130 12月 30 2022 README.md
-rw-r--r-- 1 root root 12138 12月 30 2022 README.md.gotmpl
drwxr-xr-x 3 root root  4096 2月  17 21:31 templates
-rw-r--r-- 1 root root 32125 12月 30 2022 values.yaml
```
```
# 修改 values.yaml配置文件
镜像地址：修改为国内镜像
registry: registry.cn-hangzhou.aliyuncs.com
image: google_containers/nginx-ingress-controller
image: google_containers/kube-webhook-certgen
tag: v1.3.0

hostNetwork: true
dnsPolicy: ClusterFirstWithHostNet

修改部署配置的 kind: DaemonSet
nodeSelector:
  ingress: "true" # 增加选择器，如果 node 上有 ingress=true 就部署
将 admissionWebhooks.enabled 修改为 false
将 service 中的 type 由 LoadBalancer 修改为 ClusterIP，如果服务器是云平台才用 LoadBalancer
```
```
# 卸载掉之前的
[root@master ingress-nginx]# kubectl get pod -n ingress-nginx
NAME                             READY   STATUS             RESTARTS      AGE
ingress-nginx-controller-txkvb   0/1     CrashLoopBackOff   8 (46s ago)   16m
[root@master ingress-nginx]# helm uninstall ingress-nginx -n ingress-nginx
release "ingress-nginx" uninstalled

# 给node打标签（master被认为是污节点，所以安装不上）
[root@master ingress-nginx]# kubectl label node node1 ingress=true

# 安装
[root@master ingress-nginx]# helm install ingress-nginx  -n ingress-nginx .
NAME: ingress-nginx
LAST DEPLOYED: Mon Feb 17 21:42:05 2025
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
Get the application URL by running these commands:
  export POD_NAME=$(kubectl --namespace ingress-nginx get pods -o jsonpath="{.items[0].metadata.name}" -l "app=ingress-nginx,component=controller,release=ingress-nginx")
  kubectl --namespace ingress-nginx port-forward $POD_NAME 8080:80
  echo "Visit http://127.0.0.1:8080 to access your application."

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

[root@master ingress-nginx]# kubectl get pod -n ingress-nginx
NAME                             READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-v2wch   1/1     Running   0          26s
[root@master ingress-nginx]# kubectl get pod -n ingress-nginx -owide --show-labels
NAME                             READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES   LABELS
ingress-nginx-controller-v2wch   1/1     Running   0          79s   192.168.30.162   node1   <none>           <none>            app.kubernetes.io/component=controller,app.kubernetes.io/instance=ingress-nginx,app.kubernetes.io/name=ingress-nginx,controller-revision-hash=56fc676cb,pod-template-generation=1
```
#### ingress资源基本使用（注意：上面安装的ingress-controller是它是实际处理流量的组件,现在安装的Ingress 资源依赖于 Ingress Controller 来实现流量路由，也就是ingress资源属于是负责路由管理的ingress）完整的请求路线>>>外部请求 -> Ingress Controller -> Ingress 资源（路由规则） -> Service -> endpoint -> Pod

```
apiVersion: networking.k8s.io/v1
kind: Ingress # 资源类型为 Ingress
metadata:
  name: wolfcode-nginx-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules: # ingress 规则配置，可以配置多个
  - host: k8s.wolfcode.cn # 域名配置，可以使用通配符 *
    http:
      paths: # 相当于 nginx 的 location 配置，可以配置多个
      - pathType: Prefix # 路径类型，按照路径类型进行匹配 ImplementationSpecific 需要指定 IngressClass，具体匹配规则以 IngressClass 中的规则为准。Exact：精确匹配，URL需要与path完全匹配上，且区分大小写的。Prefix：以 / 作为分隔符来进行前缀匹配
        backend:
          service: 
            name: nginx-svc # 代理到哪个 service
            port: 
              number: 80 # service 的端口
        path: /api # 等价于 nginx 中的 location 的路径前缀匹配
```

```
[root@master ingress]# kubectl apply -f wolfcode-ingress.yaml 
ingress.networking.k8s.io/wolfcode-nginx-ingress configured
[root@master ingress]# kubectl get ingress
NAME                     CLASS    HOSTS                              ADDRESS         PORTS   AGE
wolfcode-nginx-ingress   <none>   k8s.wolfcode.cn   10.102.121.38   80      15m
```

```
# 注意这里ingress-controller是在node1节点
[root@master ingress]# kubectl get po -n ingress-nginx
NAME                             READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-45nrz   1/1     Running   0          21m
[root@master ingress]# kubectl get po -n ingress-nginx -o wide
NAME                             READY   STATUS    RESTARTS   AGE   IP               NODE    NOMINATED NODE   READINESS GATES
ingress-nginx-controller-45nrz   1/1     Running   0          21m   192.168.30.162   node1   <none>           <none>
```

![image](https://github.com/user-attachments/assets/ce0789bf-6787-413c-b700-b56d3a0be845)

![image](https://github.com/user-attachments/assets/cdf56558-8491-46f0-ae17-7c8c0dbb6136)




## 配置与存储
### 配置管理
#### ConfigMap （一般用于去存储 Pod 中应用所需的一些配置信息，或者环境变量，将配置于 Pod 分开，避免应为修改配置导致还需要重新构建 镜像与容器。）
##### 创建
```
[root@master test]# ll
总用量 8
-rw-r--r-- 1 root root 28 2月  20 20:50 test2.yaml
-rw-r--r-- 1 root root 31 2月  20 20:50 test.text
[root@master config]# kubectl create cm my-cm --from-file=test/
configmap/my-cm created
[root@master config]# kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      141d
my-cm              2      4s
[root@master test]# kubectl describe cm my-cm
Name:         my-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
test.text:
----
username=admin
password=123456

test2.yaml:
----
redis: 127.0.0.1
port: 6379


BinaryData
====

Events:  <none>

[root@master test]# kubectl create cm my-cm2 --from-file=/opt/k8s/config/test/test.text 
configmap/my-cm2 created
[root@master test]# ll
总用量 8
-rw-r--r-- 1 root root 28 2月  20 20:50 test2.yaml
-rw-r--r-- 1 root root 31 2月  20 20:50 test.text
[root@master test]# kubectl describe cm my-cm2
Name:         my-cm2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
test.text:
----
username=admin
password=123456


BinaryData
====

Events:  <none>
```


```
[root@master test]# kubectl create cm my-cm3 --from-file=rename=/opt/k8s/config/test/test2.yaml 
configmap/my-cm3 created
[root@master test]# kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      141d
my-cm              2      2m10s
my-cm2             1      43s
my-cm3             1      4s
[root@master test]# kubectl describe cm my-cm3
Name:         my-cm3
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
rename:
----
redis: 127.0.0.1
port: 6379


BinaryData
====

Events:  <none>
```
```
[root@master test]# kubectl create cm literal --from-literal=username=root --from-literal=password=123456
configmap/literal created
[root@master test]# kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      141d
literal            2      4s
my-cm              2      7m3s
my-cm2             1      5m36s
my-cm3             1      4m57s
[root@master test]# kubectl describe cm literal
Name:         literal
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
password:
----
123456
username:
----
root

BinaryData
====

Events:  <none>
```

##### 基本使用
一、基于configMap把环境变量初始化到pod中
```
[root@master test]# kubectl create cm env --from-literal=JAVA_HOME_TEST='-Xms=512m -Xmx=512m' --from-literal=APPNAME=SPRINGBOOT_TEST
configmap/env created

[root@master test]# kubectl describe cm env
Name:         env
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
APPNAME:
----
SPRINGBOOT_TEST
JAVA_HOME_TEST:
----
-Xms=512m -Xmx=512m

BinaryData
====

Events:  <none>
```
```
# pod配置文件
apiVersion: v1
kind: Pod
metadata: 
  name: test-env-cm
spec: 
  containers:
    - name: env-test
      image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/alpine
      command: ["/bin/sh","-c","env;sleep 3600"]
      imagePullPolicy: IfNotPresent
      
      env: 
      - name: JAVA_VM_OPTS
        valueFrom: 
          configMapKeyRef: 
            name: env
            key: JAVA_HOME_TEST
      - name: APP
        valueFrom: 
          configMapKeyRef: 
            name: env
            key: APPNAME
  restartPolicy: Never
```
```
[root@master test]# kubectl apply -f env-test-cm.yaml 
pod/test-env-cm created

[root@master test]# kubectl logs -f test-env-cm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
NGINX_SVC_SERVICE_HOST=10.97.41.167
HOSTNAME=test-env-cm
NGINX_SVC_EXTERNAL_SERVICE_HOST=10.109.128.238
SHLVL=1
HOME=/root
JAVA_VM_OPTS=-Xms=512m -Xmx=512m
NGINX_SVC_SERVICE_PORT=80
NGINX_SVC_PORT=tcp://10.97.41.167:80
NGINX_SVC_EXTERNAL_SERVICE_PORT=80
NGINX_SVC_EXTERNAL_PORT=tcp://10.109.128.238:80
NGINX_SVC_SERVICE_PORT_WEB=80
NGINX_SVC_PORT_80_TCP_ADDR=10.97.41.167
APP=SPRINGBOOT_TEST
NGINX_SVC_EXTERNAL_SERVICE_PORT_WEB=80
NGINX_SVC_PORT_80_TCP_PORT=80
NGINX_SVC_PORT_80_TCP_PROTO=tcp
NGINX_SVC_EXTERNAL_PORT_80_TCP_ADDR=10.109.128.238
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NGINX_SVC_EXTERNAL_PORT_80_TCP_PORT=80
NGINX_SVC_EXTERNAL_PORT_80_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NGINX_SVC_PORT_80_TCP=tcp://10.97.41.167:80
NGINX_SVC_EXTERNAL_PORT_80_TCP=tcp://10.109.128.238:80
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```
二、使用数据卷的方式挂载configMap配置文件
```
[root@master test]# kubectl describe cm my-cm
Name:         my-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
test.text:
----
username=admin
password=123456

test2.yaml:
----
redis: 127.0.0.1
port: 6379


BinaryData
====

Events:  <none>
```

```
apiVersion: v1
kind: Pod
metadata: 
  name: test-env-cm
spec: 
  containers:
    - name: env-test
      image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/alpine
      command: ["/bin/sh","-c","env;sleep 3600"]
      imagePullPolicy: IfNotPresent
      
      env: 
      - name: JAVA_VM_OPTS
        valueFrom: 
          configMapKeyRef: 
            name: env # configMap名称
            key: JAVA_HOME_TEST # 表示从name的Configmap中获取名字为key的value，将其赋值为本地环境变量JAVA_VM_OPTS
      - name: APP
        valueFrom: 
          configMapKeyRef: 
            name: env
            key: APPNAME
      volumeMounts: #记载数据卷
        - name: db-config # 加载的valumes中的数据卷
          mountPath: "/usr/local/mysql/conf" # 将数据卷中的文件加载到哪个目录下
          readOnly: true # 只读
  volumes: # 数据卷挂载configMap、secret
    - name: db-config # 数据卷名称 
      configMap: # 数据卷为configMap
        name: my-cm # configmap名称，必须跟想要加载的configmap相同
        items: # 对configmap中的key进行映射，如果不指定，默认会以configmap所有的key全部转换为一个个同名的文件
        - key: "test.text" # kubectl describe cm my-cm 中的key(test.text)，而不是存储的key(username),value(123456)
          path: "test.text" # 与mountPath地址拼接
  restartPolicy: Never
```

```
[root@master test]# kubectl apply -f env-test-cm.yaml 
pod/test-env-cm created
[root@master test]# kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-788875c698-926rz   1/1     Running   0          160m
nginx-deploy-788875c698-mllbx   1/1     Running   0          160m
nginx-deploy-788875c698-pbvgc   1/1     Running   0          159m
test-env-cm                     1/1     Running   0          3s
[root@master test]# kubectl exec -it test-env-cm -- sh
/ # cd /usr/local/mysql/
/usr/local/mysql # ll
sh: ll: not found
/usr/local/mysql # ls
conf
/usr/local/mysql # cd conf/
/usr/local/mysql/conf # ls
test.text
/usr/local/mysql/conf # cat test.text 
username=admin
password=123456
```


#### 加密数据配置 Secret（与 ConfigMap 类似，用于存储配置信息，但是主要用于存储敏感信息、需要加密的信息，Secret 可以提供数据加密、解密功能。在创建 Secret 时，要注意如果要加密的字符中，包含了有特殊字符，需要使用转义符转移，例如 $ 转移后为 \$，也可以对特殊字符使用单引号描述，这样就不需要转移例如 1$289*-! 转换为 '1$289*-!'）
##### 创建
```
[root@master ~]# kubectl create secret generic orig-secret --from-literal=username=admin --from-literal=password=123456
secret/orig-secret created
[root@master ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-w6c5w   kubernetes.io/service-account-token   3      142d
orig-secret           Opaque                                2      7s
[root@master ~]# kubectl describe secret orig-secret
Name:         orig-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
username:  5 bytes
[root@master ~]# echo 'admin' | base64
YWRtaW4K
[root@master ~]# echo YWRtaW4K | base64 --decode
admin
[root@master ~]#
```
##### 基本使用
```
[root@master ~]# kubectl create secret docker-registry harbor-secret --docker-username=admin --docker-password=123456 --docker-email=xxx@qq.com
secret/harbor-secret created
[root@master ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-w6c5w   kubernetes.io/service-account-token   3      142d
harbor-secret         kubernetes.io/dockerconfigjson        1      7s
orig-secret           Opaque                                2      18m
[root@master ~]# kubectl describe secret harbot-secret
Error from server (NotFound): secrets "harbot-secret" not found
[root@master ~]# kubectl describe secret harbor-secret
Name:         harbor-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/dockerconfigjson

Data
====
.dockerconfigjson:  129 bytes

[root@master ~]# kubectl edit secret harbor-secret
Edit cancelled, no changes made.
[root@master ~]# echo 'eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJhZG1pbiIsInBhc3N3b3JkIjoiMTIzNDU2IiwiZW1haWwiOiJ4eHhAcXEuY29tIiwiYXV0aCI6IllXUnRhVzQ2TVRJek5EVTIifX19' | base64 --decode
{"auths":{"https://index.docker.io/v1/":{"username":"admin","password":"123456","email":"xxx@qq.com","auth":"YWRtaW46MTIzNDU2"}}}[root@master ~]# 
```
![image](https://github.com/user-attachments/assets/46e544c8-adc9-45bf-85a5-2f770cc7d81e)


#### SUBPATH(subPath 是一种挂载卷时的配置，它允许你将卷的特定子路径挂载到容器中，而不是整个卷。它通常用于在一个卷中挂载多个文件或目录，或者当你不希望容器看到整个卷的内容时，使用 subPath 可以只选择卷中的一部分来挂载)

##### 复制一份nginx.conf配置文件，并创建cm
```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```
```
[root@master config]# kubectl create cm nginx-conf-cm --from-file=./nginx.conf 
configmap/nginx-conf-cm created
[root@master config]# kubectl get cm
NAME               DATA   AGE
env                2      2d23h
kube-root-ca.crt   1      144d
literal            2      2d23h
my-cm              2      2d23h
my-cm2             1      2d23h
my-cm3             1      2d23h
nginx-conf-cm      1      4s
```
##### 编辑deploy并基于subpath挂载cm，通过subpath可避免全部文件挂载（否则可能会出现/etc/nginx下的文件只剩下nginx.conf导致pod启动不起来）
![image](https://github.com/user-attachments/assets/5c1ce32d-fc4b-4213-af36-271ff5518079)

```
    spec:
      containers:
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        imagePullPolicy: IfNotPresent
        name: nginx
        resources:
          limits:
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /etc/nginx/nginx.conf
          name: nginx-conf
          subPath: etc/nginx/nginx.conf
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - configMap:
          defaultMode: 420
          items:
          - key: nginx.conf
            path: etc/nginx/nginx.conf
          name: nginx-conf-cm
        name: nginx-conf
```

#### 配置热更新(我们通常会将项目的配置文件作为 configmap 然后挂载到 pod，那么如果更新 configmap 中的配置，会不会更新到 pod 中呢？这得分成几种情况：默认方式：会更新，更新周期是更新时间 + 缓存时间subPath：不会更新变量形式：如果 pod 中的一个变量是从 configmap 或 secret 中得到，同样也是不会更新的对于 subPath 的方式，我们可以取消 subPath 的使用，将配置文件挂载到一个不存在的目录，避免目录的覆盖，然后再利用软连接的形式，将该文件链接到目标位置,但是如果目标位置原本就有文件，可能无法创建软链接，此时可以基于前面讲过的 postStart 操作执行删除命令，将默认的吻技安删除即可)
##### 通过edit修改configMap

#### 不可变的 Secret 和 ConfigMap （对于一些敏感服务的配置文件，在线上有时是不允许修改的，此时在配置 configmap 时可以设置 immutable: true 来禁止修改）
```
apiVersion: v1
data:
  APPNAME: SPRINGBOOT_TEST
  JAVA_HOME_TEST: -Xms=512m -Xmx=512m
immutable: true
kind: ConfigMap
metadata:
  creationTimestamp: "2025-02-20T13:10:57Z"
  name: env
  namespace: default
  resourceVersion: "256490"
  uid: 7154eea3-b5b8-4407-b821-b0e23eae1757
```
```
# 修改时会报错
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
# configmaps "env" was not valid:
# * data: Forbidden: field is immutable when `immutable` is set
#
```


### 持久化存储
#### Volumes
##### HostPath (将节点上的文件或目录挂载到 Pod 上，此时该目录会变成持久化存储目录，即使 Pod 被删除后重启，也可以重新加载到该目录，该目录下的文件不会丢失)
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
    name: nginx-volume
    volumeMounts:
    - mountPath: /test-pd # 挂载到容器的哪个目录
      name: test-volume # 挂载哪个 volume
  volumes:
  - name: test-volume
    hostPath:
      path: /data # 节点中的目录
      type: DirectoryOrcreate # 检查类型，在挂载前对挂载目录做什么检查操作，有多种选项，默认为空字符串，不做任何检查
# 类型：
# 空字符串：默认类型，不做任何检查
# DirectoryOrCreate：如果给定的 path 不存在，就创建一个 755 的空目录
# Directory：这个目录必须存在
# FileOrCreate：如果给定的文件不存在，则创建一个空文件，权限为 644
# File：这个文件必须存在
# Socket：UNIX 套接字，必须存在
# CharDevice：字符设备，必须存在
# BlockDevice：块设备，必须存在
```
```
[root@master volumes]# kubectl apply -f hostpath-pod.yaml 
pod/test-pd created
[root@master volumes]# kubectl get pod -o wide
NAME                            READY   STATUS    RESTARTS      AGE     IP               NODE    NOMINATED NODE   READINESS GATES
nginx-deploy-6cd8b54689-7hz7q   1/1     Running   4 (37m ago)   37m     10.244.104.62    node2   <none>           <none>
nginx-deploy-6cd8b54689-jj8xz   1/1     Running   1 (55m ago)   24h     10.244.104.59    node2   <none>           <none>
nginx-deploy-6cd8b54689-nx4n9   1/1     Running   4 (36m ago)   37m     10.244.166.171   node1   <none>           <none>
test-env-cm                     0/1     Error     0             3d23h   <none>           node1   <none>           <none>
test-pd                         1/1     Running   0             8m9s    10.244.166.168   node1   <none>           <none>
```
```
# 发现这个pod部署在node1上，所以去node1的/data目录下创建文件完成挂载
# node1
[root@node1 data]# pwd
/data
[root@node1 data]# ll
总用量 4
-rw-r--r-- 1 root root 10 2月  24 21:26 volumes.txt
[root@node1 data]# 
```
```
[root@master volumes]# kubectl exec -it test-pd -- sh
# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  test-pd  tmp  usr  var
# cd test-pd
# cat volumes.txt
from pod 
# pwd
/test-pd
```

##### EmptyDir(EmptyDir 主要用于一个 Pod 中不同的 Container 共享数据使用的，由于只是在 Pod 内部使用，因此与其他 volume 比较大的区别是，当 Pod 如果被删除了，那么 emptyDir 也会被删除。存储介质可以是任意类型，如 SSD、磁盘或网络存储。可以将 emptyDir.medium 设置为 Memory 让 k8s 使用 tmpfs（内存支持文件系统），速度比较快，但是重启 tmpfs 节点时，数据会被清除，且设置的大小会计入到 Container 的内存限制中。)

```
apiVersion: v1
kind: Pod
metadata:
  name: empty-dir-pd
spec:
  containers:
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
    name: nginx-emptydir1
    volumeMounts:
    - mountPath: /test
      name: cache-volume
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/redis:latest
    name: redis-emptydir2
    volumeMounts:
    - mountPath: /test2
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

```
[root@master volumes]# kubectl apply -f empty.yaml 
pod/empty-dir-pd created
[root@master volumes]# kubectl get po
[root@master volumes]# kubectl get po
NAME                            READY   STATUS    RESTARTS       AGE
empty-dir-pd                    2/2     Running   0              55s
nginx-deploy-6cd8b54689-7hz7q   1/1     Running   4 (89m ago)    89m
nginx-deploy-6cd8b54689-jj8xz   1/1     Running   1 (107m ago)   24h
nginx-deploy-6cd8b54689-nx4n9   1/1     Running   4 (88m ago)    89m
test-env-cm                     0/1     Error     0              4d
test-pd                         1/1     Running   0              60m
```

```
# 第一个容器

[root@master volumes]# kubectl exec -it empty-dir-pd -c nginx-emptydir1 -- sh
# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  test  tmp  usr  var
# cd test
# ls
# vi test.txt
sh: 4: vi: not found
# echo 'test' > index.html
# ls
index.html
#
```

```
# 第二个容器

[root@master ~]# kubectl exec -it empty-dir-pd -c redis-empty2 -- sh
Error from server (BadRequest): container redis-empty2 is not valid for pod empty-dir-pd
[root@master ~]# kubectl exec -it empty-dir-pd -c redis-emptydir2 -- sh
# ls
# cd /
# ls
bin   data  etc   lib    media  opt   root  sbin  sys    tmp  var
boot  dev   home  lib64  mnt    proc  run   srv   test2  usr
# cd test2
# ls
# ls
index.html
#
```


#### NFS 挂载 （nfs 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 emptyDir 那样会在删除 Pod 的同时也会被删除，nfs 卷的内容在删除 Pod 时会被保存，卷只是被卸载。 这意味着 nfs 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。）

##### 安装nfs
```

# 安装 nfs (三个节点都执行)
yum install nfs-utils -y

# 启动 nfs (三个节点都执行)
systemctl start nfs-server

# 查看 nfs 版本 (三个节点都执行)
cat /proc/fs/nfsd/versions

# 创建共享目录 (node1节点执行)
mkdir -p /home/nfs
cd /home/nfs
mkdir rw
mkdir ro

# 设置共享目录 export (node1执行)
vim /etc/exports
/home/nfs/rw 192.168.30.0/24(rw,sync,no_subtree_check,no_root_squash)
/home/nfs/ro 192.168.30.0/24(ro,sync,no_subtree_check,no_root_squash)

# 重新加载 (node1执行)
exportfs -f 
systemctl reload nfs-server

# 到其他测试节点安装 nfs-utils 并加载测试
mkdir -p /mnt/nfs/rw

```

```
# master 节点验证

[root@master ~]# mount -t nfs 192.168.30.162:/home/nfs/rw /mnt/nfs/rw
[root@master ~]# cd /mnt/nfs/rw/
[root@master rw]# ll
总用量 0
[root@master rw]# mount -t nfs 192.168.30.162:/home/nfs/ro /mnt/nfs/ro
[root@master rw]# cd /mnt/nfs/ro
[root@master ro]# ll
总用量 4
-rw-r--r-- 1 root root 12 2月  25 20:31 README.md
[root@master ro]# touch master
touch: 无法创建"master": 只读文件系统

[root@master ro]# cd ../rw
[root@master rw]# ll
总用量 0
[root@master rw]# touch master
[root@master rw]# ll
总用量 0
-rw-r--r-- 1 root root 0 2月  25 20:36 master

# node1 节点

[root@node1 rw]# pwd
/home/nfs/rw
[root@node1 rw]# ll
总用量 0
-rw-r--r-- 1 root root 0 2月  25 20:36 master

```

##### nfs挂载

```
# node1节点存储

[root@node1 index]# pwd
/home/nfs/rw/www/index
[root@node1 index]# ll
总用量 4
-rw-r--r-- 1 root root 16 2月  25 20:59 index.html
[root@node1 index]# cat index.html
<h1>hello</h1>
```

```
# master节点

apiVersion: v1
kind: Pod
metadata:
  name: nfs-pd1
spec:
  containers:
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
    name: test-container
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: test-volume
  volumes:
  - name: test-volume
    nfs:
      server: 192.168.30.162 # 网络存储服务地址
      path: /home/nfs/rw/www/index # 网络存储路径
      readOnly: false # 是否只读
```

```
# master节点

[root@master volumes]# kubectl apply -f nfs.yaml 
pod/nfs-pd1 created
[root@master volumes]# kubectl get po
NAME                            READY   STATUS    RESTARTS      AGE
empty-dir-pd                    2/2     Running   2 (45m ago)   22h
nfs-pd1                         1/1     Running   0             53s
nginx-deploy-6cd8b54689-7hz7q   1/1     Running   5 (45m ago)   24h
nginx-deploy-6cd8b54689-jj8xz   1/1     Running   2 (45m ago)   47h
nginx-deploy-6cd8b54689-nx4n9   1/1     Running   5 (45m ago)   24h

[root@master volumes]# kubectl get po -o wide
NAME                            READY   STATUS    RESTARTS      AGE     IP               NODE    NOMINATED NODE   READINESS GATES
empty-dir-pd                    2/2     Running   2 (46m ago)   22h     10.244.104.13    node2   <none>           <none>
nfs-pd1                         1/1     Running   0             86s     10.244.166.170   node1   <none>           <none>
nginx-deploy-6cd8b54689-7hz7q   1/1     Running   5 (46m ago)   24h     10.244.104.10    node2   <none>           <none>
nginx-deploy-6cd8b54689-jj8xz   1/1     Running   2 (46m ago)   47h     10.244.104.11    node2   <none>           <none>
nginx-deploy-6cd8b54689-nx4n9   1/1     Running   5 (46m ago)   24h     10.244.166.173   node1   <none>           <none>
test-env-cm                     0/1     Error     0             4d23h   <none>           node1   <none>           <none>
test-pd                         1/1     Running   1 (46m ago)   23h     10.244.166.172   node1   <none>           <none>
[root@master volumes]# curl 10.244.166.170 
<h1>hello</h1>
```

```
# node1 节点

[root@node1 index]# echo '<h1>hello2</h1>' > index.html 
[root@node1 index]# cat index.html 
<h1>hello2</h1>
```

```
[root@master volumes]# curl 10.244.166.170 
<h1>hello2</h1>
```

```
# 创建第二个nfs pod （配置文件与第一个相同，只不过pod名称不同）

[root@master volumes]# kubectl apply -f nfs2.yaml 
pod/nfs-pd2 created

[root@master volumes]# kubectl get po -o wide
NAME                            READY   STATUS    RESTARTS      AGE     IP               NODE    NOMINATED NODE   READINESS GATES
empty-dir-pd                    2/2     Running   2 (49m ago)   22h     10.244.104.13    node2   <none>           <none>
nfs-pd1                         1/1     Running   0             5m4s    10.244.166.170   node1   <none>           <none>
nfs-pd2                         1/1     Running   0             22s     10.244.166.169   node1   <none>           <none>
nginx-deploy-6cd8b54689-7hz7q   1/1     Running   5 (49m ago)   24h     10.244.104.10    node2   <none>           <none>
nginx-deploy-6cd8b54689-jj8xz   1/1     Running   2 (49m ago)   47h     10.244.104.11    node2   <none>           <none>
nginx-deploy-6cd8b54689-nx4n9   1/1     Running   5 (49m ago)   24h     10.244.166.173   node1   <none>           <none>
test-env-cm                     0/1     Error     0             4d23h   <none>           node1   <none>           <none>
test-pd                         1/1     Running   1 (49m ago)   23h     10.244.166.172   node1   <none>           <none>
[root@master volumes]# curl  10.244.166.169
<h1>hello2</h1>

```
```
# 此时node1 节点修改或删除文件 这里也会同样感知，

[root@node1 index]# cat index.html 
<h1>hello2</h1>
[root@node1 index]# mv index.html index.html.bak 
```

```
# master 节点

[root@master volumes]# curl 10.244.166.170 
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.27.1</center>
</body>
</html>
[root@master volumes]# kubectl exec -it nfs-pd1  -- sh
# ll
sh: 1: ll: not found
# cd /usr/share
# ls
X11              ca-certificates  doc         gcc       libc-bin     maven-repo  pam-configs  terminfo
base-files       common-licenses  doc-base    gdb       libgcrypt20  menu        perl5        util-linux
base-passwd      debconf          dpkg        info      lintian      misc        pixmaps      xml
bash-completion  debianutils      fontconfig  java      locale       nginx       polkit-1     zoneinfo
bug              dict             fonts       keyrings  man          pam         tabset       zsh
# cd nginx
# ls
html
# cd html
# ls
index.html.bak
# mv index.html.bak index.html
# exit
[root@master volumes]# curl 10.244.166.170 
<h1>hello2</h1>

```

```
root@node1 index]# ll
总用量 4
-rw-r--r-- 1 root root 16 2月  25 21:11 index.html
```


#### PV与PVC

| 组件        | 描述                                                             |
|-------------|------------------------------------------------------------------|
| **磁盘**    | 提供底层存储，通常是物理硬盘、云存储、网络存储等存储介质。           |
| **PV**      | Persistent Volume，Kubernetes 中的存储资源抽象，代表一个已配置的存储卷。|
| **PVC**     | Persistent Volume Claim，Pod 请求存储的方式，声明所需存储的大小和特性。 |
| **Pod**     | 通过 PVC 请求存储，Pod 访问存储资源（即 PV），实现持久化存储。       |

![image](https://github.com/user-attachments/assets/fa2fb675-ef15-49e3-95ec-38da278a5cb6)


##### 生命周期
一、构建  

1.静态构建（集群管理员创建若干 PV 卷。这些卷对象带有真实存储的细节信息， 并且对集群用户可用（可见）。PV 卷对象存在于 Kubernetes API 中，可供用户消费（使用）。）  


2.动态构建（如果集群中已经有的 PV 无法满足 PVC 的需求，那么集群会根据 PVC 自动构建一个 PV，该操作是通过 StorageClass 实现的。想要实现这个操作，前提是 PVC 必须设置 StorageClass，否则会无法动态构建该 PV，可以通过启用 DefaultStorageClass 来实现 PV 的构建。）  


二、绑定 （当用户创建一个 PVC 对象后，主节点会监测新的 PVC 对象，并且寻找与之匹配的 PV 卷，找到 PV 卷后将二者绑定在一起。如果找不到对应的 PV，则需要看 PVC 是否设置 StorageClass 来决定是否动态创建 PV，若没有配置，PVC 就会一致处于未绑定状态，直到有与之匹配的 PV 后才会申领绑定关系。）  

三、使用（Pod 将 PVC 当作存储卷来使用，集群会通过 PVC 找到绑定的 PV，并为 Pod 挂载该卷。Pod 一旦使用 PVC 绑定 PV 后，为了保护数据，避免数据丢失问题，PV 对象会受到保护，在系统中无法被删除。）  


四、回收策略（当用户不再使用其存储卷时，他们可以从 API 中将 PVC 对象删除， 从而允许该资源被回收再利用。PersistentVolume 对象的回收策略告诉集群， 当其被从申领中释放时如何处理该数据卷。 目前，数据卷可以被 Retained（保留）、Recycled（回收）或 Deleted（删除）。）   


1.保留（Retain）  

```
回收策略 Retain 使得用户可以手动回收资源。当 PersistentVolumeClaim 对象被删除时，PersistentVolume 卷仍然存在，对应的数据卷被视为"已释放（released）"。
由于卷上仍然存在这前一申领人的数据，该卷还不能用于其他申领。 管理员可以通过下面的步骤来手动回收该卷：
删除 PersistentVolume 对象。与之相关的、位于外部基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
根据情况，手动清除所关联的存储资产上的数据。
手动删除所关联的存储资产。
如果你希望重用该存储资产，可以基于存储资产的定义创建新的 PersistentVolume 卷对象。
```

2.删除（Delete）  

```
对于支持 Delete 回收策略的卷插件，删除动作会将 PersistentVolume 对象从 Kubernetes 中移除，同时也会从外部基础设施（如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）中移除所关联的存储资产。
动态制备的卷会继承其 StorageClass 中设置的回收策略， 该策略默认为 Delete。管理员需要根据用户的期望来配置 StorageClass； 否则 PV 卷被创建之后必须要被编辑或者修补。
```

3.回收（Recycle）  

```
警告： 回收策略 Recycle 已被废弃。取而代之的建议方案是使用动态制备。

如果下层的卷插件支持，回收策略 Recycle 会在卷上执行一些基本的擦除 （rm -rf /thevolume/*）操作，之后允许该卷用于新的 PVC 申领。
```

##### 持久卷（PersistentVolume，PV） 是集群中的一块存储，可以由管理员事先制备， 或者使用存储类（Storage Class）来动态制备。 持久卷是集群资源，就像节点也是集群资源一样。PV 持久卷和普通的 Volume 一样， 也是使用卷插件来实现的，只是它们拥有独立于任何使用 PV 的 Pod 的生命周期。 此 API 对象中记述了存储的实现细节，无论其背后是 NFS、iSCSI 还是特定于云平台的存储系统

一、状态  
```
Available：空闲，未被绑定
Bound：已经被 PVC 绑定
Released：PVC 被删除，资源已回收，但是 PV 未被重新使用
Failed：自动回收失败
```

二、配置文件  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  capacity:
    storage: 5Gi # pv 的容量
  volumeMode: Filesystem # 存储类型为文件系统
  accessModes: # 访问模式：ReadWriteOnce、ReadWriteMany、ReadOnlyMany
    - ReadWriteOnce # 可被单节点独写
  persistentVolumeReclaimPolicy: Recycle # 回收策略
  storageClassName: slow # 创建 PV 的存储类名，需要与 pvc 的相同
  mountOptions: # 加载配置
    - hard
    - nfsvers=4.1
  nfs: # 连接到 nfs
    path: /home/nfs/rw/pv # 存储路径
    server: 192.168.30.162 # nfs 服务地址
```

三、创建pv  
```
[root@master pv]# kubectl apply -f pv1.yaml 
persistentvolume/pv0001 created
[root@master pv]# kubectl get pv 
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv0001   5Gi        RWO            Recycle          Available           slow                    6s
```

##### 持久卷申领（PersistentVolumeClaim，PVC） 表达的是用户对存储的请求。概念上与 Pod 类似。 Pod 会耗用节点资源，而 PVC 申领会耗用 PV 资源。Pod 可以请求特定数量的资源（CPU 和内存）。同样 PVC 申领也可以请求特定的大小和访问模式 （例如，可以挂载为 ReadWriteOnce、ReadOnlyMany、ReadWriteMany 或 ReadWriteOncePod， 请参阅访问模式）

一、配置文件  
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc0001
spec:
  accessModes:
    - ReadWriteOnce # 权限需要与对应的 pv 相同
  volumeMode: Filesystem
  resources:
    requests:
      storage: 4Gi # 资源可以小于 pv 的，但是不能大于，如果大于就会匹配不到 pv
  storageClassName: slow # 名字需要与对应的 pv 相同
#  selector: # 使用选择器选择对应的 pv
#    matchLabels:
#      release: "stable"
#    matchExpressions:
#      - {key: environment, operator: In, values: [dev]}
```
二、创建pvc  
```
[root@master pvc]# kubectl apply -f pvc1.yaml 
persistentvolumeclaim/pvc0001 created
[root@master pvc]# kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc0001   Bound    pv0001   5Gi        RWO            slow           4s
[root@master pvc]# kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
pv0001   5Gi        RWO            Recycle          Bound    default/pvc0001   slow                    2m33s
[root@master pvc]# 
```

三、创建pod
```
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod0001
spec:
  containers:
  - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
    name: nginx-volume
    volumeMounts:
    - mountPath: /usr/share/nginx/html # 挂载到容器的哪个目录
      name: test-volume # 挂载哪个 volume
  volumes:
  - name: test-volume
    persistentVolumeClaim: # 关联pvc
      claimName: pvc0001 # pvc名称
```

```
[root@master pvc]# kubectl apply -f pod-pvc.yaml 
pod/pvc-pod0001 created
[root@master pvc]# kubectl get pod -o wide
NAME                            READY   STATUS             RESTARTS       AGE     IP               NODE    NOMINATED NODE   READINESS GATES
empty-dir-pd                    2/2     Running            4 (119m ago)   2d23h   10.244.104.17    node2   <none>           <none>
nfs-pd1                         1/1     Running            1 (119m ago)   2d      10.244.166.174   node1   <none>           <none>
nfs-pd2                         1/1     Running            1 (119m ago)   2d      10.244.166.176   node1   <none>           <none>
nginx-deploy-6cd8b54689-7hz7q   1/1     Running            6 (119m ago)   3d      10.244.104.16    node2   <none>           <none>
nginx-deploy-6cd8b54689-jj8xz   1/1     Running            3 (119m ago)   4d      10.244.104.15    node2   <none>           <none>
nginx-deploy-6cd8b54689-nx4n9   1/1     Running            6 (119m ago)   3d      10.244.166.177   node1   <none>           <none>
pvc-pod0001                     1/1     Running            0              9s      10.244.166.178   node1   <none>           <none>
recycler-for-pv0001             0/1     ImagePullBackOff   0              4m41s   10.244.104.19    node2   <none>           <none>
test-env-cm                     0/1     Error              0              6d23h   <none>           node1   <none>           <none>
test-pd                         1/1     Running            2 (119m ago)   3d      10.244.166.175   node1   <none>           <none>
[root@master pvc]#

[root@master pvc]# curl 10.244.166.178
<h1>pv pvc pod hello</h1>
# node1 节点文件详情
[root@node1 pv]# pwd
/home/nfs/rw/pv
[root@node1 pv]# cat index.html 
<h1>pv pvc pod hello</h1>
[root@node1 pv]# 
```


##### StorageClass（k8s 中提供了一套自动创建 PV 的机制，就是基于 StorageClass 进行的，通过 StorageClass 可以实现仅仅配置 PVC，然后交由 StorageClass 根据 PVC 的需求动态创建 PV。）

![image](https://github.com/user-attachments/assets/5338c35e-3abd-4fe5-80ed-39aeb04f7425)

一、RBAC（Role-Based Access Control）是 Kubernetes 用于 控制用户或服务对集群资源访问权限 的机制。通过 RBAC，可以精细化管理 谁（用户/服务账号） 可以对 哪些资源 执行 什么操作（读/写/删除等）
```
为什么创建 StorageClass 需要 RBAC？
在 Kubernetes 中，StorageClass 只是一个定义，但真正的 存储分配是由存储 Provisioner 负责的。当 PVC（PersistentVolumeClaim）申请存储时：

Provisioner 需要访问和操作 PersistentVolume（PV）、PersistentVolumeClaim（PVC）：
读取 PVC 请求。
创建 PV 资源，并绑定到 PVC。
可能还要修改或删除 PVC/PV 资源。
Provisioner 可能还需要管理 Endpoints、StorageClass 和 Events，用于记录日志或自动分配存储。
这些操作 都需要 API 权限，而默认情况下 Kubernetes 不会授予 Pod 这些权限，因此 必须通过 RBAC 赋权
```
二、创建rbac
```
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "list", "watch", "create", "delete"]
- apiGroups: [""]
  resources: ["persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "update"]
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-client-provisioner
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
- kind: ServiceAccount
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: kube-system
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io

```
```
[root@master sc]# kubectl apply -f nfs-provisioner-rbac.yaml 
clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
role.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner unchanged
rolebinding.rbac.authorization.k8s.io/leader-locking-nfs-client-provisioner unchanged
[root@master sc]# kubectl get clusterrole
NAME                                                                   CREATED AT
admin                                                                  2024-10-02T07:19:54Z
calico-kube-controllers                                                2024-10-02T11:28:36Z
calico-node                                                            2024-10-02T11:28:36Z
cluster-admin                                                          2024-10-02T07:19:54Z
edit                                                                   2024-10-02T07:19:54Z
ingress-nginx                                                          2025-02-18T13:40:08Z
kubeadm:get-nodes                                                      2024-10-02T07:19:55Z
nfs-client-provisioner-runner                                          2025-03-02T13:24:06Z
```

三、创建Provisioner制备器（StorageClass的制备器），用来决定使用哪个卷插件制备 PV
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  namespace: kube-system
---
kind: Deployment
apiVersion: apps/v1
metadata:
  namespace: kube-system
  name: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccount: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
#          image: quay.io/external_storage/nfs-client-provisioner:latest
          image: registry.cn-beijing.aliyuncs.com/pylixm/nfs-subdir-external-provisioner:v4.0.0
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: fuseim.pri/ifs  # 关联sc。。
            - name: NFS_SERVER
              value: 192.168.30.162
            - name: NFS_PATH
              value: /home/nfs/rw
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.30.162
            path: /home/nfs/rw
```
```
[root@master sc]# kubectl apply -f nfs-provisioner-deployment.yaml 
serviceaccount/nfs-client-provisioner unchanged
deployment.apps/nfs-client-provisioner unchanged
[root@master sc]# kubectl get deploy -n kube-system
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           151d
coredns                   2/2     2            2           151d
metrics-server            1/1     1            1           15d
nfs-client-provisioner    1/1     1            1           47m

[root@master sc]# kubectl get pod -n kube-system | grep nfs
nfs-client-provisioner-dc6c4b947-z26tv    1/1     Running   0               49m
```

三、创建StorageClass
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: fuseim.pri/ifs # 关联sc。。
parameters:
  archiveOnDelete: "false"
```
```
[root@master sc]# kubectl apply -f nfs-storage-class.yaml 
storageclass.storage.k8s.io/managed-nfs-storage created
[root@master sc]# kubectl get sc
NAME                  PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   fuseim.pri/ifs   Delete          Immediate           false                  3s
```

四、创建sts以及基于步骤三创建的managed-nfs-storage（StorageClass）使用Provisioner制备器创建pv并于nginx-sc-test-pvc绑定

```
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-sc
  labels:
    app: nginx-sc
spec:
  type: NodePort
  ports:
  - name: web
    port: 80
    protocol: TCP
  selector:
    app: nginx-sc
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-sc
spec:
  replicas: 1
  serviceName: "nginx-sc"
  selector:
    matchLabels:
      app: nginx-sc
  template:
    metadata:
      labels:
        app: nginx-sc
    spec:
      containers:
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        name: nginx-sc
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - mountPath: /usr/share/nginx/html # # 挂载到容器的哪个目录
          name: nginx-sc-test-pvc # 挂载哪个 volume
  volumeClaimTemplates:
  - metadata:
      name: nginx-sc-test-pvc
    spec:
      storageClassName: managed-nfs-storage
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Gi

```

```
[root@master sc]# kubectl apply -f nfs-sc-demo-statefulset.yaml 
service/nginx-sc unchanged
statefulset.apps/nginx-sc created
[root@master sc]# kubectl get sts
NAME       READY   AGE
nginx-sc   1/1     4s
[root@master sc]# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-6cd8b54689-fm7zr   1/1     Running   0          155m
nginx-deploy-6cd8b54689-qjql2   1/1     Running   0          155m
nginx-deploy-6cd8b54689-zb2wc   1/1     Running   0          155m
nginx-sc-0                      1/1     Running   0          6s
[root@master sc]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
nginx-sc-test-pvc-nginx-sc-0   Bound    pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            managed-nfs-storage   3m22s
[root@master sc]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS          REASON   AGE
pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            Delete           Bound    default/nginx-sc-test-pvc-nginx-sc-0   managed-nfs-storage            3m26s
[root@master sc]# 
```

五、验证pod
```
[root@master sc]# kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-6cd8b54689-fm7zr   1/1     Running   0          158m
nginx-deploy-6cd8b54689-qjql2   1/1     Running   0          158m
nginx-deploy-6cd8b54689-zb2wc   1/1     Running   0          158m
nginx-sc-0                      1/1     Running   0          3m25s

[root@master sc]# kubectl exec -it nginx-sc-0      -- sh
# cd /usr/share
# ls
X11          bash-completion  common-licenses  dict      dpkg        gcc   java      libgcrypt20  man         misc   pam-configs  polkit-1  util-linux  zsh
base-files   bug              debconf          doc       fontconfig  gdb   keyrings  lintian      maven-repo  nginx  perl5        tabset    xml
base-passwd  ca-certificates  debianutils      doc-base  fonts       info  libc-bin  locale       menu        pam    pixmaps      terminfo  zoneinfo
# cd nginx
# ls
html
# cd html
# ls
# ls
index.html
# cat index.html
hello sc
#
```

```
# node1节点 nfs地址

[root@node1 rw]# pwd
/home/nfs/rw
[root@node1 rw]# ll
总用量 0
drwxrwxrwx 2 root root 24 3月   2 22:31 default-nginx-sc-test-pvc-nginx-sc-0-pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d
drwxrwxrwx 2 root root  6 3月   2 21:37 default-nginx-sc-test-pvc-nginx-sc-0-pvc-a491bcd2-3cc8-4bda-9c58-cbebf72d68d0
drwxrwxrwx 2 root root  6 3月   2 21:51 default-test-nfs-pvc-pvc-0243ebd1-42d8-4204-9f88-8864a21a20f3
-rw-r--r-- 1 root root  0 2月  25 20:36 master
drwxr-xr-x 2 root root 24 2月  27 21:37 pv
drwxr-xr-x 3 root root 19 2月  25 20:49 www
```


六、测试sc
```
[root@master sc]# cat test-sc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: managed-nfs-storage

[root@master sc]# kubectl get sc
NAME                  PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
managed-nfs-storage   fuseim.pri/ifs   Delete          Immediate           false                  15m
[root@master sc]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS          REASON   AGE
pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            Delete           Bound    default/nginx-sc-test-pvc-nginx-sc-0   managed-nfs-storage            11m
[root@master sc]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
nginx-sc-test-pvc-nginx-sc-0   Bound    pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            managed-nfs-storage   11m
[root@master sc]# kubectl apply -f test-sc.yaml 
persistentvolumeclaim/test-nfs-pvc created
[root@master sc]# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                  STORAGECLASS          REASON   AGE
pvc-079b554a-f8fb-47d2-bd8d-635cea5c637c   1Gi        RWX            Delete           Bound    default/test-nfs-pvc                   managed-nfs-storage            5s
pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            Delete           Bound    default/nginx-sc-test-pvc-nginx-sc-0   managed-nfs-storage            11m
[root@master sc]# kubectl get pvc
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS          AGE
nginx-sc-test-pvc-nginx-sc-0   Bound    pvc-4c22de7f-6099-4aff-8728-13b47c0e3c5d   1Gi        RWX            managed-nfs-storage   11m
test-nfs-pvc                   Bound    pvc-079b554a-f8fb-47d2-bd8d-635cea5c637c   1Gi        RWX            managed-nfs-storage   8s
[root@master sc]# 
```




## 高级调度

### CronJob计划任务
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  concurrencyPolicy: Allow # 并发调度策略：Allow 允许并发调度，Forbid：不允许并发执行，Replace：如果之前的任务还没执行完，就直接执行新的，放弃上一个任务
  failedJobsHistoryLimit: 1 # 保留多少个失败的任务
  successfulJobsHistoryLimit: 3 # 保留多少个成功的任务
  suspend: false # 是否挂起任务，若为 true 则该任务不会执行
#  startingDeadlineSeconds: 30 # 间隔多长时间检测失败的任务并重新执行，时间不能小于 10
  schedule: "* * * * *" # 调度策略
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
```
[root@master jobs]# kubectl apply -f cron-job.yaml 
cronjob.batch/hello created
[root@master jobs]# kubectl get cj
NAME    SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   * * * * *   False     0        <none>          5s
[root@master jobs]# kubectl get cj
NAME    SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   * * * * *   False     0        27s             4m11s

[root@master jobs]# kubectl get po
NAME                            READY   STATUS      RESTARTS      AGE
hello-29018260-7bqbh            0/1     Completed   0             46s
nginx-deploy-6cd8b54689-fm7zr   1/1     Running     1 (77m ago)   2d1h
nginx-deploy-6cd8b54689-qjql2   1/1     Running     1 (77m ago)   2d1h
nginx-deploy-6cd8b54689-zb2wc   1/1     Running     1 (77m ago)   2d1h
nginx-sc-0                      1/1     Running     1 (77m ago)   47h

[root@master jobs]# kubectl logs -f hello-29018260-7bqbh  
Tue Mar  4 13:41:01 UTC 2025
Hello from the Kubernetes cluster

[root@master jobs]# kubectl get po
NAME                            READY   STATUS      RESTARTS      AGE
hello-29018260-7bqbh            0/1     Completed   0             63s
hello-29018261-4brth            0/1     Completed   0             3s
nginx-deploy-6cd8b54689-fm7zr   1/1     Running     1 (78m ago)   2d1h
nginx-deploy-6cd8b54689-qjql2   1/1     Running     1 (78m ago)   2d1h
nginx-deploy-6cd8b54689-zb2wc   1/1     Running     1 (78m ago)   2d1h
nginx-sc-0                      1/1     Running     1 (78m ago)   47h

[root@master jobs]# kubectl logs -f hello-29018261-4brth 
Tue Mar  4 13:41:01 UTC 2025
Hello from the Kubernetes cluster
```


### 初始化容器 InitContainer （在真正的容器启动之前，先启动 InitContainer，在初始化容器中完成真实容器所需的初始化操作，完成后再启动真实的容器。相对于 postStart 来说，首先 InitController 能够保证一定在 EntryPoint 之前执行，而 postStart 不能，其次 postStart 更适合去执行一些命令操作，而 InitController 实际就是一个容器，可以在其他基础容器环境下执行更复杂的初始化功能）
```
在 pod 创建的模板中配置 initContainers 参数：
spec:
  initContainers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    command: ["sh", "-c", "echo 'inited;' >> ~/.init"]
    name: init-test


这个 initContainer： ✅ 作用：只是往 ~/.init 文件里写 "inited;"，可能是为了测试 initContainer 机制。
✅ 不会影响主容器，因为 initContainer 执行完就会被清理，~/.init 也不会保留。
✅ 实际应用中，可以用 initContainer 来创建目录、拷贝配置文件、检查依赖服务等
```


### 污点和容忍 (污点（Taint） 则相反——它使节点能够排斥一类特定的 Pod。容忍度（Toleration） 是应用于 Pod 上的。容忍度允许调度器调度带有对应污点的 Pod。 容忍度允许调度但并不保证调度：作为其功能的一部分， 调度器也会评估其他参数。)
#### 污点（是标注在节点上的，当我们在一个节点上打上污点以后，k8s 会认为尽量不要将 pod 调度到该节点上，除非该 pod 上面表示可以容忍该污点，且一个节点可以打多个污点，此时则需要 pod 容忍所有污点才会被调度该节点。）
```
# 为某个节点添加污点
[root@master ~]# kubectl taint node node2 memory=low:NoSchedule
node/node2 tainted
[root@master ~]# kubectl describe no  node2
Name:               node2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node2
                    kubernetes.io/os=linux
                    type=microservices
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.30.163/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.244.104.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 02 Oct 2024 15:44:14 +0800
Taints:             memory=low:NoSchedule
Unschedulable:      false

# 移除污点
[root@master ~]# kubectl taint node node2 memory=low:NoSchedule-
node/node2 untainted
[root@master ~]# kubectl describe no  node2
Name:               node2
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node2
                    kubernetes.io/os=linux
                    type=microservices
Annotations:        node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.30.163/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.244.104.0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 02 Oct 2024 15:44:14 +0800
Taints:             <none>
Unschedulable:      false

# 查看污点
[root@master ~]# kubectl describe no master
Name:               master
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    ingress=true
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=master
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.30.161/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.244.219.64
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Wed, 02 Oct 2024 15:19:53 +0800
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
```
###### NoSchedule
```
# 给节点 node1 增加一个污点，它的键名是 key1，键值是 value1，效果是 NoSchedule。 这表示只有拥有和这个污点相匹配的容忍度的 Pod 才能够被分配到 node1 这个节点。
[root@master ~]# kubectl taint no node1 memory=low:NoSchedule
node/node1 tainted
[root@master ~]#
```
###### NoExecute
```
# 这会影响已在节点上运行的 Pod，具体影响如下：
# 如果 Pod 不能容忍这类污点，会马上被驱逐。
# 如果 Pod 能够容忍这类污点，但是在容忍度定义中没有指定 tolerationSeconds， 则 Pod 还会一直在这个节点上运行。
# 如果 Pod 能够容忍这类污点，而且指定了 tolerationSeconds， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。 这段时间过去后，节点生命周期控制器从节点驱除这些 Pod。

[root@master ~]# kubectl taint no node1 memory=hight:NoExecute
node/node1 tainted

# （上面标记了node1污点NoExecute， 所以现在都被调度到node2了）

[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS        AGE     IP              NODE    NOMINATED NODE   READINESS GATES
hello-29022568-g8xmj            0/1     Completed   0               2m12s   10.244.104.52   node2   <none>           <none>
hello-29022569-vcjhd            0/1     Completed   0               72s     10.244.104.50   node2   <none>           <none>
hello-29022570-jz6px            0/1     Completed   0               12s     10.244.104.51   node2   <none>           <none>
nginx-deploy-7bcfd8bf84-84fkk   1/1     Running     0               2m35s   10.244.104.45   node2   <none>           <none>
nginx-deploy-7bcfd8bf84-kwk6c   1/1     Running     0               2m35s   10.244.104.46   node2   <none>           <none>
nginx-deploy-7bcfd8bf84-nh2kz   1/1     Running     1 (7h33m ago)   2d23h   10.244.104.63   node2   <none>           <none>
nginx-sc-0                      1/1     Running     0               5m39s   10.244.104.30   node2   <none>           <none>
# 删除污点
[root@master ~]# kubectl taint no node1 memory=low:NoSchedule-
node/node1 untainted
[root@master ~]# kubectl taint no node1 memory=height:NoExecute-
node/node1 untainted

[root@master ~]# kubectl delete po nginx-deploy-7bcfd8bf84-84fkk nginx-deploy-7bcfd8bf84-kwk6c nginx-deploy-7bcfd8bf84-nh2kz
pod "nginx-deploy-7bcfd8bf84-84fkk" deleted
pod "nginx-deploy-7bcfd8bf84-kwk6c" deleted
pod "nginx-deploy-7bcfd8bf84-nh2kz" deleted

[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022568-g8xmj            0/1     Completed   0          2m39s   10.244.104.52    node2   <none>           <none>
hello-29022569-vcjhd            0/1     Completed   0          99s     10.244.104.50    node2   <none>           <none>
hello-29022570-jz6px            0/1     Completed   0          39s     10.244.104.51    node2   <none>           <none>
nginx-deploy-7bcfd8bf84-wmc52   1/1     Running     0          4s      10.244.166.150   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-wrq8c   1/1     Running     0          4s      10.244.166.148   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-wzfpn   1/1     Running     0          4s      10.244.104.49    node2   <none>           <none>
nginx-sc-0                      1/1     Running     0          6m6s    10.244.104.30    node2   <none>           <none>
[root@master ~]# 
```

#### 容忍（Toleration是标注在 pod 上的，当 pod 被调度时，如果没有配置容忍，则该 pod 不会被调度到有污点的节点上，只有该 pod 上标注了满足某个节点的所有污点，则会被调度到这些节点）
##### Equal
```
[root@master ~]# kubectl taint node node2 memory=low:NoSchedule
node/node2 tainted

[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022579-fc7wp            0/1     Completed   0          2m13s   10.244.166.161   node1   <none>           <none>
hello-29022580-2g4ck            0/1     Completed   0          73s     10.244.166.160   node1   <none>           <none>
hello-29022581-wq68q            0/1     Completed   0          13s     10.244.166.159   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-wmc52   1/1     Running     0          10m     10.244.166.150   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-wrq8c   1/1     Running     0          10m     10.244.166.148   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-wzfpn   1/1     Running     0          10m     10.244.104.49    node2   <none>           <none>
nginx-sc-0                      1/1     Running     0          16m     10.244.104.30    node2   <none>           <none>
[root@master ~]# kubectl delete po nginx-deploy-7bcfd8bf84-wmc52 nginx-deploy-7bcfd8bf84-wrq8c nginx-deploy-7bcfd8bf84-wzfpn
pod "nginx-deploy-7bcfd8bf84-wmc52" deleted
pod "nginx-deploy-7bcfd8bf84-wrq8c" deleted
pod "nginx-deploy-7bcfd8bf84-wzfpn" deleted

# 可以发现在node2添加了污点后再重启pod就都跑到node1节点了
[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS   AGE    IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022580-2g4ck            0/1     Completed   0          2m6s   10.244.166.160   node1   <none>           <none>
hello-29022581-wq68q            0/1     Completed   0          66s    10.244.166.159   node1   <none>           <none>
hello-29022582-h6pfx            0/1     Completed   0          6s     10.244.166.162   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-76g2j   1/1     Running     0          10s    10.244.166.164   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-gmkh8   1/1     Running     0          10s    10.244.166.167   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-ssgnz   1/1     Running     0          10s    10.244.166.163   node1   <none>           <none>
nginx-sc-0                      1/1     Running     0          17m    10.244.104.30    node2   <none>           <none>
[root@master ~]#

```

```
# 添加容忍
[root@master ~]# kubectl edit deploy nginx-deploy
deployment.apps/nginx-deploy edited

spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      tolerations: # 主要在这里添加equal容忍
      - key: "memory"
        operator: "Equal"
        value: "low"
        effect: "NoSchedule"
      containers:
      - image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest
        imagePullPolicy: IfNotPresent

[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS              RESTARTS   AGE    IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022584-wk9vn            0/1     Completed           0          3m     10.244.166.165   node1   <none>           <none>
hello-29022585-ccbxw            0/1     Completed           0          2m     10.244.166.171   node1   <none>           <none>
hello-29022586-87zlf            0/1     Completed           0          60s    10.244.166.168   node1   <none>           <none>
hello-29022587-stggg            0/1     ContainerCreating   0          0s     <none>           node1   <none>           <none>
nginx-deploy-6d6bbbfbf4-9xsw7   1/1     Running             0          8s     10.244.104.53    node2   <none>           <none>
nginx-deploy-6d6bbbfbf4-ddp4j   0/1     PodInitializing     0          2s     10.244.166.173   node1   <none>           <none>
nginx-deploy-6d6bbbfbf4-z9zh6   1/1     Running             0          5s     10.244.166.172   node1   <none>           <none>
nginx-deploy-7bcfd8bf84-gmkh8   1/1     Running             0          5m4s   10.244.166.167   node1   <none>           <none>
nginx-sc-0                      1/1     Running             0          22m    10.244.104.30    node2   <none>           <none>
[root@master ~]#

# 可以发现部分pod又跑回到了node2，不会都被赶去node1了
```

##### Exists 容忍与污点的比较只比较 key，不比较 value，不关心 value 是什么东西，只要 key 存在，就表示可以容忍。
```
# 添加Exists容忍
[root@master ~]# kubectl edit deploy nginx-deploy

      tolerations:
      - effect: NoSchedule
        key: memory
        operator: "Exists"

deployment.apps/nginx-deploy edited
[root@master ~]# kubectl get po -o wide
NAME                           READY   STATUS      RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022591-mxhzb           0/1     Completed   0          2m24s   10.244.166.177   node1   <none>           <none>
hello-29022592-dllwn           0/1     Completed   0          84s     10.244.166.176   node1   <none>           <none>
hello-29022593-sqntc           0/1     Completed   0          24s     10.244.166.175   node1   <none>           <none>
nginx-deploy-65696579d-4p7dx   1/1     Running     0          13s     10.244.166.179   node1   <none>           <none>
nginx-deploy-65696579d-7bnp2   1/1     Running     0          6s      10.244.166.180   node1   <none>           <none>
nginx-deploy-65696579d-lrcbn   1/1     Running     0          9s      10.244.104.55    node2   <none>           <none>
nginx-sc-0                     1/1     Running     0          28m     10.244.104.30    node2   <none>           <none>
[root@master ~]# 
```

##### 添加NoExecute容忍
```
[root@master ~]# kubectl taint no node2 memory=low:NoSchedule-
node/node2 untainted

[root@master ~]# kubectl taint no node2 memory=low:NoExecute
node/node2 tainted

[root@master ~]# kubectl get po -o wide
NAME                           READY   STATUS              RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022599-h7nw7           0/1     Completed           0          3m1s    10.244.166.191   node1   <none>           <none>
hello-29022600-zth9s           0/1     Completed           0          2m1s    10.244.166.189   node1   <none>           <none>
hello-29022601-t4ds7           0/1     Completed           0          61s     10.244.166.188   node1   <none>           <none>
hello-29022602-9vcjp           0/1     ContainerCreating   0          1s      <none>           node1   <none>           <none>
nginx-deploy-65696579d-4p7dx   1/1     Running             0          8m50s   10.244.166.179   node1   <none>           <none>
nginx-deploy-65696579d-7bnp2   1/1     Running             0          8m43s   10.244.166.180   node1   <none>           <none>
nginx-deploy-65696579d-r5wjb   1/1     Running             0          31s     10.244.166.190   node1   <none>           <none>
nginx-sc-0                     1/1     Running             0          30s     10.244.166.131   node1   <none>           <none>
[root@master ~]#


[root@master ~]# kubectl edit deploy nginx-deploy
deployment.apps/nginx-deploy edited

    tolerations:
    - key: "memory"
      operator: "Exists"
      value: "value1"
      effect: "NoExecute" # 如果 Pod 能够容忍这类污点，但是在容忍度定义中没有指定 tolerationSeconds， 则 Pod 还会一直在这个节点上运行。
      tolerationSeconds: 3600 # 如果 Pod 能够容忍这类污点，而且指定了 tolerationSeconds， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。 这段时间过去后，节点生命周期控制器从节点驱除这些 Pod。

[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29022604-nmphl            0/1     Completed   0          2m36s   10.244.166.135   node1   <none>           <none>
hello-29022605-xlwjk            0/1     Completed   0          96s     10.244.166.136   node1   <none>           <none>
hello-29022606-z9mhs            0/1     Completed   0          36s     10.244.166.137   node1   <none>           <none>
nginx-deploy-8487ff77b4-2w5wt   1/1     Running     0          3m13s   10.244.104.58    node2   <none>           <none>
nginx-deploy-8487ff77b4-4w7bd   1/1     Running     0          3m15s   10.244.166.134   node1   <none>           <none>
nginx-deploy-8487ff77b4-847xz   1/1     Running     0          3m17s   10.244.104.54    node2   <none>           <none>
nginx-sc-0                      1/1     Running     0          5m5s    10.244.166.131   node1   <none>           <none>

```


### 亲和力（Affinity）
#### NodeAffinity（节点亲和力：进行 pod 调度时，优先调度到符合条件的亲和力节点上）
##### 分别给node1、node2 打标签
```
[root@master ~]# kubectl label no node1 label1=value1
node/node1 labeled
[root@master ~]# kubectl label no node2 label2=value2
node/node2 labeled
[root@master ~]# kubectl get no --show-labels
NAME     STATUS   ROLES                  AGE    VERSION   LABELS
master   Ready    control-plane,master   160d   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ingress=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=master,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
node1    Ready    <none>                 160d   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,ingress=true,kubernetes.io/arch=amd64,kubernetes.io/hostname=node1,kubernetes.io/os=linux,label1=value1,type=microservices
node2    Ready    <none>                 160d   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node2,kubernetes.io/os=linux,label2=value2,type=microservices
[root@master ~]# 
```

##### 修改deploy，添加亲和力配置
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx-deploy
spec:
  progressDeadlineSeconds: 600 # 设置滚动更新的超时时间（秒）
  replicas: 3 # 副本数量
  revisionHistoryLimit: 10 # 记录历史版本数量，方便回滚
  selector:
    matchLabels:
      app: nginx-deploy # 选择匹配的 Pod
  strategy:
    type: RollingUpdate # 采用滚动更新策略
    rollingUpdate:
      maxSurge: 25% # 在更新过程中，最多可以比期望的 Pod 数多 25%
      maxUnavailable: 25% # 在更新过程中，最多可以有 25% 的 Pod 不可用
  template:
    metadata:
      labels:
        app: nginx-deploy # Pod 标签
    spec:
      affinity: # 定义亲和性调度规则
        nodeAffinity: # 节点亲和性
          requiredDuringSchedulingIgnoredDuringExecution: # 必须满足的规则
            nodeSelectorTerms: # 定义节点选择条件
            - matchExpressions: # 匹配表达式列表
              - key: kubernetes.io/os # 需要匹配的节点标签（Key）
                operator: In # 操作符，表示 Key 的值必须在 values 列表内
                values: # 允许的值列表
                - linux # 只允许调度到 "kubernetes.io/os=linux" 的节点上
          preferredDuringSchedulingIgnoredDuringExecution: # 优先选择的规则
          - weight: 1 # 权重值（数值越高，优先级越高）
            preference: # 偏好设置
              matchExpressions: # 匹配表达式列表
              - key: label1 # 需要匹配的节点标签（Key）
                operator: In # 操作符，表示 Key 的值必须在 values 列表内
                values: # 允许的值列表
                - value1 # 该节点有 "label1=value1" 时，权重+1
          - weight: 50 # 权重值（比上面的 1 更高，优先级更高）
            preference: # 偏好设置
              matchExpressions: # 匹配表达式列表
              - key: label2 # 需要匹配的节点标签（Key）
                operator: In # 操作符，表示 Key 的值必须在 values 列表内
                values: # 允许的值列表
                - value2 # 该节点有 "label2=value2" 时，权重+50
      containers:
      - name: nginx # 容器名称
        image: swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/nginx:latest # 指定镜像
        ports:
        - containerPort: 80 # 暴露容器端口
-- INSERT --
```

##### 重启deploy
```
[root@master ~]# kubectl get po -o wide
NAME                            READY   STATUS      RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
hello-29028290-67qpw            0/1     Completed   0          2m57s   10.244.166.160   node1   <none>           <none>
hello-29028291-thq62            0/1     Completed   0          117s    10.244.166.158   node1   <none>           <none>
hello-29028292-kjc9m            0/1     Completed   0          57s     10.244.166.155   node1   <none>           <none>
nginx-deploy-5d64d79495-79b2p   1/1     Running     0          5s      10.244.104.4     node2   <none>           <none>
nginx-deploy-5d64d79495-j8g95   1/1     Running     0          12s     10.244.104.43    node2   <none>           <none>
nginx-deploy-5d64d79495-s69pn   1/1     Running     0          9s      10.244.104.3     node2   <none>           <none>
nginx-sc-0                      1/1     Running     0          57m     10.244.166.166   node1   <none>           <none>
[root@master ~]# 
```

##### 匹配类型Kubernetes `matchExpressions` 匹配类型

| **匹配类型** | **说明** | **示例** |
|-------------|---------|---------|
| `In` | 键的值必须在 `values` 列表中 | `key: kubernetes.io/os`<br>`operator: In`<br>`values: [linux]`<br>（只允许 `linux` 节点） |
| `NotIn` | 键的值不能在 `values` 列表中 | `key: kubernetes.io/os`<br>`operator: NotIn`<br>`values: [windows]`<br>（排除 `windows` 节点） |
| `Exists` | 节点必须包含该键（值无关紧要） | `key: label1`<br>`operator: Exists`<br>（只要 `label1` 存在，就匹配） |
| `DoesNotExist` | 节点不能包含该键 | `key: label1`<br>`operator: DoesNotExist`<br>（如果 `label1` 存在，则不匹配） |
| `Gt` | 键的值必须大于 `values` 中的单个数值 | `key: cpu-count`<br>`operator: Gt`<br>`values: [4]`<br>（要求 `cpu-count` 大于 4） |
| `Lt` | 键的值必须小于 `values` 中的单个数值 | `key: memory-size`<br>`operator: Lt`<br>`values: [16]`<br>（要求 `memory-size` 小于 16） |

---
**说明**：
- `In` 和 `NotIn` 需要提供 `values` 列表。
- `Exists` 和 `DoesNotExist` 只判断键是否存在，不需要 `values`。
- `Gt` 和 `Lt` 适用于数值类型的标签，`values` 只能包含一个数值。

#### PodAffinity（Pod 亲和力：将与指定 pod 亲和力相匹配的 pod 部署在同一节点）与PodAntiAffinity （Pod 反亲和力：根据策略尽量部署或不部署到一块）

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      affinity:
        podAffinity: # Pod 亲和性 
          requiredDuringSchedulingIgnoredDuringExecution: # 强制亲和规则->新 Pod 必须 被调度到 已有 app=my-app 的 Pod 所在的同一 Node，否则调度失败
          - labelSelector:
              matchLabels:
                app: my-app # 目标 Pod 必须包含这个标签
            topologyKey: "kubernetes.io/hostname" # 目标 Pod 必须在相同主机上

          preferredDuringSchedulingIgnoredDuringExecution: # 软性亲和规则->新 Pod 优先 调度到 已有 app=my-app 的 Pod 所在的相同可用区，但如果无法满足，会被调度到其他区域。
          - weight: 50 # 优先级权重
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: my-app
              topologyKey: "topology.kubernetes.io/zone" # 目标 Pod 优先调度到相同可用区

        podAntiAffinity: # Pod 反亲和性
          requiredDuringSchedulingIgnoredDuringExecution: # 强制反亲和规则->如果 node-1 上已经有 app=nginx 的 Pod，新的 Pod 不能再被调度到 node-1。
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname" # 避免调度到相同主机

          preferredDuringSchedulingIgnoredDuringExecution: # 软性反亲和规则->尽量让 nginx Pod 分布在不同的可用区，但如果资源不足，仍然可能调度到相同的区域
          - weight: 30 # 优先级权重
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: nginx
              topologyKey: "topology.kubernetes.io/zone" # 尽量不调度到相同可用区

      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

```




## 身份认证与权限（Kubernetes 中提供了良好的多租户认证管理机制，如 RBAC、ServiceAccount 还有各种策略等。通过该文件可以看到已经配置了 RBAC 访问控制/usr/lib/systemd/system/kube-apiserver.service）
### 认证
#### User Accounts 与 Service Accounts
| **类别**          | **User Accounts** | **Service Accounts** |
|-------------------|------------------|----------------------|
| **对象**          | 人类用户         | Pod 或 Controller    |
| **管理方式**      | 外部身份管理（OIDC、LDAP） | Kubernetes 自动管理 |
| **使用场景**      | `kubectl`、集群管理 | Pod 访问 API Server |
| **存储位置**      | `kubeconfig` 配置文件 | Kubernetes Secret（或 Token Projection） |
| **认证方式**      | 证书、Token、OIDC | Service Account Token |

#### Service Account 相关控制器
| **控制器**                      | **作用** |
|---------------------------------|---------|
| **Service Account Admission Controller** | 在创建 Pod 时，自动为其分配默认的 Service Account，并自动挂载 API 访问 Token。 |
| **Token Controller** | 负责为 Service Account 生成长期 Token（逐步被 Token Projection 取代）。 |
| **Service Account Controller** | 在新 `namespace` 创建时，自动创建 `default` Service Account。 |

#### Service Account Token 机制
| **Token 类型** | **特点** |
|---------------|---------|
| **长期 Token**（基于 Secret，逐步废弃） | 由 `Token Controller` 生成，存储在 `Secret` 资源中，长期有效，存在安全风险。 |
| **短期 Token**（Service Account Token Projection） | 动态创建，短时间有效，绑定 Pod 生命周期，减少 Token 泄露风险。 |

```
[root@master ~]# kubectl get sa
NAME      SECRETS   AGE
default   1         161d
[root@master ~]# kubectl get sa -n kube-system
NAME                                 SECRETS   AGE
attachdetach-controller              1         161d
bootstrap-signer                     1         161d
calico-kube-controllers              1         161d
calico-node                          1         161d
certificate-controller               1         161d
clusterrole-aggregation-controller   1         161d
coredns                              1         161d
cronjob-controller                   1         161d
daemon-set-controller                1         161d
default                              1         161d
deployment-controller                1         161d
disruption-controller                1         161d
endpoint-controller                  1         161d
endpointslice-controller             1         161d
endpointslicemirroring-controller    1         161d
ephemeral-volume-controller          1         161d
expand-controller                    1         161d
generic-garbage-collector            1         161d
horizontal-pod-autoscaler            1         161d
job-controller                       1         161d
kube-proxy                           1         161d
metrics-server                       1         25d
namespace-controller                 1         161d
nfs-client-provisioner               1         9d
node-controller                      1         161d
persistent-volume-binder             1         161d
pod-garbage-collector                1         161d
pv-protection-controller             1         161d
pvc-protection-controller            1         161d
replicaset-controller                1         161d
replication-controller               1         161d
resourcequota-controller             1         161d
root-ca-cert-publisher               1         161d
service-account-controller           1         161d
service-controller                   1         161d
statefulset-controller               1         161d
token-cleaner                        1         161d
ttl-after-finished-controller        1         161d
ttl-controller                       1         161d
[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   161d
ingress-nginx     Active   23d
kube-node-lease   Active   161d
kube-public       Active   161d
kube-system       Active   161d
[root@master ~]# kubectl get sa -n ingress-nginx
NAME            SECRETS   AGE
default         1         23d
ingress-nginx   1         21d
[root@master ~]#
```
#### 使用场景
```
Service Account 在 Pod 中的作用

Kubernetes 默认会为每个 Pod 关联一个 Service Account，并自动提供 API 访问凭证（Token）。应用程序可以使用这个凭证来与 Kubernetes API Server 进行通信，比如：

-- 监控应用（如 Prometheus Operator） 需要从 API Server 获取 Pod、Service、Metrics 信息。
-- CI/CD 工具（如 ArgoCD） 需要管理集群资源，创建或删除 Pod。
-- 自定义控制器（Operator） 监听 API Server 事件并进行自动化管理。
```

```
Service Account 如何工作

Pod 启动时，Kubernetes 会自动为它分配一个 Service Account（默认是 default，除非指定了其他 Service Account）。
Pod 内会自动挂载一个 Service Account Token（JWT 格式的凭证），路径通常是：/var/run/secrets/kubernetes.io/serviceaccount/token

Pod 内的应用可以使用这个 Token 访问 API Server，但权限取决于 Service Account 绑定的 RBAC 规则。
```



### 授权

| **资源**            | **作用范围**       | **作用** | **适用场景** |
|-------------------|----------------|--------|------------|
| **Role**         | 仅限于 **指定命名空间** | 定义该命名空间内的访问权限 | 适用于限制某个命名空间内的权限，例如 `dev` 命名空间的开发者只能查看 Pod |
| **ClusterRole**  | **整个集群**（跨命名空间） | 定义集群级别的访问权限 | 适用于需要跨多个命名空间或涉及集群资源（如 `nodes`、`namespaces`、`clusterroles`）的权限管理 |
| **RoleBinding**  | 仅限于 **指定命名空间** | 将 `Role` 绑定到用户、组或 Service Account | 适用于在 `default` 命名空间给某个用户分配 `Role` |
| **ClusterRoleBinding** | **整个集群**（跨命名空间） | 将 `ClusterRole` 绑定到用户、组或 Service Account | 适用于赋予某个用户跨命名空间的权限，例如 `cluster-admin` 权限 |
#### 示例：允许 Pod 读取 API Server 的 Pod 信息
```
# 创建一个 Service Account
apiVersion: v1 # 使用 Kubernetes v1 API 版本
kind: ServiceAccount # 定义资源类型为 ServiceAccount
metadata:
  name: pod-reader # 设置 Service Account 名称为 pod-reader
  namespace: default # 该 Service Account 归属于 default 命名空间
作用：创建一个名为 pod-reader 的 Service Account，属于 default 命名空间。
```

```
# 赋予该 Service Account 读取 Pod 的权限

apiVersion: rbac.authorization.k8s.io/v1 # 使用 RBAC v1 API 版本
kind: Role  # 定义资源类型为 Role
metadata:
  namespace: default # 该 Role 仅适用于 default 命名空间
  name: pod-read-role # 该 Role 的名称为 pod-read-role
rules:
- apiGroups: [""] # 空字符串表示访问核心 API 组
  resources: ["pods"] # 作用于 Pod 资源
  verbs: ["get", "list"]  # 允许执行 get 和 list 操作
---
apiVersion: rbac.authorization.k8s.io/v1 # 使用 RBAC v1 API 版本
kind: RoleBinding # 定义资源类型为 RoleBinding
metadata:
  namespace: default # 该 RoleBinding 作用于 default 命名空间
  name: bind-pod-reader # RoleBinding 的名称为 bind-pod-reader
subjects:
- kind: ServiceAccount # 绑定的主体类型为 ServiceAccount
  name: pod-reader  # 绑定的 ServiceAccount 名称
  namespace: default # 该 ServiceAccount 所在的命名空间
roleRef:
  kind: Role # 绑定的角色类型为 Role
  name: pod-read-role # 绑定的 Role 名称
  apiGroup: rbac.authorization.k8s.io # 使用 RBAC API 组

# 作用：允许 pod-reader 读取 default 命名空间下的 Pod 信息。注意：仅 get 和 list 权限，不能创建、删除 Pod。
```

```
# 让 Pod 使用这个 Service Account

apiVersion: v1 # 使用 Kubernetes v1 API 版本
kind: Pod # 定义资源类型为 Pod
metadata:
  name: api-client # 设置 Pod 的名称为 api-client
  namespace: default # 该 Pod 归属于 default 命名空间
spec:
  serviceAccountName: pod-reader  # 绑定 pod-reader 作为 Service Account
  containers:
  - name: curl-container  # 容器名称为 curl-container
    image: curlimages/curl # 使用 curl 官方镜像
    command: ["sleep", "3600"] # 运行 sleep 命令，保持容器运行
# 作用：这个 Pod 使用 pod-reader 作为 Service Account。运行后，它可以使用 Service Account Token 访问 API Server。
```

```
# 在 Pod 内访问 API Server

kubectl exec -it api-client -- /bin/sh # 进入 api-client Pod 的 Shell 终端
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) # 读取 Service Account Token
curl -s --header "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/default/pods # 使用 Bearer Token 通过 API Server 查询 default 命名空间下的 Pod 列表

```
