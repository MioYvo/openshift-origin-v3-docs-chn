# 项目和用户 Projects and Users
* Users
* Namespces
* Projects

---

### Users
用户用来与OpenShift Origin交互。OpenShift Origin用户对象表示可以通过[向其或其组添加角色](https://docs.openshift.org/latest/admin_guide/manage_authorization_policy.html#managing-role-bindings)来授予系统权限的人员。

用户的类型：

|类型|解释|
|:--|:--|
|普通用户 Regular users|这是OpenShift Origin用户最有代表性的方式。普通用户在系统启动第一次登录时自动被创建，或者通过API被创建。普通用户用`User`对象表示。比如：`joe` `alice`|
|系统用户 System users|它们大多是在基础设施被定义时自动创建，主要是为了让基础设施与API安全地交互。它们包括集群管理员(可访问任何事物)、节点用户(per-node user)、路由器和注册表所用的用户等等。还有个`anonymous`系统用户，默认用作未认证请求。比如：`system:admin` `system:openshift-registry` `system:node:node1.example.com`|
|服务账户 Service accounts|有一些与项目相关的特殊系统用户；一些是在项目第一次创建时自动创建，这样项目管理员可以创建更多，以便定义对每个项目的内容的访问。服务帐户用`ServiceAccount`对象表示。比如：`system:serviceaccount:default:deployer` `system:serviceaccount:foo:builder`|

每个用户必须用一些方法通过认证，来访问OpenShift Origin。未认证或无效的API请求被当做`anonymous`系统用户的请求。一旦通过身份验证，策略将决定用户[被授权](https://docs.openshift.org/latest/architecture/additional_concepts/authorization.html#architecture-additional-concepts-authorization)做什么。

### 名字空间 Namespaces
Kubernetes名字空间提供了一种整理集群资源的机制。在OpenShift Origin中，一个项目是一个Kubernetes名字空间加上额外的注解。

名字空间提供了一个唯一范围：
* 命名资源以避免基本的命名冲突。
* 向受信任的用户授予管理权限。
* 限制社区资源消耗的能力。

系统中的大多数对象都受到命名空间的限制，但有些则被排除，并且没有命名空间，包括节点和用户。

[Kubernetes文档](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/admin/namespaces.md)有更多名字空间的信息。

### 项目 Project
项目是加上额外注解的Kubernetes名字空间，是管理普通用户资源的中心工具。项目允许社区的用户组织和管理它们的内容，与其它社区隔离。用户必须被管理员授权访问项目，或者如果允许创建项目，自动有它们自己项目的权限。

项目有分离的`name`、`displayName`和`description` （名字、显示名、描述）。

* 强制性的`name`是项目的唯一标识符，使用CLI工具或API时最为明显。名字最大长度是63个字符。
* 可选的`displayName`是web控制台上项目的显示方式（默认为`name`的值）
* 可选的`description`是项目更详细的描述，也可以在web控制台上看到。

每个项目都有自己的一套：

|||
|:--|:--|
|对象 Objects|Pods, services（服务）,replication controllers（副本控制器）等等。|
|策略 Policies|用户能或不能操作对象的规则|
|限制 Constraints|限制每种对象的配额。|
|服务账户 Service accounts|服务帐户会自动对项目中对象的指定访问进行操作。|

集群管理员可以[创建项目](https://docs.openshift.org/latest/dev_guide/projects.html#dev-guide-projects)和[委托项目的管理员权限](https://docs.openshift.org/latest/admin_guide/manage_authorization_policy.html#managing-role-bindings)给任意用户社区的成员。集群管理员也可以允许开发者创建[他们自己的项目](https://docs.openshift.org/latest/admin_guide/managing_projects.html#selfprovisioning-projects)。

开发者和管理员可以用CLI或web控制台与项目进行交互。