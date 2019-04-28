# Bookinfo 示例

[Bookinfo 示例](https://istio.io/zh/docs/examples/bookinfo/)是 Istio 官方为了演示 Istio 功能而开发的一个示例应用，该应用有如下特点：

- 使用微服务方式开发，共有四个微服务
- 多语言应用，使用了 Java、Python、Ruby 和 NodeJs 语言
- 为了演示流量管理的高级功能，有的服务同时推出了多个版本

## Bookinfo 应用部署架构

以下为 Istio 官方提供的该应用的架构图。

![Istio 的 Bookinfo 示例应用架构图](../images/006tNbRwgy1fvlwjd3302j31bo0ro0x5.jpg)

Bookinfo 应用分为四个单独的微服务，其中每个微服务的部署的结构中都注入了一个 Sidecar：

- `productpage` ：`productpage` 微服务会调用 `details` 和 `reviews` 两个微服务，用来生成页面。
- `details` ：这个微服务包含了书籍的信息。
- `reviews` ：这个微服务包含了书籍相关的评论。它还会调用 `ratings` 微服务。
- `ratings` ：`ratings` 微服务中包含了由书籍评价组成的评级信息。

`reviews` 微服务有 3 个版本：

- v1 版本不会调用 `ratings` 服务。
- v2 版本会调用 `ratings` 服务，并使用 1 到 5 个黑色星形图标来显示评分信息。
- v3 版本会调用 `ratings` 服务，并使用 1 到 5 个红色星形图标来显示评分信息。

使用 [kubernetes-vagrant-centos-cluster](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster) 部署的 Kubernetes 集群和 Istio 服务的话可以直接运行下面的命令部署 Bookinfo 示例：

```bash
$ kubectl apply -n default -f <(istioctl kube-inject -f yaml/istio-bookinfo/bookinfo.yaml)
$ istioctl create -n default -f yaml/istio-bookinfo/bookinfo-gateway.yaml
```

关于该示例的介绍和详细步骤请参考 [Bookinfo 应用](https://istio.io/zh/docs/examples/bookinfo/)。

## Bookinfo 示例及 Istio 服务整体架构

从 Bookinfo 应用部署架构中可以看到该应用的几个微服务之间的关系，但是并没有描绘应用与 Istio 控制平面、Kubernetes 平台的关系，下图中描绘的是应用和平台整体的架构。

![Bookinfo 示例与 Istio 的整体架构图](../images/bookinfo-application-traffic-route-and-connections-within-istio-service-mesh.png)

从图中可以看出 Istio 整体架构的特点：

- 模块化：很多模块可以选择性的开启，如负责证书管理的 `istio-citadel` 默认就没有启用
- 可定制化：可观察性的组件可以定制化和替换

## 参考

- [Bookinfo 应用 - istio.io](https://istio.io/zh/docs/examples/bookinfo/)

