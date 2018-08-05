---
title: 使用 Node.js 开发一个 QEMU 的网页版入口
date: 2014-08-08 12:00:00
tags:
---

[源码](https://github.com/longtian/noemu)

NOEMU (Node <3 QEMU)

**Why this project**

I'm a front-end developer and I'm learning Node
I'm also learning QEMU, which works as a hypersior
I want to build a light weighted VM host using these two technology

**Why QEMU**

QEMU is the simplest hypervisor I can thinkof, it is easy to install on Ubuntu
It has great performance on virtual machines, compared to other hypervisors.
QEMU has great VDI to access virtual machines via SPICE and VNC
SPICE is friendly to HTML5

**Project Goals**

VM Host
RESTFul Management API

**What API will be provided**

Create Image
Delete Image
Create VM
Delete VM
Power On VM
Power Off VM
Configure VM
Take Snapshot
Restore Snapshot
