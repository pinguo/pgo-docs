# 项目背景
传统的基于PHP的服务端多使用开源社区流行的Yii2/Laravel等MVC框架，这种模式下nginx和php-fpm是不可或缺的，整个处理流程为nginx接受用户请求，将请求转发给php-fpm管理的php工作进程，php工作进程解析并执行PHP代码，每个请求结束后PHP销毁相关资源。这种工作模式是PHPer根深蒂固的认知，然而随着业务的发展，用户请求量的增加，这种nginx+php-fpm模式的性能瓶颈很快就暴露出来了，根据线上实际业务观察，一台4核8G的虚拟机QPS很难超200，对一些长耗时的接口除了增加机器外更是毫无办法。

在这种背景下我们需要寻求一种高性能的解决方案，团队首先尝试了`swoole+yield协程+长驻进程`的方案并开发出了[php-msf](https://github.com/pinguo/php-msf)，这个框架带来了非常大的性能提升，解决了我们迫在眉睫的服务器成本问题，不过使用过程中有不少成员特别是新手提出swoole+yield协程方式难以理解、容易出错且不好调试，希望能有一种新的方便易用的解决方案。

经过充分的学习调研，我们决定引入GO语言，GO语言学习成本非常低，原生的协程处理流程与php的同步处理方式相近，考虑到技术栈切换的成本，我们新开发了应用框架`PGO`，尽量保留yii2/msf的使用习惯，使PHPer能快速熟悉新框架.

# 基准测试
主要测试PGO框架与php-yii2，php-msf，go-gin的性能差异。

说明:
- 测试机为4核8G虚拟机
- php版本为7.1.24, 开启opcache
- go版本为1.11.2, GOMAXPROCS=4
- swoole版本1.9.21, worker_num=4, reactor_num=2
- 输出均为字符串{"code": 200, "message": "success","data": "hello world"}
- 命令: ab -n 1000000 -c 100 -k 'http://target-ip:8000/welcome'

分类 | QPS | 平均响应时间(ms) |CPU
---- | ---- | ---- | -----
php-yii2 | 2715 | 36.601 | 72%
php-msf | 20053 | 4.575 | 73%
go-gin | 41798 | 2.339 | 55%
go-pgo | 33902 | 2.842 | 64%

结论:
- pgo相比yii2性能提升10倍, 对低于php7的版本性能还要翻倍。
- pgo相比msf性能提升70%, 相较于msf的yield模拟的协程，pgo协程理解和使用更简单。
- pgo相比gin性能降低19%, 但pgo内置多种常用组件，工程化做得更好，使用方式类似yii2和msf。

# 环境要求
- GO 1.10+
- Make 3.8+
- Linux/MacOS/Cygwin
- Glide 0.13+ (建议)
- GoLand 2018 (建议)

# 项目目录
规范：
- 一个项目为一个独立的目录，不使用GO全局工作空间。
- 项目的GOPATH为项目根目录，不要依赖系统的GOPATH。
- 除GO标准库外，所有外部依赖代码放入"src/vendor"。
- 项目源码文件与目录使用大写驼峰(CamelCase)形式。

```
<project>
├── bin/                # 编译程序目录
├── conf/               # 配置文件目录
│   ├── production/     # 环境配置目录
│   │   ├── app.yaml
│   │   └── params.yaml
│   ├── testing-dev/
│   ├── testing-qa/
│   ├── app.yaml        # 项目配置文件
│   └── params.yaml     # 自定义配置文件
├── makefile            # 编译打包
├── runtime/            # 运行时目录
├── public/             # 静态资源目录
├── view/               # 视图模板目录
└── src/                # 项目源码目录
    ├── Command/        # 命令行控制器目录
    ├── Controller/     # HTTP控制器目录
    ├── Lib/            # 项目基础库目录
    ├── Main/           # 项目入口目录
    ├── Model/          # 模型目录(数据交互)
    ├── Service/        # 服务目录(业务逻辑)
    ├── Struct/         # 结构目录(数据定义)
    ├── Test/           # 测试目录
    ├── vendor/         # 第三方依赖目录
    ├── glide.lock      # 项目依赖锁文件
    └── glide.yaml      # 项目依赖配置文件
```

# 依赖管理
建议使用glide做为依赖管理工具(类似php的composer)，不使用go官方的dep工具

安装(mac)：`brew install glide`

使用(调用目录为项目的src目录)：
```
glide init              # 初始化项目
glide get <pkg>         # 下载pkg并添加依赖
    --all-dependencies  # 下载pkg的所有依赖
glide get <pkg>#v1.2    # 下载指定版本的pkg
glide install           # 根据lock文件下载依赖
glide update            # 更新依赖包
```

# 快速开始

1. 拷贝makefile

    非IDE环境(命令行)下，推荐使用make做为编译打包的控制工具，从[pgo](https://github.com/pinguo/pgo)或[pgo-demo](https://github.com/pinguo/pgo-demo)工程下将makefile复制到项目目录下。
    ```sh
    make start      # 编译并运行当前工程
    make stop       # 停止当前工程的进程
    make build      # 仅编译当前工程
    make update     # 更新glide依赖(仅更新glide.yaml中的包)
    make update-all # 更新glide依赖(递归更新依赖的依赖包)
    make pgo        # 安装pgo框架到当前工程
    make init       # 初始化工程目录
    make help       # 输出帮助信息
    ```

2. 创建项目目录(以下三种方法均可)
    - 执行`make init`创建目录
    - 参见《项目目录》手动创建
    - 从[pgo-demo](https://github.com/pinguo/pgo-demo)克隆目录结构

3. 修改配置文件(conf/app.yaml)
    ```yaml
    name: "pgo-demo"
    GOMAXPROCS: 2
    runtimePath: "@app/runtime"
    publicPath: "@app/public"
    viewPath: "@app/view"
    server:
        httpAddr: "0.0.0.0:8000"
        readTimeout: "30s"
        writeTimeout: "30s"
    components:
        log:
            levels: "ALL"
            targets:
                info:
                    class: "@pgo/FileTarget"
                    levels: "DEBUG,INFO,NOTICE"
                    filePath: "@runtime/info.log"
                error:
                    class: "@pgo/FileTarget"
                    levels: "WARN,ERROR,FATAL"
                    filePath: "@runtime/error.log"
                console: {
                    class: "@pgo/ConsoleTarget"
                    levels: "ALL"
    ```

4. 安装PGO(以下两种方法均可)
    - 在项目根目录执行`make pgo`安装PGO
    - 在项目根目录执行`export GOPATH=$(pwd) && cd src && glide get github.com/pinguo/pgo`

5. 创建控制器(src/Controller/WelcomeController.go)
    ```go
    package Controller
    
    import (
        "github.com/pinguo/pgo"
        "net/http"
        "time"
    )
    
    type WelcomeController struct {
        pgo.Controller
    }
    
    func (w *WelcomeController) ActionIndex() {
        w.OutputJson("hello world", http.StatusOK)
    }
    ```

6. 注册控制器(src/Controller/Init.go)
    ```go
    package Controller
    
    import "github.com/pinguo/pgo"
    
    func init() {
        container := pgo.App.GetContainer()
        container.Bind(&WelcomeController{})
    }
    ```

7. 创建程序入口(src/Main/main.go)
    ```go
    package main
    
    import (
        _ "Controller" // 导入控制器
    
        "github.com/pinguo/pgo"
    )
    
    func main() {
        pgo.Run() // 运行程序
    }
    ```

8. 编译运行
    ```sh
    make start
    curl http://127.0.0.1:8000/welcome
    ```
