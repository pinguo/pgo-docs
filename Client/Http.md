# Http

HTTP组件是对GO内置net/http包的Client组件封装，增加组件配置、Profile、并发请求的支持。

> 注意：go的http transport默认会对每个host缓存一定数量的连接以加快访问速度，因此对http.Transport不要随用随建而应该复用http.Transport，否则会泄漏大量未关闭连接以及连接上poll协程。

## 配置文件

```yaml
# 组件ID，默认为http，http做为核心组件，一般不用自行设置
http:
    # 组件类名称，
    class: "@pgo/Client/Http/Client"
    # 是否验证https证书，默认否
    verifyPeer: false
    # 默认User-Agent
    userAgent: "PGO Framework"
    # 默认超时时间
    timeout: "10s"
```

## 功能列表

```go
httpClient.Get()     // 执行GET操作
httpClient.Post()    // 执行POST操作
httpClient.Do()      // 执行自定义请求
httpClient.DoMulti() // 执行并行请求
```

## 使用示例

```go
// curl -v http://127.0.0.1：8000/http-client/send-query
func (h *HttpClientController) ActionSendQuery() {
    // 获取http的上下文适配对象
    httpClient := h.GetObject(Http.AdapterClass).(*Http.Adapter)

    // 简单GET请求
    url := "http://127.0.0.1:3000/get.php"
    if res := httpClient.Get(url, nil); res != nil {
        defer res.Body.Close()
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Println("content 1:", string(content))
    }

    // 带参数GET请求
    params := pgo.Map{"p1": "v1", "p2": 10, "p3": 9.9}
    if res := httpClient.Get(url, params); res != nil {
        defer res.Body.Close()
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Println("content 2:", string(content))
    }

    // 自定义cookie和header GET请求
    option := Http.Option{}
    option.SetCookie("c1", "cv1")
    option.SetHeader("h1", "hv1")
    if res := httpClient.Get(url, nil, &option); res != nil {
        defer res.Body.Close()
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Println("content 3:", string(content))
    }
}

// curl -v http://127.0.0.1:8000/http-client/send-form
func (h *HttpClientController) ActionSendForm() {
    // 获取http的上下文适配对象
    httpClient := h.GetObject(Http.AdapterClass).(*Http.Adapter)

    // 发送POST请求
    url := "http://127.0.0.1:3000/form.php"
    form := pgo.Map{"p1": "v1", "p2": 10, "p3": 9.9}
    if res := httpClient.Post(url, form); res != nil {
        defer res.Body.Close()
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Println("content 1:", string(content))
    }
}

// curl -v http://127.0.0.1:8000/http-client/send-file
func (h *HttpClientController) ActionSendFile() {
    // 获取http的上下文适配对象
    httpClient := h.GetObject(Http.AdapterClass).(*Http.Adapter)

    // 上传文件的POST请求
    url := "http://127.0.0.1:3000/file.php"
    body := bytes.Buffer{}
    writer := multipart.NewWriter(&body)

    // 创建文件form
    formFile, _ := writer.CreateFormFile("form_file", "test.png")

    // 读取文件内空填充表单
    fileHandle, _ := os.Open("test.png")
    io.Copy(formFile, fileHandle)
    fileHandle.Close()

    // 结束表单构造
    option := Http.Option{}
    option.SetHeader("Content-Type", writer.FormDataContentType())
    writer.Close() // 发送前一定要关闭writer以写入结尾

    // 发送表单，接收响应
    if res := httpClient.Post(url, &body, &option); res != nil {
        defer res.Body.Close()
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Println("content 1:", string(content))
    }
}

// curl -v http://127.0.0.1:8000/http-client/multi-request
func (h *HttpClientController) ActionMultiRequest() {
    // 获取http的上下文适配对象
    httpClient := h.GetObject(Http.AdapterClass).(*Http.Adapter)

    req1, _ := http.NewRequest("GET", "http://127.0.0.1:3000/get1.php", nil)
    req2, _ := http.NewRequest("GET", "http://127.0.0.1:3000/get2.php", nil)
    req3, _ := http.NewRequest("GET", "http://127.0.0.1:3000/get3.php", nil)
    req4, _ := http.NewRequest("GET", "http://127.0.0.1:3000/get4.php", nil)

    // 并行请求多个url
    requests := []*http.Request{req1, req2, req3, req4}
    responses := httpClient.DoMulti(requests)

    for k, res := range responses {
        content, _ := ioutil.ReadAll(res.Body)
        fmt.Printf("content of response %d: %s\n", k, string(content))
    }
}
```

