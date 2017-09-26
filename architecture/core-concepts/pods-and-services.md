# Pods和服务 Pods and Services

* Pods
* Init Containers
* Services
    * Service ExternalIPs
    * Service ingressIPs
    * Service NodePort
    * Service Proxy Mode
* Labels
* Endpoints

---

### Pods
OpenShift Origin利用了Kubernetes的pod概念，它是一个或多个部署在一个主机上的[容器](./containers-and-images.md)，是可被定义、部署和管理的最小计算单元。

Pods与容器的机器实例（物理或虚拟）粗略相当（译者注：对于容器来说，pod相当于一台主机，pod中有一个或多个的容器，pod有自己的内网IP地址）。每个pod被分配内部IP地址，**因此它拥有所有的端口空间**，在pods中的容器可以分享他们自己的本地存储和网络。

Pods有生命周期；它们被定义，然后它们被分配到一个节点上运行，它们一直运行直到它们的容器退出或被某种原因移除掉。Pods，依赖策略和退出代码，可以在退出后被移除，或者保留以访问容器的日志。

* 建议每个OpenShift Origin节点最大pods数量是110。
* 节点中断时，不被复制控制器管理的空pods不会被重新安排(rescheduled)。

下面是一个pod的示例定义，它提供了一个长期运行的服务，它是OpenShift Origin基础设施的一部分：继承的容器注册表。它展示了pod的许多功能，其中大部分在其他主题中讨论，因此在此仅作简单介绍：

*Example 1. Pod Object Definition (YAML)*

          ⑪ ⑫ ⑬ ⑭ ⑮ ⑯ ⑰ ⑱ ⑲ ⑳

```
apiVersion: v1
kind: Pod
metadata:
  annotations: { ... }
  labels:                               ①           
    deployment: docker-registry-1
    deploymentconfig: docker-registry
    docker-registry: default
  generateName: docker-registry-1-      ②   
spec:
  containers:                           ③ 
  - env:                                ④ 
    - name: OPENSHIFT_CA_DATA
      value: ...
    - name: OPENSHIFT_CERT_DATA
      value: ...
    - name: OPENSHIFT_INSECURE
      value: "false"
    - name: OPENSHIFT_KEY_DATA
      value: ...
    - name: OPENSHIFT_MASTER
      value: https://master.example.com:8443
    image: openshift/origin-docker-registry:v0.6.2  ⑤
    imagePullPolicy: IfNotPresent
    name: registry
    ports:                              ⑥
    - containerPort: 5000
      protocol: TCP
    resources: {}
    securityContext: { ... }            ⑦
    volumeMounts:                       ⑧
    - mountPath: /registry
      name: registry-storage
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-br6yz
      readOnly: true
  dnsPolicy: ClusterFirst
  imagePullSecrets:
  - name: default-dockercfg-at06w
  restartPolicy: Always
  serviceAccount: default               ⑨
  volumes:                              ⑩
  - emptyDir: {}
    name: registry-storage
  - name: default-token-br6yz
    secret:
      secretName: default-token-br6yz
```

* ① Pods可被一个或多个labels“打标签”（译者注：tag和label都可译为标签），可在一个操作中选择和管理pods的集合。labels以key/value格式存储在`metadata`哈希中。一个label在这个例子中是`docker-registry=default`。
* ② Pods必须在它们自己的名字空间里有一个唯一的名字。一个pod定义也许指定了`generateName`属性作为基础名字，自动添加随机字符生成一个唯一的名字。
* ③ `containers`指定一个容器定义(definitions)的数组；在这里（也是大部分情况），只有一个容器。
* ④ 环境变量可以被指定，用来传递需要的值到每个容器。
* ⑤ pod中每个容器都是从它自己的[Docker格式容器的镜像](./containers-and-images.md)中实例化的。
* ⑥ 容器可以绑定到端口，在**pod的IP**上可用。
* ⑦ OpenShift Origin定义了容器的安全上下文，指定了它们是否允许作为特权(privileged)容器运行，作为他们选择的用户运行等等。默认上下文是很严格的，但管理员可以根据需要修改。
* ⑧ 容器指定了哪个外部存储卷(volumes)被挂载到容器内。在这个例子中，有一个卷来存储注册表的数据，还有一个卷用来访问注册表用来请求OpenShift Origin API的证书。
* ⑨ 针对Pods请求OpenShift Origin API是一个很常见的模式，它有一个`serviceAccount`字段，用于指定pods应该在发出请求时验证哪个[服务帐户(service account)](https://docs.openshift.org/latest/dev_guide/service_accounts.html#dev-guide-service-accounts)用户。 这样可以实现对自定义基础架构组件的细粒度访问控制。
* ⑩ pod定义存储卷，它可被自己的容器使用。在这个例子中，它提供了一个暂时卷为了注册表存储和一个`secret`卷来包含服务账户的证书。


* 这个pod定义不包括在pod被创建、它的生命周期开始时被OpenShift Origin自动填写的属性。[Kubernetes API 文档](https://docs.openshift.org/latest/rest_api/kubernetes_v1.html#rest-api-kubernetes-v1)有完整的关于pod REST API 对象属性的信息，[Kubernetes pod文档](https://kubernetes.io/docs/concepts/workloads/pods/pod/)有关于pod功能和目标的详情。


### 初始化容器 Init Conatiners
一个初始化容器，是一个pod里在pod的app容器启动前启动的容器。初始化容器可以分享卷、执行网络操作和剩下容器启动前执行操作。初始化容器也可以封锁和延迟应用容器启动，直到满足先决条件。

当一个pod启动，在网路和卷初始化后，初始化容器按顺序启动。每个初始化容器必须在下一步调用前成功退出。如果一个初始化容器启动失败（因为运行时(runtime)）或者退出失败，它会根据pod`restartPolicy`（重启策略）重试，[`restartPolicy`重启策略](https://docs.openshift.org/latest/dev_guide/configmaps.html#consuming-configmap-in-pods)：
    
* `Always` - 连续尝试重启，增长延迟(exponential back-off delay) (10秒，20秒，40秒)，直到pod重启。
* `Never` - 不尝试重启。Pods立即失败，退出。
* `OnFailure` - 尝试重启，增长延迟(10秒，20秒，40秒)，在5分钟内。

一个pod在所有初始化容器成功时才会准备好。

查看Kubernetes文档的一些[初始化容器使用示例](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#examples)。

下面的例子概述了一个简单的有两个初始化容器的pod。第一个初始化容器等待`myservice`，第二个等待`mydb`。一旦两个容器成功，pod启动。

*Example 2. Sample Init Container Pod Object Definition (YAML)*

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice  ①
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb       ②
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

* ① 指定`myservice`容器
* ② 指定`mydb`容器

每个初始化容器拥有[一个app容器的所有字段](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#example-pod-definition)，除了[`readinessProbe`（准备就绪探测器）](https://docs.openshift.org/latest/dev_guide/application_health.html#container-health-checks-using-probes)。初始化容器必须退出，让pod继续启动，除了完成不能定义准备就绪(cannot define readiness other than completion)。

初始化容器可以包含pod的[`activeDeadlinesSeconds`（活跃截止日期）](https://docs.openshift.org/latest/dev_guide/jobs.html#jobs-setting-maximum-duration)，和容器的[`livenessProbe`（活性探测器）](https://docs.openshift.org/latest/dev_guide/application_health.html#container-health-checks-using-probes)，来防止容器无限失败。活跃截止时间包括初始化容器。

### 服务 Services
一个Kubernetes服务，是作为一个内部负载均衡器。它识别复制pods的集合，以代理它们收到的连接。在服务保持一贯可用的情况下，可以任意添加到服务中或从服务中删除备份pod，从而使依赖于服务的任何东西能够在一致的地址处引用。默认服务集群IP地址是从OpenShift Origin 内部网络来的，并且它们被用来许可pods互相访问。

要许可外部访问服务，可以向服务分配集群[外部](https://docs.openshift.org/latest/dev_guide/getting_traffic_into_cluster.html#using-externalIP)的额外的`externalIP`（外部IP）和`ingressIP`（入口IP）地址。这些`externalIP`地址也可以是提供服务的[高可用](https://docs.openshift.org/latest/admin_guide/high_availability.html#admin-guide-high-availability)访问的虚拟IP地址。

服务被分配一个IP地址和端口对儿，当被访问时，代理到适当的后备荚pod。服务使用label选择器来查找在某个端口上提供某种网络服务的所有运行的容器。

与pods相似，服务是一个REST对象。下面的例子展示了为上面的pod而定义的服务：

*Example 3. Service Object Definition (YAML)*

```
apiVersion: v1
kind: Service
metadata:
  name: docker-registry         ①
spec:
  selector:                     ②
    docker-registry: default
  portalIP: 172.30.136.123      ③
  ports:
  - nodePort: 0
    port: 5000                  ④
    protocol: TCP
    targetPort: 5000            ⑤
```     

* ① 服务名`docker-registry`也被用来构建带一个有服务ip的环境变量，该服务IP插入到同一命名空间中的其他pod中。名字最大长度是63个字符。
* ② label选择器识别所有带有`docker-registry=default`标签的pod作为备用pod。
* ③ 服务的虚拟IP，在创建时从内部IP池中自动分配。
* ④ 服务监听的端口。
* ⑤ 服务转发连接的后端pod上的端口。

[Kubernetes文档](http://kubernetes.io/docs/user-guide/services/)有更多services的信息。

#### 服务外部IP Service externalIPs
除了集群内部IP地址，用户还可以配置集群外部的IP地址。管理员负责保证流量能够到达具有这个IP的节点。

外部IP必须是管理员从配置在[master-config.yaml](https://docs.openshift.org/latest/admin_guide/tcp_ingress_external_ports.html#unique-external-ips-ingress-traffic-configure-cluster)文件中的`ExternalIPNetworkCIDRs`范围中选择出。当*master-config.yaml*改变，主节点服务必须重启。

*Example 4. Sample ExternalIPNetworkCIDR /etc/origin/master/master-config.yaml*

```
networkConfig:
  ExternalIPNetworkCIDR: 172.47.0.0/24
```

*Example 5. Service externalIPs Definition (JSON)*

```json
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            }
        ],
        "externalIPs" : [
            "80.11.12.10"                   ①   
        ]
    }
}
```

① 端口暴露的外部IP地址列表（除内部IP地址外）


#### 服务入口IP Service ingressIPs
在非云的集群中，可以从地址池自动分配外部IP地址。这样就无需管理员手动分配。

地址池配置在*/etc/origin/master/master-config.yaml*文件中。更改这个文件后，重启主节点服务(master service)。

`ingressIPNetworkCIDR`默认设置为`172.29.0.0/16`。如果集群环境没有准备好实用这个私有范围，那就使用默认范围或设置一个自定义范围。

* 如果你使用[高可用](https://docs.openshift.org/latest/admin_guide/high_availability.html#admin-guide-high-availability)，那么这个范围必须小于256个地址。

*Example 6. Sample ingressIPNetworkCIDR /etc/origin/master/master-config.yaml*

networkConfig:
  ingressIPNetworkCIDR: 172.29.0.0/16
  
  
#### 服务节点端口 Service NodePort
设置服务`type=NodePort`会从标志配置范围（默认：30000-32767）分配一个端口，每个节点会代理这个端口（每个节点上的相同端口号）到你的服务。

被选择的端口会被报告在服务配置中，在`spec.ports[*].nodePort.`

要指定一个自定义端口，只需要把端口号填在nodePort字段里。自定义端口号必须在nodePorts配置的范围内。更改了*master-config.yaml*，必须要重启主节点服务(master service)。

*Example 7. Sample servicesNodePortRange /etc/origin/master/master-config.yaml*

```
kubernetesMasterConfig:
  servicesNodePortRange: ""
```

服务可见，在`<NodeIP>:spec.ports[].nodePort`和`spec.clusterIp:spec.ports[].port`

* 设置nodePort是一个特权操作。

#### 服务代理模式 Service Proxy Mode
OpenShift Origin有两个不同的服务路由基础设施的实现。默认实现是全iptables基础(entirely **iptables**-based)，并使用可能性(probabilistic)的**iptables**重写规则(rewriting rules)来在端点pod间(endpoint pods)分发进入的服务连接。旧的实现使用一个用户空间进程来接受进入的连接，然后在客户端和其中一个端点pod之间代理流量。

以**iptables**为基础的实现更加高效率，但它要求所有端点总是能接受连接；用户空间的实现更慢，但能依次尝试多个端点直到找到一个工作的端点。如果你有优秀的[准备就绪检查](readiness checks)（或者可靠的节点和pod），那么以**iptables**为基础的服务代理是最好的选择。否则，你可以在安装时启用用户空间为基础的代理，或者在部署集群后编辑节点配置文件。

### Labels
Labels被用来安排、分组或选择API对象。比如，pod用labels“打上标签”，然后服务用label选择器去识别它们代理的pods。这使得服务可以引用pods组，甚至将具有潜在不同容器的pod作为相关实体。

大部分对象能包括labes到它们的元数据(metadata)中。所以labels能用于分组任意相关的对象；比如，可以对一个特定应用所有的pods、服务(services)、副本控制器(replication controllers)和部署配置(deployment configurations)进行分组。

labels是简单的键值对(key/value pairs)，例如：

```
labels:
  key1: value1
  key2: value2
```

考虑:

* 一个pod由一个**nginx**容器构成，label是**role=webserver**。
* 一个pod有一个**Apache httpd**容器组成，也有相同的label**role=webserver**。

要使用**role=webserver**标签的pod的服务或副本控制器，可以把这两个pod当做同一个组的一部分。

[Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/user-guide/labels.md)文档有更多关于lables的信息。

### 端点 Endpoints
返回服务的服务器称为其端点，并且被指定为类型`Endpoints`的一个对象，使用与服务相同的名字。当服务由pod支持时，这些pod通常由服务规范中的标签选择器指定，OpenShift Origin会自动创建指向这些pod的端点对象。（译者注：endpoints的名字与服务相同，指向pod的ip和端口）

在一些情况中，你也许想创建一个服务，但使用外部主机支持而不是OpenShift Origin集群的pod。在这种情况下，你可以把`selector`字段留空，然后[手动创建端点对象](https://docs.openshift.org/latest/dev_guide/integrating_external_services.html#dev-guide-integrating-external-services)。

请注意，OpenShift Origin不会让大多数用户手动创建一个Endpoints对象，指向为[pod和服务IP保留的网络块中的IP地址](https://docs.openshift.org/latest/install_config/configuring_sdn.html#configuring-the-pod-network-on-masters)。只有[集群管理员](../../additional-concepts/authorization.md)或其他有[创建endpoints/restricted资源权限](https://docs.openshift.org/latest/architecture/additional_concepts/authorization.html#evaluating-authorization)的用户可以创建这样的Endpoint对象。
