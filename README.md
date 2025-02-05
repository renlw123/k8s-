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
















