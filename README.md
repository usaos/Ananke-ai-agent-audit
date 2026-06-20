# Project Ananke v2.0.0

> AI Agent 不可篡改因果链记录与自动验证引擎 —— 数字时代的信任基石

## 项目简介

Project Ananke（命运女神计划）是一个面向 AI Agent 的不可篡改因果链记录与自动验证引擎。通过 SHA3-256 哈希链 + Ed25519 数字签名技术，为 AI Agent 的每一次决策、每一次工具调用、每一次状态变更建立不可篡改的审计记录。

## 核心特性

### 🔗 不可篡改因果链
- **SHA3-256 哈希链**：每个块包含前一个块的哈希，形成完整链式结构
- **Ed25519 数字签名**：每个块由 Agent 私钥签名，确保身份可信
- **双重时间戳**：墙钟时间（审计用）+ 单调时间（精确延迟计算）

### 📋 计划-执行一致性校验 (PCE)
- 声明式计划：Agent 预先声明执行计划和允许的工具
- 执行追踪：自动追踪实际执行的步骤
- 一致性检测：识别跳过步骤、额外步骤、顺序错误、工具越权

### 🌐 零侵入接入
- **HTTP 代理模式**：Agent 替换 base_url 即可接入，无需修改代码
- **SDK 装饰器**：一行代码接入 Python Agent
- **框架适配器**：LangChain、CrewAI、AutoGen 原生支持

### 🤝 多 Agent 协作追踪
- **Trace ID**：跨 Agent 任务追踪
- **父子事件**：事件级关联关系
- **完整链路还原**：按 trace_id 查询完整协作链路

### 📊 可视化 Web 控制台
- 仪表盘：全局统计概览
- Agent 管理：所有接入 Agent 一览
- 因果链查看器：时间线可视化
- 链验证：一键完整性验证
- 审计报告：多模板报告生成

### 📑 多合规标准审计报告
- **欧盟 AI 法案 (EU AI Act)** 合规报告
- **等保 2.0** 审计报告
- **数据安全法** 合规报告
- 支持 HTML / Markdown 双格式输出

### 🚀 企业级架构
- 统一存储架构：所有 Agent 同表存储，支持全局查询
- 高性能事件总线：批量消费、异步解耦
- 多索引优化：按 Agent、时间、类型、Trace 快速查询
- SQLite WAL 模式：高并发写入性能

## 快速开始

### 环境要求
- Python 3.11+
- pip

### 安装依赖

```bash
cd ananke-enhanced
pip install -r requirements.txt
```

### 运行完整 Demo

```bash
python examples/full_demo.py
```

### 启动 Web 控制台

```bash
python -m ananke.cli serve
```

访问 http://localhost:8765 查看 Web 控制台。

### 启动 HTTP 代理服务

```bash
python -m ananke.cli proxy --upstream-base-url https://api.openai.com
```

Agent 将 API base_url 替换为 `http://localhost:8766`，设置 `X-Ananke-Agent-Id` header 即可零侵入接入。

## 目录结构

```
ananke-enhanced/
├── ananke/
│   ├── __init__.py              # 版本声明
│   ├── core/                    # 核心层
│   │   ├── crypto.py            # 密码学引擎 (SHA3, Ed25519)
│   │   ├── causal_chain.py      # 因果链核心
│   │   ├── storage.py           # 存储层 (SQLite)
│   │   └── event_bus.py         # 事件总线
│   ├── verifier/                # 验证层
│   │   └── __init__.py          # 验证引擎 + PCE
│   ├── proxy/                   # 接入层
│   │   └── __init__.py          # HTTP 代理服务
│   ├── adapters/                # 框架适配器
│   │   └── __init__.py          # LangChain, CrewAI, AutoGen
│   ├── report/                  # 合规层
│   │   └── __init__.py          # 审计报告生成引擎
│   ├── server/                  # API 层
│   │   └── __init__.py          # REST API 服务
│   └── cli/                     # 命令行
│       └── __init__.py          # CLI 工具
├── frontend/                    # Web 控制台
│   └── index.html               # 单页应用
├── examples/                    # 示例代码
│   └── full_demo.py             # 完整功能演示
├── tests/                       # 测试用例
│   └── test_core.py             # 核心模块单元测试
├── data/                        # 数据目录 (运行时生成)
├── requirements.txt             # 依赖清单
├── Dockerfile                   # Docker 镜像
├── docker-compose.yml           # Docker Compose 部署
└── README.md                    # 项目文档
```

## 使用示例

### 1. 基本使用（SDK 方式）

```python
from ananke.core.causal_chain import CausalChain, CausalEvent
from ananke.core.storage import SQLiteBlockStore

# 创建存储
storage = SQLiteBlockStore(db_path="./data/ananke.db")

# 创建 Agent 因果链
chain = CausalChain("my-agent", storage=storage)
chain.initialize_identity()

# 记录输入
event = CausalEvent(
    agent_id="my-agent",
    event_type="INPUT",
    payload={"input": "用户问题"},
)
chain.append(event)

# 记录工具调用
event = CausalEvent(
    agent_id="my-agent",
    event_type="TOOL_CALL",
    payload={"tool_name": "search", "tool_args": {"q": "..."}},
)
chain.append(event)

# 记录输出
event = CausalEvent(
    agent_id="my-agent",
    event_type="OUTPUT",
    payload={"output": "回答内容"},
)
chain.append(event)

print(f"已记录 {chain.block_count} 个因果块")
```

### 2. 链完整性验证

```python
from ananke.verifier import AnankeVerifier

verifier = AnankeVerifier()
verifier.set_public_key(chain.public_key)

blocks = chain.get_chain()
result = verifier.verify_chain_summary(blocks)

print(f"验证结果: {'通过' if result['overall_passed'] else '未通过'}")
print(f"通过: {result['passed']} / {result['total']}")
```

### 3. 计划-执行一致性校验

```python
from ananke.verifier import PlanConsistencyEngine

pce = PlanConsistencyEngine()
result = pce.check_consistency(blocks)

print(f"计划步骤: {result['plan_steps_count']}")
print(f"执行步骤: {result['executed_steps_count']}")
print(f"跳过步骤: {result['skipped_steps']}")
```

### 4. 生成审计报告

```python
from ananke.report import AnankeReportEngine

report_engine = AnankeReportEngine()

# 欧盟 AI 法案合规报告
html_report = report_engine.generate_report(
    "eu_ai_act", "my-agent", blocks, verifier
)

# 等保 2.0 审计报告
db_report = report_engine.generate_report(
    "dengbao_2.0", "my-agent", blocks, verifier
)

# Markdown 简化报告
md_report = report_engine.generate_markdown_report(
    "my-agent", blocks, verifier
)
```

### 5. LangChain 适配器

```python
from ananke.adapters import create_langchain_callback

# 创建回调
callback = create_langchain_callback("my-langchain-agent")

# 在 LangChain 中使用
llm = OpenAI(callbacks=[callback])
result = llm.invoke("你好")
```

## CLI 命令

```bash
# 查看所有 Agent
python -m ananke.cli chain list

# 查看 Agent 因果链
python -m ananke.cli chain show my-agent

# 验证链完整性
python -m ananke.cli verify my-agent

# 生成审计报告
python -m ananke.cli report my-agent --template eu_ai_act

# 启动 API 服务 + Web 控制台
python -m ananke.cli serve --port 8765

# 启动 HTTP 代理服务
python -m ananke.cli proxy --port 8766
```

## Docker 部署

### 使用 Docker Compose

```bash
# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f

# 停止
docker-compose down
```

服务端口：
- API + Web 控制台：8765
- HTTP 代理：8766

## API 接口

### 健康检查
- `GET /health`

### Agent 管理
- `GET /api/v1/agents` - 列出所有 Agent
- `GET /api/v1/agents/{agent_id}` - 获取 Agent 详情
- `PUT /api/v1/agents/{agent_id}` - 更新 Agent 信息

### 因果链
- `POST /api/v1/events` - 创建事件
- `GET /api/v1/agents/{agent_id}/chain` - 查询因果链
- `GET /api/v1/traces/{trace_id}` - 按 Trace 查询

### 验证
- `POST /api/v1/verify` - 验证链完整性
- `GET /api/v1/agents/{agent_id}/verify` - 快速验证

### 报告
- `GET /api/v1/report/templates` - 列出报告模板
- `POST /api/v1/report/generate` - 生成审计报告

### 统计
- `GET /api/v1/stats/overview` - 全局统计
- `GET /api/v1/stats/agent/{agent_id}` - Agent 统计

完整 API 文档：启动服务后访问 `/docs`（Swagger UI）

## 技术栈

- **语言**：Python 3.11+
- **Web 框架**：FastAPI + Uvicorn
- **密码学**：cryptography (SHA3-256, Ed25519)
- **存储**：SQLite WAL 模式
- **前端**：原生 HTML/CSS/JS（无构建依赖）
- **CLI**：Click
- **部署**：Docker + Docker Compose

## 版本历史

### v2.0.0 增强版
- ✅ 时间戳体系重构（墙钟 + 单调双时间戳）
- ✅ 统一存储架构（单表多 Agent）
- ✅ 高性能事件总线
- ✅ 零侵入 HTTP 代理接入
- ✅ LangChain / CrewAI / AutoGen 适配器
- ✅ 计划-执行一致性校验引擎 (PCE)
- ✅ 多合规标准审计报告
- ✅ 可视化 Web 控制台
- ✅ 完整 REST API
- ✅ CLI 命令行工具
- ✅ Docker 部署支持

### v1.0.0 初始版
- 基础因果链记录
- SHA3-256 + Ed25519
- 基础验证规则
- SDK + REST API



欢迎提交 Issue 和 Pull Request！

---

**Project Ananke** — 让 AI Agent 的每一步都可追溯、可验证、可审计。
