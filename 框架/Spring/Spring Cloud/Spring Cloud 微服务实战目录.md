Spring Cloud 微服务实战

> 基础知识

⭐什么是微服务架构
	与单体服务的区别
	如何实施微服务

⭐为什么选择 Spring Cloud

Spring Cloud 简介

> 微服务构建

框架简介

⭐快速入门
	项目构建与解析
	实现 RESTful API

⭐配置详解
	配置文件
	自定义参数
	参数引用
	使用随机数
	命令行参数
	多环境配置
	加载顺序

监控与管理
	初识 actuator
	原生端点

> 服务治理 Spring Cloud Eureka

⭐服务治理
	Netfix Eureka
	搭建服务注册中心
	注册服务提供者
	高可用注册中心
	服务发现与消费

⭐Eureka 详解
	基础架构
	服务治理机制
	源码分析

⭐配置详解
	服务注册类配置
	服务实例类配置

> 客户端负载均衡 Spring Cloud Ribbon

客户端负载均衡

⭐RestTemplate 详解
	GET 请求
	POST 请求
	PUT 请求
	DELETE 请求

源码分析
	负载均衡器
	负载均衡策略

⭐配置详解
	自动化配置
	Camden 版本对 RibbonClient 配置的优化
	参数配置
	与 Eureka 结合

> 服务容错保护 Spring Cloud Hystrix

快速入门

⭐原理分析
	工作流程
	断路器原理
	依赖隔离

⭐使用详解
	创建请求命令
	自定义服务降级
	异常处理
	命令名称、分组以及线程池划分
	请求缓存
	请求合并

⭐属性详解
	Command 属性
	collapser 属性
	threadPool 属性

Hystrix 仪表盘

Turbine 集群监控
	构建监控聚合服务
	与消息代理结合

> 声明式服务调用 Spring Cloud Fegin

快速入门

参数绑定

继承特性

⭐Ribbon 配置
	全局配置
	指定服务配置
	重试机制

⭐Hystrix 配置
	全局配置
	禁用 Hystrix
	指定命令配置
	服务降级配置

其他配置
	请求压缩
	日志配置

> API 网关服务 Spring Cloud Zuul

⭐快速入门
	构建网关
	请求路由
	请求过滤

⭐路由详解
	传统路由配置
	服务路由配置
	服务路由的默认规则
	自定义路由的映射规则
	路径匹配
	路由前缀
	本地跳转
	Cookie 与头信息
	Hystrix 和 Ribbon 支持

⭐过滤器详解
	过滤器
	请求声明周期
	核心过滤器
	异常处理
	禁用过滤器

⭐动态加载
	动态路由
	动态过滤器

> 分布式配置中心 Spring Cloud Config

⭐快速入门
	构建配置中心
	配置规则详解
	客户端配置映射

⭐服务端详解
	基础架构
	Git 配置仓库
	SVN 配置仓库
	本地仓库
	本地文件系统
	健康监测
	属性覆盖
	安全保护
	加密解密
	高可用配置

⭐客户端详解
	URI 指定配置中心
	服务化配置中心
	失败快速响应与重试
	获取远程配置
	动态刷新配置

> 消息总线 Spring Cloud Bus

消息代理

RabbitMQ 实现消息总线
	基本概念
	安装与使用
	快速入门
	整合 Spring Cloud Bus
	原理分析
	指定刷新范围
	架构优化
	RabbitMQ 配置

Kafka 实现消息总线
	Kafka 简介
	快速入门整合 Spring Cloud Bus
	Kafka 配置

深入理解
	源码分析
	其他消息代理的支持

> 消息驱动的微服务 Spring Cloud Stream

快速入门

核心概念
	绑定器
	发布-订阅模式
	消费组
	消息分区

使用详解
	开启绑定功能
	绑定消息通道
	消息生产与消费
	响应式编程
	消费组与消息分区
	消息类型

绑定器详解
	绑定器 SPI
	自动化配置
	多绑定器配置
	RabbitMQ 与 Kafka 绑定器

配置详解
	基础配置
	绑定通道配置
	绑定器配置

> 分布式服务跟踪 Spring Cloud Sleuth

⭐快速入门
	准备工作
	实现跟踪

跟踪原理

抽样收集

与 Logstash 整合

⭐与 ZipKin 整合
	HTTP 收集
	消息中间件收集
	收集原理
	数据存储
	API 接口