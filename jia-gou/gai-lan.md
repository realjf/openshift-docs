# 概览

OpenShift v3是一个分层系统，旨在尽可能准确地公开底层Docker格式的容器图像和Kubernetes概念，重点是开发人员轻松组合应用程序。例如，安装Ruby，推送代码和添加MySQL。

与OpenShift v2不同，在模型的所有方面创建后，会暴露更多的配置灵活性。删除了作为单独对象的应用程序的概念，以支持更灵活的“服务”组合，允许两个Web容器重用数据库或将数据库直接暴露给网络边缘。

## 什么是layers？
Docker服务提供了打包和创建基于Linux的轻量级容器映像的抽象 。Kubernetes提供 集群管理并在多个主机上编排容器。

OKD补充：
- 开发人员的源代码管理，构建和部署
- 在流经系统时大规模管理和推广镜像
- 可伸缩应用管理
- 用于组织大型开发人员组织的团队和用户跟踪
- 支持集群的网络基础架构

![okd-architecture-overview.png](../images/okd-architecture-overview.png)

有关体系结构概述中节点类型的更多信息，请参阅[Kubernetes基础结构](./ji-chu-she-shi-zu-jian/kubernetes-ji-chu-she-shi.md)。

## 什么是OKD架构？
OKD具有基于微服务的架构，可以协同工作的小型分离单元。它运行在Kubernetes集群之上 ，其中包含有关存储在etcd中的对象的数据， 这是一个可靠的集群键值存储。这些服务按功能细分：
- REST API，它公开每个[核心对象](./he-xin-gai-nian/gai-shu.md)
- 读取这些API的控制器将更改应用于其他对象，并报告状态或写回对象。

用户调用REST API以更改系统状态。控制器使用REST API读取用户所需的状态，然后尝试使系统的其他部分同步。例如，当用户请求 构建时，他们创建“构建”对象。构建控制器看到已创建新构建，并在集群上运行进程以执行该构建。构建完成后，控制器通过REST API更新构建对象，用户可以看到构建完成。

控制器模式意味着OKD中的许多功能都是可扩展的。运行和启动构建的方式可以独立于图像的管理方式或部署方式进行自定义 。控制器正在执行系统的“业务逻辑”，采取用户行动并将其转化为现实。通过自定义这些控制器或用您自己的逻辑替换它们，可以实现不同的行为。从系统管理的角度来看，这也意味着API可用于在重复计划上编写常见管理操作的脚本。这些脚本也是监视变化并采取行动的控制器。OKD能够以这种方式自定义集群，成为一流的行为。

为了实现这一点，控制器利用可靠的系统变化流来同步他们对系统的看法与用户正在做的事情。此事件流将更改从etcd推送到REST API，然后在更改发生时立即推送到控制器，因此更改可以非常快速有效地通过系统传播。但是，由于故障可能随时发生，控制器还必须能够在启动时获得系统的最新状态，并确认一切都处于正确状态。这种重新同步很重要，因为这意味着即使出现问题，操作员也可以重新启动受影响的组件，系统会在继续之前对所有内容进行双重检查。系统最终应该收敛到用户的意图，因为控制器总是可以使系统同步。

## OKD如何获得担保？
OKD和Kubernetes API 对呈现凭据的用户进行身份验证，然后 根据其角色对其进行授权。开发人员和管理员都可以通过多种方式进行身份验证，主要是 OAuth令牌和X.509客户端证书。OAuth令牌使用JSON Web算法 RS256进行签名，该算法是带有SHA-256的RSA签名算法PKCS＃1 v1.5。

开发商（该系统的客户端）通常使来自REST API调用 客户端程序像oc或到 Web控制台通过他们的浏览器，并使用OAuth承载令牌大多数通信。基础结构组件（如节点）使用由系统生成的包含其标识的客户端证书。在容器中运行的基础架构组件使用与其服务帐户关联的令牌 来连接到API。

授权在OKD策略引擎中处理，该引擎定义诸如“创建窗格”或“列表服务”之类的操作，并将它们分组为策略文档中的角色。角色通过用户或组标识符绑定到用户或组。当用户或服务帐户尝试操作时，策略引擎在允许其继续之前检查分配给用户的一个或多个角色（例如，群集管理员或当前项目的管理员）。

由于在群集上运行的每个容器都与服务帐户相关联，因此还可以将机密关联 到这些服务帐户，并将它们自动传送到容器中。这使基础架构能够管理用于提取和推送映像，构建和部署组件的秘密，并且还允许应用程序代码轻松利用这些秘密。


## TLS支持
使用REST API以及主要组件（如etcd和API服务器）之间的所有通信通道 都使用TLS进行保护。TLS通过X.509服务器证书和公钥基础结构提供强大的加密，数据完整性和服务器身份验证。默认情况下，为每个OKD部署创建一个新的内部PKI。内部PKI使用2048位RSA密钥和SHA-256签名。 还支持公共主机的自定义证书。

OKD使用Golang的crypto / tls标准库实现， 不依赖于任何外部加密和TLS库。此外，客户端依赖外部库进行GSSAPI身份验证和OpenPGP签名。GSSAPI通常由MIT Kerberos或Heimdal Kerberos提供，它们都使用OpenSSL的libcrypto。OpenPGP签名验证由libgpgme和GnuPG处理。

不安全的SSL 2.0和SSL 3.0版本不受支持且不可用。OKD服务器和oc客户端默认只提供TLS 1.2。可以在服务器配置中启用TLS 1.0和TLS 1.1。服务器和客户端都更喜欢具有经过验证的加密算法和完美前向保密的现代密码套件。使用已弃用且不安全的算法（如RC4,3DES和MD5）的密码套件将被禁用。某些内部客户端（例如，LDAP身份验证）具有较少的限制设置，其中TLS 1.0到1.2以及启用了更多密码套件。


**OKD服务器和oc 客户端的启用密码套件列表按优先顺序排序：**
- TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
- TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_GCM_SHA384
- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_CBC_SHA
