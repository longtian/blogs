---
title: "[译] 书写灵活、可维护，可扩展的 Ansible Playbook"
date: 2018-10-05 11:20:35
categories:
    - 翻译
---

[原文](https://www.ansible.com/blog/make-your-ansible-playbooks-flexible-maintainable-and-scalable)

自从 2013 年开始使用 Ansible，我已经用 Ansible 自动化完成很多事情：SaaS 服务，树莓派集群，家庭自动化系统，甚至我自己的电脑。

从那以后，我学会了很多能够降低维护负担的技巧。对我来说项目的可维护性异常重要，因为我的很多项目，例如一个 Apache Solr 项目已经存在超过 10 年了！如果项目难维护或者架构上难以做出大的改变，我会把项目输给其它更敏捷（nimble）的对手，进而丢掉金钱，更重要的是我可能会疯掉。

今年我会在奥斯汀举办的 AnsibleFest 上做一个[同名分享](https://agenda.fest.ansible.com/SessionDetail.aspx?id=482018)，本文即总结这次分享的主题。

## 保持井井有条

我喜欢摄影和自动化，所以我花了很多时间在涉及树莓派和相机的电子项目上。如果没有图中的组织系统，想要把部件放到正确的位置会是一件让人沮丧的事情。

![组织系统](https://www.ansible.com/hs-fs/hubfs/45982928-455cdb80-c020-11e8-96e4-833efbac87f4.jpg)

同样的，在 Ansible 中，我喜欢把我常用的 task 组织起来，这样才能更轻松编写和测试它们，并且不需要太多的精力就可以管理好它们。

开始的时候我会把所有的 task 写到一个 playbook 文件里。当文件到达 100 行左右，我会把相关的任务拆分到不同的文件里，并在 playbook 中使用 include_tasks 引入它们。

随着 playbook 越来越复杂，我经常注意到有一些相关性很高的的 task 可以被独立开，例如安装一个软件、拷贝配置文件、启动（重启）一个守护进程。这种情况下我会用 `ansible-galaxy init ROLE_NAME` 命令新建一个 role，并且把那些 tasks 放进这个 role 里。

如果这个 role 够通用，我会把 role 放到 Github 并且提交到 Ansible Galaxy 里，又或者放到一个单独的私有 Git 仓库里。现在我可以通过 [Molecule](https://github.com/metacloud/molecule/) 或者其它测试工具为 role 添加一系列的通用测试，哪怕这些 role 被隶属不同的团队的不同项目所使用。

之后我会通过 `requirements.txt` 文件把外部的 role 引入到项目里。对于某些稳定性至关重要的项目，我会通过 git ref 或者 tag 指定 role 的版本。对于其它项目我则会牺牲一点稳定性以换取更好的可维护性（例如测试 playbook 或者一次性的服务器配置），我直接使用 role 的名字（如果不在 Ansible Galaxy 上就指定仓库的详情）。

对于大部分项目我都不会把外部 role 提交到代码仓库里，因为在 CI 系统里每次从头运行的时候都会去安装。但是在有一些情况下，最好把所有的 role 都提交到仓库里。比如有一些开发者日常会用到我写的 [Drupal 虚拟机](https://www.drupalvm.com/)的 playbook，这些开发者通常住在离 Ansible Galaxy 服务器很远的地方，所以他们在安装大量必要依赖的时候会遇到麻烦。因此我把所有 role 都提交到仓库里了，这样他们在构建一个新的 Drupal 虚拟机实例的时候就不用等着所有 role 安装完成了。

如果你真的把 role 都提交到仓库里了，你需要在每次更新 role 的时候有一个彻底（thorough）的流程，确保你的 `requirements.yml` 文件和已经安装的 role 同步！我通常通过 `ansible-galaxy install -r requirements.yml --force` 命令来强制替换仓库里的 role，并且保持诚实（踏实？）！

## 简化和优化

> YAML 不是一门编程语言
> \- Jeff Geerling

大家喜欢用 Ansible 的一个原因是它基于 YAML 并且拥有一套声明式的语法。如果你要安装一个模块就在 task 里这样写：`package: name=httpd state=present`。如果你要确保一个服务运行就这样写 `service: name=httpd state=started`

然而在很多情况下，你会需要让一切更智能化。举个例子，如果你用相同的 role 构建虚拟机和容器，但是你并不想在容器里启动服务，你需要增加一个只在某些条件下执行（when condition）的限制：

```yaml
- name: Ensure Apache is started
  service:
    name: httpd
    state: started
  when: 'server_type != "container" '
```

此类逻辑是简单的，并且在别人阅读 task 以搞清楚它的目的时候很有用。但是有的人会在 when condition 里写上一大堆花里胡哨的判断，甚至是 Ansible 暴露出的 Jinja2 和 Python 的接口，这种情况下容易失控（get off rails）。

根据经验（as a rule of thumb），如果你在 playbook 的 when condition 里为了正确的转义引号上花费了 10 分钟以上，你这时候就应该考虑写一个单独的模块来完成 task 用到的逻辑。Python 脚本通常应该位于独立的模块里，而不是和其它的 YAML 写到行内。当然也有例外（比如比较复杂的字典和字符串时），但我会努力避免在 Ansible playbook 里写任何复杂的代码。

除了避免复杂逻辑，还有一个很有效的方法是让 playbook 运行更快。我经常 profile 一个 playbook （通过设置 callback_whitelist = profile_roles, profile_tasks, timer 默认参数），发现一两个 task 或者 role 和 playbook 其它的相比花了很长的时间。

举个例子，有一个 playbook 里用了 copy 模块来复制一个有几十个文件的大目录。由于 Ansible 拷贝文件模块的内部实现，复制每个文件都意味着一直在 SSH 链接上等待着传输完成。

把这个 task 改成基于 synchronize 的可以在每次运行的时候节省好长时间。针对单次运行这看起来没什么，但是当 playbook 需要定期运行的时候（例如确保一台服务器的配置），或者作为 CI 流程的一部分的时候保持它的高效就很重要了。否则它会让 CPU 周期耗费在一些无用代码上，开发者通常会很讨厌等待 CI 测试通过，他们只想知道代码会不会导致问题。

请关注我在 AnsibleFest Austin 2018 的演讲 [Make your Ansible Playbooks flexible, maintainable, and scalable](https://agenda.fest.ansible.com/SessionDetail.aspx?id=482018)。如果这还不够，我在我的书 《Ansible for DevOsp》 里有很多关于编写和维护 Ansible Playbook 的文章（[你可以从 Red Hat 官网下载到摘录](https://www.ansible.com/resources/ebooks/ansible-for-devops)）。
