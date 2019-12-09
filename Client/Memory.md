# Memory

Memory组件为进程内的内存缓存，使用上同memcache/redis缓存一致。

## 配置文件

```yaml
app.yaml
components:
    # 组件ID，默认为"memory"
    memory:
        # GC间隔，默认60秒
        # gcInterval: "60s"
        # 每次GC的最大key数目，默认1000
        # gcMaxItems: 1000
```

## 功能列表

```go
mm.Get(key)          // 获取key的值
mm.MGet(keys)        // 并行获取多个key的值
mm.Set(key, val)     // 设置key的值
mm.MSet(items)       // 并行设置多个key的值
mm.Add(key, val)     // 添加key的值(key存在时添加失败)
mm.MAdd(items)       // 并行添加多个key的值
mm.Del(key)          // 删除key的值
mm.MDel(keys)        // 并行删除多个key的值
mm.Exists(key)       // 判断key是否存在
mm.Incr(key1, delta) // 累加key的值
```

## 使用示例

```go
// curl -v http://127.0.0.1:8000/memory/set
func (m *MemoryController) ActionSet() {
    // 获取memory的上下文适配对象
    mm := m.GetObj(adapter.NewMemory()).(*adapter.Memory)

    // 设置用户输入值
    key := m.Context().ValidateParam("key", "test_key1").Do()
    val := m.Context().ValidateParam("val", "test_val1").Do()
    mm.Set(key, val)

    // 设置自定义过期时间
    mm.Set("test_key2", 100, 2*time.Minute)

    // 设置map值，会自动进行json序列化
    data := pgo2.Map{"f1": 100, "f2": true, "f3": "hello"}
    mm.Set("test_key3", data)

    // 并行设置多个key
    items := pgo2.Map{
        "test_key4": []int{1, 2, 3, 4},
        "test_key5": "test_val5",
        "test_key6": pgo2.Map{"f61": 11, "f62": "hello"},
    }
    mm.MSet(items)
}

// curl -v http://127.0.0.1:8000/memory/get
func (m *MemoryController) ActionGet() {
    // 获取memory的上下文适配对象
    mm := m.GetObj(adapter.NewMemory()).(*adapter.Memory)

    // 获取string
    if val := mm.Get("test_key1"); val != nil {
        fmt.Println("value of test_key1:", val.String())
    }

    // 获取int
    if val := mm.Get("test_key2"); val != nil {
        fmt.Println("value of test_key2:", val.Int())
    }

    // 获取序列化的数据
    if val := mm.Get("test_key3"); val != nil {
        var data pgo2.Map
        val.Decode(&data)
        fmt.Println("value of test_key3:", data)
    }

    // 获取多个key
    if res := mm.MGet([]string{"test_key4", "test_key5", "test_key6"}); res != nil {
        for key, val := range res {
            fmt.Printf("value of %s: %v\n", key, val.String())
        }
    }
}
```

