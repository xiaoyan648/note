## git clone 做了什么
我们拉取原创代码会通过如下命令

`$ git clone https://github.com/xxx/demo`

这会在当前目录下创建一个名为 “demo” 的目录，并在这个目录下初始化一个 `.git` 文件夹， 从远程仓库拉取下所有数据放入 `.git` 文件夹，然后从中读取最新版本的文件的拷贝。

**git 默认会进行全部克隆**

这个`.git`文件夹包含我们所有commit的历史版本，以及所有分支。假如仓库有master和dev两个分支，且每个分支占有4MB空间，那么我们下载的仓库代码大小为8MB。再加上 commit 的历史版本，所以下载的仓库代码不止8MB。

## 如何更快克隆仓库

[gi官方文档](https://link.juejin.cn?target=https%3A%2F%2Fgit-scm.com%2Fdocs%2Fgit-clone%2Fzh_HANS-CN "https://git-scm.com/docs/git-clone/zh_HANS-CN") 已经给出了答案。添加参数 depth <深度>

推荐 `git clone XXXXXX.git --depth=1`。表示拉取一个默认的主分支，并且只有最新的commit信息。
每次提交 commit 都会对应一个版本的全量快照数据；通过 `--depth=1` 拉取的最新 commit 其实对应的是一份完整的代码快照，再是增量commit;

通过 `--depth=1` 获取的仓库是一个浅层仓库，后续想要拉取其他分支只能先切换默认分支在拉取如下
```shell
$ git remote set-branches origin 'remote_branch_name'    
$ git fetch --depth=1 origin remote_branch_name
```
想要恢复可以通过 `git fetch --unshallow`或`git pull --unshallow`命令将浅层仓库转换为完整仓库，这样可以消除浅层存储库所施加的所有限制

## 总结
1. 在只需要用到最新代码的场景可以通过 `--depth=1` 提升代码下载速度，比如 部署场景
2. 在开发协作场景不要使用浅克隆，需要协同开发，历史commit和其他分支还是需要拉取到本地的