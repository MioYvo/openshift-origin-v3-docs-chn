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

#### 方法1: 运行在容器中
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

#### 方法2: 下载二进制包
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
    * 由于需要更改`iptables`和挂载卷(volumes)，这个命令需要`root`权限

4. TLS保护OpenShift Origin服务的安全。这一步，我们在启动时生成一个自签名(self-signed)证书，你的web浏览器或客户端需要接受它。您必须将`oc`和`curl`指向适当的CA捆绑包(bundle)、客户端密钥和证书以连接到OpenShift Origin。 设置以下环境变量：

    ```
    $ export KUBECONFIG="$(pwd)"/openshift.local.config/master/admin.kubeconfig
$ export CURL_CA_BUNDLE="$(pwd)"/openshift.local.config/master/ca.crt
$ sudo chmod +r "$(pwd)"/openshift.local.config/master/admin.kubeconfig
    ```
    
    * 这只作为示例；在生产环境中，开发着会生成自己的密钥(keys)并且不会访问系统的密钥。

##### 接下来？
现在你已经成功的运行OpenShift Origin在你的环境中，从一个示例应用的生命循环(lifecycle)[试试](https://docs.openshift.org/latest/getting_started/administrators.html#try-it-out)吧。

#### 方法3: 从源码构建(build)
你可以在本地从源码构建OpenShift Origin，或者使用[Vagrant](https://www.vagrantup.com/)。查看OpenShift Origin在GitHub上的仓库里的[说明](https://github.com/openshift/origin/blob/master/CONTRIBUTING.adoc#develop-on-virtual-machine-using-vagrant)。

### 试一哈
启动一个OpenShift Origin实例后，你可以试着创建一个端到端的应用，来体验完整的OpenShift Origin概念链。

> 当OpenShift Origin运行在虚拟机(VM)中时，您需要确保主机系统可以访问容器内的端口8080和8443，以获取下面的示例。

1. 用普通用户登录进入服务器：

    ```
    $ oc login
    Username: test
    Password: test
    
    ```
2. 创建一个新项目来保存你的应用：

    ```
    $ oc new-project test
    ```
    
3. 创建一个新应用，基于Docker Hub上的一个Node.js镜像：

    ```
    $ oc new-app openshift/deployment-example
    ```
    
    注意到一个服务(service)被创建，被赋予了一个IP - 这个地址可被用来在集群内部访问应用。
    
4. 显示你创建的资源的摘要：

    ```
    $ oc status
    ```
    
5. 你的应用的容器的镜像会被拉取到本地系统并被启动。一旦它启动，即可在宿主机(host)上访问。如果这是你的笔记本或台式机，打开web浏览器到应用的的IP和端口(port)：

    ```
    http://172.30.192.169:8080 (example)
    ```
    
    如果你在一个分布式的系统中，而且没有可直接访问宿主机的网络，SSH登录到系统中然后执行`curl`命令：
    ```
    $ curl http://172.30.192.169:8080 # (example)
    ```
    
    你会看到`v1`文本显示在页面上。
    
现在你的应用已被部署，你可以通过标记`v2`镜像来触发将该镜像的新版本推送到主机。`new-app`命令创建了一个镜像流(image stream)，他会跟踪(track)你想要使用的镜像。使用`tag`命令来标记一个新的用来部署的镜像：

```
$ oc tag --source=docker openshift/deployment-example:v2 deployment-example:latest
```

应用的部署配置会监视`deployment-example:lastest`，并且当`lastest`tag被更新到`v2`会触发新的滚动部署(rolling deployment)。

* 你也可以用该命令的备用版本：

    ```
    $ oc tag docker.io/openshift/deployment-example:v2 deployment-example:latest
    ```
    
再回到浏览器或者`curl`，你可以看到`v2`文本显示在页面上。

作为一个开发者，构建一个新容器的镜像与部署它们同样重要。OpenShift Origin提供了运行构建的工具，以及通过源到图像工具链，从预定义的构建器图像中构建源代码。

对于此过程，请保证Docker可以从宿主机(host)系统拉取镜像。此外，请确保已完成有关从[主机准备](https://docs.openshift.org/latest/install_config/install/host_preparation.html#install-config-install-host-preparation)设置`--insecure-registry`标志的说明。

1. 切换到管理员用户，切换到`default`项目：

    ```
    $ oc login -u system:admin
    $ oc project default
    ```
2. 为OpenShift Origin集群设置一个集成的Docker注册表(registry)：

    ```
    $ oadm registry
    ```
    这步会化费一些时间来下载注册表的镜像并启动。使用`oc status`来看哪个注册表被启动了。
3. 切换到`test`用户和`test`项目：

    ```
    $ oc login -u test
    $ oc project test
    ```
4. 为了创建一个新的可部署的Node.js镜像，现在创建一个应用，包含有示例代码的Node.js构建器镜像(builder image)：

    ```
    $ oc new-app openshift/nodejs-010-centos7~https://github.com/openshift/nodejs-ex.git
    ```
    
    一个构建过程会被自动触发，使用提供的镜像和仓库的`master`分支的最近提交。查看构建的状态：
    
    ```
    $ oc status
    ```
    这个命令会总结构建过程。当构建完成，结果镜像会被推送的Docker注册表(registry)。
5. 等待部署的镜像启动，然后用浏览器或`curl`查看service的IP。

你可以查看更多的关于[CLI](https://docs.openshift.org/latest/cli_reference/basic_cli_operations.html#cli-reference-basic-cli-operations)(`oc`命令)的可用命令：

```
$ oc help
```

或者连接到其它系统：

```
$ oc -h <server_hostname_or_IP> [...]
```

OpenShift Origin包含了一个web控制台(web console)，它会帮助你可视化你的应用、做常见的创建和管理。你可以通过`https://<server>:8443/console`，使用我们上面创建的`test`用户来登录进入控制台。更多信息，查看[Getting Started for Developers: Web Console](https://docs.openshift.org/latest/getting_started/developers_console.html#getting-started-developers-console)

你也可以查看[OpenShift Origin 3 Application Lifecycle Sample](https://github.com/openshift/origin/blob/master/examples/sample-app)来获得更多更深的演练。

