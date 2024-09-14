
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
## Tekton使用


一条流水线常用的有如下配置
![[Excalidraw/Drawing 2024-09-14 17.16.04.excalidraw.md]]
eventlistener 接受外部事件，通过 tirggerTemplates 触发流水线运行，流水线运行多个任务
运行的流程如下图

简单解释一下它们的作用和关系

### EventListener
事件监听器，用来接受外部的事件通知；

### PipelineRun
![[IMG-20240914175445117.png]]
点击Task可以看到 有入参、出参记录，状态记录了此刻的 Yaml 资源，可以保存一些中间结果

![[IMG-20240914175714253.png]]
点击下方的 Step 可以看到每一个阶段运行的日志和Yaml资源

流水线运行的基本流程如下图：

Reference
[官方文档](https://tekton.dev/docs/getting-started/tasks/)