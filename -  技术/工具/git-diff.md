## Git Diff 常见用法对比

### 📊 --stat (统计信息)
显示文件变更的简要统计信息，包括修改的文件列表和每个文件的增删行数（图形化显示）

```bash
git diff --stat
```

**输出示例：**
```
 internal/devops/x/download/batch_download.go | 45 ++++++++++++++---
 internal/devops/service/artifact.go         | 12 +----
 README.md                                    |  3 +-
 3 files changed, 38 insertions(+), 22 deletions(-)
```

**特点：**
- 📁 显示修改的文件名
- 📈 用 `+` 和 `-` 符号图形化显示增删行数
- 📊 最后一行显示总体统计

### 🔢 --numstat (数字统计)
显示纯数字格式的统计信息，便于脚本处理

```bash
git diff --numstat
```

**输出示例：**
```
38      7       internal/devops/x/download/batch_download.go
2       10      internal/devops/service/artifact.go
1       2       README.md
```

**格式：** `添加行数 TAB 删除行数 TAB 文件名`

**特点：**
- 🤖 机器友好的格式（TAB 分隔）
- 📈 纯数字，便于解析和统计
- 🚫 没有图形化符号

### 📋 --patch-with-stat (补丁+统计)
同时显示统计信息和详细的代码变更内容

```bash
git diff --patch-with-stat
git diff HEAD~1 HEAD --patch-with-stat
git diff COMMIT1 COMMIT2 --patch-with-stat
```
**输出示例：**
```
 internal/devops/x/download/batch_download.go | 45 ++++++++++++++---
 internal/devops/service/artifact.go         | 12 +----
 2 files changed, 38 insertions(+), 10 deletions(-)

diff --git a/internal/devops/x/download/batch_download.go b/internal/devops/x/download/batch_download.go
index 1234567..abcdef8 100644
--- a/internal/devops/x/download/batch_download.go
+++ b/internal/devops/x/download/batch_download.go
@@ -212,6 +212,10 @@ func genZipFile(oriDir, destFilename string) (string, error) {
 	// 获取目录下所有文件名
 	oriFiles, err := filepath.Glob(filepath.Join(oriDir, "*"))
 	if err != nil {
+		// 确保源目录存在
+		if !file.IsExisted(oriDir) {
+			err := fmt.Errorf("source directory not exist: [%s]", oriDir)
+		}
 		err = fmt.Errorf("find files in [%s], err: [%w]", oriFiles, err)
 		return "", err
 	}
```

**特点：**
- 📊 包含 `--stat` 的统计信息
- 📝 包含完整的代码差异内容
- 🔍 显示具体的代码变更

## 🔄 使用场景对比

| 选项 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| `--stat` | 📋 快速了解变更概况 | 简洁直观，一目了然 | 看不到具体代码 |
| `--numstat` | 🤖 脚本统计分析 | 格式规整，易解析 | 可读性差 |
| `--patch-with-stat` | 🔍 代码审查时需要完整信息 | 信息最全面 | 输出冗长 |

## 💡 常见组合用法

```bash
# 查看工作区与暂存区的差异统计
git diff --stat

# 查看暂存区与最后一次提交的差异
git diff --cached --stat

# 比较两个提交之间的差异
git diff HEAD~1 HEAD --numstat

# 查看某个文件的详细变更
git diff --patch-with-stat -- path/to/file.go

# 只显示修改的文件名
git diff --name-only

# 显示修改状态（新增/修改/删除）
git diff --name-status
```

## 📌 实用技巧

1. **与其他选项组合：**
   ```bash
   git diff --stat --color=always  # 带颜色输出
   git diff --stat HEAD~3          # 与3个提交前比较
   ```

2. **限制输出范围：**
   ```bash
   git diff --stat -- "*.go"       # 只看Go文件的变更
   git diff --stat src/            # 只看src目录的变更
   ```

3. **结合管道处理：**
   ```bash
   git diff --numstat | awk '{sum+=$1+$2} END {print sum}'  # 统计总变更行数
   ```

选择哪个选项主要取决于你的需求：想要快速概览用 `--stat`，需要脚本处理用 `--numstat`，代码审查用 `--patch-with-stat`。