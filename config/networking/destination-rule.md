# DestinationRule

`DestinationRule` 定义了在路由发生后适用于服务流量的策略。这些规则指定了负载均衡的配置、来自 sidecar 的连接池大小，以及用于检测和驱逐负载均衡池中不健康主机的离群检测设置。

## 示例

例如，ratings 服务的一个简单的负载均衡策略看起来如下。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
```

可以通过定义一个命名的 `subset` 并覆盖在服务级别指定的设置来指定特定版本的策略。下面的规则对前往由带有标签（`version:v3`）的端点（如 pod）组成的名为 `testversion` 的子集的所有流量使用轮回负载均衡策略。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
  - name: testversion
    labels:
      version: v3
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
```

**注意**：只有当路由规则明确地将流量发送到这个子集，为子集指定的策略才会生效。

流量策略也可以针对特定的端口进行定制。下面的规则对所有到 80 号端口的流量使用最少连接的负载均衡策略，而对 9080 号端口的流量使用轮流负载均衡设置。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: bookinfo-ratings-port
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy: # 应用到所有端口
    portLevelSettings:
    - port:
        number: 80
      loadBalancer:
        simple: LEAST_CONN
    - port:
        number: 9080
      loadBalancer:
        simple: ROUND_ROBIN
```

关于 DestinationRule 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/networking/destination-rule/)。

## 参考

- [Destination Rule - istio.io](https://istio.io/latest/docs/reference/config/networking/destination-rule/)