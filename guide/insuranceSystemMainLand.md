保险智能核心系统 3.0
技术愿景：基于云原生的新一代保险业务中台，支撑千万级保单处理能力
系统架构图：

graph TD
A[客户端] --> B[API Gateway]
B --> C[产品中心]
B --> D[投保引擎]
B --> E[核保中心]
B --> F[理赔服务]
C --> G[规则引擎]
D --> H[Redis集群]
E --> I[分布式事务]
F --> J[OCR识别]
G --> K[Elasticsearch]
H --> L[哨兵模式]
I --> M[Seata]
J --> N[AI模型]
style A fill:#f9f,stroke:#333
style K fill:#f96,stroke:#333


一、系统核心需求清单（按学习优先级排序）
模块1：保险产品管理中心
1. 需求：支持百万级保险产品配置
    - 技术点：
      √ SpringBoot + MyBatis Plus动态SQL
      √ Redis缓存产品基础信息
      √ Elasticsearch实现多维度产品搜索
      √ 规则引擎（Drools）实现保费计算逻辑
    - 交付物：
      [产品配置界面截图]
      [ES复杂查询DSL语句文档]
      [Drools规则文件样例]

2. 需求：产品版本灰度发布
    - 技术点：
      √ Nacos配置中心动态生效
      √ K8s金丝雀发布策略
      √ SkyWalking链路追踪
    - 交付物：
      [灰度发布流水线.yaml]
      [流量染色标记代码]

模块2：高并发投保引擎
3. 需求：支持10万QPS投保请求
    - 技术点：
      √ Redis+Lua实现库存原子扣减
      √ RocketMQ削峰填谷
      √ Sentinel热点参数限流
      √ ShardingSphere分库分表
    - 交付物：
      [库存扣减Lua脚本]
      [分片键选择方案文档]
      [压测报告（JMeter）]

4. 需求：投保数据实时分析
    - 技术点：
      √ Flink实时计算投保地域分布
      √ Kafka数据管道
      √ Grafana可视化大屏
    - 交付物：
      [Flink流处理Job代码]
      [投保热力图大屏截图]

模块3：智能核保中心
5. 需求：分布式核保流程
    - 技术点：
      √ Seata AT模式分布式事务
      √ 工作流引擎（Flowable）
      √ 健康告知OCR识别（Tesseract）
    - 交付物：
      [核保流程图.bpmn]
      [事务回滚测试用例]
      [OCR识别率测试数据]

6. 需求：核保规则引擎
    - 技术点：
      √ Groovy动态脚本加载
      √ 规则版本回滚
      √ 规则命中率监控
    - 交付物：
      [Groovy脚本管理器代码]
      [规则命中率PromQL表达式]


二、高可用架构设计
容灾方案
1. 多级缓存架构：
   - L1：Guava本地缓存（失效时间30秒）
   - L2：Redis集群（三主三从+哨兵）
   - L3：MySQL热点数据归档

2. 限流降级策略：
   - 网关层：Nginx漏桶算法限流
   - 服务层：Sentinel集群流控
   - 数据层：Hystrix线程池隔离

3. 弹性扩缩容：
   - K8s HPA（基于CPU/自定义指标）
   - 阿里云ESS自动伸缩组

监控体系
1. 指标采集：
   - JVM：Micrometer + Prometheus
   - 业务指标：埋点SDK

2. 日志体系：
   - Filebeat采集日志
   - Logstash管道处理
   - ES存储 + Kibana展示

3. 告警通道：
   - Prometheus AlertManager
   - 钉钉机器人通知


三、分阶段实施计划
阶段1：基础架构搭建（3月1日-3月15日）
Week 1:
- Day1: 用Spring Initializr创建项目模板
- Day3: 编写Dockerfile打包基础服务
- Day5: 在K8s部署Nacos集群

Week 2:
- Day1: 实现ShardingSphere分库分表
- Day3: 搭建Redis哨兵集群
- Day5: 配置SkyWalking探针

阶段2：核心模块开发（3月16日-4月30日）
Week 3:
- 产品中心：实现ES多条件搜索
- 投保引擎：完成库存扣减Lua脚本

Week 4:
- 核保中心：集成Flowable工作流
- 理赔服务：开发OCR识别接口

Week 5:
- 实时计算：Flink处理投保数据
- 监控告警：配置Prometheus规则

阶段3：高可用改造（5月1日-5月31日）
- 全链路压测（模拟双11流量）
- 混沌工程实验（随机下线节点）
- 容灾演练（主库宕机切换）

阶段4：部署运维（6月1日-6月15日）
- 编写Helm Chart
- 配置GitLab CI/CD流水线
- 制作系统部署手册

四、学习成果检验标准
代码质量要求
# 必须通过以下检查
mvn clean install
-DskipTests=false
-Dcheckstyle.skip=false
-Dpmd.skip=false
-Dsonar.login=your_token

架构设计评审标准
1. 扩展性：能否在不改代码的情况下新增保险产品类型
2. 可靠性：是否做到任意两个节点宕机不影响服务
3. 可观测性：能否在1分钟内定位接口超时原因


五、面试话术准备
当被问到项目难点时：
"在实现分布式核保时遇到的最大挑战是规则引擎的动态加载，我们创新性地采用Groovy脚本+版本快照的方式：
通过ClassLoader隔离不同版本规则
使用Redis发布订阅机制同步规则变更
设计规则回滚的版本控制算法
最终使规则生效延迟从分钟级降到秒级，同时保障了线程安全"

当被问到技术选型时：
"选择RocketMQ而非Kafka作为投保消峰组件，主要基于三点考虑：
事务消息机制保障了保费计算与保单生成的最终一致性
定时消息能力满足次日生效保单的延迟处理需求
与阿里云生态的天然集成简化了运维成本"

下一步行动建议：
立即在GitHub创建insurance-system私有仓库
根据阶段1计划开始环境搭建
每日晚10点前在对话中汇报当日代码提交记录
（需要我提供详细到函数级别的代码模板吗？例如保费计算器的并发控制实现）

