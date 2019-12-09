# Memcache

Memcache组件提供了memcache客户端的功能，数据在多个server间按一致性hash分布，支持连接池，支持数据的自动序列化，如果key未明确指定过期时间，则默认过期时间为1天。

## 配置文件
```yaml
app.yaml

components:
    memcache: # 组件ID，默认为"memcache"
        # KEY前缀，默认为"pgo_"
        # prefix: "pgo_"
        # 每个地址的最大空闲连接数，默认10
        # maxIdleConn: 10
        # 连接空闲时间(在这个时间内再次进行操作不需要进行探活)，默认1分钟
        # maxIdleTime: "60s"
        # 网络超时(连接、读写等)，默认1秒
        # netTimeout: "1s"
        # 服务器探活间隔(自动剔除和添加server)，默认关闭
        # probInterval: "0s"
        # 服务器地址，如果server有权重，请自行按比例添加
        servers:
            - "127.0.0.1:11211"
            - "127.0.0.1:11212"
```

## 功能列表
```go
mc.Get(key)                 // 获取key的值
mc.MGet(keys)               // 并行获取多个key的值
mc.Set(key, val)            // 设置key的值
mc.MSet(items)              // 并行设置多个key的值
mc.Add(key, val)            // 添加key的值(key存在时添加失败)
mc.MAdd(items)              // 并行添加多个key的值
mc.Del(key)                 // 删除key的值
mc.MDel(keys)               // 并行删除多个key的值
mc.Exists(key)              // 判断key是否存在
mc.Incr(key1, delta)        // 累加key的值

mc.Retrieve(cmd, key)       // 执行获取命令
mc.MultiRetrieve(cmd, keys) // 批量执行获取命令
mc.Store(cmd, item)         // 执行储存命令
mc.MultiStore(cmd, items)   // 批量执行储存命令
```

## 使用示例
```go
// curl -v http://127.0.0.1:8000/memcache/set
func (m *MemcacheController) ActionSet() {
    // 获取memcache的上下文适配对象
    mc := m.GetObject(adapter.MewMemCache()).(*adapter.MemCache)

    // 设置用户输入值
    key := m.Context().ValidateParam("key", "test_key1").Do()
    val := m.Context().ValidateParam("val", "test_val1").Do()
    mc.Set(key, val)

    // 设置自定义过期时间
    mc.Set("test_key2", 100, 2*time.Minute)

    // 设置map值，会自动进行json序列化
    data := pgo2.Map{"f1": 100, "f2": true, "f3": "hello"}
    mc.Set("test_key3", data)

    // 并行设置多个key
    items := pgo2.Map{
        "test_key4": []int{1, 2, 3, 4},
        "test_key5": "test_val5",
        "test_key6": pgo2.Map{"f61": 11, "f62": "hello"},
    }
    mc.MSet(items)
}

// curl -v http://127.0.0.1:8000/memcache/get
func (m *MemcacheController) ActionGet() {
    // 获取memcache的上下文适配对象
    mc := m.GetObj(adapter.MewMemCache()).(*adapter.MemCache)

    // 获取string
    if val := mc.Get("test_key1"); val != nil {
        fmt.Println("value of test_key1:", val.String())
    }

    // 获取int
    if val := mc.Get("test_key2"); val != nil {
        fmt.Println("value of test_key2:", val.Int())
    }

    // 获取序列化的数据
    if val := mc.Get("test_key3"); val != nil {
        var data pgo2.Map
        val.Decode(&data)
        fmt.Println("value of test_key3:", data)
    }

    // 获取多个key
    if res := mc.MGet([]string{"test_key4", "test_key5", "test_key6"}); res != nil {
        for key, val := range res {
            fmt.Printf("value of %s: %v\n", key, val.String())
        }
    }
}
```
