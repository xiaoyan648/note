## 简介
Tekton 是一款基于 Kubernetes 的 CI/CD 开源产品，可以在 k8s 快速搭建，并且提供了大量的功能丰富的插件。

## 安装
首先，安装 Tekton Operator。
```shell
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```
等待 Tekton 所有的 Pod 就绪。
```shell
$ kubectl wait --for=condition=Ready pods --all -n tekton-pipelines --timeout=300s
```
接下来，安装 Tekton Dashboard。
```shell
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml
```
然后，分别安装 Tekton Trigger 和 Tekton Interceptors 组件。
```shell
$ kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml

$ kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

添加一个 ingress 暴露 dashborad 
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource
  namespace: tekton-pipelines
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - host: tekton.k8s.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tekton-dashboard
                port:
                  number: 9097
```

接下来可以跟进 ingress 的配置访问 比如： http://tekton.k8s.local 

![[IMG-20240914170601470.png]]
点击 PipelineRuns 可以查看流水线运行的情况
![[IMG-20240914175445117.png]]
点击Task可以看到 有入参、出参记录，状态记录了此刻的 Yaml 资源，可以保存一些中间结果

![[IMG-20240914175714253.png]]
点击下方的 Step 可以看到每一个阶段运行的日志和Yaml资源

## Tekton使用
一条CI流水线运行的流程如下图：
![[Excalidraw/Drawing 2024-09-16 21.00.02.excalidraw.md]]
简单来说就是通过 `EventListener` 接入外部事件，然后创建 `PipelineRun` 开始流水线运行，每个流水线可以运行多个 `Task`，每个 `Task` 都是一个单独的 Pod，每一个 `Task` 运行多个阶段，每个阶段运行在独立的容器中。

下面简单的介绍一下图中出现的 `EventListener、TiggerTemplates、PipelineRun、Pipeline、Task、Step` 这些概念

### EventListener
事件监听器，用来接受外部的事件通知；在代码管理工具中可以将它暴露的接口配置在 webhook 中，用来接受事件。

### TiggerTemplates
EventListener 接受到事件后会通过 TiggerTemplates 触发资源的创建，可以通过 TiggerTemplates 创建 Task、PipelineRun等资源。

### Step
每个阶段会启动一个容器并运行 shell 脚本去执行具体的逻辑。

### Task
每个任务会启动一个Pod，串联执行多个任务

### Pipeline
流水线定义编排多个任务，任务会按照依赖关系（DAG）有序执行。

### PipelineRun
可以把Pipeline看作一个模版，PipelineRun是运行实例，即每次触发运行都是不同的PipelineRun

其中我们进行如下资源的Yaml配置编写：
![[Excalidraw/Drawing 2024-09-14 17.16.04.excalidraw.md]]


## Reference
[官方文档](https://tekton.dev/docs/getting-started/tasks/)
[自托管构建：如何使用 Tekton 构建镜像](https://time.geekbang.org/column/article/623839?utm_term=iTab&utm_source=iTab&utm_medium=iTab&utm_campaign=iTab&utm_content=iTab&screen=full)