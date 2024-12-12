---
title: Kubernetes简介
date: 2024-06-27 09:05:55
tags:
---

node:集群中的一个节点，可以是物理机或虚拟机
pod:集群中的最小调度节点(可以理解为是容器的抽象)，一般情况一个pod中只运行一个容器
svc:将一组pod封装成服务，可以提供一个统一入口来访问这个服务
ing:ingres管理从集群外部访问集群内部的方式
cm:管理镜像配置信息(配置信息是明文存储)
secret:对cm信息进行Base64转码
deploy:
sts:有状态实例部署
vol:



https://jimmysong.io/book/kubernetes-handbook/architecture/