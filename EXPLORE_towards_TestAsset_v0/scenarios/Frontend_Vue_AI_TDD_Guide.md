# Frontend Vue AI+TDD 实战指南

> **目标受众**：熟练使用Vue.js和现代AI开发工具（CURSOR、WindSurf、Claude Code等）的前端开发者  
> **核心目标**：掌握在Vue项目中用测试驾驭AI生成高质量组件代码的实战技能

## 1. Vue场景深度分析与挑战（1500字）

### 1.1 Vue生态的特点和复杂性分析

Vue.js作为渐进式前端框架，具有独特的特性组合，这些特性为AI+TDD带来了特殊的挑战和机遇：

#### Vue的核心特性影响
**响应式系统的复杂性**：
- Vue的响应式数据绑定机制对AI来说是"黑盒"
- `ref()`、`reactive()`、`computed()`的行为差异AI难以精确理解
- 响应式更新的时机和副作用对测试设计要求很高

**组件系统的层次性**：
- 父子组件通信（props/emit）的测试需要特殊处理
- 插槽（slots）和作用域插槽的测试复杂度高
- Provide/Inject模式的依赖关系AI难以追踪

**生命周期的异步性**：
- `onMounted`、`onUpdated`等钩子的执行时机
- 异步组件和Suspense的加载状态
- Keep-alive组件的激活/失活状态

#### Vue生态工具链的考虑
**构建工具集成**：
- Vite的HMR机制与测试环境的配合
- TypeScript集成的类型检查和推导
- Vue SFC（单文件组件）的编译和测试

**状态管理方案**：
- Vuex/Pinia的状态变更测试
- 模块化store的Mock策略
- 全局状态与组件状态的边界

### 1.2 AI在Vue场景下的具体局限性和挑战

#### 理解层面的局限性

**1. 响应式理解偏差**
```javascript
// AI常常生成这样的错误测试
test('should update count', () => {
  const { count } = setup();
  count.value = 5;
  // ❌ AI可能期望立即生效，忽略nextTick
  expect(wrapper.text()).toContain('5');
});

// 正确的理解应该考虑Vue的更新机制
test('should update count after nextTick', async () => {
  const { count } = setup();
  count.value = 5;
  await nextTick(); // ✅ 等待DOM更新
  expect(wrapper.text()).toContain('5');
});
```

**2. 组件边界理解模糊**
AI倾向于测试组件的内部实现细节，而不是组件的公共接口：

```javascript
// ❌ AI典型的错误模式：测试内部状态
test('internal state test', () => {
  expect(wrapper.vm.internalCounter).toBe(0); // 测试私有状态
  expect(wrapper.vm.$data.isLoading).toBe(false); // 访问内部数据
});

// ✅ 正确的组件接口测试
test('should emit increment event when button clicked', async () => {
  await wrapper.find('[data-testid="increment-btn"]').trigger('click');
  expect(wrapper.emitted('increment')).toBeTruthy();
});
```

**3. 异步操作处理不当**
```javascript
// ❌ AI经常忽略异步等待
test('async operation', () => {
  wrapper.vm.loadData(); // 异步方法
  expect(wrapper.vm.data).toBeDefined(); // 立即检查，通常失败
});

// ✅ 正确的异步测试处理
test('should load data successfully', async () => {
  const loadDataPromise = wrapper.vm.loadData();
  expect(wrapper.vm.isLoading).toBe(true);
  
  await loadDataPromise;
  expect(wrapper.vm.isLoading).toBe(false);
  expect(wrapper.vm.data).toBeDefined();
});
```

#### 技术实现层面的挑战

**1. Mock策略的复杂性**
Vue组件的依赖关系复杂，AI往往选择错误的Mock点：

```javascript
// ❌ AI可能过度Mock
const mockStore = {
  state: { user: mockUser },
  getters: { isLoggedIn: jest.fn() },
  mutations: { setUser: jest.fn() },
  actions: { login: jest.fn() }
};

// ✅ 更好的策略：只Mock必要的外部依赖
const mockApiService = {
  getUserProfile: jest.fn().mockResolvedValue(mockUser)
};
```

**2. 事件系统的测试复杂度**
Vue的事件系统包括DOM事件、自定义事件、全局事件总线等多个层次：

```javascript
// AI经常混淆不同类型的事件测试
// DOM事件 vs 组件事件 vs 全局事件的不同处理方式
```

### 1.3 人类开发者需要重点关注的核心方面

#### 设计层面的关注点

**1. 测试边界的定义**
- **组件职责边界**：明确组件负责什么，不负责什么
- **状态管理边界**：区分组件内状态和全局状态
- **副作用边界**：识别和隔离外部依赖

**2. 测试策略的选择**
- **单元测试**：组件逻辑、计算属性、方法
- **集成测试**：组件间通信、路由跳转、API调用
- **用户行为测试**：真实用户交互路径

#### 质量把控的关注点

**1. AI生成测试的常见陷阱**
- 过度依赖DOM查询而忽略组件接口
- 测试实现细节而不是行为
- 忽略异步操作的正确处理
- Mock不当导致测试失去意义

**2. 业务价值的验证**
- 测试是否覆盖了真实的用户场景
- 测试是否能在重构时提供保护
- 测试是否作为组件使用说明

### 1.4 与其他前端框架的差异化考虑

#### Vue vs React的测试差异
**状态管理方式**：
- Vue的响应式 vs React的不可变状态
- 测试状态变化的不同策略
- 副作用处理的不同模式

**组件通信**：
- Vue的props/emit vs React的props/callback
- 测试组件通信的不同方法
- 事件系统的差异影响

#### Vue vs Angular的测试差异
**依赖注入**：
- Vue的provide/inject vs Angular的DI系统
- Mock策略的不同考虑
- 测试隔离的不同要求

**生命周期**：
- Vue的Composition API vs Angular的生命周期钩子
- 测试时机的不同选择
- 异步处理的不同模式

### 1.5 Vue版本演进的考虑

#### Vue 2 vs Vue 3的测试差异
**Composition API的影响**：
```javascript
// Vue 2 Options API的测试方式
test('vue 2 component', () => {
  expect(wrapper.vm.computedValue).toBe(expected);
});

// Vue 3 Composition API的测试方式
test('vue 3 composition', () => {
  const { result } = renderComposition(() => {
    return useCounter();
  });
  expect(result.current.count.value).toBe(0);
});
```

**响应式系统的变化**：
- Proxy vs Object.defineProperty
- 测试响应式数据的不同方式
- 性能测试的不同关注点

#### 工具链演进的影响
**测试工具的选择**：
- Vue Test Utils 2.x的新特性
- Vitest vs Jest的选择考虑
- Testing Library的Vue适配

**构建工具的影响**：
- Vite的测试集成
- 热更新对测试的影响
- TypeScript集成的考虑

---

**小结**：Vue场景下的AI+TDD需要深入理解Vue的响应式系统、组件模型和生态工具链。AI的主要局限在于对Vue特性的理解偏差和异步操作的处理不当。人类开发者需要重点关注测试边界定义、质量把控和业务价值验证。

## 2. 测试计划设计指导：从业务需求到AI可理解的测试蓝图（2000字）

### 2.1 测试计划MD的核心思想

在Vue+AI+TDD的实战中，**测试计划MD是人类驾驭AI的核心工具**。它起到了"翻译器"的作用：将模糊的业务需求转化为AI能够理解和执行的结构化指令。

#### 测试计划的三重价值

**1. 业务理解的结构化表达**
```markdown
# 示例：将模糊需求转为清晰计划

## 业务需求（模糊）
"用户购物车需要支持添加商品、修改数量、删除商品，还要显示总价"

## 测试计划（结构化）
### 购物车组件测试计划

#### 核心功能测试
1. **商品添加功能**
   - 场景：用户点击"加入购物车"按钮
   - 预期：商品出现在购物车列表中
   - 边界：重复添加同一商品应该累加数量

2. **数量修改功能** 
   - 场景：用户通过+/-按钮或直接输入修改数量
   - 预期：数量实时更新，总价重新计算
   - 边界：数量不能小于1，不能超过库存限制

3. **商品删除功能**
   - 场景：用户点击删除按钮
   - 预期：商品从列表移除，总价重新计算
   - 边界：删除最后一个商品后显示空购物车状态
```

**2. AI理解的上下文补充**
```markdown
## Vue技术上下文
- 组件类型：Composition API + TypeScript
- 状态管理：Pinia store
- UI库：Element Plus
- 数据流：props传入商品信息，emit事件到父组件

## 业务上下文  
- 商品类型：包含id、name、price、stock、image等属性
- 价格计算：考虑折扣、运费、税费
- 用户权限：普通用户vs VIP用户的购物车限制
```

**3. 测试策略的预先规划**
```markdown
## 测试分层策略
### 单元测试重点
- 购物车状态计算逻辑（总价、总数量）
- 商品操作的状态变更（添加、修改、删除）
- 边界条件处理（空购物车、库存限制）

### 集成测试重点  
- 与商品服务API的交互
- 与用户认证状态的联动
- 与订单系统的数据传递

### 用户行为测试重点
- 完整的购物流程（浏览→添加→修改→结算）
- 异常情况的用户体验（网络错误、库存不足）
- 不同设备和浏览器的兼容性
```

### 2.2 业务场景到测试场景的转换方法论

#### 方法1：用户故事分解法

**Step 1: 识别用户角色和目标**
```markdown
## 用户故事映射
### 主要用户角色
- **普通消费者**：追求便利和价格透明
- **VIP会员**：享受特殊折扣和优先服务  
- **企业采购**：批量购买和发票管理

### 核心用户目标
- 快速找到想要的商品
- 比较不同商品的价格和属性
- 方便地管理购物车内容
- 顺畅地完成购买流程
```

**Step 2: 场景路径分析**
```markdown
## 关键用户路径
### 路径1：首次购物用户
浏览商品 → 查看详情 → 加入购物车 → 查看购物车 → 去结算

### 路径2：老用户快速购买
搜索商品 → 直接加购物车 → 修改数量 → 立即购买

### 路径3：比较购物用户  
收藏商品 → 加入购物车 → 继续浏览 → 修改购物车 → 最终决策
```

**Step 3: 异常和边界场景识别**
```markdown
## 异常场景分析
### 技术异常
- 网络超时导致添加失败
- 库存实时更新导致的数据不一致
- 登录状态过期的处理

### 业务异常
- 商品下架但仍在购物车中
- 价格变更的用户通知
- 库存不足的替代商品推荐

### 用户行为异常
- 恶意刷购物车接口
- 超大数量的商品添加
- 同时在多个设备操作购物车
```

#### 方法2：业务规则矩阵法

```markdown
## 购物车业务规则矩阵

| 用户类型 | 商品类型 | 数量限制 | 价格计算 | 特殊行为 |
|---------|---------|---------|---------|---------|
| 普通用户 | 普通商品 | 单品≤10 | 原价 | 无 |
| 普通用户 | 限购商品 | 单品≤2 | 原价 | 提示限购 |
| VIP用户 | 普通商品 | 单品≤20 | 9.5折 | 优先库存 |
| VIP用户 | 专享商品 | 单品≤5 | 专享价 | 特殊标识 |
| 企业用户 | 批发商品 | 单品≥100 | 批发价 | 发票选项 |

## 对应测试用例设计
每个矩阵格子对应一个测试场景分组，确保业务规则的完整覆盖。
```

### 2.3 AI可理解的测试计划结构模板

#### 标准模板结构
```markdown
# [组件名称] 测试计划

## 1. 业务背景
### 功能描述
[1-2句话描述组件的核心功能]

### 用户价值
[说明这个组件为用户解决什么问题]

### 技术边界
[明确组件负责什么，不负责什么]

## 2. 技术上下文
### Vue技术栈
- Vue版本：3.x
- API风格：Composition API / Options API
- TypeScript：是/否
- UI库：[具体库名和版本]
- 状态管理：[Vuex/Pinia/local state]

### 依赖关系
- 父组件：[组件名和传入的props]
- 子组件：[使用的子组件列表]
- 外部服务：[API接口、第三方库等]

## 3. 测试场景规划
### 3.1 核心功能测试
[按功能模块列出测试场景]

### 3.2 交互行为测试  
[用户操作和界面响应的测试]

### 3.3 状态管理测试
[数据变更和响应式更新的测试]

### 3.4 异常处理测试
[错误状态和边界条件的测试]

## 4. 测试数据设计
### 4.1 Mock数据结构
[定义测试用的数据格式]

### 4.2 边界值设计
[关键数值的边界情况]

### 4.3 异常数据设计
[用于测试错误处理的数据]

## 5. 测试策略
### 5.1 单元测试策略
[组件内部逻辑的测试方法]

### 5.2 集成测试策略
[组件间协作的测试方法]

### 5.3 用户行为测试策略
[端到端交互的测试方法]

## 6. 验收标准
### 6.1 功能验收
[每个功能的验收条件]

### 6.2 性能验收
[响应时间、渲染性能等]

### 6.3 兼容性验收
[浏览器、设备等兼容性要求]
```

#### 模板使用指南

**填写原则**：
1. **具体化**：避免模糊描述，提供具体的数据和场景
2. **结构化**：使用清晰的层次和分类
3. **可执行**：每个描述都能转化为具体的测试代码
4. **可验证**：每个预期结果都是明确可检查的

**AI引导技巧**：
```markdown
## 向AI提供测试计划的最佳实践

### 1. 分阶段提供
不要一次性给AI整个测试计划，而是按模块分批提供：
- 第一轮：核心功能的基础测试
- 第二轮：交互行为的详细测试  
- 第三轮：异常和边界的完善测试

### 2. 明确输出要求
在每个阶段明确告诉AI期望的输出格式：
- 测试文件的组织结构
- 测试用例的命名约定
- 断言的详细程度
- Mock的使用策略

### 3. 提供示例代码
给AI提供期望的代码风格示例：
- 组件导入和初始化的标准方式
- 测试数据创建的工厂方法
- 异步操作测试的处理模式
- 错误情况的验证方法
```

### 2.4 从测试计划到AI指令的转换技巧

#### 技巧1：分层递进的指令设计

```markdown
# 第一层：框架搭建指令
"根据测试计划，请为购物车组件创建基础测试文件结构，包含：
- 测试文件的导入和初始化部分
- 主要测试套件的骨架（describe块）
- 每个功能模块的测试分组
- 基础的Mock设置"

# 第二层：功能实现指令  
"现在实现'商品添加功能'的测试用例：
- 测试用例1：正常添加商品到空购物车
- 测试用例2：添加商品到已有商品的购物车
- 测试用例3：重复添加同一商品（数量累加）
每个用例需要包含AAA结构和具体的断言"

# 第三层：完善优化指令
"请检查并完善刚才的测试用例：
- 添加边界条件测试（库存为0、数量上限）
- 优化断言的精确性（验证具体字段而非整体对象）
- 确保所有异步操作都有proper等待"
```

#### 技巧2：上下文保持策略

```markdown
## 长对话中的上下文管理

### 关键信息摘要
在每轮对话开始前，提供关键信息摘要：
"继续购物车组件测试开发。当前上下文：
- 组件：Vue3 + Composition API + TypeScript  
- 状态管理：Pinia store (CartStore)
- 已完成：基础添加功能测试
- 当前任务：实现数量修改功能测试"

### 代码片段引用
引用之前生成的关键代码片段：
"基于之前创建的setupCartTest工厂方法：
[贴出相关代码]
现在请为数量修改功能添加测试..."
```

#### 技巧3：质量标准的嵌入

```markdown
## 在指令中嵌入质量要求

### 明确禁止事项
"生成测试时请遵循以下约束：
- 禁止测试Vue内部实现细节（如$data、$refs）
- 禁止使用sleep或setTimeout等不确定等待
- 禁止直接访问组件私有方法或状态
- 禁止在单个测试中验证多个不相关的行为"

### 明确期望标准
"每个测试用例必须满足：
- 测试名称清晰描述业务场景
- 使用data-testid而非CSS类名定位元素
- 异步操作使用await和nextTick
- 断言验证具体的业务结果而非技术细节"
```

---

**小结**：测试计划设计是AI+TDD成功的关键起点。通过结构化的业务分析、规范化的计划模板和策略性的AI指令设计，可以显著提升AI生成测试代码的质量和业务相关性。关键在于将人类的业务理解转化为AI能够执行的技术指令。

## 3. Review方法论：Vue特定的AI测试代码质量把关（2000字）

### 3.1 Vue场景下的Review重点和优先级

在Vue+AI协作中，人类的Review职责是确保AI生成的测试代码既符合Vue框架特性，又满足业务需求。Review不是简单的代码检查，而是对AI理解偏差的系统性纠正。

#### Review的三层检查体系

**第一层：Vue框架特性检查（技术正确性）**
```javascript
// ❌ AI常见错误：忽略Vue的响应式更新机制
test('should update display when count changes', () => {
  wrapper.vm.count = 5;
  expect(wrapper.text()).toContain('5'); // 可能失败，未等待更新
});

// ✅ Review修正：确保等待Vue更新周期
test('should update display when count changes', async () => {
  wrapper.vm.count = 5;
  await wrapper.vm.$nextTick(); // 或 await nextTick()
  expect(wrapper.text()).toContain('5');
});
```

**第二层：组件接口检查（设计合理性）**
```javascript
// ❌ AI倾向于测试实现细节
test('internal state test', () => {
  expect(wrapper.vm._internalFlag).toBe(true); // 测试私有状态
  expect(wrapper.vm.$refs.childComponent.someMethod).toBeDefined();
});

// ✅ Review修正：测试公共接口和行为
test('should emit selection-change when item is selected', async () => {
  await wrapper.find('[data-testid="item-1"]').trigger('click');
  expect(wrapper.emitted('selection-change')).toBeTruthy();
  expect(wrapper.emitted('selection-change')[0][0]).toEqual({ id: 1, name: 'Item 1' });
});
```

**第三层：业务逻辑检查（业务正确性）**
```javascript
// ❌ AI可能遗漏业务规则
test('should add item to cart', () => {
  wrapper.vm.addToCart(item);
  expect(wrapper.vm.cartItems).toContain(item);
});

// ✅ Review补充：考虑完整的业务逻辑
test('should add item to cart with quantity limit check', () => {
  const item = { id: 1, name: 'Product', stock: 5 };
  
  // 正常添加
  wrapper.vm.addToCart(item, 3);
  expect(wrapper.vm.cartItems[0].quantity).toBe(3);
  
  // 超过库存限制
  expect(() => wrapper.vm.addToCart(item, 10))
    .toThrow('Quantity exceeds available stock');
});
```

### 3.2 常见AI错误模式的识别和修复

#### 错误模式1：响应式数据处理不当

**识别特征**：
- 直接修改响应式数据后立即断言
- 忽略computed属性的计算延迟
- 未考虑watch回调的异步执行

**修复策略**：
```javascript
// AI原始代码
test('computed value update', () => {
  wrapper.vm.baseValue = 10;
  expect(wrapper.vm.computedValue).toBe(20); // 可能不稳定
});

// Review修复版本
test('computed value update', async () => {
  wrapper.vm.baseValue = 10;
  await wrapper.vm.$nextTick(); // 确保computed重新计算
  expect(wrapper.vm.computedValue).toBe(20);
});

// 更好的做法：测试结果而非过程
test('should display doubled value', async () => {
  await wrapper.setData({ baseValue: 10 });
  expect(wrapper.find('[data-testid="display-value"]').text()).toBe('20');
});
```

#### 错误模式2：事件处理测试不完整

**识别特征**：
- 只测试事件触发，不验证事件参数
- 忽略事件的冒泡和修饰符
- 未测试自定义事件的完整流程

**修复策略**：
```javascript
// AI原始代码
test('click event', async () => {
  await wrapper.find('button').trigger('click');
  expect(wrapper.emitted('click')).toBeTruthy();
});

// Review完善版本
test('should emit add-to-cart with correct product data', async () => {
  const product = { id: 123, name: 'Test Product', price: 99.99 };
  await wrapper.setProps({ product });
  
  await wrapper.find('[data-testid="add-to-cart-btn"]').trigger('click');
  
  const emittedEvents = wrapper.emitted('add-to-cart');
  expect(emittedEvents).toHaveLength(1);
  expect(emittedEvents[0][0]).toEqual({
    product: product,
    quantity: 1,
    timestamp: expect.any(Number)
  });
});
```

#### 错误模式3：异步操作测试缺陷

**识别特征**：
- 异步方法调用后立即断言
- 未正确Mock异步依赖
- 忽略loading状态的测试

**修复策略**：
```javascript
// AI原始代码
test('load data', () => {
  wrapper.vm.loadUserProfile();
  expect(wrapper.vm.userProfile).toBeDefined(); // 立即检查异步结果
});

// Review修复版本
test('should load user profile with loading states', async () => {
  const mockUserData = { id: 1, name: 'John Doe' };
  const mockApiCall = jest.fn().mockResolvedValue(mockUserData);
  wrapper.vm.apiService = { getUserProfile: mockApiCall };
  
  // 测试loading状态
  const loadPromise = wrapper.vm.loadUserProfile();
  expect(wrapper.vm.isLoading).toBe(true);
  expect(wrapper.find('[data-testid="loading-spinner"]').exists()).toBe(true);
  
  // 等待异步完成
  await loadPromise;
  expect(wrapper.vm.isLoading).toBe(false);
  expect(wrapper.vm.userProfile).toEqual(mockUserData);
  expect(mockApiCall).toHaveBeenCalledWith(wrapper.vm.userId);
});
```

### 3.3 业务逻辑正确性的验证方法

#### 方法1：业务规则矩阵验证

```javascript
// 基于之前定义的业务规则矩阵，创建对应的测试验证
describe('Shopping Cart Business Rules', () => {
  const testMatrix = [
    // [userType, productType, quantity, expectedPrice, expectedBehavior]
    ['normal', 'regular', 5, 500, 'normal_checkout'],
    ['normal', 'limited', 3, 300, 'quantity_warning'], 
    ['vip', 'regular', 15, 712.5, 'vip_discount'], // 5%折扣
    ['vip', 'exclusive', 2, 200, 'exclusive_access'],
    ['enterprise', 'bulk', 150, 1350, 'bulk_pricing'] // 批发价
  ];
  
  testMatrix.forEach(([userType, productType, quantity, expectedPrice, expectedBehavior]) => {
    test(`should handle ${userType} user buying ${quantity} ${productType} products`, async () => {
      // 设置用户类型和商品类型
      await setupUserType(userType);
      const product = createProduct(productType, 10); // 单价10元
      
      // 执行添加操作
      await wrapper.vm.addToCart(product, quantity);
      
      // 验证价格计算
      expect(wrapper.vm.totalPrice).toBe(expectedPrice);
      
      // 验证特殊行为
      switch(expectedBehavior) {
        case 'quantity_warning':
          expect(wrapper.find('[data-testid="quantity-warning"]').exists()).toBe(true);
          break;
        case 'vip_discount':
          expect(wrapper.find('[data-testid="vip-badge"]').exists()).toBe(true);
          break;
        case 'exclusive_access':
          expect(wrapper.vm.hasExclusiveAccess).toBe(true);
          break;
      }
    });
  });
});
```

#### 方法2：用户旅程完整性验证

```javascript
// 验证完整的业务流程，而不是孤立的功能点
describe('Complete Shopping Journey', () => {
  test('first-time user complete purchase flow', async () => {
    // Step 1: 用户浏览商品
    await wrapper.vm.browseProducts('electronics');
    expect(wrapper.vm.currentCategory).toBe('electronics');
    
    // Step 2: 查看商品详情
    const product = wrapper.vm.products[0];
    await wrapper.vm.viewProductDetail(product.id);
    expect(wrapper.vm.currentProduct).toEqual(product);
    
    // Step 3: 添加到购物车
    await wrapper.vm.addToCart(product, 2);
    expect(wrapper.vm.cartItems).toHaveLength(1);
    expect(wrapper.vm.cartItems[0].quantity).toBe(2);
    
    // Step 4: 查看购物车
    await wrapper.vm.viewCart();
    expect(wrapper.vm.currentView).toBe('cart');
    
    // Step 5: 修改数量
    await wrapper.vm.updateCartItemQuantity(product.id, 3);
    expect(wrapper.vm.cartItems[0].quantity).toBe(3);
    
    // Step 6: 进入结算
    await wrapper.vm.proceedToCheckout();
    expect(wrapper.vm.currentView).toBe('checkout');
    expect(wrapper.vm.checkoutData.items).toEqual(wrapper.vm.cartItems);
    
    // 验证整个流程的数据一致性
    expect(wrapper.vm.checkoutData.totalAmount).toBe(product.price * 3);
  });
});
```

### 3.4 Review效率提升技巧

#### 技巧1：分层Review策略

```markdown
## Review优先级排序
### P0 - 阻断性问题（必须修复）
- Vue框架使用错误（如响应式处理不当）
- 业务逻辑严重错误（如价格计算错误）
- 测试无法正确执行或始终通过

### P1 - 重要问题（建议修复）
- 测试覆盖不完整（缺少边界条件）
- 断言精确度不足（弱断言）
- Mock策略不合理

### P2 - 优化问题（有时间再修复）
- 测试代码风格不统一
- 性能优化机会
- 更好的测试组织结构
```

#### 技巧2：批量问题模式识别

```javascript
// 创建Review辅助工具，快速识别常见问题
const reviewChecklist = {
  // 检查Vue特性
  checkVueFeatures: (testCode) => {
    const issues = [];
    
    if (testCode.includes('wrapper.vm.') && !testCode.includes('nextTick')) {
      issues.push('可能需要等待Vue更新周期');
    }
    
    if (testCode.includes('$refs') || testCode.includes('_')) {
      issues.push('可能测试了内部实现细节');
    }
    
    if (testCode.includes('setTimeout') || testCode.includes('sleep')) {
      issues.push('使用了不确定的等待方式');
    }
    
    return issues;
  },
  
  // 检查业务逻辑
  checkBusinessLogic: (testCode, businessRules) => {
    const issues = [];
    
    businessRules.forEach(rule => {
      if (!testCode.includes(rule.keyword)) {
        issues.push(`可能遗漏业务规则: ${rule.description}`);
      }
    });
    
    return issues;
  }
};
```

#### 技巧3：AI辅助Review

```markdown
## 让AI帮助Review的Prompt模板

### Review Prompt 1：技术问题检查
"请检查以下Vue测试代码是否存在技术问题：
1. 响应式数据更新是否正确等待
2. 是否测试了内部实现细节而非公共接口
3. 异步操作是否有proper的等待和Mock
4. 事件测试是否完整验证了参数

[贴入测试代码]

请指出具体问题并提供修复建议。"

### Review Prompt 2：业务逻辑检查
"基于以下业务规则：
[贴入业务规则描述]

请检查测试代码是否正确覆盖了所有业务场景：
[贴入测试代码]

重点关注：
- 是否遗漏了边界条件
- 是否考虑了异常情况
- 是否验证了业务计算的正确性"
```

### 3.5 Review结果的标准化记录

#### Review记录模板

```markdown
## Vue组件测试Review报告

### 基本信息
- 组件名称：[组件名]
- Review日期：[日期]
- Review人员：[姓名]
- AI工具：[使用的AI工具]

### 发现问题分类

#### Vue框架问题
| 问题描述 | 严重程度 | 修复建议 | 状态 |
|---------|---------|---------|------|
| 未等待nextTick就断言DOM变化 | P0 | 添加await nextTick() | 待修复 |
| 测试$refs获取子组件实例 | P1 | 改为测试emit事件 | 已修复 |

#### 业务逻辑问题
| 问题描述 | 严重程度 | 修复建议 | 状态 |
|---------|---------|---------|------|
| 未测试VIP用户折扣计算 | P0 | 添加VIP用户测试用例 | 待修复 |
| 缺少库存不足的边界测试 | P1 | 补充库存=0的测试 | 已修复 |

#### 测试质量问题
| 问题描述 | 严重程度 | 修复建议 | 状态 |
|---------|---------|---------|------|
| 使用弱断言toContain | P1 | 改为精确断言toBe | 待修复 |
| 测试命名不清晰 | P2 | 使用业务场景描述 | 已修复 |

### Review总结
- 总问题数：6个
- P0问题：2个（必须修复）
- P1问题：3个（建议修复）
- P2问题：1个（优化建议）

### 改进建议
1. 建立Vue特性检查清单，避免响应式相关问题
2. 加强业务规则理解，确保测试覆盖完整
3. 优化AI Prompt，明确禁止测试内部实现
```

---

**小结**：Vue场景下的Review是确保AI生成测试质量的关键环节。通过分层检查体系、常见错误模式识别和标准化记录，可以系统性地提升AI生成测试的技术正确性和业务价值。Review不仅是质量把关，更是AI协作能力的持续改进过程。

## 4. 业务场景深度结合：从真实需求到Vue测试实现（2000字）

### 4.1 电商场景：购物车组件的业务复杂性

电商购物车是Vue组件测试的经典场景，但真实的业务需求远比"添加、删除、修改"复杂。理解这些业务复杂性是设计有效测试的前提。

#### 真实电商购物车的业务规则层次

**第一层：基础交互规则**
```javascript
// 看似简单的"添加商品"背后的业务逻辑
const addToCartBusinessRules = {
  // 库存检查
  stockValidation: {
    rule: "添加数量不能超过当前库存",
    exceptions: ["预售商品", "虚拟商品", "定制商品"],
    edgeCases: ["库存实时变化", "多人同时购买", "库存预留机制"]
  },
  
  // 用户权限
  userPermissions: {
    guest: { maxItems: 5, maxQuantity: 10 },
    registered: { maxItems: 20, maxQuantity: 100 },
    vip: { maxItems: 50, maxQuantity: 500, earlyAccess: true },
    enterprise: { maxItems: 1000, bulkPricing: true }
  },
  
  // 商品限制
  productLimitations: {
    limitedTime: "限时商品检查活动时间",
    limitedQuantity: "限购商品检查用户历史购买",
    ageRestricted: "年龄限制商品检查用户认证",
    regionLocked: "地区限制商品检查收货地址"
  }
};
```

**第二层：动态价格计算**
```javascript
// 价格计算的业务复杂性
const priceCalculationRules = {
  // 基础价格
  basePrice: "商品原价",
  
  // 折扣层次（按优先级应用）
  discounts: [
    { type: "flashSale", priority: 1, condition: "限时抢购" },
    { type: "vipDiscount", priority: 2, condition: "会员折扣" },
    { type: "bulkDiscount", priority: 3, condition: "批量优惠" },
    { type: "coupon", priority: 4, condition: "优惠券" },
    { type: "points", priority: 5, condition: "积分抵扣" }
  ],
  
  // 附加费用
  additionalFees: {
    shipping: "运费计算（基于重量、距离、时效）",
    tax: "税费计算（基于地区、商品类型）",
    insurance: "保险费用（可选）",
    gift: "礼品包装费（可选）"
  },
  
  // 特殊情况
  specialCases: {
    freeShipping: "满减免运费条件",
    bundleOffer: "套装商品优惠",
    crossPromotion: "跨品类促销",
    loyaltyProgram: "忠诚度计划奖励"
  }
};
```

**第三层：业务流程集成**
```javascript
// 购物车与其他业务系统的集成
const integrationPoints = {
  // 用户系统集成
  userService: {
    authentication: "登录状态验证",
    profile: "用户等级、地址、偏好",
    history: "购买历史、浏览记录",
    social: "社交分享、推荐好友"
  },
  
  // 商品系统集成
  productService: {
    inventory: "实时库存查询",
    pricing: "动态价格获取",
    recommendation: "相关商品推荐",
    reviews: "商品评价集成"
  },
  
  // 营销系统集成
  marketingService: {
    promotions: "实时促销活动",
    personalization: "个性化推荐",
    analytics: "行为数据收集",
    ab_testing: "A/B测试分流"
  }
};
```

#### 业务规则转化为Vue测试策略

**策略1：分层测试对应业务层次**
```javascript
// 第一层：基础交互规则的单元测试
describe('Cart Component - Basic Interactions', () => {
  describe('Stock Validation', () => {
    test('should prevent adding more items than available stock', async () => {
      const product = createProduct({ id: 1, stock: 3 });
      await wrapper.setProps({ products: [product] });
      
      // 正常添加
      await wrapper.vm.addToCart(product.id, 2);
      expect(wrapper.vm.cartItems[0].quantity).toBe(2);
      
      // 超出库存
      await expect(wrapper.vm.addToCart(product.id, 5))
        .rejects.toThrow('Insufficient stock');
    });
    
    test('should handle concurrent stock updates', async () => {
      const product = createProduct({ id: 1, stock: 5 });
      
      // 模拟外部库存更新
      const stockUpdatePromise = mockStockService.updateStock(product.id, 2);
      const addToCartPromise = wrapper.vm.addToCart(product.id, 4);
      
      await Promise.all([stockUpdatePromise, addToCartPromise]);
      
      // 验证处理了并发冲突
      expect(wrapper.vm.error).toContain('Stock updated by another user');
    });
  });
});

// 第二层：价格计算的集成测试
describe('Cart Component - Price Calculation', () => {
  test('should apply discounts in correct priority order', async () => {
    const vipUser = createUser({ type: 'vip', discountRate: 0.1 });
    const product = createProduct({ 
      id: 1, 
      price: 100,
      flashSalePrice: 80,
      bulkDiscountThreshold: 5
    });
    
    await wrapper.setProps({ user: vipUser, products: [product] });
    await wrapper.vm.addToCart(product.id, 6);
    
    // 验证折扣应用优先级：限时抢购 > 批量折扣 > VIP折扣
    expect(wrapper.vm.cartTotal).toBe(480); // 80 * 6，限时抢购价优先
  });
});

// 第三层：业务流程的端到端测试
describe('Cart Component - Business Flow Integration', () => {
  test('should handle complete purchase journey with real-time updates', async () => {
    // 模拟复杂的业务流程
    const journey = new PurchaseJourney();
    
    await journey.userLogin('vip_user');
    await journey.browseProducts('electronics');
    await journey.addMultipleItemsToCart([
      { productId: 1, quantity: 2 },
      { productId: 2, quantity: 1 }
    ]);
    
    // 模拟外部系统变化
    await journey.simulatePromotionActivation('flash_sale');
    await journey.simulateInventoryUpdate();
    
    // 验证购物车状态正确响应外部变化
    expect(wrapper.vm.cartItems).toHaveLength(2);
    expect(wrapper.vm.hasActivePromotions).toBe(true);
    expect(wrapper.vm.totalPrice).toBe(journey.expectedTotal);
  });
});
```

### 4.2 社交应用场景：评论组件的实时性挑战

社交应用的评论系统涉及实时更新、用户交互和内容管理，测试复杂度远超简单的CRUD操作。

#### 社交评论的业务特征分析

**实时性要求**：
```javascript
// 评论的实时业务需求
const realtimeRequirements = {
  // 新评论推送
  newCommentNotification: {
    trigger: "其他用户发表评论",
    behavior: "实时显示在列表中",
    constraints: ["用户在线", "未屏蔽发言者", "内容通过审核"]
  },
  
  // 点赞/回复的即时反馈
  interactionFeedback: {
    trigger: "用户点赞或回复",
    behavior: "立即更新界面计数和状态",
    rollback: "网络失败时恢复原状态"
  },
  
  // 在线状态显示
  presenceIndicator: {
    trigger: "用户活跃状态变化",
    behavior: "显示在线、离线、正在输入状态",
    privacy: "基于用户隐私设置"
  }
};
```

**内容管理复杂性**：
```javascript
// 评论内容的业务规则
const contentManagementRules = {
  // 内容过滤
  contentFiltering: {
    profanity: "敏感词过滤",
    spam: "垃圾内容检测",
    harassment: "骚扰内容识别",
    misinformation: "虚假信息标记"
  },
  
  // 用户权限
  userPermissions: {
    guest: { canRead: true, canComment: false },
    registered: { canRead: true, canComment: true, canEdit: "own" },
    moderator: { canRead: true, canComment: true, canEdit: "any", canDelete: true },
    admin: { fullAccess: true }
  },
  
  // 评论状态流转
  commentLifecycle: {
    draft: "草稿状态（自动保存）",
    pending: "等待审核",
    published: "已发布",
    flagged: "被举报",
    hidden: "被隐藏",
    deleted: "已删除"
  }
};
```

#### 社交场景的Vue测试实现

**测试实时更新机制**：
```javascript
describe('Comment Component - Real-time Updates', () => {
  test('should receive and display new comments via WebSocket', async () => {
    const mockWebSocket = new MockWebSocket();
    await wrapper.setProps({ wsConnection: mockWebSocket });
    
    const initialCommentCount = wrapper.vm.comments.length;
    
    // 模拟其他用户发表新评论
    const newComment = {
      id: 'comment_123',
      content: 'Great post!',
      author: 'other_user',
      timestamp: Date.now()
    };
    
    mockWebSocket.simulateMessage({
      type: 'new_comment',
      data: newComment
    });
    
    await nextTick();
    
    expect(wrapper.vm.comments).toHaveLength(initialCommentCount + 1);
    expect(wrapper.find('[data-testid="comment-123"]').exists()).toBe(true);
    expect(wrapper.find('[data-testid="new-comment-indicator"]').isVisible()).toBe(true);
  });
  
  test('should handle optimistic UI updates with rollback', async () => {
    const comment = { content: 'Test comment' };
    
    // 设置网络模拟失败
    mockApiService.postComment.mockRejectedValue(new Error('Network error'));
    
    // 用户提交评论
    await wrapper.vm.submitComment(comment);
    
    // 验证乐观更新：评论立即显示
    expect(wrapper.vm.comments).toContainEqual(
      expect.objectContaining({ content: comment.content, status: 'pending' })
    );
    
    // 等待网络响应
    await flushPromises();
    
    // 验证失败回滚：评论被移除，显示错误
    expect(wrapper.vm.comments).not.toContainEqual(
      expect.objectContaining({ content: comment.content })
    );
    expect(wrapper.vm.error).toContain('Failed to submit comment');
  });
});
```

**测试内容管理流程**：
```javascript
describe('Comment Component - Content Management', () => {
  test('should filter inappropriate content before submission', async () => {
    const inappropriateComment = {
      content: 'This contains bad words and spam content'
    };
    
    // 模拟内容过滤服务
    mockContentFilter.checkContent.mockResolvedValue({
      allowed: false,
      reasons: ['profanity', 'spam'],
      suggestion: 'Please revise your comment'
    });
    
    await wrapper.vm.submitComment(inappropriateComment);
    
    expect(wrapper.vm.error).toContain('Please revise your comment');
    expect(wrapper.vm.comments).not.toContainEqual(
      expect.objectContaining({ content: inappropriateComment.content })
    );
    expect(mockContentFilter.checkContent).toHaveBeenCalledWith(inappropriateComment.content);
  });
  
  test('should handle different user permission levels', async () => {
    const testCases = [
      {
        userType: 'guest',
        expectedActions: { canComment: false, canEdit: false, canDelete: false }
      },
      {
        userType: 'registered',
        expectedActions: { canComment: true, canEdit: 'own', canDelete: false }
      },
      {
        userType: 'moderator',
        expectedActions: { canComment: true, canEdit: 'any', canDelete: true }
      }
    ];
    
    testCases.forEach(async ({ userType, expectedActions }) => {
      await wrapper.setProps({ user: createUser(userType) });
      
      expect(wrapper.find('[data-testid="comment-input"]').exists())
        .toBe(expectedActions.canComment);
      
      if (expectedActions.canEdit === 'any') {
        expect(wrapper.findAll('[data-testid="edit-button"]')).toHaveLength(
          wrapper.vm.comments.length
        );
      }
      
      expect(wrapper.find('[data-testid="delete-button"]').exists())
        .toBe(expectedActions.canDelete);
    });
  });
});
```

### 4.3 工具类应用场景：表单构建器的配置复杂性

企业级表单构建器需要处理复杂的配置、验证和动态交互，是Vue组件业务逻辑测试的高级场景。

#### 表单构建器的业务挑战

**配置管理的复杂性**：
```javascript
// 表单配置的数据结构
const formConfigStructure = {
  // 表单元数据
  metadata: {
    id: "form_001",
    name: "用户注册表单",
    version: "1.2.0",
    category: "user_management",
    permissions: ["admin", "form_designer"]
  },
  
  // 字段配置
  fields: [
    {
      id: "username",
      type: "text",
      label: "用户名",
      validation: {
        required: true,
        minLength: 3,
        maxLength: 20,
        pattern: "^[a-zA-Z0-9_]+$",
        customValidator: "checkUsernameAvailability"
      },
      conditional: {
        showWhen: "userType === 'registered'",
        requiredWhen: "accountType !== 'guest'"
      },
      ui: {
        placeholder: "请输入用户名",
        helpText: "用户名只能包含字母、数字和下划线",
        icon: "user",
        autoComplete: "username"
      }
    }
  ],
  
  // 布局配置
  layout: {
    type: "grid",
    columns: 2,
    responsive: {
      xs: 1,
      sm: 2,
      md: 3,
      lg: 4
    },
    sections: [
      {
        title: "基本信息",
        collapsible: true,
        fields: ["username", "email", "phone"]
      }
    ]
  },
  
  // 业务逻辑
  businessLogic: {
    workflows: ["user_approval", "email_verification"],
    integrations: ["crm_system", "email_service"],
    hooks: {
      beforeSubmit: "validateBusinessRules",
      afterSubmit: "triggerWorkflow",
      onFieldChange: "recalculateDependentFields"
    }
  }
};
```

#### 工具类场景的测试策略

**配置驱动的动态测试**：
```javascript
describe('Form Builder - Configuration Management', () => {
  test('should render form based on dynamic configuration', async () => {
    const formConfig = createFormConfig({
      fields: [
        { id: 'name', type: 'text', required: true },
        { id: 'email', type: 'email', required: true },
        { id: 'age', type: 'number', min: 18, max: 65 }
      ]
    });
    
    await wrapper.setProps({ config: formConfig });
    
    // 验证字段渲染
    expect(wrapper.findAll('.form-field')).toHaveLength(3);
    expect(wrapper.find('[data-field="name"]').exists()).toBe(true);
    expect(wrapper.find('[data-field="email"]').exists()).toBe(true);
    expect(wrapper.find('[data-field="age"]').exists()).toBe(true);
    
    // 验证字段属性
    const emailField = wrapper.find('[data-field="email"] input');
    expect(emailField.attributes('type')).toBe('email');
    expect(emailField.attributes('required')).toBe('');
    
    const ageField = wrapper.find('[data-field="age"] input');
    expect(ageField.attributes('min')).toBe('18');
    expect(ageField.attributes('max')).toBe('65');
  });
  
  test('should handle conditional field visibility', async () => {
    const formConfig = createFormConfig({
      fields: [
        { 
          id: 'userType', 
          type: 'select', 
          options: ['guest', 'registered', 'premium'] 
        },
        { 
          id: 'membershipDetails', 
          type: 'group',
          showWhen: 'userType === "premium"',
          fields: [
            { id: 'membershipLevel', type: 'select' },
            { id: 'paymentMethod', type: 'select' }
          ]
        }
      ]
    });
    
    await wrapper.setProps({ config: formConfig });
    
    // 初始状态：高级字段隐藏
    expect(wrapper.find('[data-field="membershipDetails"]').exists()).toBe(false);
    
    // 选择premium类型
    await wrapper.find('[data-field="userType"] select').setValue('premium');
    
    // 验证条件字段显示
    expect(wrapper.find('[data-field="membershipDetails"]').exists()).toBe(true);
    expect(wrapper.find('[data-field="membershipLevel"]').exists()).toBe(true);
    expect(wrapper.find('[data-field="paymentMethod"]').exists()).toBe(true);
  });
});
```

**复杂验证逻辑测试**：
```javascript
describe('Form Builder - Complex Validation', () => {
  test('should handle cross-field validation dependencies', async () => {
    const formConfig = createFormConfig({
      fields: [
        { id: 'startDate', type: 'date' },
        { id: 'endDate', type: 'date' },
        { id: 'duration', type: 'number', calculated: true }
      ],
      validation: {
        crossFieldRules: [
          {
            rule: 'endDate > startDate',
            message: '结束日期必须晚于开始日期'
          },
          {
            rule: 'duration <= 365',
            message: '时间跨度不能超过一年'
          }
        ]
      }
    });
    
    await wrapper.setProps({ config: formConfig });
    
    // 设置无效的日期范围
    await wrapper.find('[data-field="startDate"] input').setValue('2024-12-01');
    await wrapper.find('[data-field="endDate"] input').setValue('2024-11-01');
    
    await wrapper.vm.validateForm();
    
    expect(wrapper.vm.validationErrors).toContain('结束日期必须晚于开始日期');
    expect(wrapper.find('.field-error').text()).toContain('结束日期必须晚于开始日期');
  });
  
  test('should perform asynchronous validation', async () => {
    const formConfig = createFormConfig({
      fields: [
        { 
          id: 'companyCode', 
          type: 'text',
          asyncValidation: {
            validator: 'checkCompanyCodeExists',
            debounce: 300
          }
        }
      ]
    });
    
    // 模拟异步验证服务
    mockValidationService.checkCompanyCodeExists
      .mockResolvedValue({ exists: true, name: 'ABC Corp' });
    
    await wrapper.setProps({ config: formConfig });
    
    const input = wrapper.find('[data-field="companyCode"] input');
    await input.setValue('ABC123');
    
    // 验证显示加载状态
    expect(wrapper.find('.field-validating').exists()).toBe(true);
    
    // 等待异步验证完成
    await flushPromises();
    
    expect(wrapper.find('.field-validating').exists()).toBe(false);
    expect(wrapper.find('.field-valid').exists()).toBe(true);
    expect(wrapper.find('.company-name-hint').text()).toContain('ABC Corp');
  });
});
```

---

**小结**：真实业务场景的复杂性远超简单的技术实现。电商的动态定价、社交的实时交互、工具类的配置管理，每个场景都有其特定的业务挑战。Vue测试需要深入理解这些业务复杂性，设计相应的测试策略，确保AI生成的测试代码能够覆盖真实的业务风险和用户场景。

## 5. 三个递进式完整实例（8000字）

为了将前面的理论指导转化为可操作的实践技能，本节提供三个从简单到复杂的完整实例。每个实例都展示完整的AI+TDD流程：测试计划设计→AI对话→代码生成→Review修正→最终成果。

### 5.1 简单实例：智能计数器组件（2000字）

#### 业务需求和组件设计

**业务背景**：构建一个智能计数器组件，用于商品详情页的数量选择器。需要支持：
- 基础的增减操作
- 最小值（1）和最大值（库存量）限制  
- 步进值设置（如1、5、10）
- 禁用状态处理
- 变更事件通知

**组件接口设计**：
```typescript
interface CounterProps {
  modelValue: number
  min?: number
  max?: number
  step?: number
  disabled?: boolean
}

interface CounterEmits {
  'update:modelValue': [value: number]
  'change': [value: number, oldValue: number]
  'limit-reached': [type: 'min' | 'max', value: number]
}
```

#### 测试计划MD设计

```markdown
# SmartCounter 组件测试计划

## 1. 业务背景
### 功能描述
智能计数器组件，主要用于商品数量选择，支持步进调节和边界控制
### 用户价值  
- 提供直观的数量选择交互
- 防止用户选择无效数量（超出库存、小于最小购买量）
- 支持批量操作（大步进值）
### 技术边界
- 最小值默认为1，可配置
- 最大值由库存或业务规则确定
- 支持小数步进值（如0.5）
- 输入框直接编辑需要验证

## 2. 技术上下文
### Vue技术栈
- Vue 3 Composition API
- v-model双向绑定
- 响应式数据更新
- 事件发射机制

### 依赖关系
- 无外部API依赖
- 纯客户端逻辑
- 与父组件的数据同步

## 3. 核心测试场景
### Happy Path
- 正常的增减操作
- 步进值的正确应用
- v-model双向绑定

### Sad Path  
- 超出最大值的处理
- 低于最小值的处理
- 无效输入的处理

### Boundary Conditions
- 边界值的精确测试
- 浮点数精度问题
- 禁用状态下的行为

## 4. AI指导重点
### 容易出错的地方
- 浮点数运算精度
- 边界值的包含性判断
- 事件发射的时机和参数
- 响应式更新的时序

### 重点关注
- 数值计算的准确性
- 事件触发的完整性
- 边界条件的严格性
- 用户交互的流畅性
```

#### 详细的AI对话展示

**第一轮：基础实现**

**人类**：
```
请根据这个测试计划，为SmartCounter组件生成Vue Test Utils的测试代码。重点关注：
1. 数值计算的准确性
2. 边界条件的严格性  
3. 事件发射的完整性
4. v-model双向绑定

请先生成核心的Happy Path测试。
```

**AI回应**：
```javascript
import { mount } from '@vue/test-utils'
import SmartCounter from '@/components/SmartCounter.vue'

describe('SmartCounter - Happy Path', () => {
  test('should increment value correctly', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 5,
        min: 1,
        max: 10,
        step: 1
      }
    })
    
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    expect(wrapper.emitted('update:modelValue')).toBeTruthy()
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([6])
  })
  
  test('should decrement value correctly', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 5,
        min: 1,
        max: 10,
        step: 1
      }
    })
    
    await wrapper.find('[data-testid="decrement-btn"]').trigger('click')
    
    expect(wrapper.emitted('update:modelValue')).toBeTruthy()
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([4])
  })
})
```

**人类Review和指导**：
```
这个测试有几个问题：
1. 缺少change事件的测试
2. 没有验证步进值的应用
3. 没有测试连续操作
4. 断言不够完整

请改进。
```

**AI改进后的代码**：
```javascript
describe('SmartCounter - Happy Path', () => {
  let wrapper
  
  beforeEach(() => {
    wrapper = mount(SmartCounter, {
      props: {
        modelValue: 5,
        min: 1,
        max: 10,
        step: 2
      }
    })
  })
  
  test('should increment with correct step value', async () => {
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    // 验证v-model更新
    expect(wrapper.emitted('update:modelValue')).toBeTruthy()
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([7]) // 5 + 2
    
    // 验证change事件
    expect(wrapper.emitted('change')).toBeTruthy()
    expect(wrapper.emitted('change')[0]).toEqual([7, 5])
  })
  
  test('should handle multiple increments correctly', async () => {
    // 第一次点击：5 → 7
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    // 更新props模拟父组件响应
    await wrapper.setProps({ modelValue: 7 })
    
    // 第二次点击：7 → 9  
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    expect(wrapper.emitted('update:modelValue')[1]).toEqual([9])
    expect(wrapper.emitted('change')[1]).toEqual([9, 7])
  })
  
  test('should support decimal step values', async () => {
    await wrapper.setProps({ modelValue: 1.5, step: 0.5 })
    
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([2.0])
  })
})
```

**第二轮：边界条件测试**

**人类**：
```
很好！现在请生成边界条件的测试，重点关注：
1. 达到最大值时的行为
2. 达到最小值时的行为  
3. limit-reached事件的发射
4. 按钮禁用状态
```

**AI生成的边界测试**：
```javascript
describe('SmartCounter - Boundary Conditions', () => {
  test('should not exceed maximum value', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 9,
        min: 1,
        max: 10,
        step: 2
      }
    })
    
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    // 应该停在最大值
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([10])
    expect(wrapper.emitted('limit-reached')).toBeTruthy()
    expect(wrapper.emitted('limit-reached')[0]).toEqual(['max', 10])
  })
  
  test('should disable increment button when at maximum', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 10,
        min: 1,
        max: 10,
        step: 1
      }
    })
    
    const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
    expect(incrementBtn.attributes('disabled')).toBeDefined()
  })
  
  test('should not go below minimum value', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 2,
        min: 1,
        max: 10,
        step: 2
      }
    })
    
    await wrapper.find('[data-testid="decrement-btn"]').trigger('click')
    
    // 应该停在最小值
    expect(wrapper.emitted('update:modelValue')[0]).toEqual([1])
    expect(wrapper.emitted('limit-reached')).toBeTruthy()
    expect(wrapper.emitted('limit-reached')[0]).toEqual(['min', 1])
  })
  
  test('should handle floating point precision correctly', async () => {
    const wrapper = mount(SmartCounter, {
      props: {
        modelValue: 0.1,
        min: 0.1,
        max: 1.0,
        step: 0.1
      }
    })
    
    // 连续点击测试浮点精度
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    await wrapper.setProps({ modelValue: 0.2 })
    
    await wrapper.find('[data-testid="increment-btn"]').trigger('click')
    
    // 应该是0.3而不是0.30000000000000004
    expect(wrapper.emitted('update:modelValue')[1]).toEqual([0.3])
  })
})
```

**第三轮：完整的组件实现**

**人类**：
```
测试看起来很好！现在请根据这些测试生成对应的Vue组件实现代码。
```

**AI生成的组件代码**：
```vue
<template>
  <div class="smart-counter">
    <button
      data-testid="decrement-btn"
      :disabled="disabled || isAtMin"
      @click="decrement"
      class="counter-btn"
    >
      -
    </button>
    
    <input
      v-model.number="displayValue"
      @blur="handleInputBlur"
      @keyup.enter="handleInputBlur"
      :disabled="disabled"
      class="counter-input"
      type="number"
      :min="min"
      :max="max"
      :step="step"
    />
    
    <button
      data-testid="increment-btn"
      :disabled="disabled || isAtMax"
      @click="increment"
      class="counter-btn"
    >
      +
    </button>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'

interface Props {
  modelValue: number
  min?: number
  max?: number
  step?: number
  disabled?: boolean
}

interface Emits {
  (e: 'update:modelValue', value: number): void
  (e: 'change', value: number, oldValue: number): void
  (e: 'limit-reached', type: 'min' | 'max', value: number): void
}

const props = withDefaults(defineProps<Props>(), {
  min: 1,
  max: 100,
  step: 1,
  disabled: false
})

const emit = defineEmits<Emits>()

// 内部显示值，处理输入框编辑
const displayValue = ref(props.modelValue)

// 监听外部值变化
watch(() => props.modelValue, (newVal) => {
  displayValue.value = newVal
})

// 计算属性
const isAtMin = computed(() => props.modelValue <= props.min)
const isAtMax = computed(() => props.modelValue >= props.max)

// 精确浮点数运算
function preciseAdd(a: number, b: number): number {
  const factor = Math.pow(10, getDecimalPlaces(Math.max(a, b)))
  return Math.round((a * factor + b * factor)) / factor
}

function getDecimalPlaces(num: number): number {
  if (Math.floor(num) === num) return 0
  return num.toString().split('.')[1]?.length || 0
}

// 值更新函数
function updateValue(newValue: number) {
  const oldValue = props.modelValue
  let clampedValue = Math.max(props.min, Math.min(props.max, newValue))
  
  if (clampedValue !== newValue) {
    // 触发限制事件
    const limitType = newValue < props.min ? 'min' : 'max'
    emit('limit-reached', limitType, clampedValue)
  }
  
  if (clampedValue !== oldValue) {
    emit('update:modelValue', clampedValue)
    emit('change', clampedValue, oldValue)
  }
}

// 增减操作
function increment() {
  if (!props.disabled && !isAtMax.value) {
    updateValue(preciseAdd(props.modelValue, props.step))
  }
}

function decrement() {
  if (!props.disabled && !isAtMin.value) {
    updateValue(preciseAdd(props.modelValue, -props.step))
  }
}

// 处理直接输入
function handleInputBlur() {
  updateValue(displayValue.value)
}
</script>

<style scoped>
.smart-counter {
  display: flex;
  align-items: center;
  gap: 4px;
}

.counter-btn {
  width: 32px;
  height: 32px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  cursor: pointer;
}

.counter-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.counter-input {
  width: 60px;
  height: 30px;
  text-align: center;
  border: 1px solid #ddd;
}
</style>
```

#### 质量评审和改进过程

**Review检查清单应用**：

**Intent（意图）**：✅ 测试名称清楚表达了业务意图，如"should not exceed maximum value"

**Coverage（覆盖）**：✅ 覆盖了Happy Path、边界条件、浮点精度等关键场景

**Assertion（断言）**：✅ 断言具体且可失败，验证了具体的数值和事件参数

**Isolation（隔离）**：✅ 纯组件测试，无外部依赖

**Readability（可读性）**：✅ 采用AAA结构，测试意图清晰

**发现的改进点**：
1. 添加了浮点数精度处理
2. 增强了事件参数的验证
3. 补充了禁用状态的测试
4. 改进了连续操作的测试场景

#### 完整测试代码和实现代码

**最终的完整测试套件**：
```javascript
import { mount } from '@vue/test-utils'
import SmartCounter from '@/components/SmartCounter.vue'

describe('SmartCounter Component', () => {
  describe('Happy Path', () => {
    test('should work with v-model binding', async () => {
      const wrapper = mount(SmartCounter, {
        props: {
          modelValue: 5,
          'onUpdate:modelValue': (value) => wrapper.setProps({ modelValue: value })
        }
      })
      
      await wrapper.find('[data-testid="increment-btn"]').trigger('click')
      
      expect(wrapper.props('modelValue')).toBe(6)
      expect(wrapper.emitted('change')[0]).toEqual([6, 5])
    })
    
    test('should support custom step values', async () => {
      const wrapper = mount(SmartCounter, {
        props: { modelValue: 10, step: 5 }
      })
      
      await wrapper.find('[data-testid="increment-btn"]').trigger('click')
      
      expect(wrapper.emitted('update:modelValue')[0]).toEqual([15])
    })
  })
  
  describe('Boundary Conditions', () => {
    test('should handle min/max limits correctly', async () => {
      const wrapper = mount(SmartCounter, {
        props: { modelValue: 9, min: 1, max: 10 }
      })
      
      // 测试最大值限制
      await wrapper.find('[data-testid="increment-btn"]').trigger('click')
      expect(wrapper.emitted('limit-reached')[0]).toEqual(['max', 10])
      
      // 测试最小值限制  
      await wrapper.setProps({ modelValue: 2 })
      await wrapper.find('[data-testid="decrement-btn"]').trigger('click')
      await wrapper.find('[data-testid="decrement-btn"]').trigger('click')
      expect(wrapper.emitted('limit-reached')[1]).toEqual(['min', 1])
    })
  })
  
  describe('Edge Cases', () => {
    test('should handle direct input editing', async () => {
      const wrapper = mount(SmartCounter, {
        props: { modelValue: 5, min: 1, max: 20 }
      })
      
      const input = wrapper.find('input')
      await input.setValue(15)
      await input.trigger('blur')
      
      expect(wrapper.emitted('update:modelValue')[0]).toEqual([15])
    })
    
    test('should clamp invalid direct input', async () => {
      const wrapper = mount(SmartCounter, {
        props: { modelValue: 5, min: 1, max: 10 }
      })
      
      const input = wrapper.find('input')
      await input.setValue(25) // 超过最大值
      await input.trigger('blur')
      
      expect(wrapper.emitted('update:modelValue')[0]).toEqual([10])
      expect(wrapper.emitted('limit-reached')[0]).toEqual(['max', 10])
    })
  })
})
```

这个简单实例展示了AI+TDD的完整流程：从业务需求分析到测试计划设计，再到与AI的迭代对话，最终产出高质量的测试代码和组件实现。关键在于人类的设计思维和AI的实现能力的有效结合。

### 5.2 中等实例：购物车组件（3000字）

#### 业务需求和复杂性分析

**业务背景**：构建一个功能完整的购物车组件，用于电商网站的购物车页面。相比简单的计数器，购物车具有更复杂的业务逻辑和状态管理需求：

**核心功能要求**：
- 商品列表展示（名称、价格、数量、小计）
- 商品数量修改（增减、直接输入）
- 商品删除和批量删除
- 价格计算（商品小计、总价、优惠券折扣）
- 库存检查和数量限制
- 优惠券应用和验证
- 结算前验证

**技术复杂性**：
- 状态管理：多个商品的复杂状态同步
- 异步操作：库存检查、优惠券验证API调用
- 计算属性：动态价格计算、优惠应用
- 事件通信：与父组件、服务层的多层通信
- 错误处理：网络错误、业务规则验证失败

**组件架构设计**：
```typescript
interface CartItem {
  id: string
  productId: string
  name: string
  price: number
  quantity: number
  maxQuantity: number // 库存限制
  imageUrl: string
  attributes?: Record<string, any> // 商品属性（颜色、尺寸等）
}

interface Coupon {
  code: string
  type: 'percentage' | 'fixed'
  value: number
  minAmount?: number
  maxDiscount?: number
}

interface CartState {
  items: CartItem[]
  selectedItems: string[] // 选中的商品ID
  coupon: Coupon | null
  isLoading: boolean
  errors: Record<string, string>
}

interface CartProps {
  initialItems?: CartItem[]
  allowBulkOperations?: boolean
  showCouponInput?: boolean
}

interface CartEmits {
  'items-updated': [items: CartItem[]]
  'checkout-requested': [selectedItems: CartItem[], totalAmount: number]
  'item-removed': [item: CartItem]
  'coupon-applied': [coupon: Coupon, discount: number]
  'error': [error: string, context: string]
}
```

#### 分层测试计划设计

```markdown
# ShoppingCart 组件测试计划

## 1. 业务背景
### 功能描述
完整的购物车管理组件，支持商品管理、价格计算、优惠券应用和结算
### 用户价值
- 直观的购物车商品管理体验
- 实时的价格计算和优惠展示
- 安全的库存控制和数量验证
- 流畅的结算前检查流程
### 技术边界
- 依赖外部库存API进行数量验证
- 需要优惠券服务进行折扣计算
- 支持复杂的商品属性和规格
- 处理网络异常和业务规则错误

## 2. 技术上下文
### Vue技术栈
- Pinia状态管理（购物车全局状态）
- 异步组件和Suspense（加载状态）
- 计算属性和侦听器（价格计算）
- 组件通信（父子组件事件）

### 外部依赖
- 库存服务API：检查商品库存和数量限制
- 优惠券服务API：验证和计算优惠券折扣
- 商品服务API：获取商品最新价格和信息

## 3. 测试场景分层

### Layer 1: 组件基础功能
- 商品列表渲染和数据绑定
- 数量修改操作（增减、输入）
- 商品删除和选择操作
- 基础价格计算（不含异步）

### Layer 2: 状态管理集成
- Pinia store的正确集成
- 跨组件状态同步
- 本地存储持久化
- 状态变更的事件通知

### Layer 3: 异步业务逻辑
- 库存检查的异步验证
- 优惠券应用的远程验证
- 网络错误的处理和重试
- 加载状态的用户体验

### Layer 4: 复杂业务场景
- 多商品批量操作
- 复杂优惠规则的计算
- 结算前的完整性检查
- 错误恢复和数据一致性

## 4. AI协作重点

### 容易出错的领域
- 异步状态管理和竞态条件
- 浮点数价格计算的精度
- Mock外部API的复杂场景
- 复杂组件层次的事件传递

### 关键测试策略
- 逐层测试：从纯组件到完整集成
- Mock策略：精确控制外部依赖
- 状态验证：确保状态一致性
- 异步处理：正确处理Promise和错误
```

#### AI对话过程：分层实现

**第一轮：基础组件功能**

**人类**：
```
请为购物车组件生成基础功能的测试代码。首先关注：
1. 商品列表的正确渲染
2. 数量修改的基础逻辑
3. 商品删除操作
4. 价格计算（不涉及异步）

注意：暂时不要涉及Pinia状态管理和异步API，先测试纯组件逻辑。
```

**AI初始回应**：
```javascript
import { mount } from '@vue/test-utils'
import ShoppingCart from '@/components/ShoppingCart.vue'

describe('ShoppingCart - Basic Functionality', () => {
  const mockCartItems = [
    {
      id: '1',
      productId: 'prod-1',
      name: 'iPhone 15',
      price: 999.99,
      quantity: 1,
      maxQuantity: 5,
      imageUrl: '/images/iphone15.jpg'
    },
    {
      id: '2',
      productId: 'prod-2',
      name: 'MacBook Pro',
      price: 2399.00,
      quantity: 2,
      maxQuantity: 3,
      imageUrl: '/images/macbook.jpg'
    }
  ]

  test('should render cart items correctly', () => {
    const wrapper = mount(ShoppingCart, {
      props: {
        initialItems: mockCartItems
      }
    })

    expect(wrapper.findAll('[data-testid="cart-item"]')).toHaveLength(2)
    expect(wrapper.text()).toContain('iPhone 15')
    expect(wrapper.text()).toContain('MacBook Pro')
  })

  test('should calculate item subtotal correctly', () => {
    const wrapper = mount(ShoppingCart, {
      props: {
        initialItems: mockCartItems
      }
    })

    const firstItem = wrapper.find('[data-testid="cart-item-1"]')
    expect(firstItem.find('.subtotal').text()).toContain('$999.99')
    
    const secondItem = wrapper.find('[data-testid="cart-item-2"]')
    expect(secondItem.find('.subtotal').text()).toContain('$4,798.00')
  })
})
```

**人类Review和改进指导**：
```
测试基础结构不错，但有几个问题：
1. 缺少数量修改操作的测试
2. 价格计算的断言不够精确
3. 没有测试商品删除功能
4. 缺少边界条件的考虑

请改进，并添加更完整的交互测试。
```

**AI改进后的测试代码**：
```javascript
describe('ShoppingCart - Basic Functionality', () => {
  let wrapper
  const mockCartItems = [
    {
      id: '1',
      productId: 'prod-1',
      name: 'iPhone 15',
      price: 999.99,
      quantity: 1,
      maxQuantity: 5,
      imageUrl: '/images/iphone15.jpg'
    },
    {
      id: '2',
      productId: 'prod-2',
      name: 'MacBook Pro',
      price: 2399.00,
      quantity: 2,
      maxQuantity: 3,
      imageUrl: '/images/macbook.jpg'
    }
  ]

  beforeEach(() => {
    wrapper = mount(ShoppingCart, {
      props: {
        initialItems: [...mockCartItems]
      }
    })
  })

  describe('Item Rendering', () => {
    test('should render all cart items with correct information', () => {
      expect(wrapper.findAll('[data-testid="cart-item"]')).toHaveLength(2)
      
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      expect(firstItem.find('.item-name').text()).toBe('iPhone 15')
      expect(firstItem.find('.item-price').text()).toBe('$999.99')
      expect(firstItem.find('.item-quantity input').element.value).toBe('1')
      
      const secondItem = wrapper.find('[data-testid="cart-item-2"]')
      expect(secondItem.find('.item-name').text()).toBe('MacBook Pro')
      expect(secondItem.find('.item-price').text()).toBe('$2,399.00')
      expect(secondItem.find('.item-quantity input').element.value).toBe('2')
    })

    test('should calculate subtotals correctly', () => {
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      expect(firstItem.find('.subtotal').text()).toBe('$999.99') // 999.99 * 1
      
      const secondItem = wrapper.find('[data-testid="cart-item-2"]')
      expect(secondItem.find('.subtotal').text()).toBe('$4,798.00') // 2399.00 * 2
    })

    test('should calculate total amount correctly', () => {
      const total = wrapper.find('[data-testid="cart-total"]')
      expect(total.text()).toBe('$5,797.99') // 999.99 + 4798.00
    })
  })

  describe('Quantity Modification', () => {
    test('should increase quantity when increment button clicked', async () => {
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      const incrementBtn = firstItem.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // 验证数量更新
      expect(firstItem.find('.item-quantity input').element.value).toBe('2')
      
      // 验证小计更新
      expect(firstItem.find('.subtotal').text()).toBe('$1,999.98')
      
      // 验证总价更新
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$6,797.98')
      
      // 验证事件发射
      expect(wrapper.emitted('items-updated')).toBeTruthy()
    })

    test('should decrease quantity when decrement button clicked', async () => {
      const secondItem = wrapper.find('[data-testid="cart-item-2"]')
      const decrementBtn = secondItem.find('[data-testid="decrement-btn"]')
      
      await decrementBtn.trigger('click')
      
      expect(secondItem.find('.item-quantity input').element.value).toBe('1')
      expect(secondItem.find('.subtotal').text()).toBe('$2,399.00')
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$3,398.99')
    })

    test('should handle direct quantity input', async () => {
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      const quantityInput = firstItem.find('.item-quantity input')
      
      await quantityInput.setValue(3)
      await quantityInput.trigger('blur')
      
      expect(firstItem.find('.subtotal').text()).toBe('$2,999.97')
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$7,797.97')
    })

    test('should not exceed maximum quantity', async () => {
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      const quantityInput = firstItem.find('.item-quantity input')
      
      // 尝试设置超过maxQuantity (5) 的值
      await quantityInput.setValue(10)
      await quantityInput.trigger('blur')
      
      // 应该被限制在最大值
      expect(quantityInput.element.value).toBe('5')
      expect(firstItem.find('.subtotal').text()).toBe('$4,999.95')
      
      // 应该显示错误提示
      expect(wrapper.find('.error-message').text()).toContain('超出库存限制')
    })

    test('should not go below minimum quantity (1)', async () => {
      const firstItem = wrapper.find('[data-testid="cart-item-1"]')
      const decrementBtn = firstItem.find('[data-testid="decrement-btn"]')
      
      // 当前数量为1，再次点击减少应该无效果
      await decrementBtn.trigger('click')
      
      expect(firstItem.find('.item-quantity input').element.value).toBe('1')
      
      // 减少按钮应该被禁用
      expect(decrementBtn.attributes('disabled')).toBeDefined()
    })
  })

  describe('Item Removal', () => {
    test('should remove item when delete button clicked', async () => {
      const deleteBtn = wrapper.find('[data-testid="cart-item-1"] [data-testid="delete-btn"]')
      
      await deleteBtn.trigger('click')
      
      // 商品应该从列表中移除
      expect(wrapper.findAll('[data-testid="cart-item"]')).toHaveLength(1)
      expect(wrapper.text()).not.toContain('iPhone 15')
      
      // 总价应该重新计算
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$4,798.00')
      
      // 应该发射移除事件
      expect(wrapper.emitted('item-removed')).toBeTruthy()
      expect(wrapper.emitted('item-removed')[0][0]).toMatchObject({
        id: '1',
        name: 'iPhone 15'
      })
    })

    test('should show empty cart message when all items removed', async () => {
      // 删除所有商品
      await wrapper.find('[data-testid="cart-item-1"] [data-testid="delete-btn"]').trigger('click')
      await wrapper.find('[data-testid="cart-item-2"] [data-testid="delete-btn"]').trigger('click')
      
      expect(wrapper.find('[data-testid="empty-cart"]').exists()).toBe(true)
      expect(wrapper.text()).toContain('购物车为空')
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$0.00')
    })
  })
})
```

**第二轮：状态管理集成**

**人类**：
```
基础功能测试很好！现在请生成Pinia状态管理集成的测试。重点关注：
1. 购物车状态的正确同步
2. 跨组件状态共享
3. 本地存储持久化
4. 状态变更通知

请使用合适的Mock策略来隔离Pinia store。
```

**AI生成的状态管理测试**：
```javascript
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import { useCartStore } from '@/stores/cart'
import ShoppingCart from '@/components/ShoppingCart.vue'

// Mock localStorage
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn(),
  clear: vi.fn()
}
global.localStorage = localStorageMock

describe('ShoppingCart - State Management Integration', () => {
  let pinia
  let cartStore
  let wrapper

  beforeEach(() => {
    // 创建新的Pinia实例
    pinia = createPinia()
    setActivePinia(pinia)
    cartStore = useCartStore()
    
    // 清理localStorage mock
    localStorageMock.getItem.mockClear()
    localStorageMock.setItem.mockClear()
    
    // 初始化store状态
    cartStore.items = [
      {
        id: '1',
        productId: 'prod-1',
        name: 'iPhone 15',
        price: 999.99,
        quantity: 1,
        maxQuantity: 5
      }
    ]

    wrapper = mount(ShoppingCart, {
      global: {
        plugins: [pinia]
      }
    })
  })

  describe('Store Integration', () => {
    test('should sync with Pinia store state', () => {
      expect(wrapper.findAll('[data-testid="cart-item"]')).toHaveLength(1)
      expect(wrapper.text()).toContain('iPhone 15')
      
      // 组件应该显示store中的数据
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$999.99')
    })

    test('should update store when quantity changes', async () => {
      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // store状态应该同步更新
      expect(cartStore.items[0].quantity).toBe(2)
      expect(cartStore.totalAmount).toBe(1999.98)
    })

    test('should update store when item is removed', async () => {
      const deleteBtn = wrapper.find('[data-testid="delete-btn"]')
      
      await deleteBtn.trigger('click')
      
      // store中的商品应该被移除
      expect(cartStore.items).toHaveLength(0)
      expect(cartStore.totalAmount).toBe(0)
    })

    test('should react to external store changes', async () => {
      // 外部代码修改store状态
      cartStore.addItem({
        id: '2',
        productId: 'prod-2',
        name: 'MacBook Pro',
        price: 2399.00,
        quantity: 1,
        maxQuantity: 3
      })
      
      await wrapper.vm.$nextTick()
      
      // 组件应该自动更新
      expect(wrapper.findAll('[data-testid="cart-item"]')).toHaveLength(2)
      expect(wrapper.text()).toContain('MacBook Pro')
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$3,398.99')
    })
  })

  describe('Persistence', () => {
    test('should persist cart data to localStorage on changes', async () => {
      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // 应该调用localStorage.setItem
      expect(localStorageMock.setItem).toHaveBeenCalledWith(
        'cart-data',
        JSON.stringify({
          items: expect.arrayContaining([
            expect.objectContaining({
              id: '1',
              quantity: 2
            })
          ]),
          timestamp: expect.any(Number)
        })
      )
    })

    test('should restore cart data from localStorage on mount', () => {
      const savedCartData = {
        items: [
          {
            id: 'saved-1',
            productId: 'prod-saved',
            name: 'Saved Item',
            price: 100.00,
            quantity: 3,
            maxQuantity: 10
          }
        ],
        timestamp: Date.now() - 1000 // 1秒前保存
      }
      
      localStorageMock.getItem.mockReturnValue(JSON.stringify(savedCartData))
      
      // 重新挂载组件
      const newWrapper = mount(ShoppingCart, {
        global: {
          plugins: [pinia]
        }
      })
      
      // 应该显示保存的数据
      expect(newWrapper.text()).toContain('Saved Item')
      expect(newWrapper.find('[data-testid="cart-total"]').text()).toBe('$300.00')
    })

    test('should ignore expired localStorage data', () => {
      const expiredCartData = {
        items: [{ id: 'expired', name: 'Expired Item' }],
        timestamp: Date.now() - (7 * 24 * 60 * 60 * 1000) // 7天前
      }
      
      localStorageMock.getItem.mockReturnValue(JSON.stringify(expiredCartData))
      
      const newWrapper = mount(ShoppingCart, {
        global: {
          plugins: [pinia]
        }
      })
      
      // 过期数据应该被忽略
      expect(newWrapper.find('[data-testid="empty-cart"]').exists()).toBe(true)
    })
  })

  describe('Cross-Component Communication', () => {
    test('should emit global events for other components', async () => {
      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // 应该发射全局事件
      expect(wrapper.emitted('items-updated')).toBeTruthy()
      
      const emittedData = wrapper.emitted('items-updated')[0][0]
      expect(emittedData).toEqual(
        expect.arrayContaining([
          expect.objectContaining({
            id: '1',
            quantity: 2
          })
        ])
      )
    })

    test('should handle concurrent modifications correctly', async () => {
      // 模拟并发修改：组件操作 + 外部store操作
      const incrementPromise = wrapper.find('[data-testid="increment-btn"]').trigger('click')
      
      // 同时外部修改store
      cartStore.updateItemQuantity('1', 5)
      
      await incrementPromise
      await wrapper.vm.$nextTick()
      
      // 最终状态应该一致（后面的操作生效）
      expect(cartStore.items[0].quantity).toBe(5) // 外部操作的值
      expect(wrapper.find('.item-quantity input').element.value).toBe('5')
    })
  })
})
```

**第三轮：异步业务逻辑测试**

**人类**：
```
状态管理测试很全面！现在请生成异步业务逻辑的测试，重点关注：
1. 库存检查的异步验证
2. 优惠券应用的API调用
3. 网络错误处理和重试机制
4. 加载状态的用户体验

请使用适当的Mock策略来模拟API响应。
```

**AI生成的异步业务逻辑测试**：
```javascript
import { mount, flushPromises } from '@vue/test-utils'
import { vi } from 'vitest'
import ShoppingCart from '@/components/ShoppingCart.vue'

// Mock API services
const mockInventoryService = {
  checkStock: vi.fn(),
  updateReservation: vi.fn()
}

const mockCouponService = {
  validateCoupon: vi.fn(),
  applyCoupon: vi.fn()
}

// Mock services
vi.mock('@/services/inventory', () => ({
  inventoryService: mockInventoryService
}))

vi.mock('@/services/coupon', () => ({
  couponService: mockCouponService
}))

describe('ShoppingCart - Async Business Logic', () => {
  let wrapper

  beforeEach(() => {
    // 重置所有mocks
    vi.clearAllMocks()
    
    wrapper = mount(ShoppingCart, {
      props: {
        initialItems: [
          {
            id: '1',
            productId: 'prod-1',
            name: 'iPhone 15',
            price: 999.99,
            quantity: 1,
            maxQuantity: 5
          }
        ],
        showCouponInput: true
      }
    })
  })

  describe('Inventory Validation', () => {
    test('should validate stock when quantity increases', async () => {
      // Mock成功的库存检查
      mockInventoryService.checkStock.mockResolvedValue({
        available: true,
        maxQuantity: 5,
        currentStock: 10
      })

      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // 应该调用库存检查API
      expect(mockInventoryService.checkStock).toHaveBeenCalledWith('prod-1', 2)
      
      // 等待异步操作完成
      await flushPromises()
      
      // 数量应该成功更新
      expect(wrapper.find('.item-quantity input').element.value).toBe('2')
      expect(wrapper.find('.error-message').exists()).toBe(false)
    })

    test('should handle insufficient stock error', async () => {
      // Mock库存不足
      mockInventoryService.checkStock.mockResolvedValue({
        available: false,
        maxQuantity: 1,
        currentStock: 1,
        error: 'INSUFFICIENT_STOCK'
      })

      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      await flushPromises()
      
      // 数量不应该更新
      expect(wrapper.find('.item-quantity input').element.value).toBe('1')
      
      // 应该显示错误信息
      expect(wrapper.find('.error-message').text()).toContain('库存不足')
      expect(wrapper.find('.error-message').text()).toContain('最多只能购买 1 件')
    })

    test('should show loading state during stock check', async () => {
      // Mock延迟的API响应
      let resolveStock
      mockInventoryService.checkStock.mockReturnValue(
        new Promise(resolve => {
          resolveStock = resolve
        })
      )

      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      
      // 应该显示加载状态
      expect(wrapper.find('.loading-spinner').exists()).toBe(true)
      expect(incrementBtn.attributes('disabled')).toBeDefined()
      
      // 完成API调用
      resolveStock({
        available: true,
        maxQuantity: 5,
        currentStock: 10
      })
      
      await flushPromises()
      
      // 加载状态应该消失
      expect(wrapper.find('.loading-spinner').exists()).toBe(false)
      expect(incrementBtn.attributes('disabled')).toBeUndefined()
    })

    test('should handle network errors with retry mechanism', async () => {
      // Mock网络错误
      mockInventoryService.checkStock
        .mockRejectedValueOnce(new Error('Network Error'))
        .mockResolvedValueOnce({
          available: true,
          maxQuantity: 5,
          currentStock: 10
        })

      const incrementBtn = wrapper.find('[data-testid="increment-btn"]')
      
      await incrementBtn.trigger('click')
      await flushPromises()
      
      // 应该显示重试选项
      expect(wrapper.find('.error-retry').exists()).toBe(true)
      expect(wrapper.text()).toContain('网络错误，请重试')
      
      // 点击重试
      await wrapper.find('[data-testid="retry-btn"]').trigger('click')
      await flushPromises()
      
      // 重试成功后应该更新数量
      expect(wrapper.find('.item-quantity input').element.value).toBe('2')
      expect(wrapper.find('.error-retry').exists()).toBe(false)
    })
  })

  describe('Coupon Application', () => {
    test('should apply valid coupon correctly', async () => {
      const validCoupon = {
        code: 'SAVE20',
        type: 'percentage',
        value: 20,
        minAmount: 500
      }

      mockCouponService.validateCoupon.mockResolvedValue({
        valid: true,
        coupon: validCoupon,
        discount: 199.998 // 20% of 999.99
      })

      const couponInput = wrapper.find('[data-testid="coupon-input"]')
      const applyBtn = wrapper.find('[data-testid="apply-coupon-btn"]')
      
      await couponInput.setValue('SAVE20')
      await applyBtn.trigger('click')
      await flushPromises()
      
      // 应该显示优惠信息
      expect(wrapper.find('.coupon-applied').exists()).toBe(true)
      expect(wrapper.text()).toContain('SAVE20')
      expect(wrapper.text()).toContain('优惠 $199.998')
      
      // 总价应该扣减优惠
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$799.99')
      
      // 应该发射优惠券应用事件
      expect(wrapper.emitted('coupon-applied')).toBeTruthy()
      expect(wrapper.emitted('coupon-applied')[0]).toEqual([validCoupon, 199.998])
    })

    test('should handle invalid coupon', async () => {
      mockCouponService.validateCoupon.mockResolvedValue({
        valid: false,
        error: 'COUPON_EXPIRED',
        message: '优惠券已过期'
      })

      const couponInput = wrapper.find('[data-testid="coupon-input"]')
      const applyBtn = wrapper.find('[data-testid="apply-coupon-btn"]')
      
      await couponInput.setValue('EXPIRED20')
      await applyBtn.trigger('click')
      await flushPromises()
      
      // 应该显示错误信息
      expect(wrapper.find('.coupon-error').text()).toContain('优惠券已过期')
      
      // 总价不应该变化
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$999.99')
      
      // 不应该发射优惠券应用事件
      expect(wrapper.emitted('coupon-applied')).toBeFalsy()
    })

    test('should handle minimum amount requirement', async () => {
      mockCouponService.validateCoupon.mockResolvedValue({
        valid: false,
        error: 'MINIMUM_AMOUNT_NOT_MET',
        message: '需要满 $1500 才能使用此优惠券',
        minAmount: 1500
      })

      const couponInput = wrapper.find('[data-testid="coupon-input"]')
      const applyBtn = wrapper.find('[data-testid="apply-coupon-btn"]')
      
      await couponInput.setValue('VIP50')
      await applyBtn.trigger('click')
      await flushPromises()
      
      // 应该显示最低金额提示
      expect(wrapper.find('.coupon-error').text()).toContain('需要满 $1500')
      
      // 应该显示还需要添加的金额
      expect(wrapper.text()).toContain('还需 $500.01')
    })

    test('should remove coupon when clicked', async () => {
      // 先应用优惠券
      mockCouponService.validateCoupon.mockResolvedValue({
        valid: true,
        coupon: { code: 'SAVE20', type: 'percentage', value: 20 },
        discount: 199.998
      })

      const couponInput = wrapper.find('[data-testid="coupon-input"]')
      const applyBtn = wrapper.find('[data-testid="apply-coupon-btn"]')
      
      await couponInput.setValue('SAVE20')
      await applyBtn.trigger('click')
      await flushPromises()
      
      // 点击移除优惠券
      const removeBtn = wrapper.find('[data-testid="remove-coupon-btn"]')
      await removeBtn.trigger('click')
      
      // 优惠应该被取消
      expect(wrapper.find('.coupon-applied').exists()).toBe(false)
      expect(wrapper.find('[data-testid="cart-total"]').text()).toBe('$999.99')
      
      // 输入框应该清空
      expect(couponInput.element.value).toBe('')
    })
  })

  describe('Checkout Validation', () => {
    test('should validate all items before checkout', async () => {
      // Mock所有验证成功
      mockInventoryService.checkStock.mockResolvedValue({
        available: true,
        maxQuantity: 5,
        currentStock: 10
      })

      const checkoutBtn = wrapper.find('[data-testid="checkout-btn"]')
      
      await checkoutBtn.trigger('click')
      await flushPromises()
      
      // 应该检查所有商品的库存
      expect(mockInventoryService.checkStock).toHaveBeenCalledWith('prod-1', 1)
      
      // 应该发射结算事件
      expect(wrapper.emitted('checkout-requested')).toBeTruthy()
      expect(wrapper.emitted('checkout-requested')[0][1]).toBe(999.99) // 总金额
    })

    test('should block checkout if validation fails', async () => {
      // Mock库存检查失败
      mockInventoryService.checkStock.mockResolvedValue({
        available: false,
        error: 'OUT_OF_STOCK'
      })

      const checkoutBtn = wrapper.find('[data-testid="checkout-btn"]')
      
      await checkoutBtn.trigger('click')
      await flushPromises()
      
      // 应该显示验证错误
      expect(wrapper.find('.checkout-error').text()).toContain('部分商品库存不足')
      
      // 不应该发射结算事件
      expect(wrapper.emitted('checkout-requested')).toBeFalsy()
      
      // 结算按钮应该保持禁用
      expect(checkoutBtn.attributes('disabled')).toBeDefined()
    })
  })
})
```

这个中等复杂度的实例展示了在状态管理、异步操作和复杂业务逻辑环境下的AI+TDD实践。关键点在于：

1. **分层测试策略**：从基础功能到状态管理再到异步业务逻辑的逐层构建
2. **精确的Mock策略**：针对不同类型的外部依赖采用合适的Mock方案
3. **异步测试处理**：正确处理Promise、加载状态和错误处理
4. **状态一致性验证**：确保组件、store和持久化层的状态同步

通过这种方式，AI能够理解复杂的业务场景并生成相应的测试代码，而人类则负责设计测试策略和验证业务正确性。

### 5.3 复杂实例：搜索列表页组件（3000字）

#### 业务需求和架构复杂性分析

**业务背景**：构建一个功能完整的商品搜索页面组件，这是电商网站的核心功能之一。相比前面的实例，搜索页面具有更高的技术复杂度和业务复杂性：

**核心功能要求**：
- 多维度搜索过滤（关键词、价格区间、品牌、分类等）
- 搜索结果分页和虚拟滚动
- 搜索历史记录和智能推荐
- 商品收藏和比较功能
- 搜索结果排序（价格、销量、评分等）
- 实时搜索建议和自动补全
- 搜索统计和埋点上报

**技术架构复杂性**：
- 路由集成：URL状态同步、前进后退支持
- 性能优化：防抖搜索、结果缓存、图片懒加载
- 状态管理：复杂的搜索状态、过滤器状态
- 异步协调：多个API调用的协调和依赖处理
- 用户体验：加载状态、空状态、错误恢复
- 响应式设计：移动端适配、无限滚动

**组件架构设计**：
```typescript
interface SearchFilters {
  keyword: string
  priceRange: [number, number]
  brands: string[]
  categories: string[]
  sortBy: 'price' | 'sales' | 'rating' | 'newest'
  sortOrder: 'asc' | 'desc'
}

interface SearchResult {
  products: Product[]
  total: number
  page: number
  pageSize: number
  facets: SearchFacets // 搜索面板数据
  suggestions: string[]
  didYouMean?: string
}

interface SearchFacets {
  brands: Array<{ name: string; count: number }>
  categories: Array<{ id: string; name: string; count: number }>
  priceRanges: Array<{ min: number; max: number; count: number }>
}

interface SearchPageState {
  filters: SearchFilters
  results: SearchResult | null
  isLoading: boolean
  isLoadingMore: boolean
  searchHistory: string[]
  favoriteProducts: Set<string>
  compareProducts: Set<string>
  errors: Record<string, string>
  cacheKey: string
}

interface SearchPageProps {
  initialKeyword?: string
  enableVirtualScroll?: boolean
  enableInfiniteScroll?: boolean
  maxHistoryItems?: number
}

interface SearchPageEmits {
  'search-performed': [keyword: string, filters: SearchFilters]
  'product-clicked': [product: Product, position: number]
  'filter-changed': [filterType: string, value: any]
  'product-favorited': [product: Product]
  'product-compared': [products: Product[]]
}
```

#### 分层测试计划设计

```markdown
# SearchPage 组件测试计划

## 1. 业务背景
### 功能描述
综合性商品搜索页面，支持多维度搜索、过滤、排序和结果展示
### 用户价值
- 快速准确地找到目标商品
- 丰富的过滤和排序选项
- 流畅的搜索和浏览体验
- 智能的搜索建议和历史记录
### 技术边界
- 依赖搜索API和推荐API
- 需要处理大量商品数据的性能问题
- 支持复杂的URL状态管理
- 需要埋点统计和用户行为分析

## 2. 技术上下文
### Vue技术栈
- Vue Router集成（URL状态同步）
- 虚拟滚动（大数据量性能优化）
- 响应式搜索（防抖和缓存）
- 复杂组件层次（搜索框、过滤器、结果列表）

### 外部依赖
- 搜索服务API：关键词搜索和过滤
- 推荐服务API：智能推荐和相关商品
- 统计服务API：搜索埋点和用户行为
- 收藏服务API：商品收藏状态管理

## 3. 测试场景分层

### Layer 1: 基础搜索功能
- 关键词搜索和结果展示
- 基础过滤器操作（价格、品牌、分类）
- 搜索结果排序和分页
- 搜索历史记录

### Layer 2: 高级交互功能
- 实时搜索建议和自动补全
- 商品收藏和比较功能
- 搜索结果的无限滚动
- 响应式布局和移动端适配

### Layer 3: 路由和状态集成
- URL状态同步和浏览器前进后退
- 搜索状态的持久化和恢复
- 跨页面状态共享
- 深度链接支持

### Layer 4: 性能和用户体验
- 防抖搜索和结果缓存
- 虚拟滚动和图片懒加载
- 加载状态和错误处理
- 埋点统计和性能监控

## 4. AI协作重点

### 复杂度挑战
- 多个异步操作的协调和依赖处理
- 复杂的组件层次和事件传递
- 性能优化相关的测试设计
- 路由状态和组件状态的同步

### 测试策略重点
- 分层Mock：精确隔离不同层次的依赖
- 异步协调：处理多个API调用的时序
- 状态验证：确保路由、组件、store状态一致
- 性能测试：验证防抖、缓存、虚拟滚动等优化
```

#### AI对话过程：复杂场景实现

**第一轮：基础搜索功能和路由集成**

**人类**：
```
请为搜索页面组件生成基础功能的测试代码。重点关注：
1. 关键词搜索和结果展示
2. URL状态同步（搜索参数反映在URL中）
3. 基础过滤器操作
4. 路由前进后退的正确处理

注意：这是一个复杂的页面级组件，需要Mock路由和多个API服务。
```

**AI初始回应**：
```javascript
import { mount } from '@vue/test-utils'
import { createRouter, createWebHistory } from 'vue-router'
import { vi } from 'vitest'
import SearchPage from '@/pages/SearchPage.vue'

// Mock API services
const mockSearchService = {
  search: vi.fn(),
  getSuggestions: vi.fn(),
  getFacets: vi.fn()
}

const mockStatsService = {
  trackSearch: vi.fn(),
  trackProductClick: vi.fn()
}

vi.mock('@/services/search', () => ({
  searchService: mockSearchService
}))

vi.mock('@/services/stats', () => ({
  statsService: mockStatsService
}))

describe('SearchPage - Basic Search & Routing', () => {
  let router
  let wrapper

  beforeEach(async () => {
    vi.clearAllMocks()
    
    // 创建测试路由
    router = createRouter({
      history: createWebHistory(),
      routes: [
        { path: '/search', component: SearchPage, name: 'search' }
      ]
    })
    
    // Mock成功的搜索响应
    mockSearchService.search.mockResolvedValue({
      products: [
        {
          id: 'prod-1',
          name: 'iPhone 15 Pro',
          price: 999.99,
          brand: 'Apple',
          category: 'smartphones',
          imageUrl: '/images/iphone15.jpg',
          rating: 4.5,
          salesCount: 1000
        },
        {
          id: 'prod-2', 
          name: 'Samsung Galaxy S24',
          price: 899.99,
          brand: 'Samsung',
          category: 'smartphones',
          imageUrl: '/images/galaxy.jpg',
          rating: 4.3,
          salesCount: 800
        }
      ],
      total: 120,
      page: 1,
      pageSize: 20,
      facets: {
        brands: [
          { name: 'Apple', count: 45 },
          { name: 'Samsung', count: 38 }
        ],
        categories: [
          { id: 'smartphones', name: '智能手机', count: 83 }
        ],
        priceRanges: [
          { min: 500, max: 1000, count: 67 },
          { min: 1000, max: 2000, count: 53 }
        ]
      },
      suggestions: ['iPhone 15', 'iPhone 14', 'iPhone Pro']
    })

    wrapper = mount(SearchPage, {
      global: {
        plugins: [router]
      }
    })

    await router.push('/search?q=iPhone')
    await wrapper.vm.$nextTick()
  })

  describe('Basic Search Functionality', () => {
    test('should perform search with keyword from URL', async () => {
      // 应该根据URL参数进行搜索
      expect(mockSearchService.search).toHaveBeenCalledWith({
        keyword: 'iPhone',
        page: 1,
        pageSize: 20,
        filters: expect.any(Object)
      })
      
      // 应该显示搜索结果
      expect(wrapper.findAll('[data-testid="product-item"]')).toHaveLength(2)
      expect(wrapper.text()).toContain('iPhone 15 Pro')
      expect(wrapper.text()).toContain('Samsung Galaxy S24')
      
      // 应该显示结果统计
      expect(wrapper.find('[data-testid="results-count"]').text()).toContain('共找到 120 个商品')
    })

    test('should update URL when search keyword changes', async () => {
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      await searchInput.setValue('MacBook')
      await searchInput.trigger('keyup.enter')
      
      // URL应该更新
      expect(router.currentRoute.value.query.q).toBe('MacBook')
      
      // 应该发起新的搜索
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          keyword: 'MacBook'
        })
      )
    })

    test('should handle empty search results', async () => {
      // Mock空结果
      mockSearchService.search.mockResolvedValue({
        products: [],
        total: 0,
        page: 1,
        pageSize: 20,
        facets: { brands: [], categories: [], priceRanges: [] },
        suggestions: ['iPhone', 'Samsung', 'Xiaomi']
      })

      const searchInput = wrapper.find('[data-testid="search-input"]')
      await searchInput.setValue('NonExistentProduct')
      await searchInput.trigger('keyup.enter')
      
      await wrapper.vm.$nextTick()
      await flushPromises()
      
      // 应该显示空状态
      expect(wrapper.find('[data-testid="empty-results"]').exists()).toBe(true)
      expect(wrapper.text()).toContain('没有找到相关商品')
      
      // 应该显示搜索建议
      expect(wrapper.find('[data-testid="search-suggestions"]').exists()).toBe(true)
      expect(wrapper.text()).toContain('试试这些搜索词')
    })
  })

  describe('URL State Synchronization', () => {
    test('should sync filter changes to URL', async () => {
      // 选择品牌过滤器
      const brandFilter = wrapper.find('[data-testid="brand-filter-Apple"]')
      await brandFilter.setChecked(true)
      
      await wrapper.vm.$nextTick()
      
      // URL应该包含过滤器参数
      expect(router.currentRoute.value.query.brands).toBe('Apple')
      
      // 应该发起带过滤器的搜索
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          filters: expect.objectContaining({
            brands: ['Apple']
          })
        })
      )
    })

    test('should restore state from URL parameters', async () => {
      // 导航到带参数的URL
      await router.push('/search?q=iPhone&brands=Apple,Samsung&sort=price&order=asc')
      await wrapper.vm.$nextTick()
      
      // 搜索框应该显示正确的关键词
      expect(wrapper.find('[data-testid="search-input"]').element.value).toBe('iPhone')
      
      // 过滤器应该被正确选中
      expect(wrapper.find('[data-testid="brand-filter-Apple"]').element.checked).toBe(true)
      expect(wrapper.find('[data-testid="brand-filter-Samsung"]').element.checked).toBe(true)
      
      // 排序选项应该被正确设置
      expect(wrapper.find('[data-testid="sort-select"]').element.value).toBe('price')
      expect(wrapper.find('[data-testid="order-select"]').element.value).toBe('asc')
    })

    test('should handle browser back/forward correctly', async () => {
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      // 执行第一次搜索
      await searchInput.setValue('iPhone')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 执行第二次搜索
      await searchInput.setValue('Samsung')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 模拟浏览器后退
      await router.back()
      await wrapper.vm.$nextTick()
      
      // 应该恢复到前一个搜索状态
      expect(wrapper.find('[data-testid="search-input"]').element.value).toBe('iPhone')
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          keyword: 'iPhone'
        })
      )
    })
  })

  describe('Filter Operations', () => {
    test('should apply price range filter', async () => {
      const minPriceInput = wrapper.find('[data-testid="min-price-input"]')
      const maxPriceInput = wrapper.find('[data-testid="max-price-input"]')
      const applyBtn = wrapper.find('[data-testid="apply-price-filter"]')
      
      await minPriceInput.setValue(500)
      await maxPriceInput.setValue(1000)
      await applyBtn.trigger('click')
      
      // 应该发起带价格过滤的搜索
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          filters: expect.objectContaining({
            priceRange: [500, 1000]
          })
        })
      )
      
      // URL应该包含价格参数
      expect(router.currentRoute.value.query.minPrice).toBe('500')
      expect(router.currentRoute.value.query.maxPrice).toBe('1000')
    })

    test('should clear all filters', async () => {
      // 先设置一些过滤器
      await wrapper.find('[data-testid="brand-filter-Apple"]').setChecked(true)
      await wrapper.find('[data-testid="min-price-input"]').setValue(500)
      
      // 点击清除所有过滤器
      const clearBtn = wrapper.find('[data-testid="clear-filters-btn"]')
      await clearBtn.trigger('click')
      
      // 过滤器应该被清除
      expect(wrapper.find('[data-testid="brand-filter-Apple"]').element.checked).toBe(false)
      expect(wrapper.find('[data-testid="min-price-input"]').element.value).toBe('')
      
      // URL参数应该被清除
      expect(router.currentRoute.value.query.brands).toBeUndefined()
      expect(router.currentRoute.value.query.minPrice).toBeUndefined()
    })

    test('should show filter count badges', async () => {
      // 选择多个过滤器
      await wrapper.find('[data-testid="brand-filter-Apple"]').setChecked(true)
      await wrapper.find('[data-testid="brand-filter-Samsung"]').setChecked(true)
      await wrapper.find('[data-testid="category-filter-smartphones"]').setChecked(true)
      
      // 应该显示过滤器数量徽章
      const filterBadge = wrapper.find('[data-testid="active-filters-count"]')
      expect(filterBadge.text()).toBe('3')
      
      // 应该显示具体的过滤器标签
      expect(wrapper.findAll('[data-testid="filter-tag"]')).toHaveLength(3)
      expect(wrapper.text()).toContain('Apple')
      expect(wrapper.text()).toContain('Samsung')
      expect(wrapper.text()).toContain('智能手机')
    })
  })
})
```

**人类Review和指导**：
```
基础功能测试很好，但还需要补充：
1. 搜索防抖功能的测试
2. 分页和无限滚动的测试
3. 商品交互功能（收藏、比较）
4. 性能相关功能的测试

请继续完善。
```

**第二轮：高级交互和性能功能**

**AI生成的高级功能测试**：
```javascript
describe('SearchPage - Advanced Features & Performance', () => {
  let wrapper
  let router

  beforeEach(async () => {
    // ... 基础设置代码 ...
    
    // Mock虚拟滚动和懒加载相关的API
    Object.defineProperty(window, 'IntersectionObserver', {
      writable: true,
      value: vi.fn().mockImplementation(() => ({
        observe: vi.fn(),
        unobserve: vi.fn(),
        disconnect: vi.fn()
      }))
    })
  })

  describe('Search Debouncing & Caching', () => {
    test('should debounce search input to prevent excessive API calls', async () => {
      vi.useFakeTimers()
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      // 快速输入多个字符
      await searchInput.setValue('i')
      await searchInput.setValue('ip')
      await searchInput.setValue('iph')
      await searchInput.setValue('ipho')
      await searchInput.setValue('iphone')
      
      // 在防抖时间内不应该发起搜索
      expect(mockSearchService.search).not.toHaveBeenCalled()
      
      // 等待防抖时间结束
      vi.advanceTimersByTime(500)
      await flushPromises()
      
      // 应该只发起一次搜索
      expect(mockSearchService.search).toHaveBeenCalledTimes(1)
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          keyword: 'iphone'
        })
      )
      
      vi.useRealTimers()
    })

    test('should use cached results for repeated searches', async () => {
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      // 第一次搜索
      await searchInput.setValue('iPhone')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      expect(mockSearchService.search).toHaveBeenCalledTimes(1)
      
      // 搜索其他关键词
      await searchInput.setValue('Samsung')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 再次搜索iPhone（应该使用缓存）
      await searchInput.setValue('iPhone')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // API应该只被调用2次（第二次iPhone搜索使用缓存）
      expect(mockSearchService.search).toHaveBeenCalledTimes(2)
      
      // 结果应该立即显示（无加载状态）
      expect(wrapper.find('[data-testid="loading-spinner"]').exists()).toBe(false)
      expect(wrapper.findAll('[data-testid="product-item"]')).toHaveLength(2)
    })

    test('should invalidate cache when filters change', async () => {
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      // 搜索iPhone
      await searchInput.setValue('iPhone')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 添加品牌过滤器
      await wrapper.find('[data-testid="brand-filter-Apple"]').setChecked(true)
      await flushPromises()
      
      // 再次搜索iPhone（过滤器变化，缓存应该失效）
      await searchInput.setValue('iPhone')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 应该发起新的API调用
      expect(mockSearchService.search).toHaveBeenCalledTimes(3)
    })
  })

  describe('Pagination & Infinite Scroll', () => {
    test('should load more results on infinite scroll', async () => {
      // Mock第二页数据
      mockSearchService.search.mockResolvedValueOnce({
        products: [
          {
            id: 'prod-3',
            name: 'iPhone 14',
            price: 799.99,
            brand: 'Apple'
          }
        ],
        total: 120,
        page: 2,
        pageSize: 20
      })
      
      // 模拟滚动到底部
      const scrollContainer = wrapper.find('[data-testid="results-container"]')
      await scrollContainer.trigger('scroll', {
        target: {
          scrollTop: 1000,
          scrollHeight: 1100,
          clientHeight: 100
        }
      })
      
      await flushPromises()
      
      // 应该加载更多数据
      expect(mockSearchService.search).toHaveBeenCalledWith(
        expect.objectContaining({
          page: 2
        })
      )
      
      // 新商品应该被添加到列表中
      expect(wrapper.findAll('[data-testid="product-item"]')).toHaveLength(3)
      expect(wrapper.text()).toContain('iPhone 14')
    })

    test('should show loading state during infinite scroll', async () => {
      let resolveSearch
      mockSearchService.search.mockReturnValue(
        new Promise(resolve => {
          resolveSearch = resolve
        })
      )
      
      // 触发无限滚动
      const scrollContainer = wrapper.find('[data-testid="results-container"]')
      await scrollContainer.trigger('scroll', {
        target: {
          scrollTop: 1000,
          scrollHeight: 1100,
          clientHeight: 100
        }
      })
      
      // 应该显示加载更多状态
      expect(wrapper.find('[data-testid="loading-more"]').exists()).toBe(true)
      
      // 完成加载
      resolveSearch({
        products: [{ id: 'prod-3', name: 'iPhone 14' }],
        total: 120,
        page: 2
      })
      await flushPromises()
      
      // 加载状态应该消失
      expect(wrapper.find('[data-testid="loading-more"]').exists()).toBe(false)
    })

    test('should handle virtual scrolling for large datasets', async () => {
      // Mock大量数据
      const largeDataset = Array.from({ length: 1000 }, (_, i) => ({
        id: `prod-${i}`,
        name: `Product ${i}`,
        price: 100 + i
      }))
      
      mockSearchService.search.mockResolvedValue({
        products: largeDataset,
        total: 1000,
        page: 1
      })
      
      await wrapper.find('[data-testid="search-input"]').setValue('test')
      await wrapper.find('[data-testid="search-input"]').trigger('keyup.enter')
      await flushPromises()
      
      // 虚拟滚动应该只渲染可见的商品
      const visibleItems = wrapper.findAll('[data-testid="product-item"]')
      expect(visibleItems.length).toBeLessThan(1000)
      expect(visibleItems.length).toBeGreaterThan(0)
      
      // 应该有滚动容器的正确高度
      const virtualContainer = wrapper.find('[data-testid="virtual-scroll-container"]')
      expect(virtualContainer.attributes('style')).toContain('height')
    })
  })

  describe('Product Interactions', () => {
    test('should toggle product favorite status', async () => {
      const favoriteService = {
        addFavorite: vi.fn().mockResolvedValue(true),
        removeFavorite: vi.fn().mockResolvedValue(true),
        getFavoriteStatus: vi.fn().mockResolvedValue(false)
      }
      
      wrapper.vm.$options.provide = {
        favoriteService
      }
      
      const favoriteBtn = wrapper.find('[data-testid="product-item-prod-1"] [data-testid="favorite-btn"]')
      
      // 点击收藏
      await favoriteBtn.trigger('click')
      
      // 应该调用收藏API
      expect(favoriteService.addFavorite).toHaveBeenCalledWith('prod-1')
      
      // 收藏按钮状态应该改变
      expect(favoriteBtn.classes()).toContain('favorited')
      
      // 应该发射收藏事件
      expect(wrapper.emitted('product-favorited')).toBeTruthy()
    })

    test('should add products to comparison', async () => {
      const product1 = wrapper.find('[data-testid="product-item-prod-1"]')
      const product2 = wrapper.find('[data-testid="product-item-prod-2"]')
      
      // 选择第一个商品比较
      await product1.find('[data-testid="compare-checkbox"]').setChecked(true)
      
      // 选择第二个商品比较
      await product2.find('[data-testid="compare-checkbox"]').setChecked(true)
      
      // 应该显示比较按钮
      const compareBtn = wrapper.find('[data-testid="compare-products-btn"]')
      expect(compareBtn.exists()).toBe(true)
      expect(compareBtn.text()).toContain('比较商品 (2)')
      
      // 点击比较
      await compareBtn.trigger('click')
      
      // 应该发射比较事件
      expect(wrapper.emitted('product-compared')).toBeTruthy()
      expect(wrapper.emitted('product-compared')[0][0]).toHaveLength(2)
    })

    test('should limit comparison to maximum items', async () => {
      // 假设最多只能比较3个商品
      const products = wrapper.findAll('[data-testid="product-item"]')
      
      // 尝试选择更多商品
      for (let i = 0; i < products.length; i++) {
        const checkbox = products[i].find('[data-testid="compare-checkbox"]')
        await checkbox.setChecked(true)
        
        if (i >= 2) { // 超过限制后
          // 复选框应该被禁用或显示提示
          expect(checkbox.attributes('disabled')).toBeDefined()
          break
        }
      }
      
      // 应该显示限制提示
      expect(wrapper.find('[data-testid="compare-limit-message"]').exists()).toBe(true)
      expect(wrapper.text()).toContain('最多只能比较3个商品')
    })

    test('should track product click events', async () => {
      const productLink = wrapper.find('[data-testid="product-item-prod-1"] [data-testid="product-link"]')
      
      await productLink.trigger('click')
      
      // 应该调用统计API
      expect(mockStatsService.trackProductClick).toHaveBeenCalledWith({
        productId: 'prod-1',
        position: 1,
        searchKeyword: 'iPhone',
        filters: expect.any(Object)
      })
      
      // 应该发射商品点击事件
      expect(wrapper.emitted('product-clicked')).toBeTruthy()
    })
  })

  describe('Auto-complete & Suggestions', () => {
    test('should show search suggestions while typing', async () => {
      vi.useFakeTimers()
      
      mockSearchService.getSuggestions.mockResolvedValue([
        'iPhone 15',
        'iPhone 14',
        'iPhone Pro',
        'iPhone Mini'
      ])
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      await searchInput.setValue('iph')
      await searchInput.trigger('input')
      
      // 等待防抖
      vi.advanceTimersByTime(300)
      await flushPromises()
      
      // 应该显示建议列表
      const suggestions = wrapper.findAll('[data-testid="suggestion-item"]')
      expect(suggestions).toHaveLength(4)
      expect(suggestions[0].text()).toBe('iPhone 15')
      
      // 点击建议项应该填充搜索框
      await suggestions[0].trigger('click')
      expect(searchInput.element.value).toBe('iPhone 15')
      
      vi.useRealTimers()
    })

    test('should handle keyboard navigation in suggestions', async () => {
      mockSearchService.getSuggestions.mockResolvedValue([
        'iPhone 15',
        'iPhone 14'
      ])
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      
      await searchInput.setValue('iph')
      await searchInput.trigger('input')
      await flushPromises()
      
      // 使用方向键导航
      await searchInput.trigger('keydown.down')
      await searchInput.trigger('keydown.down')
      
      // 第二个建议应该被高亮
      const highlightedSuggestion = wrapper.find('[data-testid="suggestion-item"].highlighted')
      expect(highlightedSuggestion.text()).toBe('iPhone 14')
      
      // 按回车应该选择高亮的建议
      await searchInput.trigger('keydown.enter')
      expect(searchInput.element.value).toBe('iPhone 14')
    })
  })

  describe('Error Handling & Edge Cases', () => {
    test('should handle search API errors gracefully', async () => {
      mockSearchService.search.mockRejectedValue(new Error('API Error'))
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      await searchInput.setValue('test')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 应该显示错误信息
      expect(wrapper.find('[data-testid="search-error"]').exists()).toBe(true)
      expect(wrapper.text()).toContain('搜索失败，请稍后重试')
      
      // 应该提供重试按钮
      const retryBtn = wrapper.find('[data-testid="retry-search-btn"]')
      expect(retryBtn.exists()).toBe(true)
      
      // 重试应该重新发起搜索
      mockSearchService.search.mockResolvedValue({ products: [], total: 0 })
      await retryBtn.trigger('click')
      
      expect(mockSearchService.search).toHaveBeenCalledTimes(2)
    })

    test('should handle network timeout', async () => {
      // Mock超时错误
      mockSearchService.search.mockRejectedValue(new Error('timeout'))
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      await searchInput.setValue('test')
      await searchInput.trigger('keyup.enter')
      await flushPromises()
      
      // 应该显示超时提示
      expect(wrapper.text()).toContain('请求超时')
      expect(wrapper.find('[data-testid="retry-search-btn"]').exists()).toBe(true)
    })

    test('should handle concurrent search requests correctly', async () => {
      let resolveFirstSearch, resolveSecondSearch
      
      // 第一次搜索
      mockSearchService.search.mockReturnValueOnce(
        new Promise(resolve => {
          resolveFirstSearch = resolve
        })
      )
      
      const searchInput = wrapper.find('[data-testid="search-input"]')
      await searchInput.setValue('first')
      await searchInput.trigger('keyup.enter')
      
      // 第二次搜索（在第一次完成前）
      mockSearchService.search.mockReturnValueOnce(
        new Promise(resolve => {
          resolveSecondSearch = resolve
        })
      )
      
      await searchInput.setValue('second')
      await searchInput.trigger('keyup.enter')
      
      // 第二次搜索先完成
      resolveSecondSearch({
        products: [{ id: 'second-result', name: 'Second Result' }],
        total: 1
      })
      await flushPromises()
      
      // 第一次搜索后完成
      resolveFirstSearch({
        products: [{ id: 'first-result', name: 'First Result' }],
        total: 1
      })
      await flushPromises()
      
      // 应该显示第二次搜索的结果（最新的）
      expect(wrapper.text()).toContain('Second Result')
      expect(wrapper.text()).not.toContain('First Result')
    })
  })
})
```

#### 完整测试总结和最佳实践

这个复杂的搜索页面实例展示了企业级Vue组件的AI+TDD实践精髓：

**核心价值体现**：

1. **分层复杂度处理**：从基础搜索→路由集成→性能优化→错误处理的递进式测试策略
2. **精确的Mock策略**：针对不同类型的依赖（API、浏览器API、第三方库）采用不同的Mock方案
3. **异步协调测试**：正确处理多个API调用、竞态条件、防抖和缓存逻辑
4. **用户体验验证**：加载状态、错误恢复、性能优化等关键用户体验的完整测试覆盖

**AI协作关键点**：

- **人类设计**：复杂的测试分层策略、业务场景分析、Mock架构设计
- **AI实现**：具体的测试代码生成、边界条件覆盖、异步处理逻辑
- **迭代改进**：通过Review过程不断完善测试的完整性和准确性

通过这三个递进式实例，我们完整展示了从简单组件到复杂页面的AI+TDD实践方法，为Vue开发者提供了可直接应用的实战指导。

## 6. 深度实战指导与最佳实践（2500字）

### 6.1 Vue AI+TDD的核心驾驭技巧

#### 理解AI在Vue测试中的认知盲区

AI在处理Vue组件测试时存在一些固有的认知限制，了解这些限制是成功驾驭AI的关键：

**响应式系统的时序理解不足**：
```javascript
// AI常生成的错误模式
test('数据更新测试', async () => {
  wrapper.vm.count = 5
  expect(wrapper.text()).toContain('5') // ❌ 忽略了响应式更新的异步性
})

// 正确的引导方式
test('数据更新需要等待nextTick', async () => {
  wrapper.vm.count = 5
  await wrapper.vm.$nextTick() // ✅ 明确指出需要等待DOM更新
  expect(wrapper.text()).toContain('5')
})
```

**组件边界理解模糊**：
```javascript
// ❌ AI倾向于测试内部实现
expect(wrapper.vm.internalState).toBe(true)

// ✅ 正确的组件接口测试
expect(wrapper.emitted('state-changed')).toBeTruthy()
expect(wrapper.find('[data-testid="indicator"]').classes()).toContain('active')
```

#### 精确引导AI的策略技巧

**策略1：具体化需求描述**
```markdown
❌ 模糊描述："测试组件的基本功能"

✅ 精确描述：
"测试购物车组件的以下行为：
1. 当用户点击'+1'按钮时，商品数量增加1且触发'quantity-changed'事件
2. 当数量达到库存上限时，'+1'按钮应该被禁用
3. 当用户手动输入数量时，需要验证输入范围并更新小计金额"
```

**策略2：提供Vue特定的上下文**
```javascript
// 在Prompt中明确Vue版本和技术栈
"请使用Vue 3 Composition API和Vue Test Utils生成测试，重点关注：
- 响应式数据的正确更新时机
- 组件间通信的事件验证  
- 计算属性的依赖关系测试
- 生命周期钩子的执行顺序"
```

**策略3：分层递进式指导**
```markdown
第一步：基础功能（纯组件逻辑）
第二步：状态管理集成（Pinia/Vuex）
第三步：异步行为（API调用、防抖）
第四步：边界条件和错误处理
```

#### 常见AI错误模式的预防和修正

**错误模式1：过度依赖内部实现**
```javascript
// AI生成的问题代码
test('内部状态测试', () => {
  expect(wrapper.vm.$data.isLoading).toBe(false)
  expect(wrapper.vm.computedValue).toBe(100)
})

// 修正指导："重新设计这个测试，聚焦组件的公共接口而非内部实现"
test('加载状态的用户可见行为', () => {
  expect(wrapper.find('[data-testid="loading-spinner"]').exists()).toBe(false)
  expect(wrapper.find('[data-testid="computed-result"]').text()).toBe('100')
})
```

**错误模式2：异步处理不当**
```javascript
// AI常见错误
test('异步操作', async () => {
  wrapper.vm.loadData()
  expect(wrapper.vm.data).toBeDefined() // ❌ 没有等待异步完成
})

// 修正指导："添加适当的异步等待机制"
test('异步数据加载', async () => {
  const loadPromise = wrapper.vm.loadData()
  expect(wrapper.vm.isLoading).toBe(true) // 验证加载状态
  
  await loadPromise
  expect(wrapper.vm.isLoading).toBe(false)
  expect(wrapper.vm.data).toBeDefined()
})
```

### 6.2 复杂Vue场景的测试架构设计

#### 大型组件的分层测试策略

**层次1：纯逻辑层测试**
```javascript
// 隔离业务逻辑，独立测试计算函数
import { calculateCartTotal, validateQuantity } from '@/utils/cart-logic'

describe('购物车计算逻辑', () => {
  test('价格计算准确性', () => {
    const items = [
      { price: 100, quantity: 2 },
      { price: 50, quantity: 3 }
    ]
    expect(calculateCartTotal(items)).toBe(350)
  })
})
```

**层次2：组件交互层测试**
```javascript
// 测试组件间的数据流和事件通信
describe('父子组件交互', () => {
  test('子组件事件正确传递到父组件', async () => {
    const parent = mount(ParentComponent)
    const child = parent.findComponent(ChildComponent)
    
    await child.vm.$emit('item-selected', { id: 1 })
    
    expect(parent.emitted('selection-changed')).toBeTruthy()
  })
})
```

**层次3：集成测试层**
```javascript
// 测试完整的用户工作流
describe('购物车完整流程', () => {
  test('从添加商品到结算的完整用户路径', async () => {
    // 模拟完整的用户操作序列
    await wrapper.find('[data-testid="add-to-cart"]').trigger('click')
    await wrapper.find('[data-testid="quantity-input"]').setValue(3)
    await wrapper.find('[data-testid="apply-coupon"]').trigger('click')
    await wrapper.find('[data-testid="checkout"]').trigger('click')
    
    // 验证最终状态
    expect(wrapper.emitted('checkout-initiated')).toBeTruthy()
  })
})
```

#### Vue Router集成的测试模式

**模式1：路由参数同步测试**
```javascript
describe('URL状态同步', () => {
  test('组件状态变化应反映到URL', async () => {
    await wrapper.setData({ currentTab: 'reviews' })
    
    expect(router.currentRoute.value.query.tab).toBe('reviews')
  })
  
  test('URL变化应更新组件状态', async () => {
    await router.push('/product/123?tab=specs')
    
    expect(wrapper.vm.currentTab).toBe('specs')
    expect(wrapper.vm.productId).toBe('123')
  })
})
```

**模式2：导航守卫测试**
```javascript
describe('路由导航守卫', () => {
  test('未登录用户访问受保护页面应重定向', async () => {
    // Mock未登录状态
    mockAuthStore.isAuthenticated = false
    
    await router.push('/dashboard')
    
    expect(router.currentRoute.value.path).toBe('/login')
  })
})
```

#### Pinia/Vuex状态管理的测试集成

**状态同步测试模式**：
```javascript
describe('Pinia Store集成', () => {
  test('组件操作应同步更新store状态', async () => {
    const store = useCartStore()
    
    await wrapper.find('[data-testid="add-item"]').trigger('click')
    
    expect(store.items).toHaveLength(1)
    expect(store.totalAmount).toBeGreaterThan(0)
  })
  
  test('store状态变化应反映到组件', async () => {
    const store = useCartStore()
    
    store.addItem({ id: 1, name: 'Test', price: 100 })
    
    await wrapper.vm.$nextTick()
    expect(wrapper.text()).toContain('Test')
  })
})
```

### 6.3 AI协作中的高级技巧和经验

#### 提示词工程的最佳实践

**结构化Prompt模板**：
```markdown
# Vue组件测试生成请求

## 组件背景
- 组件名称：ShoppingCartItem
- 技术栈：Vue 3 + TypeScript + Pinia
- 主要功能：商品数量修改、删除、收藏

## 测试重点
1. 数量修改的边界处理（最小值1，最大值=库存）
2. 删除确认流程的用户体验
3. 收藏状态的异步更新
4. 价格计算的准确性验证

## 技术约束  
- 使用Vue Test Utils进行组件挂载
- Mock外部API调用（收藏服务、库存验证）
- 关注Vue响应式更新的时序
- 验证事件发射的参数完整性

## 期望产出
- 覆盖Happy Path和边界条件的完整测试套件
- 包含异步操作的正确处理方式
- 遵循AAA测试结构的可读代码
```

**渐进式细化策略**：
```markdown
第一轮：生成基础功能测试骨架
第二轮：补充边界条件和异常处理
第三轮：添加性能和用户体验相关测试
第四轮：完善Mock策略和集成测试
```

#### 测试代码质量的评审要点

**Vue特有的评审清单**：

1. **响应式处理**：
   - 是否正确使用`nextTick`等待DOM更新
   - 计算属性的依赖关系测试是否完整
   - 监听器的触发条件是否准确验证

2. **组件通信**：
   - Props验证是否覆盖类型和默认值
   - Events发射是否包含正确的参数
   - 插槽内容是否正确渲染

3. **生命周期**：
   - 异步组件的加载状态处理
   - 组件卸载时的资源清理
   - Keep-alive场景的激活/失活行为

4. **状态管理**：
   - Store状态与组件状态的同步性
   - 跨组件状态共享的一致性
   - 持久化存储的正确性

#### 迭代改进的循环模式

**标准改进循环**：
```
1. AI生成初版测试 
   ↓
2. 人工Review识别问题
   ↓  
3. 具体化改进指导
   ↓
4. AI修正和优化
   ↓
5. 验证业务覆盖度
   ↓
6. 重复循环直至满意
```

**每轮Review的关注点**：
- 第1轮：基础功能覆盖度
- 第2轮：边界条件完整性  
- 第3轮：异步处理正确性
- 第4轮：用户体验验证
- 第5轮：代码可维护性

### 6.4 团队协作和知识沉淀

#### 建立Vue测试规范

**组件测试标准模板**：
```javascript
// 标准的Vue组件测试文件结构
describe('ComponentName', () => {
  // 基础设置
  let wrapper
  
  beforeEach(() => {
    wrapper = mount(Component, {
      props: { /* 标准props */ },
      global: { /* 全局配置 */ }
    })
  })
  
  describe('渲染和Props', () => {
    // Props传递和渲染测试
  })
  
  describe('用户交互', () => {
    // 用户操作和事件测试
  })
  
  describe('业务逻辑', () => {
    // 核心业务功能测试
  })
  
  describe('边界条件', () => {
    // 异常情况和边界测试
  })
  
  describe('集成场景', () => {
    // 与其他组件/系统的集成测试
  })
})
```

#### AI协作经验的团队共享

**经验知识库建设**：
```markdown
# Vue AI+TDD经验库

## 常见问题解决方案
### Q: AI生成的测试总是忽略nextTick？
A: 在Prompt中明确要求"注意Vue的响应式更新需要等待nextTick"

### Q: 如何让AI理解复杂的组件通信？
A: 提供完整的组件接口定义和交互流程图

## 最佳Prompt模板库
- 组件基础功能测试模板
- 状态管理集成测试模板  
- 路由集成测试模板
- 性能优化测试模板

## 代码Review清单
- Vue响应式处理检查项
- 组件通信验证要点
- 异步处理评审标准
- 用户体验测试要求
```

#### 持续改进和工具优化

**测试质量监控**：
```javascript
// 测试覆盖度和质量指标
const testMetrics = {
  coverageTarget: {
    statements: 90,
    branches: 85,
    functions: 90,
    lines: 90
  },
  
  qualityChecks: {
    hasAsyncTests: true,
    hasBoundaryTests: true,
    hasErrorHandling: true,
    hasUserExperienceTests: true
  }
}
```

**AI协作效果评估**：
```markdown
## 定期评估指标
- AI生成测试的准确率（需人工修正的比例）
- 测试编写效率提升（时间对比）
- 缺陷捕获率（测试发现问题的比例）
- 团队满意度（AI协作体验评分）

## 持续优化方向
- 优化Prompt模板库
- 完善Review清单
- 更新最佳实践文档
- 提升团队AI协作技能
```

---

**总结**：Vue AI+TDD的实战精髓在于理解AI的能力边界，通过精确的引导策略和系统化的测试架构，实现人机协作的最大效能。关键是建立可重复的协作模式，持续积累和分享经验，让整个团队都能掌握这种高效的开发方式。