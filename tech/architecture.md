# 技术架构

> 本文档描述 Univers 平台的技术架构细节。关于架构决策的背景和原因，请参考 [三层架构设计决策](../decisions/001-三层架构设计.md)。

## 架构概览

Univers 采用三层架构，实现人机协作的 AIoT 平台：

```
┌────────────────────────────────────────────────────────┐
│                    Workbench 层                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ 用户界面 │  │ AI协作台 │  │ 设备管理 │  │ 数据层 │ │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘ │
└─────────────────────┬──────────────────────────────────┘
                      │ REST API / GraphQL
                      ↓
┌────────────────────────────────────────────────────────┐
│                     Skills 层                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐ │
│  │ 设备控制 │  │ 数据查询 │  │ 告警通知 │  │ 规则引擎│ │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘ │
└─────────────────────┬──────────────────────────────────┘
                      │ Skills API (Python/HTTP)
                      ↓
┌────────────────────────────────────────────────────────┐
│                   Operation 层                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              hvac-operation                       │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │  │
│  │  │ AI Agent │  │ 优化引擎 │  │ 故障诊断 │      │  │
│  │  └──────────┘  └──────────┘  └──────────┘      │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │           lighting-operation (未来)               │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

## Workbench 层

### 定位
通用 AIoT 基础设施平台 + 人机协作工作台

### 核心模块

#### 1. 设备管理 (Device Management)
**功能**：
- 设备注册和认证
- 协议适配（MQTT、HTTP、CoAP、Modbus 等）
- 设备状态监控
- 设备命令下发

**技术选型**（待定）：
- IoT 网关：EMQ X / Eclipse Hono
- 设备注册：PostgreSQL
- 实时连接：MQTT Broker

#### 2. 数据层 (Data Layer)
**功能**：
- 时序数据存储
- 实时数据流处理
- 历史数据查询
- 数据聚合和统计

**技术选型**（待定）：
- 时序数据库：InfluxDB / TimescaleDB
- 流处理：Apache Kafka / Redis Streams
- 缓存：Redis
- 分析：ClickHouse

#### 3. AI 协作台 (AI Collaboration)
**功能**：
- 目标设定界面（让用户定义优化目标）
- 策略审批流程（AI 建议 → 人类审批）
- 实时监控大屏（展示 AI 运行状态）
- 异常告警和干预（人类可随时介入）

**技术选型**（待定）：
- 前端：React / Vue.js
- 后端：FastAPI / Django
- 实时通信：WebSocket

#### 4. 用户和权限 (IAM)
**功能**：
- 多租户管理
- 角色和权限控制
- API 认证和授权

**技术选型**（待定）：
- 认证：OAuth 2.0 / JWT
- 权限：RBAC / ABAC
- 存储：PostgreSQL

### 对外接口
- **REST API**：标准的 CRUD 操作
- **GraphQL**（可选）：灵活的数据查询
- **WebSocket**：实时数据推送

## Skills 层

### 定位
AI 与 workbench 的连接桥梁，提供标准化的能力封装

### Skills 设计原则

1. **简单清晰**：API 命名直观，AI 容易理解
2. **自描述**：每个 skill 有清晰的文档和示例
3. **版本管理**：skills 可独立升级，向后兼容
4. **错误处理**：明确的错误码和提示

### Skills 示例

#### Skill: 设备控制
```python
# 文件：skills/device.py

class DeviceSkill:
    """控制 IoT 设备的能力"""

    def control(self, device_id: str, action: str, params: dict) -> dict:
        """
        控制设备

        Args:
            device_id: 设备 ID
            action: 操作类型 (如 "set_temp", "turn_on", "turn_off")
            params: 操作参数

        Returns:
            {"success": True, "message": "操作成功"}

        Example:
            skill.device.control(
                device_id="hvac-3f-01",
                action="set_temp",
                params={"value": 22, "mode": "cool"}
            )
        """
        # 调用 workbench API
        return workbench_api.send_command(device_id, action, params)
```

#### Skill: 数据查询
```python
class DataSkill:
    """查询时序数据的能力"""

    def query(self, sensor_id: str, time_range: str, aggregation: str = "avg") -> list:
        """
        查询传感器数据

        Args:
            sensor_id: 传感器 ID
            time_range: 时间范围 ("last_1h", "last_24h", "last_7d")
            aggregation: 聚合方式 ("avg", "min", "max", "sum")

        Returns:
            [{"timestamp": "2025-11-02T10:00:00", "value": 22.5}, ...]
        """
        return workbench_api.query_timeseries(sensor_id, time_range, aggregation)
```

### Skills 发现机制

AI 如何知道有哪些 skills？

1. **Skills 注册表**：所有 skills 注册到中心目录
2. **自动文档生成**：从代码生成 API 文档
3. **AI 训练**：让 AI 学习 skills 的使用方法

## Operation 层

### 定位
行业特定的业务逻辑和 AI 自动化执行

### hvac-operation 架构

```
hvac-operation/
├── agents/              # AI 智能体
│   ├── optimizer.py     # 能效优化 agent
│   ├── diagnostics.py   # 故障诊断 agent
│   └── scheduler.py     # 调度优化 agent
├── rules/               # 业务规则
│   ├── comfort.py       # 舒适度规则
│   ├── energy.py        # 节能规则
│   └── safety.py        # 安全规则
├── models/              # AI 模型
│   ├── load_prediction.py   # 负荷预测
│   └── fault_detection.py   # 故障检测
└── skills_client/       # Skills 调用封装
    └── client.py
```

### 典型工作流

**场景：AI 自动优化温度**

1. **感知**：optimizer agent 通过 `skills.data.query()` 获取温度、湿度、人流数据
2. **决策**：基于规则和模型，计算最优温度设定
3. **检查**：调用 `skills.rule.check()` 验证是否符合约束（舒适度、安全等）
4. **执行**：
   - 如果在自主权限内 → 直接调用 `skills.device.control()`
   - 如果需要审批 → 调用 `skills.approval.request()`，等待 workbench 上人类批准
5. **反馈**：调用 `skills.alert.send()` 通知结果

### 自主程度配置

Operation 的自主程度是**可配置**的：

```python
# 配置示例
autonomy_config = {
    "temperature_adjustment": {
        "auto_range": (20, 26),      # 这个范围内可自动调整
        "approval_required": (15, 30), # 超出需要审批
        "forbidden": (0, 15) + (30, 50)  # 禁止
    },
    "device_control": {
        "auto_allowed": ["set_temp", "set_mode"],
        "approval_required": ["reset", "maintenance_mode"],
        "forbidden": ["factory_reset"]
    }
}
```

## 数据流

### 典型场景：温度优化

```
1. 传感器 → Workbench (MQTT)
   温度数据: 3楼 25°C

2. Workbench → 时序数据库
   存储历史数据

3. hvac-operation → Skills → Workbench
   查询: "过去1小时3楼温度"

4. Skills → hvac-operation
   返回: [24.5, 24.8, 25.0, 25.2, 25.0]

5. hvac-operation (AI 决策)
   判断: 温度上升趋势，建议调低到23°C

6. hvac-operation → Skills → Workbench
   检查: 是否在自主权限内？

7. Skills → hvac-operation
   返回: 是，可自动执行

8. hvac-operation → Skills → Workbench
   执行: 设置3楼温度为23°C

9. Workbench → 设备
   下发命令 (MQTT)

10. Workbench → AI 协作台
    记录日志: "AI 自动调整 3F 温度 25°C→23°C"
```

## 技术栈（初步规划）

### Workbench
- **后端**：Python (FastAPI) / Go
- **前端**：React / Vue.js
- **数据库**：PostgreSQL + InfluxDB
- **消息队列**：Kafka / RabbitMQ
- **IoT 协议**：MQTT (EMQ X)

### Skills
- **语言**：Python（与 AI 集成友好）
- **框架**：自研轻量级框架
- **文档**：OpenAPI 规范

### Operation
- **语言**：Python
- **AI 框架**：LangChain / AutoGen
- **模型**：Claude API / 自训练模型

## 部署架构

### 开发环境
- Docker Compose：一键启动所有服务
- 本地开发：各层独立运行

### 生产环境（待定）
- **Workbench**：Kubernetes 集群
- **Operation**：独立容器，可水平扩展
- **边缘部署**：部分 operation 可部署到边缘设备

## 安全考虑

1. **API 认证**：所有 API 需要认证
2. **权限控制**：operation 只能访问授权的 skills
3. **审计日志**：所有 AI 决策和操作记录
4. **数据加密**：传输和存储加密
5. **设备安全**：设备证书和密钥管理

## 监控和运维

1. **系统监控**：Prometheus + Grafana
2. **日志收集**：ELK Stack
3. **链路追踪**：Jaeger / Zipkin
4. **告警**：AlertManager

## 弥补 AI 的缩放局限

> 核心理念：AI 难以主动缩放尺度和质疑框架（见 [PHILOSOPHY.md](../PHILOSOPHY.md)），我们通过**系统设计**来弥补这个局限。

### 挑战

AI（特别是 LLM）的缩放能力有根本限制：
1. **固定上下文窗口**：难以处理超大尺度的信息
2. **被动处理**：不能主动选择思考尺度
3. **难以元推理**：很难跳出训练框架质疑问题本身
4. **缺乏驱动力**：没有真实目标驱动缩放的必要性

### 我们的技术策略

#### 策略 1：多尺度视图系统

**不等AI自己缩放，系统主动展示多个尺度**

**Workbench 实现**：
```python
# 多尺度数据聚合服务
class MultiScaleDataService:
    """
    同时维护多个时间尺度的数据视图
    """
    def get_multi_scale_view(self, entity_id: str):
        return {
            "micro": self.get_realtime(entity_id),      # 实时（秒级）
            "short": self.get_hourly(entity_id),         # 短期（小时）
            "medium": self.get_daily(entity_id),         # 中期（天）
            "long": self.get_monthly(entity_id),         # 长期（月）
            "strategic": self.get_yearly(entity_id)      # 战略（年）
        }

# 前端自动切换尺度
class ScaleAwareComponent:
    """
    根据用户任务自动选择合适尺度
    """
    def render(self, task_type):
        if task_type == "monitor":
            return self.render_realtime_view()
        elif task_type == "analyze":
            return self.render_daily_view()
        elif task_type == "plan":
            return self.render_monthly_view()
```

**Skills层提供缩放能力**：
```python
class ScaleSkill:
    """
    专门的缩放 skill
    """
    def zoom_out(self, current_view, levels=1):
        """
        放大视野（更高尺度）
        例：从"今天"到"本周"到"本月"
        """
        pass

    def zoom_in(self, current_view, levels=1):
        """
        缩小视野（更低尺度）
        例：从"本月"到"本周"到"今天"
        """
        pass

    def compare_scales(self, entity, scales):
        """
        并排对比不同尺度的视图
        """
        pass
```

#### 策略 2：缩放提示引擎

**AI在单一尺度优化时，系统主动提醒其他尺度**

**实现方案**：
```python
class ScaleReminderEngine:
    """
    监控AI的决策，提醒被忽视的尺度
    """
    def analyze_decision(self, ai_decision):
        """
        分析AI决策在哪个尺度上
        提醒其他尺度的风险
        """
        current_scale = self.detect_scale(ai_decision)

        reminders = []

        # 时间尺度提醒
        if current_scale.time == "short_term":
            reminders.append({
                "scale": "long_term",
                "message": "这个决策短期省电20%，但长期可能影响设备寿命"
            })

        # 空间尺度提醒
        if current_scale.space == "single_device":
            reminders.append({
                "scale": "system_wide",
                "message": "单个设备优化了，但可能影响整体平衡"
            })

        # 抽象层级提醒
        if current_scale.abstraction == "operational":
            reminders.append({
                "scale": "strategic",
                "message": "这符合'用户价值优先'的战略原则吗？"
            })

        return reminders
```

**展示给人类**：
```
AI建议：降低3F温度到20°C，节能15%

⚠️ 系统提醒：
┌─────────────────────────────────────┐
│ 从其他尺度看看？                    │
├─────────────────────────────────────┤
│ [短期] ✓ 本周节省¥500               │
│ [长期] ⚠️ 本年投诉可能增加15%       │
│ [用户] ⚠️ 3F有孕妇，温度敏感        │
│ [战略] ❓ 是否符合"舒适优先"原则？  │
└─────────────────────────────────────┘
```

#### 策略 3：元层控制系统

**人类可以轻松切换到元层，质疑框架**

**Skills 设计**：
```python
class MetaSkill:
    """
    元层能力：质疑和重构框架
    """
    def question_goal(self, current_goal):
        """
        质疑当前目标
        """
        return {
            "goal": current_goal,
            "questions": [
                "这个目标本身合理吗？",
                "我们是否在优化错误的指标？",
                "有没有更高层的目标？"
            ],
            "alternatives": self.suggest_alternatives(current_goal)
        }

    def reframe_problem(self, problem):
        """
        重新定义问题
        """
        pass
```

**Workbench 界面**：
```
当前优化目标：节能20%

[质疑这个目标] ← 随时可点击进入元层

┌─────────────────────────────────────┐
│ 元层思考                             │
├─────────────────────────────────────┤
│ 问题：                               │
│ - 节能是最终目标吗？                 │
│ - 用户真正在乎的是什么？             │
│ - 这个指标能代表用户价值吗？         │
│                                      │
│ 替代目标：                           │
│ ○ 提升用户满意度（能耗是手段）       │
│ ○ 降低总成本（不只是电费）           │
│ ○ 实现碳中和（更高层目标）           │
│                                      │
│ [重新定义目标] [保持当前目标]        │
└─────────────────────────────────────┘
```

#### 策略 4：反馈驱动的缩放学习

**从人类的缩放行为中学习**

**实现**：
```python
class ScaleLearningSystem:
    """
    记录和学习人类的缩放行为
    """
    def record_scale_switch(self, user_id, context, from_scale, to_scale, reason):
        """
        记录用户的尺度切换
        """
        self.db.insert({
            "user": user_id,
            "context": context,
            "from": from_scale,
            "to": to_scale,
            "reason": reason,
            "timestamp": now()
        })

    def learn_patterns(self):
        """
        分析模式：
        - 什么情况下，人类倾向于切换到哪个尺度？
        - 哪些尺度经常被忽视？
        """
        patterns = self.analyze_switch_patterns()
        return self.generate_prompts(patterns)

    def improve_ai_prompts(self):
        """
        基于学习到的模式，改进AI的提示词
        使AI更倾向于考虑常被忽视的尺度
        """
        pass
```

**效果**：
- AI 逐渐学会"什么时候应该考虑长期影响"
- 系统提醒越来越精准
- 但人类始终保留最终决策权

#### 策略 5：Skills 的元能力设计

**专门设计支持缩放的 Skills**

**元能力 Skills**：
```python
# 1. 尺度切换 Skill
class ScaleSwitchSkill:
    def switch_time_scale(self, from_scale, to_scale):
        """从一个时间尺度切换到另一个"""
        pass

    def switch_abstraction_level(self, from_level, to_level):
        """从一个抽象层级切换到另一个"""
        pass

# 2. 框架质疑 Skill
class FrameworkQuestionSkill:
    def question_assumption(self, assumption):
        """质疑一个假设"""
        pass

    def list_alternatives(self, current_framework):
        """列出替代框架"""
        pass

# 3. 多视角 Skill
class MultiPerspectiveSkill:
    def get_stakeholder_view(self, decision, stakeholder):
        """获取某个利益相关者的视角"""
        pass

    def compare_perspectives(self, decision, stakeholders):
        """对比多个视角"""
        pass
```

### 技术实现要点

#### 1. 数据架构

**多时间尺度的数据存储**：
```sql
-- 原始数据（秒级）
CREATE TABLE raw_data (
    id SERIAL,
    timestamp TIMESTAMPTZ,
    sensor_id VARCHAR,
    value FLOAT
);

-- 预聚合（小时级）
CREATE TABLE hourly_aggregates (
    hour TIMESTAMPTZ,
    sensor_id VARCHAR,
    avg FLOAT,
    min FLOAT,
    max FLOAT
);

-- 预聚合（天级）
CREATE TABLE daily_aggregates (...);

-- 预聚合（月级）
CREATE TABLE monthly_aggregates (...);
```

**查询优化**：
- 用户请求不同尺度时，直接查询对应的聚合表
- 快速响应，支持流畅的尺度切换

#### 2. 前端架构

**状态管理**：
```typescript
// 全局状态
interface GlobalState {
    currentScale: {
        time: 'realtime' | 'hourly' | 'daily' | 'monthly' | 'yearly',
        space: 'device' | 'zone' | 'building' | 'all',
        abstraction: 'data' | 'operation' | 'strategy' | 'goal'
    },
    scaleHistory: Scale[],  // 允许"返回上一个尺度"
    reminderConfig: {
        enabled: boolean,
        frequency: number
    }
}

// 尺度切换
function switchScale(newScale: Scale) {
    // 1. 保存当前尺度到历史
    state.scaleHistory.push(state.currentScale);

    // 2. 切换到新尺度
    state.currentScale = newScale;

    // 3. 重新加载对应尺度的数据
    refetchDataForScale(newScale);

    // 4. 记录切换行为（用于学习）
    recordScaleSwitch(user, currentContext, oldScale, newScale);
}
```

#### 3. AI Prompt 设计

**在 Operation 层的 Prompt 中嵌入缩放提示**：
```python
system_prompt = """
你是一个HVAC优化AI。在做决策时，请考虑多个尺度：

时间尺度：
- 短期（今天）：立即效果
- 中期（本月）：趋势和模式
- 长期（本年）：战略影响

空间尺度：
- 设备层：单个设备性能
- 区域层：区域间协调
- 系统层：整体优化

抽象层级：
- 数据层：传感器数据是否准确？
- 操作层：控制策略是否合理？
- 目标层：优化目标是否正确？

在给出建议时：
1. 明确你在哪个尺度上思考
2. 提醒可能在其他尺度上的风险
3. 如果不确定，说明需要人类在什么尺度上决策
"""
```

### 演进计划

**第一阶段（MVP）**：
- 多尺度视图（至少3个时间尺度）
- 基础的缩放提示
- 元层控制界面

**第二阶段**：
- 缩放学习系统
- 更智能的提示
- 完整的元能力 Skills

**第三阶段**：
- AI 主动建议缩放（但人类决策）
- 个性化的缩放偏好
- 团队协作的缩放（多人共享视图）

### 成功指标

如何判断我们弥补AI缩放局限的效果？

1. **用户行为**：
   - 用户主动切换尺度的频率（> 每次会话3次）
   - 用户采纳系统缩放提醒的比例（> 40%）

2. **决策质量**：
   - 重大决策前，考虑的尺度数量（> 3个）
   - 后悔决策的比例下降（< 5%）

3. **AI 改进**：
   - AI 提示的相关性（> 80% 用户认为有帮助）
   - AI 自发考虑多尺度的频率（通过 prompt 分析）

## 演进路线

### 第一阶段：MVP (当前)
- Workbench 基础功能
- 核心 skills（设备控制、数据查询）
- hvac-operation 基础实现

### 第二阶段：完善 (6个月)
- 完整的 AI 协作台
- 更多 skills
- 自主程度可配置

### 第三阶段：扩展 (1年)
- 支持第二个行业（照明）
- Skills 开放生态
- 边缘部署能力

## 相关文档

- [核心哲学](../PHILOSOPHY.md) - 理解为什么这样设计
- [三层架构决策](../decisions/001-三层架构设计.md) - 架构决策背景
- [产品设计原则](../product/design-principles.md) - 产品层面的应用

---

**最后更新**：2025-11-02
**维护者**：架构团队
