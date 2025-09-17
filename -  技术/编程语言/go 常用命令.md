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