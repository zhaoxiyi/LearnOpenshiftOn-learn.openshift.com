# 第二步
###### 快捷链接
[场景综述](../istio_intro.md) <br>
[上一步](Step1.md) <br>
[下一步](Step3.md) <br>

#### 操作过程
[查看版本](#查看版本) <br>
[设定指定API反馈深度](#设定指定API反馈深度) <br>
[建立一个API代理点](#建立一个API代理点) <br>
   - 可选步骤 <br>
      [列出指定范围所有openAPI](#列出指定范围所有openAPI) <br>
      [使用jQuery语法](#使用jQuery语法) <br>
      [其它常用操作](#其它常用操作) <br>

### 查看版本

Istio服务网格在逻辑上分为数据平面和控制平面。

数据平面由部署为边车的一组智能代理（Envoy）组成。这些代理与通用策略和遥测集线器Mixer一起协调和控制微服务之间的所有网络通信。

控制平面管理和配置代理以路由流量。此外，控制平面将混合器配置为执行策略并收集遥测。

下图显示了组成每个平面的不同组件：

Istio组件
使者
Envoy是使用C ++开发的高性能代理，可为服务网格中的所有服务调解所有入站和出站流量。 Istio利用Envoy的许多内置功能，例如：

动态服务发现
负载均衡
TLS终止
HTTP/2 和 gRPC 代理
断路器
健康检查
分阶段推出，按百分比分配流量
故障注入
丰富的指标
Envoy被部署为同一Kubernetes容器中相关服务的辅助工具。此部署使Istio可以提取有关流量行为的大量信号作为属性。 Istio可以依次在Mixer中使用这些属性来实施策略决策，并将其发送给监视系统以提供有关整个网格行为的信息。

混合器
Mixer是独立于平台的组件。 Mixer跨服务网格实施访问控制和使用策略，并从Envoy代理和其他服务收集遥测数据。代理提取请求级别属性，并将其发送到Mixer进行评估。

Mixer包括灵活的插件模型。该模型使Istio可以与各种主机环境和基础架构后端进行交互。因此，Istio从这些细节中提取了Envoy代理和Istio管理的服务。

飞行员
Pilot提供了Envoy边车的服务发现，智能路由的流量管理功能（例如A / B测试，金丝雀部署等）以及弹性（超时，重试，断路器等）。

飞行员将控制交通行为的高级路由规则转换为Envoy特定的配置，并在运行时将其传播到边车。 Pilot提取了特定于平台的服务发现机制，并将它们合成为标准格式，任何符合Envoy数据平面API的Sidecar都可以使用。这种松散的耦合使Istio可以在Kubernetes，Consul或Nomad等多种环境中运行，同时保持用于流量管理的相同操作员界面。

堡垒
Citadel通过内置的身份和凭证管理功能提供了强大的服务到服务和最终用户身份验证。您可以使用Citadel升级服务网格中的未加密流量。使用Citadel，运营商可以根据服务身份而非网络控制来实施策略。

An Istio service mesh is logically split into a data plane and a control plane.

The data plane is composed of a set of intelligent proxies (Envoy) deployed as sidecars. These proxies mediate and control all network communication between microservices along with Mixer, a general-purpose policy and telemetry hub.

The control plane manages and configures the proxies to route traffic. Additionally, the control plane configures Mixers to enforce policies and collect telemetry.

The following diagram shows the different components that make up each plane:



Istio Components
Envoy
Envoy is a high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh. Istio leverages Envoy’s many built-in features, for example:

Dynamic service discovery
Load balancing
TLS termination
HTTP/2 and gRPC proxies
Circuit breakers
Health checks
Staged rollouts with %-based traffic split
Fault injection
Rich metrics
Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod. This deployment allows Istio to extract a wealth of signals about traffic behavior as attributes. Istio can, in turn, use these attributes in Mixer to enforce policy decisions, and send them to monitoring systems to provide information about the behavior of the entire mesh.

Mixer
Mixer is a platform-independent component. Mixer enforces access control and usage policies across the service mesh, and collects telemetry data from the Envoy proxy and other services. The proxy extracts request level attributes, and sends them to Mixer for evaluation.

Mixer includes a flexible plugin model. This model enables Istio to interface with a variety of host environments and infrastructure backends. Thus, Istio abstracts the Envoy proxy and Istio-managed services from these details.

Pilot
Pilot provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing (e.g., A/B tests, canary deployments, etc.), and resiliency (timeouts, retries, circuit breakers, etc.).

Pilot converts high level routing rules that control traffic behavior into Envoy-specific configurations, and propagates them to the sidecars at runtime. Pilot abstracts platform-specific service discovery mechanisms and synthesizes them into a standard format that any sidecar conforming with the Envoy data plane APIs can consume. This loose coupling allows Istio to run on multiple environments such as Kubernetes, Consul, or Nomad, while maintaining the same operator interface for traffic management.

Citadel
Citadel provides strong service-to-service and end-user authentication with built-in identity and credential management. You can use Citadel to upgrade unencrypted traffic in the service mesh. Using Citadel, operators can enforce policies based on service identity rather than on network controls.

查看 OpenShift(Kubernetes) 版本
Verify the currently available Kubernetes API versions:
```
oc api-versions
```

### 设定指定API反馈深度
使用 '--v' 参数可以指定API反馈的深度。关于这部分内容你可以去查看 OpenShift blog 来了解更多的用法
https://blog.openshift.com/oc-command-newbies/
Blog 中用的是 -loglevel ，亲自用一下就会发现这两者是完全一样的，只是不同写法而已。
Use the --v flag to set a verbosity level. This will allow you to see the request/responses against the Kubernetes API:
```
oc get pods --v=8
```
在本场景中只有一个 Pod 'my-two-container-pod' 显示这一个 Pod 会反馈出如下内容来：
```
$ oc get pods --v=8
I0104 13:33:31.356806    2769 loader.go:359] Config loaded from file /root/.kube/config
I0104 13:33:31.358072    2769 loader.go:359] Config loaded from file /root/.kube/config
I0104 13:33:31.363297    2769 loader.go:359] Config loaded from file /root/.kube/config
I0104 13:33:31.372226    2769 loader.go:359] Config loaded from file /root/.kube/config
I0104 13:33:31.372560    2769 round_trippers.go:415] GET https://master:8443/api/v1/namespaces/myproject/pods?limit=500
I0104 13:33:31.372585    2769 round_trippers.go:422] Request Headers:
I0104 13:33:31.372598    2769 round_trippers.go:426]     Accept: application/json;as=Table;v=v1beta1;g=meta.k8s.io, application/json
I0104 13:33:31.372609    2769 round_trippers.go:426]     User-Agent: oc/v1.11.0+d4cacc0 (linux/amd64) kubernetes/d4cacc0
I0104 13:33:31.388750    2769 round_trippers.go:441] Response Status: 200 OK in 16 milliseconds
I0104 13:33:31.389035    2769 round_trippers.go:444] Response Headers:
I0104 13:33:31.389215    2769 round_trippers.go:447]     Content-Length: 3046
I0104 13:33:31.389414    2769 round_trippers.go:447]     Date: Sat, 04 Jan 2020 13:33:31 GMT
I0104 13:33:31.389564    2769 round_trippers.go:447]     Cache-Control: no-store
I0104 13:33:31.389744    2769 round_trippers.go:447]     Content-Type: application/json
I0104 13:33:31.390088    2769 request.go:897] Response Body: {"kind":"Table","apiVersion":"meta.k8s.io/v1beta1","metadata":{"selfLink":"/api/v1/namespaces/myproject/pods","resourceVersion":"36335"},"columnDefinitions":[{"name":"Name","type":"string","format":"name","description":"Name must be unique within a namespace. Is required when creating resources, although some resources may allow a client to requestthe generation of an appropriate name automatically. Name is primarily intended for creation idempotence and configuration definition. Cannot be updated. More info: http://kubernetes.io/docs/user-guide/identifiers#names","priority":0},{"name":"Ready","type":"string","format":"","description":"The aggregate readiness state of this pod for accepting traffic.","priority":0},{"name":"Status","type":"string","format":"","description":"The aggregate status of the containers in this pod.","priority":0},{"name":"Restarts","type":"integer","format":"","description":"The number of times the containers in this pod have been restarted.","priority":0},{"name":"Age","type":"stri [truncated 2022 chars]
I0104 13:33:31.391337    2769 get.go:443] no kind is registered for the type v1beta1.Table in scheme "k8s.io/kubernetes/pkg/api/legacyscheme/scheme.go:29"
NAME                   READY     STATUS    RESTARTS   AGE
my-two-container-pod   2/2       Running   0          1m
$
```
从这个反馈中你可以了解到，oc 开始执行就会加载本地的配置文件，默认在 ‘/root/.kube/config’ 。当然如果你用的是远程 oc 客户端你会发现加载的是本地的一个config目录，
```
I0104 21:42:04.857921   31789 loader.go:357] Config loaded from file /Users/xiyzhao/.kube/config
I0104 21:42:04.868066   31789 round_trippers.go:383] GET https://master.sso-3e67.open.redhat.com:443/api?timeout=32s
I0104 21:42:04.868082   31789 round_trippers.go:390] Request Headers:
I0104 21:42:04.868090   31789 round_trippers.go:393]     Authorization: Bearer g7Y61rbsUlLf4T3QTCesB0gUo9hYEVmM1CB0Us6XYTo
I0104 21:42:04.868097   31789 round_trippers.go:393]     Accept: application/json, */*
I0104 21:42:04.868103   31789 round_trippers.go:393]     User-Agent: oc/v1.10.0+b81c8f8 (darwin/amd64) kubernetes/b81c8f8
I0104 21:42:08.882326   31789 round_trippers.go:408] Response Status:  in 4014 milliseconds
I0104 21:42:08.882362   31789 round_trippers.go:411] Response Headers:
I0104 21:42:08.882434   31789 cached_discovery.go:124] skipped caching discovery info due to Get https://master.sso-3e67.open.redhat.com:443/api?timeout=32s: dial tcp: lookup master.sso-3e67.open.redhat.com on 192.168.3.1:53: no such host
```
上面这个例子里面就可以看到如果远程，它会现加载当前用户下已有的配置文件（如果没有会创建一个并告知还未登录），你的安全信息
也在这里，然后找到指定服务器所在的位置，这个也是在config里指定的，你只要‘oc login’过就会有相应的安全信息，如果没有则指引用户指定服务器并登陆。如果找到服务器，则将 json 请求发给服务器，并获得服务器的反馈。当你不了解 OpenShift/Kubernetes 时，除了任何问题，这个手段都是最有效的理解和排查手段，你可以看到你所使用的所有程序，比如这个例子中就会经过 loader.go:359 行加载文件 -> round_trippers.go:415 询问地址 -> request.go:897 发送 json -> get.go:443 获得反馈 -> 在get.go中获得 scheme.go:29 执行的信息。

### 建立一个API代理点
这一步非常简单通过一个简单的命令将 Kubernetes API 发布到本机端口，供其它程序使用。
Use the oc proxy command to proxy local requests on port 8001 to the Kubernetes API:
```
oc proxy --port=8001
```
这个命令的价值在[本场景的综述](../KubernetesAPIFund.md#场景说明)中已经描述过了，这里就不做更多的解释了。

接下来可以在 Katacoda 里面可以再开一个 Termianl 窗口，模拟从另一个位置来访问这个 proxy。从 proxy 我们可以像使用正常的内部 API Server 一样从外部使用任何 API。谨慎使用，此时你已经通过你客户端将 Kubernetes 所有能力开放给外部了。
Open up another terminal by clicking the + button and select Open New Terminal. 
Send a GET request to the Kubernetes API using curl:
```
curl -X GET http://localhost:8001
```
curl 是一个极好用的命令，他可以对任何 HTTP 或 HTTPS 端点发起测试请求，只要使用正确的参数，合适的命令脚本，就可以完成如 Postman 等其它测试工具才能完成的功能。
### 列出指定范围所有openAPI
用 ’curl‘ 命令来要求 OpenShift 列出所有 openapi 中可以使用的 API。这里需要注意，openapi v2 所包含的详细内容将有5M 多些，如果在国内使用这个命令你将会发现屏幕会不停的输出内容直至 TimeOut 。 所以建议在做着练习的时候改变一个方式
原练习描述：
We can explore the OpenAPI definition file to see complete API details.
```
curl localhost:8001/openapi/v2
```
我们可以改为：
```
curl localhost:8001/openapi/v2 > openapiv2.json
```
然后使用 'more openapiv2.json' 来查看这个文件（用 cat 或是用 Katacoda 自带的文件浏览器都会在显示文件的过程中超时）
接下来可以尝试用一些特定的 API 来获得指定的信息，例如 Pods 的执行信息。
Send a GET request to list all pods in the environment:
```
curl -X GET http://localhost:8001/api/v1/pods
```
如果返回的 json 内容还是太多就可以使用下面介绍的 jQuery 来获得精准的指定信息。
### 使用jQuery语法
jQuery是一个快速、小型且功能丰富的JavaScript库。它使用一个在多种浏览器上工作的易于使用的API，使得HTML文档遍历和操作、事件处理、动画和Ajax等工作变得更加简单。随着多功能性和可扩展性的结合，jQuery改变了数百万人编写JavaScript的方式。
关于 jQuery 的细节信息大家可以到 jQuery 社区去学习
https://jquery.com
jQuery是一个学习成本非常低的小手段，但是它可以在很多场景中派上用场，比如在过滤 json 输出的复杂内容（它也可以用于过滤任何符合 HTML/XML 句法的报文，例如 REST 报文。） 
Use jq to parse the json response:
```
$ curl -X GET http://localhost:8001/api/v1/pods | jq .items[].metadata.name
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  220k    0  220k    0     0  9304k      0 --:--:-- --:--:-- --:--:-- 9591k
"docker-registry-1-9n4f5"
"registry-console-1-xv5vq"
"router-1-qslzg"
"apiserver-7cpq2"
"controller-manager-fdfmk"
"master-api-master"
"master-controllers-master"
"master-etcd-master"
"my-two-container-pod"
"asb-1-deploy"
"console-5677c7c58d-g48pf"
"alertmanager-main-0"
"alertmanager-main-1"
"alertmanager-main-2"
"cluster-monitoring-operator-6465f8fbc7-skz6c"
"grafana-6b9f85786f-2b9x6"
"kube-state-metrics-7449d589bc-6x6vw"
"node-exporter-hvktg"
"prometheus-k8s-0"
"prometheus-k8s-1"
"prometheus-operator-6644b8cd54-rbwmb"
"sync-gtngx"
"ovs-4kg9c"
"sdn-pdw7j"
"apiserver-7lncd"
"webconsole-7df4f9f689-lf7br"
$
```
这个例子里面介绍了将 pods API 返回的 json 报文进行整理，首先将所有条目 '.items[]' ('[]'代表取得所有，如果写为 '[3]') 代表要取得第三个条目 item 的所有内容。 在所有 items 里选择 metadata 段下面的 name 下所输出的值。所以写为 '.metadata.name'。 而通过 '|' 管道符将 API 返回结果交给 jq 及相应的公式来处理，最终返回值只剩下如上列表，它输出了一系列 String 对象，每个对象都是一个 metadata 下的 name。 真实含义是返回了 OpenShift 里目前都有那些 Pods 它们的名字是什么。

### 其它常用操作
[1] 获得所有 Pods 的信息
We can scope the response by only viewing all pods in a particular namespace:
```
curl -X GET http://localhost:8001/api/v1/namespaces/myproject/pods
```
[2] 获得某个特定 Pod 的信息
Get more details on a particular pod within the myproject namespace:
```
curl -X GET http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod
```
[3] 使用 json 格式来接收或更改某项配置（你可以将json再转给其它系统使用）
为了使用 json 格式的 manifist 文件来作为参数输入 API 从而改变 Pods 的配置，我们首先需要先有一个基础。我们可以向现有的 Pod 要一份基础信息。 oc 命令允许我们使用 export 参数来输出一份 manifest ，当然你需要通过 '-o' 参数告知 oc 命令，你需要的是一个 json 格式。
Export the manifest associated with my-two-container-pod in json format:
```
oc get pods my-two-container-pod --export -o json > podmanifest.json
```
有了 podmanifest.json 这个基础，你可以调整任何你希望修改的地方，比如这个例子中是将基础镜像从 1.13 版本改为 1.14 版本。下面命令通过 sed 找到 podmanifest.json 文件中所有nginx:1.13 然后改为 nginx:1.14
Within the manifest, replace the 1.13 version of alpine with 1.14:
```
sed -i 's|nginx:1.13-alpine|nginx:1.14-alpine|g' podmanifest.json
```
有了这个新的 manifest 文件，你就可以使用 pods API 直接修改在线 Pods 而无需关心其它细节，比如应用怎们办，代码怎么办。这一切期都会沿用之前 Pods 部署的设置直接在服务器上重新执行一遍，从而将老的基础镜像换成新的，这种操作就是典型 DevOps 福利，让 Ops 人员无需关注 Dev 即可完成部分原本 Dev 人员才能完成的任务。
Update/Replace the current pod manifest with the newly updated manifest:
```
curl -X PUT http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod -H "Content-type: application/json" -d @podmanifest.json
```
[4] 为 Pod 更新镜像版本
curl 的 PATCH 方法是 PUT 方法的一种补充，它可以只更新部分参数， 我们可以看到在下面这个例子里面，我们只将 'spec' 部分进行了更新，而不需要关心完整的 json 是什么样子的。这样我们就不需要先向 oc 命令去要一个原本 manifest 文件了。我们只要保证 spec 部分的格式没有问题我们就能完成刚刚的更新操作。下面这个例子将基础镜像更新到了 nginx:1.15-alpine 版本。
Patch the current pod with a newer container image (1.15):
```
curl -X PATCH http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod -H "Content-type: application/strategic-merge-patch+json" -d '{"spec":{"containers":[{"name": "server","image":"nginx:1.15-alpine"}]}}'
```
[5] 删除 Pod
删除 Pod 就更加简单了，我们使用 DELETE 方法请求具体的 pods API 中的指定 pod，它就被删除了。
Delete the current pod by sending the DELETE request method:
```
curl -X DELETE http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod
```
验证一下 Pod 是否被删除了。
Verify the pod is in Terminating status:
```
oc get pods
```
再验证一下 API 上是不是也没了。
Verify the pod no longer exists:
```
curl -X GET http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod
```

[6] 在真实 OpenShift 环境中尝试 Poxy 的安全性
此外为了了解您的真实权限，你可以尝试在真实的 'oc' 客户端环境上尝试一下 proxy 的效果，对于那些你的登陆用户没有权限的 API 命令，你会发现，你可以获得如下反馈。
```
(base) $ curl -X GET http://localhost:8001/api/v1/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "pods is forbidden: User \"user1\" cannot list resource \"pods\" in API group \"\" at the cluster scope",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
(base) $ 

```
在这个尝试中，你可以从返回内容中了解到，我使用的 'oc login' 的用户是 'user1' 而希望列出所有 '\' 下的 Pods 这是不在我的权限范围内的。


[回到顶部](#第二步) <br>
[上一步骤](Step1.md) <br>
[下一步骤](Step2.md) <br>