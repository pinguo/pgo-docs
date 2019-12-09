# 视图组件(View)

视图组件使用golang内置的`html/template`进行视图模板的渲染输出，支持通用模板(如页头、页脚)设置，视图模板文件位于`@app/view`目录下。

## 配置文件

```yaml
# 视图组件配置(app.components.view)
view:
    # 视图文件默认后缀，默认为".html"
    suffix: ".html",
    
    # 通用视图文件列表，默认为空
    commons:
        - "@view/common/header.html"
        - "@view/common/footer.html"
```

## 使用示例

```go
// 渲染并输出到stdout
pgo2.App().View().Display(os.Stdout, "hello.html", pgo2.Map{"name": "tom", "age": 25})

// 渲染并获取输出bytes
data := pgo2.App().View().Render("hello.html", pgo2.Map{"name": "tom", "age": 25})

// 控制器中渲染并输出
ctr.View("hello.html", pgo2.Map{"name": "tom", "age": 25})
```



