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

