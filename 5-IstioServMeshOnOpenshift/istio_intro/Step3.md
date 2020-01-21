# 第三步
将 Custmer 部署为微服务
###### 快捷链接
[场景综述](../istio_intro.md) <br>
[上一步](Step2.md) <br>
[下一步](Step4.md) <br>

#### 操作过程
[Istio 安装](#Istio安装) <br>


#### Istio安装
Istio installation
To install Istio in the cluster, we need first to make sure that we are logged in as an system:admin user.

To log in the OpenShift cluster, type oc login -u system:admin

Now that you are logged in, it's time to extract the existing istio installation: tar -xvzf istio-1.0.5-linux.tar.gz

Before the installation
Istio uses Custom Resources like VirtualServices and DestinationRules.

To allow OpenShift/Kubernetes to understand those values, we first need to install the 'CustomResourceDefinitions' file using the command oc apply -f istio-1.0.5/install/kubernetes/helm/istio/templates/crds.yaml

Continue the installation
Istio provides a file install/kubernetes/istio-demo.yaml that contains the definition of all objects that needs to be created in the Kubernetes cluster.

Let's apply these definitions to the cluster by executing oc apply -f istio-1.0.5/install/kubernetes/istio-demo.yaml

After the execution, Istio objects will be created.

To watch the creation of the pods, execute oc get pods -w -n istio-system

Once that they are all Running, you can hit CTRL+C. This concludes this scenario.

Create external routes
OpenShift uses the concept of Routes to expose HTTP services outside the cluster.

Let's create routes to external services like Grafana, Prometheus, Tracing, etc using the following command:

oc expose svc istio-ingressgateway -n istio-system; \
oc expose svc servicegraph -n istio-system; \
oc expose svc grafana -n istio-system; \
oc expose svc prometheus -n istio-system; \
oc expose svc tracing -n istio-system

Add Istio to the path
Now we need to add istioctl to the path.

Execute export PATH=$PATH:/root/installation/istio-1.0.5/bin/.

Now try it. Check the version of istioctl.

Execute istioctl version.

[回到顶部](#第三步) <br>
[上一步骤](Step2.md) <br>
[下一步骤](Step4.md) <br>