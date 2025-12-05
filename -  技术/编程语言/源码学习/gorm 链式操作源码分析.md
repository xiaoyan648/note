


 [![Ayang](https://ayang.ink/images/avatar.jpg "Ayang") Ayang](https://github.com/AyangHuang "作者") 收录于  [Go](https://ayang.ink/categories/go/)

 2023-08-10  约 701 字 

对 gorm 的链式操作一直有疑惑，最近做项目，翻了下源码看了具体的实现，做下记录。

当然推荐先看官方文档了解下链式操作罗：  
[https://gorm.io/zh_CN/docs/method_chaining.html](https://gorm.io/zh_CN/docs/method_chaining.html)

看完是不是有很多疑问？例如：

[![/images/gorm 链式操作分析/why.png](https://ayang.ink/images/gorm%20%e9%93%be%e5%bc%8f%e6%93%8d%e4%bd%9c%e5%88%86%e6%9e%90/why.png "/images/gorm 链式操作分析/why.png")](https://ayang.ink/images/gorm%20%e9%93%be%e5%bc%8f%e6%93%8d%e4%bd%9c%e5%88%86%e6%9e%90/why.png)

why

## [](https://ayang.ink/go_gorm-%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C%E5%88%86%E6%9E%90/#getinstance-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90)getInstance () 源码分析

链式方法在每次调用时，内部会先调用 `getInstance()` 函数，该函数的实现如下：

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>```|```go<br>func (db *DB) getInstance() *DB {<br>	if db.clone > 0 {<br>        // 创建新的 DB 实例，由于 Go 变量零值的原因，新的 DB.clone = 0 <br>		tx := &DB{Config: db.Config, Error: db.Error}<br><br>		if db.clone == 1 {<br>			// 创建一个全新的 statement<br>			tx.Statement = &Statement{<br>				DB:       tx,<br>				ConnPool: db.Statement.ConnPool,<br>				Context:  db.Statement.Context,<br>				Clauses:  map[string]clause.Clause{},<br>				Vars:     make([]interface{}, 0, 8),<br>			}<br>		} else {<br>            // 复用原来的的 statement，即 SQL 语句会复用<br>			tx.Statement = db.Statement.clone()<br>			tx.Statement.DB = tx<br>		}<br><br>		return tx<br>	}<br><br>    // clone = 0，直接返回原 db<br>	return db<br>}<br>```|

再查看 clone 值的使用：

[![/images/gorm 链式操作分析/clone.png](https://ayang.ink/images/gorm%20%e9%93%be%e5%bc%8f%e6%93%8d%e4%bd%9c%e5%88%86%e6%9e%90/clone.png "/images/gorm 链式操作分析/clone.png")](https://ayang.ink/images/gorm%20%e9%93%be%e5%bc%8f%e6%93%8d%e4%bd%9c%e5%88%86%e6%9e%90/clone.png)

clone

追进去，发现只有两个函数三处地方对 clone 修改：

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>```|```go<br>func Open(dialector Dialector, opts ...Option) (db *DB, err error) {<br>    ...<br>    // 第一个函数，第一处<br>	db = &DB{Config: config, clone: 1}<br>    ...<br>}<br><br>func (db *DB) Session(config *Session) *DB {<br>	var (<br>		txConfig = *db.Config<br>		tx       = &DB{<br>			Config:    &txConfig,<br>			Statement: db.Statement,<br>			Error:     db.Error,<br>            // 第二个函数，第一处<br>			clone:     1,<br>		}<br>	)<br><br>    ...<br>    // 第二个函数，第二处<br>    // 只有 db.Session(&gorm.config{NewDB:true}) 才不会执行这里的逻辑<br>    //（因为 NewDB 的零值为 false），调用即 Session 后的新 DB 的 clone 值默认设置为 2<br>	if !config.NewDB {<br>		tx.clone = 2<br>	}<br>    ...<br>	return tx<br>}<br>```|

其实还有第三个函数会对 clone 值做出修改，即 `getInstance()` 函数，因为创建一个新的 DB 后的 clone 为零值，默认为 0 。

根据以上源码分析，可以得到下图：

1. 箭头表示调用 `getInstance` 的 `clone` 的变化；
    
2. 椭圆旁边的文字表示 clone 强制改变的方式。
    

[![/images/gorm 链式操作分析/clone 状态转变.png](https://ayang.ink/svg/loading.min.svg "/images/gorm 链式操作分析/clone 状态转变.png")](https://ayang.ink/images/gorm%20%e9%93%be%e5%bc%8f%e6%93%8d%e4%bd%9c%e5%88%86%e6%9e%90/clone%20%e7%8a%b6%e6%80%81%e8%bd%ac%e5%8f%98.png)

clone 状态转变

注意：

1. 一旦 `clone` 为 1 或 2，在首次调用 `getInstance()` 后，`clone` 都为变成 0；而 clone 为 0，调用 `getInstance()` 时会陷入死循环，即无限为 0。
    
2. 只有两个函数能修改，将 `clone` 从 0 变 1 或 2：`WithContext()` 和 `Session()`（`WithContext()` 内部调用 `Session()`）。`Session()` 可以根据传入的 `config.NewDB` 的值来决定把 `clone` 变成 1 还是 2。
    

## [](https://ayang.ink/go_gorm-%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C%E5%88%86%E6%9E%90/#end)End

更新于 2023-08-10 

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

返回 | [主页](https://ayang.ink/)

[grpc 基于 etcd 的服务发现](https://ayang.ink/%E5%88%86%E5%B8%83%E5%BC%8F_grpc-%E5%9F%BA%E4%BA%8E-etcd-%E7%9A%84%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0/ "grpc 基于 etcd 的服务发现")[Go 适合网络 IO](https://ayang.ink/go_go-%E9%80%82%E5%90%88%E7%BD%91%E7%BB%9C-io/ "Go 适合网络 IO")

由 [Hugo](https://gohugo.io/ "Hugo 0.105.0-DEV") 强力驱动 | 主题 -  [![FixIt logo](https://ayang.ink/fixit.min.svg) FixIt](https://github.com/hugo-fixit/FixIt "FixIt v0.2.16")

 2022 - 2024 [Ayang](https://github.com/AyangHuang)[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

[闽公网安备 35021102001594 号](http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=35021102001594)[粤 ICP 备 2022116941 号](https://beian.miit.gov.cn/)