# 状态码组件(Status)

状态码组件用于集中管理接口的响应状态码，并且支持多语言。PGO2建议接口只处理成功的响应，所有异常的响应通过perror.New(status)抛出来，由框架自动查询状态码及响应消息，并根据配置处理消息的国际化，以简化流程。

状态码查找顺序：

1. 指定status如果在配置文件中有mapping设置，则使用mapping中对应的message

2. 如果没有mapping配置，但提供的默认的message，使用默认message

3. 如果都没有但状态码是http标准状态码，则使用http标准状态消息

4. 如果以上情况都不满足，则抛出异常

如果`status.useI18n`为true，开启国际化输出，则会检查请求的`Accept-Language`输出message在i18n组件中对应的语言配置

## 配置文件

```yaml
status:
    # 是否处理消息国际化，默认为false
    # 国际化组件参见I18n组件
    useI18n: false,
    
    # 状态码及消息映射
    mapping:
        1000: "Success"
        1001: "System Error"
        1002: "Verify Sign Error"
```

## 使用示例

```go
// 成功输出
func (t *WelcomeController) ActionIndex() {
    // 使用标准http标准状态码
    t.Json("hello world", http.StatusOK)
    
    // 使用配置文件中的自定义状态码
    t.Json("hello world", 1000)
}

// 异常输出，由于框架处理处理
func (t *WelcomeController) ActionError() {
    // 通过抛出异常的方式由框架自动处理状态码
    panic(perror.New(1001))
    
    // 可以覆盖配置中的消息映射
    panic(perror.New(1002, "another error message"))
}
```

