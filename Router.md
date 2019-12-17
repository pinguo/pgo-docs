# 路由组件(Router)
Router处理URL到Controller/Action的路径映射，支持url路由和正则路由。

route和controller/action的映射规则：
- 当route等于"/" 时 ，默认查找index/index 

例如：/path/to/welcome/say-hello,/path/to/welcome/sayHello，controller类名为Controller/Path/To/WelcomeController, action方法为ActionSayHello.

## URL路由
url路由通过url直接映射，映射规则为将'/'和'-'后面的第一个字母转换成大写，例如：`/api/foo-bar/say-hello => /Api/FooBar/SayHello`。

## 正则路由
指定正则表达式来匹配路由对应的url，支持参数捕获，捕获的参数按序通过Action方法的参数进行传递。

路由规则可以通过配置文件(app.router.rules)进行配置，也可以在pgo2.Run()调用前通过代码配置。

文件配置:
```yaml
router:
    rules:
        # 前半部分为url规则，后半部分为对应的路由
        - "^/foo/all$ => /foo/index"
        # json配置文件中注意转义
        - "^/api/user/(\d+)$ => /api/user"
```

代码配置：
```go
// 添加正则路由
router := pgo2.App().Router()
router.AddRoute(`^/foo/all$`, "/foo/index")
router.AddRoute(`^/api/user/(\d+)$`, "/api/user")

pgo2.Run() // 开始服务
```
