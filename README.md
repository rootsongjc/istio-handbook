# Istio Handbook——Istio 服务网格进阶实战

**注意：本书已转移到 [servicemesher/istio-handbook](https://github.com/servicemesher/istio-handbook)，将由 [ServiceMesher 社区](http://www.servicemesher.com)共同撰写，本仓库已不再维护。**

---

[Istio](https://istio.io/zh) 是由 Google、IBM、Lyft 等共同开源的 Service Mesh（服务网格）框架，于2017年初开始进入大众视野。本书是 [Kubernetes Handbook——Kubernetes 中文指南/云原生应用架构实践手册](https://github.com/rootsongjc/kubernetes-handbook)的续篇，Kubernetes 解决了云原生应用的部署问题，Istio 解决是应用的服务（流量）治理问题。在 Kubernetes Handbook 中有大量的关于 Istio 和服务网格相关的章节，这些内容已经与 Kubernetes 本身没有太强的关系，随着 2018年7月31日 [Istio 1.0](https://istio.io/zh/blog/2018/announcing-1.0/) 发布，Istio 本身已经日趋稳定，作为 [ServiceMesher 社区](http://www.servicemesher.com) 的联合创始人我也在社区活动中积累的大量的资料，为了回馈社区，我决定组合社区中已有的资料加上个人撰写，将服务网格部分独立出来单独成书。

本书的主题包括：

- 服务网格概念解析
- 控制平面和数据平面的原理
- Istio 架构详解
- Istio 进阶实战
- 基于 Istio 的自定义扩展

本书基于 Istio 1.0+ 版本编写，您可以通过以下地址参与到本书的编写或阅读本书：

- GitHub 地址：https://github.com/rootsongjc/istio-handbook
- Gitbook 在线浏览：https://jimmysong.io/istio-handbook/

## 快速开始

阅读本书前希望您有容器和 Kubernetes 的基础知识，如果您想要从零开始，那么可以使用 [**kubernetes-vagrant-centos-cluster**](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster) 并运行 [Bookinfo 应用](https://istio.io/zh/docs/examples/bookinfo/)来快速体验服务网格。

## 致谢

[ServiceMesher 社区](http://www.servicemesher.com) 负责翻译了 [Envoy 官方文档](http://www.servicemesher.com/envoy/)，并负责了 [Istio 官方中文文档](https://istio.io/zh)的维护，同时还编译了大量资料，本书所有文章的文末**参考**栏目里标注了参考文章的链接，感谢大家对本书的大力支持。

## 版权

<p align="center">
  <a href="https://jimmysong.io/istio-handbook">
    <img src="cover.jpg" width="40%" alt="Istio handbook - istio服务网格实践指南" />
  </a>
</p>

本书概念图，封面图片[上海静安寺夜景](https://jimmysongio.tuchong.com/24318231/)，[Jimmy Song](https://jimmysong.io) 摄。

本书发行版权归属于电子工业出版社博文视点，未经授权请勿私自印刷发行。

## 参与本书

请参考 [Istio 网站样式指南](https://istio.io/zh/about/contribute/style-guide/)。

## ServiceMesher 社区

更多关于服务网格的资讯、技术干货请关注我们的社区。

- 社区网站：http://www.servicemesher.com

- 微信公众号：ServiceMesher

![ServiceMesher 微信公众号](https://ws2.sinaimg.cn/large/006tNc79ly1fz6cq93dwmj31jt0beq9s.jpg)