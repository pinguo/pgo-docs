# 核心应用(Application)
Application是PGO2的应用对象，其全局实例pgo2.App()是框架初始化的第一个对象，通过pgo2.App()可以访问到框架的所有组件。

## 配置文件
Application的配置文件即是整个程序的配置文件(configs/app.yaml)，通过配置文件可以对Application及所有其它组件进行配置。文件格式：
```yaml
# 应用名称，默认为可执行程序名称
name: "app-name"

# GO线程数，默认为CPU核心数
GOMAXPROCS: 2

# 运行时目录，默认: @app/runtime
runtimePath: "@app/runtime"

# 静态资源目录，默认: @app/public
publicPath: "@app/public"

# 视图模板目录，默认: @app/view
viewPath: "@app/view"

# 服务器配置
server:
    # http服务地址，若httpAddr和httpsAddr都为空，则使用"0.0.0.0:8000"
    httpAddr: "0.0.0.0:8000"

    # debug服务地址，/debug/pprof及服务状态输出，默认为空
    debugAddr: "0.0.0.0:8100"

    # https服务地址，需要同时指定crtFile和keyFile，默认为空
    httpsAddr: "0.0.0.0:8443"

    # https证书路径
    crtFile: "@app/conf/site.crt"

    # https私钥路径
    keyFile: "@app/conf/site.key"

    # 插件组件列表，不指定时默认支持gzip, 指定该字段但列表为空时，表示不使用任何插件
    plugins: ["gzip"]

# 组件配置
components:
    # 日志组件
    log: {...}
    # 其它组件
```

## 初始化流程：
1. 导入PGO2
    - 在main包中通过`import "github.com/pinguo/pgo2"`导入pgo2
    - 执行pgo2的init函数，构造并初始化App的属性
    - 初始化config, container, server三个核心组件
    - 添加核心组件的默认配置
    - 注册路由、日志等核心组件
2. 调用App及其组件的方法，定制化各种组件，如添加自定义路由规则等，通常这不需要执行，应该通过配置文件进行配置。
3. 调用pgo2.Run()，启动http服务或处理command命令

## 组件
每个组件都是全局单例对象，在第一次使用时自动构造、配置和初始化，通常在配置文件中指定组件配置。
也可以在main.go中自定义

示例：

```yaml
# 日志组件配置(app.components.log)，
# 核心组件的class由框架设置，eg. log组件为"@pgo2/Log"，
# 核心组件通过框架提供的方法获取，eg. log := pgo2.App().GetLog()
log:
    levels: "ALL"
    traceLevels: "DEBUG"
    chanLen: 1000
    flushInterval: "60s"
    targets:
        info:
            levels: "DEBUG,INFO,NOTICE"
            filePath: "@runtime/info.log"
            maxLogFile: 10
            rotate: "daily"
        error: 
            levels: "WARN,ERROR,FATAL"
            filePath: "@runtime/error.log"
            maxLogFile: 10
            rotate: "daily"
```

```yaml
# redis组件(app.components.redis)，
# 获取非核心组件需要进行类型转换，例如：
# redis := pgo2.App().Component("redis", redis.New).(*Redis.Client)
redis:
    prefix: "pgo2_"
    password: ""
    db: 0
    maxIdleConn: 10
    maxIdleTime: "60s"
    netTimeout: "1s"
    probInterval: "0s"
    servers:
        - "127.0.0.1:6379"
        - "127.0.0.1:6380"
```

框架定义的核心组件如下：
```go
map[string]string{
    "router": "@pgo2/Router",        // 路由组件
    "log":    "@pgo2/Log",           // 日志组件
    "status": "@pgo2/Status",        // 状态码组件
    "i18n":   "@pgo2/I18n",          // 国际化组件
    "view":   "@pgo2/View",          // 视图组件
    "gzip":   "@pgo2/Gzip",          // Gzip组件
    "file":   "@pgo2/File",          // 静态文件组件

}
```

## 别名字符串
框架支持别名字符串，可以用在代码和配置文件中，PGO定义的别名如下：
- `@app` 项目根目录绝对路径
- `@runtime` 项目运行时目录绝对路径
- `@view` 项目视图模板目录绝对路径
- `@pgo2` PGO2框架import路径

使用SetAlias/GetAlias设置和解析别名字符串，例如：
`GetAlias("@runtime/info.log")`会将@runtime替换成实际的运行目录

## 命令行参数
- `--env production`, 指定程序的环境配置目录，默认为develop
- `--cmd /foo/bar`, 指定程序为命令行模式，并运行指定命令，默认为WEB模式
- `--base /base/path`, 指定基础目录，默认为项目根目录，通常在单测时需要指定
- `--cmdList`, 显示所有命令，包括cmd参数列表 


