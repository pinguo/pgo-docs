# 插件(Plugin)
- configs配置插件,只能配置内置的插件`file`和`gzip`
```
app.yaml:

server:
    plugins:
        - "gzip"
        - "file"
```
- 手动
```
pgo2.App().Server().AddPlugin(v IPlugin) // 增加插件
```
插件用于扩展应用的功能，pgo2.Server本身就是一个插件，应用初始化时，会将pgo2.App().Server放到插件列表的最后一个，插件的执行顺序为`app.server.plugins`的定义顺序，插件做为一个组件定义，假设定义了两个插件`app.server.plugins=["p1", "p2"]`，那么执行顺序为：

1. p1.HandleRequest(ctx)
2. p2.HandleRequest(ctx)
3. server.HandleRequest(ctx)

PGO2内置插件只有`file`和`gzip`两个插件，file插件用于处理@public目录下的静态文件(必须带后缀)，gzip插件用于对输出进行gzip压缩。默认启用gzip插件，当需要禁用所有插件时，设置`app.server.plugins = []`。


## 使用示例：

```go
type DemoPlugin struct {
    // 插件属性
}

// 处理服务请求
func (d *DemoPluin) HandleRequest(ctx *Context) {
    // 检查是否需要该插件处理，不需要时直接返回
    // if !d.shouldApply() {
    //    return
    // }

    // 可调用Abort阻止后续插件的执行
    // ctx.Abort()

    // 如果不需要关心后续插件的执行情况，处理完业务后直接返回
    // return

    // 其它插件处理开始前的逻辑

    // 调用后续插件并等待后续插件全部执行完毕
    ctx.Next()

    // 其它插件处理完毕后的逻辑
}
```
