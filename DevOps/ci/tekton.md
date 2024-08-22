
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


Reference
[官方文档](https://tekton.dev/docs/getting-started/tasks/)