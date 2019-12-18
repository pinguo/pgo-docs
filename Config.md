# 配置组件(Config)
用于管理应用程序的配置文件，支持全局配置和环境配置，支持yaml和json配置文件，推荐使用yaml配置文件以支持注释等特性。
用户可以自定义Config组件
pgo2.App().SetConfig(config config.IConfig) // (可选)配置组件  

pgo2.App().Config().AddParser(ext string, parser IConfigParser) // (可选)自定义配置解析器 根据后缀自动选择

用户可以自行添加特定文件后缀的配置文件解析器，参见Config.AddParser().

说明：
- 配置文件根目录为`@app/configs`，根目录下支持环境目录，如：production, dev, qa等
- 应用配置为`app.yaml`，可任意添加自定义配置文件， 如：`params.yaml`
- 运行程序时通过`--env production`指定环境目录为`production`
- 环境目录中的配置会递归合并到全局配置中
- 配置文件支持环境变量，格式`${envName||default}`，当envName不存在时使用default
- 配置文件中的路径及类名支持别名字符串

## 配置目录

典型的配置目录结构如下：

```sh
configs/
    ├── production/     # 生产环境配置目录
    │   ├── app.yaml	# 生产环境app配置(递归合并到基础配置中)
    │   └── params.yaml
    ├── testing-dev/    # 开发测试配置目录
    ├── testing-qa/     # QA测试配置目录
    ├── development/	# 本地开发配置目录
    ├── app.yaml        # 项目基础配置文件
    └── params.yaml     # 自定义基础配置文件(自定义文件可任意添加)
```

## 使用示例

```go
cfg := pgo2.App().Config() // 获取配置对象
name := cfg.GetString("app.name", "demo") // 获取String，不存在返回"demo"
procs := cfg.GetInt("app.GOMAXPROCS", 2) // 获取Integer, 不存在返回2
price := cfg.GetFloat("params.goods.price", 0) // 获取Float, 不存在返回0
enable := cfg.GetBool("params.detect.enable", false) // 获取Bool, 不存在返回false
sliceName := cfg.GetSliceString("app.name") // 获取[]String，不存在返回nil
sliceProcs := cfg.GetSliceInt("app.GOMAXPROCS") // 获取[]Integer, 不存在返回nil
slicePrice := cfg.GetSliceFloat("params.goods.price") // 获取[]Float, 不存在返回nil
sliceEnable := cfg.GetSliceBool("params.detect.enable") // 获取[]Bool, 不存在返回nil

// 除基本类型外，通过Get方法获取原始配置数据，需要进行类型转换
plugins, ok := cfg.Get("app.servers.plugins").([]interface{}) // 获取数组
log, ok := cfg.Get("app.conponents.log").(map[string]interface{}) // 获取对象

```
