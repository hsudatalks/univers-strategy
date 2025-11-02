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
