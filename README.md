# Univers Strategy

这个仓库用于记录和管理 Univers 项目的战略规划、产品路线图和重要决策。

## 核心文档

**必读**：[PHILOSOPHY.md](PHILOSOPHY.md) - Univers 的核心哲学和价值观，是所有战略决策的基础。

## 快速导航

### 新人入门
1. [核心哲学](PHILOSOPHY.md) - 理解 Univers 是什么，为什么这样设计
2. [三层架构决策](decisions/001-三层架构设计.md) - 理解系统架构
3. [产品设计原则](product/design-principles.md) - 如何设计产品

### 技术人员
- [系统架构](tech/architecture.md) - 技术实现细节
- [决策记录](decisions/) - 重要技术决策

### 产品经理
- [产品设计原则](product/design-principles.md) - 产品设计指南
- [产品路线图](product/roadmap.md) - 功能规划

### 管理层 / HR
- [核心哲学](PHILOSOPHY.md) - 战略方向和组织理念
- [人才战略](business/talent-strategy.md) - 招聘、组织、文化
- [业务目标](business/objectives.md) - 年度/季度目标

## 核心更新（2025-11-02）

本次更新围绕两个核心战略展开：

**1. 技术层面：弥补 AI 的缩放局限**
- 认识到 AI 难以主动在多个尺度间切换思考
- 通过系统设计（多尺度视图、缩放提示引擎等）来弥补
- 详见 [PHILOSOPHY.md](PHILOSOPHY.md) 和 [tech/architecture.md](tech/architecture.md)

**2. 组织层面：找准人类价值，组织正确的人才**
- 我们需要的不是"会用 AI 的人"，而是"善于缩放的人"
- 重新定义招聘标准、组织结构、文化和考核
- 详见 [business/talent-strategy.md](business/talent-strategy.md)

## 目录结构

```
univers-strategy/
├── PHILOSOPHY.md         # 核心哲学（必读）✓✓✓
├── business/             # 业务战略
│   ├── talent-strategy.md        # 人才战略 ✓
│   ├── vision.md                 # 愿景和使命
│   └── objectives.md             # 年度/季度目标
├── product/              # 产品规划
│   ├── design-principles.md      # 产品设计原则 ✓
│   └── roadmap.md                # 产品路线图
├── tech/                 # 技术战略
│   └── architecture.md           # 系统架构（含 AI 局限弥补）✓
├── operations/           # 运营战略
│   ├── marketing.md              # 市场营销
│   └── growth.md                 # 增长策略
└── decisions/            # 决策记录 (ADR)
    ├── README.md                 # ADR 模板说明
    └── 001-三层架构设计.md        # 核心架构决策 ✓
```

**说明**：
- ✓ 标记的文档已完成
- 未标记的为模板，待填充

## 文档说明

### 核心哲学 (PHILOSOPHY.md)

Univers 的"宪法"，定义：
- 我们是谁
- 我们相信什么
- 核心设计理念
- 战略方向

**所有决策都应该符合核心哲学。**

### 决策记录 (decisions/)

重要决策的记录，采用 ADR (Architecture Decision Records) 风格。

**现有决策**：
- [001-三层架构设计](decisions/001-三层架构设计.md) - Workbench/Skills/Operation 三层架构

### 技术战略 (tech/)

技术架构和实现细节：
- [architecture.md](tech/architecture.md) - 系统架构、技术栈、部署方案

### 产品规划 (product/)

产品设计和规划：
- [design-principles.md](product/design-principles.md) - 产品设计原则和用户界面指南
- roadmap.md - 产品路线图（待填充）

### 业务战略 (business/)

长期愿景、目标和人才：
- [talent-strategy.md](business/talent-strategy.md) - 人才战略：基于 AI 局限性的组织设计 ✓
- vision.md - 愿景和使命（待填充）
- objectives.md - 年度/季度目标（待填充）

### 运营战略 (operations/)

市场和增长相关：
- 待填充

## 关键概念

### 上下文自由缩放（Context-Free Scaling）

人类在 AI 时代的核心优势：能在多个维度上自由切换思考的尺度。

**维度**：
- **抽象层级**：细节 ↔ 规则 ↔ 原则 ↔ 哲学
- **时间尺度**：当下 ↔ 今天 ↔ 本月 ↔ 本年 ↔ 长期
- **空间范围**：个体 ↔ 团队 ↔ 公司 ↔ 客户 ↔ 社会
- **框架质疑**：在框架内 → 跳出框架 → 重新定义问题

**为什么重要**：
- AI 只能在给定尺度和框架内优化
- 人类选择尺度和框架，这是无法被 AI 替代的能力
- Univers 的所有设计都围绕支持这个能力

详见 [PHILOSOPHY.md](PHILOSOPHY.md) 第 2 节

## 协作原则

- 重要战略和决策应该经过团队讨论
- 使用 Pull Request 进行审查
- 保持文档更新，反映最新的战略方向
- 定期回顾和调整

## 相关仓库

- [univers-container](../univers-container) - 容器管理和开发环境配置
