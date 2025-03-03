一、核心技术图谱
mindmap
root((保险系统技能树))
云原生
Docker容器化
Kubernetes编排
Helm包管理
Service Mesh
高并发
分布式锁
缓存击穿/穿透
异步化设计
分库分表
分布式
CAP理论
分布式事务
服务治理
一致性协议
中间件
Redis高级特性
RocketMQ事务消息
Elasticsearch聚合
Flink流处理
架构设计
微服务拆分
领域驱动设计
容灾方案
可观测体系

二、分模块技术收获
1. 基础架构层
   技术点	对应场景	掌握的深度要求
   SpringBoot	产品中心REST API开发	能自定义Starter和Actuator端点
   MyBatis Plus	动态数据源配置	理解动态SQL原理和分页拦截器
   Nacos	配置中心与服务发现	能实现配置热更新和集群部署
   Kubernetes	容器化部署	会编写Deployment/Service文件
2. 高并发层
   技术点	解决的问题	典型代码案例
   Redis+Lua	库存原子操作	EVAL "local stock = ..."
   Sentinel	热点参数限流	@SentinelResource注解配置
   ShardingSphere	分库分表	自定义分片算法实现类
   RocketMQ	异步消峰	事务消息+本地消息表方案
3. 分布式层
   技术点	实现原理	避坑要点
   Seata	AT模式两阶段提交	避免全局锁长时间持有
   SkyWalking	分布式追踪	TraceID穿透第三方调用
   Flowable	工作流引擎	补偿机制设计
   Hystrix	服务熔断	信号量隔离与线程池隔离选择
4. 智能应用层
   技术点	业务价值	实现路径
   Drools	保费计算规则化	.drl规则文件编写
   Tesseract	OCR识别健康告知	图像预处理参数调优
   Flink	实时投保分析	Window聚合+状态管理
   Grafana	可视化监控	编写PromQL自定义指标
