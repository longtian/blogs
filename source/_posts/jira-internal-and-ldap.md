---
title: JIRA 内置用户目录与 LDAP 的关系
date: 2020-08-12 10:56:34
tags:
---

作为运维开发怎么能不折腾一下 JIRA 呢。JIRA 支持从多个来源获取用户信息，默认的是内置用户目录，一个比较常见需求是改用 LDAP 作为用户目录，特别适合公司里有多个账号体系的情况。

如果想了解其中的细节最好方式还是搭个独立的测试环境，主要是 Jira, OpenLDAP 和 MySQL，可以用一个 docker-compose 搞定。安装和配置部分略去，以下直接给出分析过程：

## 相关的表

 表名            | 功能
----------------|--------------------
cwd_directory   | 用户目录
cwd_group       | 用户组
cwd_user        | 用户
cwd_membership  | 用户组 ->　用户


**cwd_directory**

ID      |  directory_name         | impl_class                                      | directory_type
--------|-------------------------|-------------------------------------------------|---------------
1       | JIRA Internal Directory | com.atlassian.crowd.directory.InternalDirectory | INTERNAL
10000   | My OpenLDAP             | com.atlassian.crowd.directory.OpenLDAP          | CONNECTOR

之前看到网上有一条语句改库完成 LDAP 迁移的神操作，这个操作会假设 OpenLDAP 的库 ID 是 10000，果然是艺高人胆大。

**cwd_group**

ID      | directory_id        |  group_name         | active  | local | group_type  
--------|---------------------|---------------------|---------|-------|-----------
10000   | 1                   | jira-administrators | 1       | 0     | GROUP
10010   | 1                   | jira-software-users | 1       | 0     | GROUP
10110   | 1                   | jira-software-users | 1       | 1     | GROUP

**cwd_user**

ID     | directory_id        | username             | active  | email_address         | CREDENTIAL
-------|---------------------|----------------------|---------|-----------------------|--------------------
10000  | 1                   | root                 | 1       | root@example.com      | {PKCS5S2}********
10102  | 10000               | foo                  | 1       | foo@example.com       | nopass
10100  | 1                   | foo                  | 1       | foo@example.com       | {PKCS5S2}********

迁移之前最大的顾虑就是迁移前的数据能够保存了，这块主要分两部分

- 各种任务，评论等
- 在项目里的角色
- 用户组信息

通过实验发现只要保证 internal 和 ldap 里的 username 一致 1 和 2 就可以得到保留，同样，只要保证 group_name 一致 3 就可以得到保留。

## 结论

结合表里的数据可得到以下结论：

- username 和 group_name 在各自的库里都是唯一的
- 从全局看 username 对应的用户只能有一个，取决于目录的优先级顺序
- 从全局看 group_name 是不同目录下的用户的集合
