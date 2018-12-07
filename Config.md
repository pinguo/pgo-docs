# 配置组件(Config)
用于管理应用程序的配置文件，支持全局配置和环境配置，支持yaml和json配置文件，推荐使用yaml配置文件以支持注释等特性。

用户可以自行添加特定文件后缀的配置文件解析器，参见Config.AddParser().

说明：
- 配置文件根目录为`@app/conf`，根目录下支持环境目录，如：production, dev, qa等
- 应用配置为`app.yaml`，可任意添加自定义配置文件， 如：`params.yaml`
- 运行程序时通过`--env production`指定环境目录为`production`
- 环境目录中的配置会递归合并到全局配置中
- 配置文件支持环境变量，格式`${envName||default}`，当envName不存在时使用default
- 配置文件中的路径及类名支持别名字符串

## 使用示例
```go
cfg := pgo.App.GetConfig() // 获取配置对象
name := cfg.GetString("app.name", "demo") // 获取String，不存在返回"demo"
procs := cfg.GetInt("app.GOMAXPROCS", 2) // 获取Integer, 不存在返回2
price := cfg.GetFloat("params.goods.price", 0) // 获取Float, 不存在返回0
enable := cfg.GetBool("params.detect.enable", false) // 获取Bool, 不存在返回false

// 除基本类型外，通过Get方法获取原始配置数据，需要进行类型转换
plugins, ok := cfg.Get("app.servers.plugins").([]interface{}) // 获取数组
log, ok := cfg.Get("app.conponents.log").(map[string]interface{}) // 获取对象

```
