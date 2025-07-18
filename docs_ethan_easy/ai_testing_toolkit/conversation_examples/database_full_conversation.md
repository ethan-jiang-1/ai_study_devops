# 数据库测试完整对话示例

## 📖 示例背景

这是一个真实的AI对话示例，展示如何使用 [start_here/database_prompts.md](../../start_here/database_prompts.md) 中的关系型数据库模板生成完整的数据库测试方案。

**场景**：为一个电商系统的订单数据库设计测试用例

## 💬 完整对话过程

### 👨‍💻 用户输入（使用模板）

```
# 订单管理数据库测试用例生成

## 你的角色
你是一个专业的数据库测试专家，专注于电商平台的数据库测试设计。

## 数据库基本信息
- **数据库名称**：电商订单数据库
- **数据库类型**：MySQL
- **数据库版本**：MySQL 8.0
- **业务系统**：电商平台订单管理系统
- **数据规模**：千万级订单数据，日增长10万+

## 核心数据表
### 主要业务表
- **orders**：订单主表
  - 用途：存储订单基本信息
  - 关键字段：order_id(主键)、user_id(用户ID)、order_status(订单状态)、total_amount(订单总额)、created_at(创建时间)、updated_at(更新时间)
  - 约束条件：order_id自增主键、user_id外键关联users表、order_status枚举值、total_amount非负数

- **order_items**：订单商品表
  - 用途：存储订单中的商品详情
  - 关键字段：item_id(主键)、order_id(订单ID)、product_id(商品ID)、quantity(数量)、price(单价)、subtotal(小计)
  - 约束条件：item_id自增主键、order_id外键关联orders表、product_id外键关联products表、quantity和price正数

- **payments**：支付记录表
  - 用途：存储订单支付信息
  - 关键字段：payment_id(主键)、order_id(订单ID)、payment_method(支付方式)、payment_status(支付状态)、amount(支付金额)、paid_at(支付时间)
  - 约束条件：payment_id自增主键、order_id外键关联orders表、amount非负数、payment_status枚举值

### 表关系
- **orders → order_items**：一对多关系，一个订单包含多个商品
- **orders → payments**：一对多关系，一个订单可能有多次支付记录
- **orders → users**：多对一关系，一个用户可以有多个订单

## 业务规则
### 数据完整性规则
1. 订单总额必须等于所有商品小计之和
2. 订单状态只能是：pending(待支付)、paid(已支付)、shipping(配送中)、delivered(已送达)、cancelled(已取消)
3. 支付金额不能超过订单总额
4. 已支付的订单不能被删除，只能取消

### 业务逻辑规则
1. 创建订单时必须同时创建订单商品记录
2. 订单状态流转规则：pending → paid → shipping → delivered
3. 取消订单时需要同时处理库存恢复和退款记录
4. 支付成功后订单状态自动从pending变为paid

## 性能要求
- **查询性能**：单个订单查询 < 10ms，订单列表查询 < 100ms
- **并发处理**：支持1000+并发下单操作
- **数据量**：单表千万级记录，分库分表支持
- **响应时间**：订单创建 < 500ms，状态更新 < 100ms

## 关键操作
### 数据操作
- **INSERT操作**：创建订单(orders + order_items + payments)
- **UPDATE操作**：更新订单状态、修改商品数量、支付状态变更
- **DELETE操作**：取消订单（逻辑删除）、清理过期数据
- **SELECT操作**：订单查询、订单列表、统计报表

### 存储过程/函数
- **create_order**：创建订单的存储过程，包含事务处理
- **update_order_status**：更新订单状态的存储过程
- **calculate_order_total**：计算订单总额的函数

## 数据安全要求
- **敏感数据**：用户支付信息需要加密存储
- **访问控制**：不同角色对订单数据的访问权限不同
- **数据备份**：每日全量备份，每小时增量备份
- **审计要求**：所有数据变更需要记录审计日志

## 我需要你生成
1. **数据完整性测试**（主键、外键、唯一约束、检查约束）
2. **CRUD操作测试**（增删改查的各种场景）
3. **事务处理测试**（事务的提交、回滚、隔离级别）
4. **并发控制测试**（锁机制、死锁、并发冲突）
5. **性能测试用例**（查询性能、大数据量处理）
6. **数据安全测试**（权限控制、SQL注入防护）
7. **存储过程测试**（参数验证、异常处理、返回值）
8. **备份恢复测试**（数据备份、恢复验证）

每个测试用例包含：
- 测试目标（明确测试目的）
- 前置条件（数据准备、环境设置）
- 测试步骤（具体的SQL语句和操作）
- 测试数据（输入数据和期望结果）
- 验证方法（如何验证结果正确性）
- 清理步骤（测试后的数据清理）
```

---

### 🤖 AI回复（生成的测试方案）

```
# 电商订单数据库测试方案

## 1. 测试场景概览

### 1.1 数据完整性测试
- **主键约束**：验证主键的唯一性和非空性
- **外键约束**：验证外键的引用完整性
- **唯一约束**：验证业务唯一性字段
- **检查约束**：验证数据范围和格式限制

### 1.2 业务逻辑测试
- **订单创建**：完整的订单创建流程
- **状态流转**：订单状态的合法变更
- **金额计算**：订单总额的正确计算
- **业务规则**：各种业务规则的验证

### 1.3 性能压力测试
- **查询性能**：各类查询的性能表现
- **并发处理**：高并发下的数据一致性
- **大数据量**：海量数据下的系统表现
- **索引效果**：索引对查询性能的影响

## 2. 详细测试用例

### 2.1 数据完整性测试用例

#### TC-001: 订单表主键约束测试
**测试目标**：验证orders表主键order_id的唯一性约束
**优先级**：高
**前置条件**：
- 数据库连接正常
- orders表存在且结构正确
- 清空orders表数据

**测试步骤**：
1. 插入第一条订单记录
2. 尝试插入相同order_id的记录
3. 验证第二条记录插入失败

**测试数据**：
```sql
-- 步骤1：插入第一条记录
INSERT INTO orders (order_id, user_id, order_status, total_amount, created_at) 
VALUES (1001, 1, 'pending', 299.99, NOW());

-- 步骤2：尝试插入重复主键
INSERT INTO orders (order_id, user_id, order_status, total_amount, created_at) 
VALUES (1001, 2, 'pending', 399.99, NOW());
```

**预期结果**：
- 第一条记录插入成功
- 第二条记录插入失败，报错："Duplicate entry '1001' for key 'PRIMARY'"
- orders表中只有一条记录

**验证方法**：
```sql
SELECT COUNT(*) FROM orders WHERE order_id = 1001;
-- 期望结果：1
```

**清理步骤**：
```sql
DELETE FROM orders WHERE order_id = 1001;
```

#### TC-002: 订单商品表外键约束测试
**测试目标**：验证order_items表order_id外键的引用完整性
**优先级**：高
**前置条件**：
- 数据库连接正常
- orders和order_items表存在
- 清空相关表数据

**测试步骤**：
1. 尝试插入引用不存在订单的商品记录
2. 插入正确的订单记录
3. 插入引用存在订单的商品记录
4. 尝试删除被引用的订单记录

**测试数据**：
```sql
-- 步骤1：插入引用不存在订单的商品（应该失败）
INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) 
VALUES (9999, 1001, 2, 99.99, 199.98);

-- 步骤2：插入正确的订单记录
INSERT INTO orders (order_id, user_id, order_status, total_amount, created_at) 
VALUES (1001, 1, 'pending', 199.98, NOW());

-- 步骤3：插入引用存在订单的商品（应该成功）
INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) 
VALUES (1001, 1001, 2, 99.99, 199.98);

-- 步骤4：尝试删除被引用的订单（应该失败）
DELETE FROM orders WHERE order_id = 1001;
```

**预期结果**：
- 步骤1：插入失败，外键约束错误
- 步骤3：插入成功
- 步骤4：删除失败，外键约束错误

**验证方法**：
```sql
SELECT o.order_id, oi.item_id 
FROM orders o 
JOIN order_items oi ON o.order_id = oi.order_id;
-- 期望结果：显示关联的记录
```

**清理步骤**：
```sql
DELETE FROM order_items WHERE order_id = 1001;
DELETE FROM orders WHERE order_id = 1001;
```

#### TC-003: 订单状态枚举约束测试
**测试目标**：验证order_status字段的枚举值约束
**优先级**：高
**前置条件**：
- 数据库连接正常
- orders表存在且有正确的CHECK约束

**测试步骤**：
1. 插入有效状态的订单记录
2. 尝试插入无效状态的订单记录
3. 尝试更新为无效状态

**测试数据**：
```sql
-- 步骤1：插入有效状态（应该成功）
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 299.99, NOW());

-- 步骤2：插入无效状态（应该失败）
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'invalid_status', 299.99, NOW());

-- 步骤3：更新为无效状态（应该失败）
UPDATE orders SET order_status = 'wrong_status' WHERE order_id = LAST_INSERT_ID();
```

**预期结果**：
- 步骤1：插入成功
- 步骤2：插入失败，CHECK约束错误
- 步骤3：更新失败，CHECK约束错误

**验证方法**：
```sql
SELECT order_status FROM orders WHERE order_status NOT IN ('pending', 'paid', 'shipping', 'delivered', 'cancelled');
-- 期望结果：空结果集
```

#### TC-004: 订单总额非负约束测试
**测试目标**：验证total_amount字段的非负约束
**优先级**：高
**前置条件**：数据库连接正常

**测试步骤**：
1. 插入正数金额的订单
2. 插入零金额的订单
3. 尝试插入负数金额的订单

**测试数据**：
```sql
-- 步骤1：正数金额（应该成功）
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 299.99, NOW());

-- 步骤2：零金额（应该成功）
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 0.00, NOW());

-- 步骤3：负数金额（应该失败）
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', -100.00, NOW());
```

**预期结果**：
- 步骤1和2：插入成功
- 步骤3：插入失败，CHECK约束错误

### 2.2 业务逻辑测试用例

#### TC-005: 订单创建完整流程测试
**测试目标**：验证订单创建的完整业务流程
**优先级**：高
**前置条件**：
- 数据库连接正常
- 相关表清空
- 存在有效的用户和商品数据

**测试步骤**：
1. 开启事务
2. 插入订单主记录
3. 插入订单商品记录
4. 计算并验证订单总额
5. 提交事务

**测试数据**：
```sql
START TRANSACTION;

-- 步骤2：插入订单主记录
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 0, NOW());

SET @order_id = LAST_INSERT_ID();

-- 步骤3：插入订单商品记录
INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) VALUES
(@order_id, 1001, 2, 99.99, 199.98),
(@order_id, 1002, 1, 199.99, 199.99),
(@order_id, 1003, 3, 49.99, 149.97);

-- 步骤4：计算并更新订单总额
UPDATE orders 
SET total_amount = (
    SELECT SUM(subtotal) FROM order_items WHERE order_id = @order_id
) 
WHERE order_id = @order_id;

COMMIT;
```

**预期结果**：
- 事务成功提交
- 订单记录创建成功
- 订单商品记录创建成功
- 订单总额正确计算为549.94

**验证方法**：
```sql
SELECT 
    o.order_id,
    o.total_amount,
    SUM(oi.subtotal) as calculated_total,
    COUNT(oi.item_id) as item_count
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = @order_id
GROUP BY o.order_id;
-- 期望：total_amount = calculated_total = 549.94, item_count = 3
```

#### TC-006: 订单状态流转测试
**测试目标**：验证订单状态的合法流转规则
**优先级**：高
**前置条件**：
- 存在状态为'pending'的订单记录
- 创建测试用存储过程update_order_status

**测试步骤**：
1. 测试正常状态流转：pending → paid → shipping → delivered
2. 测试异常状态流转：paid → pending（应该失败）
3. 测试取消状态：任何状态 → cancelled

**测试数据**：
```sql
-- 准备测试数据
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 299.99, NOW());

SET @order_id = LAST_INSERT_ID();

-- 步骤1：正常状态流转测试
CALL update_order_status(@order_id, 'paid');
CALL update_order_status(@order_id, 'shipping');
CALL update_order_status(@order_id, 'delivered');

-- 步骤2：异常状态流转测试（应该失败）
CALL update_order_status(@order_id, 'pending');

-- 步骤3：取消状态测试
UPDATE orders SET order_status = 'paid' WHERE order_id = @order_id;
CALL update_order_status(@order_id, 'cancelled');
```

**预期结果**：
- 步骤1：所有状态变更成功
- 步骤2：状态变更失败，抛出业务异常
- 步骤3：取消状态变更成功

**验证方法**：
```sql
SELECT order_id, order_status, updated_at 
FROM orders 
WHERE order_id = @order_id;
-- 验证最终状态和更新时间
```

#### TC-007: 订单金额计算验证测试
**测试目标**：验证订单总额与商品小计的一致性
**优先级**：高
**前置条件**：存在包含多个商品的订单

**测试步骤**：
1. 创建包含多个商品的订单
2. 修改商品数量
3. 重新计算订单总额
4. 验证金额一致性

**测试数据**：
```sql
-- 步骤1：创建订单和商品
INSERT INTO orders (user_id, order_status, total_amount, created_at) 
VALUES (1, 'pending', 449.97, NOW());

SET @order_id = LAST_INSERT_ID();

INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) VALUES
(@order_id, 1001, 2, 99.99, 199.98),
(@order_id, 1002, 1, 199.99, 199.99),
(@order_id, 1003, 1, 49.99, 49.99);

-- 步骤2：修改商品数量
UPDATE order_items 
SET quantity = 3, subtotal = price * 3 
WHERE order_id = @order_id AND product_id = 1001;

-- 步骤3：重新计算订单总额
UPDATE orders 
SET total_amount = (
    SELECT SUM(subtotal) FROM order_items WHERE order_id = @order_id
),
updated_at = NOW()
WHERE order_id = @order_id;
```

**预期结果**：
- 商品数量更新成功
- 订单总额重新计算正确
- 总额为549.95（199.98 + 99.99 + 199.99 + 49.99）

**验证方法**：
```sql
SELECT 
    o.order_id,
    o.total_amount,
    SUM(oi.subtotal) as calculated_total,
    CASE 
        WHEN o.total_amount = SUM(oi.subtotal) THEN 'PASS'
        ELSE 'FAIL'
    END as validation_result
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = @order_id
GROUP BY o.order_id;
-- 期望：validation_result = 'PASS'
```

### 2.3 并发控制测试用例

#### TC-008: 并发下单测试
**测试目标**：验证多用户同时下单的并发处理能力
**优先级**：高
**前置条件**：
- 数据库支持事务隔离
- 准备并发测试工具

**测试步骤**：
1. 模拟100个用户同时下单
2. 每个用户创建包含相同商品的订单
3. 验证数据一致性
4. 检查是否有死锁或数据冲突

**测试数据**：
```sql
-- 并发测试脚本（伪代码）
DELIMITER //
CREATE PROCEDURE concurrent_order_test(IN user_id INT, IN iteration INT)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT CONCAT('Error in iteration ', iteration, ' for user ', user_id) as error_message;
    END;
    
    START TRANSACTION;
    
    INSERT INTO orders (user_id, order_status, total_amount, created_at) 
    VALUES (user_id, 'pending', 299.99, NOW());
    
    SET @order_id = LAST_INSERT_ID();
    
    INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) 
    VALUES (@order_id, 1001, 2, 99.99, 199.98),
           (@order_id, 1002, 1, 99.99, 99.99);
    
    COMMIT;
    
    SELECT CONCAT('Success for user ', user_id, ' iteration ', iteration) as success_message;
END //
DELIMITER ;

-- 执行并发测试
-- 使用测试工具同时执行多个连接
```

**预期结果**：
- 所有订单创建成功
- 无数据冲突或不一致
- 无死锁发生
- 性能在可接受范围内

**验证方法**：
```sql
SELECT 
    COUNT(*) as total_orders,
    COUNT(DISTINCT order_id) as unique_orders,
    SUM(total_amount) as total_amount
FROM orders 
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 1 HOUR);
-- 验证订单数量和金额正确
```

#### TC-009: 库存并发扣减测试
**测试目标**：验证多用户购买同一商品时的库存扣减正确性
**优先级**：高
**前置条件**：
- 商品库存表存在
- 初始库存为100

**测试步骤**：
1. 50个用户同时购买相同商品，每人购买2件
2. 验证库存正确扣减
3. 检查是否有超卖现象

**测试数据**：
```sql
-- 初始化商品库存
UPDATE products SET stock = 100 WHERE product_id = 1001;

-- 并发库存扣减测试脚本
DELIMITER //
CREATE PROCEDURE concurrent_stock_test(IN user_id INT, IN quantity INT)
BEGIN
    DECLARE current_stock INT;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT CONCAT('Stock error for user ', user_id) as error_message;
    END;
    
    START TRANSACTION;
    
    -- 检查库存
    SELECT stock INTO current_stock FROM products WHERE product_id = 1001 FOR UPDATE;
    
    IF current_stock >= quantity THEN
        -- 扣减库存
        UPDATE products SET stock = stock - quantity WHERE product_id = 1001;
        
        -- 创建订单
        INSERT INTO orders (user_id, order_status, total_amount, created_at) 
        VALUES (user_id, 'pending', quantity * 99.99, NOW());
        
        SET @order_id = LAST_INSERT_ID();
        
        INSERT INTO order_items (order_id, product_id, quantity, price, subtotal) 
        VALUES (@order_id, 1001, quantity, 99.99, quantity * 99.99);
        
        COMMIT;
        SELECT CONCAT('Success for user ', user_id) as success_message;
    ELSE
        ROLLBACK;
        SELECT CONCAT('Insufficient stock for user ', user_id) as error_message;
    END IF;
END //
DELIMITER ;
```

**预期结果**：
- 前50个用户成功下单
- 库存正确扣减为0
- 无超卖现象
- 后续用户收到库存不足提示

**验证方法**：
```sql
SELECT 
    p.product_id,
    p.stock as current_stock,
    SUM(oi.quantity) as sold_quantity,
    100 - SUM(oi.quantity) as expected_stock
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE p.product_id = 1001
GROUP BY p.product_id;
-- 验证 current_stock = expected_stock
```

### 2.4 性能测试用例

#### TC-010: 订单查询性能测试
**测试目标**：验证单个订单查询的响应时间
**优先级**：高
**前置条件**：
- 数据库包含100万条订单记录
- 存在适当的索引

**测试步骤**：
1. 准备100万条测试数据
2. 执行单个订单查询
3. 测量执行时间
4. 重复测试100次取平均值

**测试数据**：
```sql
-- 数据准备脚本
DELIMITER //
CREATE PROCEDURE prepare_test_data()
BEGIN
    DECLARE i INT DEFAULT 1;
    WHILE i <= 1000000 DO
        INSERT INTO orders (user_id, order_status, total_amount, created_at) 
        VALUES (
            FLOOR(RAND() * 10000) + 1,
            ELT(FLOOR(RAND() * 5) + 1, 'pending', 'paid', 'shipping', 'delivered', 'cancelled'),
            ROUND(RAND() * 1000, 2),
            DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 365) DAY)
        );
        SET i = i + 1;
    END WHILE;
END //
DELIMITER ;

CALL prepare_test_data();

-- 性能测试查询
SELECT 
    o.order_id,
    o.user_id,
    o.order_status,
    o.total_amount,
    o.created_at,
    COUNT(oi.item_id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = 500000
GROUP BY o.order_id;
```

**预期结果**：
- 查询响应时间 < 10ms
- 使用了适当的索引
- CPU和内存使用合理

**验证方法**：
```sql
-- 使用EXPLAIN分析查询计划
EXPLAIN SELECT * FROM orders WHERE order_id = 500000;

-- 使用性能监控查看执行时间
SELECT 
    SCHEMA_NAME,
    DIGEST_TEXT,
    EXEC_COUNT,
    AVG_TIMER_WAIT/1000000 as avg_time_ms
FROM performance_schema.events_statements_summary_by_digest
WHERE DIGEST_TEXT LIKE '%orders%'
ORDER BY AVG_TIMER_WAIT DESC;
```

#### TC-011: 订单列表查询性能测试
**测试目标**：验证订单列表分页查询的性能
**优先级**：高
**前置条件**：
- 数据库包含大量订单数据
- 存在复合索引

**测试步骤**：
1. 查询用户的订单列表（分页）
2. 按状态筛选订单
3. 按时间范围筛选订单
4. 测量各种查询的响应时间

**测试数据**：
```sql
-- 用户订单列表查询
SELECT 
    o.order_id,
    o.order_status,
    o.total_amount,
    o.created_at,
    COUNT(oi.item_id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.user_id = 1001
ORDER BY o.created_at DESC
LIMIT 20 OFFSET 0;

-- 状态筛选查询
SELECT COUNT(*) 
FROM orders 
WHERE order_status = 'pending' 
AND created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 时间范围查询
SELECT 
    DATE(created_at) as order_date,
    COUNT(*) as order_count,
    SUM(total_amount) as total_amount
FROM orders
WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(created_at)
ORDER BY order_date DESC;
```

**预期结果**：
- 分页查询响应时间 < 100ms
- 统计查询响应时间 < 200ms
- 所有查询都使用了索引

**验证方法**：
```sql
-- 检查索引使用情况
SHOW INDEX FROM orders;

-- 分析查询性能
EXPLAIN FORMAT=JSON 
SELECT * FROM orders 
WHERE user_id = 1001 
ORDER BY created_at DESC 
LIMIT 20;
```

### 2.5 数据安全测试用例

#### TC-012: SQL注入防护测试
**测试目标**：验证应用程序对SQL注入攻击的防护能力
**优先级**：高
**前置条件**：
- 应用程序连接数据库
- 存在订单查询接口

**测试步骤**：
1. 构造SQL注入攻击语句
2. 通过应用程序接口发送恶意请求
3. 验证数据库未受到攻击
4. 检查日志记录

**测试数据**：
```sql
-- 模拟SQL注入攻击
-- 正常查询：SELECT * FROM orders WHERE user_id = 1001
-- 注入攻击：SELECT * FROM orders WHERE user_id = 1001 OR 1=1 --

-- 测试用例1：联合查询注入
user_id = "1001 UNION SELECT * FROM users --"

-- 测试用例2：布尔盲注
user_id = "1001 AND (SELECT COUNT(*) FROM users) > 0 --"

-- 测试用例3：时间盲注
user_id = "1001 AND (SELECT SLEEP(5)) --"

-- 测试用例4：报错注入
user_id = "1001 AND (SELECT COUNT(*) FROM (SELECT 1 UNION SELECT 2 UNION SELECT 3)x GROUP BY CONCAT(version(),floor(rand()*2))) --"
```

**预期结果**：
- 所有注入攻击都被阻止
- 应用程序返回正常结果或错误信息
- 数据库数据未被泄露或破坏
- 安全日志记录攻击尝试

**验证方法**：
```sql
-- 检查数据库日志
SELECT * FROM mysql.general_log 
WHERE command_type = 'Query' 
AND argument LIKE '%UNION%' 
OR argument LIKE '%---%';

-- 验证数据完整性
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM users;
-- 确认数据量未发生异常变化
```

#### TC-013: 权限控制测试
**测试目标**：验证不同用户角色的数据访问权限
**优先级**：高
**前置条件**：
- 创建不同权限的数据库用户
- 设置表级和列级权限

**测试步骤**：
1. 创建只读用户，测试查询权限
2. 创建应用用户，测试增删改查权限
3. 创建管理员用户，测试所有权限
4. 测试跨权限访问

**测试数据**：
```sql
-- 创建测试用户
CREATE USER 'readonly_user'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'admin_user'@'localhost' IDENTIFIED BY 'password123';

-- 分配权限
GRANT SELECT ON ecommerce.orders TO 'readonly_user'@'localhost';
GRANT SELECT, INSERT, UPDATE ON ecommerce.orders TO 'app_user'@'localhost';
GRANT ALL PRIVILEGES ON ecommerce.* TO 'admin_user'@'localhost';

-- 测试只读用户权限
-- 使用readonly_user连接
SELECT * FROM orders LIMIT 10; -- 应该成功
INSERT INTO orders (user_id, order_status, total_amount) VALUES (1, 'pending', 100); -- 应该失败

-- 测试应用用户权限
-- 使用app_user连接
SELECT * FROM orders LIMIT 10; -- 应该成功
INSERT INTO orders (user_id, order_status, total_amount) VALUES (1, 'pending', 100); -- 应该成功
DELETE FROM orders WHERE order_id = 1; -- 应该失败

-- 测试管理员权限
-- 使用admin_user连接
SELECT * FROM orders; -- 应该成功
DELETE FROM orders WHERE order_id = 1; -- 应该成功
```

**预期结果**：
- 只读用户只能查询数据
- 应用用户能够查询和修改数据，但不能删除
- 管理员用户拥有所有权限
- 权限之外的操作被拒绝

**验证方法**：
```sql
-- 检查用户权限
SHOW GRANTS FOR 'readonly_user'@'localhost';
SHOW GRANTS FOR 'app_user'@'localhost';
SHOW GRANTS FOR 'admin_user'@'localhost';

-- 检查权限表
SELECT * FROM mysql.user WHERE User IN ('readonly_user', 'app_user', 'admin_user');
SELECT * FROM mysql.db WHERE User IN ('readonly_user', 'app_user', 'admin_user');
```

### 2.6 存储过程测试用例

#### TC-014: 订单创建存储过程测试
**测试目标**：验证create_order存储过程的功能和异常处理
**优先级**：高
**前置条件**：
- 存储过程create_order已创建
- 相关表数据正常

**测试步骤**：
1. 测试正常订单创建流程
2. 测试参数验证
3. 测试异常情况处理
4. 测试事务回滚

**测试数据**：
```sql
-- 存储过程定义
DELIMITER //
CREATE PROCEDURE create_order(
    IN p_user_id INT,
    IN p_items JSON,
    OUT p_order_id INT,
    OUT p_result_code INT,
    OUT p_result_message VARCHAR(255)
)
BEGIN
    DECLARE v_total_amount DECIMAL(10,2) DEFAULT 0;
    DECLARE v_item_count INT DEFAULT 0;
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_result_code = -1;
        SET p_result_message = 'Database error occurred';
    END;
    
    START TRANSACTION;
    
    -- 参数验证
    IF p_user_id IS NULL OR p_user_id <= 0 THEN
        SET p_result_code = -2;
        SET p_result_message = 'Invalid user ID';
        ROLLBACK;
        LEAVE proc_label;
    END IF;
    
    -- 计算总金额
    SET v_total_amount = JSON_EXTRACT(p_items, '$.total');
    
    -- 创建订单
    INSERT INTO orders (user_id, order_status, total_amount, created_at) 
    VALUES (p_user_id, 'pending', v_total_amount, NOW());
    
    SET p_order_id = LAST_INSERT_ID();
    
    -- 插入订单商品
    INSERT INTO order_items (order_id, product_id, quantity, price, subtotal)
    SELECT 
        p_order_id,
        JSON_EXTRACT(items.value, '$.product_id'),
        JSON_EXTRACT(items.value, '$.quantity'),
        JSON_EXTRACT(items.value, '$.price'),
        JSON_EXTRACT(items.value, '$.subtotal')
    FROM JSON_TABLE(p_items, '$.items[*]' COLUMNS (
        value JSON PATH '$'
    )) items;
    
    SET p_result_code = 0;
    SET p_result_message = 'Order created successfully';
    
    COMMIT;
END //
DELIMITER ;

-- 测试正常情况
CALL create_order(
    1001,
    JSON_OBJECT(
        'total', 299.98,
        'items', JSON_ARRAY(
            JSON_OBJECT('product_id', 1001, 'quantity', 2, 'price', 99.99, 'subtotal', 199.98),
            JSON_OBJECT('product_id', 1002, 'quantity', 1, 'price', 99.99, 'subtotal', 99.99)
        )
    ),
    @order_id,
    @result_code,
    @result_message
);

SELECT @order_id, @result_code, @result_message;

-- 测试异常情况
CALL create_order(
    NULL,
    JSON_OBJECT('total', 100, 'items', JSON_ARRAY()),
    @order_id2,
    @result_code2,
    @result_message2
);

SELECT @order_id2, @result_code2, @result_message2;
```

**预期结果**：
- 正常情况：订单创建成功，返回订单ID
- 异常情况：返回错误码和错误信息
- 事务正确提交或回滚
- 数据一致性得到保证

**验证方法**：
```sql
-- 验证订单创建结果
SELECT o.*, COUNT(oi.item_id) as item_count
FROM orders o
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_id = @order_id
GROUP BY o.order_id;

-- 验证异常情况无数据残留
SELECT COUNT(*) FROM orders WHERE user_id IS NULL;
```

### 2.7 备份恢复测试用例

#### TC-015: 数据备份测试
**测试目标**：验证数据备份的完整性和可用性
**优先级**：中
**前置条件**：
- 数据库包含完整的业务数据
- 备份工具配置正确

**测试步骤**：
1. 执行全量备份
2. 执行增量备份
3. 验证备份文件完整性
4. 测试备份文件可读性

**测试数据**：
```bash
# 全量备份脚本
mysqldump -u root -p \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    --hex-blob \
    --databases ecommerce > ecommerce_full_backup_$(date +%Y%m%d_%H%M%S).sql

# 增量备份脚本
mysqldump -u root -p \
    --single-transaction \
    --where="updated_at >= '2024-01-01 00:00:00'" \
    --databases ecommerce > ecommerce_incremental_backup_$(date +%Y%m%d_%H%M%S).sql
```

**预期结果**：
- 备份文件创建成功
- 备份文件大小合理
- 备份文件可以正常读取
- 备份过程不影响业务

**验证方法**：
```bash
# 验证备份文件完整性
file ecommerce_full_backup_*.sql
head -20 ecommerce_full_backup_*.sql
tail -20 ecommerce_full_backup_*.sql

# 验证备份内容
grep -c "INSERT INTO" ecommerce_full_backup_*.sql
```

#### TC-016: 数据恢复测试
**测试目标**：验证数据恢复的完整性和准确性
**优先级**：高
**前置条件**：
- 存在有效的备份文件
- 准备恢复测试环境

**测试步骤**：
1. 创建测试数据库
2. 恢复全量备份
3. 验证数据完整性
4. 测试应用程序连接

**测试数据**：
```sql
-- 创建测试数据库
CREATE DATABASE ecommerce_test;

-- 恢复数据
-- mysql -u root -p ecommerce_test < ecommerce_full_backup_20240101_120000.sql

-- 验证数据完整性
USE ecommerce_test;

SELECT 
    table_name,
    table_rows,
    data_length,
    index_length
FROM information_schema.tables
WHERE table_schema = 'ecommerce_test';

-- 验证关键数据
SELECT COUNT(*) as order_count FROM orders;
SELECT COUNT(*) as order_item_count FROM order_items;
SELECT COUNT(*) as payment_count FROM payments;

-- 验证数据关系
SELECT 
    COUNT(DISTINCT o.order_id) as orders_with_items,
    COUNT(DISTINCT p.order_id) as orders_with_payments
FROM orders o
LEFT JOIN order_items oi ON o.order_id = oi.order_id
LEFT JOIN payments p ON o.order_id = p.order_id;
```

**预期结果**：
- 恢复过程无错误
- 数据量与原数据库一致
- 数据关系完整
- 应用程序可正常连接

**验证方法**：
```sql
-- 对比原数据库和恢复数据库
SELECT 
    'original' as source,
    COUNT(*) as order_count,
    SUM(total_amount) as total_amount
FROM ecommerce.orders
UNION ALL
SELECT 
    'restored' as source,
    COUNT(*) as order_count,
    SUM(total_amount) as total_amount
FROM ecommerce_test.orders;
```

## 3. 性能基准和监控

### 3.1 性能基准指标
| 操作类型 | 目标响应时间 | 并发用户数 | 吞吐量 |
|----------|-------------|------------|--------|
| 单订单查询 | < 10ms | 1000 | 10000 QPS |
| 订单列表查询 | < 100ms | 500 | 5000 QPS |
| 订单创建 | < 500ms | 100 | 200 TPS |
| 订单更新 | < 100ms | 200 | 500 TPS |

### 3.2 监控指标
```sql
-- 查询性能监控
SELECT 
    digest_text,
    count_star,
    avg_timer_wait/1000000 as avg_time_ms,
    max_timer_wait/1000000 as max_time_ms
FROM performance_schema.events_statements_summary_by_digest
WHERE digest_text LIKE '%orders%'
ORDER BY avg_timer_wait DESC;

-- 锁等待监控
SELECT 
    waiting_query,
    waiting_age,
    waiting_lock_type,
    blocking_query,
    blocking_age
FROM sys.innodb_lock_waits;

-- 连接数监控
SELECT 
    user,
    host,
    db,
    command,
    time,
    state
FROM information_schema.processlist
WHERE db = 'ecommerce';
```

## 4. 测试环境配置

### 4.1 数据库配置
```sql
-- 测试环境配置
SET innodb_flush_log_at_trx_commit = 1;
SET sync_binlog = 1;
SET transaction_isolation = 'READ-COMMITTED';
SET innodb_lock_wait_timeout = 50;
SET max_connections = 1000;
```

### 4.2 测试数据生成
```sql
-- 生成测试数据的存储过程
DELIMITER //
CREATE PROCEDURE generate_test_data(IN record_count INT)
BEGIN
    DECLARE i INT DEFAULT 1;
    DECLARE batch_size INT DEFAULT 1000;
    
    WHILE i <= record_count DO
        INSERT INTO orders (user_id, order_status, total_amount, created_at) 
        SELECT 
            FLOOR(RAND() * 10000) + 1,
            ELT(FLOOR(RAND() * 5) + 1, 'pending', 'paid', 'shipping', 'delivered', 'cancelled'),
            ROUND(RAND() * 1000, 2),
            DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 365) DAY)
        FROM 
            (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION 
             SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9 UNION SELECT 10) t1,
            (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION 
             SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9 UNION SELECT 10) t2,
            (SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION 
             SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9 UNION SELECT 10) t3
        LIMIT batch_size;
        
        SET i = i + batch_size;
    END WHILE;
END //
DELIMITER ;
```

---

**测试方案总结**：
本测试方案涵盖了电商订单数据库的16个详细测试用例，包括数据完整性、业务逻辑、并发控制、性能测试、数据安全、存储过程和备份恢复等方面。通过执行这些测试用例，可以确保数据库系统的稳定性、性能和数据一致性。
```

---

## 📊 对话分析

### 🎯 用户做得好的地方
1. **数据结构清晰**：详细描述了表结构、字段类型和约束条件
2. **业务规则明确**：提供了具体的业务逻辑和数据验证规则
3. **技术环境准确**：说明了数据库版本、数据规模和性能要求
4. **安全考虑周全**：明确了数据安全、权限控制和审计要求

### 🤖 AI生成质量分析
1. **测试类型全面**：涵盖了数据完整性、业务逻辑、并发、性能、安全等
2. **SQL语句实用**：提供了具体可执行的SQL测试脚本
3. **测试数据丰富**：包含了正常、异常、边界等各种测试数据
4. **验证方法具体**：每个测试用例都有明确的验证方法

### 📈 改进建议
1. **自动化支持**：可以补充数据库测试自动化框架
2. **监控集成**：可以增加与监控系统的集成方案
3. **容灾测试**：可以添加数据库容灾和高可用测试

## 🎓 学习要点

### ✅ 数据库测试的关键技巧
1. **分层测试**：从数据完整性到业务逻辑再到性能测试
2. **并发关注**：重点测试高并发场景下的数据一致性
3. **性能基准**：建立明确的性能基准和监控指标
4. **安全第一**：充分考虑数据安全和权限控制

### ✅ 提高测试质量的方法
1. **业务理解**：深入理解业务规则和数据关系
2. **场景完整**：考虑正常、异常、边界各种情况
3. **工具结合**：使用SQL脚本、存储过程、监控工具
4. **持续监控**：建立持续的性能和安全监控

---

**下一步**：查看 [checklists](../../checklists/) 目录，用业务专家检查清单验证这个数据库测试方案是否符合业务规则要求！