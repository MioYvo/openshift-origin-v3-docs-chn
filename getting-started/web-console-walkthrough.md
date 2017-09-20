# 用Web Console创建并构建一个镜像(Image)
> 原文地址 https://docs.openshift.org/latest/getting_started/developers_console.html

* <a href="#overview">概况</a>
    * <a href="#browser-requirements">浏览器要求</a>
* <a href="#before-you-begin">开始之前</a>
* <a href="#forking">Fork示例仓库</a>
* <a href="#create-a-project">创建一个项目(Project)</a>
* <a href="#create-an-application">创建一个应用(Application)</a>
* <a href="#verify-running">确认应用运行</a>
* <a href="#configuring-auto-builds">配置自动构建</a>
* <a href="#write-a-change">更改一次代码</a>
    * <a href="#manually-rebuild">手动重新构建镜像</a>

----

### <a name="overview">概况</a>
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

#### <a name="browser-requirements">浏览器要求</a>
|浏览器（最新稳定版）|操作系统|
|:---|:---|
|Firefox|Fedora 23, Windows 8|
|Internet Explorer|Windows 8|
|Chrome|Fedora 23, Windows 8, and MacOSX|
|Safari|MacOSX, iPad 2, iPhone 4|

### <a name="before-you-begin">开始之前</a>
在开始前：

* 你必须可以进入一个OpenShift Origin实例。否则请联系你的集群管理员。
* 你的实例必须被集群管理员用[即时应用模板(Instant App templates)](https://docs.openshift.org/latest/dev_guide/templates.html#using-the-instantapp-templates)和[构建器镜像(builder images)](https://docs.openshift.org/latest/using_images/s2i_images/index.html#using-images-s2i-images-index)预设过。
* 你必须[下载并安装](https://docs.openshift.org/latest/cli_reference/get_started_cli.html#cli-reference-get-started-cli)好了OpenShift Origin CLI。

### <a name="forking">Forking 示例仓库</a>
1. 登录进入Github，访问[Ruby示例](https://github.com/openshift/ruby-ex)页面。
2. Fork该仓库。
   你的浏览器会被重定向到你的fork仓库中。
3. 复制你fork的仓库的 clone URL。
4. clone到你的本地机器。

### <a name="create-a-project">创建一个项目(project)</a>
要创建一个应用(application)，你比选先创建一个新的项目(project)，然后选择一个即时应用模板(InstantApp template)。从这里开始，OpenShift Origin开始构建程序并创建一个新的部署(deployment)。

1. 用浏览器访问OpenShift Origin web console。web console用一个自签名的证书，所以如果有提示，选择继续，跳过浏览器的警告（译者注：建议使用Chrome，Safari不可跳过警告）。
2. 提示需要输入登录信息，输入你的管理员建议的用户名和密码。
3. 点击**New Project**来创建项目。
4. 输入新项目的唯一的名字(name)、显示名(display name)和描述。
5. 点击**Create**。浏览器会加载web console的欢迎页。


### <a name="create-an-application"> 创建一个应用(application)</a>

选择镜像或模板(Select Image or Template)页面，给你机会选择从公开访问的git仓库还是从一个模板创建应用。

1. 如果创建新项目没有自动重定向你到选择镜像或模板(Select Image or Template)页面，你需要点击**Add to Project**。
2. 点击**Browse**，然后从下来列表中选择**ruby**。
3. 点击**ruby:latest**构建器镜像(builder image)。
4. 为你的应用输入一个**名字(name)**，然后指定**Git仓库URL(Git Repository URL)**为`https://github.com/<your_github_username>/ruby-ex.git`。
5. 可选，点击**显示路由、构建和部署的高级选项(Show advanced routing, build, and deployment options)**，尽管默认情况下这个示例应用会自动创建一个路由(route)、网络钩子触发器(webhook trigger)和构建改变触发器(build change triggers)。
6. 点击**Create**。

创建之后，一些设置可从web console更改，点击**Browse**、**Builds**，选择你的构建，然后点击**Actions**，然后点击**Edit**或**Edit YAML**。

创建过程可能会需要一些时间。接下来你可以去web console的Overview页面，查看新的资源被创建，build和deployment的进度。

当Ruby pod被创建时，它的状态是pending。之后Ruby pod启动并显示新分配的IP地址。当Ruby pod 状态是running时，构建(build)完成。

### <a name="verify-running">确认应用运行</a>
如果你的DNS设置正确，你的新应用就可以用浏览器访问了。如果你无法访问你的应用，去问问系统管理员。

查看你的新应用：

1. 在web console中，通过overview页面来看应用的web地址。比如在**SERVICE: RUBY-EX**下面一行你应该能看到这样的东西：`ruby-ex-my-test.example.openshiftapps.com`
2. 访问这个web地址。

### <a name="configuring-auto-builds">配置自动构建</a>

你为这个应用fork的源码来自[OpenShift Origin GitHub repository](https://github.com/openshift/ruby-ex)。因此，你可以用一个网络钩子(webhook)来自动触发当你更改了代码就重新构建你的应用。

设置webhook：

1. 在Web Console上，导航到包含你应用的项目(project)中。
2. 点击**Browse**，然后点击**Builds**。
3. 点击你的构建的名字，然后点击**Configuration**。
4. 复制**GitHub webhook URL**后面的URL。
5. 登录Github导航到你自己的fork仓库。
6. 点击**Webhooks & Services**。
7. 粘贴webhook URL到**Payload Url**。
8. 点击**Add webhook**保存。

GitHub现在会尝试发送一个ping 负载(payload)到你的OpenShift Origin服务器来保证通讯成功。如果你看到绿色的检查标志显示在你的webhook地址后面，说明它设置好了。把鼠标放在检查标志上查看上一次发送的状态。

下一次你push代码更改到你自己的fork仓库，你的应用会自动重新构建。

### <a name="write-a-change">更改一次代码</a>
本地工作，推送更改到你的应用：

1. 在你的本地机器上，用文本编辑器改变示例应用的源码文件`ruby-ex/config.ru`
2. 做一个代码更改，可以在你的应用中以视觉查看的。比如229行，把标题从`Welcome to your Ruby application on OpenShift `改成`This is my Awesome OpenShift Application`，然后保存。
3. 在git上提交更改，并推送到你的fork上。如果你的webhook正确配置了，你的应用会立刻应用你的更改重新构建自己。一旦重新构建成功了，用之前创建好的路由查看被更新过的应用。

现在继续吧，你需要做的就是push代码更新，OpenShift Origin会处理剩下所有的事。

### <a name="manually-rebuild">手动重新构建镜像</a>
你也许发现如果webhook失效了，或者构建失败了但你想重启构建之后再改代码，这时手动重构建很有用。找到fork仓库里上一次的提交，然后开始手动构建：

1. 点击**Browse**，然后点击**Builds**。
2. 找到你的构建，然后点击**Start Build**。
