## test
### 测试某个目录的覆盖率
```
go test -v -cover ./internal/domain/services


go test -v -cover ./internal/domain/services/stack_matcher.go -run TestCalculateSimilarity
```

### 生成 coverage.out 文件
```
go test -v -coverprofile=coverage.out -covermode=atomic ./internal/domain/services -run TestCalculateSimilarity

# 通过go tool查看每个方法的覆盖率
go tool cover -func=coverage.out
```

### 性能测试
"^$" 表示所有 .eg : "-run=^\$"
```
go test -v ./internal/domain/services -run BenchmarkCalculateSimilarity -bench=BenchmarkCalculateSimilarity
```

### pprof
```shell
1.生成 cpu.pprof 文件

# go run
go run -cpuprofile cpu.pprof main.go

# http 采集
go tool pprof -output=/tmp/cpu.pprof http://localhost:16061/debug/pprof/profile\?seconds\=30

# test 采集
go test -bench=. -cpuprofile cpu.pprof -benchmem ./...

# test 采集内存
go test -bench=. -memprofile mem.pprof -memprofilerate=1 ./...

# test 采集阻塞
go test -bench=. -blockprofile block.pprof ./...

# 远程采样30秒CPU数据，直接生成火焰图（无需本地保存pprof文件） 
go tool pprof -svg http://localhost:6060/debug/pprof/profile?seconds=30 > remote_cpu_flamegraph.svg

  

2. 分析 pprof 文件

# 场景 1：离线文件 + 启动 Web 界面

go tool pprof -http=:8080 cpu.pprof

# 场景 2. 进入交互模式 排查细节问题

go tool pprof cpu.pprof


# 在交互模式中输入 web 命令，自动生成 svg 并打开

(pprof) web


# 不进入交互模式生产
go tool pprof -svg cpu.pprof > cpu_flamegraph.svg


# 场景 3：实时采集 + 启动 Web 界面（采集 30 秒 CPU 数据）

go tool pprof -http=:8080 http://localhost:6060/debug/pprof/profile?seconds=30
```