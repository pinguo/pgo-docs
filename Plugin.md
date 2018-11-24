# 插件(Plugin)


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
