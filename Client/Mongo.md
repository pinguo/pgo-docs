# Mongo

Mongo组件是对`github.com/globalsign/mgo`的封装，更多文档请参见[mgo](https://github.com/globalsign/mgo)。

## 配置文件

```yaml
app.yaml
components:
    # 组件ID，默认为"mongo"
    mongo:
        # mongo地址
        dsn: "mongodb://host1:port1/[db][?options]"
        # 连接超时，默认1秒
        # connectTimeout: "1s"
        # 读取超时，默认10秒
        # readTimeout: "10s"
        # 写入超时，默认10秒
        # writeTimeout: "10s"
```

其中`dsn`配置字段参考[mgo.Dial()](https://godoc.org/github.com/globalsign/mgo#Dial)。默认的dsn参数：

| 参数           | 默认           | 说明                                       |
| -------------- | -------------- | ------------------------------------------ |
| replicaSet     | 空             | 复本集名称                                 |
| connect        | replicaSet     | 连接方式(direct\|replicaSet)               |
| maxPoolSize    | 100            | 连接池最大连接数                           |
| minPoolSize    | 1              | 连接池最小连接数                           |
| maxIdleTimeMS  | 300000         | 连接最大空闲毫秒时间                       |
| ssl            | false          | 是否通过ssl连接                            |
| j              | false          | 是否等待journal写入                        |
| wtimeoutMS     | 10000          | 写操作超时毫秒时间                         |
| readPreference | readPreference | 读取优先选项(readPreference\|primary\|...) |

## 功能列表

```go
NewMongo(db, coll string, componentId ...string) // 对象 this.GetObject(adapter.NewMongo(db, coll)).(adapter.IMongo)/(*adapter.Mongo)
NewMongoPool(ctr iface.IContext, args ...interface{}) // 对象池 this.GetObjectPool(adapter.NewMongoPool,db, coll)).(adapter.IMongo)/(*adapter.Mongo)
mongo.FindOne()        // 查找满足条件的单个文档
mongo.FindAll()        // 查找满足条件的所有文档
mongo.FindAndModify()  // 查找并更新
mongo.FindDistinct()   // 查找不同值
mongo.InsertOne()      // 插入一个文档
mongo.InsertAll()      // 插入多个文档
mongo.UpdateOne()      // 更新满足条件的单个文档
mongo.UpdateAll()      // 更新满足条件的所有文档
mongo.UpdateOrInsert() // 更新文档或不存在时插入文档
mongo.DeleteOne()      // 删除满足条件的单个文档
mongo.DeleteAll()      // 删除满足条件的所有文档
mongo.Count()          // 获取文档计数
mongo.PipeOne()        // 进行pipe计算并获取第一个结果文档
mongo.PipeAll()        // 进行pipe计算并获取所有结果文档
mongo.MapReduce()      // 进行mapreduce计算并获取所有结果
```

## 使用说明

```go
// curl -v http://127.0.0.1:8000/mongo/insert
func (m *MongoController) ActionInsert() {
    // 获取mongo的上下文适配对象
    mongo := m.GetObj(adapter.NewMongo(), "test", "test").(*adapter.Mongo)

    // 通过map插入
    doc1 := pgo2.Map{"f1": "val1", "f2": true, "f3": 99}
    err := mongo.InsertOne(doc1)
    fmt.Println("insert one doc1:", err)

    // 通过bson.M插入
    doc2 := bson.M{"f1": "val2", "f2": false, "f3": 10}
    err = mongo.InsertOne(doc2)
    fmt.Println("insert one doc2:", err)

    // 通过struct插入
    doc3 := struct {
        F1 string `bson:"f1"`
        F2 bool   `bson:"f2"`
        F3 int    `bson:"f3"`
    }{"val3", false, 6}
    err = mongo.InsertOne(doc3)
    fmt.Println("insert one  doc3:", err)

    // 批量插入
    docs := []interface{}{
        bson.M{"f1": "val4", "f2": true, "f3": 7},
        bson.M{"f1": "val5", "f2": false, "f3": 8},
        bson.M{"f1": "val6", "f2": true, "f3": 9},
    }
    err = mongo.InsertAll(docs)
    fmt.Println("insert all docs:", err)
}

// curl -v http://127.0.0.1:8000/mongo/update
func (m *MongoController) ActionUpdate() {
    // 获取mongo的上下文适配对象
    mongo := m.GetObj(adapter.NewMongo(), "test", "test").(*adapter.Mongo)

    // 更新单个文档
    query := bson.M{"f1": "val1"}
    update := bson.M{"$inc": bson.M{"f3": 2}}
    err := mongo.UpdateOne(query, update)
    fmt.Println("update one f1==val1:", err)

    // 更新多个文档
    query = bson.M{"f3": bson.M{"$gte": 7}}
    update = bson.M{"$set": bson.M{"f2": false}}
    err = mongo.UpdateAll(query, update)
    fmt.Println("update all f3>=7:", err)

    // 更新或插入
    query = bson.M{"f1": "val10"}
    update = bson.M{"$inc": bson.M{"f3": 2}}
    err = mongo.UpdateOrInsert(query, update)
    fmt.Println("update or insert f1==val10:", err)
}

// curl -v http://127.0.0.1:8000/mongo/query
func (m *MongoController) ActionQuery() {
    // 获取mongo的上下文适配对象
    mongo := m.GetObj(adapter.NewMongo(), "test", "test").(*adapter.Mongo)

    // 查询单个文档(未指定结果类型，结果为bson.M)
    var v1 interface{}
    err := mongo.FindOne(bson.M{"f1": "val1"}, &v1)
    fmt.Println("query one f1==val1:", v1, err)

    // 查询单个文档(结果类型为map)
    var v2 pgo2.Map
    err = mongo.FindOne(bson.M{"f1": "val2"}, &v2)
    fmt.Println("query one f1==val2:", v2, err)

    // 查询单个文档(结果类型为struct)
    var v3 struct {
        Id bson.ObjectId `bson:"_id"`
        F1 string        `bson:"f1"`
        F2 bool          `bson:"f2"`
        F3 int           `bson:"f3"`
    }
    err = mongo.FindOne(bson.M{"f1": "val3"}, &v3)
    fmt.Println("query one f1==val3:", v3, err)

    // 查询多个文档(指定结果为map)
    var docs []pgo2.Map
    err = mongo.FindAll(bson.M{"f3": bson.M{"$gte": 6}}, &docs)
    fmt.Println("query all f3>=6:", docs, err)
}
```

