# 控制器(Controller)

- 支持HTTP(Controller)和命令行(Command)控制器
- 支持`URL`路由和`正则`路由(详见Router组件)
- 支持URL动作(Action)和RESTFULL动作(Action)
- 支持参数验证(详见ValidateXxx方法)
- 支持BeforeAction/AfterAction钩子
- 支持HandlePanic钩子，捕获未处理异常
- 提供Json,Jsonp,Data,Xml,ProtoBuf方法，方便输出各种类型数据
- 提供自定义数据类型输出方法Render

由于控制器是请求的入口，需要在main包中手动导入，为方便起见，建议不论Controller层有多深统一在Controller或Command包的init方法中注册。

HTTP控制器位于`pkg/controller`目录下，类后后缀为`Controller`。

命令控制器位于`pkg/command`目录下，类后缀为`Command`，运行命令控制器需要通过`--cmd`选项指定，例如：`bin/pgo-demo --cmd /test/index`。

## 使用示例

```go
package controller

import (
    "net/http"
    "github.com/pinguo/pgo2"
)

// 定义HTTP控制器
type WelcomeController struct {
    pgo2.Controller	// 继承于框架基类
}

// 执行action前调用的钩子，可在此执行权限验证，公共参数处理等
func (w *WelcomeController) BeforeAction(action string) {
}

// 执行action完调用的钩子(无论是否成功)，可在此执行自定义访问日志输出等
func (w *WelcomeController) AfterAction(action string) {
}

// 执行异常捕获，不建议用户自己实现
func (w *WelcomeController) HandlePanic(v interface{}) {
}

// 默认动作为index, 通过/welcome或/welcome/index调用
func (w *WelcomeController) ActionIndex() {
    data := pgo2.Map{"text": "welcome to pgo-demo", "now": time.Now()}
    w.Json(data, http.StatusOK)
}

// URL路由动作，根据url自动映射控制器及方法，不需要配置.
// url的最后一段为动作名称，不存在则为index,
// url的其余部分为控制器名称，不存在则为index,
// 例如：/welcome/say-hello，控制器类名为
// Controller/WelcomeController 动作方法名为ActionSayHello
func (w *WelcomeController) ActionSayHello() {
    ctx := w.Context() // 获取PGO请求上下文件

    // 验证参数，提供参数名和默认值，当不提供默认值时，表明该参数为必选参数。
    // 详细验证方法参见Validate.go
    name := ctx.ValidateParam("name").Min(5).Max(50).Do() // 验证GET/POST参数(string)，为空或验证失败时panic
    age := ctx.ValidateQuery("age", 20).Int().Min(1).Max(100).Do() // 只验证GET参数(int)，为空或失败时返回20
    ip := ctx.ValidatePost("ip", "").IPv4().Do() // 只验证POST参数(string), 为空或失败时返回空字符串

    // 打印日志
    ctx.Info("request from welcome, name:%s, age:%d, ip:%s", name, age, ip)
    ctx.PushLog("clientIp", ctx.ClientIp()) // 生成clientIp=xxxxx在pushlog中

    // 调用业务逻辑，一个请求生命周期内的对象都要通过GetObj()获取，
    // 这样可自动查找注册的类，并注入请求上下文(Context)到对象中。
    svc := w.GetObj(service.NewWelcome()).(*service.Welcome)

    // 添加耗时到profile日志中
    ctx.ProfileStart("Welcome.SayHello")
    svc.SayHello(name, age, ip)
    ctx.ProfileStop("Welcome.SayHello")

    data := pgo2.Map{
        "name": name,
        "age": age,
        "ip": ip,
    }

    // 输出json数据
    w.Json(data, http.StatusOK)
}

// 正则路由动作，需要配置Router组件(components.router.rules)
// 规则中捕获的参数通过动作函数参数传递，没有则为空字符串.
// eg. "^/reg/eg/(\\w+)/(\\w+)$ => /welcome/regexp-example"
func (w *WelcomeController) ActionRegexpExample(p1, p2 string) {
    data := pgo2.Map{"p1": p1, "p2": p2}
    w.Json(data, http.StatusOK)
}

// RESTFULL动作，url中没有指定动作名，使用请求方法作为动作的名称(需要大写)
// 例如：GET方法请求ActionGET(), POST方法请求ActionPOST()
func (w *WelcomeController) ActionGET() {
    w.Context().End(http.StatusOK, []byte("call restfull GET"))
}
```