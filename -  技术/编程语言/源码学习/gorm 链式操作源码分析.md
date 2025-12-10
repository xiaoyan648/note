


### GORM DB 对象是线程安全的

**GORM 官方说明：**

- `*gorm.DB` 实例设计为线程安全，可以在多个 goroutine 之间共享

- `db.WithContext(ctx)` 会创建一个新的 DB session，不会修改原始对象

- 所有链式调用方法（Where、Select、Joins 等）都采用 **Copy-on-Write** 模式，返回新实例

```go
// WithContext change current instance db's context to ctx

func (db *DB) WithContext(ctx context.Context) *DB {

	return db.Session(&Session{Context: ctx})

}

  

// Session create new db session

func (db *DB) Session(config *Session) *DB {

	var (
		// new DB - copy on write
		txConfig = *db.Config
		
		tx = &DB{
		
			Config: &txConfig,
			
			Statement: db.Statement,
			
			Error: db.Error,
			
			clone: 1, // 重点设置为1
		
		}
	
	)
	
	if config.CreateBatchSize > 0 {
	
	tx.Config.CreateBatchSize = config.CreateBatchSize
	
	}
	
	  
	
	if config.SkipDefaultTransaction {
	
	tx.Config.SkipDefaultTransaction = true
	
	}
	
	  
	
	if config.AllowGlobalUpdate {
	
	txConfig.AllowGlobalUpdate = true
	
	}
	
	  
	
	if config.FullSaveAssociations {
	
	txConfig.FullSaveAssociations = true
	
	}
	
	  
	
	if config.PropagateUnscoped {
	
	txConfig.PropagateUnscoped = true
	
	}
	
	  
	
	if config.Context != nil || config.PrepareStmt || config.SkipHooks {
	
	tx.Statement = tx.Statement.clone()
	
	tx.Statement.DB = tx
	
	}
	
	  
	
	if config.Context != nil {
	
	tx.Statement.Context = config.Context
	
	}
	// ...
```

## getInstance () 源码分析

链式方法在每次调用时，内部会先调用 `getInstance()` 函数，该函数的实现如下：

再查看 clone 值的使用：
```go
func (db *DB) getInstance() *DB {
	
	if db.clone > 0 {
	
		tx := &DB{Config: db.Config, Error: db.Error}
		
		  
		// db.clone == 1
		// 复用：链接池等资源
		// 不复用：tx（事务）Vars&Clauses 等sql语句
		if db.clone == 1 {
		
			// clone with new statement
			
			tx.Statement = &Statement{
				DB: tx,
				
				ConnPool: db.Statement.ConnPool,
				
				Context: db.Statement.Context,
				
				Clauses: map[string]clause.Clause{},
				
				Vars: make([]interface{}, 0, 8),
				
				SkipHooks: db.Statement.SkipHooks,
			
			}
			
			if db.Config.PropagateUnscoped {
			
			tx.Statement.Unscoped = db.Statement.Unscoped
			
			}
		
		} else {
		
			// 复用：链接池等资源，Vars&Clauses 等sql语句
			// 不复用：tx
			
			tx.Statement = db.Statement.clone()
			
			tx.Statement.DB = tx
		
		}
		
		return tx
	
	}
	
	return db

}
```


注意：

1. 一旦 `clone` 为 1 或 2，在首次调用 `getInstance()` 后，`clone` 都为变成 0；而 clone 为 0，调用 `getInstance()` 时会陷入死循环，即无限为 0。
    
2. 只有两个函数能修改，将 `clone` 从 0 变 1 或 2：`WithContext()` 和 `Session()`（`WithContext()` 内部调用 `Session()`）。`Session()` 可以根据传入的 `config.NewDB` 的值来决定把 `clone` 变成 1 还是 2。
    