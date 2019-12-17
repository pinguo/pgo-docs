# 日志组件(Log)

日志提供常用的分级(DEBUG/INFO/NOTICE/WARN/ERROR/FATAL)日志功能，并且增加了追踪ID, pushlog, profile, counting等功能 。

目前内置控制台(ConsoleTarget)和文件(FileTarget)两种输出方式，满足绝大部分使用场景，用户可以添加自定义的格式化类，也可以自行实现输出目标(Target)，以满足特殊定制化的需求。

在没有任何配置的情况下，日志组件会将所有级别的日志输出到控制台。

## 自定义
## pgo2.App().Log().SetTarget(name string, target ITarget) // (可选)自定义日志目标类

## pgo2.App().Log().Target.SetFormatter(format IFormatter) // (可选) 自定义日志格式

## 配置文件
```yaml
# app.components.log
log:
    # 总级别开关，默认为ALL
    levels: "ALL"

    # 追加trace的级别，默认为DEBUG
    # traceLevels: "DEBUG"

    # 日志管道缓冲长度，默认为1000
    # chanLen: 1000

    # 日志冲刷周期，默认60s
    # flushInterval: "60s"

    # 输出目标，为空时输出到控制台
    targets:
        # 信息日志
        info:
            # 文件输出
            name: "file"

            # 处理的日志级别
            levels: "DEBUG,INFO,NOTICE"

            # 日志文件路径，支持别名
            filePath: "@runtime/info.log"

            # 保留历史日志文件个数，默认10
            # maxLogFile: 10

            # 最大缓存日志字节数，默认10MB
            # maxBufferBytes: 10485760

            # 最大缓存日志行数，默认10000
            # maxBufferLine: 10000

            # 日志回滚策略(none/hourly/daily), 默认按天回滚
            # rotate: "daily"

        # 错误日志
        error:
            name: "file"
            levels: "WARN,ERROR,FATAL"
            filePath: "@runtime/error.log"
            maxLogFile: 10
            rotate: "daily"

        # 控制台日志
        console:
            name: "console"
            levels: "ALL"
```

## 全局日志

全局日志对象通过`pgo2.GLogger()`获取，全局日志对象只在没有上下文的组件中使用，通常建议在组件中使用panic将错误信息抛出来，由相应的请求对象来记录日志，这样会带上请求ID，方便调试。

示例：
```go
pgo2.GLogger().Debug("debug log")
pgo2.GLogger().Info("info log, timeRun:%s", pgo2.TimeRun().String())
pgo2.GLogger().Warn("warn log, err:%s", err.Error())
pgo2.GLogger().Error("error log, err:%s", err.Error())
```

## 上下文日志
所有请求生命周期内的对象都应该通过上下文来记录日志，这样整个请求生命周期内的日志都有同一个日志ID，并且会带上pushlog, profile, counting信息，极大地方便的请求跟踪和调试。

示例：
```go
ctx := this.Context()
ctx.Info("info log in context, args:%v", args)
ctx.Error("error log in context, err:%s", err.Error())

// 在访问日志中生成key1=value1的pushlog
ctx.PushLog("key1", "value1")

// 记录耗时，结果输出到访问日志的profile[]中
ctx.ProfileStart("tag1")
// run business logic
ctx.ProfileStop("tag1")

// 使用及缓存等的计数，结果输出到访问日志的counting[]中
ctx.Counting("tag1", 1, 1)
```
