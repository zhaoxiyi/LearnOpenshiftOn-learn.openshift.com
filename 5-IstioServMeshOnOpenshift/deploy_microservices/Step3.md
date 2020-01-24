# 第三步
将 Custmer 部署为微服务
###### 快捷链接
[场景综述](../istio_intro.md) <br>
[上一步](Step2.md) <br>
[下一步](Step4.md) <br>

#### 操作过程
[查看版本](#查看版本) <br>
[设定指定API反馈深度](#设定指定API反馈深度) <br>
[建立一个API代理点](#建立一个API代理点) <br>
   - 可选步骤 <br>
      [列出指定范围所有openAPI](#列出指定范围所有openAPI) <br>
      [使用jQuery语法](#使用jQuery语法) <br>
      [其它常用操作](#其它常用操作) <br>

Go to the source folder of customer microservice.

Execute: cd ~/projects/istio-tutorial/customer/java/springboot

Now execute mvn package to create the customer.jar file.

Create the customer docker image.
We will now use the provided /customer/java/springboot/Dockerfile to create a docker image.

This image will be called example/customer.

To build a docker image type: docker build -t example/customer .

You can check the image that was create by typing docker images | grep customer

Injecting the sidecar proxy.
Currently using the "manual" way of injecting the Envoy sidecar.

Check the version of istioctl. Execute istioctl version.

Also, make sure that you are using tutorial project. Execute oc project tutorial

Now let's deploy the customer pod with its sidecar.

Execute: oc apply -f <(istioctl kube-inject -f ../../kubernetes/Deployment.yml) -n tutorial

Also create a service: oc create -f ../../kubernetes/Service.yml

Since customer is the forward most microservice (customer -> preference -> recommendation), let's add an OpenShift Route that exposes that endpoint.

Execute: oc expose service customer.

Check the route: oc get route

To watch the creation of the pods, execute oc get pods -w

Once that the customer pod READY column is 2/2, you can hit CTRL+C.

Try the microservice by typing curl http://customer-tutorial.2886795287-80-host18nc.environments.katacoda.com

You should see the following error because the services preference and recommendation are not yet deployed.

customer => I/O error on GET request for "http://preference:8080": preference; nested exception is java.net.UnknownHostException: preference

This concludes the deployment of the customer microservice.

### 查看版本
istioctl kubeinject 究竟做了什么

<table><td>
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: customer
    version: v1
  name: customer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: customer
      version: v1
  template:
    metadata:
      labels:
        app: customer
        version: v1
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - env:
        - name: JAVA_OPTIONS
          value: -Xms15m -Xmx15m -Xmn15m
        name: customer
        image: quay.io/rhdevelopers/istio-tutorial-customer:v1.1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        - containerPort: 8778
          name: jolokia
          protocol: TCP
        - containerPort: 9779
          name: prometheus
          protocol: TCP
        resources:
          requests: 
            memory: "20Mi" 
            cpu: "200m" # 1/5 core
          limits:
            memory: "40Mi"
            cpu: "500m" 
        livenessProbe:
          exec:
            command:
            - curl
            - localhost:8080/health/live
          initialDelaySeconds: 5
          periodSeconds: 4
          timeoutSeconds: 1
        readinessProbe:
          exec:
            command:
            - curl
            - localhost:8080/health/ready
          initialDelaySeconds: 6
          periodSeconds: 5
          timeoutSeconds: 1
        securityContext:
          privileged: false
```
</td><td>
```
$ istioctl kube-inject -f ../../kubernetes/Deployment.yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: customer
    version: v1
  name: customer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: customer
      version: v1
  strategy: {}
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "true"
        sidecar.istio.io/status: '{"version":"50128f63e7b050c58e1cdce95b577358054109ad2aff4bc4995158c06924a43b","initContainers":["istio-init"],"containers":["istio-proxy"],"volumes":["istio-envoy","istio-certs"],"imagePullSecrets":null}'
      creationTimestamp: null
      labels:
        app: customer
        version: v1
    spec:
      containers:
      - env:
        - name: JAVA_OPTIONS
          value: -Xms15m -Xmx15m -Xmn15m
        image: quay.io/rhdevelopers/istio-tutorial-customer:v1.1
        imagePullPolicy: IfNotPresent
        livenessProbe:
          exec:
            command:
            - curl
            - localhost:8080/health/live
          initialDelaySeconds: 5
          periodSeconds: 4
          timeoutSeconds: 1
        name: customer
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        - containerPort: 8778
          name: jolokia
          protocol: TCP
        - containerPort: 9779
          name: prometheus
          protocol: TCP
        readinessProbe:
          exec:
            command:
            - curl
            - localhost:8080/health/ready
          initialDelaySeconds: 6
          periodSeconds: 5
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 500m
            memory: 40Mi
          requests:
            cpu: 200m
            memory: 20Mi
        securityContext:
          privileged: false
      - args:
        - proxy
        - sidecar
        - --configPath
        - /etc/istio/proxy
        - --binaryPath
        - /usr/local/bin/envoy
        - --serviceCluster
        - customer
        - --drainDuration
        - 45s
        - --parentShutdownDuration
        - 1m0s
        - --discoveryAddress
        - istio-pilot.istio-system:15007
        - --discoveryRefreshDelay
        - 1s
        - --zipkinAddress
        - zipkin.istio-system:9411
        - --connectTimeout
        - 10s
        - --proxyAdminPort
        - "15000"
        - --controlPlaneAuthPolicy
        - NONE
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: INSTANCE_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: ISTIO_META_POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: ISTIO_META_INTERCEPTION_MODE
          value: REDIRECT
        - name: ISTIO_METAJSON_ANNOTATIONS
          value: |
            {"sidecar.istio.io/inject":"true"}
        - name: ISTIO_METAJSON_LABELS
          value: |
            {"app":"customer","version":"v1"}
        image: docker.io/istio/proxyv2:1.0.5
        imagePullPolicy: IfNotPresent
        name: istio-proxy
        ports:
        - containerPort: 15090
          name: http-envoy-prom
          protocol: TCP
        resources:
          requests:
            cpu: 10m
        securityContext:
          readOnlyRootFilesystem: true
          runAsUser: 1337
        volumeMounts:
        - mountPath: /etc/istio/proxy
          name: istio-envoy
        - mountPath: /etc/certs/
          name: istio-certs
          readOnly: true
      initContainers:
      - args:
        - -p
        - "15001"
        - -u
        - "1337"
        - -m
        - REDIRECT
        - -i
        - '*'
        - -x
        - ""
        - -b
        - 8080,8778,9779
        - -d
        - ""
        image: docker.io/istio/proxy_init:1.0.5
        imagePullPolicy: IfNotPresent
        name: istio-init
        resources: {}
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
      volumes:
      - emptyDir:
          medium: Memory
        name: istio-envoy
      - name: istio-certs
        secret:
          optional: true
          secretName: istio.default
status: {}
---
```
</td>
</table>

```
$ istioctl kube-inject -f Deployment.yml 
Error: could not read valid configmap "istio" from namespace "istio-system": Get https://api.cluster-fb7d.fb7d.sandbox509.opentlc.com:6443/api/v1/namespaces/istio-system/configmaps/istio: dial tcp: lookup api.cluster-fb7d.fb7d.sandbox509.opentlc.com on 192.168.31.1:53: no such host - Use --meshConfigFile or re-run kube-inject with `-i <istioSystemNamespace> and ensure valid MeshConfig exists
```
```
$ curl localhost:8001/api/v1/namespaces/istio-system/configmaps/istio
{
  "kind": "ConfigMap",
  "apiVersion": "v1",
  "metadata": {
    "name": "istio",
    "namespace": "istio-system",
    "selfLink": "/api/v1/namespaces/istio-system/configmaps/istio",
    "uid": "120c03e0-351f-11ea-9516-0242ac110017",
    "resourceVersion": "104120",
    "creationTimestamp": "2020-01-12T09:36:55Z",
    "labels": {
      "app": "istio",
      "chart": "istio-1.0.5",
      "heritage": "Tiller",
      "release": "istio"
    },
    "annotations": {
      "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"v1\",\"data\":{\"mesh\":\"# Set the following variable to true to disable policy checks by the Mixer.\\n# Note that metrics will still be reported to the Mixer.\\ndisablePolicyChecks: false\\n\\n# Set enableTracing to false to disable request tracing.\\nenableTracing: true\\n\\n# Set accessLogFile to empty string to disable access log.\\naccessLogFile: \\\"/dev/stdout\\\"\\n#\\n# Deprecated: mixer is using EDS\\nmixerCheckServer: istio-policy.istio-system.svc.cluster.local:9091\\nmixerReportServer: istio-telemetry.istio-system.svc.cluster.local:9091\\n\\n# policyCheckFailOpen allows traffic in cases when the mixer policy service cannot be reached.\\n# Default is false whichmeans the traffic is denied when the client is unable to connect to Mixer.\\npolicyCheckFailOpen: false\\n\\n# Unix Domain Socket through which envoy communicates with NodeAgent SDS to get\\n# key/cert for mTLS. Use secret-mount files instead of SDS if set to empty. \\nsdsUdsPath: \\\"\\\"\\n\\n# How frequently should Envoy fetch key/cert from NodeAgent.\\nsdsRefreshDelay: 15s\\n\\n#\\ndefaultConfig:\\n  #\\n  # TCP connection timeout between Envoy \\u0026 the application, and between Envoys.\\n  connectTimeout: 10s\\n  #\\n  ### ADVANCED SETTINGS #############\\n  # Where should envoy's configuration be stored in the istio-proxy container\\n  configPath: \\\"/etc/istio/proxy\\\"\\n  binaryPath: \\\"/usr/local/bin/envoy\\\"\\n  # The pseudo service name used for Envoy.\\n  serviceCluster: istio-proxy\\n  # These settings that determine how long an old Envoy\\n  # process should be kept alive after an occasional reload.\\n  drainDuration: 45s\\n  parentShutdownDuration: 1m0s\\n  #\\n  # The mode used to redirect inbound connections to Envoy. This setting\\n  # has no effect on outbound traffic: iptables REDIRECT is always used for\\n  # outbound connections.\\n  # If \\\"REDIRECT\\\", use iptables REDIRECT to NAT and redirect to Envoy.\\n  # The \\\"REDIRECT\\\" mode loses source addresses during redirection.\\n  # If \\\"TPROXY\\\", use iptables TPROXY to redirect to Envoy.\\n  # The \\\"TPROXY\\\" mode preserves both the source and destination IP\\n  # addresses and ports, so that they can be used for advanced filtering\\n  # and manipulation.\\n  # The \\\"TPROXY\\\" mode also configures the sidecar to run with the\\n  # CAP_NET_ADMIN capability, which is required to use TPROXY.\\n  #interceptionMode: REDIRECT\\n  #\\n  # Port where Envoy listens (on local host) for admincommands\\n  # You can exec into the istio-proxy container in a pod and\\n  # curl the admin port (curl http://localhost:15000/) to obtain\\n  # diagnostic information from Envoy. See\\n  #https://lyft.github.io/envoy/docs/operations/admin.html\\n  # for more details\\n  proxyAdminPort: 15000\\n  #\\n  # Set concurrency to a specific number to control the number of Proxy worker threads.\\n  # If set to 0 (default), then start worker thread for each CPU thread/core.\\nconcurrency: 0\\n  #\\n  # Zipkin trace collector\\n  zipkinAddress: zipkin.istio-system:9411\\n  #\\n  # Mutual TLS authentication between sidecars and istio control plane.\\n  controlPlaneAuthPolicy: NONE\\n  #\\n  # Address where istio Pilot service is running\\n  discoveryAddress: istio-pilot.istio-system:15007\"},\"kind\":\"ConfigMap\",\"metadata\":{\"annotations\":{},\"labels\":{\"app\":\"istio\",\"chart\":\"istio-1.0.5\",\"heritage\":\"Tiller\",\"release\":\"istio\"},\"name\":\"istio\",\"namespace\":\"istio-system\"}}\n"
    }
  },
  "data": {
    "mesh": "# Set the following variable to true to disable policy checks by the Mixer.\n# Note that metrics will still be reported to the Mixer.\ndisablePolicyChecks: false\n\n# Set enableTracing to false to disable request tracing.\nenableTracing: true\n\n# Set accessLogFile to empty string to disable access log.\naccessLogFile: \"/dev/stdout\"\n#\n# Deprecated: mixer is using EDS\nmixerCheckServer: istio-policy.istio-system.svc.cluster.local:9091\nmixerReportServer:istio-telemetry.istio-system.svc.cluster.local:9091\n\n# policyCheckFailOpen allows traffic incases when the mixer policy service cannot be reached.\n# Default is false which means the traffic is denied when the client is unable to connect to Mixer.\npolicyCheckFailOpen: false\n\n# Unix Domain Socket through which envoy communicates with NodeAgent SDS to get\n# key/cert for mTLS. Use secret-mount files instead of SDS if set to empty. \nsdsUdsPath: \"\"\n\n# How frequently should Envoy fetch key/cert from NodeAgent.\nsdsRefreshDelay: 15s\n\n#\ndefaultConfig:\n  #\n  # TCP connection timeout between Envoy \u0026 the application, and between Envoys.\n  connectTimeout: 10s\n  #\n  ### ADVANCED SETTINGS #############\n  # Where should envoy's configuration be stored in the istio-proxy container\n  configPath: \"/etc/istio/proxy\"\n  binaryPath: \"/usr/local/bin/envoy\"\n  # The pseudo service name used for Envoy.\n  serviceCluster: istio-proxy\n  # These settings that determine how long an old Envoy\n  # process should be kept aliveafter an occasional reload.\n  drainDuration: 45s\n  parentShutdownDuration: 1m0s\n  #\n  # The mode used to redirect inbound connections to Envoy. This setting\n  # has no effect on outbound traffic: iptables REDIRECT is always used for\n  # outbound connections.\n  # If \"REDIRECT\", use iptables REDIRECT to NAT and redirect to Envoy.\n  # The \"REDIRECT\" mode loses source addresses during redirection.\n  # If \"TPROXY\", use iptables TPROXY to redirect to Envoy.\n  # The \"TPROXY\" mode preserves both the source and destination IP\n  # addresses and ports, sothat they can be used for advanced filtering\n  # and manipulation.\n  # The \"TPROXY\" mode also configures the sidecar to run with the\n  # CAP_NET_ADMIN capability, which is required to use TPROXY.\n  #interceptionMode: REDIRECT\n  #\n  # Port where Envoy listens (on local host) for admin commands\n  # You can exec into the istio-proxy container in a pod and\n  # curl the admin port (curl http://localhost:15000/) to obtain\n  # diagnostic information from Envoy. See\n  # https://lyft.github.io/envoy/docs/operations/admin.html\n  # for more details\n  proxyAdminPort: 15000\n  #\n  # Set concurrency to a specific number to control the number of Proxy worker threads.\n  # If set to 0 (default), then start worker thread for each CPU thread/core.\n  concurrency: 0\n  #\n  # Zipkin trace collector\n  zipkinAddress: zipkin.istio-system:9411\n  #\n  # Mutual TLS authentication between sidecars and istio control plane.\n  controlPlaneAuthPolicy: NONE\n  #\n  # Address where istio Pilot service is running\n  discoveryAddress: istio-pilot.istio-system:15007"
  }
  ```


[回到顶部](#第三步) <br>
[上一步骤](Step2.md) <br>
[下一步骤](Step4.md) <br>