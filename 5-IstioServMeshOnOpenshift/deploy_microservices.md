# Istio Introduction

使用下面链接可以直接进入场景
https://learn.openshift.com/servicemesh/1-introduction

场景的定义难度为基础，所需时间为30分钟<br>
   Difficulty: Basic  Estimated Time: 30 minutes    <br>
该场景仅仅是为了给学习者一个概念，服务网格（Service Mesh）究竟是什么，Istio是什么他们究竟表现为一个什么形式。<br>

快速链接: <br>
[场景说明](#场景说明) <br>
[操作步骤](#操作步骤) <br>
[返回](../README.md) <br>

#### 场景说明
Redhat 给该场景的说明

This Scenario will present you the concept of Service Mesh and why we need it.
Then you will be introduced to terms like Mixer, Pilot and Proxy/Sidecar
Finally, you will learn how to install Istio on a Kubernetes/OpenShift cluster

本方案将向您介绍服务网格的概念以及我们为什么需要它。
然后将向您介绍 Envoy，Pilot和 Proxy/Sidecar 等术语
最后，您将学习如何在Kubernetes / OpenShift集群上安装Istio

> 除了以上官方解释外，需要补充的是，本场景使用的是开源的 Istio 1.0.x 版本的 OpenShift 服务网格方案。除此外 Redhat 还提供了基于 Operator 的 Istio 版本和相应的支持，这两个版本虽然管理方式、开发方式基本相同，但安装方式完全不同，且由于版本的不同，内部的一些细节是不同的。有些操作会产生相同操作但结果异常的情况，在这里我希望能尽量解释它们之间的不同和解决方式，以便让读者了解在真实世界的服务网格环境中，怎样屏蔽技术细节对全局架构产生的误导，让大家回归到业务目标方向上来理解服务网格究竟应该用来做什么。
>> 本场景中用到了一个例子，红帽公司提供的 istio-tutorial ，这个例子提供的内容远比本场景提供的要全面和广泛。你可以到 GitHub 上去了解更多的细节，同时在 GitHub 上也提供了关于微服务的一本电子书的链接，只要简单的在 Redhat 公司的网站上免费注册一个账户既可以下载这边电子书。即使是电子书中描述的也并没有完整覆盖完整的所有演示代码内容，更多的是关注于微服务可以实现的流量控制，安全，管控等方面的操作。
>> - https://github.com/redhat-developer-demos/istio-tutorial
>> - https://developers.redhat.com/books/introducing-istio-service-mesh-microservices/
>> 关于整个 Tutorial Redhat 也提供了详细的文档来进行说明
>> - https://redhat-developer-demos.github.io/istio-tutorial/
>>> 但即使有这些说明也没有对 Tutorial 下面的所有内容进行完整的解释，在这个 Tutorial 中 customer/preference 提供了基于 Java 的 `camel-springboot``microprofile``quarkus``springboot` 四种实现方式，还有基于 `donet` `nodejs` 的实现。recommendation 则提供了 基于 Java 的 `camel-springboot``microprofile``quarkus``springboot``virt.x` 的五种实现，和nodejs的实现。大家也可以通过这个 Tutorial 来尝试各种技术组合搭建微服务网格的实验。这题特别推荐一下 Quarkus 技术，Quarkus 是 Redhat 为了推广云化实现所推出的一种极轻量级 Java 执行框架，它能够兼容 90% 的 SpringBoot Annotation，但同时能大幅度降低 Java 实现的 Footprint 使 Java 实现的业务可以更轻量级、更高效的执行 Java 代码。从而使 Java 实现更加适应云化需求、容器化需求。大家可以到 https://quarkus.io 上去了解更多细节。

#### 操作过程
[Step 1](deploy_microservices/Step1.md) <br>
[Step 2](deploy_microservices/Step2.md) <br>
[Step 3](deploy_microservices/Step3.md) <br>
