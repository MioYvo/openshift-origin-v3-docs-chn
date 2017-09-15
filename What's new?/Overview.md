# 概论 Overview
OpenShift v3 带了很多架构上的改变，以及新的概念和组件。它是围绕着运行在Docker[容器](https://docs.openshift.org/latest/architecture/core_concepts/containers_and_images.html#containers)中的应用来构建的，由[Kubernetes](http://kubernetes.io/)项目提供调度和管理的支持，此外还有增强部署、编排和路由功能。

最重要的改变是围绕着容器模型，以及它们如何被监控和互联。Kubernetes [pods](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#pods)是容器的组合，看起来像一个虚拟机：它们共享一个IP地址，它们可以分享同一个文件系统，它们有相似的安全设置。组合到一起的容器们大幅提高了OpenShift能够承载的应用数量。pods使开发者能够移植那些需要共享本地资源的已有应用，同时还能享受以容器为基础的模型带来的好处，而不仅仅是专注于微服务模型而排除其他模型。

第二个改变，OpenShift的容器应该是不可变的：镜像(Image)保存了应用代码和依赖组件的快照，其它的配置、密钥和持久化存储在运行时挂载进来。这让管理员和集成商将代码、补丁，与配置、数据分开。虽然容器仍然可以被改变，对于正在运行的代码，更高层级(higher level)的构建和部署的概念，提供了更高的不可变容器的保证。

第三个重要的改变在核心系统的设计：OpenShift和Kubernetes作为一系列微服务中的一环，通过通用的[REST APIs](https://docs.openshift.org/latest/rest_api/index.html#rest-api-index)来一起工作。这些相同的APIs对系统集成商可用，还可以禁用这些相同的核心组件来使用其他替代品。最好的例子是**watch** APIs: clients能够连接和接收一系列pods（也可以是其它对象）改变的流，当pods可用时做出相应的动作（来记录错误和日志到系统中）。OpenShift对Open APIs提供了细粒度的访问控制使这个集成实现，这也就意味着集成商在系统中可以做到想做的任何事。

接下来的话题会提供更多关于新架构和新概念的信息，帮助你理解OpenShift v3的新变化。

