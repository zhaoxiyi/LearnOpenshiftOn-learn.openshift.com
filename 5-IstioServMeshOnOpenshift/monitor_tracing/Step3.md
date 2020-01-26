# 第三步
将 Custmer 部署为微服务
###### 快捷链接
[场景综述](../deploy_microservices.md) <br>
[上一步](Step2.md) <br>
[下一步](Step4.md) <br>

#### 操作过程
[发布Customer微服务](#发布Customer微服务) <br>
[执行mvn](#执行mvn) <br>
[打包容器镜像](#打包容器镜像) <br>
[注入边车proxy](#注入边车proxy) <br>
[创建Route供外部访问](#创建Route供外部访问) <br>
[验证微服务](#验证微服务) <br>


### 发布Customer微服务 
Deploy Customer microservice
从上一个步骤中下载的的 git 目录中可以找到 customer 的 java 代码，他是基于 springboot 实现的。
```
cd ~/projects/istio-tutorial/customer/java/springboot
```

### 执行mvn
然后执行 mvn 来打包成 customer.jar 
```
mvn package
```
mvn 会帮助用户编译并构建完整的 java 包。

### 打包容器镜像
完成 mvn 过程后，使用 `/customer/java/springboot/Dockerfile` 可以构建一个 docker image.

编译后的镜像会命名为 example/customer. 使用以下命令来实现： 
```
docker build -t example/customer .
```
You can check the image that was create by typing
```
 docker images | grep customer
```

### 注入边车proxy
注入 sidecar 代理。
目前 Istio 采用“手动”方式注入 Envoy sidecar。为了这种
检查istioctl的版本。执行istioctl版本。
另外，请确保使用的是tutorial project。执行 `oc project tutorial` 来确保转入 tutorial 项目。

现在让我们用它的 sidecar 部署客户舱。
```
oc apply -f <(istioctl kube-inject -f ../../kubernetes/Deployment.yml) -n tutorial
```
部署这个 sidecar 以后还要在部署相应的 Service: 
```
oc create -f ../../kubernetes/Service.yml
```

### 创建Route供外部访问
由于 customer 微服务主要是一个向后传递请求的服务 (customer -> preference -> recommendation)，因此我们需要加入 OpenShift Route 来将服务的端点 endpoint 发布给外部使用.
```
oc expose service customer.
```
检查 route: 
```
oc get route
```
检测 pods 的建立进度, 执行以下命令来持续观看 pods 当前状态。 
```
oc get pods -w
```
一旦所有的 pod 的 `READY` 状态列变为 `2/2`, 就可以使用 `CTRL+C` 来结束观察

### 验证微服务
通过 curl 命令可以验证微服务是否按照我们预期的情况所提供服务 
```
curl http://customer-tutorial.2886795342-80-cykoria05.environments.katacoda.com
```
> curl 命令的一些简单技巧可以从 [KubernetesAPI 介绍那一节](https://github.com/zhaoxiyi/LearnOpenshiftOn-learn.openshift.com/blob/master/1-FoundationsOfOpenshift/APIFund/Step2.md#列出指定范围所有openAPI)中的一些练习中学习。
通过命令可以观察到以下 error 信息，因为微服务 `preference` 和 `recommendation` 目前还没有发布.
```
customer => I/O error on GET request for "http://preference:8080": preference; nested exception is java.net.UnknownHostException: preference
```
上述操作可以验证 customer 微服务已经发布成功。

[回到顶部](#第三步) <br>
[上一步骤](Step2.md) <br>
[下一步骤](Step4.md) <br>