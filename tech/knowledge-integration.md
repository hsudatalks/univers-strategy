# 垂直知识集成技术方案

## 文档概述

本文档描述如何在 Univers 系统中技术性地集成和管理垂直领域知识。

**目标**：
1. 让 AI 能执行可编码知识
2. 让 AI 能学习情境知识
3. 让系统能捕获、沉淀、复用元层知识
4. 构建知识飞轮的技术基础设施

**非目标**：
- ❌ 让 AI 生成元层知识（AI 做不到）
- ❌ 完全自动化知识管理（需要人类判断）

---

## 一、系统架构

### 1.1 三层知识架构

```
┌─────────────────────────────────────────────┐
│           元层知识层 (Meta Layer)             │
│   - 洞察库 (Insight Repository)              │
│   - 缩放提示引擎 (Scale Reminder Engine)      │
│   - 人类主导 + AI 辅助                        │
└─────────────────────────────────────────────┘
                    ↓ 驱动
┌─────────────────────────────────────────────┐
│         情境知识层 (Contextual Layer)         │
│   - 经验模式库 (Pattern Library)             │
│   - 情境匹配引擎 (Context Matcher)           │
│   - 人机协作                                  │
└─────────────────────────────────────────────┘
                    ↓ 指导
┌─────────────────────────────────────────────┐
│        可编码知识层 (Codifiable Layer)        │
│   - 规则引擎 (Rule Engine)                   │
│   - 参数库 (Parameter Database)              │
│   - AI 主导执行                               │
└─────────────────────────────────────────────┘
```

### 1.2 核心组件

#### 组件 1：知识捕获系统 (Knowledge Capture)
**职责**：低摩擦地记录专家的工作流中产生的知识

**技术栈**：
- 前端：工作流中自然嵌入的表单、语音输入
- 后端：知识提取管道 (NLP + 结构化)
- 存储：多模态知识库 (向量数据库 + 关系数据库 + 图数据库)

---

#### 组件 2：知识执行系统 (Knowledge Execution)
**职责**：让 AI 执行已有的知识

**技术栈**：
- 可编码知识：规则引擎 (Drools / 自研)
- 情境知识：RAG (检索增强生成)
- 元层知识：提示词工程 + 引导式对话

---

#### 组件 3：知识演化系统 (Knowledge Evolution)
**职责**：知识的验证、更新、淘汰

**技术栈**：
- 效果追踪：A/B 测试框架
- 知识评分：贡献者声誉系统
- 自动淘汰：低质量知识降权/删除

---

#### 组件 4：缩放提示引擎 (Scale Reminder)
**职责**：提醒用户"被忽视的尺度"（参见 [architecture.md](architecture.md) 第 3 节）

**技术栈**：
- 尺度检测：LLM 分析决策的隐含尺度
- 提醒生成：对比多个尺度，生成提醒
- 插入时机：在用户做决策时主动介入

---

## 二、可编码知识管理

### 2.1 数据模型

```python
from dataclasses import dataclass
from typing import List, Dict, Any
from enum import Enum

class KnowledgeType(Enum):
    RULE = "rule"           # 规则
    FORMULA = "formula"     # 公式
    PROCEDURE = "procedure" # 流程
    STANDARD = "standard"   # 标准

@dataclass
class CodifiableKnowledge:
    """可编码知识的数据结构"""

    id: str
    type: KnowledgeType
    domain: str              # 例如："HVAC", "lighting", "security"
    subdomain: str           # 例如："cooling", "heating", "ventilation"

    # 知识内容
    name: str
    description: str
    content: Dict[str, Any]  # 结构化内容

    # 适用条件
    conditions: List[str]    # 例如：["outdoor_temp < 5", "humidity > 60%"]

    # 效果数据
    effectiveness: float     # 0-1，基于历史使用效果
    usage_count: int
    success_rate: float

    # 元数据
    source: str              # 来源：专家、文献、数据挖掘
    created_at: str
    updated_at: str
    verified: bool


# 示例：HVAC 参数优化规则
example_rule = CodifiableKnowledge(
    id="rule_hvac_001",
    type=KnowledgeType.RULE,
    domain="HVAC",
    subdomain="heating",

    name="低温场景节能规则",
    description="当室外温度低于 5°C 时，适当降低室内目标温度和湿度可节能 10-15%",
    content={
        "action": "adjust_targets",
        "parameters": {
            "indoor_temp_delta": -2,      # 降低 2°C
            "humidity_delta": -5,          # 降低 5%
        },
        "expected_saving": 0.12            # 预期节能 12%
    },

    conditions=[
        "outdoor_temp < 5",
        "heating_mode == True",
        "building_type in ['office', 'retail']"
    ],

    effectiveness=0.87,
    usage_count=234,
    success_rate=0.85,

    source="aggregated_from_100_customers",
    created_at="2024-11-15",
    updated_at="2025-01-20",
    verified=True
)
```

### 2.2 规则引擎

```python
class RuleEngine:
    """
    执行可编码知识的引擎
    """

    def __init__(self, knowledge_base: List[CodifiableKnowledge]):
        self.knowledge_base = knowledge_base
        self.index = self._build_index()

    def _build_index(self):
        """构建快速查询索引"""
        index = {}
        for k in self.knowledge_base:
            key = f"{k.domain}:{k.subdomain}"
            if key not in index:
                index[key] = []
            index[key].append(k)
        return index

    def find_applicable_knowledge(self, context: Dict[str, Any]) -> List[CodifiableKnowledge]:
        """
        找到适用于当前情境的知识

        Args:
            context: 当前情境，例如：
                {
                    "domain": "HVAC",
                    "subdomain": "heating",
                    "outdoor_temp": 3,
                    "heating_mode": True,
                    "building_type": "office"
                }

        Returns:
            适用的知识列表，按 effectiveness 排序
        """
        key = f"{context['domain']}:{context['subdomain']}"
        candidates = self.index.get(key, [])

        applicable = []
        for k in candidates:
            if self._check_conditions(k.conditions, context):
                applicable.append(k)

        # 按效果排序
        applicable.sort(key=lambda x: x.effectiveness, reverse=True)
        return applicable

    def _check_conditions(self, conditions: List[str], context: Dict[str, Any]) -> bool:
        """检查条件是否满足"""
        for cond in conditions:
            try:
                # 简单的表达式求值（生产环境需要更安全的实现）
                if not eval(cond, {"__builtins__": {}}, context):
                    return False
            except:
                return False
        return True

    def execute(self, knowledge: CodifiableKnowledge, context: Dict[str, Any]) -> Dict[str, Any]:
        """
        执行知识

        Returns:
            执行结果和推荐的行动
        """
        return {
            "knowledge_id": knowledge.id,
            "knowledge_name": knowledge.name,
            "action": knowledge.content["action"],
            "parameters": knowledge.content["parameters"],
            "expected_outcome": knowledge.content.get("expected_saving"),
            "confidence": knowledge.effectiveness,
            "explanation": f"{knowledge.description}（基于 {knowledge.usage_count} 次历史使用，成功率 {knowledge.success_rate:.1%}）"
        }


# 使用示例
if __name__ == "__main__":
    # 初始化规则引擎
    engine = RuleEngine([example_rule])

    # 当前情境
    context = {
        "domain": "HVAC",
        "subdomain": "heating",
        "outdoor_temp": 3,
        "heating_mode": True,
        "building_type": "office"
    }

    # 查找适用知识
    applicable = engine.find_applicable_knowledge(context)

    if applicable:
        # 执行最佳知识
        result = engine.execute(applicable[0], context)
        print(f"推荐行动：{result['knowledge_name']}")
        print(f"参数调整：{result['parameters']}")
        print(f"预期节能：{result['expected_outcome']:.1%}")
        print(f"置信度：{result['confidence']:.1%}")
```

### 2.3 知识更新机制

```python
class KnowledgeUpdater:
    """
    基于实际效果更新知识的有效性
    """

    def record_execution(self, knowledge_id: str, context: Dict, outcome: Dict):
        """
        记录知识的执行结果

        Args:
            knowledge_id: 知识 ID
            context: 执行时的情境
            outcome: 实际结果
                {
                    "success": True/False,
                    "actual_saving": 0.13,  # 实际节能 13%
                    "feedback": "效果很好",
                }
        """
        # 更新使用次数
        knowledge = self._get_knowledge(knowledge_id)
        knowledge.usage_count += 1

        # 更新成功率（指数移动平均）
        alpha = 0.1  # 平滑系数
        if outcome["success"]:
            knowledge.success_rate = (1 - alpha) * knowledge.success_rate + alpha * 1.0
        else:
            knowledge.success_rate = (1 - alpha) * knowledge.success_rate + alpha * 0.0

        # 更新有效性（考虑实际效果 vs 预期效果）
        if "actual_saving" in outcome and "expected_saving" in knowledge.content:
            expected = knowledge.content["expected_saving"]
            actual = outcome["actual_saving"]
            accuracy = min(actual / expected, 1.0) if expected > 0 else 0
            knowledge.effectiveness = (1 - alpha) * knowledge.effectiveness + alpha * accuracy

        # 持久化
        self._save_knowledge(knowledge)

    def prune_low_quality(self, threshold: float = 0.5):
        """淘汰低质量知识"""
        for k in self.knowledge_base:
            if k.usage_count > 10 and k.effectiveness < threshold:
                k.verified = False
                print(f"标记低质量知识：{k.name}（有效性：{k.effectiveness:.2f}）")
```

---

## 三、情境知识管理

### 3.1 数据模型

```python
@dataclass
class ContextualKnowledge:
    """情境知识的数据结构"""

    id: str
    domain: str

    # 知识内容
    title: str
    description: str               # 自然语言描述

    # 情境特征
    context_features: Dict[str, Any]  # 结构化特征
    context_embedding: List[float]    # 向量表示（用于相似度匹配）

    # 经验性知识
    diagnosis: str                    # 诊断
    action: str                       # 推荐行动
    reasoning: str                    # 推理过程

    # 来源与验证
    source_expert: str                # 来源专家
    validation_count: int             # 被验证次数
    validation_success_rate: float    # 验证成功率

    # 元数据
    created_at: str
    updated_at: str


# 示例：老师傅的经验
example_contextual = ContextualKnowledge(
    id="ctx_hvac_001",
    domain="HVAC",

    title="压缩机轴承磨损的早期征兆",
    description="当压缩机启动电流波动超过 5%，且运行声音频率偏移超过 2Hz，虽然温度控制仍然正常，但这通常是轴承磨损的早期征兆",

    context_features={
        "symptom_current_fluctuation": "> 5%",
        "symptom_sound_frequency_shift": "> 2Hz",
        "symptom_temp_control": "normal",
        "equipment_type": "compressor",
        "equipment_age": "> 3 years"
    },

    context_embedding=[0.23, -0.45, 0.67, ...],  # 768 维向量

    diagnosis="压缩机轴承磨损（早期）",
    action="建议 2 周内安排更换",
    reasoning="轴承磨损会导致转轴不稳定，表现为电流波动和声音异常，但短期内不影响温度控制。如果不及时处理，2-4 周后可能突然故障。",

    source_expert="张师傅（20 年经验）",
    validation_count=15,
    validation_success_rate=0.87,

    created_at="2024-08-10",
    updated_at="2025-01-15"
)
```

### 3.2 情境匹配引擎（RAG）

```python
from typing import List
import numpy as np

class ContextMatcher:
    """
    基于 RAG (Retrieval-Augmented Generation) 的情境知识匹配
    """

    def __init__(self, knowledge_base: List[ContextualKnowledge], embedding_model):
        self.knowledge_base = knowledge_base
        self.embedding_model = embedding_model

        # 构建向量索引（使用 FAISS / Pinecone / Qdrant）
        self.vector_index = self._build_vector_index()

    def _build_vector_index(self):
        """构建向量索引（简化示例，生产环境使用 FAISS）"""
        embeddings = [k.context_embedding for k in self.knowledge_base]
        return np.array(embeddings)

    def find_similar_contexts(self, current_situation: str, top_k: int = 5) -> List[ContextualKnowledge]:
        """
        找到与当前情境相似的历史情境

        Args:
            current_situation: 当前情境的自然语言描述
                例如："压缩机启动时电流波动 6%，声音频率偏移 3Hz，但温度控制正常"
            top_k: 返回最相似的 k 个

        Returns:
            相似情境的知识列表
        """
        # 1. 将当前情境编码为向量
        current_embedding = self.embedding_model.encode(current_situation)

        # 2. 计算相似度
        similarities = np.dot(self.vector_index, current_embedding)

        # 3. 返回 top-k
        top_indices = np.argsort(similarities)[-top_k:][::-1]

        results = []
        for idx in top_indices:
            k = self.knowledge_base[idx]
            results.append({
                "knowledge": k,
                "similarity": similarities[idx],
                "confidence": k.validation_success_rate
            })

        return results

    def generate_recommendation(self, current_situation: str, llm) -> Dict[str, Any]:
        """
        生成推荐（RAG 模式）

        Steps:
            1. Retrieval: 检索相似情境
            2. Augmentation: 构建增强提示词
            3. Generation: LLM 生成推荐
        """
        # 1. Retrieval
        similar = self.find_similar_contexts(current_situation, top_k=3)

        # 2. Augmentation
        prompt = f"""
你是一个 HVAC 领域的专家。现在遇到以下情况：

**当前情境**：
{current_situation}

**相似的历史案例**：

"""
        for i, item in enumerate(similar, 1):
            k = item["knowledge"]
            prompt += f"""
案例 {i}（相似度：{item["similarity"]:.2f}，可信度：{item["confidence"]:.2f}）：
- 情境：{k.description}
- 诊断：{k.diagnosis}
- 推荐行动：{k.action}
- 推理：{k.reasoning}
- 来源：{k.source_expert}

"""

        prompt += """
**任务**：
基于以上历史案例，对当前情境进行：
1. 诊断（最可能的问题是什么？）
2. 推荐行动（应该怎么做？）
3. 推理过程（为什么这样判断？）
4. 置信度（0-100%）
"""

        # 3. Generation
        response = llm.generate(prompt)

        return {
            "diagnosis": response["diagnosis"],
            "action": response["action"],
            "reasoning": response["reasoning"],
            "confidence": response["confidence"],
            "similar_cases": [item["knowledge"].id for item in similar]
        }
```

### 3.3 经验沉淀机制

```python
class ExperienceCapture:
    """
    在工作流中捕获专家的情境知识
    """

    def capture_from_interaction(self, interaction: Dict) -> ContextualKnowledge:
        """
        从专家与系统的交互中提取情境知识

        Args:
            interaction: 交互记录
                {
                    "situation": "压缩机启动时...",
                    "expert_diagnosis": "轴承磨损",
                    "expert_action": "建议更换",
                    "expert_reasoning": "因为...",
                    "outcome": {"success": True, ...}
                }
        """
        # 1. 自然语言 → 结构化特征
        features = self._extract_features(interaction["situation"])

        # 2. 生成向量表示
        embedding = self.embedding_model.encode(interaction["situation"])

        # 3. 创建情境知识
        knowledge = ContextualKnowledge(
            id=self._generate_id(),
            domain="HVAC",
            title=self._generate_title(interaction),
            description=interaction["situation"],
            context_features=features,
            context_embedding=embedding,
            diagnosis=interaction["expert_diagnosis"],
            action=interaction["expert_action"],
            reasoning=interaction["expert_reasoning"],
            source_expert=interaction["expert_id"],
            validation_count=1,
            validation_success_rate=1.0 if interaction["outcome"]["success"] else 0.0,
            created_at=datetime.now().isoformat(),
            updated_at=datetime.now().isoformat()
        )

        return knowledge

    def _extract_features(self, situation_text: str) -> Dict:
        """使用 NLP 提取结构化特征"""
        # 使用 LLM 进行信息抽取
        prompt = f"""
从以下描述中提取关键特征：

{situation_text}

请以 JSON 格式返回：
{{
    "equipment_type": "...",
    "symptoms": ["...", "..."],
    "parameters": {{"key": "value"}},
    ...
}}
"""
        return llm_extract(prompt)
```

---

## 四、元层知识管理

### 4.1 数据模型

```python
@dataclass
class MetaKnowledge:
    """元层知识（洞察）的数据结构"""

    id: str

    # 洞察内容
    title: str
    insight: str                      # 核心洞察
    evidence: List[str]               # 支撑证据
    implications: List[str]           # 战略含义

    # 影响范围
    scales_affected: Dict[str, str]   # 影响的尺度
        # {
        #     "abstraction": "从'如何优化参数'到'是否应该优化参数'",
        #     "time": "从'短期节能'到'长期设备寿命'",
        #     "scope": "从'单个客户'到'整个行业'"
        # }

    # 价值数据
    value_created: float              # 创造的商业价值（$）
    usage_count: int                  # 被使用次数
    customer_impact: List[str]        # 影响的客户 ID

    # 来源
    source_expert: str
    source_data: List[str]            # 支撑数据来源
    created_at: str

    # 状态
    status: str                       # "draft", "validated", "published"
    validation_notes: List[str]


# 示例：元层洞察
example_meta = MetaKnowledge(
    id="meta_hvac_001",

    title="HVAC 行业的真正痛点是故障预测，而非能效优化",

    insight="""
行业普遍认为 HVAC 的核心问题是能效，但通过 200 个客户访谈发现：
1. 能耗成本占总成本 < 15%
2. 故障导致的停机损失 > 能耗成本 3 倍
3. 客户愿意为"零故障"支付 20% 溢价

因此，应该重新定位产品，从"能效优化"转向"故障预测"。
""",

    evidence=[
        "200 个客户访谈数据",
        "50 个客户的成本结构分析",
        "3 个标杆客户的案例研究",
        "行业报告：故障停机年均损失 $XX"
    ],

    implications=[
        "产品定位：从'省钱'到'避免损失'",
        "销售话术：强调'零故障'而非'节能'",
        "研发重点：投入故障预测算法",
        "定价策略：按'避免的损失'收费，而非按设备数"
    ],

    scales_affected={
        "abstraction": "从'如何优化能耗'到'什么是真正的痛点'",
        "time": "从'短期节能'到'长期业务战略'",
        "scope": "从'单个客户'到'整个行业'"
    },

    value_created=10_000_000.0,  # $10M
    usage_count=23,
    customer_impact=["customer_001", "customer_005", ...],

    source_expert="李工（15 年行业经验）",
    source_data=[
        "interview_data_2025q1.csv",
        "cost_analysis_50_customers.xlsx",
        "case_study_customer_A.pdf"
    ],
    created_at="2025-01-15",

    status="validated",
    validation_notes=[
        "2025-02-01: 在客户 A 验证，合同金额增加 40%",
        "2025-02-15: 在客户 B 验证，续约率提升",
        "2025-03-01: 发布到平台"
    ]
)
```

### 4.2 洞察捕获系统

```python
class InsightCapture:
    """
    捕获专家的元层洞察
    """

    def initiate_capture(self, trigger: str) -> Dict:
        """
        启动洞察捕获流程

        Args:
            trigger: 触发场景
                - "expert_question": 专家提出了好问题
                - "pattern_discovery": 发现了反复出现的模式
                - "client_feedback": 客户反馈暴露了误解
        """
        # 展示引导式表单
        return {
            "questions": [
                "你刚才的洞察是什么？（一句话）",
                "为什么这很重要？",
                "有什么证据支持？",
                "这个洞察会如何改变我们的策略？",
                "这影响哪些尺度？（时间/空间/抽象）"
            ],
            "examples": [example_meta],  # 展示示例
            "assistance": "AI 会帮你整理和结构化你的想法"
        }

    def process_expert_input(self, raw_input: Dict) -> MetaKnowledge:
        """
        处理专家输入，生成结构化洞察
        """
        # 1. 使用 LLM 帮助结构化
        prompt = f"""
一位领域专家提出了以下洞察：

{raw_input["insight_text"]}

请帮助结构化这个洞察：
1. 提炼核心观点（1-2 句）
2. 识别支撑证据
3. 推断战略含义
4. 分析影响的尺度

以 JSON 格式返回。
"""
        structured = llm_structure(prompt)

        # 2. 创建元层知识对象
        meta = MetaKnowledge(
            id=self._generate_id(),
            title=structured["title"],
            insight=structured["core_insight"],
            evidence=structured["evidence"],
            implications=structured["implications"],
            scales_affected=structured["scales"],
            value_created=0.0,  # 初始为 0，后续跟踪
            usage_count=0,
            customer_impact=[],
            source_expert=raw_input["expert_id"],
            source_data=raw_input.get("data_sources", []),
            created_at=datetime.now().isoformat(),
            status="draft",
            validation_notes=[]
        )

        return meta

    def suggest_insights(self, context: Dict) -> List[str]:
        """
        AI 主动建议："你刚才的思考可能是一个洞察"
        """
        # 检测元层思考的信号
        signals = [
            "质疑框架"：专家说了"也许我们问错了问题"
            "尺度切换"：专家从细节跳到了原则
            "模式识别"：专家说了"我注意到一个规律"
        ]

        if self._detect_meta_thinking(context):
            return [
                "你刚才提出的观点可能是一个重要洞察！",
                "要不要花 2 分钟记录下来？",
                "这可能帮助到其他客户，也能为你带来收益。"
            ]
```

### 4.3 洞察执行系统

```python
class InsightExecutor:
    """
    让 AI 执行元层洞察
    """

    def apply_insight(self, insight: MetaKnowledge, context: Dict) -> Dict:
        """
        将洞察应用到具体决策中

        Args:
            insight: 元层洞察
            context: 当前决策情境

        Returns:
            修改后的决策建议
        """
        # 构建提示词
        prompt = f"""
你正在帮助客户做一个关于 {context["decision_topic"]} 的决策。

但是，有一个重要的洞察需要考虑：

**洞察**：{insight.title}
{insight.insight}

**战略含义**：
{chr(10).join('- ' + imp for imp in insight.implications)}

**任务**：
基于这个洞察，重新审视当前决策：
1. 当前决策是否对齐这个洞察？
2. 是否应该调整决策？
3. 如果调整，如何调整？

当前决策草案：
{context["draft_decision"]}

请提供：
1. 洞察相关性（是否适用于当前决策）
2. 调整建议
3. 调整理由
"""

        response = llm_execute(prompt)

        return {
            "insight_id": insight.id,
            "relevance": response["relevance"],
            "adjustment": response["adjustment"],
            "reasoning": response["reasoning"]
        }

    def execute_with_insights(self, decision_context: Dict, insight_library: List[MetaKnowledge]) -> Dict:
        """
        在所有相关洞察的指导下执行决策
        """
        # 1. 找到相关洞察
        relevant_insights = self._find_relevant_insights(decision_context, insight_library)

        # 2. 对每个洞察，生成调整建议
        adjustments = []
        for insight in relevant_insights:
            adj = self.apply_insight(insight, decision_context)
            if adj["relevance"] > 0.7:  # 高相关性
                adjustments.append(adj)

        # 3. 综合调整
        final_decision = self._synthesize_adjustments(
            original=decision_context["draft_decision"],
            adjustments=adjustments
        )

        return {
            "original_decision": decision_context["draft_decision"],
            "final_decision": final_decision,
            "applied_insights": [adj["insight_id"] for adj in adjustments],
            "reasoning": self._generate_explanation(adjustments)
        }
```

---

## 五、缩放提示引擎

### 5.1 尺度检测

```python
class ScaleDetector:
    """
    检测决策中隐含的尺度
    """

    def detect_scales(self, decision_text: str) -> Dict[str, str]:
        """
        检测决策文本中隐含的尺度

        Returns:
            {
                "time": "short_term",       # short_term / medium_term / long_term
                "scope": "individual",      # individual / team / company / industry
                "abstraction": "detail",    # detail / rule / principle / philosophy
                "stakeholder": "user"       # user / team / business / society
            }
        """
        prompt = f"""
分析以下决策中隐含的思考尺度：

决策：{decision_text}

请识别：
1. 时间尺度：关注的是短期（< 1 月）、中期（1-12 月）、长期（> 1 年）？
2. 空间范围：关注的是个体、团队、公司、行业、社会？
3. 抽象层级：在讨论细节、规则、原则、还是哲学？
4. 利益相关方：主要考虑用户、团队、业务、还是社会？

以 JSON 格式返回。
"""
        return llm_analyze(prompt)


class ScaleReminderEngine:
    """
    缩放提示引擎：提醒被忽视的尺度
    （详见 architecture.md 第 3.2 节）
    """

    def __init__(self, detector: ScaleDetector):
        self.detector = detector

    def generate_reminders(self, decision: Dict) -> List[Dict]:
        """
        生成尺度提醒

        Args:
            decision: {
                "text": "我们应该优化这个参数来省电",
                "context": {...}
            }

        Returns:
            提醒列表
        """
        # 1. 检测当前尺度
        current_scales = self.detector.detect_scales(decision["text"])

        # 2. 生成其他尺度的提醒
        reminders = []

        # 时间尺度提醒
        if current_scales["time"] == "short_term":
            reminders.append({
                "type": "time_scale",
                "current": "短期（省电）",
                "reminder": "长期尺度",
                "message": "优化这个参数短期能省电 10%，但长期可能导致设备磨损加快，3 年后维护成本增加 30%。是否考虑长期权衡？"
            })

        # 抽象层级提醒
        if current_scales["abstraction"] == "detail":
            reminders.append({
                "type": "abstraction_scale",
                "current": "细节（参数优化）",
                "reminder": "原则层面",
                "message": "在细节层面，我们在优化参数；但在原则层面，也许应该问：'客户真正在乎的是省电，还是舒适度？'切换到原则层面思考可能发现不同的解决方案。"
            })

        # 利益相关方提醒
        if current_scales["stakeholder"] == "user":
            reminders.append({
                "type": "stakeholder_scale",
                "current": "用户视角",
                "reminder": "业务视角",
                "message": "从用户视角，省电是好的；但从业务视角，这可能降低我们的耗材销售收入。是否需要平衡？"
            })

        return reminders

    def present_reminders(self, reminders: List[Dict], ui_context: str) -> Dict:
        """
        决定何时、如何展示提醒

        Args:
            ui_context: "before_decision" / "after_decision" / "review"

        Returns:
            展示策略
        """
        if ui_context == "before_decision":
            # 决策前：主动介入，但不要打断
            return {
                "timing": "show_as_suggestion",
                "style": "gentle",
                "message": "💡 在做最终决定前，也许可以从其他尺度考虑一下？"
            }
        elif ui_context == "after_decision":
            # 决策后：作为反思
            return {
                "timing": "show_in_review",
                "style": "reflective",
                "message": "回顾一下，如果从这些尺度思考，会有不同的决策吗？"
            }
```

### 5.2 集成到工作流

```python
# 工作流示例
class DecisionWorkflow:
    """
    集成缩放提示的决策工作流
    """

    def __init__(self):
        self.scale_reminder = ScaleReminderEngine(ScaleDetector())
        self.insight_executor = InsightExecutor()

    def run(self, user_input: Dict):
        """
        执行决策工作流
        """
        # 1. 用户输入决策草案
        draft_decision = user_input["decision"]

        # 2. 生成尺度提醒
        reminders = self.scale_reminder.generate_reminders({
            "text": draft_decision,
            "context": user_input.get("context", {})
        })

        # 3. 展示提醒，让用户选择是否切换尺度
        if reminders:
            print("💡 也许可以从其他尺度考虑：")
            for r in reminders:
                print(f"   - {r['message']}")

            user_choice = input("要切换尺度重新思考吗？(y/n)")

            if user_choice == 'y':
                # 用户选择切换尺度，AI 辅助重新思考
                refined_decision = self._rethink_at_scale(draft_decision, reminders)
            else:
                refined_decision = draft_decision
        else:
            refined_decision = draft_decision

        # 4. 应用元层洞察
        final_decision = self.insight_executor.execute_with_insights(
            decision_context={"draft_decision": refined_decision},
            insight_library=self._load_insights()
        )

        return final_decision
```

---

## 六、知识飞轮技术实现

### 6.1 知识贡献追踪

```python
class KnowledgeContributionTracker:
    """
    追踪知识的贡献和价值创造
    """

    def track_usage(self, knowledge_id: str, user_id: str, outcome: Dict):
        """
        追踪知识使用和效果
        """
        event = {
            "knowledge_id": knowledge_id,
            "user_id": user_id,
            "timestamp": datetime.now().isoformat(),
            "outcome": outcome,  # {"success": True, "value_created": 5000}
        }

        self._log_event(event)

        # 更新知识的价值统计
        self._update_knowledge_value(knowledge_id, outcome.get("value_created", 0))

    def calculate_contributor_revenue(self, contributor_id: str, period: str) -> float:
        """
        计算贡献者在指定周期的收益
        """
        # 查询该贡献者的知识被使用情况
        contributions = self._get_contributions(contributor_id)

        total_revenue = 0.0
        for k in contributions:
            # 知识创造的价值
            value_created = k.value_created

            # 分成比例（例如 30%）
            revenue_share = 0.30

            # 贡献者收益
            contributor_revenue = value_created * revenue_share
            total_revenue += contributor_revenue

        return total_revenue
```

### 6.2 知识推荐系统

```python
class KnowledgeRecommender:
    """
    向用户推荐相关知识
    """

    def recommend(self, user_context: Dict, top_k: int = 3) -> List[Dict]:
        """
        基于用户当前情境推荐知识
        """
        # 1. 理解用户情境
        context_embedding = self._embed_context(user_context)

        # 2. 检索相关知识（可编码 + 情境 + 元层）
        candidates = []

        # 可编码知识
        codifiable = self._search_codifiable(user_context)
        candidates.extend([{"type": "codifiable", "knowledge": k} for k in codifiable])

        # 情境知识
        contextual = self._search_contextual(context_embedding)
        candidates.extend([{"type": "contextual", "knowledge": k} for k in contextual])

        # 元层知识
        meta = self._search_meta(user_context)
        candidates.extend([{"type": "meta", "knowledge": k} for k in meta])

        # 3. 排序（按相关性 × 质量 × 新鲜度）
        ranked = self._rank_candidates(candidates, user_context)

        return ranked[:top_k]

    def _rank_candidates(self, candidates, context):
        """综合排序"""
        for c in candidates:
            k = c["knowledge"]
            score = (
                0.4 * self._relevance_score(k, context) +
                0.3 * self._quality_score(k) +
                0.2 * self._freshness_score(k) +
                0.1 * self._diversity_score(k, candidates)
            )
            c["score"] = score

        return sorted(candidates, key=lambda x: x["score"], reverse=True)
```

---

## 七、部署架构

```
┌──────────────────────────────────────────────────────┐
│                     Web UI (React)                    │
│  - Workbench (用户工作台)                              │
│  - 知识捕获表单                                         │
│  - 缩放提示展示                                         │
└───────────────────────┬──────────────────────────────┘
                        │ API
┌───────────────────────┴──────────────────────────────┐
│              Backend Services (Python)                │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ Rule Engine │  │ RAG Pipeline │  │ Insight Mgr │ │
│  └─────────────┘  └──────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐ │
│  │ Scale       │  │ Knowledge    │  │ Contribution│ │
│  │ Reminder    │  │ Recommender  │  │ Tracker     │ │
│  └─────────────┘  └──────────────┘  └─────────────┘ │
└───────────────────────┬──────────────────────────────┘
                        │
┌───────────────────────┴──────────────────────────────┐
│                  Data Layer                           │
│  ┌──────────────┐  ┌───────────────┐  ┌───────────┐ │
│  │ PostgreSQL   │  │ Vector DB     │  │ Graph DB  │ │
│  │ (关系数据)    │  │ (Qdrant/      │  │ (Neo4j)   │ │
│  │              │  │  Pinecone)    │  │ (知识图谱) │ │
│  └──────────────┘  └───────────────┘  └───────────┘ │
└──────────────────────────────────────────────────────┘
                        │
┌───────────────────────┴──────────────────────────────┐
│                  LLM Services                         │
│  ┌──────────────┐  ┌───────────────┐                 │
│  │ GPT-4 / Claude│ │ Embedding    │                  │
│  │ (推理/生成)    │  │ Model        │                  │
│  └──────────────┘  └───────────────┘                 │
└──────────────────────────────────────────────────────┘
```

---

## 八、性能优化

### 8.1 缓存策略
```python
# 热门知识缓存
cache = {
    "codifiable_rules": TTLCache(maxsize=1000, ttl=3600),
    "contextual_patterns": TTLCache(maxsize=500, ttl=1800),
    "meta_insights": TTLCache(maxsize=100, ttl=86400)
}
```

### 8.2 异步处理
```python
# 知识提取异步化
async def extract_knowledge_async(interaction):
    task = asyncio.create_task(
        process_interaction(interaction)
    )
    # 不阻塞用户，后台处理
```

---

## 九、监控与评估

### 9.1 知识质量指标
```yaml
可编码知识:
  - 执行成功率 > 85%
  - 平均效果偏差 < 15%

情境知识:
  - 验证成功率 > 75%
  - 使用频率 > 5 次/月

元层知识:
  - 价值创造 > $10,000/洞察
  - 复用次数 > 10
```

### 9.2 系统性能指标
```yaml
响应时间:
  - 规则查询 < 100ms
  - 情境匹配 < 500ms
  - 元层推理 < 2s

准确性:
  - 规则适用性 > 95%
  - 情境匹配相关性 > 80%
  - 洞察推荐接受率 > 60%
```

---

## 十、总结

### 核心技术栈
- **可编码知识**：规则引擎 + PostgreSQL
- **情境知识**：RAG (Embedding + Vector DB + LLM)
- **元层知识**：结构化存储 + 提示词工程 + 引导式捕获

### 关键创新
1. **三层知识架构**：区分可编码/情境/元层
2. **低摩擦捕获**：在工作流中自然沉淀
3. **缩放提示引擎**：主动提醒被忽视的尺度
4. **知识飞轮**：贡献-使用-收益-激励

### 下一步
1. 实现 MVP（最小可行产品）
2. 在标杆客户验证
3. 迭代优化

### 相关文档
- [产品体验](../product/domain-expert-ux.md) - 用户如何使用这些技术
- [业务战略](../business/knowledge-strategy.md) - 为什么这样设计
- [决策记录](../decisions/002-vertical-knowledge-management.md) - 技术选型的背景
