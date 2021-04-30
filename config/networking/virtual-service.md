# VirtualService

`VirtualService` 主要配置流量路由。以下是在流量路由背景下定义的几个有用的术语。

`Service` 是与服务注册表（service registry）中的唯一名称绑定的应用行为单元。服务由多个网络 * 端点（endpoint）* 组成，这些端点由运行在 pod、容器、虚拟机等的工作负载实例实现。

服务版本，又称子集（subset）：在持续部署方案中，对于一个给定的服务，可能有不同的实例子集，运行应用程序二进制的不同变体。这些变体不一定是不同的 API 版本。它们可能是同一服务的迭代变化，部署在不同的环境（prod、staging、dev 等）。发生这种情况的常见场景包括 A/B 测试、金丝雀发布等。一个特定版本的选择可以根据各种标准（header、URL 等）和 / 或分配给每个版本的权重来决定。每个服务都有一个由其所有实例组成的默认版本。

源（source）：下游客户端调用服务。

Host：客户端在尝试连接到服务时使用的地址。

访问模型（access model）：应用程序只针对目标服务（Host），而不了解各个服务版本（子集）。版本的实际选择是由代理 /sidecar 决定的，使应用程序代码能够从依赖服务的演变中解脱出来。

`VirtualService` 定义了一套当主机被寻址时应用的流量路由规则。每个路由规则定义了特定协议流量的匹配标准。如果流量被匹配，那么它将被发送到注册表中定义的指定目标服务（或它的子集 / 版本）。

流量的来源也可以在路由规则中进行匹配。这允许为特定的客户环境定制路由。

## 示例

以下是 Kubernetes 上的例子，默认情况下，所有的 HTTP 流量都会被路由到标签为 `version: v1` 的 reviews 服务的 pod 上。此外，路径以 `/wpcatalog/` 或 `/consumercatalog/` 开头的 HTTP 请求将被重写为 `/newcatalog`，并被发送到标签为 `version: v2` 的 pod 上。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  http:
  - name: "reviews-v2-routes"
    match:
    - uri:
        prefix: "/wpcatalog"
    - uri:
        prefix: "/consumercatalog"
    rewrite:
      uri: "/newcatalog"
    route:
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v2
  - name: "reviews-v1-route"
    route:
    - destination:
        host: reviews.prod.svc.cluster.local
        subset: v1
```

途径目的地的一个子集/版本是通过对一个命名的服务子集的引用来识别的，这个子集必须在一个相应的 `DestinationRule` 中声明。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews.prod.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

关于 VirtualService 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/networking/virtual-service/)。

## 参考

- [Virtual Service - istio.io](https://istio.io/latest/docs/reference/config/networking/virtual-service/)