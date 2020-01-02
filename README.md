# LearnOpenshiftOn-learn.openshift.com
Leverage redhat learn.openshift.com web site to learn Openshift and Kubernetes features and skills. Chinese version.

Redhat 基于 katacoda 技术提供了一个非常有价值的学习/研究网站
https://learn.openshift.com
所有有志于学习kubernetes或是OpenShift（Orgin）技术的同学，无论基于何种角度或是何种目的，都可以从这个网站上快速学习到很多很有价值的知识和信息。
作为现役 Redhat 员工，我个人觉得这个网站虽然设计的非常出色，而且信息非常充足，但更多的关注于技术本身，更多的描述“How to...”而很少解释"Why to ...."。 因此基于个人经验在github上分享一下我对这些技术的理解和应用场景的讨论。 基于个人立场的不同，每个人对技术的发展与目标总用各种不同的理解。 因此本文更多的是个人行为，不完全代表 Redhat 公司的官方观点。

## learn.openshift.com and Katacoda
https://learn.openshift.com

目前 Redhat 为 learn.openshift.com 规划了六个大类的课程，每个课程分为很多场景（Scenario），每个场景中包含众多的步骤。 实际上很多步骤都包含众多的知识点。跟随场景的教程一一完成教学内容后，用户通常似乎了解了很多但又似乎有多细节不甚了了。因此在这个 Repo 中，作者更注重另读者可以更好的理解这些场景为何要这样做，他在真实环境中究竟能解决什么问题。

## OpenShift Course 课程系列

1. Foundations of OpenShift 
    -Openshift基础能力介绍

### 教程场景 Scenarios

2. Building Applications On OpenShift 
    -如何借助 Openshift 快速搭建App

### 教程场景 Scenarios

3. Subsystems, Components, and Internals 
    -Openshift的子系统、组件和内部模块

### 教程场景 Scenarios

4. OpenShift Playgrounds 
    -Openshift Playground 完整场景模拟使用

### 教程场景 Scenarios


5. Service Mesh Workshop with Istio 
    -Openshift 微服务 Istio 构建教程

### 教程场景 Scenarios

6. Building Operators on OpenShift 
    -Openshift Operator Framework 框架介绍
  Operator Framework 服务框架是目前非常重要的一个 kubernetes 开源项目。由 Redhat 主导。它的目标是令 Kubernetes 上的应用以一种更加封装，更加整洁，更加原子化实现业务能力的整体发布、供应、服务、回收。是一个为业务能力能够实现整体能力供应，并能够实现全生命周期管理的服务框架。它的发展主旨是令 Kubernetes 框架更加符合 PaaS 平台的业务需求。因此是当前非常重要且热度很高的社区项目。 （Operator Framework 社区项目地址 https://github.com/operator-framework）

### 教程场景 Scenarios

    - [Kubernetes API Fundamentals](https://github.com/zhaoxiyi/LearnOpenshiftOn-learn.openshift.com/blob/master/1-FoundationsOfOpenshift/KubernetesAPIFund.md)
        Kubernetes API 的基础服务方式与如何提供 Operator API 给Operator 消费者或供应者

    - Etcd Operator
        标准的 Etcd Operator ，Etcd Operator 由社区提供，可以基于OpenShift 直接借助 Operator 的能力实现一键或单命令实现 Etcd 集群供应，包括供应时直接可以包含相应的持久化定制和集群 Scale up Scale down 能力。将 Etcd 的供应基于 RBAC 角色能力实现 Etcd 数据库供应者与数据库消费者角色分离等更加贴合真实世界使用场景的 PaaS 化使用方式。

    - Operator SDK with Go
        使用 Go 语言快速操作 Operator SDK 来完成多种复杂任务

    - Operator Lifecycle Manager
        基于 Operator Framework 的能力来实现 Operator 控制业务能力的生命周期控制

    - Ansible Refresher
        Ansible Operator 实现 Refresher 能力

    - Ansible Kubernetes Modules
        Openshift 上已经实现的 Ansible Operator 能力，实现基于 Ansible 控制各类 Kubernetes Module 模块。实现多种常用 Kubernetes 能力

    - Ansible Operator Overview
        详述 Ansible Operator 可以实现那些能力

    - Mcrouter Operator powered by Ansible Operator
        通过 Ansible Operator 实现的 Mcrouter Operator 用以演示如何使用 Ansible Operator 来实现更有业务价值的能力 Operator

    - Operator SDK with Helm
        通过 Helm 操作 Operator SDK ，协助消费者或供应者快速基于 Helm 使用或者实现新的 Operator
