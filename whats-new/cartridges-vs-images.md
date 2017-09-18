# Cartridges vs Images

> 原文地址 https://docs.openshift.org/latest/whats_new/carts_vs_images.html

- <a href="#overview">概览</a>
- <a href="#dependecies">依赖</a>
- <a href="#collocation">搭配</a>
- <a href="#source-code">源码</a>
- <a href="#build">构建</a>
- <a href="#routing">路由</a>

---

### <a name="overview">概览</a>
在OpenShift v2中，墨盒(cartriges)是构建应用的关键点。每个墨盒提供了所需的库、源代码、构建机制、连接逻辑和路由逻辑，以及一个预设环境来运行应用的组件们。

然而，墨盒使开发者的内容和墨盒内容的区别界限不清晰。比如应用中所有的齿轮(gear)，你都没有home目录的所有权。另一个缺点是，对大型的二进制包来说墨盒并不是最好的分发机制。当你从墨盒中用额外的依赖做这些事，就会失去封装的好处。

### <a name="dependecies">依赖</a>
墨盒的依赖被定义在墨盒清单(cartridge manifest) 的`Configure-Order`或`Requires`。OpenShift v3使用声明式模型，pod会使自身与预定义的状态相符。做好的明确的依赖，会在运行时完成，而仅仅是在安装时排个序。

比如你要求另一个服务在你启动前可用。这样的依赖检查一直可用，而不只是你创建这两个组件时。因此，将依赖到推进到运行时依赖，让系统随着时间推移保持健康。

### <a name="collocation">搭配</a>
鉴于OpenShift v2中的墨盒并置在齿轮中，OpenShift v3 的 [images](https://docs.openshift.org/latest/architecture/core_concepts/containers_and_images.html#docker-images)与[containers](https://docs.openshift.org/latest/architecture/core_concepts/containers_and_images.html#containers)一比一映射。容器用[pods](https://docs.openshift.org/latest/architecture/core_concepts/pods_and_services.html#pods)作为搭配机制。

### <a name="source-code">源码</a>
在OpenShift v2中，应用被要求至少有一个含git库的eb框架。OpenShift v3中，你能选择哪些镜像是从源码构建，这里的源码可以在OpenShift外部。因为源码并不连接到镜像中，源码只是个选项，所以镜像和源码的选择被清晰隔离开。

### <a name="build">构建</a>
在OpenShift v2中，构建发生在您的应用齿轮的一部分。由于资源的限制，这些就意味着无扩展应用的停机时间（增加）。在v3中，构建发生在隔离的容器中。

另一个关于构建的变化时是，这些构建动作是如何在容器间传输的。在v2中，构建结果在齿轮间同步。而v3中的构建结果先辈提交作为不可变的镜像(image)被发布到内部仓库(registry)。这个镜像就可以被集群中任一节点上应用或回滚。

这些意味着构建不再是墨盒的一部分，而是被作为清晰的、不同的选择。

### <a name="routing">路由</a>
[路由(routing)](https://docs.openshift.org/latest/architecture/networking/routes.html#architecture-core-concepts-routes)是在OpenShift v3中经历了大量正式化的另一个领域。在v2中，你必须做选择：要么应用可扩展，要么应用中的路由层高可用。在OpenShift v3中，路由是通过将应用程序组件扩展到2个或更多副本的HA功能的第一类对象(first class objects)。永远不需要重建你的应用或者改它的DNS入口。

路由自己也提高了一个层级，不再与镜像(images)连接。之前墨盒定义了一个默认的路由集合，你能够为应用加入额外的别名。v3中，你可以用模板(templates)为任一镜像这是0到N个路由。这些路由让你更改schema、host、路径，而且系统路由和用户别名间没有差别。