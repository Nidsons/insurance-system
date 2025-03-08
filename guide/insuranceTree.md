保险智能核心系统 3.0 架构设计

一、系统全景架构
graph TD
A[客户端/渠道] --> B(API Gateway)
B --> C[产品中心]
B --> D[投保引擎]
B --> E[核保中心]
B --> F[理赔服务]
C --> G[规则引擎]
D --> H[分布式事务]
E --> I[工作流引擎]
F --> J[文件服务]
G --> K[Redis集群]
H --> L[RocketMQ]
I --> M[OCR服务]
J --> N[对象存储]
style A fill:#f9f,stroke:#333
style B fill:#6f9,stroke:#333

二、分模块设计与技术栈
1. 产品中心（Product Center）
核心功能：产品配置管理、保费计算规则、灰度发布
技术栈：
- 框架：SpringBoot 3.1 + MyBatis Plus
- 缓存：Redis 7.0（哨兵模式）
- 搜索：Elasticsearch 8.0 + IK分词
- 规则引擎：Drools 8.0
- 部署：Docker + K8s Deployment
链路设计：
- 用户->>+产品服务: 创建产品
- 产品服务->>+MySQL: 写入主数据
- MySQL-->>-产品服务: 返回ID
- 产品服务->>+Drools: 解析规则
- Drools-->>-产品服务: 规则校验结果
- 产品服务->>+Redis: 缓存产品信息
- Redis-->>-产品服务: 确认缓存
- 产品服务-->>-用户: 返回创建结果

2. 投保引擎（Policy Engine）
核心功能：高并发投保、库存管理、分布式事务
技术栈：
- 框架：SpringCloud Alibaba 2022
- 消息队列：RocketMQ 5.0（事务消息）
- 分库分表：ShardingSphere 5.3
- 限流熔断：Sentinel 1.8
- 部署：K8s StatefulSet + HPA
链路设计：
- 用户->>+投保服务: 提交投保请求
- 投保服务->>+Redis: 原子扣减库存(Lua)
- Redis-->>-投保服务: 扣减成功
- 投保服务->>+RocketMQ: 发送预扣消息
- RocketMQ-->>-投保服务: 存储消息
- 投保服务->>+MySQL: 写入保单数据
- MySQL-->>-投保服务: 写入成功
- 投保服务->>+RocketMQ: 确认消息
- RocketMQ-->>用户: 投保完成通知

3. 核保中心（Underwriting Center）
   核心功能：智能核保流程、健康告知OCR、分布式锁
技术栈：
- 工作流引擎：Flowable 6.7
- OCR识别：Tesseract 5.0 + OpenCV
- 分布式锁：Redisson 3.20
- 事务管理：Seata 1.7
- 部署：K8s CronJob（定时任务）
链路设计：
- 投保服务->>+核保服务: 发起核保 
- 核保服务->>+Redis: 获取分布式锁 
- Redis-->>-核保服务: 获得锁 
- 核保服务->>+OCR服务: 解析健康告知 
- OCR服务-->>-核保服务: 返回文本 
- 核保服务->>+规则引擎: 执行核保规则 
- 规则引擎-->>-核保服务: 核保结果 
- 核保服务->>+MySQL: 更新核保状态 
- 核保服务->>+Redis: 释放锁 
- 核保服务-->>-投保服务: 返回结果

4. 理赔服务（Claim Service）
   核心功能：理赔申请、材料审核、反欺诈检测
技术栈：
- 文件存储：MinIO（兼容S3协议）
- 实时计算：Flink 1.16
- AI模型：TensorFlow Serving
- 区块链：Hyperledger Fabric（存证）
- 部署：K8s DaemonSet
- 用户->>+理赔服务: 上传材料 
- 理赔服务->>+MinIO: 存储文件 
- MinIO-->>-理赔服务: 返回URL 
- 理赔服务->>+AI模型: 反欺诈检测 
- AI模型-->>-理赔服务: 检测结果 
- 理赔服务->>+Flink: 实时统计 
- Flink-->>-监控大屏: 展示数据 
- 理赔服务-->>-用户: 理赔结果

5. 监控中心（Observability）
   核心功能：全链路监控、日志分析、智能告警
技术栈：
- 链路追踪：SkyWalking 9.4
- 指标采集：Prometheus 2.42
- 日志分析：ELK Stack（7.17）
- 可视化：Grafana 9.3
- 部署：K8s Operator  

架构设计：
graph LR
A[应用服务] --> B[Prometheus Exporter]
A --> C[Filebeat]
B --> D[Prometheus]
C --> E[Logstash]
D --> F[Grafana]
E --> G[Elasticsearch]
G --> H[Kibana]
A --> I[SkyWalking Agent]
I --> J[SkyWalking OAP]
J --> K[SkyWalking UI]


三、推荐学习与开发路径
1. 阶段一：基础能力构建（1-2个月）
   journey
   title 新手村任务
   section 产品中心
   创建产品API → 集成Redis缓存 → 实现规则引擎 → 完成灰度发布
   section 技术目标
   掌握SpringBoot开发 → 理解分布式缓存 → 熟悉规则引擎 → 接触云原生

具体任务：
使用SpringBoot实现产品CRUD接口
为产品详情添加Redis二级缓存
编写Drools保费计算规则文件
通过Nacos配置中心实现灰度开关

2. 阶段二：高并发进阶（3-4个月）
   markdown
   复制
1. 投保引擎开发：
   - RocketMQ事务消息处理投保
   - ShardingSphere分库分表
   - Sentinel热点参数限流

2. 核保中心搭建：
   - Flowable工作流设计
   - Tesseract+OpenCV图像处理
   - Redisson分布式锁实践

3. 阶段三：系统完善（5-6个月）
   markdown
   复制
- 集成ELK日志系统
- 搭建Flink实时计算平台
- 实现Prometheus+Grafana监控
- 完成K8s多集群部署


三、推荐学习与开发路径
1. 阶段一：基础能力构建（1-2个月）
   mermaid
   复制
   journey
   title 新手村任务
   section 产品中心
   创建产品API → 集成Redis缓存 → 实现规则引擎 → 完成灰度发布
   section 技术目标
   掌握SpringBoot开发 → 理解分布式缓存 → 熟悉规则引擎 → 接触云原生
   具体任务：

使用SpringBoot实现产品CRUD接口

为产品详情添加Redis二级缓存

编写Drools保费计算规则文件

通过Nacos配置中心实现灰度开关

2. 阶段二：高并发进阶（3-4个月）
   markdown
   复制
1. 投保引擎开发：
   - RocketMQ事务消息处理投保
   - ShardingSphere分库分表
   - Sentinel热点参数限流

2. 核保中心搭建：
   - Flowable工作流设计
   - Tesseract+OpenCV图像处理
   - Redisson分布式锁实践
3. 阶段三：系统完善（5-6个月）
   markdown
   复制
- 集成ELK日志系统
- 搭建Flink实时计算平台
- 实现Prometheus+Grafana监控
- 完成K8s多集群部署


四、高可用部署方案
1. 多集群架构
   mermaid
   复制
   graph TB
   A[华东1] -->|同步| B[华北2]
   A -->|异步| C[华南1]
   B --> D[MySQL集群]
   C --> D
   D --> E[Global Load Balancer]
2. 容灾策略
   markdown
   复制
- 数据层：MySQL半同步复制 + Redis Cluster
- 服务层：K8s Pod反亲和性部署
- 接入层：Nginx流量镜像 + 跨区容灾
3. 压测指标
   bash
   复制
# 投保引擎压测要求
wrk -t12 -c5000 -d60s --latency http://api/apply
# 预期指标：
# - QPS ≥ 10,000
# - P99延迟 ≤ 800ms
# - 错误率 < 0.01%