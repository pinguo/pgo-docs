# Memcache

Memcache组件提供了memcache客户端的功能，数据在多个server间按一致性hash分布，支持连接池，支持数据的自动序列化，如果key未明确指定过期时间，则默认过期时间为1天。

## 配置文件

```yaml
# 组件ID，默认为"memcache"
memcache:
	# 组件类名称
    class: "@pgo/Client/Memcache/Client"
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
mc.Get(key)          // 获取key的值
mc.MGet(keys)        // 并行获取多个key的值
mc.Set(key, val)     // 设置key的值
mc.MSet(items)       // 并行设置多个key的值
mc.Add(key, val)     // 添加key的值(key存在时添加失败)
mc.MAdd(items)       // 并行添加多个key的值
mc.Del(key)          // 删除key的值
mc.MDel(keys)        // 并行删除多个key的值
mc.Exists(key)       // 判断key是否存在
mc.Incr(key1, delta) // 累加key的值
```

## 使用示例