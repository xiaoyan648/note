## 部署工具


### Kubeadm


Kubeadm 是由官方维护的开源项目，我认为是非常简单的一个测试方式，其本身会透过 systemd 的方式维护 Kubelet。

之后再透过 container 的方式启动 controller/scheduler/kube-proxy 等 Kubernetes 核心组件。

使用方面 Kubeadm 本身并不困难，可以透过指令列的方式来创建一切所需资源，唯一要注意的是安装完毕之后还需要人为手动安装 CNI 的解决方案，整个 Kubernetes 才算是安装完毕。

Kubeadm 本身也支持架设多节点的集群，只是在使用上没有这么方便，需要先创建 Master 节点，并且产生相对应的 token/key，接下来其他节点使用 kubeadm 的指令加入到已经创建的集群中。

总体来说， Kubeadm 能够满足上述要求，但是实作上会稍嫌麻烦，特别是多节点的情况下还要处理 Token/key 的信息，此外 CNI 的安装也需要自己处理，但是作为一个单节点的测试环境也算是容易上手。

### Minikube


Minikube 也是由官方维护的项目，其本身的架构一开始是依赖于 VM (虚拟机器) 来帮使用者创建一个全新测试的 Kubernetes 集群，任何平台的开发者都可以轻松使用，因为背后都会帮你起一个全新的 VM 。当 VM 起来之后，其会透过 kubeadm 的方式帮助你建立与设定 Kubernetes 集群，并且帮你把 CNI 等指令都安装完成。

除了依赖 VM 之外，其也有提供不同底层，譬如 none 就可以直接在该机器上透过 kubeadm 来建立，基本上整个架构会变得跟 kubeadm 非常类似，比较大的差异是 CNI 也会一并帮你安装完成。

此外 Mnikube 本身也有一些属于自己的套件，可以把一些功能整包装进去，对于这个功能我的想法是不好也不坏，不坏的地方在于提供一个环境让使用者去测试功能，确实方便，不好的地方在于可能会让使用者以为这些功能都是 Kubernetes 本来就有的，反而会有所误解，甚至对于其背后使用原理都不太清楚就草草学习完毕。

总体来说， Minikube 也可以满足上述的部分要求，多节点的部分可能就会跑起来多个 VM 来建立，消耗的资源会相对多一点。

### KIND


KIND 的全名是 Kubernetes In Docker，顾名思义就是把 Kubernetes 的节点都用 Docker 的方式运行起来，每一个 Docker Container 就是一个 Kubernetes 节点，可以充当 Worker 也可以充当 Master。

使用方面非常简单，使用 KIND 的指令搭配一个设置档案就可以轻松地建立起 Kubernetes 集群，由于全部的操作都是由 KIND 完成，所以要建立多节点的方式也非常简单，只要设置文件中描述需要多少节点以及各自什么身份，接下来就一个指令搞定全部，连 CNI 方面都不需要处理， KIND 会自行搞定。

总体来说， KIND 可以满足上述所有需求，多节点的部分则是用 Docker 来管理，因此在资源与启动速度方面都有良好的效果，搭配 Vagrant 的方式就可以轻松打包一个多节点的 VM 环境供测试者开发，确实方便。

### K3D


K3D 是由 Rancher 所开发 K3S 的 Docker 版本， K3S 是一个轻量级的 Kubernetes 平台，本身适合用在一些低运算资源系统上。

而 K3D 直接将 K3S 给移植到 Docker 之中，让使用者可以更方便的创建一个 K3S 集群。

使用起来也是很简单的，整个主要架构都在 k3d 这个执行文件上面，使用该指令搭配不同的参数就可以快速地建立起多节点的 Kubernetes Cluster，此外也可以透过指令动态增加节点，使用上也是非常方便。

与 KIND一样， CNI 的部分也会一并被处理，所以使用者真的只需要一个指令就可以处理好所有的事情，总体来说， K3D 可以满足上述所有要求，优点基本上跟 KIND 完全类似，搭配上 Vagrant 真的可以轻松地建立起多节点的模拟环境。

## 部署流程


构建本地Kubernetes（K8s）环境有多种方法，其中最常见和推荐的方法是使用Minikube和Kind工具。请确保已经按照并运行了 Docker，在进行如下步骤：

### 使用Minikube搭建本地K8s环境


[https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download)

Minikube依赖于Docker来运行虚拟机，因此首先需要在你的机器上安装Docker。

可以通过以下命令安装Minikube：

```other
   curl -Lo minikube https://storage.googleapis.com/minikube/minikube-latest 操作系统版本
   sudo bash minikube.sh 
```


运行以下命令启动Minikube：

```other
  minikube start --kubernetes-version v1.25.14
```


如果遇到网络问题，可以尝试使用国内镜像进行安装。

启动成功后，可以通过以下命令检查Minikube的状态：

```other
   minikube status
```


Minikube会自动配置kubectl，你可以直接使用它来管理Kubernetes集群：

```other
   kubectl get pods
```


### 使用Kind搭建本地K8s环境


[https://kind.sigs.k8s.io/docs/user/quick-start/](https://kind.sigs.k8s.io/docs/user/quick-start/)

Kind依赖于Docker和Go语言环境，因此需要先安装这两个工具。

下载并安装Kind工具：

```other
go install sigs.k8s.io/kind@v0.23.0
```


使用Kind创建一个单节点的Kubernetes集群：

```other
   kind create cluster --name my群集名称
```


创建完成后，可以使用以下命令检查集群状态：

```other
   kind get clusters
```


Kind会自动配置kubectl，你可以直接使用它来管理Kubernetes集群：

```other
   kubectl get pods
```


### 其他注意事项

- **网络配置**：确保你的网络环境允许访问Docker仓库和Kubernetes API。
- **资源要求**：根据你的操作系统和硬件配置，可能需要调整资源分配以满足Kubernetes的需求。
- **多节点支持**：如果需要模拟多节点环境，可以考虑使用Vagrant或Terraform等基础设施即代码工具。

通过以上步骤，你可以在本地快速搭建一个Kubernetes环境，适用于开发、测试和学习用途。选择适合你的工具和方法，可以根据具体需求灵活调整。
