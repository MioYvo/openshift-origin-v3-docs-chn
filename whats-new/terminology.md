# Terminology

> 术语

### 概览

由于OpenShift v3中的架构变化，OpenShift v2中使用的一些核心术语已经改变，以更好地反映新模型。下面的小姐高亮了一些重要的改变。新模型的概念和对象的细节信息，查看[核心概念\(Core Concepts\)](https://docs.openshift.org/latest/architecture/core_concepts/index.html#architecture-core-concepts-index)话题。

##### Application

一个具体的_应用_术语或概念在OpenShift v3中不再存在。更多这个改变的深度解析，查看[应用](https://docs.openshift.org/latest/whats_new/applications.html#whats-new-applications)话题。

##### Cartridge vs Image

在OpenShift v3 中，_墨盒\(cartridge\)_最简单的替换，是[_镜像\(image\)_](https://docs.openshift.org/latest/architecture/core_concepts/containers_and_images.html#docker-images)。从包装的角度来看，镜像不仅仅是一个墨盒，它提供了更好的封装和灵活性。但是墨盒这个概念包含了一系列逻辑：构建、部署和路由，镜像并不包含这些。在OpenShift v3中，[Source-to-Image\(S2I\)](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#source-build)和[模板配置](https://docs.openshift.org/latest/architecture/core_concepts/templates.html#architecture-core-concepts-templates)满足了这些额外的需求。

更多关于这些改变的细节，查看[Cartridges vs Images](https://docs.openshift.org/latest/whats_new/carts_vs_images.html#whats-new-carts-vs-images)话题。

##### Project vs Domain

[Project](https://docs.openshift.org/latest/architecture/core_concepts/projects_and_users.html#projects)本质上是一个OpenShift v2中_domain_的重命名。Projects也有一些v2 中domains不具有的特性。

##### Gear vs Container

_齿轮\(gear\)_和[_容器\(container\)_](https://docs.openshift.org/latest/architecture/core_concepts/containers_and_images.html#containers)是可互换的。容器有一个更清晰的与镜像一比一的映射，而多个墨盒可被加入到一个齿轮中。对容器来说，pods满足了搭配的概念。

##### Master vs Broker

OpenShift v3 的[_Masters_](https://docs.openshift.org/latest/architecture/infrastructure_components/kubernetes_infrastructure.html#master)，其实是v2中的_broker_层。然而，OpenShift v2中所使用的MongoDB和ActiveMQ 层，在v3已不再需要，因为[键值对存储\(key-value store\)](https://docs.openshift.org/latest/architecture/infrastructure_components/kubernetes_infrastructure.html#master) **etcd**在每个master都安装了。

