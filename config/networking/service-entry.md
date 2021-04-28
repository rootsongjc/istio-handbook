# ServiceEntry

`ServiceEntry` 可以在 Istio 的内部服务注册表中添加额外的条目，这样网格中自动发现的服务就可以访问 / 路由到这些手动指定的服务。一个服务条目描述了一个服务的属性（DNS 名称、VIP、端口、协议、端点）。这些服务可以是网格的外部服务（如 Web API），也可以是不属于平台服务注册表的网格内部服务（如与 Kubernetes 中的服务对话的一组虚拟机）。此外，服务条目的端点也可以通过使用 `workloadSelector` 字段动态选择。这些端点可以是使用 `WorkloadEntry` 对象声明的虚拟机工作负载或 Kubernetes pod。在单一服务下同时选择 pod 和 VM 的能力允许将服务从 VM 迁移到 Kubernetes，而不必改变与服务相关的现有 DNS 名称。

## 示例

下面的例子声明了一些内部应用程序通过 HTTPS 访问的外部 API。Sidecar 检查了 ClientHello 消息中的 SNI 值，以路由到适当的外部服务。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-https
spec:
  hosts:
  - api.dropboxapi.com
  - www.googleapis.com
  - api.facebook.com
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: TLS
  resolution: DNS
```

下面的配置在 Istio 的注册表中添加了一组运行在未被管理的虚拟机上的 MongoDB 实例，因此这些服务也可以被视为网格中的任何其他服务。相关的 DestinationRule 被用来启动与数据库实例的 mTLS 连接。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-mongocluster
spec:
  hosts:
  - mymongodb.somedomain # 未使用
  addresses:
  - 192.192.192.192/24 # VIP
  ports:
  - number: 27018
    name: mongodb
    protocol: MONGO
  location: MESH_INTERNAL
  resolution: STATIC
  endpoints:
  - address: 2.2.2.2
  - address: 3.3.3.3
```

相关的 DestinationRule。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: mtls-mongocluster
spec:
  host: mymongodb.somedomain
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```

下面的例子在一个虚拟服务中使用服务条目和 TLS 路由的组合，根据 SNI 值将流量引导到内部出口（egress）防火墙。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-redirect
spec:
  hosts:
  - wikipedia.org
  - "*.wikipedia.org"
  location: MESH_EXTERNAL
  ports:
  - number: 443
    name: https
    protocol: TLS
  resolution: NONE
```

相关的 VirtualService，以根据 SNI 值进行路由。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tls-routing
spec:
  hosts:
  - wikipedia.org
  - "*.wikipedia.org"
  tls:
  - match:
    - sniHosts:
      - wikipedia.org
      - "*.wikipedia.org"
    route:
    - destination:
        host: internal-egress-firewall.ns1.svc.cluster.local
```

 带有 TLS 匹配的虚拟服务是为了覆盖默认的 SNI 匹配。在没有虚拟服务的情况下，流量将被转发到维基百科的域。

下面的例子演示了专用出口（egress）网关的使用，所有外部服务流量都通过该网关转发。`exportTo` 字段允许控制服务声明对网格中其他命名空间的可见性。默认情况下，服务会被输出到所有命名空间。下面的例子限制了对当前命名空间的可见性，用 `.` 表示，所以它不能被其他命名空间使用。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-httpbin
  namespace : egress
spec:
  hosts:
  - httpbin.com
  exportTo:
  - "."
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
```

定义一个网关来处理所有的出口（egress）流量。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
 name: istio-egressgateway
 namespace: istio-system
spec:
 selector:
   istio: egressgateway
 servers:
 - port:
     number: 80
     name: http
     protocol: HTTP
   hosts:
   - "*"
```

和相关的 VirtualService，从 Sidecar 路由到网关服务（`istio-egressgateway.istio-system.svc.cluster.local`），以及从网关路由到外部服务。请注意，虚拟服务被导出到所有命名空间，使它们能够通过网关将流量路由到外部服务。迫使流量通过像这样一个受管理的中间代理是一种常见的做法。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: gateway-routing
  namespace: egress
spec:
  hosts:
  - httpbin.com
  exportTo:
  - "*"
  gateways:
  - mesh
  - istio-egressgateway
  http:
  - match:
    - port: 80
      gateways:
      - mesh
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
  - match:
    - port: 80
      gateways:
      - istio-egressgateway
    route:
    - destination:
        host: httpbin.com
```

下面的例子演示了在外部服务的主机中使用通配符。如果连接必须被路由到应用程序请求的 IP 地址（即应用程序解析 DNS 并试图连接到一个特定的 IP），发现模式必须被设置为`NONE`。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-wildcard-example
spec:
  hosts:
  - "*.bar.com"
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: NONE
```

下面的例子演示了一个通过客户主机上的 Unix 域套接字提供的服务。解析必须设置为`STATIC`以使用 Unix 地址端点。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: unix-domain-socket-example
spec:
  hosts:
  - "example.unix.local"
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: STATIC
  endpoints:
  - address: unix:///var/run/example/socket
```

对于基于 HTTP 的服务，可以创建一个由多个 DNS 可寻址端点支持的虚拟服务。在这种情况下，应用程序可以使用`HTTP_PROXY`环境变量来透明地将 VirtualService 的 API 调用重新路由到所选择的后端。例如，下面的配置创建了一个不存在的外部服务，名为`foo.bar.com`，由三个域名支持：`us.foo.bar.com:8080`，`uk.foo.bar.com:9080`和`in.foo.bar.com:7080`。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-dns
spec:
  hosts:
  - foo.bar.com
  location: MESH_EXTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  endpoints:
  - address: us.foo.bar.com
    ports:
      http: 8080
  - address: uk.foo.bar.com
    ports:
      http: 9080
  - address: in.foo.bar.com
    ports:
      http: 7080
```

有了`HTTP_PROXY=http://localhost/`，从应用程序到`http://foo.bar.com`的调用将在上面指定的三个域中进行负均衡。换句话说，对`http://foo.bar.com/baz`的调用将被转译成`http://uk.foo.bar.com/baz`。

下面的例子说明了包含主题（subject）替代名称的 `ServiceEntry` 的用法，其格式符合 [SPIFFE 标准](https://github.com/spiffe/spiffe/blob/master/standards/SPIFFE-ID.md)。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin
  namespace : httpbin-ns
spec:
  hosts:
  - httpbin.com
  location: MESH_INTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: STATIC
  endpoints:
  - address: 2.2.2.2
  - address: 3.3.3.3
  subjectAltNames:
  - "spiffe://cluster.local/ns/httpbin-ns/sa/httpbin-service-account"
```

下面的例子演示了使用带有`workloadSelector`的`ServiceEntry`来处理服务`details.bookinfo.com`从 VM 到 Kubernetes 的迁移。该服务有两个基于虚拟机的实例，带有 sidecar，以及一组由标准部署对象管理的 Kubernetes pod。网格中该服务的消费者将自动在虚拟机和 Kubernetes 之间进行负载均衡。`details.bookinfo.com`服务的虚拟机安装了 sidecar，并使用`details-legacy`服务账户进行引导。Sidecar 接收 80 端口的 HTTP 流量（用 istio mutual TLS 包装），并将其转发给同一端口的 localhost 上的应用程序。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: WorkloadEntry
metadata:
  name: details-vm-1
spec:
  serviceAccount: details
  address: 2.2.2.2
  labels:
    app: details
    instance-id: vm1
---
apiVersion: networking.istio.io/v1alpha3
kind: WorkloadEntry
metadata:
  name: details-vm-2
spec:
  serviceAccount: details
  address: 3.3.3.3
  labels:
    app: details
    instance-id: vm2
```

假设还有一个 Kubernetes 部署，带有 pod 标签`app: details`，使用相同的服务账户`details`，下面的服务条目声明了一个横跨虚拟机和 Kubernetes 的服务。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: details-svc
spec:
  hosts:
  - details.bookinfo.com
  location: MESH_INTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: STATIC
  workloadSelector:
    labels:
      app: details
```

关于 ServiceEntry 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/networking/service-entry/)。

## 参考

- [Service Entry - istio.io](https://istio.io/latest/docs/reference/config/networking/service-entry/)