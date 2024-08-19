---
title: 我的第一个 Alpine OS Commit
date: 2024-05-01 00:00:00
tags:
 - alpine
 - devops
---

最近特别爱用 Alpine OS， 试着在上面使用 K3S 搭建一个 Kubernetes 集群。

当时使用的版本是 `Alpine v3.19` ，包管理器里自带的 Kubernetes 版本是 `k3s-1.28.8.1-r1`，安装过程没有报错，但是启动的时候有一个 `Fatal Error`

```bash
time="2024-04-10T00:58:17+08:00" level=fatal msg="Failed to validate golang version: kubernetes golang build version not set - see 'golang: upstream version' in https://github.com/kubernetes/kubernetes/blob/v1.28.8/build/dependencies.yaml"
```

之后定位到是 Alpine OS 包管理器没有及时同步上游的改动的问题，看到有人已经在 `Edge` 频道里修复了 [#15748](https://gitlab.alpinelinux.org/alpine/aports/-/issues/15748)，我就基于这个改动，在 `Community` 频道里也修复了 [#63824](https://gitlab.alpinelinux.org/alpine/aports/-/merge_requests/63824)

<!-- more -->

基本就是照葫芦画瓢，不过还是被 Alpine Maintainer 的响应速度和的 CICD 的流程惊艳到了，提交了 Mege Request 后就会自动出发一个构建 [#225327](https://gitlab.alpinelinux.org/longtian/aports/-/pipelines/225327
)，验证不同的架构下能否正常编译 


| stage | status |
|------|-------|
build-riscv64-emulated | warning
lint | warning
build-aarch64 | pass
build-armhf | pass
build-armv7 | pass
build-ppc64le | pass
build-s390x | pass
build-x86 | pass
build-x86_64 | pass

一天之内这个 Mege Request 就被合并了，修复后的 k3s 版本号为 `k3s-1.28.8.1-r3`，国内的 Alpine 镜像站会有几天的延迟。