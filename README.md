# LearnOpenshiftOn-learn.openshift.com
Leverage redhat learn.openshift.com web site to learn Openshift and Kubernetes features and skills. Chinese version.

Redhat 基于 katacode 技术提供了一个非常有价值的学习/研究网站 <br>
https://learn.openshift.com
所有有志于学习kubernetes或是OpenShift（Orgin）技术的同学，无论基于何种角度或是何种目的，都可以从这个网站上快速学习到很多很有价值的知识和信息。
作为现役 Redhat 员工，我个人觉得这个网站虽然设计的非常出色，而且信息非常充足，但更多的关注于技术本身，更多的描述“How to...”而很少解释"Why to ...."。 因此基于个人经验在github上分享一下我对这些技术的理解和应用场景的讨论。 基于个人立场的不同，每个人对技术的发展与目标总用各种不同的理解。 因此本文更多的是个人行为，不完全代表 Redhat 公司的官方观点。

## learn.openshift.com and Katacode
目前 Redhat 为 learn.openshift.com 规划了六个大类的课程，每个课程分为很多场景（Scenario），每个场景中包含众多的步骤。 实际上很多步骤都包含众多的知识点。跟随场景的教程一一完成教学内容后，用户通常似乎了解了很多但又似乎有多细节不甚了了。因此在这个 Repo 中，作者更注重另读者可以更好的理解这些场景为何要这样做，他在真实环境中究竟能解决什么问题。<br>
<i>如果您对 Katacode 是如何构建在 Openshift 上感兴趣，您可以参考下面这个网站的介绍。 
https://github.com/openshift-labs/learn-katacoda 
这个项目详细描述了 Redhat 是如何在 OpenShift 上建立一个新的实验环境的。也许这对您的企业同样有价值。</i>

## OpenShift Course 课程系列

1. Foundations of OpenShift 
    -Openshift基础能力介绍

    ###### 教程场景 Scenarios 1

2. Building Applications On OpenShift 
    -如何借助 Openshift 快速搭建App

    ###### 教程场景 Scenarios 2

3. Subsystems, Components, and Internals 
    -Openshift的子系统、组件和内部模块

    ###### 教程场景 Scenarios 3

4. OpenShift Playgrounds 
    -Openshift Playground 完整场景模拟使用

    ###### 教程场景 Scenarios 4


5. Service Mesh Workshop with Istio 
    -Openshift 微服务 Istio 构建教程
    <font color=red>（制作中···）</font><br>
    ###### 教程场景 Scenarios 5

    - [Istio 介绍](5-IstioServMeshOnOpenshift/istio_intro.md)

    - [微服务发布](5-IstioServMeshOnOpenshift/deploy_microservices.md)

    - [监控与跟踪](5-IstioServMeshOnOpenshift/monitor_tracing.md)

    - [简单路由](5-IstioServMeshOnOpenshift/simple_routing.md)

    - [高级路由规则](5-IstioServMeshOnOpenshift/advanced_routerule.md)

    - [故障注入](5-IstioServMeshOnOpenshift/falut_injection.md)

    - [熔断](5-IstioServMeshOnOpenshift/circuit_breaker.md)

    - [Egress 外调](5-IstioServMeshOnOpenshift/egress.md)

    - [通过 Kiali 监控](5-IstioServMeshOnOpenshift/observing_with_kiali.md)

    - [安全通讯 Mutual TLS](5-IstioServMeshOnOpenshift/istio_intro/mutual_tls.md)

6. Building Operators on OpenShift 
    -Openshift Operator Framework 框架介绍
    <font color=red>（制作中···）</font><br>
    <I>Operator Framework 服务框架是目前非常重要的一个 kubernetes 开源项目。由 Redhat 主导。它的目标是令 Kubernetes 上的应用以一种更加封装，更加整洁，更加原子化实现业务能力的整体发布、供应、服务、回收。是一个为业务能力能够实现整体能力供应，并能够实现全生命周期管理的服务框架。它的发展主旨是令 Kubernetes 框架更加符合 PaaS 平台的业务需求。因此是当前非常重要且热度很高的社区项目。 （Operator Framework 社区项目地址 https://github.com/operator-framework）</I>

    ###### 教程场景 Scenarios 6

    - [Kubernetes API Fundamentals](https://github.com/zhaoxiyi/LearnOpenshiftOn-learn.openshift.com/blob/master/1-FoundationsOfOpenshift/KubernetesAPIFund.md)
        

    - [Etcd Operator]()
        

    - [Operator SDK with Go]()
        

    - [Operator Lifecycle Manager]()
        

    - [Ansible Refresher]()
        

    - [Ansible Kubernetes Modules]()
        

    - [Ansible Operator Overview]()
        

    - [Mcrouter Operator powered by Ansible Operator]()
        

    - [Operator SDK with Helm]()
        
