# Istio 开发环境配置

本文将概述如何配置 Istio 的开发环境及编译和生成二进制文件和 Kubernetes 的 YAML 文件，更高级的测试、格式规范、原型和参考文档编写等请参考 [Istio  Dev Guide](https://github.com/istio/istio/wiki/Dev-Guide)。

## 依赖环境

Istio 开发环境依赖以下软件：

- [Docker](https://docs.docker.com/install/)：测试和运行时
- [Go 1.11](https://golang.org)：程序开发
- [fpm 包构建工具](https://fpm.readthedocs.io/en/latest/intro.html)：用来打包
- [Kubernetes](https://jimmysong.io/kubernetes-handbook) 1.7.3+

## 设置环境变量

在编译过程中需要依赖以下环境变量，请根据你自己的

```bash
export ISTIO=$GOPATH/src/istio.io
# DockerHub 的用户名
USER=jimmysong
export HUB="docker.io/$USER"

# Docker 镜像的 tag，这里为了方便指定成了固定值，也可以使用 install/updateVersion.sh 来生成 tag
export TAG=$USER

# GitHub 的用户名
export GITHUB_USER=rootsongjc

# 指定 Kubernetes 集群的配置文件地址
export KUBECONFIG=${HOME}/.kube/config
```

## 全量编译

编译过程中需要下载很多依赖包，请确认你的机器可以科学上网。

执行下面的命令可以编译 Istio 所有组件的二进制文件。

```bash
make
```

以在 Mac 下编译为例，编译完成后所有的二进制文件将位于 `$GOPATH/out/darwin_amd64/release`。

执行下面的命令构建镜像。

```bash
make docker
```

执行下面的命令将镜像推送到 DockerHub。

```bash
make push
```

也可以编译单独组件的镜像，详见[开发指南](https://github.com/istio/istio/wiki/Dev-Guide)。

## 构建 YAML 文件

执行下面的命令可以生成 YAML 文件。

```bash
make generate_yaml
```

生成的 YAML 文件位于 repo 根目录的 `install/kubernetes` 目录下。

## 参考

- [Istio Dev Guide - github.com](https://github.com/istio/istio/wiki/Dev-Guide)
