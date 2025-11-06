# ADR 005-9: 技术基础设施 - 从Skills到专业系统

**状态**：已通过
**日期**：2025-11-06
**所属系列**：[ADR 005: AI时代的生产关系转变](../005-production-relations-transformation.md)

---

## 执行摘要

本文档揭示Workbench成功的**技术真相**：它不是靠"写文档让AI学习"，而是靠开发专业软件系统。

**核心发现**：
- Skills = 可执行脚本（不是知识文档）
- 知识 = 编码在hvac-workbench系统中（软件代码，不是SKILL.md）
- $1M投资 = 开发专业系统（不是写文档）
- Claude的角色 = 会用工具的智能助手（不是学习专业知识的专家）

**本文目标**：
- 澄清对Skills架构的常见误解
- 解释hvac-workbench系统的组成
- 说明如何开发你的第一个专业系统

---

## 一、常见误解与真相

### 1.1 误解：Skills是知识文档

```yaml
很多人以为的架构:

  ┌─────────────┐
  │   Claude    │
  └──────┬──────┘
         │ reads
         ↓
  ┌─────────────┐
  │  SKILL.md   │ ← "如何review代码的文档"
  │  (知识描述) │    "如何优化HVAC的指南"
  └─────────────┘

  Claude读文档 → 学会知识 → 自己执行任务

投资方向:
  ✗ 写更好的文档
  ✗ 编写详细的知识库
  ✗ 训练AI理解领域知识
```

**为什么这个理解是错的？**

1. **LLM的能力边界**：
   - Claude擅长理解自然语言、调用工具、做决策
   - Claude不擅长深度专业分析、复杂算法、历史数据分析
   - 文档再详细，Claude也无法替代专业系统

2. **知识编码的局限**：
   - 文字描述永远是有损的（"代码质量"无法用文字穷尽）
   - 边界情况无法全部列举
   - 判断标准难以形式化

3. **可复用性差**：
   - 每个新客户都要重写文档
   - 知识无法积累
   - 边际成本高

### 1.2 真相：Skills是工具接口

```yaml
实际的架构:

  ┌─────────────┐
  │   Claude    │
  └──────┬──────┘
         │ calls
         ↓
  ┌─────────────┐
  │ Skill脚本   │ ← bin/cm, bin/check-quality等
  │ (工具接口)  │    (可执行的bash/脚本)
  └──────┬──────┘
         │ invokes
         ↓
  ┌─────────────┐
  │hvac-workbench│ ← 真实的软件系统 ⭐核心投资⭐
  │   (系统)     │   - 代码分析引擎
  └─────────────┘   - HVAC优化算法
                    - 性能基准测试
                    - 自动化CI/CD
                    - ...

投资方向:
  ✓ 开发专业系统（软件代码）
  ✓ 优化算法准确性
  ✓ 建立自动化管道
```

**为什么这个架构work？**

1. **专业逻辑在系统中**：
   - 代码质量检查 = pattern-detector.ts（算法）
   - HVAC优化 = control-optimizer.ts（计算模型）
   - 不依赖Claude"理解"专业知识

2. **结构化输出**：
   - 系统返回JSON/数据（不是自然语言）
   - Claude只需理解数据格式（容易）
   - 不需要理解专业逻辑（困难）

3. **可复用、可积累**：
   - 系统一次开发，多次使用
   - 优化系统 = 优化所有客户的服务
   - 边际成本低

---

## 二、Skills的真实架构

### 2.1 container-manage skill案例分析

**文件结构**：
```bash
.claude/skills/container-manage/
├── SKILL.md                    # 简短描述（约50行）
└── bin/
    ├── cm                      # 主脚本（470行bash）
    └── tmux-manager.sh         # 辅助脚本
```

**SKILL.md的真实作用**（不是知识文档！）：
```markdown
# Container Management Skill

**What it does**: Manage development containers and workbench environment

**Commands**:
- cm init: Initialize hvac-workbench
- cm doctor: Check system health
- cm tmux start: Start tmux sessions

**Usage**: Claude calls `cm <command>` to manage containers
```

→ 只是简短描述，不包含"如何管理容器"的知识

**bin/cm脚本的真实作用**（这才是核心！）：
```bash
#!/bin/bash

# 初始化hvac-workbench（调用真实系统）
init_workbench() {
    local skip_build=$1
    local wb_path="/home/ubuntu/repos/hvac-workbench"

    cd "$wb_path"
    pnpm install --no-frozen-lockfile      # 安装依赖
    pnpm --filter @univers/dev-tools build # 构建工具
    pnpm dev setup                         # 初始化开发环境

    log_success "hvac-workbench initialized"
}

# 系统健康检查
check_doctor() {
    # 检查Node.js, pnpm, Python, Rust, tmux, Git
    # 检查项目路径
    # 返回系统健康状态JSON
    echo '{"status": "healthy", "issues": []}'
}

# 启动tmux会话
tmux_start() {
    bash "$tmux_manager" start "$view_type"
}
```

→ 这是实际执行的代码，调用真实系统

**Claude的工作流程**：
```yaml
用户: "初始化工作环境"

Claude的思考:
  1. 理解任务: 需要初始化环境
  2. 选择工具: container-manage skill
  3. 调用: cm init

系统执行:
  bin/cm init
  → 调用hvac-workbench的pnpm命令
  → 返回: "✓ hvac-workbench initialized"

Claude理解输出:
  - 成功标志: ✓
  - 继续下一步

Claude报告: "工作环境已初始化"
```

**关键点**：Claude没有"学习如何初始化环境"，它只是调用了工具。

### 2.2 code-review skill（推测设计）

**文件结构**（推测）：
```bash
.claude/skills/code-review/
├── SKILL.md                    # 简短描述
└── bin/
    └── review-pr               # review脚本
```

**bin/review-pr脚本**（推测）：
```bash
#!/bin/bash

# 代码review脚本
# 不是"描述如何review"，而是"实际执行review"

PR_NUMBER=$1
PR_FILES=$(gh pr view $PR_NUMBER --json files -q '.files[].path')

# 1. 调用ESLint（基础检查）
echo "Running ESLint..."
pnpm eslint $PR_FILES

# 2. 调用TypeScript检查
echo "Running TypeScript check..."
pnpm tsc --noEmit

# 3. 运行测试
echo "Running tests..."
pnpm test $PR_FILES

# 4. 调用自定义质量检查工具（在hvac-workbench中）⭐关键⭐
echo "Running quality check..."
pnpm quality-check $PR_FILES

# 5. 调用架构一致性检查（在hvac-workbench中）
echo "Running architecture check..."
pnpm arch-check $PR_FILES

# 6. 生成结构化输出
generate_review_report() {
    cat <<EOF
{
  "pr_number": "$PR_NUMBER",
  "eslint": {"passed": true, "issues": []},
  "typescript": {"passed": true, "errors": []},
  "tests": {"passed": true, "coverage": 85},
  "quality": {
    "passed": false,
    "issues": [
      {
        "file": "src/service.ts",
        "line": 42,
        "type": "over-engineering",
        "message": "Unnecessary abstraction detected",
        "severity": "medium"
      }
    ]
  },
  "architecture": {"passed": true, "violations": []},
  "recommendation": "request_changes"
}
EOF
}

generate_review_report
```

**hvac-workbench中的quality-check系统**（这才是$1M投资！）：
```typescript
// packages/quality-checker/src/index.ts

export class QualityChecker {
  private patternDetector: PatternDetector;
  private architectureValidator: ArchitectureValidator;
  private complexityAnalyzer: ComplexityAnalyzer;

  async check(files: string[]): Promise<QualityReport> {
    const issues: Issue[] = [];

    for (const file of files) {
      const ast = await parseToAST(file);

      // 检测问题模式（编码的专业知识）
      issues.push(...this.patternDetector.detect(ast));

      // 验证架构一致性
      issues.push(...this.architectureValidator.validate(ast));

      // 分析复杂度
      issues.push(...this.complexityAnalyzer.analyze(ast));
    }

    return {
      passed: issues.filter(i => i.severity === 'high').length === 0,
      issues
    };
  }
}
```

**专业知识在哪里？**
```typescript
// packages/quality-checker/src/pattern-detector.ts

export class PatternDetector {
  // 这里编码了"什么是过度工程化"
  detectOverEngineering(ast: AST): Issue[] {
    const issues: Issue[] = [];

    // 规则1: 不必要的抽象
    // （这是专业知识，编码在算法中）
    const abstractions = findAbstractions(ast);
    for (const abstraction of abstractions) {
      if (abstraction.usageCount === 1 &&
          abstraction.complexity > 3 &&
          !abstraction.isInInterface) {
        issues.push({
          type: 'over-engineering',
          location: abstraction.location,
          message: 'Unnecessary abstraction: used only once',
          severity: 'medium'
        });
      }
    }

    // 规则2: 过度嵌套
    const maxNesting = getMaxNesting(ast);
    if (maxNesting > 4) {
      issues.push({
        type: 'complexity',
        message: `Excessive nesting: ${maxNesting} levels`,
        severity: 'high'
      });
    }

    // ... 50+ 其他规则

    return issues;
  }

  // 这里编码了"什么是状态管理混乱"
  detectStateManagementIssues(ast: AST): Issue[] {
    // 实际的检测逻辑
    // 基于AST分析、数据流追踪等
  }
}
```

**关键洞察**：
- 专业知识（"什么是好代码"）编码在`pattern-detector.ts`中
- 不是在SKILL.md文档中
- Claude不需要"懂"这些规则
- Claude只需要调用`quality-check`并理解返回的JSON

### 2.3 hvac-optimize skill（推测设计）

**bin/optimize-hvac脚本**：
```bash
#!/bin/bash

BUILDING_ID=$1

# 1. 获取传感器数据
SENSOR_DATA=$(pnpm hvac-system fetch-sensors --building $BUILDING_ID)

# 2. 调用优化系统（在hvac-workbench中）⭐关键⭐
OPTIMIZATION=$(pnpm hvac-optimizer optimize --data "$SENSOR_DATA")

# 3. 返回结构化结果
echo "$OPTIMIZATION"
```

**hvac-workbench中的hvac-optimizer系统**：
```typescript
// packages/hvac-optimizer/src/control-optimizer.ts

export class ControlOptimizer {
  // 这里编码了HVAC专业知识（不是文档！）
  optimize(sensorData: SensorData): OptimizationResult {
    // 1. 分析当前状态
    const currentState = this.analyzeSensors(sensorData);

    // 2. 计算最优控制参数（专业算法）
    const optimalParams = this.calculateOptimal(currentState);

    // 3. 评估风险
    const risk = this.assessRisk(currentState, optimalParams);

    // 4. 估算节能收益
    const savings = this.estimateSavings(currentState, optimalParams);

    return {
      current_efficiency: currentState.efficiency,
      recommended_changes: optimalParams.changes,
      estimated_savings: savings,
      risk_level: risk.level,
      confidence: risk.confidence
    };
  }

  private calculateOptimal(state: HVACState): OptimalParams {
    // 专业的HVAC优化算法
    // 热力学计算、能耗模型、舒适度评估等
    // 这是真正的专业知识（代码，不是文档）

    const thermalModel = this.buildThermalModel(state);
    const energyModel = this.buildEnergyModel(state);
    const comfortModel = this.buildComfortModel(state);

    // 多目标优化
    return this.multiObjectiveOptimization(
      thermalModel,
      energyModel,
      comfortModel
    );
  }
}
```

**Claude的工作流程**：
```yaml
用户: "优化Building A的HVAC系统"

Claude:
  1. 理解任务: 需要优化HVAC
  2. 调用工具: hvac-optimize --building A

系统执行:
  bin/optimize-hvac A
  → 调用hvac-optimizer系统
  → 返回JSON:
    {
      "current_efficiency": 0.72,
      "recommended_changes": [
        {"zone": "1F", "action": "reduce_cooling", "value": -2},
        {"zone": "2F", "action": "increase_ventilation", "value": 5}
      ],
      "estimated_savings": "$1200/month",
      "risk_level": "low",
      "confidence": 0.87
    }

Claude决策:
  - 分析: 风险低，收益明确
  - 决定: 自动执行

Claude报告: "已优化Building A的HVAC，预计节省$1200/月"
```

**Claude不需要懂HVAC**：
- Claude不知道热力学公式
- Claude不理解能耗模型
- Claude只需要理解JSON输出
- Claude只做决策（执行/升级人类）

---

## 三、hvac-workbench系统的组成

### 3.1 系统架构

```yaml
hvac-workbench/
├── packages/                    # 核心专业系统
│   ├── quality-checker/         # 代码质量检查引擎 ⭐
│   │   ├── src/
│   │   │   ├── pattern-detector.ts        # 问题模式识别
│   │   │   ├── architecture-validator.ts  # 架构验证
│   │   │   ├── complexity-analyzer.ts     # 复杂度分析
│   │   │   ├── best-practices.ts          # 最佳实践检查
│   │   │   └── index.ts
│   │   ├── rules/
│   │   │   ├── univers-patterns.json      # 项目特定规则
│   │   │   ├── anti-patterns.json         # 反模式库
│   │   │   └── thresholds.json            # 阈值配置
│   │   └── package.json
│   │
│   ├── hvac-optimizer/          # HVAC优化系统 ⭐
│   │   ├── src/
│   │   │   ├── control-optimizer.ts       # 控制优化
│   │   │   ├── sensor-analyzer.ts         # 传感器分析
│   │   │   ├── energy-calculator.ts       # 能耗计算
│   │   │   ├── comfort-evaluator.ts       # 舒适度评估
│   │   │   ├── anomaly-detector.ts        # 异常检测
│   │   │   └── index.ts
│   │   ├── models/
│   │   │   ├── thermal-model.json         # 热力学模型
│   │   │   ├── energy-model.json          # 能耗模型
│   │   │   └── comfort-model.json         # 舒适度模型
│   │   └── package.json
│   │
│   ├── test-runner/             # 测试执行和覆盖率
│   │   ├── src/
│   │   │   ├── test-executor.ts
│   │   │   ├── coverage-analyzer.ts
│   │   │   └── report-generator.ts
│   │   └── package.json
│   │
│   ├── performance-bench/       # 性能基准测试
│   │   ├── src/
│   │   │   ├── benchmark-runner.ts
│   │   │   ├── metric-collector.ts
│   │   │   └── regression-detector.ts
│   │   └── package.json
│   │
│   └── dev-tools/               # 开发工具
│       ├── src/
│       │   ├── cli/             # CLI接口
│       │   │   ├── review.ts    # review命令
│       │   │   ├── optimize.ts  # optimize命令
│       │   │   ├── monitor.ts   # monitor命令
│       │   │   └── index.ts
│       │   └── utils/
│       └── package.json
│
├── tools/                       # Skills脚本
│   └── skills/
│       ├── code-review/
│       │   └── bin/review-pr
│       └── hvac-optimize/
│           └── bin/optimize-hvac
│
├── config/                      # 配置
│   ├── quality-gates.yaml       # 质量门禁配置
│   ├── hvac-thresholds.yaml     # HVAC阈值
│   └── ci-cd-pipeline.yaml      # CI/CD配置
│
└── package.json
```

### 3.2 核心模块详解

#### 模块1: quality-checker（代码质量检查引擎）

**投资**: $200K（Year 0-1）

**功能**：
```typescript
// 这是一个完整的代码分析系统，不是文档！

export class QualityChecker {
  // 1. 模式识别（50+种问题模式）
  private patternDetector: PatternDetector;

  // 2. 架构验证
  private architectureValidator: ArchitectureValidator;

  // 3. 复杂度分析
  private complexityAnalyzer: ComplexityAnalyzer;

  // 4. 最佳实践检查
  private bestPracticesChecker: BestPracticesChecker;

  async check(files: string[]): Promise<QualityReport> {
    // 执行全面检查
    // 返回结构化报告
  }
}
```

**编码的专业知识**（不是文档，是代码）：
```typescript
// pattern-detector.ts

export const PATTERNS = {
  // 过度工程化
  over_engineering: {
    name: 'Over-Engineering',
    detect: (ast: AST) => {
      // 算法：检测不必要的抽象
      // 输入：AST
      // 输出：Issue[]
    }
  },

  // 状态管理混乱
  state_management_mess: {
    name: 'State Management Issues',
    detect: (ast: AST) => {
      // 算法：检测状态流混乱
      // 数据流分析、依赖追踪
    }
  },

  // ... 50+ 其他模式
};
```

**为什么需要$200K？**
- 开发AST解析器
- 实现50+种模式检测算法
- 建立规则库（基于真实案例）
- 优化准确率（降低误报）
- 集成到CI/CD

#### 模块2: hvac-optimizer（HVAC优化系统）

**投资**: $250K（Year 0-1）

**功能**：
```typescript
export class HVACOptimizer {
  // 1. 传感器数据分析
  private sensorAnalyzer: SensorAnalyzer;

  // 2. 热力学建模
  private thermalModeler: ThermalModeler;

  // 3. 能耗计算
  private energyCalculator: EnergyCalculator;

  // 4. 多目标优化
  private optimizer: MultiObjectiveOptimizer;

  // 5. 异常检测
  private anomalyDetector: AnomalyDetector;

  optimize(sensorData: SensorData): OptimizationResult {
    // 1. 分析当前状态
    const state = this.analyzeCurrent(sensorData);

    // 2. 建立模型
    const model = this.buildModel(state);

    // 3. 计算最优解
    const optimal = this.optimize(model);

    // 4. 评估风险
    const risk = this.assessRisk(optimal);

    return { optimal, risk, savings };
  }
}
```

**编码的HVAC专业知识**：
```typescript
// thermal-modeler.ts

export class ThermalModeler {
  // 热力学第一定律
  private applyFirstLaw(state: HVACState): ThermalState {
    // Q = m * c_p * ΔT
    // 计算热量传递
  }

  // 热力学第二定律
  private applySecondLaw(state: HVACState): EntropyState {
    // 计算熵变
  }

  // 建立热力学模型
  buildModel(sensorData: SensorData): ThermalModel {
    // 复杂的热力学计算
    // 不是"让LLM学习热力学"
    // 而是"系统已经实现了热力学算法"
  }
}
```

**为什么需要$250K？**
- HVAC领域调研（咨询专家）
- 实现热力学算法
- 开发能耗计算模型
- 建立历史数据分析系统
- 传感器数据集成
- 优化算法准确性

#### 模块3: 其他支持系统

**test-runner**: $50K
- 自动化测试执行
- 覆盖率分析
- 报告生成

**performance-bench**: $50K
- 性能基准测试
- 回归检测
- 指标追踪

**dev-tools**: $50K
- CLI接口开发
- 与系统集成
- 输出格式化

### 3.3 系统的进化路径

```yaml
v1.0 (Month 1-3): 基础功能
  投资: $300K
  功能:
    - ESLint/TypeScript集成
    - 基础测试运行
    - 10种问题模式识别
    - HVAC基础优化

  产出:
    - 能检测明显问题
    - 能执行基础优化
    - 误报率 30%

v1.5 (Month 4-6): 优化准确率
  投资: $200K
  功能:
    + 架构一致性验证
    + 50+问题模式
    + HVAC高级优化
    + 性能回归检测

  产出:
    - 误报率降至 15%
    - 能处理80%的常规case
    - 人工监督时间减少50%

v2.0 (Month 7-12): 智能化
  投资: $300K
  功能:
    + 自动修复建议
    + 历史数据学习
    + 预测性维护
    + 异常自动处理

  产出:
    - 误报率降至 8%
    - 85%的case完全自动化
    - 人工监督时间减少90%
    - 形式3真正work ✓

v3.0 (Year 2+): 持续优化
  投资: $200K/年
  功能:
    + 新领域支持
    + 算法优化
    + 客户定制规则

  产出:
    - 多客户复用
    - 边际成本低
    - 规模化 ✓
```

---

## 四、$1M投资的真实去向

### 4.1 投资分解（详细版）

```yaml
总投资: $1.49M (3年)

Year 0-1 ($600K):

  1. 系统开发基础设施 ($150K):
     - CI/CD管道搭建
     - 测试框架
     - 监控系统
     - 开发环境

  2. quality-checker开发 ($200K):
     - 3个月（1个架构师 + 2个工程师）
     - AST解析器开发
     - 50+模式识别算法
     - 规则库建立
     - 降低误报率优化

  3. hvac-optimizer开发 ($250K):
     - 4个月（1个HVAC专家 + 1个算法工程师 + 1个工程师）
     - HVAC领域调研
     - 热力学建模
     - 能耗计算系统
     - 优化算法实现
     - 传感器集成
     - 异常检测逻辑

Year 2 ($400K):

  4. 系统优化和扩展 ($400K):
     - 降低误报率（30% → 8%）
     - 新功能开发（自动修复建议）
     - 性能优化
     - 新领域支持（如果有新客户）
     - 持续维护

Year 3 ($490K):

  5. 规模化和产品化 ($300K):
     - 多租户支持
     - 客户定制规则系统
     - SaaS化（如果需要）
     - 文档和培训材料

  6. 运营和监督 ($190K):
     - Manager工资（部分）
     - 系统运维
     - 客户支持

关键:
  85%的投资在开发软件系统 ⭐
  15%在运营和管理
  0%在"写文档"
```

### 4.2 为什么这个投资值得？

**传统方式（人工专家）**：
```yaml
每个客户配置:
  1个架构师 @ $150K/年
  1个HVAC工程师 @ $120K/年
  1个DevOps工程师 @ $130K/年
  小计: $400K/年/客户

10个客户:
  Year 1: $4M
  Year 2: $4M
  Year 3: $4M
  3年总计: $12M

扩展性: 无，每个新客户都需要新团队
```

**hvac-workbench方式**：
```yaml
初期投资:
  $1.49M（开发系统）

边际成本（每个新客户）:
  - 系统运维: $5K/年
  - 客户定制: $10K（一次性）
  - Manager监督: $15K/年（10%时间）
  小计: ~$20K/年/客户

10个客户:
  Year 1: $1.49M + $200K = $1.69M
  Year 2: $200K
  Year 3: $200K
  3年总计: $2.09M

节省: $12M - $2.09M = $9.91M (83%节省！)

扩展性: 指数级，系统可以服务100+客户
```

**投资回收期**：
```yaml
客户1: 不盈利（覆盖部分开发成本）
客户2: 不盈利（覆盖剩余开发成本）
客户3: 微盈利
客户4-10: 高盈利（边际成本低）
客户10+: 指数级盈利

Break-even: 第3-4个客户
```

这就是形式3的经济学！⭐

---

## 五、Claude的实际角色

### 5.1 Claude不是什么

```yaml
Claude不是:
  ✗ HVAC专家（不懂热力学）
  ✗ 架构师（不懂设计模式本质）
  ✗ 数据科学家（不能做复杂分析）

Claude不需要:
  ✗ 学习HVAC知识（系统已经懂）
  ✗ 理解代码架构（系统会检查）
  ✗ 分析历史数据（系统会分析）
```

### 5.2 Claude是什么

```yaml
Claude是: "会用专业工具的智能助手"

类比:
  传统医生:
    需要: 10年医学训练
    工作: 诊断、开药方

  护士+医疗设备:
    护士需要: 会用设备
    设备提供: 诊断建议
    护士决策: 执行还是升级医生

  Claude = 护士
  hvac-workbench = 医疗设备
```

### 5.3 Claude的实际能力

**Claude擅长的**：
```yaml
1. 自然语言理解:
   用户: "优化Building A的能耗"
   Claude: 理解 → 任务是HVAC优化

2. 工具选择:
   Claude: 需要用hvac-optimize skill

3. 参数构建:
   Claude: 调用 hvac-optimize --building A

4. 输出理解:
   JSON: {"savings": "$1200", "risk": "low"}
   Claude: 节省高、风险低 → 可以执行

5. 决策制定:
   - 风险低 → 自动执行
   - 风险高 → 升级人类
   - 不确定 → 询问

6. 自然语言生成:
   Claude: "已优化Building A，预计节省$1200/月"
```

**Claude不需要的**：
```yaml
✗ 懂热力学公式（系统懂）
✗ 理解优化算法（系统执行）
✗ 分析传感器数据（系统分析）
✗ 评估架构质量（系统评估）
```

### 5.4 工作分工

```yaml
┌─────────────────────────────────────────┐
│           任务：优化Building A          │
└─────────────┬───────────────────────────┘
              │
              ↓
    ┌─────────────────┐
    │     Claude      │
    │   (智能助手)    │
    └────┬────────────┘
         │
         ├── 1. 理解任务 ✓ (Claude擅长)
         │
         ├── 2. 调用工具 ✓ (Claude擅长)
         │    hvac-optimize --building A
         │
         ↓
    ┌─────────────────┐
    │ hvac-workbench  │
    │  (专业系统)     │
    └────┬────────────┘
         │
         ├── 3. 获取传感器数据 ✓ (系统执行)
         ├── 4. 分析当前状态 ✓ (系统执行)
         ├── 5. 热力学建模 ✓ (系统执行)
         ├── 6. 计算最优解 ✓ (系统执行)
         ├── 7. 评估风险 ✓ (系统执行)
         └── 8. 返回JSON ✓ (系统执行)
         │
         ↓
    ┌─────────────────┐
    │     Claude      │
    └────┬────────────┘
         │
         ├── 9. 理解输出 ✓ (Claude擅长)
         ├── 10. 决策 ✓ (Claude擅长)
         │     风险低 → 执行
         └── 11. 报告 ✓ (Claude擅长)

专业逻辑 → 系统
任务理解、决策、沟通 → Claude
```

---

## 六、如何开发你的第一个专业系统

### 6.1 不要这样做（常见错误）

```yaml
❌ Week 1: 写SKILL.md文档
   内容: "代码review的最佳实践..."
   问题: Claude学不会专业知识

❌ Week 2: 优化prompt
   内容: "你是一个资深架构师..."
   问题: Prompt不能替代系统

❌ Week 3: 让Claude"学习"案例
   内容: 提供100个good/bad代码示例
   问题: LLM没有长期记忆

❌ Week 4: 失败
   结果: Claude仍然不能准确review代码
```

### 6.2 应该这样做（正确方法）

**Phase 1 (Month 1-3): 开发核心系统**

```yaml
目标: 建立实际的专业系统（软件代码）

Step 1: 选择领域
  - 从小做起（例如：代码质量检查）
  - 确保有客观标准

Step 2: 开发检查工具（实际的软件）

  2.1 集成现有工具:
      - ESLint
      - TypeScript
      - 测试框架

  2.2 开发自定义检测器:
      ```typescript
      // src/pattern-detector.ts
      export class PatternDetector {
        detect(code: string): Issue[] {
          const ast = parse(code);
          const issues: Issue[] = [];

          // 检测规则1: 过度嵌套
          if (getMaxNesting(ast) > 4) {
            issues.push({...});
          }

          // 检测规则2: 复杂函数
          if (getCyclomaticComplexity(ast) > 10) {
            issues.push({...});
          }

          return issues;
        }
      }
      ```

  2.3 创建CLI接口:
      ```bash
      # bin/check-quality
      #!/bin/bash
      node dist/cli.js check "$@"
      ```

      返回JSON:
      ```json
      {
        "passed": false,
        "issues": [
          {
            "file": "src/service.ts",
            "line": 42,
            "type": "complexity",
            "message": "Function too complex (CC=15)",
            "severity": "high"
          }
        ]
      }
      ```

Step 3: 创建Skill脚本

  ```bash
  # .claude/skills/code-quality/bin/check
  #!/bin/bash

  FILES=$1

  # 调用系统
  /path/to/check-quality $FILES
  ```

Step 4: 测试Claude集成

  - 让Claude调用skill
  - 验证能否理解JSON输出
  - 验证决策是否正确

产出:
  ✓ 可用的检查系统
  ✓ Claude能调用
  ✓ 准确率约60%（初期）
```

**Phase 2 (Month 4-6): 优化系统**

```yaml
目标: 提升准确率，降低误报

Step 1: 收集真实案例
  - 记录误报（False Positive）
  - 记录漏报（False Negative）

Step 2: 优化检测算法（修改系统代码，不是文档！）

  2.1 分析误报原因:
      - 规则太严格？
      - 缺少上下文？
      - 阈值不对？

  2.2 修改代码:
      ```typescript
      // 优化前
      if (complexity > 10) return issue;

      // 优化后
      if (complexity > 10 &&
          !isTestFile &&
          !hasComplexityJustification) {
        return issue;
      }
      ```

Step 3: 添加新规则
  - 从真实问题中提炼
  - 编码到系统中

Step 4: 更新CLI输出（如果需要）
  - 添加更多上下文
  - 改进JSON格式

产出:
  ✓ 准确率提升至 80%
  ✓ 误报率降至 20%
  ✓ Claude能处理大部分case
```

**Phase 3 (Month 6-12): 规模化**

```yaml
目标: 达到生产级质量

Step 1: 建立质量反馈循环
  - 自动收集误报
  - 持续优化算法

Step 2: 添加自动修复建议
  - 85%的问题能提供修复方案
  - Claude能自动应用（低风险）

Step 3: 扩展到其他领域
  - 性能检查
  - 安全检查
  - 依赖分析

Step 4: 多客户支持
  - 客户定制规则系统
  - 多租户架构

产出:
  ✓ 准确率 90%+
  ✓ 形式3真正work
  ✓ 可复制到其他客户
```

### 6.3 最小可行产品（MVP）

**2周MVP**：
```yaml
目标: 验证架构可行性

Week 1: 开发最简系统
  - 只做一件事（例如：检测函数复杂度）
  - 使用现有工具（ESLint）
  - 返回JSON

Week 2: Claude集成
  - 创建Skill脚本
  - 测试调用
  - 验证决策

成功标准:
  ✓ Claude能调用系统
  ✓ Claude能理解输出
  ✓ Claude能做简单决策

如果成功 → 继续投资
如果失败 → 重新设计架构
```

### 6.4 投资估算

**小型系统（单一功能）**：
```yaml
时间: 3-6个月
团队: 2-3人
投资: $100K-$200K

示例: 代码质量检查（仅检测，无修复）
```

**中型系统（多功能）**：
```yaml
时间: 6-12个月
团队: 3-5人
投资: $300K-$600K

示例: 完整的代码质量系统（检测+修复建议+架构验证）
```

**大型系统（多领域）**：
```yaml
时间: 12-24个月
团队: 5-10人
投资: $1M-$2M

示例: hvac-workbench（代码质量+HVAC优化+性能分析+...）
```

### 6.5 关键成功因素

```yaml
✓ 从小做起（MVP优先）
✓ 选择有客观标准的领域
✓ 投资开发系统（不是写文档）
✓ 建立快速反馈循环
✓ 持续优化算法
✓ 长期视角（6-12个月才能work）

✗ 不要期望"写文档让AI学会"
✗ 不要低估开发系统的复杂度
✗ 不要忽视误报率优化
✗ 不要短期思维（3个月就放弃）
```

---

## 七、对其他ADR的补充

### 7.1 对ADR 005-8的补充

**ADR 005-8说的**：
- 超敏捷开发 ✓
- 质量门禁 ✓
- 运营式开发 ✓

**本文档补充**：
- 质量门禁 = hvac-workbench系统（软件代码）
- 不是文字规则
- 需要$1M投资开发

### 7.2 对ADR 005-5的补充

**ADR 005-5说的**：
- 形式3需要$1.49M投资 ✓
- 其中$350K是"领域知识编码" ✓

**本文档澄清**：
- "领域知识编码" = 开发专业系统（软件）
- 不是"编写知识文档"
- 是开发hvac-optimizer系统（TypeScript代码）

### 7.3 对ADR 005主文档的补充

**主文档说的**：
- Workbench投资$1.49M ✓
- 包含"系统开发+知识编码" ✓

**本文档澄清**：
- 85%投资在开发软件系统
- 15%在运营和管理
- 0%在"写文档"

---

## 八、常见问题（FAQ）

### Q1: 我可以用详细的文档替代系统吗？

```yaml
答: 不能

原因:
  1. LLM的局限:
     - 无法做复杂算法（热力学计算）
     - 无法分析历史数据
     - 判断不稳定（同样输入，不同输出）

  2. 文档的局限:
     - 边界情况无法穷尽
     - 自然语言有歧义
     - 知识无法精确编码

  3. 可复用性差:
     - 每个新客户重写文档
     - 无法积累
     - 边际成本高

系统才能解决这些问题。
```

### Q2: 开发系统的成本太高，有替代方案吗？

```yaml
答: 有，但各有限制

方案1: 使用现有工具
  - 例如：ESLint, TypeScript, pytest
  - 成本: 低（$0-$10K）
  - 限制: 只能做通用检查，不能做领域特定分析

方案2: 购买第三方服务
  - 例如：SonarQube, CodeClimate
  - 成本: 中（$50K-$100K/年）
  - 限制: 不能定制，不适合专业领域（如HVAC）

方案3: 开发定制系统
  - 投资: 高（$100K-$1M）
  - 优势: 完全定制，可复用，长期ROI高
  - 适合: 形式3模式，多客户服务

选择方案1/2 → 形式1或形式2
选择方案3 → 形式3
```

### Q3: 我的领域没有hvac-workbench这样的系统，我该怎么办？

```yaml
答: 开发你的专业系统

步骤:
  1. MVP验证（2周）
     - 选择最小功能
     - 开发简单系统
     - 验证Claude能调用

  2. 迭代开发（3-6个月）
     - 扩展功能
     - 优化准确率
     - 降低误报

  3. 规模化（6-12个月）
     - 生产级质量
     - 多客户支持

或者:
  - 购买第三方工具（如果有）
  - 雇佣专家开发
  - 与技术公司合作

重点: 必须有系统，不能只靠文档
```

### Q4: Skills的SKILL.md到底有什么用？

```yaml
答: 给Claude的"工具说明书"

内容（简短）:
  - 这个skill能做什么
  - 有哪些命令
  - 如何调用
  - 输出格式

不是:
  ✗ 专业知识教学
  ✗ 详细操作指南
  ✗ 最佳实践文档

类比:
  iPhone说明书:
    ✓ 如何开机、充电、拍照
    ✗ 不教摄影技术
    ✗ 不讲芯片原理

SKILL.md:
  ✓ 如何调用工具
  ✗ 不教专业知识
  ✗ 不讲算法原理
```

### Q5: 形式3的技术门槛有多高？

```yaml
答: 很高

必要能力:
  1. 软件开发:
     - 能开发生产级系统
     - TypeScript/Python/Rust等

  2. DevOps:
     - CI/CD管道
     - 自动化测试
     - 监控系统

  3. 领域专业知识:
     - 代码架构（如果做code review）
     - HVAC工程（如果做HVAC优化）
     - 需要雇佣专家或合作

  4. AI工具使用:
     - Claude API
     - Skills开发
     - Prompt工程

  5. 系统思维:
     - 能把工作"运营化"
     - 长期视角

这就是为什么"AI时代的首席架构师"稀缺！

门槛高 → 竞争少 → Univers的护城河
```

---

## 九、总结

### 9.1 核心洞察

```yaml
Workbench成功的技术真相:

1. Skills ≠ 知识文档
   Skills = 可执行脚本 + 专业系统

2. 知识编码 ≠ 写文档
   知识编码 = 开发软件系统（代码）

3. $1M投资 ≠ 写文档
   $1M投资 = 开发hvac-workbench（TypeScript/Python代码）

4. Claude ≠ 学习专业知识的专家
   Claude = 会用专业工具的智能助手

5. 持续改进 ≠ 更新文档
   持续改进 = 优化系统算法（代码）
```

### 9.2 技术栈示例

```yaml
Workbench可能的技术栈:

语言:
  - TypeScript（质量检查）
  - Python（HVAC算法、数据分析）
  - Rust（性能关键部分）
  - Bash（Skills脚本）

框架/库:
  - AST解析: @babel/parser, typescript
  - 数据分析: pandas, numpy
  - 优化算法: scipy.optimize
  - CLI: commander, yargs

工具:
  - 包管理: pnpm
  - 测试: Jest, pytest
  - CI/CD: GitHub Actions
  - 监控: Prometheus, Grafana

系统:
  - Node.js runtime
  - Python runtime
  - Docker（容器化）
  - PostgreSQL（数据存储）
```

### 9.3 投资回报

```yaml
形式3的经济学:

初期（Year 0-1）:
  投资: $600K-$1M
  回报: 负（开发期）

中期（Year 2-3）:
  客户1-3: 收回成本
  客户4-10: 开始盈利

长期（Year 3+）:
  客户10+: 指数级盈利
  边际成本: ~$20K/客户/年
  扩展性: 可服务100+客户

传统方式 vs Workbench:
  10客户, 3年:
    传统: $12M
    Workbench: $2.09M
    节省: 83%

这就是形式3的威力！⭐⭐⭐
```

### 9.4 适用性判断

```yaml
你应该投资开发专业系统，如果:

✓ 有多个客户（摊薄成本）
✓ 长期业务（不是一次性项目）
✓ 有客观标准（可以系统化）
✓ 有资本支持（$500K+）
✓ 有技术团队（能开发系统）
✓ 长期视角（12个月+）

你应该用现有工具，如果:

✓ 单个客户
✓ 短期项目（<6个月）
✓ 预算有限（<$100K）
✓ 需要快速启动

不同选择 → 不同形式
  现有工具 → 形式1/2
  专业系统 → 形式3
```

---

## 相关文档

- [ADR 005: 生产关系转变分析（主文档）](../005-production-relations-transformation.md)
- [005-8: Workbench百万行代码库案例](./005-8-workbench-case-study.md)
- [005-5: 形式3批判性分析](./005-5-form3-critical-analysis.md)
- [005-3: 形式3分析 - 管理者+AI执行](./005-3-form3-manager-with-ai.md)

---

**最后更新**: 2025-11-06
**版本**: 1.0

**作者注**：本文档基于对Workbench项目的深入分析和对Skills架构的真实理解。感谢用户纠正关于Skills的误解，让我们看到了技术真相。

**致读者**：如果你读完本文后仍然认为"写好文档就能让AI学会专业知识"，请重读第一部分。技术真相是：必须开发专业系统（软件代码），不能只靠文档。这是$1M投资的真正去向，也是Workbench成功的技术基础。
