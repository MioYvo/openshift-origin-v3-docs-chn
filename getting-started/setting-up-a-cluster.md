# 管理员：设置集群

* Overview
* Prerequisites
* Installation Methods
    * Method 1: Running in a Container
    * Method 2: Downloading the Binary
    * Method 3: Building from Source
* Try It Out

---
# 概述
OpenShift Origin 有多种安装方法，每种都能让你快速地启动并运行自己的OpenShift Origin示例。你可以依据你的环境选择最好的安装方式。

部署一个完全的OpenShift cluster，[查看高级安装指引](https://docs.openshift.org/latest/install_config/install/advanced_install.html#install-config-install-advanced-install)。

# 先决条件
在选择安装方式之前，你必须让你的主机[满足先决条件](https://docs.openshift.org/latest/install_config/install/prerequisites.html#install-config-install-prerequisites)，包括确认系统和环境的要求，安装、配置Docker。确认你的主机已经设置好后，接下来选择一种安装方式。

Docker和OpenShift Origin必须运行在Linux操作系统中。如果你希望运行在Windows或Mac OS X主机上，你必须按照方法3(本页，下方)中的描述，下载并运行Origin Vagrant镜像。

* OpenShift Origin 和Docker使用[iptables](https://docs.openshift.org/latest/admin_guide/iptables.html#admin-guide-iptables)管理网络。确保本地防火墙规则或或其它会更改iptabl的软件，不会改变OpenShift Origin和Docker服务的设置。

### 安装方法
选择下面安装方法中最适合你的一种。

### 方法1: 运行在容器中
你可以快速用Docker Hub中的镜像来快速的在Linux系统中运行OpenShift Origin。

* OpenShift Origin监听53和8443端口。如果其它服务已经监听这些端口，你需要先关闭它们，再启动OpenShift Origin容器。

##### 安装、启动一个All-in-One服务器
1. 通过容器加载服务器

    ```
    $ sudo docker run -d --name "origin" \
        --privileged --pid=host --net=host \
        -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys -v /sys/fs/cgroup:/sys/fs/cgroup:rw \
        -v /var/lib/docker:/var/lib/docker:rw \
        -v /var/lib/origin/openshift.local.volumes:/var/lib/origin/openshift.local.volumes:rslave \ 
        openshift/origin start
    ```
    
    1. `rslave`只能运行在Docker版本大于1.10并且是Red Hat的发行版。
    2. 如果要使用OpenShift Origin[聚合日志记录(aggregated logging)](https://docs.openshift.org/latest/install_config/aggregate_logging.html#install-config-aggregate-logging)，则必须将`-v /var/log:/var/log`添加到docker命令行。原始容器必须能够访问宿主机的 `var/log/containers/`和`/var/log/messages`。

    这个命令会：
    
    * 启动OpenShfit Origin 监听主机上的所有接口(`0.0.0.0:8443`)
    * 启动web console监听`/console`的所有接口(`0.0.0.0:8443`)
    * 加载一个`etcd`服务器来存储持久化数据
    * 加载Kubernetes系统组件

2. 容器启动后，你可以在容器里开启一个控制台：

    ```
    $ sudo docker exec -it origin bash
    ```

如果你删除了容器，所有的配置和存好的应用定义都会被移除。

##### 接下来？
现在你已经成功的运行OpenShift Origin在你的环境中，从一个示例应用的生命循环(lifecycle)[试试](https://docs.openshift.org/latest/getting_started/administrators.html#try-it-out)吧。

### 方法2: 下载二进制包
Red Hat周期性的发布二进制包到GitHub，你可以到OpenShift Origin 仓库的[发布页](https://github.com/openshift/origin/releases)下载。有Linux、Windows和Mac OS X 64位的二进制包；注意Mac和Windows版本只有CLI（命令行界面）。

Linux和Mac OS X的发行档案包含服务器二进制`opensshift`，它是一个一体化(all-in-one)的OpenShift Origin安装。 所有平台的存档包括CLI（`oc`命令）和Kubernetes客户端（`kubectl`命令）。

##### 安装运行一体化服务器
1. 从发布页下载二进制包，解压到你的本地系统。
2. 添加你解压的目录到path中：

    ```
    $ export PATH="$(pwd)":$PATH
    ```
3. 加载服务器

    ```
    $ sudo ./openshift start
    ```
    
    这个命令会：
    * 启动OpenShfit Origin 监听主机上的所有接口(`0.0.0.0:8443`)
    * 启动web console监听`/console`的所有接口(`0.0.0.0:8443`)
    * 加载一个`etcd`服务器来存储持久化数据
    * 加载Kubernetes系统组件

    服务器在前台运行直到你终止进程。
    * 由于需要更改`iptables`和挂载卷，这个命令需要`root`权限
    