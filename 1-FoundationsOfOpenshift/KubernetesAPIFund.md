# Kubernetes API Fundamentals

使用下面链接可以直接进入场景
https://learn.openshift.com/operatorframework/k8s-api-fundamentals/
本场景用于帮助用户理解 Kubernetes API 的基础服务方式与如何提供 Operator API 给Operator 消费者或供应者。

场景的定义难度为初级，所需时间为30分钟
     Difficulty: beginner  Estimated Time: 30 minutes
但实际上他可以应用到的真实场景却远比场景介绍的要多且复杂。

快速链接
[场景说明](#场景说明)
[操作步骤](#操作步骤)
[其它](#其它)

#### 场景说明
Redhat 给该场景的说明

Before diving into the Operator Framework, this section will give an overview of Kubernetes API fundamentals. Although the Operator SDK makes creating an Operator fun and easy, understanding the structure and features of the Kubernetes API is required.

在深入讨论Operator框架之前，本节将概述Kubernetes API的基本原理。虽然OperatorSDK使创建一个操作员变得有趣和容易，但是需要理解KubernetesAPI的结构和特性。
除了官方解释外，以我个人的经验来看，其实这个场景可以用于 Kubernetes 部分能力的外联和外放上。Kubernetes 或是 OpenShift 都会供应核心的 API Server。 API Server 可以将平台的各种能力以 REST API 的形式直接提供给所有的平台 API 消费者使用。因此无论是我们给 Kubernetes 定制了一个门户，还是使用 OpenShift 自带的 Consle UI 抑或是给 OpenShift 又定义了一个个性化的 UI Portal 其本质都是在使用这个核心 API Server 的 API 能力。 这使得很多种场景的定制变得简单化。
比如我们现在经常自定义的多域 Kubernetes 、 多域 OpenShift、多集群统一管理、混合云管理，都可以基于 API 来定义实现。 包括很多时候 Kubernetes 或是 OpenShift 能力与外围某个业务系统的能力联动或混合都可以以这个 API 为基础实现。 且基于 API 的实现也可以确保用户 UI 与 Kubernetes 核心版本之间的送耦合。 按照现在企业级系统的发展频率来看，用户 UI 的更新与核心 PaaS 平台能力的更新往往是彼此交错的。依赖于 API 协调两者之间的版本交错是一个相当有效的保持系统有效性更新的手段。 
另一个方面，作为容器化 PaaS 平台， Kubernetes 调度核心和 OpenShift 这样的基本完备的 PaaS 平台融入用户自身的系统中，通常是与企业内部多系统多边整合交叉来完成 PaaS 植入的过程，通常需要与企业的 DevOps 体系多边整合，这种整合也需要一种可以供应指定角色权限范围内指向性对接机制。本场景中介绍的 OpenShift 提供的 ‘oc proxy’ 恰好是一种可以提供这种对接机制的外延体系。多数场景下，我们并不希望将 Server API 接口提供给外围使用，因为这意味着我们将 Kubernetes 所有的能力“裸露”提供给了外部，这将面临太多安全问题。因此主流的对外发布能力手段是通过在 OpenShift 内部以 Project 的形式开发一套交互 UI。再通过 UI 将能力发布给用户。但这种“官方模式”并不一定能够符合所有场景，比如与外部 DevOps 工具整合。像这种局部暴露能力场景我们通常希望能够快速、简单、有限的能力交给外部安全使用。 OpenShift 通过 oc 命令客户端体系提供了一种灵活的外延体系。首先我们可以通过 ‘oc’ 命令的认证来认证所在位置的 proxy 能力范围。其次通过简单的 ‘oc proxy [port]' 命令将所在位置作为 API 中转访问点，将能力提供给外部使用，使用户可以很简单的形成一个远程的前置节点。因此这个教学场景可以衍生出一系列复杂的外围操作模型。
当然作为一个原理展示的简单例子，本场景设计之初并没有考虑更多复杂的问题。其中有很多没有涉及到的内容。例如如何与认证体系挂钩，‘oc proxy’ 命令需要又集群管理角色才能使用，那么如何设计一个合理的角色来实现中转是我们需要考虑的问题。此外如何限制使用者使用 API 的能力范围也没有在本场景中设计，需要我们自己考虑。

#### 操作过程
[1] 第一步，建立项目，设计多容器单Pod项目描述
Let's begin my creating a new project called myproject:
```
oc new-project myproject
```
Create a new pod manifest that specifies two containers:
```
cat > pod-multi-container.yaml <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: my-two-container-pod
  namespace: myproject
  labels:
    environment: dev
spec:
  containers:
    - name: server
      image: nginx:1.13-alpine
      ports:
        - containerPort: 80
          protocol: TCP
    - name: side-car
      image: alpine:latest
      command: ["/usr/bin/tail", "-f", "/dev/null"]
  restartPolicy: Never
EOF
```
Create the pod by specifying the manifest:
```
oc create -f pod-multi-container.yaml
```
View the detail for the pod and look at the events:
```
oc describe pod my-two-container-pod
```
Let's first execute a shell session inside the server container by using the -c flag:
```
oc exec -it my-two-container-pod -c server -- /bin/sh
```
Run some commands inside the server container:
```
ip address
netstat -ntlp
hostname
ps
exit
```
Let's now execute a shell session inside the side-car container:
```
oc exec -it my-two-container-pod -c side-car -- /bin/sh
```
Run the same commands in side-car container. Each container within a pod runs it's own cgroup, but shares IPC, Network, and UTC (hostname) namespaces:
```
ip address
netstat -ntlp
hostname
exit
```

