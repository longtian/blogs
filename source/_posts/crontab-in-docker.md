---
title: 在 Docker 容器里使用 Crontab
date: 2020-06-22 22:11:55
tags:
 - docker
---

最近公司有一个需求是在容器里运行 Cron 服务

## Dockerfile

CentOS 上安装 Cron 很简单，一条命令就可以搞定

```bash
yum install cronie
```

目前的 `cronie` 版本是 `1.4.11`

```
Name        : cronie
Arch        : x86_64
Version     : 1.4.11
Release     : 23.el7
Size        : 215 k
Repo        : installed
From repo   : base
Summary     : Cron daemon for executing programs at set times
URL         : https://github.com/cronie-crond/cronie
License     : MIT and BSD and ISC and GPLv2+
Description : Cronie contains the standard UNIX daemon crond that runs specified programs at
            : scheduled times and related tools. It is a fork of the original vixie-cron and
            : has security and configuration enhancements like the ability to use pam and
            : SELinux.
```

直接基于 CentOS7 编写 Dockerfile

```dockerfile
FROM centos:7
RUN groupadd --gid 1000 foo &&\
    useradd  --uid 1000 --gid foo foo
RUN yum install cronie -y
CMD ["crond","-n","-x","misc,load","-i"]
```

## 启动命令

Cron 的入口当然是 crond 啦，不过 crond 需要前台运行

```bash
crond -n -i -x misc,load
```

其中：

`-n` 进程挂前台

`-i` 关闭 inotify，因为容器里的 inotify 不生效因此检测不到挂载文件的变更

`-x` 开启调试信息： misc 可以打印命令的输出，load 可以显示读取了哪些配置，其它配置项见文末参考

## 配置文件

cron 的配置文件可以在构建镜像的时候打进去，也可以按照需要挂在

### 挂载为用户服务

```bash
docker run -it --rm -v ./foo:/var/spool/cron/foo cron crond -n -x misc,load -i
```

`foo` 文件

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
 
* * * * * id
```

### 挂载为系统服务

```bash
docker run -it --rm -v ./root:/etc/crontab cron crond -n -x misc,load -i
```

`crontab` 文件

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
 
* * * * * root id
```


注意 /etc/crontab 和 /var/spool/cron/foo 不一样还有第 6 个参数，即指定运行的用户



## 参考

| 参数        | 备注                     |
|------------|--------------------------|
| ext	     | 通常和其它几个变量组合使用
| sch	     | scheduler
| proc	     | 进程控制
| pars	     | 语法解析
| load	     | 读取
| misc	     | 杂项
| test	     | 测试模式，命令实际不会执行              
| bit	     | ？？     