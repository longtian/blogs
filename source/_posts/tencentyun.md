---
date: 2021-03-18 13:28:28
tags:
---

# 腾讯 Centos 软件源的使用


```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
sed -i 's/cloud\.tencent\.com/tencentyun\.com/1'
```