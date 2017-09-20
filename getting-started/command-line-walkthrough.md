# Create and Build an Image Using the CLI 用命令行界面创建并构建镜像

* Overview
* Before You Begin
* Forking the Sample Repository
* Creating a Project
* Creating an Application
* Create a Route
* Verify the Application is Running
* Configuring Automated Builds
* Writing a Code Change
    * Manually Rebuilding Images
* Troubleshooting

----

### 概况
这个入门教程让你一步步用最简单的方法，在OpenShift Origin上启动一个示例项目。有为数不多的几种方法在项目(project)中加载镜像(images)，但在这个话题中，我们关注的是最快和最简单的方法。

如果这是你看的第一部分文档，你对OpenShift Origin第三版(v3)的核心概念不熟悉的话，可以看[这篇(what's new)](https://docs.openshift.org/latest/whats_new/index.html#whats-new-index)再开始。这个版本的OpenShift Origin与第二版(v2)有明显的不同。

OpenShift Origin 3为开发人员提供了一套[语言](https://docs.openshift.org/latest/using_images/s2i_images/index.html#using-images-s2i-images-index)和[数据库](https://docs.openshift.org/latest/using_images/db_images/index.html#using-images-db-images-index)，有相应的实现和教程，可让您启动应用程序开发。[语言支持中心](https://docs.openshift.org/latest/dev_guide/app_tutorials/quickstarts.html#dev-guide-app-tutorials-quickstarts)和[镜像构建器](https://docs.openshift.org/latest/using_images/s2i_images/index.html#using-images-s2i-images-index)相互促进。

|语言|实现和教程|
|:---|:---|
|Ruby|[Rails](https://github.com/openshift/rails-ex)|
|Python|[Django](https://github.com/openshift/django-ex)|
|Node.js|[Node.js](https://github.com/openshift/nodejs-ex)|
|PHP|[CakePHP](https://github.com/openshift/cakephp-ex)|
|Perl|[Dancer](https://github.com/openshift/dancer-ex)|
|Java|[Maven](https://docs.openshift.org/latest/using_images/s2i_images/java.html#using-images-s2i-images-java)|

其他OpenShift Origin提供的镜像：

* [MySQL](https://github.com/openshift/mysql)
* [MongoDB](https://github.com/openshift/mongodb)
* [PostgreSQL](https://github.com/openshift/postgresql)
* [Jenkins](https://github.com/openshift/jenkins)

本主题讨论了[快速入门](https://docs.openshift.org/latest/dev_guide/app_tutorials/quickstarts.html#dev-guide-app-tutorials-quickstarts)和[即时应用程序](https://docs.openshift.org/latest/dev_guide/templates.html#using-the-instantapp-templates)模板和应用程序。 Quickstart为应用程序开发提供了起点，但它们依赖于开发来创建一个有用的应用程序。 相比之下，像Jenkins这样的即时应用即时可用。

### 开始之前
在开始前：

* 你必须可以进入一个OpenShift Origin实例。否则请联系你的集群管理员。
* 你的实例必须被集群管理员用[即时应用模板(Instant App templates)](https://docs.openshift.org/latest/dev_guide/templates.html#using-the-instantapp-templates)和[构建器镜像(builder images)](https://docs.openshift.org/latest/using_images/s2i_images/index.html#using-images-s2i-images-index)预设过。
* 你必须[下载并安装](https://docs.openshift.org/latest/cli_reference/get_started_cli.html#cli-reference-get-started-cli)好了OpenShift Origin CLI。

### Fork示例仓库
1. 登录进入Github，访问[Ruby示例](https://github.com/openshift/ruby-ex)页面。
2. Fork该仓库。
   你的浏览器会被重定向到你的fork仓库中。
3. 复制你fork的仓库的 clone URL。
4. clone到你的本地机器。

### 创建项目

要创建一个应用(application)，你必须先建一个项目(project)然后指定资源的位置。从这里开始，OpenShift Origin 开始构建(build)进程，创建了一个新的部署(deployment)

1. 用CLI(command-line interface)登入OpenShift Origin
    * 用用户名和密码

        ```
        $ oc login -u=<username> -p=<password> --server=<your-openshift-server> --insecure-skip-tls-verify
        ```
    * 用oauth token

        ```
        $ oc login <https://api.your-openshift-server.com> --token=<tokenID>
        ```
        
2. 创建一个新项目

    ```
    $ oc new-project <projectname> --description="<description>" --display-name="<display_name>"
    ```
    
创建好新项目，你会被自动切换到新项目的名字空间(namespace)。

### 创建新应用
用你forked的仓库(ruby示例)的代码创建一个新应用。

1. 通过指定代码源来创建应用

    ```
    $ oc new-app openshift/ruby-20-centos7~https://github.com/<your_github_username>/ruby-ex
    ```
    OpenShift Origin找到对应构建器的镜像（这里就是**ruby-20-centos7**），然后为应用创建资源(image stream镜像流, build configuration构建配置, deployment configuration部署配置, service服务)。它还会调度构建。

2. 追踪构建过程

    ```
    $ oc logs -f bc/ruby-ex
    ```
3. 一旦构建完成了，结果镜像被成功的推送到你的注册处(registry)，检查应用的状态：

    ```
    $ oc status
    ```
    你也可以从web console查看构建。

创建你的应用也许会费些时间。你可以停留在web console的Overview 页面，看着新资源被创建，监视构建和部署的过程。你也可以用命令`oc get pods`检查pod是否启动并运行，或者`oc get builds`命令查看构建的统计信息。

当Ruby pod正在被创建，它的状态显示的是pending。然后Ruby pod启动，显示它新得到的IP地址。当Ruby pod运行起来(running)，构建就完成了。

`oc status`命令告诉你服务(service)正运行在哪个IP地址上；默认部署的端口是8080.

### 创建一个路由(Route)
一个OpenShift Origin路由用主机名(host name)暴露一个服务(service)，所以外部的客户端能用名字接触它。为你的新应用创建一个路由：

1. 暴露一个**ruby-ex**的服务:
    
    ```
    $ oc expose service ruby-ex
    ```
2. 查看你的路由

    ```
    $ oc get route
    ```
3. 复制路由的位置，比如`ruby-ex-my-test.example.openshiftapps.com`

### 确认应用正在运行
要查看你的新应用，把你在上一节拷贝的路由地址，粘贴到浏览器的地址栏中，然后按回车键即可。

示例应用`ruby-ex`是一个简单的欢迎页面，包含了如何部署代码的更改、管理你的应用和一些其他的部署信息。

接下来，通过一个GitHub webhook trigger来配置自动构建，这样的话代码的更改就会触发重新构建。

### 配置自动构建
你刚刚已经从[OpenShift Origin GitHub repository](https://github.com/openshift/ruby-ex)fork了这个应用的源码。所以你可以用一个webhook来自动触发重新构建(rebuild)，当你push代码更改到你的仓库中时。

为你的应用设置webhook：

1. 在`BuildConfig`找到关于触发器(triggers)的部分，来确认GitHub webhook trigger的存在：

    ```
    $ oc edit bd/ruby-ex
    ```
    
    你应该能看到类似这样的东西：
    
    ```
    triggers
    - github:
        secret: Q1tGY0i9f1ZFihQbX07S
        type: GitHub
    ```
    密钥(secret)保证了只有你和你的仓库可以触发构建
    
2. 运行以下命令来显示与`BuildConfig`关联的webhook URL：

    ```
    $ oc describe bc ruby-ex
    ```
    
3. 拷贝上面命令输出的GitHub webhook的URL
4. 在GitHub找到你已fork的仓库，点击`Settings`
5. 点击`Webhooks & Services`
6. 点击`Add webhook`
7. 粘贴你的webhook URL到`Payload URL`一栏。
8. 点击`Add webhook`保存。

GitHub现在会尝试发送一个ping 负载(payload)到你的OpenShift Origin服务器来保证通讯成功。如果你看到绿色的检查标志显示在你的webhook地址后面，说明它设置好了。把鼠标放在检查标志上查看上一次发送的状态。

下一次你push代码更改到你自己的fork仓库，你的应用会自动重新构建。

### 更改一次代码
本地工作，推送更改到你的应用：

1. 在你的本地机器上，用文本编辑器改变示例应用的源码文件`ruby-ex/config.ru`
2. 做一个代码更改，可以在你的应用中以视觉查看的。比如229行，把标题从`Welcome to your Ruby application on OpenShift `改成`This is my Awesome OpenShift Application`，然后保存。
3. 在git上提交更改，并推送到你的fork上。如果你的webhook正确配置了，你的应用会立刻应用你的更改重新构建自己。一旦重新构建成功了，用之前创建好的路由查看被更新过的应用。

现在继续吧，你需要做的就是push代码更新，OpenShift Origin会处理剩下所有的事。

### 手动重新构建(rebuild)镜像
如果你的webhook不好用，或者一个构建失败了你不想在重启构建之前改动代码，你可以用手动方式来重新构建镜像。以你上一次提交的更改为基础，重新构建镜像：

```
$ oc start-build ruby-ex
```

### 查错
##### 更改项目 Changing Projects
尽管`oc new-project`命令自动设置你当前的项目为刚刚创建的，你仍然可以切换项目使用命令：

```
$ oc project <project-name>
```

查看项目列表：

```
$ oc get projects
```

##### 手动触发构建
如果构建没有自动启动，启动构建并流式(stream)输出日志：
```
$ oc start-build ruby-ex --follow
```

或者，不要在上述命令中包含`--follow`，而是在触发构建后发出以下命令，其中`n`是要跟踪的构建编号：

```
$ oc logs -f build/ruby-ex-n
```