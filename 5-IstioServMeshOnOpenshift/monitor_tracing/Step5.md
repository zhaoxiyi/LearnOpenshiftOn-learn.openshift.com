# 第五步
发布 Recommendation 微服务
Deploy Recommendation microservice

###### 快捷链接
[场景综述](../istio_intro.md) <br>
[上一步](Step4.md) <br>
[下一练习](../) <br>

#### 操作过程
[编译代码](#编译代码) <br>
[构建镜像](#构建镜像) <br>
[注入边车proxy](#注入边车proxy) <br>
[验证微服务](#验证微服务) <br>
 
### 编译代码
进入到 perference 的源码目录,这个目录是在[第二步](stop2.md)中下载下来的。perference 也是基于 vertx 构建的。
```
cd ~/projects/istio-tutorial/recommendation/java/vertx
```
按照上一个步骤中的过程,使用 `mvn package` 编译出 recommendations.jar 文件。

### 构建镜像
同样按照上一个步骤中的过程建立 preference 的 docker image。
本场景中已经提供了用于构建镜像的 Dockerfile 描述文件。
`/recommendation/java/vertx/Dockerfile` 

```
build -t example/recommendation:v1 
```
有了镜像以后我们可以通过 `docker images` 命令来查看镜像，如下。镜像的名字会是 `example/recommendation:v1` ,这里面的 "v1" 代表着 preference 的第一个版本，在后面的练习中会练习到切换版本，因此每个镜像必需有独立的版本号，用于后续练习微服务分流等场景使用。
```
docker images | grep preference
```

### 注入边车proxy
注入 sidecar 代理。
目前 Istio 采用“手动”方式注入 Envoy sidecar。为了这种

```
oc apply -f <(istioctl kube-inject -f ../../kubernetes/Deployment.yml) -n tutorial
```
同样需要建立对应的 Service
```
oc create -f ../../kubernetes/Service.yml
```

部署这个 sidecar 以后还要在部署相应的 Service: 
```
oc create -f ../../kubernetes/Service.yml
```
完成服务部署后还是用下面命令来观察是否部署完成。
```
oc get pods -w
```
一旦所有的 pod 的 `READY` 状态列变为 `2/2`, 就可以使用 `CTRL+C` 来结束观察

### 验证微服务
还是通过 curl 命令访问 customer 服务来验证 preference 的部署成果 
```
curl http://customer-tutorial.2886795342-80-cykoria05.environments.katacoda.com
```

这次调用途径将返回到 Preferences ，然后返回  missing recommendations service 的 error message  。
```
customer => preference => recommendation v1 from '2039379827-rjnqj': 1
```
这表明 recommendation microservice 微服务已经部署完成。

[回到顶部](#第五步) <br>
[上一步骤](Step4.md) <br>
[下一练习]() <br>