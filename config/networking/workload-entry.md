#WorkloadEntry

`WorkloadEntry` 使运维能够描述单个非 Kubernetes 工作负载的属性，如虚拟机或裸机服务器，因为它被加入网格。一个 `WorkloadEntry` 必须伴随着一个 Istio `ServiceEntry`，通过适当的标签选择工作负载，并提供 `MESH_INTERNAL` 服务的服务定义（主机名、端口属性等）。一个 `ServiceEntry` 对象可以根据服务条目中指定的标签选择器来选择多个工作负载条目以及 Kubernetes pod。

当工作负载连接到 istiod 时，自定义资源中的状态字段将被更新，以表明工作负载的健康状况以及其他细节，类似于 Kubernetes 更新 pod 状态的方式。

## 示例

下面的例子声明了一个工作负载条目，代表 `details.bookinfo.com` 服务的一个虚拟机。这个虚拟机安装了 sidecar，并使用 `details-legacy` 服务账户进行引导。该服务通过 80 端口暴露给网格中的应用程序。通往该服务的 HTTP 流量被 Istio mTLS 封装，并被发送到目标端口 8080 的虚拟机上的 sidecar，后者又将其转发到同一端口的 localhost 上的应用程序。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: WorkloadEntry
metadata:
  name: details-svc
spec:
  # 使用服务账户表明工作负载有一个用该服务账户引导的 sidecar 代理。有 sidecar 的 pod 会自动使用 istio mTLS 与工作负载通信。
  serviceAccount: details-legacy
  address: 2.2.2.2
  labels:
    app: details-legacy
    instance-id: vm1
```

与其相关的服务条目如下。

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
    targetPort: 8080
  resolution: STATIC
  workloadSelector:
    labels:
      app: details-legacy
```

下面的例子使用其完全限定的 DNS 名称声明了同一个虚拟机工作负载。服务条目的解析模式应改为 DNS，以表明客户端侧设备在转发请求之前应在运行时动态地解析 DNS 名称。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: WorkloadEntry
metadata:
  name: details-svc
spec:
  # 使用服务账户表明工作负载有一个用该服务账户引导的 sidecar 代理。有 sidecar 的 pod 会自动使用 istio mTLS 与工作负载通信。
  serviceAccount: details-legacy
  address: vm1.vpc01.corp.net
  labels:
    app: details-legacy
    instance-id: vm1
```

与其相关的服务条目如下。

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
    targetPort: 8080
  resolution: DNS
  workloadSelector:
    labels:
      app: details-legacy
```

关于 WorkloadEntry 配置的详细用法请参考 [Istio 官方文档](https://istio.io/latest/docs/reference/config/networking/workload-entry/)。

## 参考

- [WorkloadEntry - istio.io](https://istio.io/latest/docs/reference/config/networking/workload-entry/)

