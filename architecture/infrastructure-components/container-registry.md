# 容器注册表(Container Registry)
* 概述 Overview
* 集成的OpenShift容器注册表
* 第三方注册表
    * 认证

---
### 概述 Overview
OpenShift Origin可以使用任何实现了Docker注册表API的服务器作为图像来源，包括Docker Hub、由第三方运行的私人注册表以及集成的OpenShift Origin注册表。

### 集成的OpenShift容器注册表
OpenShift Origin提供了一个集成的容器注册表，叫做*Openshift Container Registry*(OCR)，它可以根据需要自动配置新的镜像存储库。这为用户提供了内置的应用构建位置，以推送生成的图像。

不论何时新镜像被推送到OCR，注册表通知OpenShift Origin并传递关于新镜像的所有信息，比如名字空间(namespace)、名字和镜像元数据(metadata)。不同的OpenShift Origin对新图像做出反应，创建新的[构建](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#builds)和[部署](https://docs.openshift.org/latest/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations)。

OCR也可以被部署为独立组件，完全表现为一个容器注册表，没有构建和部署的集成。查看[安装独立部署的OpenShift Container Registry](https://docs.openshift.org/latest/install_config/install/stand_alone_registry.html#install-config-installing-stand-alone-registry)获取详情。

### 第三方注册表
OpenShift Origin可以创建容器，使用第三方的镜像，但不提供OCR相似的镜像通知。在这种情况下，OpenShift Origin会在镜像流(imagestream)创造时从远端注册表获取标签(tags)。刷新已获取的标签，只需运行`oc import-image <stream>`。当新镜像被检测到，会发生先前描述的构建和部署反应。

#### 认证
OpenShift Origin可以与注册表沟通，使用用户提供的证书来访问私有的镜像存储。这使OpenShift Origin可以推送或拉取镜像到私有存储。[认证话题](https://docs.openshift.org/latest/architecture/additional_concepts/authentication.html#architecture-additional-concepts-authentication)有更多的详情。
