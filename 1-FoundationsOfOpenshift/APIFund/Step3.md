# 第三步
建立 Replica Sets
###### 快捷链接
[场景综述](../KubernetesAPIFund.md) <br>
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

### 查看版本

Get a list of all pods in the myproject Namespace:

oc get pods -n myproject

Create a ReplicaSet object manifest file:

cat > replica-set.yaml <<EOF
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: myfirstreplicaset
  namespace: myproject
spec:
  selector:
    matchLabels:
     app: myfirstapp
  replicas: 3
  template:
    metadata:
      labels:
        app: myfirstapp
    spec:
      containers:
        - name: nodejs
          image: openshiftkatacoda/blog-django-py
EOF

Create the ReplicaSet:

oc apply -f replica-set.yaml

In a new terminal window, select all pods that match app=myfirstapp:

oc get pods -l app=myfirstapp --show-labels -w

Delete the pods and watch new ones spawn:

oc delete pod -l app=myfirstapp

Imperatively scale the ReplicaSet to 6 replicas:

oc scale replicaset myfirstreplicaset --replicas=6

Imperatively scale down the ReplicaSet to 3 replicas:

oc scale replicaset myfirstreplicaset --replicas=3

The oc scale command interacts with the /scale endpoint:

curl -X GET http://localhost:8001/apis/extensions/v1beta1/namespaces/myproject/replicasets/myfirstreplicaset/scale

Use the PUT method against the /scale endpoint to change the number of replicas to 5:

curl  -X PUT localhost:8001/apis/extensions/v1beta1/namespaces/myproject/replicasets/myfirstreplicaset/scale -H "Content-type: application/json" -d '{"kind":"Scale","apiVersion":"extensions/v1beta1","metadata":{"name":"myfirstreplicaset","namespace":"myproject"},"spec":{"replicas":5}}'

You can also get information regarding the pod by using the GET method against the /status endpoint

curl -X GET http://localhost:8001/apis/extensions/v1beta1/namespaces/myproject/replicasets/myfirstreplicaset/status

The status endpoint's primary purpose is to allow a controller (with proper RBAC permissions) to send a PUT method along with the desired status.

[回到顶部](#第三步) <br>
[上一步骤](Step2.md) <br>
[下一步骤](Step4.md) <br>