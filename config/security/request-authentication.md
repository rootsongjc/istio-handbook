# RequestAuthentication

`RequestAuthentication`（请求认证）定义了工作负载支持哪些请求认证方法。如果请求包含无效的认证信息，它将根据配置的认证规则拒绝该请求。不包含任何认证凭证的请求将被接受，但不会有任何认证的身份。

## 示例

为了限制只对经过认证的请求进行访问，应该伴随着一个授权规则。

例如，要求对具有标签 `app:httpbin` 的工作负载的所有请求使用 JWT 认证。 

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: httpbin
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "issuer-foo"
    jwksUri: https://example.com/.well-known/jwks.json
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  rules:
  - from:
    - source:
        requestPrincipals: ["*"]
```

下一个例子展示了如何为不同的 `host` 设置不同的 JWT 要求。`RequestAuthentication` 声明它可以接受由 `issuer-foo` 或 `issuer-bar` 签发的 JWT（公钥集是由 OpenID Connect 规范隐性设置的）。

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: httpbin
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  jwtRules:
  - issuer: "issuer-foo"
  - issuer: "issuer-bar"
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  rules:
  - from:
    - source:
        requestPrincipals: ["issuer-foo/*"]
    to:
    - operation:
        hosts: ["example.com"]
  - from:
    - source:
        requestPrincipals: ["issuer-bar/*"]
    to:
    - operation:
        hosts: ["another-host.com"]
```

你可以对授权策略进行微调，为每个路径设置不同的要求。例如，除了 `/healthz`，所有路径都需要 JWT，可以使用相同的 `RequestAuthentication`，但授权策略可以是：

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: httpbin
  namespace: foo
spec:
  selector:
    matchLabels:
      app: httpbin
  rules:
  - from:
    - source:
        requestPrincipals: ["*"]
  - to:
    - operation:
        paths: ["/healthz"]
```

关于 `RequestAuthentication` 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/security/request_authentication/)。

# 参考

- [RequestAuthentication - istio.io](https://istio.io/latest/docs/reference/config/security/request_authentication/)

