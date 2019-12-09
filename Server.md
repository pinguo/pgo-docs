# 服务器组件(Server)
Server封装了命令行和HTTP的处理逻辑，实现了http.Handler接口和pgo2.IPlugin接口，主要的逻辑实现在ServeCMD, ServeHTTP, HandleRequest三个函数中。

Server支持SIGTERM/SIGINT信号的平滑关闭，默认每1分钟会输出程序状态到INFO日志中。

## 配置说明
```yaml
server:
    # http服务地址，当httpAddr和httpsAddr都为空时，默认为"0.0.0.0:8000"
    httpAddr: "0.0.0.0:8000"

    # debug服务地址，用于/debug/pprof及服务状态输出，默认为空
    debugAddr: "0.0.0.0:8100"

    # https服务地址，需同时配置crtFile/keyFile，默认为空
    httpsAddr: "0.0.0.0:8443"

    # https证书路径，默认为空
    crtFile: "@app/conf/site.crt"

    # https私钥路径，默认为空
    keyFile: "@app/conf/site.key"

    # 最大http头字节数，默认1MB
    # maxHeaderBytes: 1024000

    # 请求读取超时，默认30s
    # readTimeout: "30s"

    # 请求发送超时，默认30s
    # writeTimeout: "30s"

    # 写状态日志间隔，默认60s
    # statsInterval: "60s"

    # 是否输出访问日志，默认true
    # enableAccessLog: true

    # 插件列表，默认["gzip"]
    # plugins: ["gzip"]
    
    # 提交body 限制，默认为go限制
    # maxPostBodySize:int64 
    
```

## 插件说明
插件必须实现pgo2.IPlugin接口，插件配置为组件，并在server.plugins中添加，添加顺序为插件的调用顺序。

当不指定plugins时，插件列表默认包含gzip，若指定了plugins配置，但列表为空时，没有插件生效。

插件只对http/https服务生效，command和debug服务不生效。

插件针对全局所有请求生效，若只需要处理某些特定url的请求，推荐在Controller.BeforeAction钩子中实现。

