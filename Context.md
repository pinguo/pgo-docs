# 上下文(Context)

- 上下文存在于一个请求的生命周期中
- 包含一个请求的上下文信息(输入、输出、自定义数据)
- 继承pgo2.Object的类通过pgo2.GetObj()会自动注入当前上下文

请求结束后默认会输出请求访问日志，如果不希望输出访问日志或想输出自定义格式的访问日志可以通过设置`app.server.enableAccessLog`为false来关闭。

## 使用示例

```go
// 获取当前对象的上下文信息
ctx := this.GetContext()

// 获取请求数据
ctx.Param("p1", "")  // 获取GET/POST参数，默认空串
ctx.ParamAll()		// 获取所有GET/POST参数
ctx.Query("p2", "")  // 获取GET参数，默认空串
ctx.QueryAll()		// 获取所有GET参数
ctx.Post("p3", "")   // 获取POST参数，默认空串
ctx.PostAll()		// 获取所有POST参数
ctx.Header("h1", "") // 获取Header，默认空串
ctx.HeaderAll()		// 获取所有Header
ctx.Cookie("c1", "") // 获取Cookie，默认空串
ctx.CookieAll()		// 获取所有Cookie
ctx.ParamMap("p4")	// 获取GET/POST中p4[k1]=xx&p4[k2]=xx类型参数
ctx.QueryMap("p5")	// 获取GET中p5[k1]=xx&p5[k2]=xx类型参数
ctx.PostMap("p6")	// 获取POST中p6[k1]=xx&p6[k2]=xx类型参数
ctx.QueryArray("p7") // 获取GET/POST中p7[]=v1&p7[]=v2类型参数
ctx.ParamArray("p7") // 获取GET中p7[]=v1&p7[]=v2类型参数
ctx.PostArray("p7") // 获取POST中p7[]=v1&p7[]=v2类型参数

ctx.Path()           // 获取请求路径
ctx.ClientIp()       // 获取客户端IP

ctx.SetUserData("u1", "v1") // 设置自定义数据
ctx.UserData("u1", "")   // 获取自定义数据

// 验证请求参数
ctx.ValidateParam("p1", "").Do()    // 获取并验证GET/POST参数(有默认值)
ctx.ValidateQuery("p2").Do()        // 获取并验证GET参数(必选参数)
ctx.ValidatePost("p3").Do()         // 获取并验证POST参数(必选参数)

// 输出响应数据
ctx.SetHeader("name", "value")	// 设置响应header
ctx.SetCookie(cookie)			// 设置响应cookie
ctx.End(status, data)			// 输出状态码及数据

ctx.Debug/Info/Notice/Warn/Error/Fatal()    // 日志输出函数(带日志跟踪ID)
ctx.PushLog("key", "val")                   // 记录pushlog
ctx.Counting("key", 1, 1)                   // 记录命中记数
ctx.ProfileStart/ProfileStop/ProfileAdd()   // 记录耗时数据

// 其它
ctx.Next()			// 等待后续插件链处理完毕
ctx.Abort()			// 结束插件链处理并退出
ctx.Copy()			// 复制当前上下文
ctx.ElapseMs()	// 获取当前请求耗时
ctx.LogId()		// 获取当前请求日志ID
ctx.Process(plugins)   // 运行server 插件
ctx.Start(plugins) // 初始化context 
ctx.HttpRW(enableAccessLog bool, r *http.Request, w http.ResponseWriter) // 赋值请求上下文
```