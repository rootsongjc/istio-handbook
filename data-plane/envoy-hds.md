# HDS（健康发现服务）

HDS（健康发现服务）支持管理服务器对其管理的 Envoy 实例进行高效的端点健康发现。单个 Envoy 实例通常会收到 HDS 指令，以检查所有端点的子集（subset）。运行状况检查子集可能并不是 Envoy 实例 EDS 端点的子集。

## 参考

- [Health Discovery Service (HDS) - github.com](https://github.com/envoyproxy/envoy/blob/master/api/envoy/service/discovery/v2/hds.proto)
