# PeerAuthentication

`PeerAuthentication`（对等认证）定义了流量将如何被隧道化（或不被隧道化）到 sidecar。

## 示例

策略允许命名空间 `foo` 下所有工作负载的 mTLS 流量。

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  mtls:
    mode: STRICT
```

对于网格级别，根据你的 Istio 安装，将策略放在根命名空间。

策略允许命名空间 `foo` 下的所有工作负载的 mTLS 和明文流量，但 `finance` 的工作负载需要 mTLS。

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  mtls:
    mode: PERMISSIVE
---
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  selector:
    matchLabels:
      app: finance
  mtls:
    mode: STRICT
```

政策允许所有工作负载严格 mTLS，但 8080 端口保留为明文。

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  selector:
    matchLabels:
      app: finance
  mtls:
    mode: STRICT
  portLevelMtls:
    8080:
      mode: DISABLE
```

从命名空间（或网格）设置中继承 mTLS 模式的策略，并覆盖 8080 端口的设置。

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: foo
spec:
  selector:
    matchLabels:
      app: finance
  mtls:
    mode: UNSET
  portLevelMtls:
    8080:
      mode: DISABLE
```

关于 `PeerAuthentication` 配置的详细用法请参考 [Istio 官方文档](https://preliminary.istio.io/latest/docs/reference/config/security/peer_authentication/)。

# 参考

- [PeerAuthentication- istio.io](https://istio.io/latest/docs/reference/config/security/peer_authentication/)