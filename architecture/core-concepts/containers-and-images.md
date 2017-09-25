# 容器和镜像 Containers and Images

* Containers
    * Init Containers
* Images
* Container Registries

---

### 容器
OpenShift Origin应用的最基本单位称作*容器(containers)*。[Linux容器技术](https://access.redhat.com/articles/1353593)是用于隔离运行进程的轻量级机制，它们被限制只能与指定的资源交互。

很多应用进程可以运行在一个主机上的容器中，进程、文件、网络等等互相不可见。通常，每一个容器提供一个服务（经常被叫做“微服务 micro-service”），比如一个web服务器或者一个数据库，尽管容器可用于任意工作负载。

Linux内核多年来一直在集成容器技术的功能。最近Docker项目为主机上的Linux容器开发了一个方便的管理界面。OpenShift Origin 和 Kubernetes增加了跨主机编排Docker格式容器的能力。

当你使用OpenShift Origin时，尽管你不会直接与Docker CLI或者服务交互，但理解他们的功能和术语，对理解他们在OpenShift中的角色和应用程序在容器中是如何运行的，是很重要的。**docker**RPM作为RHEL 7 是可用的，包括CentOS和Fedora，所以你能在OpenShift Origin之外做实验。可浏览入门介绍文章：[Get Started with Docker Formatted Container Images on Red Hat Systems](https://access.redhat.com/articles/881893)。

#### 初始化容器 Init Containers
一个pod除应用程序容器之外，还可以有初始化容器。初始化容器让你能重新整理设置脚本和绑定代码(setup scripts and binding code)。初始化容器与常规容器不同之处在于它总是运行到完成。每个初始化容器必须在下一步开始之前成功完成。

更多信息，查看[Pods and Services](./pods-and-services.md)

### 镜像 Images
OpenShift Origin中的容器基于Docker格式的容器*镜像 images*。镜像是包含运行单个容器的所有要求的二进制文件，以及描述其需求和功能的元数据。

你可以把它理解为一种打包(packaging)技术。容器只能访问镜像中定义过的资源，除非在创建容器时给容器额外的访问权限。通过在不同主机不同容器中部署相同的镜像，并且负载均衡它们，OpenShift Origin为打包成镜像的服务提供了冗余和水平的缩放(scaling)。

你可使用Docker CLI（命令行界面）直接构建镜像，但OpenShift Origin也支持构建器镜像(builder image)，它通过添加你的代码或者配置到已有镜像中，来帮助你创建新镜像。

因为应用程序随着时间的推移而发展，所以单个镜像的名称实际上可以引用许多不同版本的“相同”图像。每个不同的镜像通过哈希（长十六进制数字，例如`fd44297e2ddb050ec4f...`，这里通常被缩短为12个字符，例如`fd44297e2ddb`）来唯一地引用。（译者注：每个镜像都有唯一的哈希，镜像名字和哈希之间是一对多的关系）。

