- 简单rpc框架
https://twitchtv.github.io/twirp/docs/intro.html
- 轻量规则引擎
https://github.com/expr-lang/expr

- retry

[https://github.com/sethvargo/go-retry/blob/main/retry.go](https://github.com/sethvargo/go-retry/blob/main/retry.go)

[https://github.com/zeromicro/go-zero/blob/master/core/fx/retry.go](https://github.com/zeromicro/go-zero/blob/master/core/fx/retry.go)

[https://github.com/go-resty/resty/blob/v2/retry.go](https://github.com/go-resty/resty/blob/v2/retry.go)

- protoc-gen-validate

支持了 自定义 errmsg的 fork包

string a=1 [(validate.rules).string = {in: ["1","2","3"],error_msg: "this is custom errormsg"}];

[https://github.com/xushuhui/protoc-gen-validate](https://github.com/xushuhui/protoc-gen-validate)

- 泛型工具库
    
    "[github.com/samber/lo](http://github.com/samber/lo)" // 主要是一些逻辑和集合处理
    
    "[github.com/samber/do](http://github.com/samber/lo)" // 动态依赖注入
    
    [](https://github.com/duke-git/lancet/tree/main)[https://github.com/duke-git/lancet](https://github.com/duke-git/lancet) // 更全面
    
- 事件中心-消息队列集成库
    
    [https://github.com/ThreeDotsLabs/watermill](https://github.com/ThreeDotsLabs/watermill)
    
- 限流器
    
    令牌桶灵活，可以根据需要选择休眠还是拒绝，在网关限流常用；
    
    漏桶不会限制请求来的流量，只是按照恒定的速度消费，如果不能消费则休眠；所以不适合网关限流，无法限制上行流量可能越堆积越多；比较适合一些脚本、任务里请求下游服务是做限流，可以保护下游服务减少下游服务触发频控的次数
    
    如果拿不准就使用令牌桶
    
    ||仓库|star|特性|
    |---|---|---|---|
    |didi|[https://github.com/didip/tollbooth](https://github.com/didip/tollbooth)||http 限流中间件，令牌桶|
    |bilibili|[https://github.com/go-kratos/aegis/tree/main/ratelimit/bbr](https://github.com/go-kratos/aegis/tree/main/ratelimit/bbr)||动态限流，一般用在grpc中间件|
    |bytedance|[https://github.com/cloudwego/kitex/tree/develop/pkg/limiter](https://github.com/cloudwego/kitex/tree/develop/pkg/limiter)|||
    |..|[github.com/juju/ratelimit](http://github.com/juju/ratelimit)|||
    |uber|[github.com/uber-go/ratelimit](https://github.com/uber-go/ratelimit)||漏桶|
    |官方|[golang.org/x/time/rate](https://pkg.go.dev/golang.org/x/time/rate)||令牌桶|
    

[https://www.cnblogs.com/jiujuan/p/16283141.html](https://www.cnblogs.com/jiujuan/p/16283141.html)

- webhook
    
    [https://github.com/go-playground/webhooks](https://github.com/go-playground/webhooks)
    
    - [https://github.com/xanzy/go-gitlab/tree/57c03d9cba1d54638ab7ba887bdc8665eeb67dc4](https://github.com/xanzy/go-gitlab/tree/57c03d9cba1d54638ab7ba887bdc8665eeb67dc4)
- http
    
    - [https://github.com/go-resty/resty](https://github.com/go-resty/resty)
- 压测
    
    - [https://github.com/tsenart/vegeta](https://github.com/tsenart/vegeta)
- markdown 解析md，生成 xhtml、html等
    

[https://github.com/yuin/goldmark](https://github.com/yuin/goldmark)

[https://github.com/88250/lute](https://github.com/88250/lute)

Blackfriday

邮件

- hermes：邮件样式构建丰富

[https://github.com/matcornic/hermes](https://github.com/matcornic/hermes)

缓存

- gocache 集成redis、memcache、memeroy 等多种使用方式

[https://github.com/eko/gocache](https://github.com/eko/gocache)

- gcache 简单的内存缓存包，支持过期和多种淘汰策略

[https://github.com/bluele/gcache](https://github.com/bluele/gcache)

- fastcache 高效的内存缓存包
- bigcache 适用于大空间缓存的管理
- [sync.map](http://sync.map) 适应与多读极少写的场景

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8fa22da3-c0f1-4f51-9c45-ed75831aa49f/42f7f7d4-5490-445f-b546-0cf58fa5e61b/Untitled.png)

工具库

- 泛型工具

[https://github.com/liyue201/gostl](https://github.com/liyue201/gostl)

[](https://github.com/chen3feng/stl4go/blob/master/README_zh.md)

[https://github.com/songzhibin97/go-baseutils](https://github.com/songzhibin97/go-baseutils)

- 结构体拷贝
    
    [https://github.com/jinzhu/copier](https://github.com/jinzhu/copier)
    
- dsl自定义语言
    
    [Internals | Expression language](https://expr.medv.io/docs/Internals)
    

gateway

[GitHub - TykTechnologies/tyk: Tyk Open Source API Gateway written in Go, supporting REST, GraphQL, TCP and gRPC protocols](https://github.com/TykTechnologies/tyk/tree/master)

kong

tcp服务器

[01 QuickStart](https://github.com/aceld/zinx/wiki/01-QuickStart#download-zinx-source)

第三方系统

- [https://github.com/silenceper/wechat](https://github.com/silenceper/wechat)
    
- 流程并发处理 Reactive Extensions for the Go language.
    

[https://github.com/ReactiveX/RxGo/blob/master/doc/last.md](https://github.com/ReactiveX/RxGo/blob/master/doc/last.md)

- [https://github.com/dtm-labs/dtm](https://github.com/dtm-labs/dtm) 分布式事务
- 常用 dockerfile [https://github.com/Mr-houzi/php-docker/blob/master/docker-compose](https://github.com/Mr-houzi/php-docker/blob/master/docker-compose)

### 分析工具

- [apicompat](https://github.com/bradleyfalzon/apicompat) - 检查Go项目的最新更改是否存在向后不兼容的更改。
- [dupl](https://github.com/mibk/dupl) - 代码克隆检测工具。
- [errcheck](https://github.com/kisielk/errcheck) - Errcheck是用于检查Go程序中未经检查的错误的程序。
- [gcvis](https://github.com/davecheney/gcvis) - 实时可视化Go程序GC跟踪数据。
- [go-checkstyle](https://github.com/qiniu/checkstyle) - checkstyle是样式检查工具，例如java checkstyle。该工具的灵感来自java checkstyle golint。该样式涉及“ Go Code评论注释”中的某些要点。
- [go-cleanarch](https://github.com/roblaszczak/go-cleanarch) - 创建go-cleanarch是为了验证Clean Architecture规则，例如The Dependency Rule和Go项目中程序包之间的交互。
- [go-critic](https://github.com/go-critic/go-critic) - 源代码linter，它带来了当前在其他linter中未实现的检查。
- [go-mod-outdated](https://github.com/psampaz/go-mod-outdated) - 查找Go项目的过时依赖项的简便方法。
- [go-outdated](https://github.com/firstrow/go-outdated) - 显示过期软件包的控制台应用程序。
- [goast-viewer](https://github.com/yuroyoro/goast-viewer) - 基于Web的Golang AST可视化工具。
- [GoCover.io](http://gocover.io/) - GoCover.io提供任何golang软件包即服务的代码覆盖率。
- [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) - 自动修复（添加，删除）Go导入的工具。
- [GolangCI](https://golangci.com/) - GolangCI是针对GitHub拉取请求的自动化Golang代码审查服务。服务是开源的，对于开源项目是免费的。
- [GoLint](https://github.com/golang/lint) - Golint是Go源代码的皮特。
- [Golint online](http://go-lint.appspot.com/) - 使用golint软件包在线上GitHub，Bitbucket和Google Project Hosting上在线Go源文件。
- [GoPlantUML](https://github.com/jfeliu007/goplantuml) - 生成文本plantump类图的库和CLI，该类图包含有关结构和接口以及它们之间的关系的信息。
- [goreturns](https://sourcegraph.com/github.com/sqs/goreturns) - 添加零值返回语句以匹配func返回类型。
- [gosimple](https://github.com/dominikh/go-tools/tree/master/cmd/gosimple) - gosimple是Go源代码的linter，专门研究简化代码。
- [gostatus](https://github.com/shurcooL/gostatus) - 命令行工具，显示包含Go软件包的存储库的状态。
- [lint](https://github.com/surullabs/lint) - 作为go测试的一部分运行棉绒。
- [php-parser](https://github.com/z7zmey/php-parser) - 用Go编写的PHP解析器。
- [staticcheck](https://github.com/dominikh/go-tools/tree/master/cmd/staticcheck) - staticcheck go vet用于类固醇，对ReSharper for C＃等工具应用了大量的静态分析检查。
- [tarp](https://github.com/verygoodsoftwarenotvirus/tarp) - tarp在Go源代码中查找没有直接单元测试的函数和方法。
- [tickgit](https://github.com/augmentable-dev/tickgit) - CLI和go软件包，用于显示代码注释TODO（以任何语言显示）并应用git blame标识作者。
- [unconvert](https://github.com/mdempsky/unconvert) - 从Go源代码中删除不必要的类型转换。
- [unused](https://github.com/dominikh/go-tools/tree/master/cmd/unused) - 未使用的检查将代码用于未使用的常量，变量，函数和类型。
- [validate](https://github.com/mccoyst/validate) - 自动验证带有标签的结构域。