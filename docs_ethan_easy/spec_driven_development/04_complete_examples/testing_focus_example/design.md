# 金融风控系统 - 设计文档

## 1. 项目概述

### 项目名称
金融风控系统

### 设计目标
设计一个高可靠、高精度、高性能的金融风控系统架构，
支持实时风险评估（响应时间<100ms），日处理10万笔交易，
风险识别准确率>95%，系统可用性>99.9%。

### 关联需求
- 需求文档：[requirements.md](./requirements.md)
- 主要需求：REQ-001 至 REQ-007
- 设计范围：完整的金融风控系统架构

### 设计原则
- **测试驱动设计**：以测试为中心的设计理念，确保每个组件都可测试
- **高可靠性**：系统7×24小时稳定运行，故障自动恢复
- **高精度**：风险识别准确率>95%，误报率<5%
- **高性能**：实时风险评估，99%请求在100ms内完成
- **可扩展性**：支持算法扩展、业务扩展、性能扩展
- **可观测性**：全链路监控，问题快速定位和解决
- **安全合规**：满足金融行业安全和合规要求

## 2. 系统架构设计

### 总体架构
```
金融风控系统总体架构：
┌─────────────────────────────────────────────────────────────┐
│                        接入层                                │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              API Gateway + 负载均衡                     ││
│  │        (限流、鉴权、路由、监控、熔断)                     ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        应用层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │风险评估引擎  │  │规则引擎服务  │  │模型管理服务  │          │
│  │RiskEngine   │  │RuleEngine   │  │ModelManager │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │名单管理服务  │  │监控预警服务  │  │报告分析服务  │          │
│  │ListManager  │  │MonitorSystem│  │ReportSystem │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │案件管理服务  │  │配置管理服务  │  │用户管理服务  │          │
│  │CaseManager  │  │ConfigSystem │  │UserManager  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                        数据层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   MySQL     │  │   Redis     │  │  Kafka      │          │
│  │(业务数据)    │  │ (缓存+计算) │  │ (消息队列)   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │Elasticsearch│  │  ClickHouse │  │   HDFS      │          │
│  │ (日志搜索)   │  │ (实时分析)   │  │ (历史存储)   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
├─────────────────────────────────────────────────────────────┤
│                        基础设施层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │         Kubernetes + Docker + 服务网格                  ││
│  │    监控告警 + 链路追踪 + 日志收集 + 配置中心             ││
│  │   (Prometheus + Jaeger + ELK + Nacos)                  ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 技术选型
| 技术分类 | 选择的技术 | 版本 | 选择理由 |
|---------|-----------|------|---------|
| 开发语言 | Java | 11+ | 性能稳定，生态完善，团队熟悉 |
| 应用框架 | Spring Boot | 2.7.x | 快速开发，自动配置，监控完善 |
| 微服务框架 | Spring Cloud | 2021.0.x | 服务治理，熔断降级，配置管理 |
| 数据库 | MySQL | 8.0.x | 事务支持，数据一致性，性能优秀 |
| 缓存 | Redis | 6.2.x | 高性能，实时计算，分布式锁 |
| 消息队列 | Apache Kafka | 3.3.x | 高吞吐量，消息持久化，流处理 |
| 搜索引擎 | Elasticsearch | 8.6.x | 全文搜索，日志分析，实时聚合 |
| 时序数据库 | ClickHouse | 22.x | OLAP分析，实时计算，高性能 |
| 机器学习 | Python + Scikit-learn | 1.2.x | 算法丰富，生态完善，易于集成 |
| 规则引擎 | Drools | 8.x | 规则管理，动态配置，性能优秀 |
| 容器化 | Docker + Kubernetes | 1.26.x | 环境一致，自动扩缩，服务发现 |
| 监控 | Prometheus + Grafana | 2.42.x | 指标监控，可视化，告警管理 |
| 链路追踪 | Jaeger | 1.41.x | 分布式追踪，性能分析 |
| 日志 | ELK Stack | 8.6.x | 日志收集，搜索分析，可视化 |

### 部署架构
```
生产环境部署架构：
┌─────────────────────────────────────────────────────────────┐
│                        高可用部署                            │
│  ┌─────────────────┐         ┌─────────────────┐            │
│  │     机房A       │         │     机房B       │            │
│  │   (主机房)      │◄────────┤   (备机房)      │            │
│  │                 │         │                 │            │
│  └─────────────────┘         └─────────────────┘            │
├─────────────────────────────────────────────────────────────┤
│                        主机房详细架构                        │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    接入层                                ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     ││
│  │  │   SLB       │  │ API Gateway │  │  WAF + CDN  │     ││
│  │  │             │  │             │  │             │     ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘     ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    应用层                                ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     ││
│  │  │K8s集群-核心  │  │K8s集群-分析  │  │K8s集群-管理  │     ││
│  │  │(风险评估)    │  │(报告分析)    │  │(配置管理)    │     ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘     ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    数据层                                ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     ││
│  │  │MySQL集群    │  │Redis集群    │  │Kafka集群    │     ││
│  │  │(主从+分片)   │  │(哨兵+集群)   │  │(3副本)      │     ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘     ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## 3. 核心服务设计

### 服务1：风险评估引擎(RiskEngine)
**服务职责**：实时风险评估，集成规则引擎和机器学习模型

**核心功能**：
- 交易数据接收：接收并验证交易数据完整性
- 特征工程：提取和转换风险特征
- 规则评估：执行预定义的风险规则
- 模型预测：调用机器学习模型进行风险预测
- 风险评分：综合规则和模型结果计算最终评分
- 决策建议：根据评分给出处理建议

**技术架构**：
- 框架：Spring Boot + Spring Cloud
- 缓存：Redis（特征缓存、结果缓存）
- 消息队列：Kafka（异步处理、事件通知）
- 监控：Micrometer + Prometheus

**接口设计**：
```java
// 风险评估API
@PostMapping("/api/risk/evaluate")
public RiskResult evaluateRisk(@RequestBody TransactionData transaction) {
    // 1. 数据验证和预处理
    // 2. 特征提取和工程化
    // 3. 规则引擎评估
    // 4. 机器学习模型预测
    // 5. 综合评分计算
    // 6. 决策建议生成
    // 7. 结果缓存和日志记录
}

// 批量风险评估API
@PostMapping("/api/risk/batch-evaluate")
public List<RiskResult> batchEvaluateRisk(@RequestBody List<TransactionData> transactions);

// 风险评估状态查询API
@GetMapping("/api/risk/status/{transactionId}")
public RiskStatus getRiskStatus(@PathVariable String transactionId);
```

**数据模型**：
```sql
-- 风险评估记录表
CREATE TABLE risk_evaluations (
    id VARCHAR(64) PRIMARY KEY,
    transaction_id VARCHAR(64) NOT NULL,
    user_id VARCHAR(64) NOT NULL,
    transaction_amount DECIMAL(15,2) NOT NULL,
    transaction_type VARCHAR(32) NOT NULL,
    device_fingerprint VARCHAR(128),
    ip_address VARCHAR(45),
    risk_score DECIMAL(5,2) NOT NULL,
    risk_level ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL') NOT NULL,
    decision ENUM('PASS', 'REVIEW', 'REJECT') NOT NULL,
    rule_results JSON,
    model_results JSON,
    processing_time_ms INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_transaction_id (transaction_id),
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at),
    INDEX idx_risk_level (risk_level)
);

-- 特征存储表
CREATE TABLE risk_features (
    id VARCHAR(64) PRIMARY KEY,
    evaluation_id VARCHAR(64) NOT NULL,
    feature_name VARCHAR(128) NOT NULL,
    feature_value TEXT,
    feature_type VARCHAR(32) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (evaluation_id) REFERENCES risk_evaluations(id),
    INDEX idx_evaluation_id (evaluation_id),
    INDEX idx_feature_name (feature_name)
);
```

**性能设计**：
- 响应时间：99%请求在100ms内完成
- 并发处理：支持1000 TPS
- 缓存策略：特征缓存5分钟，结果缓存1小时
- 异步处理：非核心逻辑异步执行

**测试设计**：
```java
// 单元测试示例
@Test
public void testRiskEvaluationBasicScenario() {
    // Given: 正常交易数据
    TransactionData transaction = createNormalTransaction();
    
    // When: 执行风险评估
    RiskResult result = riskEngine.evaluate(transaction);
    
    // Then: 验证结果
    assertThat(result.getRiskScore()).isBetween(0.0, 1.0);
    assertThat(result.getRiskLevel()).isEqualTo(RiskLevel.LOW);
    assertThat(result.getDecision()).isEqualTo(Decision.PASS);
    assertThat(result.getProcessingTimeMs()).isLessThan(100);
}

@Test
public void testRiskEvaluationHighRiskScenario() {
    // Given: 高风险交易数据
    TransactionData transaction = createHighRiskTransaction();
    
    // When: 执行风险评估
    RiskResult result = riskEngine.evaluate(transaction);
    
    // Then: 验证结果
    assertThat(result.getRiskLevel()).isIn(RiskLevel.HIGH, RiskLevel.CRITICAL);
    assertThat(result.getDecision()).isIn(Decision.REVIEW, Decision.REJECT);
}

// 性能测试示例
@Test
public void testRiskEvaluationPerformance() {
    // Given: 1000个测试交易
    List<TransactionData> transactions = createTestTransactions(1000);
    
    // When: 并发执行风险评估
    long startTime = System.currentTimeMillis();
    List<RiskResult> results = riskEngine.batchEvaluate(transactions);
    long endTime = System.currentTimeMillis();
    
    // Then: 验证性能
    assertThat(results).hasSize(1000);
    assertThat(endTime - startTime).isLessThan(1000); // 1秒内完成
    
    // 验证每个结果的响应时间
    results.forEach(result -> {
        assertThat(result.getProcessingTimeMs()).isLessThan(100);
    });
}
```

### 服务2：规则引擎服务(RuleEngine)
**服务职责**：管理和执行风险规则，支持动态配置和实时生效

**核心功能**：
- 规则管理：规则的增删改查、版本管理
- 规则编译：规则语法验证、逻辑编译
- 规则执行：高性能规则执行引擎
- 规则测试：规则效果测试和验证
- 规则审计：规则变更记录和追踪

**技术架构**：
- 规则引擎：Drools + 自定义DSL
- 缓存：Redis（规则缓存、执行结果缓存）
- 数据库：MySQL（规则存储、版本管理）
- 监控：规则执行统计、性能监控

**规则类型设计**：
```java
// 规则类型枚举
public enum RuleType {
    THRESHOLD_RULE,    // 阈值规则：金额、频次、比例等
    BLACKLIST_RULE,    // 黑名单规则：用户、设备、IP等
    PATTERN_RULE,      // 模式规则：行为模式、时间模式等
    ASSOCIATION_RULE,  // 关联规则：关联分析、图计算等
    ML_RULE           // 机器学习规则：模型预测、特征工程等
}

// 规则执行结果
public class RuleResult {
    private String ruleId;
    private String ruleName;
    private RuleType ruleType;
    private boolean triggered;
    private double score;
    private String reason;
    private Map<String, Object> details;
    private long executionTimeMs;
}
```

**规则DSL设计**：
```
// 示例：大额交易规则
rule "大额交易风险检查"
when
    transaction: Transaction(amount > 50000)
    user: User(riskLevel == "HIGH")
then
    RiskScore.addScore(0.8);
    RiskReason.add("大额交易且用户风险等级高");
end

// 示例：异常时间交易规则
rule "异常时间交易检查"
when
    transaction: Transaction(hour < 6 || hour > 23)
    frequency: TransactionFrequency(count > 5, timeWindow == "1H")
then
    RiskScore.addScore(0.6);
    RiskReason.add("异常时间高频交易");
end
```

**测试设计**：
```java
// 规则引擎测试
@Test
public void testThresholdRule() {
    // Given: 大额交易规则
    Rule rule = createThresholdRule("amount > 50000", 0.8);
    Transaction transaction = new Transaction(60000.0);
    
    // When: 执行规则
    RuleResult result = ruleEngine.execute(rule, transaction);
    
    // Then: 验证结果
    assertThat(result.isTriggered()).isTrue();
    assertThat(result.getScore()).isEqualTo(0.8);
    assertThat(result.getExecutionTimeMs()).isLessThan(10);
}

@Test
public void testRuleEnginePerformance() {
    // Given: 100条规则，1000笔交易
    List<Rule> rules = createTestRules(100);
    List<Transaction> transactions = createTestTransactions(1000);
    
    // When: 批量执行规则
    long startTime = System.currentTimeMillis();
    Map<String, List<RuleResult>> results = ruleEngine.batchExecute(rules, transactions);
    long endTime = System.currentTimeMillis();
    
    // Then: 验证性能
    assertThat(results).hasSize(1000);
    assertThat(endTime - startTime).isLessThan(5000); // 5秒内完成
}
```

### 服务3：模型管理服务(ModelManager)
**服务职责**：机器学习模型的训练、部署、监控和管理

**核心功能**：
- 模型训练：自动化模型训练流水线
- 模型评估：模型效果评估和对比
- 模型部署：灰度发布、A/B测试
- 模型监控：性能监控、漂移检测
- 模型版本：版本管理、回滚机制

**技术架构**：
- 机器学习：Python + Scikit-learn + XGBoost
- 模型服务：Flask + Docker + Kubernetes
- 特征存储：Redis + ClickHouse
- 模型存储：MinIO + 模型注册中心

**模型流水线设计**：
```python
# 模型训练流水线
class ModelTrainingPipeline:
    def __init__(self, config):
        self.config = config
        self.feature_extractor = FeatureExtractor()
        self.model_trainer = ModelTrainer()
        self.model_evaluator = ModelEvaluator()
    
    def run(self, start_date, end_date):
        # 1. 数据准备
        raw_data = self.load_training_data(start_date, end_date)
        
        # 2. 特征工程
        features = self.feature_extractor.extract(raw_data)
        
        # 3. 模型训练
        model = self.model_trainer.train(features)
        
        # 4. 模型评估
        metrics = self.model_evaluator.evaluate(model, features)
        
        # 5. 模型注册
        if self.is_model_better(metrics):
            self.register_model(model, metrics)
        
        return model, metrics
```

**模型评估指标**：
```java
// 模型评估指标
public class ModelMetrics {
    private double accuracy;        // 准确率
    private double precision;       // 精确率
    private double recall;          // 召回率
    private double f1Score;         // F1分数
    private double auc;             // AUC值
    private double ks;              // KS值
    private double psi;             // PSI值（群体稳定性指标）
    private Map<String, Double> confusionMatrix;
    private List<FeatureImportance> featureImportances;
}
```

**测试设计**：
```java
@Test
public void testModelPrediction() {
    // Given: 训练好的模型和测试数据
    Model model = loadTrainedModel("fraud_detection_v1.0");
    TransactionFeatures features = createTestFeatures();
    
    // When: 执行预测
    PredictionResult result = modelManager.predict(model, features);
    
    // Then: 验证结果
    assertThat(result.getScore()).isBetween(0.0, 1.0);
    assertThat(result.getPredictionTimeMs()).isLessThan(50);
    assertThat(result.getModelVersion()).isEqualTo("v1.0");
}

@Test
public void testModelPerformance() {
    // Given: 模型和大量测试数据
    Model model = loadTrainedModel("fraud_detection_v1.0");
    List<TransactionFeatures> testData = loadTestData(10000);
    
    // When: 批量预测
    long startTime = System.currentTimeMillis();
    List<PredictionResult> results = modelManager.batchPredict(model, testData);
    long endTime = System.currentTimeMillis();
    
    // Then: 验证性能
    assertThat(results).hasSize(10000);
    assertThat(endTime - startTime).isLessThan(5000); // 5秒内完成
    
    // 验证模型指标
    ModelMetrics metrics = calculateMetrics(results, testData);
    assertThat(metrics.getAccuracy()).isGreaterThan(0.95);
    assertThat(metrics.getPrecision()).isGreaterThan(0.90);
    assertThat(metrics.getRecall()).isGreaterThan(0.85);
}
```

### 服务4：监控预警服务(MonitorSystem)
**服务职责**：实时监控交易行为，识别异常模式，发送预警通知

**核心功能**：
- 实时监控：流式数据处理，实时指标计算
- 异常检测：统计异常、模式异常、时序异常
- 预警规则：可配置的预警规则和阈值
- 通知管理：多渠道通知，通知去重和聚合
- 监控大盘：实时数据展示，历史趋势分析

**技术架构**：
- 流处理：Apache Kafka + Kafka Streams
- 时序数据：ClickHouse + TimescaleDB
- 缓存：Redis（实时指标、预警状态）
- 通知：短信、邮件、钉钉、企业微信

**监控指标设计**：
```java
// 监控指标定义
public class MonitorMetrics {
    // 业务指标
    private long transactionCount;          // 交易笔数
    private double transactionAmount;       // 交易金额
    private double avgTransactionAmount;    // 平均交易金额
    private long uniqueUsers;               // 独立用户数
    private double riskTransactionRatio;    // 风险交易比例
    
    // 系统指标
    private double avgResponseTime;         // 平均响应时间
    private double errorRate;               // 错误率
    private long qps;                       // 每秒请求数
    private double systemLoad;              // 系统负载
    
    // 风控指标
    private long highRiskTransactions;      // 高风险交易数
    private long blockedTransactions;       // 拦截交易数
    private double falsePositiveRate;       // 误报率
    private double truePositiveRate;        // 真阳性率
}
```

**预警规则配置**：
```yaml
# 预警规则配置示例
alert_rules:
  - name: "交易量异常"
    metric: "transaction_count"
    condition: "rate_1h > baseline * 2"
    severity: "WARNING"
    channels: ["email", "sms"]
    
  - name: "风险交易激增"
    metric: "risk_transaction_ratio"
    condition: "current > 0.1"
    severity: "CRITICAL"
    channels: ["email", "sms", "dingtalk"]
    
  - name: "系统响应时间异常"
    metric: "avg_response_time"
    condition: "current > 200"
    severity: "WARNING"
    channels: ["email"]
```

**测试设计**：
```java
@Test
public void testAnomalyDetection() {
    // Given: 正常基线数据和异常数据
    List<MetricData> baselineData = createBaselineData(24 * 60); // 24小时基线
    MetricData anomalyData = createAnomalyData(5.0); // 5倍异常
    
    // When: 执行异常检测
    AnomalyResult result = monitorSystem.detectAnomaly(baselineData, anomalyData);
    
    // Then: 验证结果
    assertThat(result.isAnomaly()).isTrue();
    assertThat(result.getAnomalyScore()).isGreaterThan(0.8);
    assertThat(result.getDetectionTimeMs()).isLessThan(100);
}

@Test
public void testAlertNotification() {
    // Given: 预警规则和触发数据
    AlertRule rule = createAlertRule("transaction_count", "rate_1h > 1000");
    MetricData triggerData = createTriggerData(1500);
    
    // When: 触发预警
    AlertEvent event = monitorSystem.processAlert(rule, triggerData);
    
    // Then: 验证预警
    assertThat(event.isTriggered()).isTrue();
    assertThat(event.getSeverity()).isEqualTo(Severity.WARNING);
    assertThat(event.getNotificationChannels()).contains("email", "sms");
    
    // 验证通知发送
    verify(notificationService).sendAlert(event);
}
```

## 4. 数据架构设计

### 数据分层架构
```
数据架构分层：
┌─────────────────────────────────────────────────────────────┐
│                        应用数据层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  业务数据库(MySQL) + 缓存(Redis) + 消息队列(Kafka)       ││
│  │        实时交易数据 + 用户数据 + 配置数据               ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        分析数据层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  时序数据库(ClickHouse) + 搜索引擎(Elasticsearch)        ││
│  │        实时指标计算 + 日志分析 + 全文搜索               ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        存储数据层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │        大数据存储(HDFS) + 对象存储(MinIO)                ││
│  │        历史数据归档 + 模型文件存储 + 备份数据           ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 数据库设计

#### 核心业务表设计
```sql
-- 用户信息表
CREATE TABLE users (
    id VARCHAR(64) PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    name VARCHAR(100) NOT NULL,
    risk_level ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL') DEFAULT 'LOW',
    status ENUM('ACTIVE', 'INACTIVE', 'SUSPENDED') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_phone (phone),
    INDEX idx_risk_level (risk_level),
    INDEX idx_status (status)
);

-- 交易表
CREATE TABLE transactions (
    id VARCHAR(64) PRIMARY KEY,
    user_id VARCHAR(64) NOT NULL,
    amount DECIMAL(15,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'CNY',
    transaction_type VARCHAR(32) NOT NULL,
    merchant_id VARCHAR(64),
    device_fingerprint VARCHAR(128),
    ip_address VARCHAR(45),
    location VARCHAR(200),
    status ENUM('PENDING', 'SUCCESS', 'FAILED', 'CANCELLED') DEFAULT 'PENDING',
    risk_score DECIMAL(5,2),
    risk_level ENUM('LOW', 'MEDIUM', 'HIGH', 'CRITICAL'),
    decision ENUM('PASS', 'REVIEW', 'REJECT'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_id (user_id),
    INDEX idx_amount (amount),
    INDEX idx_type (transaction_type),
    INDEX idx_status (status),
    INDEX idx_risk_level (risk_level),
    INDEX idx_created_at (created_at),
    INDEX idx_ip_address (ip_address)
);

-- 规则表
CREATE TABLE risk_rules (
    id VARCHAR(64) PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    rule_type ENUM('THRESHOLD', 'BLACKLIST', 'PATTERN', 'ASSOCIATION', 'ML') NOT NULL,
    rule_content TEXT NOT NULL,
    priority INT DEFAULT 0,
    weight DECIMAL(3,2) DEFAULT 1.0,
    status ENUM('ACTIVE', 'INACTIVE', 'TESTING') DEFAULT 'INACTIVE',
    version VARCHAR(32) NOT NULL,
    created_by VARCHAR(64) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_type (rule_type),
    INDEX idx_status (status),
    INDEX idx_priority (priority),
    INDEX idx_created_at (created_at)
);

-- 黑白名单表
CREATE TABLE risk_lists (
    id VARCHAR(64) PRIMARY KEY,
    list_type ENUM('BLACKLIST', 'WHITELIST') NOT NULL,
    item_type ENUM('USER', 'DEVICE', 'IP', 'EMAIL', 'PHONE') NOT NULL,
    item_value VARCHAR(500) NOT NULL,
    reason VARCHAR(500),
    source VARCHAR(100),
    expire_at TIMESTAMP NULL,
    status ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_by VARCHAR(64) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_list_item (list_type, item_type, item_value),
    INDEX idx_type (list_type, item_type),
    INDEX idx_value (item_value),
    INDEX idx_expire_at (expire_at),
    INDEX idx_status (status)
);

-- 案件表
CREATE TABLE risk_cases (
    id VARCHAR(64) PRIMARY KEY,
    case_number VARCHAR(100) UNIQUE NOT NULL,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    case_type ENUM('FRAUD', 'SUSPICIOUS', 'VIOLATION', 'OTHER') NOT NULL,
    priority ENUM('LOW', 'MEDIUM', 'HIGH', 'URGENT') DEFAULT 'MEDIUM',
    status ENUM('OPEN', 'INVESTIGATING', 'RESOLVED', 'CLOSED') DEFAULT 'OPEN',
    assigned_to VARCHAR(64),
    created_by VARCHAR(64) NOT NULL,
    resolved_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_case_number (case_number),
    INDEX idx_type (case_type),
    INDEX idx_priority (priority),
    INDEX idx_status (status),
    INDEX idx_assigned_to (assigned_to),
    INDEX idx_created_at (created_at)
);
```

### 数据分片策略
```sql
-- 交易表按时间分片（月分片）
CREATE TABLE transactions_202404 LIKE transactions;
CREATE TABLE transactions_202405 LIKE transactions;
-- ... 按月创建分片表

-- 用户表按用户ID哈希分片
CREATE TABLE users_shard_0 LIKE users;
CREATE TABLE users_shard_1 LIKE users;
CREATE TABLE users_shard_2 LIKE users;
CREATE TABLE users_shard_3 LIKE users;
```

### 缓存架构设计
```java
// 缓存层次设计
public class CacheManager {
    // L1缓存：本地缓存（Caffeine）
    private Cache<String, Object> localCache;
    
    // L2缓存：分布式缓存（Redis）
    private RedisTemplate<String, Object> redisTemplate;
    
    // L3缓存：数据库缓存（MySQL Query Cache）
    
    public <T> T get(String key, Class<T> clazz) {
        // 1. 尝试从本地缓存获取
        T value = localCache.getIfPresent(key);
        if (value != null) return value;
        
        // 2. 尝试从Redis获取
        value = (T) redisTemplate.opsForValue().get(key);
        if (value != null) {
            localCache.put(key, value);
            return value;
        }
        
        // 3. 从数据库加载
        value = loadFromDatabase(key, clazz);
        if (value != null) {
            redisTemplate.opsForValue().set(key, value, Duration.ofMinutes(30));
            localCache.put(key, value);
        }
        
        return value;
    }
}
```

## 5. 安全设计

### 安全架构
```
多层安全防护架构：
┌─────────────────────────────────────────────────────────────┐
│                        网络安全层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │     防火墙 + WAF + DDoS防护 + VPN + SSL证书             ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        应用安全层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │     认证授权 + API鉴权 + 输入验证 + 输出编码             ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        数据安全层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │     数据加密 + 脱敏处理 + 访问控制 + 审计日志           ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        基础设施安全层                        │
│  ┌─────────────────────────────────────────────────────────┐│
│  │     主机安全 + 容器安全 + 网络隔离 + 密钥管理           ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 认证授权设计
```java
// JWT Token设计
public class JwtTokenManager {
    private String secretKey;
    private long accessTokenExpiry = 2 * 60 * 60; // 2小时
    private long refreshTokenExpiry = 7 * 24 * 60 * 60; // 7天
    
    public TokenPair generateTokens(User user) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", user.getId());
        claims.put("roles", user.getRoles());
        claims.put("permissions", user.getPermissions());
        
        String accessToken = Jwts.builder()
            .setClaims(claims)
            .setSubject(user.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + accessTokenExpiry * 1000))
            .signWith(SignatureAlgorithm.HS512, secretKey)
            .compact();
            
        String refreshToken = generateRefreshToken(user);
        
        return new TokenPair(accessToken, refreshToken);
    }
}

// RBAC权限模型
@Entity
public class Permission {
    private String id;
    private String resource;    // 资源：user, transaction, rule, model
    private String action;      // 操作：read, write, delete, execute
    private String condition;   // 条件：自己的数据、部门数据、全部数据
}

@Entity 
public class Role {
    private String id;
    private String name;
    @ManyToMany
    private Set<Permission> permissions;
}

@Entity
public class User {
    private String id;
    private String username;
    @ManyToMany
    private Set<Role> roles;
}
```

### 数据加密设计
```java
// 敏感数据加密
@Component
public class DataEncryption {
    private final AESUtil aesUtil;
    
    // 加密存储
    public String encryptSensitiveData(String plaintext) {
        return aesUtil.encrypt(plaintext);
    }
    
    // 解密读取
    public String decryptSensitiveData(String ciphertext) {
        return aesUtil.decrypt(ciphertext);
    }
    
    // 数据脱敏
    public String maskSensitiveData(String data, SensitiveType type) {
        switch (type) {
            case PHONE:
                return data.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
            case EMAIL:
                return data.replaceAll("(\\w+)@(\\w+)", "$1***@$2");
            case ID_CARD:
                return data.replaceAll("(\\d{6})\\d{8}(\\d{4})", "$1********$2");
            default:
                return "***";
        }
    }
}
```

### 审计日志设计
```java
// 审计日志记录
@Component
public class AuditLogger {
    public void logOperation(String userId, String operation, String resource, 
                           String resourceId, Object oldValue, Object newValue) {
        AuditLog log = AuditLog.builder()
            .userId(userId)
            .operation(operation)
            .resource(resource)
            .resourceId(resourceId)
            .oldValue(JsonUtils.toJson(oldValue))
            .newValue(JsonUtils.toJson(newValue))
            .ipAddress(getClientIpAddress())
            .userAgent(getUserAgent())
            .timestamp(LocalDateTime.now())
            .build();
            
        auditLogRepository.save(log);
        
        // 异步发送到日志中心
        eventPublisher.publishEvent(new AuditLogEvent(log));
    }
}

// 审计日志表
CREATE TABLE audit_logs (
    id VARCHAR(64) PRIMARY KEY,
    user_id VARCHAR(64) NOT NULL,
    operation VARCHAR(100) NOT NULL,
    resource VARCHAR(100) NOT NULL,
    resource_id VARCHAR(64),
    old_value TEXT,
    new_value TEXT,
    ip_address VARCHAR(45),
    user_agent VARCHAR(500),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_operation (operation),
    INDEX idx_resource (resource),
    INDEX idx_timestamp (timestamp)
);
```

## 6. 测试策略设计

### 测试金字塔
```
测试金字塔架构：
        ┌─────────────────┐
        │   E2E测试       │  完整业务流程测试
        │   (5%)          │  用户场景验证
        ├─────────────────┤
        │   集成测试       │  服务间集成测试
        │   (20%)         │  API接口测试
        ├─────────────────┤
        │   单元测试       │  业务逻辑测试
        │   (75%)         │  算法验证测试
        └─────────────────┘
```

### 测试分层策略
| 测试层次 | 测试目标 | 测试工具 | 覆盖率要求 | 执行频率 |
|---------|---------|---------|-----------|----------|
| 单元测试 | 业务逻辑、算法正确性 | JUnit 5 + Mockito | >90% | 每次提交 |
| 集成测试 | 服务集成、数据一致性 | Spring Boot Test | >80% | 每天 |
| 接口测试 | API功能、性能 | REST Assured | 100% | 每天 |
| 性能测试 | 响应时间、并发能力 | JMeter + K6 | 关键路径 | 每周 |
| 安全测试 | 安全漏洞、权限控制 | OWASP ZAP | 100% | 每月 |
| E2E测试 | 用户场景、业务流程 | Selenium + Cucumber | 核心场景 | 发布前 |

### 单元测试设计
```java
// 风险评估单元测试
@ExtendWith(MockitoExtension.class)
class RiskEngineTest {
    
    @Mock
    private RuleEngine ruleEngine;
    
    @Mock
    private ModelManager modelManager;
    
    @InjectMocks
    private RiskEngine riskEngine;
    
    @Test
    @DisplayName("正常交易应返回低风险评分")
    void shouldReturnLowRiskForNormalTransaction() {
        // Given
        TransactionData transaction = TransactionData.builder()
            .userId("user123")
            .amount(new BigDecimal("1000.00"))
            .transactionType("TRANSFER")
            .build();
            
        RuleResult ruleResult = new RuleResult(0.2, "正常交易");
        PredictionResult modelResult = new PredictionResult(0.1, "低风险");
        
        when(ruleEngine.evaluate(transaction)).thenReturn(ruleResult);
        when(modelManager.predict(any())).thenReturn(modelResult);
        
        // When
        RiskResult result = riskEngine.evaluate(transaction);
        
        // Then
        assertThat(result.getRiskScore()).isLessThan(0.5);
        assertThat(result.getRiskLevel()).isEqualTo(RiskLevel.LOW);
        assertThat(result.getDecision()).isEqualTo(Decision.PASS);
        assertThat(result.getProcessingTimeMs()).isLessThan(100);
        
        verify(ruleEngine).evaluate(transaction);
        verify(modelManager).predict(any());
    }
    
    @Test
    @DisplayName("高风险交易应返回高风险评分")
    void shouldReturnHighRiskForSuspiciousTransaction() {
        // Given
        TransactionData transaction = TransactionData.builder()
            .userId("user456")
            .amount(new BigDecimal("100000.00"))
            .transactionType("TRANSFER")
            .deviceFingerprint("suspicious_device")
            .build();
            
        RuleResult ruleResult = new RuleResult(0.9, "大额异常交易");
        PredictionResult modelResult = new PredictionResult(0.85, "高风险");
        
        when(ruleEngine.evaluate(transaction)).thenReturn(ruleResult);
        when(modelManager.predict(any())).thenReturn(modelResult);
        
        // When
        RiskResult result = riskEngine.evaluate(transaction);
        
        // Then
        assertThat(result.getRiskScore()).isGreaterThan(0.8);
        assertThat(result.getRiskLevel()).isIn(RiskLevel.HIGH, RiskLevel.CRITICAL);
        assertThat(result.getDecision()).isIn(Decision.REVIEW, Decision.REJECT);
    }
    
    @ParameterizedTest
    @CsvSource({
        "1000.00, TRANSFER, 0.1",
        "50000.00, TRANSFER, 0.6", 
        "100000.00, WITHDRAWAL, 0.8"
    })
    @DisplayName("不同金额和类型的交易应返回合理的风险评分")
    void shouldReturnReasonableRiskScore(BigDecimal amount, String type, double expectedMinScore) {
        // Given
        TransactionData transaction = TransactionData.builder()
            .userId("user123")
            .amount(amount)
            .transactionType(type)
            .build();
            
        when(ruleEngine.evaluate(any())).thenReturn(new RuleResult(expectedMinScore, "规则评估"));
        when(modelManager.predict(any())).thenReturn(new PredictionResult(expectedMinScore, "模型预测"));
        
        // When
        RiskResult result = riskEngine.evaluate(transaction);
        
        // Then
        assertThat(result.getRiskScore()).isGreaterThanOrEqualTo(expectedMinScore);
    }
}
```

### 集成测试设计
```java
// 风险评估集成测试
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@Testcontainers
class RiskEvaluationIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("risk_test")
            .withUsername("test")
            .withPassword("test");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:6.2-alpine")
            .withExposedPorts(6379);
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Autowired
    private RiskEvaluationRepository repository;
    
    @Test
    @DisplayName("完整的风险评估流程集成测试")
    void shouldCompleteFullRiskEvaluationFlow() {
        // Given
        TransactionRequest request = TransactionRequest.builder()
            .userId("integration_user_001")
            .amount(new BigDecimal("5000.00"))
            .transactionType("TRANSFER")
            .build();
        
        // When
        ResponseEntity<RiskResponse> response = restTemplate.postForEntity(
            "/api/risk/evaluate", 
            request, 
            RiskResponse.class
        );
        
        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        RiskResponse riskResponse = response.getBody();
        assertThat(riskResponse).isNotNull();
        assertThat(riskResponse.getRiskScore()).isBetween(0.0, 1.0);
        assertThat(riskResponse.getDecision()).isNotNull();
        assertThat(riskResponse.getProcessingTimeMs()).isLessThan(100);
        
        // 验证数据持久化
        Optional<RiskEvaluation> savedEvaluation = repository.findByTransactionId(
            riskResponse.getTransactionId()
        );
        assertThat(savedEvaluation).isPresent();
        assertThat(savedEvaluation.get().getRiskScore()).isEqualTo(riskResponse.getRiskScore());
    }
    
    @Test
    @DisplayName("并发风险评估性能测试")
    void shouldHandleConcurrentRiskEvaluations() throws InterruptedException {
        // Given
        int threadCount = 10;
        int requestsPerThread = 100;
        ExecutorService executor = Executors.newFixedThreadPool(threadCount);
        CountDownLatch latch = new CountDownLatch(threadCount * requestsPerThread);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicLong totalResponseTime = new AtomicLong(0);
        
        // When
        for (int i = 0; i < threadCount; i++) {
            executor.submit(() -> {
                for (int j = 0; j < requestsPerThread; j++) {
                    try {
                        long startTime = System.currentTimeMillis();
                        
                        TransactionRequest request = TransactionRequest.builder()
                            .userId("concurrent_user_" + Thread.currentThread().getId())
                            .amount(new BigDecimal("1000.00"))
                            .transactionType("TRANSFER")
                            .build();
                        
                        ResponseEntity<RiskResponse> response = restTemplate.postForEntity(
                            "/api/risk/evaluate", 
                            request, 
                            RiskResponse.class
                        );
                        
                        if (response.getStatusCode() == HttpStatus.OK) {
                            successCount.incrementAndGet();
                            totalResponseTime.addAndGet(System.currentTimeMillis() - startTime);
                        }
                    } finally {
                        latch.countDown();
                    }
                }
            });
        }
        
        // Then
        boolean completed = latch.await(30, TimeUnit.SECONDS);
        assertThat(completed).isTrue();
        
        int totalRequests = threadCount * requestsPerThread;
        double successRate = (double) successCount.get() / totalRequests;
        double avgResponseTime = (double) totalResponseTime.get() / successCount.get();
        
        assertThat(successRate).isGreaterThan(0.99); // 99%成功率
        assertThat(avgResponseTime).isLessThan(100); // 平均响应时间<100ms
        
        executor.shutdown();
    }
}
```

### 性能测试设计
```java
// JMeter性能测试配置
public class PerformanceTestPlan {
    
    public static TestPlan createRiskEvaluationPerformanceTest() {
        TestPlan testPlan = new TestPlan("风险评估性能测试");
        
        // 线程组配置
        ThreadGroup threadGroup = new ThreadGroup();
        threadGroup.setNumThreads(100);        // 100并发用户
        threadGroup.setRampTime(60);           // 60秒启动时间
        threadGroup.setDuration(300);          // 运行5分钟
        
        // HTTP请求配置
        HTTPSampler httpSampler = new HTTPSampler();
        httpSampler.setDomain("localhost");
        httpSampler.setPort(8080);
        httpSampler.setPath("/api/risk/evaluate");
        httpSampler.setMethod("POST");
        httpSampler.setPostBodyRaw(true);
        httpSampler.addNonEncodedArgument("", generateTestData(), "");
        httpSampler.getHeaderManager().add(new Header("Content-Type", "application/json"));
        
        // 断言配置
        ResponseAssertion assertion = new ResponseAssertion();
        assertion.setTestFieldResponseCode();
        assertion.setAssumeSuccess(false);
        assertion.addTestString("200");
        
        // 响应时间断言
        DurationAssertion durationAssertion = new DurationAssertion();
        durationAssertion.setAllowedDuration(100); // 100ms内完成
        
        // 监听器配置
        SummaryReport summaryReport = new SummaryReport();
        ViewResultsFullVisualizer resultsVisualizer = new ViewResultsFullVisualizer();
        
        // 组装测试计划
        testPlan.addThreadGroup(threadGroup);
        threadGroup.addSampler(httpSampler);
        httpSampler.addAssertion(assertion);
        httpSampler.addAssertion(durationAssertion);
        testPlan.addListener(summaryReport);
        testPlan.addListener(resultsVisualizer);
        
        return testPlan;
    }
    
    private static String generateTestData() {
        return "{"
            + "\"userId\":\"perf_test_user_" + System.currentTimeMillis() + "\","
            + "\"amount\":1000.00,"
            + "\"transactionType\":\"TRANSFER\","
            + "\"deviceFingerprint\":\"test_device\""
            + "}";
    }
}
```

### E2E测试设计
```java
// Cucumber E2E测试
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class RiskEvaluationE2ESteps {
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    private TransactionRequest transactionRequest;
    private ResponseEntity<RiskResponse> response;
    
    @Given("用户发起一笔金额为 {double} 元的 {string} 交易")
    public void 用户发起交易(double amount, String transactionType) {
        transactionRequest = TransactionRequest.builder()
            .userId("e2e_test_user")
            .amount(new BigDecimal(amount))
            .transactionType(transactionType)
            .build();
    }
    
    @When("系统进行风险评估")
    public void 系统进行风险评估() {
        response = restTemplate.postForEntity(
            "/api/risk/evaluate", 
            transactionRequest, 
            RiskResponse.class
        );
    }
    
    @Then("系统应返回风险等级为 {string}")
    public void 系统应返回风险等级(String expectedRiskLevel) {
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getRiskLevel()).isEqualTo(expectedRiskLevel);
    }
    
    @Then("响应时间应小于 {int} 毫秒")
    public void 响应时间应小于毫秒(int maxResponseTime) {
        assertThat(response.getBody().getProcessingTimeMs()).isLessThan(maxResponseTime);
    }
}
```

### 测试数据管理
```java
// 测试数据工厂
@Component
public class TestDataFactory {
    
    public TransactionData createNormalTransaction() {
        return TransactionData.builder()
            .userId("normal_user_001")
            .amount(new BigDecimal("1000.00"))
            .transactionType("TRANSFER")
            .deviceFingerprint("normal_device")
            .ipAddress("192.168.1.100")
            .build();
    }
    
    public TransactionData createHighRiskTransaction() {
        return TransactionData.builder()
            .userId("high_risk_user_001")
            .amount(new BigDecimal("100000.00"))
            .transactionType("WITHDRAWAL")
            .deviceFingerprint("suspicious_device")
            .ipAddress("10.0.0.1")
            .location("异常地点")
            .build();
    }
    
    public List<TransactionData> createTestTransactions(int count) {
        List<TransactionData> transactions = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            transactions.add(TransactionData.builder()
                .userId("test_user_" + i)
                .amount(new BigDecimal(1000 + i * 100))
                .transactionType(i % 2 == 0 ? "TRANSFER" : "WITHDRAWAL")
                .build());
        }
        return transactions;
    }
}
```

## 7. 监控和运维设计

### 监控架构
```
全方位监控体系：
┌─────────────────────────────────────────────────────────────┐
│                        业务监控层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  风控指标监控 + 业务异常监控 + 用户行为监控             ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        应用监控层                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  应用性能监控 + 错误监控 + 链路追踪 + 日志监控         ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                        基础设施监控层                        │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  系统监控 + 网络监控 + 存储监控 + 容器监控             ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 关键监控指标
```yaml
# 监控指标配置
monitoring:
  business_metrics:
    - name: "transaction_count"
      description: "交易笔数"
      alert_threshold: "rate_1h > 10000"
      
    - name: "risk_transaction_ratio" 
      description: "风险交易比例"
      alert_threshold: "current > 0.1"
      
    - name: "false_positive_rate"
      description: "误报率"
      alert_threshold: "current > 0.05"
      
  application_metrics:
    - name: "response_time_p99"
      description: "99%响应时间"
      alert_threshold: "current > 100"
      
    - name: "error_rate"
      description: "错误率"
      alert_threshold: "current > 0.01"
      
    - name: "qps"
      description: "每秒请求数"
      alert_threshold: "current > 1000"
      
  infrastructure_metrics:
    - name: "cpu_usage"
      description: "CPU使用率"
      alert_threshold: "current > 0.8"
      
    - name: "memory_usage"
      description: "内存使用率"
      alert_threshold: "current > 0.8"
      
    - name: "disk_usage"
      description: "磁盘使用率"
      alert_threshold: "current > 0.9"
```

### 告警管理
```java
// 告警管理器
@Component
public class AlertManager {
    
    private final NotificationService notificationService;
    private final AlertRuleRepository alertRuleRepository;
    
    @EventListener
    public void handleMetricEvent(MetricEvent event) {
        List<AlertRule> rules = alertRuleRepository.findByMetricName(event.getMetricName());
        
        for (AlertRule rule : rules) {
            if (shouldTriggerAlert(event, rule)) {
                AlertEvent alertEvent = createAlertEvent(event, rule);
                processAlert(alertEvent);
            }
        }
    }
    
    private void processAlert(AlertEvent alertEvent) {
        // 告警去重
        if (isDuplicateAlert(alertEvent)) {
            return;
        }
        
        // 告警升级
        AlertLevel level = calculateAlertLevel(alertEvent);
        alertEvent.setLevel(level);
        
        // 发送通知
        List<String> channels = getNotificationChannels(level);
        notificationService.sendAlert(alertEvent, channels);
        
        // 记录告警
        alertEventRepository.save(alertEvent);
    }
    
    private boolean isDuplicateAlert(AlertEvent alertEvent) {
        return alertEventRepository.existsByRuleIdAndStatusAndCreatedAtAfter(
            alertEvent.getRuleId(),
            AlertStatus.ACTIVE,
            LocalDateTime.now().minusMinutes(5)
        );
    }
}
```

## 8. 开发计划

### 开发阶段
| 阶段 | 主要工作 | 预计时间 | 交付成果 |
|------|---------|---------|---------|
| 第一期 | 核心风控引擎 | 1.5个月 | 风险评估、规则引擎、基础框架 |
| 第二期 | 完整业务功能 | 1.5个月 | 模型管理、监控预警、名单管理 |
| 第三期 | 分析和管理 | 1个月 | 报告分析、案件管理、系统完善 |

### 关键里程碑
- **M1**：基础架构搭建完成（2周）
- **M2**：核心风控引擎开发完成（6周）
- **M3**：完整功能开发完成（10周）
- **M4**：系统测试和优化完成（14周）
- **M5**：系统上线和验收完成（16周）

---

## ✅ 设计文档质量自检

### 完整性检查
- [x] 项目概述和设计目标明确
- [x] 系统架构设计完整（总体架构、技术选型、部署架构）
- [x] 核心服务设计详细（风险引擎、规则引擎、模型管理、监控预警）
- [x] 数据架构设计合理（分层架构、数据库设计、缓存设计）
- [x] 安全设计完善（多层防护、认证授权、数据加密、审计日志）
- [x] 测试策略详细（测试金字塔、分层测试、自动化测试）
- [x] 监控运维考虑（监控架构、关键指标、告警管理）
- [x] 开发计划合理（阶段规划、里程碑设置）

### 可行性检查
- [x] 技术选型成熟稳定，符合金融行业要求
- [x] 架构设计符合高性能、高可靠性要求
- [x] 性能目标可达成（100ms响应时间，1000TPS）
- [x] 安全要求可满足（加密、审计、权限控制）
- [x] 开发计划现实可行（4个月开发周期）

### 测试驱动特色检查
- [x] 体现测试驱动设计理念
- [x] 详细的测试策略和分层测试
- [x] 完整的单元测试、集成测试、E2E测试设计
- [x] 性能测试和安全测试考虑
- [x] 测试自动化和持续集成

### 金融风控特色检查
- [x] 符合金融行业特点（实时性、准确性、合规性）
- [x] 风控业务逻辑完整（评估、规则、模型、监控）
- [x] 性能要求严格（100ms响应时间）
- [x] 安全要求高（数据加密、审计日志）
- [x] 可观测性强（全链路监控、问题追踪）

---

**文档状态**：已完成初稿，待评审
**创建人**：架构师
**创建时间**：2024-04-02
**最后更新**：2024-04-02
**版本号**：v1.0