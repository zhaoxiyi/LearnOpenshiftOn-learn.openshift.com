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

除了以上官方解释外，需要补充的是，本场景使用的是开源的 Istio 1.0.x 版本的 OpenShift 服务网格方案。除此外 Redhat 还提供了基于 Operator 的 Istio 版本和相应的支持，这两个版本虽然管理方式、开发方式基本相同，但安装方式完全不同，且由于版本的不同，内部的一些细节是不同的。有些操作会产生相同操作但结果异常的情况，在这里我希望能尽量解释它们之间的不同和解决方式，以便让读者了解在真实世界的服务网格环境中，怎样屏蔽技术细节对全局架构产生的误导，让大家回归到业务目标方向上来理解服务网格究竟应该用来做什么。


#### 操作过程
[Step 1](istio_intro/Step1.md) <br>
[Step 2](istio_intro/Step2.md) <br>
[Step 3](istio_intro/Step3.md) <br>
