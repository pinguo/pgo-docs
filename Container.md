# 容器组件(Container)
Container用于类的注册创建，用于解决GO语言下不能通过字符串实例化一个类的问题。

通过Container实例化的对象会自动按序调用以下函数(如果有)：
- 构造函数(Construct), 支持任意参数
- 属性设置(SetXxx方法或导出字段)
- 初始函数(Init)

通常组件对象都是全局单例，而一个请求周期内的对象会随着请求结束而销毁，为尽量减少GC的次数和时间，后续会对请求周期的对象进行缓存。

除全局对象外，一个请求周期内的对象通常会继承pgo.Object以加入请求上下文件支持。

## 使用示例
```go
type People struct {
    // 继承自pgo.Object可增加上下文支持，
    // 由于组件是全局对象，没有请求上下文，
    // 所以组件是不能继承自pgo.Object的。
    pgo.Object

    name    string
    age     int
    sex     string
}

// 可选构造函数，用于设置初始值
func (p *People) Construct() {
    p.name = "unknown"
    p.age  = 0
    p.sex  = "unknown"
}

// 可选初始函数，对象创建完成回调
func (p *People) Init() {
    fmt.Printf("people created, name:%s age:%d sex:%s\n", p.name, p.age, p.sex)
}

// 可选设置函数，根据配置自动调用
func (p *People) SetName(name string) {
    p.name = name
}

func (p *People) SetAge(age int) {
    p.age = age
}

func (p *People) SetSex(sex string) {
    p.sex = sex
}

// 注册类，通常将init方法放入包中的Inig.go文件中
func init() {
    container := pgo.App.GetContainer() // 获取容器对象
    container.Bind(&People{}) // 注册类模板对象
}

// 获取People的新对象
func (t *TestController) ActionTest() {
    // 方法1: 通过类字符串获取
    p1 := t.GetObject("People").(*People)

    // 方法2: 如果构造函数定义有参数，可以传递构造参数
    p2 := t.GetObject("People", arg1, arg2).(*People)

    // 方法3: 指定配置属性(通常用于从配置文件生成组件)
    conf := map[string]interface{} {
        "class" "People",   // 配置必须包含class字段
        "name": "zhang san",
        "age": 30,
        "sex": "male",
    }
    p3 := t.GetObject(conf).(*People)
}
```
