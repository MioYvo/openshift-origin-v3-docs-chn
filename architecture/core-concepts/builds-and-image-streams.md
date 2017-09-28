# 构建和镜像流 Builds and Image Streams

* Builds
    * Docker Build
    * Source-to-Image (S2I) Build
    * Custom Build
    * Pipeline Build
* Image Streams
    * Image Stream Image
    * Image Stream Tag
    * Image Stream Mapping

---
### 构建 Build
*构建(Build)*是转换输入参数到结果对象(resulting object)的过程。大部分情况，这个过程是把输入参数或源代码转换成一个可运行的镜像(image)。`BuildConfig`对象是整个构建过程的定义。

OpenShift Origin通过从构建镜像(build image)创建Docker格式的容器并把它推到[容器注册表(container registry)](https://docs.openshift.org/latest/architecture/infrastructure_components/image_registry.html#integrated-openshift-registry)。

构建对象共享共同特征：构建的输入，完成构建过程的所需，记录构建过程，从成功构建发布资源以及发布构建的最终状态。构建利用资源限制，指定资源的限制，例如CPU使用率，内存使用量和构建或pod执行时间。

OpenShift Origin构建系统为基于build API中指定的可选类型的构建策略提供可扩展的支持。有三种可用的主要构建策略：
* Docker build
* Source-to-Image (S2I) build
* Custom build

默认情况下支持Docker builds和S2I builds。

构建生成的对象(resulting object)取决于用于创建它的构建器(builder)。对于Docker和S2I builds，结果对象是可运行的镜像。对于Custom builds，生成的对象是构建器镜像作者指定的任何内容。

此外，[管道构建](https://docs.openshift.org/latest/architecture/core_concepts/builds_and_image_streams.html#pipeline-build)策略可用来实现复杂的工作流：
* 持续集成 (continuous integration)
* 持续部署 (continuous deployment)

构建的命令列表，查看[开发者指导书](https://docs.openshift.org/latest/dev_guide/builds/index.html#dev-guide-how-builds-work)。

在构建上，更多OpenShift Origin 如何利用Docker的信息，查看[upstream documentation](https://github.com/openshift/origin/blob/master/docs/builds.md#how-it-works)。

#### Docker Build
Docker构建策略调用docker构建命令，因此它期望具有Dockerfile和所有必需组件的存储库(repository)以生成可运行映像。

#### Source-to-Image (S2I) Build
[Source-to-Image (S2I)](https://docs.openshift.org/latest/creating_images/s2i.html#creating-images-s2i)是一个用来构建可复制的、Docker格式的容器镜像的工具。它通过将应用程序源注入容器镜像并组装新镜像来生成随时可运行的映像(ready-to-run images)。新镜像包含基本镜像（构建器）和构建源码，并可使用`docker run`命令。S2I支持递增的构建，可重用预下载的依赖、预构建的组件等等。

S2I的优点包括：

|优点|解释|
|:---|:---|
|镜像灵活性 Image flexibility|S2I脚本可被重写以注入应用代码到几乎任意已存在的Docker格式容器镜像中，利用现存的生态系统。注意，现在S2I依靠`tar`来注入应用源码，所以镜像需要能处理被tar过的内容。|
|速度 Speed|用S2I,组装阔成可以执行大量复杂操作，而不会在每个不走中创建新层，从而实现一个快速的过程。而且，S2I脚本可以被重写来重用存在前一个版本的应用景象中的组件，而不必在它们每一次构建运行的时候再次下载或构建。|
|可补丁能力 Patchability|如果由于安全问题底层镜像需要补丁，S2I允许你一致地重建应用。|
|运营效率 Operational efficiency|通过限制构建操作，而不是像Dockerfile一样允许任意操作，PaaS操作员可以避免意外或故意滥用构建系统。|
|运营安全 Operational security|构建任意的Docker文件将主机系统暴露给root权限是有风险的。这可以被恶意用户利用，因为整个Docker构建过程作为具有Docker特权的用户运行。 S2I限制作为root用户执行的操作，并且能以非root用户身份运行脚本。|
|用户效率User efficiency|S2I在应用程序构建期间阻止开发人员执行任何类似`yum install`的会减慢开发迭代速度的操作。|
|生态系统|S2I鼓励镜像共享生态系统，你可以利用应用程序的最佳实践。|
|再生性|生成的镜像包括了所有输入，包括构建工具和依赖关系的特定版本。这确保了镜像可以精确地再现。|

#### Custom Build
Custome build(自定义构建)策略允许开发者定义指定的、为了整个构建过程负责的构建器镜像。使用你自己的构建器镜像让你能自定义你的构建过程。

一个[Custom Builder image（自定义构建器镜像）](https://docs.openshift.org/latest/creating_images/custom.html#creating-images-custom)是一个嵌入了构建过程逻辑的普通Docker格式容器镜像，比如构建RPM或基本镜像。`openshift/origin-custom-docker-builder`镜像在[Docker Hub](https://registry.hub.docker.com/u/openshift/origin-custom-docker-builder)注册表上，作为Custom builder image的示例实现。

#### Pipeline Build
Pipeline Build（管道构建）策略让开发者能定义一个Jenkins管道插件执行的*Jenkins pipeline*。这种构建像其它构建类型一样，可以被OpenShift Origin同样的方法启动、监控和管理。

管道工作流被定义在一个Jenkinsfile，可直接嵌入到构建配置，或在Git存储库中提供，并被构建配置引用。

应用第一次定义一个使用管道策略的构建配置时，OpenShift Origin实例化一个Jenkins服务器来执行管道。项目中随后的管道构建配置共享这个Jenkin服务器。

Jenkins服务器如何部署的，如何配置或禁用自动配置行为(autoprovisioning behavior)，查看[配置管道执行](https://docs.openshift.org/latest/install_config/configuring_pipeline_execution.html#install-config-configuring-pipeline-execution)。

* Jenkins服务器不会自动移除，就算所有的管道构建配置都被删除了。他必须被用户手动删除。

更多关于Jenkins管道的信息，请看[Jenkins文档](https://jenkins.io/doc/pipeline/)。

