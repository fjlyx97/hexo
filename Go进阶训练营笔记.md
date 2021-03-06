---
title: Go进阶训练营笔记
date: 2020-12-07 21:30:00
tags:
- linux
- 学习
categories : "golang家族"
---

> - Go进阶训练营笔记
> - 基于极客时间专栏

<!-- more-->

# 微服务概览与治理
微服务可以看成SOA（面向服务）的一种实践
- 小即是美：代码少，bug少，易于测试维护
- 单一职责：一个服务只需要做好一件事
- 尽可能早创建原型：尽可能早的提供服务API，建立服务契约，达成服务间沟通的一致性约定
- 可移植性比效率更重要

# 可用性&兼容性设计
一旦采用微服务，服务变更时要小心，时刻需要记住兼容性。
- 发送时要保守，接收时要开放。最小化的传送必要的信息，接收时可以容忍冗余数据，保证兼容性

# microservice安全
## 外网服务
对于外网请求，一般通过api gateway进行统一认证拦截，认证成功后，使用JWT,通过rpc元数据的方式，带到bff层，bff校验完token完整性，把身份注入到应用的context中。

## 内网服务
先身份认证（基于证书），再授权(RBAC服务)