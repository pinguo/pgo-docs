# Db

关系数据库组件(DB)是对GO内置的`database/sql`包的组件封装，需要和具体的数据库驱动一起使用，驱动列表参见 <https://golang.org/s/sqldrivers>，例如，使用mysql需在程序的main入口中导入mysql的驱动包：
```go
import _ "github.com/go-sql-driver/mysql"
```

Db组件支持读写分离，`SELECT`语句默认在从库中执行，其余语句在主库中执行，支持慢日志记录。

## 配置文件

```yaml
# 组件ID，db组件可以配置多个，只要分配唯一的组件ID, 通过
# GetObject(Db.AdapterClass, <id>)来获取非默认db组件
db:
    # Db组件类名称
    class:  "@pgo/Client/Db/Client"
    # 驱动名(需导入指定的驱动包)
    driver: "mysql"
    # 主库地址(参见各驱动包说明)
    dsn: "user:pass@tcp(127.0.0.1:3306)/db?charset=utf8&timeout=0.5s"
    # 从库地址(可选，默认空)
    # slaves: ["slave1 dsn", "slave2 dsn"]
    # 最大空闲连接数(默认5)
    # maxIdleConn: 5
    # 数据库建立连接的最大数目(默认0 无限制)
    # maxOpenConn: 10
    # 是完整的记录sql日志，且记录耗时(默认false 只记录执行方法和耗时)
    # sqlLog:true
    # 最大连接维持时间(默认1小时)
    # maxConnTime: "1h"
    # 慢日志最小时间(默认100ms)
    # slowLogTime: "100ms"
```

## 功能列表

```go
db.Begin()           // 开始一个新事务(默认超时上下文)
db.BeginContext()    // 开始一个新事务(指定上下文)
db.Commit()          // 提交一个事务
db.Rollback()        // 回滚一个事务
db.QueryOne()        // 查询单个文档(默认超时上下文)
db.QueryOneContext() // 查询单个文档(指定上下文)
db.Query()           // 查询多个文档(默认超时上下文)
db.QueryContext()    // 查询多个文档(指定上下文)
db.Exec()            // 非查询操作(默认超时上下文)
db.ExecContext()     // 非查询操作(指定上下文)
db.Prepare()         // 批量操作(默认超时上下文)
db.Prepare()         // 批量操作(指定上下文)
```

## 使用示例

```go
// 使用db.Exec/db.ExecContext在主库上执行非查询操作
func (m *MysqlController) ActionExec() {
    // 获取db的上下文适配对象
    db := m.GetObject(Db.AdapterClass).(*Db.Adapter)

    // 执行插入操作
    query := "INSERT INTO test (name, age) VALUES (?, ?)"
    db.Exec(query, "name11", 11)

    // 执行修改操作
    query = "UPDATE test SET age=age+1 WHERE id=?"
    db.Exec(query, lastId)

    // 执行删除操作
    query = "DELETE FROM test WHERE id=?"
    db.Exec(query, lastId)
}

// 使用db.Query/QueryOne/QueryContext/QueryOneContext来查询数据，
// 若当前db对象未执行过任何操作，则使用从库进行查询，否则复用上一次获取的db连接。
func (m *MysqlController) ActionQuery() {
    // 获取db的上下文适配对象
    db := m.GetObject(Db.AdapterClass).(*Db.Adapter)
    
    // 查询单条数据
    id, name, age := 0, "", 0
    query := "SELECT id, name, age FROM test WHERE id=?"
    db.QueryOne(query, 3).Scan(&id, &name, &age)
    fmt.Println("query one for id=3, ", id, name, age)

    // 查询多条数据
    query = "SELECT id, name, age FROM test WHERE age>?"
    rows := db.Query(query, 10)
    defer rows.Close()

    for rows.Next() {
        id, name, age := 0, "", 0
        rows.Scan(&id, &name, &age)
        fmt.Println("user: ", id, name, age)
    }
}

// 使用db.Prepare/db.PrepareContext来执行批量操作，默认查询操作在
// 从库上进行，其余操作在主库上进行，若当前db对象有过其它操作，则查询
// 操作会复用之前的连接。
func (m *MysqlController) ActionPrepare() {
    // 获取db的上下文适配对象
    db := m.GetObject(Db.AdapterClass).(*Db.Adapter)

    query := "INSERT INTO test (name, age) VALUES (?, ?)"
    stmt := db.Prepare(query)
    defer stmt.Close()

    for i := 0; i < 10; i++ {
        stmt.Exec(fmt.Sprintf("name%d", i), i)
    }
}

// 使用db.Begin/BeginContext/Commit/Rollback来进行事务操作
func (m *MysqlController) ActionTransaction() {
    // 获取db的上下文适配对象
    db := m.GetObject(Db.AdapterClass).(*Db.Adapter)

    db.Begin()
    db.Exec("INSERT INTO test (name, age) VALUES (?, ?)", "name1", 1)
    db.Exec("UPDATE test SET age=age+1 WHERE id=?", 1)
    db.Commit()
}
```



