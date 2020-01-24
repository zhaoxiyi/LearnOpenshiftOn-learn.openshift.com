# 第一步 Step2
###### 快捷链接
[场景综述](../istio_intro.md) <br>
[上一步](Step3.md) <br>
[下一步](Step5.md) <br>

#### 操作过程
[查看版本](#查看版本) <br>
[设定指定API反馈深度](#设定指定API反馈深度) <br>
[建立一个API代理点](#建立一个API代理点) <br>
   - 可选步骤 <br>
      [列出指定范围所有openAPI](#列出指定范围所有openAPI) <br>
      [使用jQuery语法](#使用jQuery语法) <br>
      [其它常用操作](#其它常用操作) <br>

### 查看版本
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
Open up another terminal by clicking the + button and select Open New Terminal. 

Send a GET request to the Kubernetes API using curl:
```
curl -X GET http://localhost:8001
```
### 列出指定范围所有openAPI
We can explore the OpenAPI definition file to see complete API details.
```
curl localhost:8001/openapi/v2
```
Send a GET request to list all pods in the environment:
```
curl -X GET http://localhost:8001/api/v1/pods
```
### 使用jQuery语法
Use jq to parse the json response:
```
curl -X GET http://localhost:8001/api/v1/pods | jq .items[].metadata.name
```
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
Export the manifest associated with my-two-container-pod in json format:
```
oc get pods my-two-container-pod --export -o json > podmanifest.json
```
Within the manifest, replace the 1.13 version of alpine with 1.14:
```
sed -i 's|nginx:1.13-alpine|nginx:1.14-alpine|g' podmanifest.json
```
Update/Replace the current pod manifest with the newly updated manifest:
```
curl -X PUT http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod -H "Content-type: application/json" -d @podmanifest.json
```
[4] 为 Pod 更新镜像版本
Patch the current pod with a newer container image (1.15):
```
curl -X PATCH http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod -H "Content-type: application/strategic-merge-patch+json" -d '{"spec":{"containers":[{"name": "server","image":"nginx:1.15-alpine"}]}}'
```
[5] 删除 Pod
Delete the current pod by sending the DELETE request method:
```
curl -X DELETE http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod
```
Verify the pod is in Terminating status:
```
oc get pods
```
Verify the pod no longer exists:
```
curl -X GET http://localhost:8001/api/v1/namespaces/myproject/pods/my-two-container-pod
```


[回到顶部](#第四步) <br>
[上一步骤](Step3.md) <br>
[下一步骤](Step5.md) <br>