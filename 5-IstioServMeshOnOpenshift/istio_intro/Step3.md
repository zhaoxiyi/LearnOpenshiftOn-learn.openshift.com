# 第三步
将 Custmer 部署为微服务
###### 快捷链接
[返回场景综述](../istio_intro.md) <br>
[上一步](Step2.md) <br>
[下一步](Step4.md) <br>

#### 操作过程
[Istio 安装](#Istio安装) <br>
  [已管理员身份登陆](#已管理员身份登陆) <br>
  [获得Istio安装内容](#获得Istio安装内容) <br>
  [准备OpenShift相关环境](#准备OpenShift相关环境) <br>
  [继续安装](#继续安装) <br>
  [建立外部访问routes](#建立外部访问routes) <br>
  [其它](#其它) <br>



### Istio安装
Istio installation

#### 已管理员身份登陆
安装 Istio 需要有 Cluster 的管理权限，在这个例子里我们需要以 system:admin 这个特殊的内部管理员身份来登陆。
```
$oc login -u system:admin
```
> system:admin 是 Kubernetes 和 OpenShift 都具有的内部用户。这个用户在初始化 Kubernetes 时就会供应。但这个用户时如何较验的呢？<br>
> 在 Kubernetes/OpenShift 供应的时候会自己建立一个 kubeconfig 和 kubeadmin-password 文件。这两个文件kubeconfig 一个为本机描述了 Kubernetes/OpenShift 是如何配置的，同时包含了基础用户的完整密钥（记得如果你使用了 OpenShift 并对接了 LDAP 之类的认证源，这里是不会有任何信息的，它只包含初始配置时的内部认证用户），kubeadmin-password 则记录了 Kubernetes/Openshift system:admin 这个特殊用户的 Tocken。 只要你有了它们，你就可以按照系统内部管理员的身份来登陆了。
> ```
[user@clientvm 0 ~]$ sudo ls /home/user/cluster-82c7
auth  metadata.json  terraform.aws.auto.tfvars	terraform.tfstate  terraform.tfvars  tls
[user@clientvm 0 ~]$ sudo ls /home/ec2-user/cluster-82c7/auth
kubeadmin-password  kubeconfig
[user@clientvm 0 ~]$ sudo cat /home/ec2-user/cluster-82c7/auth/kubeadmin-password
Bec7L-swnFu-JUweV-E4HYY
[user@clientvm 0 ~]$ 
[user@clientvm 0 ~]$ sudo cp -R /home/ec2-user/cluster-${GUID}/auth $HOME/cluster-${GUID}
[user@clientvm 0 ~]$ export KUBECONFIG=${HOME}/cluster-${GUID}/auth/kubeconfig
[user@clientvm 0 ~]$ oc whoami
system:admin
> ```
> 从上面这段操作可以看出，只要你把初始配置的 auth 目录发给某个特定的机器，它就可以作为管理机无密码登陆了，你甚至可以在一个管理机上配置很多 cluster 目录来实现中央管理，但要注意安全，因为只要有了这两个文件，所在的服务器就相当于进了数据库的 DBA Console 了。在这台管理机上什么都可以无密码操作。

#### 获得Istio安装内容
这里需要特殊说明一下，如[场景综述](../istio_intro.md)中所提及的，目前 OpenShift 上的 Istio 安装分为 Operator 安装和社区原生安装，这里使用的是社区1.0.5安装。因此为了安装 istio 你首先要有安装程序，learn.openshift.com 已经帮你准备好了安装程序，这个安装程序是 istio.io 网站上发布的 Istio 标准安装包，你可以 https://istio.io/docs/setup/getting-started/#download 在这里找到最新版本的内容: 
```
$tar -xvzf istio-1.0.5-linux.tar.gz
```
> 查看一下我们刚刚究竟做了什么？
```
$ ls istio-1.0.5
bin  install  istio.VERSION  LICENSE  README.md  samples  tools
```
> 这个 istio 安装包中会包含 `istio/bin/istioctl` 这个 istio 专用命令，也会包含一套标准例子 `istio/samples` . 还包含在 Kubernetes/Openshift 上所需的各种角色/权限/资源等描述文件 `istio/install` 。 以及一些关于更新、 `istio/tools`

#### 准备OpenShift相关环境
在社区版 Istio 安装之前先要准备好相应的一些资源，这些资源包括 VirtualServices 、 DestinationRules 等。在本次学习场景中，为了让使用者更简单的理解 OpenShift/Kubernetes 上的这些值，我们需通过演示准备好的 yaml 文件来快速定制 'CustomResourceDefinitions' 。 
```
oc apply -f istio-1.0.5/install/kubernetes/helm/istio/templates/crds.yaml
```

> 而如果使用 Operator 安装，则上述这些都不再需要了，只需要按顺序安装相应的 Operator 就可以了。这里先剧透一点，Istio 的安装不只是一个 Operator 哦，为了 Istio 的安装更加灵活、便于调试、可选，因此在 Operator 上其实要分别选择 Istio 的几个主要部分。<br>
> crds.yaml 的安装包括了 Istio 所需的所有 'CustomResourceDefinitions' 所以叫做 crd 。这里可以观察一下 Istio 究竟都构建了多少 CRD 
```
$ oc apply -f istio-1.0.5/install/kubernetes/helm/istio/templates/crds.yaml
customresourcedefinition.apiextensions.k8s.io/virtualservices.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/destinationrules.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/policies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/meshpolicies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/bypasses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/circonuses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/deniers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/fluentds.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/kubernetesenvs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/listcheckers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/memquotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/noops.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/opas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/prometheuses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rbacs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/redisquotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/servicecontrols.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/signalfxs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/solarwindses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/stackdrivers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/statsds.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/stdios.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/apikeys.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/authorizations.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/checknothings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/kuberneteses.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/listentries.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/logentries.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/edges.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/metrics.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotas.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/reportnothings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/servicecontrolreports.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/tracespans.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/instances.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/templates.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io created
```
> 从上面这个构建列表里面可以看出，围绕着 Isio 需要建立大量的辅助资源，例如各种权限 “RBAC”， 各种角色 “Role”，各种组件所需的配额 “Quota” ，各种资源的配置 “Config” ，一部分网络设置 “Network”， 一部分授权 “Authentication” 等等。

#### 继续安装
在这个演示中 Istio 提供了一个 yaml 描述文件，`install/kubernetes/istio-demo.yaml` 这个文件中包含了 istio 所需要的所有对象的描述，通过使用这个文件可以在 Kubernetes cluster 直接建立 istio-system 这个 namespace 以及 namespace 下所有对象。

```
oc apply -f istio-1.0.5/install/kubernetes/istio-demo.yaml
```
> 
```
 oc apply -f istio-1.0.5/install/kubernetes/istio-demo.yaml
namespace/istio-system created
configmap/istio-galley-configuration created
configmap/istio-grafana-custom-resources created
configmap/istio-grafana-configuration-dashboards created
configmap/istio-grafana created
configmap/istio-statsd-prom-bridge created
configmap/prometheus created
configmap/istio-security-custom-resources created
configmap/istio created
configmap/istio-sidecar-injector created
serviceaccount/istio-galley-service-account created
serviceaccount/istio-egressgateway-service-account created
serviceaccount/istio-ingressgateway-service-account created
serviceaccount/istio-grafana-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-grafana-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-grafana-post-install-role-binding-istio-system created
job.batch/istio-grafana-post-install created
serviceaccount/istio-mixer-service-account created
serviceaccount/istio-pilot-service-account created
serviceaccount/prometheus created
serviceaccount/istio-cleanup-secrets-service-account created
clusterrole.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
job.batch/istio-cleanup-secrets created
serviceaccount/istio-security-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-security-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-security-post-install-role-binding-istio-system created
job.batch/istio-security-post-install created
serviceaccount/istio-citadel-service-account created
serviceaccount/istio-sidecar-injector-service-account created
customresourcedefinition.apiextensions.k8s.io/virtualservices.networking.istio.io configured
customresourcedefinition.apiextensions.k8s.io/destinationrules.networking.istio.io configured
customresourcedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io configured
customresourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io configured
customresourcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io configured
customresourcedefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/bypasses.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/circonuses.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/deniers.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/fluentds.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/kubernetesenvs.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/listcheckers.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/memquotas.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/noops.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/opas.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/prometheuses.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/rbacs.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/redisquotas.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/servicecontrols.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/signalfxs.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/solarwindses.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/stackdrivers.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/statsds.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/stdios.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/apikeys.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/authorizations.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/checknothings.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/kuberneteses.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/listentries.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/logentries.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/edges.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/metrics.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/quotas.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/reportnothings.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/servicecontrolreports.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/tracespans.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io configured
customresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io configured
customresourcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io configured
customresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/instances.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/templates.config.istio.io configured
customresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io configured
clusterrole.rbac.authorization.k8s.io/istio-galley-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-egressgateway-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-ingressgateway-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-mixer-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrole.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-sidecar-injector-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-galley-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-egressgateway-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-ingressgateway-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-mixer-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-sidecar-injector-admin-role-binding-istio-system created
service/istio-galley created
service/istio-egressgateway created
service/istio-ingressgateway created
service/grafana created
service/istio-policy created
service/istio-telemetry created
service/istio-pilot created
service/prometheus created
service/istio-citadel created
service/servicegraph created
service/istio-sidecar-injector created
deployment.extensions/istio-galley created
deployment.extensions/istio-egressgateway created
deployment.extensions/istio-ingressgateway created
deployment.extensions/grafana created
deployment.extensions/istio-policy created
deployment.extensions/istio-telemetry created
deployment.extensions/istio-pilot created
deployment.extensions/prometheus created
deployment.extensions/istio-citadel created
deployment.extensions/servicegraph created
deployment.extensions/istio-sidecar-injector created
deployment.extensions/istio-tracing created
gateway.networking.istio.io/istio-autogenerated-k8s-ingress created
horizontalpodautoscaler.autoscaling/istio-egressgateway created
horizontalpodautoscaler.autoscaling/istio-ingressgateway created
horizontalpodautoscaler.autoscaling/istio-policy created
horizontalpodautoscaler.autoscaling/istio-telemetry created
horizontalpodautoscaler.autoscaling/istio-pilot created
service/jaeger-query created
service/jaeger-collector created
service/jaeger-agent created
service/zipkin created
service/tracing created
mutatingwebhookconfiguration.admissionregistration.k8s.io/istio-sidecar-injector created
attributemanifest.config.istio.io/istioproxy created
attributemanifest.config.istio.io/kubernetes created
stdio.config.istio.io/handler created
logentry.config.istio.io/accesslog created
logentry.config.istio.io/tcpaccesslog created
rule.config.istio.io/stdio created
rule.config.istio.io/stdiotcp created
metric.config.istio.io/requestcount created
metric.config.istio.io/requestduration created
metric.config.istio.io/requestsize created
metric.config.istio.io/responsesize created
metric.config.istio.io/tcpbytesent created
metric.config.istio.io/tcpbytereceived created
prometheus.config.istio.io/handler created
rule.config.istio.io/promhttp created
rule.config.istio.io/promtcp created
kubernetesenv.config.istio.io/handler created
rule.config.istio.io/kubeattrgenrulerule created
rule.config.istio.io/tcpkubeattrgenrulerule created
kubernetes.config.istio.io/attributes created
destinationrule.networking.istio.io/istio-policy created
destinationrule.networking.istio.io/istio-telemetry created
$
```
> 从上面这个例子来看，可以基本了解 Istio 究竟包含哪些组件和配置等等，可以帮助大家理解 Istio 究竟如何构成的。

执行这个命令后 OpenShift 会陆续开始建立 Istio 和其相关的对象，想要观察当前构建的情况可以执行下述命令
```
oc get pods -w -n istio-system
``` 
一旦发现所有的 Pod 都是 `Running` 状态就可以使用 `CTRL+C` 来停止这个监视命令
> oc get pods `-w` 

#### 建立外部访问routes
Create external routes
当所有服务都具备服务能力后，就可以允许外部访问对应的服务 Service 。 这些 Service 需要通过 `expose` 命令来发布到外部。 OpenShift 会将相应的 `Routes` 发布给外部的 HTTP 服务，使外部访问可以通过 HTTP 协议访问集群内资源。
```
oc expose svc istio-ingressgateway -n istio-system; \
oc expose svc servicegraph -n istio-system; \
oc expose svc grafana -n istio-system; \
oc expose svc prometheus -n istio-system; \
oc expose svc tracing -n istio-system
```
> 以上命令会将 istio-system 下的几个需要供外部访问的服务发布出去，并且会为对应的服务创建相应的 `Routes` 。
```
$ oc get routes -n istio-system
NAME                   HOST/PORT      PATH      SERVICES               PORT              TERMINATION   WILDCARD
grafana                grafana-istio-system.2886795325-80-cykoria05.environments.katacoda.com                grafana                http                            None
istio-ingressgateway   istio-ingressgateway-istio-system.2886795325-80-cykoria05.environments.katacoda.com             istio-ingressgateway   http2                           None
prometheus             prometheus-istio-system.2886795325-80-cykoria05.environments.katacoda.com                prometheus             http-prometheus                 None
servicegraph           servicegraph-istio-system.2886795325-80-cykoria05.environments.katacoda.com                servicegraph           http                            None
tracing                tracing-istio-system.2886795325-80-cykoria05.environments.katacoda.com                tracing                http-query                      None
```
> 用上面这个命令可以看到系统为 istio 发布出来的可用外部接口，包括了用于管理、监控、跟踪用的地址。

#### 将istioctl命令加入系统路径
Add Istio to the path
在使用 Istio 的过程中，有很多操作可以借助 istioctl 命令来进行简化操作，包括一些对服务网格的注入、移除、流量管控的描述更改等都可以通过这个命令简化对 yaml 的更改和操作等。
```
export PATH=$PATH:/root/installation/istio-1.0.5/bin/.
```
使用以下命令来查看目前使用的 istioctl 命令的版本
```
 istioctl version.
```

#### 其它
上文中列出了使用社区版 Istio 安装后创建的各类对象与规则等。社区版与 Redhat 提供的 Operator 版所构建的对象与目标基本是一致的，只是借助 Operator 安装的 Redhat 版 Istio 会将一些内部组件或是数据存储等组件的耦合相应的进行认证与规范，从而使整体 Istio 的使用过程更便于标准化操作并便于支持。从下面这个输出可以看出 Redhat Operator 将相应的构建过程封装为很简单的几个 CRD/Role/ServiceAccount/Servie/Deployment 进行快速部署，从而简化使用者所需理解的内容。特别是如果内部出现一些特殊异常，使用者将不再需要手工调试，而是回退 Operator 然后重新使用 Operator 整体操作即可。如遇到核心问题，则直接从 Redhat 客服来获取帮助即可。这就是 Operator 的核心存在目的。
```
customresourcedefinition.apiextensions.k8s.io "servicemeshcontrolplanes.maistra.io" created
customresourcedefinition.apiextensions.k8s.io "servicemeshmemberrolls.maistra.io" created
clusterrole.rbac.authorization.k8s.io "maistra-admin" created
clusterrolebinding.rbac.authorization.k8s.io "maistra-admin" created
clusterrole.rbac.authorization.k8s.io "istio-operator" created
serviceaccount "istio-operator" created
clusterrolebinding.rbac.authorization.k8s.io "istio-operator-account-istio-operator-cluster-role-binding" created
service "admission-controller" created
deployment.apps "istio-operator" created
```

[回到顶部](#第三步) <br>
[上一步骤](Step2.md) <br>
[下一场景](../deploy_microservices.md) <br>