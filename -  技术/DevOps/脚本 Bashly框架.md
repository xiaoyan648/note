
本项目使用 [Bashly](https://bashly.dannyb.co/) 框架构建命令行工具。Bashly 是一个用于生成 Bash 脚本的命令行工具生成器。

  

## 环境准备

  

### 安装 Bashly

  

```bash

# 安装 Ruby（如果尚未安装）

brew install ruby

  

# 安装 Bashly

gem install bashly

```

  

### 验证安装

  

```bash

bashly --version

```

  

## 项目结构

  

```

.

├── src/ # 源代码目录

│ ├── bashly.yml # Bashly 配置文件

│ ├── mr_get_command.sh # mr get 命令实现

│ ├── mr_tree_command.sh # mr tree 命令实现

│ ├── mr_patches_command.sh # mr patches 命令实现

│ ├── mr_comment_add_command.sh # mr comment add 命令实现

│ ├── repo_file_command.sh # repo file 命令实现

│ └── repo_diff_command.sh # repo diff 命令实现

├── codeup # 生成的可执行文件

├── README.md # 使用文档

└── DEVELOP.md # 开发文档（本文件）

```

  

## 开发流程

使用 Curosr IDE 开发。示例：

  

```bash

# 整体代码搭建提示词

  

根据文档：@https://help.aliyun.com/zh/yunxiao/developer-reference/merge-request-1/?scm=20140722.H_2846797._.OR_help-T_cn~zh-V_1 实现：

1. 查询合并请求

2. 查询合并请求的变更文件树

3. 查询合并请求版本列表

4. 评审合并请求

使用 bashly 框架，为整个 codeup 的 mr 子命令

  

# 新增 repo 子命令提示词

现在实现 repo 子命令：

1. file，参照文档：@https://help.aliyun.com/zh/yunxiao/developer-reference/getfileblobs?scm=20140722.H_2848248._.OR_help-T_cn~zh-V_1

2. diff，参照文档：@https://help.aliyun.com/zh/yunxiao/developer-reference/getcompare?scm=20140722.H_2846747._.OR_help-T_cn~zh-V_1

```

  

## 调试技巧

  

### 1. 启用调试模式

  

```bash

export DEBUG=1

./codeup mr get demo-repo 1

```

  

### 2. 查看生成的代码

  

生成的 `codeup` 脚本是可读的 Bash 代码，可以直接查看：

  

```bash

cat codeup

```

  

### 3. 测试单个命令脚本

  

```bash

# 手动设置变量并测试

declare -A args

args[repository]="demo-repo"

args[id]="1"

source src/mr_get_command.sh

```

  

## 部署

  

### 1. 构建发布版本

  

```bash

bashly generate

chmod +x codeup

```

  

### 2. 安装到系统

  

```bash

# 复制到系统 PATH 目录

sudo cp codeup /usr/local/bin/

  

# 或者创建符号链接

sudo ln -sf $(pwd)/codeup /usr/local/bin/codeup

```

  

## 相关资源

  

- [Bashly 官方文档](https://bashly.dannyb.co/)

- [Bashly GitHub 仓库](https://github.com/DannyBen/bashly)

- [云效 Codeup API 文档](https://thoughts.aliyun.com/workspaces/5e8c54ac553c7d00014c5043/docs/5e8c54ac553c7d00014c5045)

  

## 常见问题

  

### Q: 如何添加全局选项？

  

A: 在 `bashly.yml` 的根级别添加 `flags` 配置：

  

```yaml

name: codeup

help: Codeup CLI 工具

version: 0.1.0

  

flags:

- long: --verbose

short: -v

help: 启用详细输出

  

commands:

# ...

```

  

### Q: 如何处理可选参数的默认值？

  

A: 使用 Bash 参数扩展：

  

```bash

type="${args[--type]:-GLOBAL_COMMENT}"

```

  

### Q: 如何添加命令别名？

  

A: 在命令定义中添加 `alias` 字段：

  

```yaml

- name: get

alias: g

help: 获取合并请求详情

```

  

### Q: 如何验证重新生成后的代码？

  

A: 运行以下命令进行基本检查：

  

```bash

# 语法检查

bash -n codeup

  

# 帮助信息检查

./codeup --help

```