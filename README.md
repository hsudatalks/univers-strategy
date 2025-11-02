# Univers Strategy

这个仓库用于记录和管理 Univers 项目的战略规划、产品路线图和重要决策。

## 目录结构

```
univers-strategy/
├── business/          # 业务战略
│   ├── vision.md             # 愿景和使命
│   ├── objectives.md         # 年度/季度目标
│   └── competitive-analysis.md # 竞争分析
├── product/           # 产品规划
│   ├── roadmap.md            # 产品路线图
│   ├── features.md           # 功能规划
│   └── user-research.md      # 用户研究
├── tech/              # 技术战略
│   ├── architecture.md       # 架构演进
│   ├── tech-stack.md         # 技术栈选型
│   └── infrastructure.md     # 基础设施规划
├── operations/        # 运营战略
│   ├── marketing.md          # 市场营销
│   └── growth.md             # 增长策略
└── decisions/         # 决策记录 (ADR 风格)
    └── README.md             # 决策记录模板说明
```

## 使用指南

### 业务战略 (business/)

记录公司/产品的长期愿景、目标和市场分析。

### 产品规划 (product/)

包含产品路线图、功能规划和用户研究成果。

### 技术战略 (tech/)

技术架构演进方向、技术栈选型依据和基础设施规划。

### 运营战略 (operations/)

市场营销、用户增长等运营相关的战略规划。

### 决策记录 (decisions/)

重要决策的记录，采用 ADR (Architecture Decision Records) 风格，帮助团队理解为什么做某个决定。

每个决策文档格式：
- 标题：`XXX-简短描述.md`
- 内容包含：背景、决策、后果、替代方案

## 协作原则

- 重要战略和决策应该经过团队讨论
- 使用 Pull Request 进行审查
- 保持文档更新，反映最新的战略方向
- 定期回顾和调整

## 相关仓库

- [univers-container](../univers-container) - 容器管理和开发环境配置
