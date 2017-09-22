# Kubernetes 基础设施

* Overview 概述
* Masters 主节点
    * High Availablility Masters 高可用主节点
* Nodes 节点
    * kubelet
    * Service Proxy
    * Node Object Definition

---

### Overview 概述
在OpenShift Origin中，Kubernetes跨容器或主机管理容器化的应用，提供了部署、维护和应用扩展的机制。Docker服务于包(packages)、实例化和运行容器化应用。

一个Kubernetes集群由一个或多个主节点(masters)，和一些节点(nodes)构成。你可以把主节点配置城高可用的(high availability(HA))，来保证集群没有单点故障。

* OpenShift Origin 使用Kubernetes 1.6和Docker 1.12。

### Masters 主节点
主机诶单是一个或多个主机(host)构成，它包含主节点组件，包括API服务器、控制器管理服务器(controller manager manager server)和**etcd**。主节点在它的Kubernetes集群中管理节点(nodes)，调度[pods](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#pods)运行在节点上。

*表1. 主节点组件*

|组件|描述|
|:---|:---|
|API服务器|Kubernetes API服务器验证和配置pod，服务(services)和副本控制器(replication controllers)的数据。它还将pod分配给节点，并将pod信息与服务配置同步。 可以作为独立进程运行。|
|**etcd**|**etcd**存储持久化主节点状态，其它组件监视**etcd**的变化来改变自己到想要状态。**etcd**可以被设置为高可用（可选），通常部署于2n+1对等服务(peer services)。|
|控制器管理服务器 Controller Manager Server|控制器管理服务器监视**etcd**中复本的变化，然后使用API强制执行所需的状态。可以作为独立进程运行。一些这样的进程一次创建一个具有一个主动领导(leader)者的集群。|
|HAProxy|可选的，当用`native`方法配置高可用主节点时使用，用于平衡API主端点(endpoints)之间的负载。[高级安装方法](https://docs.openshift.org/latest/install_config/install/advanced_install.html#install-config-install-advanced-install)可以为你用`native`方法配置HAProxy|


### 高可用主节点 High Availability Masters
在一个单主节点配置中，如果主节点或其任何服务故障，运行的应用程序的可用性将被保留。但是，主节点服务的故障，会降低系统响应应用故障或创建新应用的能力。可选的，你可以配置你的主节点为高可用的，来保证集群没有单点故障。

为了减轻对主节点可用性的担忧，两个建议：

1. 应该创建一个重建主节点的运行手册。一个运行手册是任何高可用服务的必要支持。额外的解决方法只是控制必须查阅运行手册的频率。例如，主节点的冷待机(cold standby)可以充分满足SLA，SLA需要不超过几分钟的停机时间来创建新应用程序或恢复失败的应用程序组件。
2. 使用高可用方案来配置你的主节点，保证集群没有单点故障。[高级安装方法](https://docs.openshift.org/latest/install_config/install/advanced_install.html#install-config-install-advanced-install)提供了具体的例子，使用`native`HA方法并且配置了HAProxy。你也可以使用`native`方法而不用HAProxy，应用这些概念到你已有的HA方案中。

* 不支持在安装之后，从单主节点变为多主节点。

当使用具有HAProxy的`native` HA方法时，主节点的组件有一下好处：

*表2.HAProxy带来的好处*

|角色 role |风格 style|注 notes|
|:---|:---|:---|
|**etcd**|主动-主动|通过负载均衡完全冗余部署|
|API服务器|主动-主动|被HAProxy管理|
|控制器管理服务器|主动-被动|一次一个实例被选为集群领导者|
|HAProxy|主动-被动|平衡API主端点之间的负载|

### Nodes节点
节点提供容器的运行时环境。每一个Kubernetes集群中的节点都具有要由主节点管理的所需服务。节点还具有运行pod的所需服务，包括Docker服务，kubelet和服务代理(service proxy)。

OpenShift Origin 从云提供商、物理系统或虚拟系统中创建节点。Kubernetes与节点对象(node objects)进行交互，节点对象代表这些节点。主节点使用来自节点对象的信息，通过健康检查来验证节点。一个节点被忽略，直到它通过健康检查，主节点继续检查节点直到它们被验证。[Kubernetes文档](https://kubernetes.io/docs/concepts/architecture/nodes/#management)有更多关于节点管理的信息。

管理元可以通过CLI在OpenShift Origin实例中管理节点。在加载节点服务器时，为了定义完整的配置和安全选项，使用[可被探测的节点配置文件](https://docs.openshift.org/latest/install_config/master_node_configuration.html#install-config-master-node-configuration)。

* 建议的节点最大数量是300。

### kubelet
每个节点都有一个kubelet，更新节点容器清单(manifest)中的节点，容器清单是描述一个pod的YAML文件。kubelet使用一系列清单来保证它的容器启动并继续运行。

一个容器清单可以被提供给kubelet 通过：

* 命令行上的文件路径，每20秒检查一次。
* 在命令行上传递的HTTP端点，每20秒检查一次。
* kubelet监视**etcd**服务器，比如`/registry/hosts/$(hostname -f)`，在任何更改时作出动作
* kubelet监听HTTP，响应一个简单的API来提交新清单。

### Service Proxy 服务代理
每个节点也运行了一个简单的网络代理，它反射在节点上API中定义的服务。这允许节点在一组后端执行简单的TCP和UDP流转发。

### 节点对象定义
下面是一个Kubernetes中节点定义的例子：

```
apiVersion: v1 ①
kind: Node ②
metadata:
  creationTimestamp: null
  labels: ③
    kubernetes.io/hostname: node1.example.com
  name: node1.example.com ④
spec:
  externalID: node1.example.com ⑤
status:
  nodeInfo:
    bootID: ""
    containerRuntimeVersion: ""
    kernelVersion: ""
    kubeProxyVersion: ""
    kubeletVersion: ""
    machineID: ""
    osImage: ""
    systemUUID: "" 
```

* ① `apiVersion`定义使用的API版本
* ② `kind`设为`node`，把这个看成是节点对象的定义
* ③ `metadata.labels`列出了任意被已加入到节点中的[标签(labels)](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#labels)
* ④ `metadata.name`是一个必需值，它定义了节点对象的名字。当你运行`oc get nodes`命令时，这个值显示在`NAME`列。
* ⑤ `spec.externalID`定义可以到达节点的完全限定域名。空时默认为`metadata.name`值。

[REST API参考](https://docs.openshift.org/latest/rest_api/kubernetes_v1.html#v1-node)有更多关于这些定义的细节。