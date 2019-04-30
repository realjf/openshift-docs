# kubernetes基础设施
[toc]

## 概览
在OKD中，Kubernetes跨一组容器或主机管理容器化应用程序，并提供部署，维护和应用程序扩展的机制。容器运行时包，实例化和运行容器化应用程序。Kubernetes集群由一个或多个主服务器和一组节点组成。

您可以选择将主服务器配置为 高可用性（HA），以确保群集没有单点故障。

## 大师
主服务器是包含控制平面组件的主机，包括API服务器，控制器管理器服务器和etcd。主服务器管理其Kubernetes集群中的节点，并调度 pod以在这些节点上运行。


| 零件 | 描述 |
| --:| :-- |
| API服务器 | Kubernetes API服务器验证和配置容器，服务和复制控制器的数据。它还将pod分配给节点，并将pod信息与服务配置同步。 |
| ETCD | etcd存储持久主状态，而其他组件监视etcd以进行更改以使其自身进入所需状态。etcd可以选择性地配置为高可用性，通常使用2n + 1个对等服务进行部署。|
| Controller Manager Server | 控制器管理器服务器监视etcd以更改复制控制器对象，然后使用API​​强制执行所需的状态。几个这样的过程一次创建一个具有一个活跃领导者的集群。 |
| HAProxy的 | 可选，在使用方法配置 高可用主服务器时使用，native 以平衡API主端点之间的负载。在群集安装过程 可以为您提供配置HAProxy的native方法。或者，您可以使用该native方法，但预先配置您自己选择的负载均衡器。|

## 控制平面静态窗格
核心控制平面组件，API服务器和控制器管理器组件作为 由kubelet操作的静态pod运行。

对于将etcd共同位于同一主机上的主服务器，etcd也会移动到静态pod。在不是主服务器的etcd主机上仍然支持基于RPM的etcd。

此外，节点组件openshift-sdn和 openvswitch现在使用DaemonSet而不是systemd服务运行。

![control-plane-host-architecture-changes](../../images/control-plane-host-architecture-changes.png)


即使控制平面组件作为静态pod运行，主控主机仍然从/etc/origin/master/master-config.yaml 文件中获取其配置，如 主和节点配置主题中所述。

主节点上的kubelet会自动在API服务器上为每个控制平面静态pod 创建镜像窗格，以便它们在kube-system项目的集群中可见。这些静态pod的清单默认由openshift-ansible安装程序安装，该安装程序位于 主控主机上的/ etc / origin / node / pods目录中。

这些pod具有以下hostPath定义的卷：
- /etc/origin/master Contains all certificates, configuration files, and the admin.kubeconfig file.
- /var/lib/origin Contains volumes and potential core dumps of the binary.
- /etc/origin/cloudprovider Contains cloud provider specific configuration (AWS, Azure, etc.).
- /usr/libexec/kubernetes/kubelet-plugins Contains additional third party volume plug-ins.
- /etc/origin/kubelet-plugins Contains additional third party volume plug-ins for system containers.


## 重启master服务

重启master api
```bash
master-restart api
```
重启controllers
```bash
master-restart controllers
```

重启etcd
```bash
master-restart etcd
```

## 查看master服务日志
```bash
master-logs api api
master-logs controllers controllers
master-logs etcd etcd

```

## master节点高可用









