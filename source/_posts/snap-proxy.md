---
title: Snap 设置代理
date: 2020-05-30 10:34:25
tags: 
 - ubuntu
---

Snap 从 `2.28` 版本开始支持代理设置了, 这样安装开发用的工具就方便很多

```bash
sudo snap set system proxy.http="http://<proxy_addr>:<proxy_port>"
sudo snap set system proxy.https="http://<proxy_addr>:<proxy_port>"
```

分享几个我常用的

**Redis Desktop Manager**

Redis 的一个图形界面客户端

```bash
sudo snap install redis-desktop-manager
```

**Slack**

Slack 客户端

```bash
sudo snap install slack
```

