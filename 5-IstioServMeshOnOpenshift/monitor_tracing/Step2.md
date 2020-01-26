# 第二步
Clone the Example project

###### 快捷链接
[场景综述](../deploy_microservices.md) <br>
[上一步](Step1.md) <br>
[下一步](Step3.md) <br>

#### 操作过程
[Clone样本项目](#Clone样本项目) <br>


### Clone样本项目
样例项目在 github 上 https://github.com/redhat-developer-demos/istio-tutorial
```
git clone https://github.com/redhat-developer-demos/istio-tutorial/
```

> github.com/redhat-developer-demos 是 Redhat 为开发者提供的各种 Demos 。包括了各种较新项目的原型演示，用于帮助开发者快速理解各种新技术构成、使用与目的。这里使用到的 istio-tutorial 是红帽为了帮助开发者快速理解 istio 的逻辑与概念而设计的快速演示程序。
```
 $ ls istio-tutorial/
bin           diferencia       generate_docs.sh      keycloak    readme.adoc     site-gh-pages.yml
customer      documentation    generate_workshop.sh  LICENSE     recommendation  site-workshop.yml
dev-site.yml  generate_dev.sh  istiofiles            preference  scripts         supplemental-ui
```
> 从 clone 下来的这个目录结构里看，istio-tutorial 除了包含了 customer、preference、recommendation 三个目录，包含了三个微服务相应的资源及描述 ，还包含了一系列为了 kubernetes/Openshift 准备的脚本与 yaml 文件。

[回到顶部](#第二步) <br>
[上一步骤](Step1.md) <br>
[下一步骤](Step2.md) <br>