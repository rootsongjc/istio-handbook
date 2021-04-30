# AuthorizationPolicy

`AuthorizationPolicy`（授权策略）实现了对网格中工作负载的访问控制。

授权策略支持访问控制的 `CUSTOM`、`DENY` 和 `ALLOW` 操作。当 `CUSTOM`、`DENY` 和 `ALLOW` 动作同时用于一个工作负载时，首先评估 `CUSTOM` 动作，然后是 `DENY` 动作，最后是 `ALLOW` 动作。评估是按以下顺序进行：

1. 如果有任何 `CUSTOM` 策略与请求相匹配，如果评估结果为拒绝，则拒绝该请求。
2. 如果有任何 `DENY` 策略与请求相匹配，则拒绝该请求。
3. 如果没有适合该工作负载的 `ALLOW` 策略，允许该请求。
4. 如果有任何 `ALLOW` 策略与该请求相匹配，允许该请求。
5. 拒绝该请求。

Istio 授权策略还支持 `AUDIT` 动作，以决定是否记录请求。`AUDIT` 策略不影响请求是否被允许或拒绝到工作负载。请求将完全基于 `CUSTOM`、`DENY` 和 `ALLOW` 动作被允许或拒绝。

如果工作负载上有一个与请求相匹配的 AUDIT 策略，则请求将被内部标记为应该被审计。必须配置并启用一个单独的插件，以实际履行审计决策并完成审计行为。如果没有启用这样的支持插件，该请求将不会被审计。目前，唯一支持的插件是 [Stackdriver](https://preliminary.istio.io/latest/docs/reference/config/proxy_extensions/stackdriver/) 插件。

## 示例

下面是一个 Istio 授权策略的例子。

它将 `action` 设置为 `ALLOW` 来创建一个允许策略。默认动作是 `ALLOW`，但在策略中明确规定是很有用的。

它允许请求来自：

- 服务账户 `cluster.local/ns/default/sa/sleep` 或
- 命名空间 `test`

来访问以下工作负载：

- 在前缀为 `/info` 的路径上使用 `GET` 方法，或者
- 在路径 `/data` 上使用 `POST` 方法

且当请求具有由 `https://accounts.google.com` 发布的有效 JWT 令牌时。

任何其他请求将被拒绝。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin
  namespace: foo
spec:
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/sleep"]
    - source:
        namespaces: ["test"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/info*"]
    - operation:
        methods: ["POST"]
        paths: ["/data"]
    when:
    - key: request.auth.claims[iss]
      values: ["https://accounts.google.com"]
```

下面是另一个例子，它将 `action` 设置为 `DENY` 以创建一个拒绝策略。它拒绝来自 `dev` 命名空间对 `foo` 命名空间中所有工作负载的 `POST` 请求。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin
  namespace: foo
spec:
  action: DENY
  rules:
  - from:
    - source:
        namespaces: ["dev"]
    to:
    - operation:
        methods: ["POST"]
```

下面的授权策略将 `action` 设置为 `AUDIT`。它将审核任何对前缀为 `/user/profile` 的路径的 GET 请求。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  namespace: ns1
  name: anyname
spec:
  selector:
    matchLabels:
      app: myapi
  action: AUDIT
  rules:
  - to:
    - operation:
        methods: ["GET"]
        paths: ["/user/profile/*"]
```

授权策略的范围（目标）由 `metadata`/`namespace` 和一个可选的 `selector` 决定。

- `metadata`/`namespace` 告诉策略适用于哪个命名空间。如果设置为根命名空间，该策略适用于网格中的所有命名空间。
- 工作负载 `selector` 可以用来进一步限制策略的适用范围。

例如，以下授权策略适用于 `bar` 命名空间中包含标签 `app: httpbin` 的工作负载。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: policy
  namespace: bar
spec:
  selector:
    matchLabels:
      app: httpbin
```

以下授权策略适用于命名空间 `foo` 中的所有工作负载。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
 name: policy
 namespace: foo
spec:
  {}
```

以下授权策略适用于网格中所有命名空间中包含标签 `version: v1` 的工作负载（假设根命名空间被配置为 `istio-config`）。

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
 name: policy
 namespace: istio-config
spec:
 selector:
   matchLabels:
     version: v1
```

- 关于 `AuthorizationPolicy` 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/security/authorization-policy/)。
- 关于认证策略条件的详细配置请参考 [Istio 官方文档](https://preliminary.istio.io/latest/docs/reference/config/security/conditions/)。

## 参考

- [Authorization Policy - istio.io](https://istio.io/latest/docs/reference/config/security/authorization-policy/)
- [Authorization Policy Conditions - istio.io](https://istio.io/latest/docs/reference/config/security/conditions/)