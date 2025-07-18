# AI测试资产生产流程指导体系

## 概述

本指导体系旨在建立一个跨职能团队协作的AI测试资产生产流程，让程序员、产品经理、业务专家都能有效参与测试资产的生产和质量保障。

## 核心问题解决

### 1. 程序员如何驱动AI生产测试资产？
- **指导原则**：[driving_principles/](./driving_principles/) 提供场景化的驱动策略
- **模板库**：[md_instruction_templates/](./driving_principles/md_instruction_templates/) 提供标准化指令模板
- **实战指导**：如何将业务需求转化为AI可理解的MD指令

### 2. 非研发人员如何REVIEW测试思路？
- **REVIEW框架**：[review_framework/](./review_framework/) 提供系统化的评审方法
- **检查清单**：业务逻辑、场景覆盖、边界条件的评估标准
- **质量门禁**：明确的质量标准和验收准则

### 3. 如何给出有效反馈进行迭代？
- **反馈流程**：[feedback_iteration/](./feedback_iteration/) 提供结构化的反馈方法
- **分类优先级**：如何对反馈进行分类和优先级设定
- **迭代收敛**：如何判断测试资产已达到可用标准

## 使用指南

### 快速开始
1. 阅读 [00_workflow_overview.md](./00_workflow_overview.md) 了解整体流程
2. 根据角色查看对应的指导文件：
   - 程序员：[roles_and_responsibilities/programmer_guide.md](./roles_and_responsibilities/programmer_guide.md)
   - 产品经理：[roles_and_responsibilities/product_manager_guide.md](./roles_and_responsibilities/product_manager_guide.md)
   - 业务专家：[roles_and_responsibilities/business_expert_guide.md](./roles_and_responsibilities/business_expert_guide.md)

### 按场景使用
- **API开发测试**：[driving_principles/scenario_specific/api_scenario.md](./driving_principles/scenario_specific/api_scenario.md)
- **前端开发测试**：[driving_principles/scenario_specific/frontend_scenario.md](./driving_principles/scenario_specific/frontend_scenario.md)
- **数据库相关测试**：[driving_principles/scenario_specific/database_scenario.md](./driving_principles/scenario_specific/database_scenario.md)

## 文档结构

```
ai_test_workflow/
├── README.md                        # 本文件
├── 00_workflow_overview.md          # 整体流程概览
├── roles_and_responsibilities/      # 角色职责指导
├── driving_principles/              # 驱动AI的指导原则
├── review_framework/                # 非研发人员REVIEW指导
├── feedback_iteration/              # 反馈调整流程
├── best_practices/                  # 最佳实践
└── templates/                       # 实用模板
```

## 与技术指南的关系

- **本指导体系**：面向跨职能团队协作，重点在流程和沟通
- **EXPLORE_TestAsset**：面向程序员深度实践，重点在技术细节
- **互补关系**：流程指导 + 技术指南 = 完整的AI测试资产生产体系

## 贡献指南

欢迎基于实际使用经验提供反馈和改进建议，共同完善这个跨职能协作框架。

---

**更新时间**：2025-07-17  
**维护者**：AI测试资产生产流程工作组