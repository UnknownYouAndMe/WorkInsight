# WorkInsight Enterprise X

> 企业组织效能分析平台 / 企业 AI 运营平台
> Enterprise AI Workforce Intelligence Platform

**项目状态：🚧 开发中（v0.12.0）—— 运营增强：设备内外网 IP + 本地时区显示、日期默认当天、活跃工时排除锁屏/睡眠、自愿加班按工作日/法定节假日口径、多大模型配置（默认+优先级+回退）、设备远程操作（截屏/执行命令/消息互动·已读回执，默认全关）。此前：服务端 AI 日报、可观测/反馈/版本推送/多租户。**

---

## 这是什么

把零散的"窗口/软件/OCR"行为记录，升级为面向企业的 **AI 工作任务识别 + 组织效能分析 +
企业知识沉淀** 平台。前身是 UserActivityTracking（UAT v1.0.0）。

**核心卖点**：AI 日报/周报/月报、组织效能分析、企业知识大脑、软件资产管理、工作流自动化、企业 AI Copilot。

**不是**：单纯的"员工监控/截屏审计"软件（对外定位避开此标签，风险分析模块强制"策略可配置"）。

## 产品线

| 版本       | 价格         | 主要能力                              |
| ---------- | ------------ | ------------------------------------- |
| Lite       | 299 元/端/年 | 软件统计 / 活跃时间 / 日报            |
| Pro        | 599 元/端/年 | OCR / AI 日报周报 / 组织分析          |
| Enterprise | 999 元/端/年 | 知识库 / 审计 / 策略中心 / 资产 / SSO |
| Private AI | 定制 20万+   | 本地化部署 DeepSeek-R1 / Qwen3 / GLM  |

## 架构概览（精简栈）

针对 ~1 万终端的批量遥测负载（写多读少、可延迟），刻意**精简**为最少常驻中间件：

```text
FastAPI（后端 + Jinja2 后台）   ·   PySide6（Agent 客户端）
PostgreSQL（OLTP，开发期 SQLite）   ·   Redis（缓存 + Streams 轻量队列）
文件日志   ·   截图：文件系统 + TTL
[设计保留] ClickHouse（OLAP 分析层）   ·   pgvector（知识库向量检索）
```

被刻意**砍掉/延后**的重型件：RabbitMQ、对象存储(MinIO)、OpenSearch、独立向量库(Qdrant)、
ClickHouse（设计保留，到量再上）。理由与"再引入阈值"见 `需求方案.md` §十一 与 `CLAUDE.md` §4。

## 技术栈

- **语言**：Python 3.13
- **后端**：FastAPI + Jinja2 + SQLAlchemy
- **客户端**：PySide6（系统托盘 Agent）
- **数据**：PostgreSQL（prod）/ SQLite（dev）；ClickHouse（OLAP，延后）
- **缓存/队列**：Redis（含 Redis Streams）
- **打包**：Nuitka（客户端，CPU 基线锁 `-mcpu=x86_64`）

## 模块路线图（按落地批次）

- **批次 0 地基** ✅：Telemetry Center · License Center · Update Center（后端可运营 + Agent 采集上报）
- **批次 1 核心** 🚧：AI 工作分析中心（✅ 服务端 AI 日报 v0.11.0：OpenAI 兼容多提供商）+ Prompt Center · 组织效能分析 · 软件资产管理
- **批次 2 进阶**：企业知识大脑（pgvector）· Workflow Engine · Chat Center

详见 `需求方案.md` §十二（含商业价值 × 落地成本校准表）。

## 目录结构

```text
backend/   FastAPI 后端 + Jinja2 后台      ✅ 已落地（批次 0，可运营）
agent/     PySide6 客户端                  ✅ 七类采集 → 上报 + 自更新 + License 授权
deploy/    部署脚本                        （规划）
config/    运行时配置                      （规划）
docs/      设计文档                        （规划）
```

## 文档导航

| 看什么                     | 读哪个          |
| -------------------------- | --------------- |
| 产品/架构愿景（权威规格）  | `需求方案.md`   |
| 协作规范 / 架构总览 / 约束 | `CLAUDE.md`     |
| 会话断点续接 / 当前进度    | `PROGRESS.md`   |
| 内部研发日志（根因+决策）  | `hangelog.txt`  |
| 对外版本变更               | `changelog.txt` |

## 开发环境

```bash
# 后端（开发期 SQLite，零外部依赖）：
#   start_server.bat        或   cd backend && uvicorn app.main:app --reload
#   浏览器开 /  → 首次自动走 /install 引导建管理员，并自动注册默认应用 wie-agent
# Agent 客户端（先装依赖：.venv\Scripts\python -m pip install -r agent/requirements.txt）：
#   start_agent.bat         静默驻留托盘（pythonw）
#   python agent/main.py    带控制台日志，便于调试
```

> 后端"可运营"、Agent 首版可采集上报；运行细节见 `agent/README.md` 与 `backend/README.md`。

## 版本

当前 `0.12.0`（见 `version.txt`）。打包：客户端 `scripts/build_agent_nuitka.bat`、服务端 `scripts/build_server_nuitka.bat`
（控制台/黑窗，见 `scripts/build_server_nuitka.md`）；产物按版本目录 `dist\agent\WorkInsightAgent_v版本\` 与
`dist\server\WorkInsightServer_v版本\`。使用见 `docs/用户使用手册.md`。
