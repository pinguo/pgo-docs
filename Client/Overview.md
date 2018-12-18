# Overview

Client目录下包含一些常用的第三方客户端包的组件封装，如Db组件(mysql)、Http组件、Redis组件、Mongo组件等。

一个Client组件至少包含以下三个文件：

1. `Init.go`注册Client和Adapter到对象容器，定义常量等。
2. `Client.go`组件对象封装，组件对象是全局单例，第一次使用到时自动构造和初始化。
3. `Adapter.go`组件的上下文适配对象，在请求中应该使用这个适配对象，Adapter对象仅存在于一个请求的上下文生命周期中，注入了上下文的日志和Profile分析功能。
