# Gateway

`Gateway` 描述了一个在网格边缘运行的负载均衡器，接收传入或传出的 HTTP/TCP 连接。该规范描述了一组应该暴露的端口、要使用的协议类型、负载均衡器的 SNI 配置等。

## 示例

例如，下面的 Gateway 配置设置了一个代理，作为负载均衡器，暴露了 80 和 9080 端口（http）、443（https）、9443（https）和 2379 端口（TCP）的入口（ingress）。网关将被应用于运行在标签为 `app: my-gateway-controller` 的 pod 上的代理。虽然 Istio 将配置代理来监听这些端口，但用户有责任确保这些端口的外部流量被允许进入网格。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      httpsRedirect: true # 对 HTTP 请求发送 301 重定向
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      mode: SIMPLE # 在此端口上启用 HTTPS
      serverCertificate: /etc/certs/servercert.pem
      privateKey: /etc/certs/privatekey.pem
  - port:
      number: 9443
      name: https-9443
      protocol: HTTPS
    hosts:
    - "bookinfo-namespace/*.bookinfo.com"
    tls:
      mode: SIMPLE # 在此端口上启用 HTTPS
      credentialName: bookinfo-secret # 从 Kubernetes secret 中获取证书
  - port:
      number: 9080
      name: http-wildcard
      protocol: HTTP
    hosts:
    - "*"
  - port:
      number: 2379 # 通过此端口暴露内部服务
      name: mongo
      protocol: MONGO
    hosts:
    - "*"
```

上面的网关规范描述了负载均衡器的 L4 到 L6 属性。然后，一个 VirtualService 可以被绑定到一个 Gateway 上，以控制到达特定主机或网关端口的流量的转发。

例如，下面的 VirtualService 将 `https://uk.bookinfo.com/reviews`、`https://eu.bookinfo.com/reviews`、 `http://uk.bookinfo.com:9080/reviews`、`http://eu.bookinfo.com:9080/reviews` 的流量分成两个版本（prod 和 qa）的内部 reviews 服务，端口为 9080。此外，包含 cookie `user: dev-123` 的请求将被发送到 qa 版本的特殊端口 7777。同样的规则也适用于网格内部对 `reviews.prod.svc.cluster.local` 服务的请求。这个规则适用于 443、9080 端口。请注意，`http://uk.bookinfo.com` 会被重定向到 `https://uk.bookinfo.com`（即 80 重定向到 443）。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-rule
  namespace: bookinfo-namespace
spec:
  hosts:
  - reviews.prod.svc.cluster.local
  - uk.bookinfo.com
  - eu.bookinfo.com
  gateways:
  - some-config-namespace/my-gateway
  - mesh # 应用到网格中所有的 sidecar
  http:
  - match:
    - headers:
        cookie:
          exact: "user=dev-123"
    route:
    - destination:
        port:
          number: 7777
        host: reviews.qa.svc.cluster.local
  - match:
    - uri:
        prefix: /reviews/
    route:
    - destination:
        port:
          number: 9080 # 如果它是 reviews 的唯一端口，则可以省略。
        host: reviews.prod.svc.cluster.local
      weight: 80
    - destination:
        host: reviews.qa.svc.cluster.local
      weight: 20
```

下面的 VirtualService 将到达（外部）27017 端口的流量转发到 5555 端口的内部 Mongo 服务器。这个规则在网格内部不适用，因为网关列表中省略了保留名称 `mesh`。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo-mongo
  namespace: bookinfo-namespace
spec:
  hosts:
  - mongosvr.prod.svc.cluster.local # 内部 Mongo 服务的名字
  gateways:
  - some-config-namespace/my-gateway # 如果网关与虚拟服务处于同一命名空间，可以省略命名空间。
  tcp:
  - match:
    - port: 27017
    route:
    - destination:
        host: mongo.prod.svc.cluster.local
        port:
          number: 5555
```

以使用 `hosts` 字段中的命名空间/主机名语法来限制可以绑定到网关服务器的虚拟服务集。例如，下面的网关允许 `ns1` 命名空间中的任何虚拟服务与之绑定，而只限制 `ns2` 命名空间中的 `foo.bar.com` 主机的虚拟服务与之绑定。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "ns1/*"
    - "ns2/foo.bar.com"
```

关于 Gateway 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/networking/gateway/)。

## 参考

- [Gateway - istio.io](https://istio.io/latest/docs/reference/config/networking/gateway/)