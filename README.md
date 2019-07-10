# PGO参考文档
PGO应用框架即"Pinguo GO application framework"，是Camera360广告服务端团队研发的一款简单、高性能、组件化的GO应用框架。受益于GO语言高性能与原生协程，业务从php+yii2升级到PGO后，线上表现单机处理能力提高10倍。

本文档介绍PGO核心组件的概念、使用与原理，再结合[pgo-demo](https://github.com/pinguo/pgo-demo)项目，开发者就能够使用PGO轻松地开发出高性能的WEB应用程序。

# 目录
* [框架概述(Overview)](Overview.md)
* [核心应用(Application)](Application.md)
* [配置组件(Config)](Config.md)
* [容器组件(Container)](Container.md)
* [服务器组件(Server)](Server.md)
* [路由组件(Router)](Router.md)
* [日志组件(Log)](Log.md)
* [状态码组件(Status)](Status.md)
* [国际化组件(I18n)](I18n.md)
* [视图组件(View)](View.md)
* [上下文(Context)](Context.md)
* [控制器(Controller)](Controller.md)
* [插件(Plugin)](Plugin.md)
* [客户端(Client)](Client/Overview.md)
    * [Db](Client/Db.md)
    * [Http](Client/Http.md)
    * [MaxMind](Client/MaxMind.md)
    * [Memcache](Client/Memcache.md)
    * [Memory](Client/Memory.md)
    * [Mongo](Client/Mongo.md)
    * [Redis](Client/Redis.md)
* [单元测试](UnitTesting.md)
