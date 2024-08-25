我们通过在项目的.github文件加下创建 build.yaml 文件来编排工作流
![[IMG-20240825180511306.png]]
一个 build.yaml 文件用来定义 GitHub Action 工作流，总结来说，它定义了工作流的：
- 工作流名称
- 在什么时候触发
- 在什么环境下运行
- 具体执行的步骤是什么

具体如下：
```yaml
name: build

on:
    push:
        branches:
            - "feature/ci"
env:
    DOCKERHUB_USERNAME: 1425895909

jobs:
    docker:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout
              uses: actions/checkout@v3
            - name: Set outputs
              id: vars
              run: echo "::set-output name=sha_short::$(git rev-parse --short HEAD)"
            - name: Set up QEMU
              uses: docker/setup-qemu-action@v2
            - name: Set up Docker Buildx
              uses: docker/setup-buildx-action@v2
            - name: Login to Docker Hub
              uses: docker/login-action@v2
              with:
                  username: ${{ env.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}
            - name: Build backend and push
              uses: docker/build-push-action@v3
              with:
                  context: backend
                  push: true
                  platforms: linux/amd64,linux/arm64
                  tags: ${{ env.DOCKERHUB_USERNAME }}/backend:${{ steps.vars.outputs.sha_short }}
            - name: Build frontend and push
              uses: docker/build-push-action@v3
              with:
                  context: frontend
                  push: true
                  platforms: linux/amd64,linux/arm64
                  tags: ${{ env.DOCKERHUB_USERNAME }}/frontend:${{ steps.vars.outputs.sha_short }}
            # - name: Update helm values.yaml
            #   uses: fjogeleit/yaml-update-action@main
            #   with:
            #     valueFile: 'helm/values.yaml'
            #     commitChange: true
            #     branch: main
            #     message: 'Update Image Version to ${{ steps.vars.outputs.sha_short }}'
            #     changes: |
            #       {
            #         "backend.tag": "${{ steps.vars.outputs.sha_short }}",
            #         "frontend.tag": "${{ steps.vars.outputs.sha_short }}"
            #       }

```

解释一下其中主要的字段：
- on 里面描述了触发条件，案例里是指当 `feature/ci`发生变化时触发
- jobs 里描述了整个工作流的流程，一个叫 `docker` 的工作流，有 7个 stages
- steps 是具体执行的步骤：
	“Checkout”阶段负责将代码检出到运行环境。
	“Set outputs”阶段会输出 sha_short 环境变量，值为 short commit id，这可以方便在后续阶段引用。
	“Set up QEMU”和“Set up Docker Buildx”阶段负责初始化 Docker 构建工具链。
	“Login to Docker Hub”阶段通过 docker login 来登录到 Docker Hub，以便获得推送镜像的权限。要注意的是，with 字段是向插件传递参数的，在这里我们传递了 username 和 password，值的来源分别是我们定义的环境变量 DOCKERHUB_USERNAME 和 GitHub Action Secret，GitHub Action Secret在如下位置配置，前往DockerHub获取token并进行配置![[IMG-20240825180550210.png]]
	“Build backend and push”和“Build frontend and push”阶段负责构建前后端镜像，并且将镜像推送到 Docker Hub，在这个阶段中，我们传递了 context、push 和 tags 参数，context 和 tags 实际上就是 docker build 的参数。在 tags 参数中，我们通过表达式 ${{ env.DOCKERHUB_USERNAME }} 和 ${{ steps.vars.outputs.sha_short }} 分别读取了 在 YAML 中预定义的 Docker Hub 的用户名，以及在“Set outputs”阶段输出的 short commit id。

