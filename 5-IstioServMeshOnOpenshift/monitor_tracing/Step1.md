# 第一步
Create the Example project <br>
建立一个样例项目

###### 快捷链接
[返回场景综述](../deploy_microservices.md) <br>
[下一步](Step2.md) <br>

#### 操作过程
[概念解释](#概念解释) <br>
[创建项目](#创建项目) <br>
[赋予privileged权限](#赋予privileged权限) <br>

### 概念解释
在这个示例中会部署三个微服务。 (customer, preference, recommendation)  这三个微服务程序基于 Spring Boot 和 Vert.x. 实现
这三个微服务的逻辑为，首先当外部请求 customer 微服务时，customer 会产生请求至 perference ，而 perference 则会基于请求向 recommendation 发起请求。

### 创建项目
要部署微服务首先要建立一个项目作为所有微服务的整体组织框架。
> 这一点 OpenShfit 与 Kubernetes 是不一样的，OpenShift 针对 kubernetes 只有 namespace 不方便对应用之间的逻辑进行组织设计，而且也无法完全体现出多租户的逻辑来，甚至无法基于租户实现具体的租户资源隔离这一问题。OpenShift 设计了 Project 这个基础结构，在 Project 下租户可以有效的分配自己的资源，把基于业务相关的应用组织在一起。一个用户可以有若干个 Projects ，每个 Project 下可以包含相对紧密的一组业务逻辑。每个 Project 也可以指定相对独立的资源限定或资源配额。
> - 这里稍微赘述一点，根据我个人的经验，在中国的众多项目中，大部分平台，仅有 Project 也不足以满足真实项目的需求。因为在中国的真实大型 PaaS 项目中，通常使用者会分为业务维护组、业务代维组、业务开发者（外部开发商）、平台运维组、平台代维组、平台支撑组（外部集成商）等等众多身份。同时还会涉及到承担不同项目责任的从属于不同法人主体的参与单位或小组。并且会涉及到项目参与人员在众多项目中交叉参与的复杂情况。例如，某个独立项目的 PM 恰好是另一个项目的项目参与人。这样，当这个人使用平台时，为了合规与便于管理他需要始终使用他个人的平台登陆身份。但同时，当他参与到不同项目中时，他所面临的配额、权限、项目群都不一样。在这种情况下，通常我们需要将 OpenShift 中的 User、 ServiceAccount、Project 结合他的 SCC 权限进行复杂组合，这样才能满足真实业务场景需求。 甚至有时候我们可以以此为基础在增加一个虚拟的总体租户管理平台，租户本身是虚拟的概念，但其配额及权限可以根据其登陆平台时的身份进行二次鉴权，这样每个登陆平台的身份和其它身份之间的资源或项目资源都可以完全隔离，能够更好适应中国大型 PaaS 平台的实际业务及安全的需求。

```
oc new-project tutorial
```
### 赋予privileged权限
拥有了 tutorial 这个项目后，需要对这个项目进行一定的特殊赋权

```
oc adm policy add-scc-to-user privileged -z default -n tutorial
```
> 我们这里是将 default 这个 namespace 的 privileged 这个特殊权限交给 tutorial 这个项目，这样能够使 tutorial 项目的参与者都能够使用 default namespace 下的所有资源，这里其实主要是为了将 default 下的 docker registry 组件权限交给 tutorial 
```
$ oc adm policy add-scc-to-user privileged -z default -n tutorial
scc "privileged" added to: ["system:serviceaccount:tutorial:default"]
$
$ oc get scc privileged
NAME         PRIV      CAPS      SELINUX    RUNASUSER   FSGROUP    SUPGROUP   PRIORITY   READONLYROOTFS   VOLUMES
privileged   true      [*]       RunAsAny   RunAsAny    RunAsAny   RunAsAny   <none>     false   [*]
$
$ oc get all -n default
NAME                           READY     STATUS    RESTARTS   AGE
pod/docker-registry-1-dtw5r    1/1       Running   0          3h
pod/registry-console-1-jt4xp   1/1       Running   0          3h
pod/router-1-zzfpm             1/1       Running   0          3h

NAME                                       DESIRED   CURRENT   READY     AGE
replicationcontroller/docker-registry-1    1         1         1         3h
replicationcontroller/registry-console-1   1         1         1         3h
replicationcontroller/router-1             1         1         1         3h

NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                   AGE
service/docker-registry    ClusterIP   172.30.184.35   <none>        5000/TCP                  3h
service/kubernetes         ClusterIP   172.30.0.1      <none>        443/TCP,53/UDP,53/TCP     3h
service/registry-console   ClusterIP   172.30.14.88    <none>        9000/TCP                  3h
service/router             ClusterIP   172.30.88.202   <none>        80/TCP,443/TCP,1936/TCP   3h

NAME                                                  REVISION   DESIRED   CURRENT   TRIGGERED BY
deploymentconfig.apps.openshift.io/docker-registry    1          1         1         config
deploymentconfig.apps.openshift.io/registry-console   1          1         1         config
deploymentconfig.apps.openshift.io/router             1          1         1         config

NAME                                        HOST/PORT                PATH      SERVICES           PORT      TERMINATION   WILDCARD
route.route.openshift.io/docker-registry    docker-registry-default.2886795284-80-kitek03.environments.katacoda.com              docker-registry    <all>     passthrough   None
route.route.openshift.io/registry-console   registry-console-default.2886795284-80-kitek03.environments.katacoda.com             registry-console   <all>     passthrough   None
$
```


[回到顶部](#第一步)
[下一步骤](Step2.md)