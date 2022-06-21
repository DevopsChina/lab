
# 云原生 CI/CD Tekton

![](media/16553854211360.jpg)

tektōn 在古希腊语中有工匠、手艺人的意思，比如木匠、石匠、建筑工人。

## Tekton 介绍

Tekton 是 Google 开源的 Kubernetes 原生 CI/CD 系统，功能强大扩展性强。前身是 Knavite 里的 build-pipeline 项目，后期孵化成独立的项目。并成为 CDF 下的四大初始项目之一，其他三个是 Jenkins, Jenkins X, Spinnaker。

### 优势

* 可定制
* 可重用
* 可扩展
* 标准化
* 可伸缩

### 概念

* Step
* Task
* Pipeline
* TaskRun
* PipelineRun

![](media/16553864459332.png)

### CRD

为什么说 Tekton 是 Kubernetes 原生的，因为其基于 Kubernetes 的 CRD 定义了 Pipeline 流水线。

* [`Tasks`](https://github.com/tektoncd/pipeline/blob/main/docs/tasks.md)
* [`Pipeline`](https://github.com/tektoncd/pipeline/blob/main/docs/pipelines.md)
* [`TaskRun`](https://github.com/tektoncd/pipeline/blob/main/docs/taskruns.md)
* [`PipelineRun`](https://github.com/tektoncd/pipeline/blob/main/docs/pipelineruns.md)

### Tekton CRD VS Native Resource

![](media/16553856425843.jpg)

## 工作原理

从 PipelineRun 到 TaskRun 再到 Pod 和容器。

![tekton-concept](media/tekton-concept.jpg)


详细分析见[Tekton 的工作原理](https://atbug.com/how-tekton-works/)

## Tekton 生态

### 组件

Tekton 包含了多个组件：

* [Tekton Pipelines](https://github.com/tektoncd/pipeline/blob/main/docs/README.md)
* [Tekton Triggers](https://github.com/tektoncd/triggers/blob/main/README.md)
* [Tekton CLI](https://github.com/tektoncd/cli/blob/main/README.md)
* [Tekton Dashboard](https://github.com/tektoncd/dashboard/blob/main/README.md)
* [Tekton Catalog](https://github.com/tektoncd/catalog/blob/v1beta1/README.md)
* [Tekton Hub](https://github.com/tektoncd/hub/blob/main/README.md)
* [Tekton Operator](https://github.com/tektoncd/operator/blob/main/README.md)

## 演示

既然 Tekton 是 Kubernetes 原生的框架，在正式开始之前需要创建一个 Kubernetes 集群。

在这个集群上我们会安装 Tekton，为了简化架构，在 CD 阶段会将应用也部署这个集群上（实际场景下，CI/CD 的集群基本不会与应用共享集群。当然，共享也没有问题）。

这个演示中我们会实现一个简单的 CI/CD 的流水线：完成一个 [Java 项目](https://github.com/addozhang/tekton-demo)从代码到部署的整个流程。

这个 Java 项目是个 web 服务，有一个 `/hi` 端点，返回 `hello world`。演示的重点是流水线的实现，所以选用了最简单的项目。

### 环境介绍

* k3s v1.21.13+k3s1
* 2c8g 虚拟机 Ubuntu 20.04
* 本地 macOS

### 安装集群

我们使用 k3s 作为 Kubernetes 集群，通过下面的命令可以初始化单节点的集群。

这里我使用的是 2c8g 的 vm 作为节点，既是控制节点也是计算节点。

```shell
export INSTALL_K3S_VERSION=v1.21.13+k3s1
curl -sfL https://get.k3s.io | sh -s - --disable traefik --write-kubeconfig-mode 644 --write-kubeconfig ~/.kube/config
```

### 安装 Tekton Pipeline

最新版本是 v0.36，从 v0.33.x 开始要求 Kubernetes 的版本至少是 1.21。

```shell
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

检查相关的 CRD。

```shell
kubectl api-resources --api-group=tekton.dev
NAME                SHORTNAMES   APIVERSION            NAMESPACED   KIND
clustertasks                     tekton.dev/v1beta1    false        ClusterTask
conditions                       tekton.dev/v1alpha1   true         Condition
pipelineresources                tekton.dev/v1alpha1   true         PipelineResource
pipelineruns        pr,prs       tekton.dev/v1beta1    true         PipelineRun
pipelines                        tekton.dev/v1beta1    true         Pipeline
runs                             tekton.dev/v1alpha1   true         Run
taskruns            tr,trs       tekton.dev/v1beta1    true         TaskRun
tasks                            tekton.dev/v1beta1    true         Task
```

检查组件运行

```shell
kubectl get po -n tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-5cfb9b8cfc-q4crs   1/1     Running   0          24s
tekton-pipelines-webhook-6c9d4d5798-7xg8n      1/1     Running   0          24s
```

### 安装 Tekton Dashboard

通过 Dashboard 我们可以实时查看 `PipelineRun` 和 `TaskRun` 的状态，以及运行的日志；还可以查看定义的各种 CR。

```shell
kubectl apply --filename \
https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```

### Hello, Tekton

创建 Task

```shell
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-tekton
spec:
  steps:
    - name: echo
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello, Tekton"   
EOF
```

运行

```shell
kubectl apply -f - <<EOF
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: hello-tekton-task-run
spec:
  taskRef:
    name: hello-tekton
EOF
```

### Demo

我们使用Spring Initializer生成的项目为例, 演示如何使用 Tekton 实现 CICD.

开始之前简单整理下这个项目的 CICD 流程：

1. 拉取代码
2. maven 打包
3. 构建镜像并推送
4. 部署

#### 0x01 RBAC

用于 PipelineRun 运行的 service account。

```shell
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-build

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pipeline-admin-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin # user cluster role admin
subjects:
- kind: ServiceAccount
  name: tekton-build
  namespace: tekton-pipelines
EOF
```

#### 0x02 克隆代码

```shell
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.4/git-clone.yaml
```

或者

```shell
tkn hub install task git-clone
```

#### 0x03 maven 打包

```yaml
spec:
  steps:
    - name: maven
      image: maven:3.5-jdk-8-alpine
      imagePullPolicy: IfNotPresent
      workingDir: $(workspaces.source.path)
      command:
        - mvn
      args:
        - clean
        - install
        - -DskipTests
      volumeMounts:
        - name: m2
          mountPath: /root/.m2
  volumes:
    - name: m2
      hostPath:
        path: /data/.m2
```

#### 0x04 构建并推送镜像

推送镜像到 Docker Hub，需要相关的登录凭证。

kaniko 需要将 docker config 的文件存在于 /kanika/.docker 目录下。这里的思路是将 docker 的 config.json，以 secret 的方式持久化，在通过先添加 docker-registry类型的 secret，然后通过 workspace 的方式输入到 kaniko 运行环境中。

```shell
kubectl create secret docker-registry dockerhub --docker-server=https://index.docker.io/v1/ --docker-username=[USERNAME] --docker-password=[PASSWORD] --dry-run=client -o json | jq -r '.data.".dockerconfigjson"' | base64 -d > /tmp/config.json && kubectl create secret generic docker-config --from-file=/tmp/config.json && rm -f /tmp/config.json
```

构建镜像需要指定资源，比如 Dockerfile 的路径、镜像 URL、tag 等，通过 `params` 输入。

```yaml
spec:
  params:
    - name: pathToDockerFile
      description: The path to the dockerfile to build (relative to the context)
      default: Dockerfile
    - name: imageUrl
      description: Url of image repository
    - name: imageTag
      description: Tag to apply to the built image
      default: latest
    - name: IMAGE
      description: Name (reference) of the image to build.  
  steps:
    - name: build-and-push
      image: gcr.io/kaniko-project/executor:v1.6.0-debug
      imagePullPolicy: IfNotPresent
      command:
        - /kaniko/executor
      args:
        - --dockerfile=$(params.pathToDockerFile)
        - --destination=$(params.imageUrl):$(params.imageTag)
        - --context=$(workspaces.source.path)
        - --digest-file=$(results.IMAGE_DIGEST.path)
```

#### 0x05 部署

在部署阶段，也就是 CD 中的 delivery。我们将使用项目中的 yaml 文件，对应用进行部署。

前面提到，应用会部署到当前的集群中。部署成功后，我们可以通过访问 `http://[node-ip]:30080/hi` 来进行验证。

```yaml
spec:
  params:
    - name: pathToYamlFile
      description: The path to the yaml file to deploy within the git source
      default: deployment.yaml
  workspaces:
    - name: source
  steps:
    - name: run-kubectl
      image: lachlanevenson/k8s-kubectl:v1.21.11
      imagePullPolicy: IfNotPresent
      command: ["kubectl"]
      args:
        - "apply"
        - "-f"
        - "$(workspaces.source.path)/$(params.pathToYamlFile)"
```

#### 0x06 组装流水线

在前面我们已经完成了流水线的各个 `step` 和 `task`，接下来就是将所有的 task 组装成真正的流水线 `Pipeline`。

在 `Pipeline` 中，我们设定流水线各个 `step` 所需的入参，并按照顺序将 `task` “摆放”。默认情况下这些 `task` 会同时执行，我们通过 `runAfter` 字段对执行顺序进行编排。

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-pipeline
spec:
  params:
    - name: git-url
    - name: git-revision
    - name: pathToContext
      description: The path to the build context, used by Kaniko - within the workspace
      default: .
    - name: imageUrl
      description: Url of image repository
    - name: imageTag
      description: Tag to apply to the built image
  workspaces:
    - name: git-source
    - name: docker-config
  tasks:
    - name: fetch-from-git
      taskRef:
        name: git-clone
      params:
        - name: url
          value: "$(params.git-url)"
        - name: revision
          value: "$(params.git-revision)"
      workspaces:
        - name: output
          workspace: git-source
    - name: source-to-image
      taskRef:
        name: source-to-image
      params:
        - name: imageUrl
          value: "$(params.imageUrl)"
        - name: IMAGE  
          value: "$(params.imageUrl)"
        - name: imageTag
          value: "$(params.imageTag)"
      workspaces:
        - name: source
          workspace: git-source
        - name: dockerconfig
          workspace: docker-config
      runAfter:
        - fetch-from-git
    - name: deploy-to-k8s
      taskRef: 
        name: deploy-to-k8s
      params:
        - name: pathToYamlFile
          value: deployment.yaml
      workspaces:
        - name: source
          workspace: git-source
      runAfter:
        - source-to-image
```

#### 0x07 执行流水线

前面我们已经完成了流水线的定义，在执行的时候，需要通过定义 `PipelineRun` 来为其指定入参。比如这里的 `git-revision`、`git-url`、`imageUrl`、`imageTag`。

拉取代码和编译两个 `task` 的运行是在不同的 Pod 中完成的，因此需要出持久化的存储来进行数据（代码仓库）的共享。

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  generateName: generic-pr-
  name: generic-pipeline-run
spec:
  pipelineRef:
    name: build-pipeline
  params:
    - name: git-revision
      value: main
    - name: git-url
      value: https://github.com/addozhang/tekton-demo.git  
    - name: imageUrl
      value: addozhang/tekton-test
    - name: imageTag
      value: latest
  workspaces:
    - name: git-source
      volumeClaimTemplate:
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 1Gi
    - name: docker-config
      secret:
        secretName: docker-config
  serviceAccountName: tekton-build
```

###  测试

执行下面的命令创建 `PipelineRun` 资源启动流水线。

```yaml
kubectl apply -f run/run.yaml
```

在执行的过程中，你会看到下面几个 Pod 被创建：

* generic-pipeline-run-deploy-to-k8s-xxx
* generic-pipeline-run-fetch-from-git-xxx
* generic-pipeline-run-source-to-image-xxx

同时还有我们应用的 Pod `tekton-test-xxx`。

尝试发送请求到 `http://[node-ip]:30080/hi`，查看返回结果。

## 总结

Tekton 是个很有意思的项目，其生态也在一步步的壮大，与此同时业界也不断涌现出各种周边的工具。由于时间原因，无法一一介绍。有兴趣的同学，可以看下我之前写过的文档。后续有时间，也希望能在这里继续给大家分享。

* [CICD 的供应链安全工具 Tekton Chains](https://atbug.com/tekton-chains-secure-supply-chain/)
* [Jenkins 如何与 Kubernetes 集群的 Tekton Pipeline 交互？](https://atbug.com/jenkins-interact-with-tekton-pipelines-via-plugin/)
* [云原生CICD: Tekton Trigger 实战](https://atbug.com/tekton-trigger-practice/)

CI/CD 平台是一件有挑战且充满乐趣的事情，在这个过程中我们会将现实世界中的工作流程以软件的方式实现出来。

![CI:CD流程](media/CI:CD%E6%B5%81%E7%A8%8B.png)

各家企业有自己独特的组织架构、管理制度，以及研发流程，即使是发展的不同阶段对平台也会有不同的需求。

平台的实现可以简单，也可以很复杂。